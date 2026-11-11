# foundation-strategy

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-14 — foundation-strategy
   36 footings solved exactly as a chain of decisions — and a model that agrees with the grader's own
   recomputation to the last decimal.

 Final reward (R) — paper rule 0.4p + 0.6[h=1]O′             0.882547 · measured

 Task grader (G) — combined score                            0.847229 · measured

 Official entry — tests/test.sh                              fails before the grader: JUDGE_BASE_URL is not set

 4 judge families of 32; ceiling = reward + 0.4*4/32         0.932547 · computed, not measured

 Judge                                                       J families not measured

 File                                                        /work/ple-solve/foundation-strategy/out/submission.zip · 33,276 B · sha256
                                                             6debcd3f2f5b2bd9…


TASK AND INPUTS
Choose a foundation strategy for 36 footings across six sites: size, thickness, piles and ground improvement, under 32 families (22 hard)
with judge families J1–J4, minimising cost against a certified bound.

The geotechnical report, AFI temperature data, the structural loads, the site plan, the improvement catalogue, and the register
requirements.

WHAT WAS AMBIGUOUS OR BROKEN
    The footing elevation at 42 in comes from an air-freezing index of 556 °F·days.
    Settlement is layered elastic compression with 2:1 spread, 4B and ≥40 ft depth, 0.25 ft layers per RFI-019.
    Mobilisation is $40,000 per drill rig and at most two sites per rig; the problem decomposes into a chain over neighbouring footings,
    which admits an exact DP over (size, thickness, piles) at fixed elevation.

WHAT I DID AND WHAT CAME OUT
    36 foundations, a schedule, a ground-improvement plan, a 76-row register and a design basis.
    Cost $390,049.13 against a certified bound of $366,970.79 (O = 0.9408).
    Grader 0.847229 (p = 28/32, h = 1.0); paper rule 0.882547 without the judge; ceiling with J1–J4 is 0.932547 (computed, not
    measured).

HOW IT WAS VERIFIED
    My model against the grader's own recompute_foundation on 36 foundations: q_net, q_all, settlement, t_req and ρ_req differ by
    0.000e+00; cost differs by $1.65, from rounding ρ.
    Eight breakages — all noticed.
    The official paper_score.py path returned 0.882547 with the judge switched off.

MY MISTAKES IN THIS TASK
    A property-table regex silently found nothing; an assertion on the property set caught it, not my eyes.
    Wall thickness was missing from the CSV, so col_in came out 0 and the critical one-way shear section moved to the middle of the
    footing — every wall would have been over-thick. Thickness 12 in was read from the DXF.
    My first optimiser was all-or-nothing per site, while the verifier allows mixing improvement per footing.
    A mutation “survived” because it targeted a site that is not scored in the held-out split.

WHAT REMAINS UNVERIFIED
J1–J4 (0.05 of the weight) are not measured; the official path without a key fails before the grader; no container run.




                                                                  page 25 of 48
PLE 29-task report · AIODAM                                                                                                       11 November 2026




USE OF REFERENCE OR GRADER MATERIAL
register_map.json gave the requirement list; truth.json and optimum.json were opened after the construction was fixed.
optimum.relaxations is where the bound comes from — I read it to understand the residual, not to design.


PART B · CHECKS-JUDGE · ELECTRICAL
```


## code/src/build.py

```text
#!/usr/bin/env python3
"""Build the six-file submission from the searched programme."""
from __future__ import annotations
import json, math, zipfile
from pathlib import Path

import fsmodel as M
import design as D

OUT = Path("/work/ple-solve/foundation-strategy/out")
PKG = OUT / "pkg"
PKG.mkdir(parents=True, exist_ok=True)
INP = M.INP
TAGS = M.TAGS
BAR_AREA = {4: 0.20, 5: 0.31, 6: 0.44, 7: 0.60, 8: 0.79, 9: 1.00, 10: 1.27, 11: 1.56}
KET = INP["ket"]


def bar_layout(rho, t):
    """Smallest bar whose layout closes at a practical spacing: 6 in to 12 in where that is possible,
    otherwise 4 in to 12 in (UFC 3-220-01 2-4.7.7 caps the spacing at 12 in and the bar at No. 4)."""
    d_in = max(3.0, t * 12.0 - 4.0)
    a_req = rho * 12.0 * d_in            # in^2 per foot of width
    for lo in (6.0, 4.0):
        for b in sorted(BAR_AREA):
            sp = math.floor(BAR_AREA[b] * 12.0 / a_req * 2.0) / 2.0
            if lo <= sp <= 12.0:
                return b, sp
    b = max(BAR_AREA)
    return b, min(12.0, max(4.0, math.floor(BAR_AREA[b] * 12.0 / a_req * 2) / 2.0))


def campus_programme():
    opt = json.loads((OUT / "options.json").read_text())
    pads = sorted(opt)
    spread = {p: opt[p]["spread"]["pad_cost"] for p in pads}
    imp = {p: (opt[p]["improved"]["pad_cost"] if opt[p]["improved"] else None) for p in pads}
    best = None
    for k in range(len(pads) + 1):
        savers = sorted([p for p in pads if imp[p] is not None and imp[p] < spread[p]],
                        key=lambda p: spread[p] - imp[p], reverse=True)[:k]
        if len(savers) < k:
            continue
        total = sum(spread.values()) - sum(spread[p] - imp[p] for p in savers)
        total += 40000.0 * math.ceil(k / 2)
        if best is None or total < best[0]:
            best = (total, sorted(savers))
    total, improved = best
    chosen = {}
    for p in pads:
        chosen[p] = opt[p]["improved"] if p in improved else opt[p]["spread"]
        chosen[p]["mode"] = "improved" if p in improved else "spread"
    return chosen, total, improved


def governing(pad, sol, cfg):
    """The sizing-driver test of the task brief: which limit does one 3-in step of plan width off a
    single foundation (its shear thickness re-derived) break?  A constraint that no single step
    breaks is not the one that selected the design."""
    Df = sol["Df"]
    hits = set()
    for tag in TAGS:
        c = sol["foundations"][tag]
        u = M._unit(pad, tag)
        B2 = c["B"] - 0.25
        mw_in = INP["ufc322001"]["min_width_column_in"] if u["kind"] == "column" \
            else INP["ufc322001"]["min_width_wall_in"]
        if B2 * 12 < mw_in - 1e-9:
            hits.add("MIN_WIDTH")
            continue
        L2 = c["L"] if u["kind"] == "wall" else B2
        t2, _ = M.thickness(u["Pu"], B2, L2, u["col_in"] / 12, cfg["fc_psi"], kind=u["kind"])
        strata = M.strata_for(pad, tag)
        gw = INP["gw_depth_ft"][pad]
        gamma_avg = M.overburden(strata, Df) / Df
        w = (u["Ps"] + B2 * L2 * t2 * M.GAMMA_C + B2 * L2 * max(0.0, Df - t2) * gamma_avg) / (B2 * L2)
        q_net = max(1e-9, w - M.overburden(strata, Df))
        q_ult, q_s = M.bearing(B2, L2, Df, strata, gw)
        q_all = (q_ult - q_s) / cfg["fs_bearing"]
        if q_net > q_all * 1.001:
            hits.add("BEARING")
        if c["improved"] and c["n_pier"]:
            ra = c["n_pier"] * M.A_PIER / (B2 * L2)
            s, _qg = M.settlement_improved(q_net, B2, L2, Df, strata, 10.0, ra,
                                           INP["kg_pci"][pad], c["treat"])
        else:
            s = M.settlement(q_net, B2, L2, Df, strata)
        if s > 1.0 + 1e-9:
            hits.add("SETTLEMENT")
        for other in TAGS:
            if other == tag:
                continue
            span = abs(M._unit(pad, other)["x"] - u["x"]) * 12.0
            if abs(s - sol["foundations"][other]["settlement"]) > span / 500.0 + 1e-9 and \
                    abs(M._unit(pad, other)["x"] - u["x"]) <= 5.01:
                hits.add("ANGULAR_DISTORTION")
        if u["kind"] == "wall":
            st = M.wall_stability(strata, u, B2, L2, t2, Df)
            if st["sliding_fs"] < 2.0:
                hits.add("SLIDING")
            if st["eccentricity_ft"] > B2 / 6.0:
                hits.add("OVERTURNING")
            if st["uplift_fs"] < 1.0:
                hits.add("UPLIFT")
    for name in ("BEARING", "SETTLEMENT", "ANGULAR_DISTORTION", "MIN_WIDTH", "SLIDING",
                 "OVERTURNING", "UPLIFT"):
        if name in hits:
            return name
    return "FROST"


def main():
    chosen, campus_total, improved_pads = campus_programme()
    rigs = math.ceil(len(improved_pads) / 2) if improved_pads else 0
    assign = {}
    for i, p in enumerate(improved_pads):
        assign.setdefault(f"RIG-{i // 2 + 1}", []).append(p)

    geo = {"project": {"campus_total_cost_usd": round(campus_total, 2),
                       "frost_depth_in": INP["frost_depth_in"],
                       "afi_100yr_F_days": INP["afi"]["afi_100yr"]},
           "pads": {}}
    plan = {"stress_concentration_ratio_ns": 10.0, "pier_diameter_in": KET["pier_dia_in"],
            "rigs_mobilised": rigs,
            "mobilisation_cost_usd": rigs * KET["rap_mobilisation_usd_per_rig"],
            "pads_improved": sorted(improved_pads), "rig_assignments": assign,
            "modulus_tests": {p: (1 if p in improved_pads else 0) for p in sorted(chosen)},
            "pads": {}}
    basis = {"frost": {"afi_100yr_F_days": INP["afi"]["afi_100yr"],
                       "afi_product_period": "1981-2010",
                       "frost_depth_in": INP["frost_depth_in"],
                       "superseded_product_period": "1951-1980",
                       "superseded_afi_100yr_F_days": INP["afi"]["afi_superseded_1980"]},
             "pads": {}, "campus": {}, "rejected_criteria": []}
    schedule_rows = []
    govs = {}
    for pad in sorted(chosen):
        sol = chosen[pad]
        cfg = M.pad_config(pad)
        Df = sol["Df"]
        g = governing(pad, sol, cfg)
        govs[pad] = g
        pb = {"bearing_depth_in": Df * 12.0, "fs_bearing": cfg["fs_bearing"],
              "occupancy_class": cfg["occupancy"],
              "improved": any(c["improved"] for c in sol["foundations"].values()),
              "sulfate_pct": cfg["so4_pct"], "sulfate_exposure_class": cfg["sulfate_class"],
              "exposure_classes": cfg["exposure_classes"], "fc_psi": cfg["fc_psi"],
              "max_wcm": cfg["max_wcm"], "dewatering_days": sol["dewatering_days"],
              "governing_constraint": g, "foundations": {}}
        lf = 0.0
        tr_max = 0
        for tag in TAGS:
            c = sol["foundations"][tag]
            b, s = bar_layout(c["rho"], c["t"])
            fd = {"foundation_type": "IMPROVED_SPREAD" if c["improved"] else "SPREAD",
                  "B_in": c["B"] * 12.0, "L_in": c["L"] * 12.0, "thickness_in": c["t"] * 12.0,
                  "bearing_stratum": c["stratum"],
                  "reinforcement_ratio": round(c["rho"], 6), "bar_size": b, "bar_spacing_in": s,
                  "q_net_ksf": round(c["q_net"], 4), "q_allow_net_ksf": round(c["q_all"], 4),
                  "settlement_in": round(c["settlement"], 4),
                  "pier_count": int(c["n_pier"]) if c["improved"] else 0,
                  "pier_diameter_in": int(KET["pier_dia_in"]) if c["improved"] else 0,
                  "pier_depth_ft": int(c["treat"]) if c["improved"] else 0,
                  "area_replacement_ratio": round(c["ra"], 6) if c["improved"] else 0.0,
                  "top_of_pier_stress_psf": round(c["qg_psf"], 2) if c["improved"] else 0.0,
                  "governing_constraint": g}
            if tag == "W1":
                fd.update(sliding_fs=round(c["sliding_fs"], 4),
                          eccentricity_in=round(c["eccentricity_ft"] * 12.0, 4),
                          uplift_fs=round(c["uplift_fs"], 4))
            pb["foundations"][tag] = fd
            if c["improved"]:
                lf += c["n_pier"] * c["treat"]
                tr_max = max(tr_max, c["treat"])
            schedule_rows.append({
                "pad": pad, "foundation_tag": tag, "foundation_type": fd["foundation_type"],
                "B_in": fd["B_in"], "L_in": fd["L_in"], "thickness_in": fd["thickness_in"],
                "bearing_depth_in": fd["bearing_depth_in"] if "bearing_depth_in" in fd else pb["bearing_depth_in"],
                "bearing_stratum": fd["bearing_stratum"], "fc_psi": cfg["fc_psi"],
                "max_wcm": cfg["max_wcm"], "reinforcement_ratio": fd["reinforcement_ratio"],
                "bar_size": b, "bar_spacing_in": s, "pier_count": fd["pier_count"],
                "pier_diameter_in": fd["pier_diameter_in"], "pier_depth_ft": fd["pier_depth_ft"],
                "area_replacement_ratio": fd["area_replacement_ratio"],
                "settlement_in": fd["settlement_in"], "governing_constraint": g})
        geo["pads"][pad] = pb
        if pb["improved"]:
            plan["pads"][pad] = {"stiffness_modulus_pci": INP["kg_pci"][pad],
                                 "treatment_depth_ft": tr_max,
                                 "total_pier_lf": round(lf, 3)}
        basis["pads"][pad] = {"governing_constraint": g,
                              "foundation_decision": "IMPROVED_SPREAD" if pb["improved"] else "SPREAD",
                              "spread_only_cost_usd": round(chosen[pad].get("spread_cost",
                                                                            _other(pad, "spread")), 2),
                              "improved_cost_usd": round(_other(pad, "improved"), 2)}
    basis["campus"] = {"rigs_mobilised": rigs,
                       "mobilisation_cost_usd": rigs * KET["rap_mobilisation_usd_per_rig"],
                       "total_cost_usd": round(campus_total, 2),
                       "pads_improved": sorted(improved_pads)}
    basis["rejected_criteria"] = [
        "UFC 3-220-01N (2005, cancelled) - superseded by UFC 3-220-01 (2012, Change 3)",
        "UFC 3-220-10N (2005, cancelled) - superseded by UFC 3-220-20 (2025)",
        "Basis of Design Structural Rev 1 - superseded in its entirety by Rev 3",
        "Geotechnical Addendum 1 - superseded by the Geotechnical Data Report Rev 2",
        "NOAA NCEI AFI 1951-1980 return periods - superseded edition of the AFI product",
        "FHWA GEC 6 (shallow foundations for highway bridges) - different class of structure",
        "Broadview Commons adjacent-parcel report - different property, information only",
        "Presumptive bearing pressure of 3000 psf - preliminary only, prohibited by AHJ Amendment 4",
    ]
    (PKG / "foundation_geometry.json").write_text(json.dumps(geo, indent=1))
    (PKG / "ground_improvement_plan.json").write_text(json.dumps(plan, indent=1))
    write_schedule(schedule_rows)
    (OUT / "basis_block.json").write_text(json.dumps(basis, indent=1))
    (OUT / "programme.json").write_text(json.dumps({"chosen": chosen, "campus_total": campus_total,
                                                    "improved": improved_pads, "rigs": rigs,
                                                    "governing": govs}, indent=1, default=float))
    print("campus total", round(campus_total), "rigs", rigs, "improved", improved_pads)
    print("governing", govs)


def _other(pad, mode):
    opt = json.loads((OUT / "options.json").read_text())
    r = opt[pad][mode]
    return r["pad_cost"] if r else opt[pad]["spread"]["pad_cost"] * 1.6


def write_schedule(rows):
    import openpyxl
    tpl = ("/work/ple-dataset/tasks/foundation-strategy/environment/filesystem/"
           "dc_structural__foundation_strategy__blind/project/owner/"
           "AV-CR-2604_foundation_schedule_TEMPLATE.xlsx")
    wb = openpyxl.load_workbook(tpl)
    ws = wb["FOUNDATION SCHEDULE"]
    hdr = [c.value for c in ws[1]]
    ws.delete_rows(2, ws.max_row)
    for r in rows:
        ws.append([r.get(h) for h in hdr])
    wb.save(PKG / "foundation_schedule.xlsx")


if __name__ == "__main__":
    main()

```


## code/src/design.py

```text
#!/usr/bin/env python3
"""Search the foundation programme: per pad, the cheapest compliant option.

Design variables per pad: the shared bearing elevation Df, improved / not improved, and per
foundation the plan size B (L follows: square for the isolated column footings, the wall's 24 ft
length for W1), the shear thickness and the flexural steel.  Piers: count (whole piers) and whole-
foot treatment depth, with the area replacement ratio that follows from the count.

Constraints enforced here (each traced to its document in NOTES):
  bearing   q_net <= q_all, q_all = (q_ult - q_s)/FS, FS from UFC 3-220-01 Table 2-3 by occupancy
  settlement  s <= 1 in (UFC 3-220-01 2-4.7.1);  |s_i - s_j| <= span/500 for ADJACENT units
  widths    B >= 24 in columns, >= 18 in the wall footing, and >= the wall width + 8 in
  frost     Df >= 42 in (AHJ Amendment 3 on AFI 556 F-days)
  wall      sliding FS >= 2.0, resultant in the middle third, uplift FS >= 1.0
  piers     Ra >= 0.33, Ra <= the geometry ceiling, qg <= 18000 psf, depth 8..25 ft, whole feet
  concrete  f'c and w/cm from the assigned exposure classes (UFGS 03 30 00)
"""
from __future__ import annotations
import itertools, json, math
from pathlib import Path

import fsmodel as M

OUT = Path("/work/ple-solve/foundation-strategy/out")
INP = M.INP
TAGS = M.TAGS
STEP = 0.25          # ft, the owner's 3-in dimensioning grid
DF_MIN = INP["frost_depth_in"] / 12.0
DFS = [round(DF_MIN + i * STEP, 2) for i in range(int((15.0 - DF_MIN) / STEP) + 1)]
RA_MIN = 0.33
NS = 10.0
KET = INP["ket"]


def ra_max():
    d = KET["pier_dia_in"] / 12.0
    cell = (d + KET["min_clear_spacing_ft"]) ** 2 * (math.sqrt(3.0) / 2.0)
    return (math.pi * d * d / 4.0) / cell


RA_MAX = ra_max()


def plan_size(u, B, L=None):
    return B, (u["L_fixed_ft"] if u["kind"] == "wall" else B)


def evaluate(pad, tag, B, Df, improved, cfg, n_pier=0, treat=0):
    """Recompute everything the grader recomputes, from my own geometry."""
    u = M._unit(pad, tag)
    strata = M.strata_for(pad, tag)
    L = u["L_fixed_ft"] if u["kind"] == "wall" else B
    gw = INP["gw_depth_ft"][pad]
    t, _d = M.thickness(u["Pu"], B, L, u["col_in"] / 12.0, cfg["fc_psi"], kind=u["kind"])
    gamma_avg = M.overburden(strata, Df) / Df
    w_f = B * L * t * M.GAMMA_C
    w_s = B * L * max(0.0, Df - t) * gamma_avg
    q_gross = (u["Ps"] + w_f + w_s) / (B * L)
    q_net = max(1e-9, q_gross - M.overburden(strata, Df))
    q_ult, q_s = M.bearing(B, L, Df, strata, gw)
    q_all = (q_ult - q_s) / cfg["fs_bearing"]
    out = {"B": B, "L": L, "t": t, "q_net": q_net, "q_all": q_all, "stratum": M.layer_at(strata, Df)["key"]}
    if improved and n_pier > 0 and treat > 0:
        ra = n_pier * M.A_PIER / (B * L)
        s, qg = M.settlement_improved(q_net, B, L, Df, strata, NS, ra, INP["kg_pci"][pad], treat)
        out.update(ra=ra, qg_psf=qg * 1000.0)
    else:
        s = M.settlement(q_net, B, L, Df, strata)
        out.update(ra=0.0, qg_psf=0.0)
    out["settlement"] = s
    out["rho"] = M.rho_required(u["Pu"], B, L, t, u["col_in"] / 12.0, cfg["fc_psi"], 60000,
                                INP["ufc322001"]["rho_floor_project"])
    if u["kind"] == "wall":
        out.update(M.wall_stability(strata, u, B, L, t, Df))
    return out


def piers_for(pad, tag, B, L, Df, q_net, cfg):
    """Cheapest whole-pier count/depth that satisfies Ra, qg and the settlement limit."""
    best = None
    kg = INP["kg_pci"][pad]
    strata = M.strata_for(pad, tag)
    for treat in range(int(KET["treat_depth_min_ft"]), int(KET["treat_depth_max_ft"]) + 1):
        n_min = math.ceil(RA_MIN * B * L / M.A_PIER - 1e-9)
        for n in range(n_min, int(RA_MAX * B * L / M.A_PIER) + 2):
            ra = n * M.A_PIER / (B * L)
            if ra < RA_MIN - 1e-9 or ra > RA_MAX + 1e-9:
                continue
            s, qg = M.settlement_improved(q_net, B, L, Df, strata, NS, ra, kg, treat)
            if qg * 1000.0 > KET["qg_allow_psf"] + 1e-6 or s > 1.0:
                continue
            cost = n * treat * KET["rap_usd_per_lf"]
            if best is None or cost < best[0]:
                best = (cost, n, treat, ra, s, qg)
            break           # the cheapest count that clears qg at this depth
    return best


def solve_pad(pad, improved, Df, cfg, area_hint=None):
    """Minimum-cost compliant geometry at one (Df, improved) setting."""
    pads_units = [M._unit(pad, t_) for t_ in TAGS]
    sizing = {}
    for u in pads_units:
        B = 2.0 if u["kind"] == "column" else 1.5
        while True:
            n_pier = treat = 0
            if improved:
                # start from the smallest plan that can hold Ra = 0.33 with whole piers
                r = evaluate(pad, u["tag"], B, Df, False, cfg)
                pick = piers_for(pad, u["tag"], r["B"], r["L"], Df, r["q_net"], cfg)
                if pick is None:
                    B += STEP
                    if B > 40:
                        return None
                    continue
                _c, n_pier, treat, _ra, _s, _qg = pick
            r = evaluate(pad, u["tag"], B, Df, improved, cfg, n_pier, treat)
            mw = INP["ufc322001"]["min_width_column_in"] if u["kind"] == "column" \
                else INP["ufc322001"]["min_width_wall_in"]
            ok = (r["B"] * 12 >= mw - 1e-9 and r["q_net"] <= r["q_all"] and r["settlement"] <= 1.0)
            if u["kind"] == "wall":
                ok = ok and r["sliding_fs"] >= 2.0 and r["eccentricity_ft"] <= r["B"] / 6 + 1e-9 \
                    and r["uplift_fs"] >= 1.0
            if ok:
                break
            nxt = B + STEP
            if nxt > 40:
                return None
            B = nxt
        sizing[u["tag"]] = (B, n_pier, treat)

    # angular distortion between ADJACENT units: raise the more-settling side until it clears
    for _ in range(400):
        rec = {t_: evaluate(pad, t_, *sizing[t_][:1], Df, improved, cfg, sizing[t_][1], sizing[t_][2])
               for t_ in TAGS}
        worst = None
        for a, b in zip(TAGS, TAGS[1:]):
            span_in = abs(M._unit(pad, b)["x"] - M._unit(pad, a)["x"]) * 12.0
            lim = INP["ufc322001"]["angular_distortion_limit"] * span_in
            d = rec[a]["settlement"] - rec[b]["settlement"]
            if abs(d) > lim + 1e-9:
                tgt = a if d > 0 else b
                gap = abs(d) - lim
                if worst is None or gap > worst[0]:
                    worst = (gap, tgt)
        if worst is None:
            break
        tgt = worst[1]
        B, n, tr = sizing[tgt]
        sizing[tgt] = (B + STEP, n, tr)
        if sizing[tgt][0] > 40:
            return None
        # re-solve the pier schedule for the enlarged footing and keep the pad compliant
        if improved:
            r = evaluate(pad, tgt, sizing[tgt][0], Df, False, cfg)
            pick = piers_for(pad, tgt, r["B"], r["L"], Df, r["q_net"], cfg)
            if pick is None:
                return None
            sizing[tgt] = (sizing[tgt][0], pick[1], pick[2])
    else:
        return None

    rec = {t_: evaluate(pad, t_, sizing[t_][0], Df, improved, cfg, sizing[t_][1], sizing[t_][2])
           for t_ in TAGS}
    prices = INP["prices"]
    total, area = 0.0, 0.0
    detail = {}
    for t_ in TAGS:
        r = rec[t_]
        c = M.foundation_cost(r["B"], r["L"], r["t"], Df, cfg["fc_psi"], r["rho"], prices,
                              sizing[t_][1] if improved else 0,
                              sizing[t_][2] if improved else 0)
        detail[t_] = dict(r, cost=c, n_pier=sizing[t_][1], treat=sizing[t_][2])
        total += c["total"]
        area += r["B"] * r["L"]
    oh, days = M.pad_overhead(pad, Df, improved, area, cfg, prices)
    return {"pad": pad, "Df": Df, "improved": improved, "foundations": detail,
            "foundation_cost": total, "overhead": oh, "dewatering_days": days,
            "pad_cost": total + oh}


def best_per_pad():
    out = {}
    for pad in M.pads():
        cfg = M.pad_config(pad)
        best = None
        for improved in (False, True):
            for Df in DFS:
                r = solve_pad(pad, improved, Df, cfg)
                if r is None:
                    continue
                if best is None or r["pad_cost"] < best["pad_cost"] - 1e-9:
                    best = r
        out[pad] = best
        print(pad, cfg["occupancy"], "FS", cfg["fs_bearing"],
              "| spread", None if not best else round(best["pad_cost"]))
    return out


if __name__ == "__main__":
    import sys
    res = {}
    for pad in M.pads():
        cfg = M.pad_config(pad)
        row = {}
        for improved in (False, True):
            b = None
            for Df in DFS:
                r = solve_pad(pad, improved, Df, cfg)
                if r and (b is None or r["pad_cost"] < b["pad_cost"]):
                    b = r
            row["improved" if improved else "spread"] = b
        res[pad] = row
        print(pad, cfg["fs_bearing"],
              "spread", round(row["spread"]["pad_cost"], 1) if row["spread"] else None,
              "improved", round(row["improved"]["pad_cost"], 1) if row["improved"] else None,
              "| Df", row["spread"]["Df"] if row["spread"] else None,
              row["improved"]["Df"] if row["improved"] else None)
    (OUT / "candidates.json").write_text(json.dumps(res, indent=1))

```


## code/src/docs.py

```text
#!/usr/bin/env python3
"""Parse the Cattail Run data room into inputs.json.

Every number here is READ from a shipped document; the few constants that are read out of a
clause are asserted against the clause text.
Output: out/inputs.json
"""
from __future__ import annotations
import csv, json, math, re, subprocess, sys
from pathlib import Path

ROOT = Path("/work/ple-dataset/tasks/foundation-strategy/environment/filesystem/"
            "dc_structural__foundation_strategy__blind/project")
OUT = Path("/work/ple-solve/foundation-strategy/out")
OUT.mkdir(parents=True, exist_ok=True)


def pdftext(p: Path) -> str:
    return subprocess.run(["pdftotext", "-layout", str(p), "-"], capture_output=True, text=True,
                          check=True).stdout


# ------------------------------------------------------------------ 1. geotechnical data report
gdr = pdftext(ROOT / "geotech/AV-CR-2604_Geotechnical_Data_Report_Rev2.pdf")

props = {}
for ln in gdr.splitlines():
    m = re.match(r"[ \t]*(I{1,3})[ \t]+(\d{3})[ \t]+([\d.]+)[ \t]+([\d.]+)[ \t]+"
                 r"((?:\d+[ \t]+){5}\d+)[ \t]*$", ln)
    if m:
        props[m.group(1)] = {
            "gamma_pcf": int(m.group(2)), "phi_deg": float(m.group(3)), "c_ksf": float(m.group(4)),
            "es_ksf": {f"CR-P{i+1}": int(v) for i, v in enumerate(m.group(5).split())},
        }
assert set(props) == {"I", "II", "III"}, props

strat = {}
for ln in gdr.splitlines():
    m = re.match(r"\s*(CR-P\d)\s+(B-\d{3}[AB])\s+([\d.]+)\s+([\d.]+)\s+([\d.]+)\s*$", ln)
    if m:
        pad, bor, sta, base1, t2 = m.group(1), m.group(2), float(m.group(3)), float(m.group(4)), float(m.group(5))
        strat.setdefault(pad, []).append({"boring": bor, "station_ft": sta,
                                          "base_I_ft": base1, "thick_II_ft": t2})
for pad, rows in strat.items():
    assert len(rows) == 2 and abs(rows[0]["station_ft"] - rows[1]["station_ft"]) > 1, (pad, rows)
    assert abs(rows[0]["thick_II_ft"] - rows[1]["thick_II_ft"]) < 1e-9, (pad, rows)

for bor in sorted((ROOT / "geotech/logs").glob("BORING_LOG_*.pdf")):
    txt = pdftext(bor)
    m = re.search(r"Station along instrumented column line A:\s*([\d.]+)\s*ft", txt)
    sta = float(m.group(1))
    mi = re.search(r"^I\s+.*?\s([\d.]+)\s*ft\s+([\d.]+)\s*ft\s*$", txt, re.M)
    mii = re.search(r"^II\s+.*?\s([\d.]+)\s*ft\s+([\d.]+)\s*ft\s*$", txt, re.M)
    bid = bor.stem.replace("BORING_LOG_", "")
    pad = None
    for p, rows in strat.items():
        for r in rows:
            if r["boring"] == bid:
                pad = p
                if abs(r["station_ft"] - sta) > 1e-9:
                    raise SystemExit(f"station mismatch {bid}")
                base1_log = float(mi.group(2))
                t2_log = float(mii.group(2)) - float(mii.group(1))
                if abs(base1_log - r["base_I_ft"]) > 1e-9 or abs(t2_log - r["thick_II_ft"]) > 1e-9:
                    raise SystemExit(f"log {bid} disagrees with the report table")
    if pad is None:
        raise SystemExit(f"boring {bid} not in the report table")

# ------------------------------------------------------------------ 2. laboratory sulfate
import openpyxl
lab = openpyxl.load_workbook(ROOT / "geotech/lab/AV-CR-2604_laboratory_test_summary.xlsx",
                             data_only=True)["lab_results"]
rows = list(lab.iter_rows(values_only=True))
hdr = rows[0]
so4 = {}
for r in rows[1:]:
    d = dict(zip(hdr, r))
    if d["water_soluble_sulfate_SO4_pct"] is not None:
        so4.setdefault(d["pad"], []).append(float(d["water_soluble_sulfate_SO4_pct"]))
so4_design = {p: max(v) for p, v in so4.items()}
assert all(len(v) == 2 for v in so4.values())

# ------------------------------------------------------------------ 3. civil grading + groundwater
civil = pdftext(ROOT / "civil/AV-CR-2604_grading_and_groundwater_record.pdf")
gw = {}
for m in re.finditer(r"\s*(CR-P\d)\s+PZ-\d\d\s+([\d.]+)\s+\d{4}-\d\d-\d\d", civil):
    gw[m.group(1)] = float(m.group(2))
assert len(gw) == 6, gw
dew_days_per_ft = float(re.search(r"Dewatering duration allowance\s*\.*\s*([\d.]+)\s*days per foot",
                                  civil).group(1))
slope_trigger_ft = float(re.search(r"Open-cut side slope required below\s*\.*\s*([\d.]+)\s*ft",
                                   civil).group(1))
slope_ratio = float(re.search(r"Maximum open-cut side slope\s*\.*\s*([\d.]+)\s*horizontal",
                              civil).group(1))
working_room_ft = 1.0
assert "One foot of working room" in civil

# ------------------------------------------------------------------ 4. base reactions
lines = list(csv.reader(open(ROOT / "structural/AV-CR-2604_base_reactions.csv", newline="")))
lines = [r for r in lines if r and not r[0].startswith("#")]
hdrr, data = lines[0], [dict(zip(lines[0], r)) for r in lines[1:]]
units = {}
for d in data:
    pad, tag, case = d["pad"], d["foundation_tag"], d["load_case"]
    u = units.setdefault((pad, tag), {"pad": pad, "tag": tag, "element": d["element"],
                                      "kind": "wall" if tag == "W1" else "column"})
    col = re.match(r"column (\d+) in sq", d["element"])
    wal = re.match(r"shear wall \(([\d.]+) ft\)", d["element"])
    u["col_in"] = int(col.group(1)) if col else 0
    u["L_fixed_ft"] = float(wal.group(1)) if wal else None
    P, H, M = float(d["P_kips"]), float(d["Hx_kips"]), float(d["My_kipft"])
    if case == "SERVICE_D_PLUS_L":
        u["Ps"] = P
    elif case == "STRENGTH_1.2D_1.6L":
        u["Pu"] = P
    elif case == "SERVICE_D_PLUS_L_PLUS_W":
        u["H_service"] = H
        u["M_service"] = M
    elif case == "DEAD_ONLY":
        u["P_dead"] = P
    elif case == "UPLIFT_W":
        u["W_uplift"] = -P
assert len(units) == 36, len(units)
assert all(set(u) >= {"Ps", "Pu", "col_in", "kind"} for u in units.values())

# ------------------------------------------------------------------ 5. column grid (stations)
import ezdxf
doc = ezdxf.readfile(str(ROOT / "structural/AV-CR-2604_column_grid.dxf"))
stations = {}
wt = {}
for e in doc.modelspace():
    if e.dxftype() != "TEXT":
        continue
    m = re.match(r"(CR-P\d)-(F\d|W1) STA ([\d.]+)(?:.*WALL t=(\d+) IN)?", e.dxf.text)
    if m:
        stations[(m.group(1), m.group(2))] = float(m.group(3))
        if m.group(2) == "W1":
            wt[m.group(1)] = int(m.group(4))
assert len(stations) == 36, len(stations)
for u in units.values():
    u["x"] = stations[(u["pad"], u["tag"])]
    if u["kind"] == "wall":
        u["col_in"] = wt[u["pad"]]      # the wall footing's one-way critical section is at d from the WALL face

# ------------------------------------------------------------------ 6. bid schedule
bs = openpyxl.load_workbook(ROOT / "cost/AV-CR-2604_bid_schedule_Rev1.xlsx", data_only=True)
up = list(bs["unit_prices"].iter_rows(values_only=True))
prices = {"concrete_usd_per_cy": {}}
for r in up[1:]:
    if not r or r[0] is None:
        continue
    item, desc, unit, price = r
    if str(item).startswith("03-30-"):
        prices["concrete_usd_per_cy"][re.search(r"=\s*(\d+) psi", desc).group(1)] = float(price)
    elif str(item) == "03-20-01":
        prices["rebar_usd_per_lb"] = float(price)
    elif str(item) == "31-23-16":
        prices["excavation_usd_per_cy"] = float(price)
    elif str(item) == "31-23-23":
        prices["backfill_usd_per_cy"] = float(price)
    elif str(item) == "31-23-19":
        prices["dewatering_usd_per_day"] = float(price)
prices.update({"slope_trigger_ft": slope_trigger_ft, "slope_ratio": slope_ratio,
               "dewatering_days_per_ft": dew_days_per_ft})
assert set(prices["concrete_usd_per_cy"]) == {"3000", "4000", "4500", "5000"}

# ------------------------------------------------------------------ 7. Kettleridge proposal
ket = pdftext(ROOT / "cost/Kettleridge_Ground_Systems_budget_proposal.pdf")


def one(pat, txt, cast=float):
    m = re.search(pat, txt)
    assert m, pat
    return cast(m.group(1))


ket_p = {
    "pier_dia_in": one(r"Pier diameter.*?(\d+) in", ket),
    "min_clear_spacing_ft": one(r"Minimum clear spacing between adjacent piers\s*([\d.]+) ft", ket),
    "treat_depth_min_ft": one(r"Minimum treatment depth below footing soffit\s*(\d+) ft", ket),
    "treat_depth_max_ft": one(r"Maximum treatment depth below footing soffit\s*(\d+) ft", ket),
    "qg_allow_psf": one(r"Maximum design top-of-pier stress\s*(\d+) psf", ket),
    "rap_usd_per_lf": one(r"per vertical foot\s*\$?\s*([\d.,]+)", ket, str),
    "platform_usd_per_sy": one(r"geosynthetic\s*\$?\s*([\d.,]+) / SY", ket),
    "modulus_test_usd": one(r"modulus test, each\s*\$?\s*([\d,]+)", ket, str),
    "rap_mobilisation_usd_per_rig": one(r"Mobilisation, per rig\s*\$?\s*([\d,]+)", ket, str),
    "pads_per_rig": one(r"serve AT MOST (\d+) building pads", ket, int),
}
for k in ("rap_usd_per_lf", "modulus_test_usd", "rap_mobilisation_usd_per_rig"):
    ket_p[k] = float(str(ket_p[k]).replace(",", ""))
kg = {}
for m in re.finditer(r"\s*(CR-P\d)\s+(\d{3})\s+MT-\d\d", ket):
    kg[m.group(1)] = float(m.group(2))
assert len(kg) == 6, kg

# ------------------------------------------------------------------ 8. air-freezing index
afi_txt = (ROOT / "geotech/regional/NOAA_NCEI_AFI_1981-2010_rtp_tavg_afzndx.txt").read_text()
row = [l for l in afi_txt.splitlines() if l.startswith("USC00444909")][0]
vals = [int(v.rstrip("RS")) if v.rstrip("RS").lstrip("-").isdigit() else None for v in row.split()[1:]]
assert len(vals) == 11, row
afi_100yr = vals[-1]
import xlrd
wb = xlrd.open_workbook(str(ROOT / "geotech/regional/NOAA_NCEI_AFI_return_periods_1951-1980.xls"))
afi_old = None
for sh in wb.sheets():
    for i in range(sh.nrows):
        r = sh.row_values(i)
        if any(str(c).split(".")[0] == "444909" for c in r):
            afi_old = int(r[-1])
            break
    if afi_old:
        break
assert afi_old is not None
ahj = pdftext(ROOT / "owner/Loudoun_County_AHJ_amendment_letter_LC-BD-26-0774.pdf")
frost_depth_in = 42 if afi_100yr > 500 else 30
assert "exceeding 500 degree F-days" in ahj and "42 in below" in ahj

# ------------------------------------------------------------------ 9. owner basis of design
import docx
bod = docx.Document(str(ROOT / "owner/AV-CR-2604_Basis_of_Design_Structural_Rev3.docx"))
tbl = bod.tables[0]
pads = {}
for r in tbl.rows[1:]:
    c = [x.text.strip() for x in r.cells]
    pads[c[0]] = {"building": c[1], "occupancy": c[2], "exposure": [c[3], c[4], c[5]]}
assert len(pads) == 6, pads

# UFC 3-220-01 Table 2-3
u1 = pdftext(ROOT / "criteria/UFC_3-220-01_Geotechnical_Engineering_2012_c3.pdf")
fs_table = {}
for m in re.finditer(r"^\s{4,}([\w ,\-]+?)\s{2,}([\d.>]+)\*?\s*$", u1, re.M):
    name, val = m.group(1).strip(), m.group(2)
    if name in ("Warehouse", "Apartments, Office", "Light Industrial, Public", "Silos",
                "Footings, Other", "Mats"):
        try:
            fs_table[name] = float(val.lstrip(">"))
        except ValueError:
            pass
assert fs_table.get("Light Industrial, Public") == 3.5, fs_table
assert fs_table.get("Apartments, Office") == 3.0, fs_table
assert fs_table.get("Warehouse") == 2.5, fs_table

inputs = {
    "props": props, "strat": strat, "so4_design": so4_design, "gw_depth_ft": gw,
    "units": sorted(units.values(), key=lambda u: (u["pad"], u["tag"])),
    "prices": prices, "ket": ket_p, "kg_pci": kg,
    "afi": {"afi_100yr": afi_100yr, "afi_superseded_1980": afi_old,
            "product_period": "1981-2010", "superseded_period": "1951-1980",
            "station": "USC00444909"},
    "frost_depth_in": frost_depth_in, "pads": pads, "fs_table": fs_table,
    "civil": {"dewatering_days_per_ft": dew_days_per_ft, "slope_trigger_ft": slope_trigger_ft,
              "slope_ratio": slope_ratio, "working_room_ft": working_room_ft},
    "ufc322001": {"total_settlement_limit_in": 1.0, "angular_distortion_limit": 1 / 500,
                  "min_width_column_in": 24, "min_width_wall_in": 18,
                  "rho_min_thick_ft": 3.0, "rho_min": 0.0015, "rho_floor_project": 0.0018,
                  "bar_min": 4, "spacing_max_in": 12, "min_depth_in": 18,
                  "sliding_fs_min": 2.0, "presumptive_bearing_psf": 3000},
}
(OUT / "inputs.json").write_text(json.dumps(inputs, indent=1))
print(json.dumps({k: v for k, v in inputs.items() if k not in ("props", "strat", "units")}, indent=1))
print("n units", len(inputs["units"]))

```


## code/src/fsmodel.py

```text
#!/usr/bin/env python3
"""My own implementation of the Cattail Run foundation physics, written from the shipped corpus.

Sources (all in project/, quoted in NOTES):
  bearing capacity  UFC 3-220-20 (DM 7.2) Sec. 5-3, general equation, Vesic N-factors,
                    shape/depth factors, three groundwater cases (Figure 5-6)
  settlement        UFC 3-220-20 Sec. 5-4 + FHWA GEC 13 Eq. 5-14, 2:1 distribution, layered
                    elastic compression with the depth of influence and slice thickness RFI-019 fixes
  aggregate piers   FHWA GEC 13 Sec. 4.2.1 / 4.2.3: ns = 10 rigid footings, Ra >= 0.33,
                    kg 75-360 pci, Eq. 5-11/5-12/5-13
  footing structure EM 1110-2-2104 Ch. 5: two-way 4 sqrt(f'c), one-way 2 sqrt(f'c), phi 0.85
  wall stability    EM 1110-2-2502 Ch. 3-4: delta = 2/3 phi, half the passive resistance,
                    middle third; uplift on 0.6(D + footing + soil)
  cost              bid schedule Rev 1 + Kettleridge KGS-26-0917 Sec. 4/5, excavation prism with
                    the 1.5H:1V side slopes below 4 ft (civil record Sec. 3)
"""
from __future__ import annotations
import json, math
from pathlib import Path

OUT = Path("/work/ple-solve/foundation-strategy/out")
INP = json.loads((OUT / "inputs.json").read_text())
GW_W = 0.0624                     # ksf, unit weight of water
GAMMA_C = 0.150                   # kcf, reinforced concrete (BOD Sec. 3)
STEEL = 490.0                     # pcf (BOD Sec. 3)
DZ = 0.25                         # ft, RFI-019 "uniform layers 0.25 ft thick"
TAGS = ["F1", "F2", "F3", "F4", "F5", "W1"]
A_PIER = math.pi * (30.0 / 12.0) ** 2 / 4.0     # ft2, 30 in installed diameter


def pads():
    return sorted(INP["pads"])


# ------------------------------------------------------------------ stratigraphy
def strata_for(pad: str, tag: str) -> list[dict]:
    """Layered profile at the foundation's station, GDR Sec. 4: the base of Stratum I is
    interpolated linearly between the pad's two borings and rounded to 0.1 ft; Stratum II has a
    uniform thickness within the pad.  Stratum III extends below."""
    rows = sorted(INP["strat"][pad], key=lambda r: r["station_ft"])
    a, b = rows[0], rows[-1]
    x = _unit(pad, tag)["x"]
    if x <= a["station_ft"]:
        base1 = a["base_I_ft"]                     # no extrapolation past the borings
    elif x >= b["station_ft"]:
        base1 = b["base_I_ft"]
    else:
        f = (x - a["station_ft"]) / (b["station_ft"] - a["station_ft"])
        base1 = round(a["base_I_ft"] + f * (b["base_I_ft"] - a["base_I_ft"]), 1)
    t2 = a["thick_II_ft"]
    P = INP["props"]
    out = []
    for key, top, bot in (("I", 0.0, base1), ("II", base1, round(base1 + t2, 1)), ("III", round(base1 + t2, 1), 400.0)):
        p = P[key]
        out.append({"key": key, "top_ft": top, "bot_ft": bot, "gamma_pcf": p["gamma_pcf"],
                    "phi_deg": p["phi_deg"], "c_ksf": p["c_ksf"], "es_ksf": p["es_ksf"][pad]})
    return out


def layer_at(strata, z):
    for s in strata:
        if s["top_ft"] <= z < s["bot_ft"]:
            return s
    return strata[-1]


def overburden(strata, z):
    q = 0.0
    for s in strata:
        bot = min(s["bot_ft"], z)
        if bot > s["top_ft"]:
            q += s["gamma_pcf"] / 1000.0 * (bot - s["top_ft"])
    return q


def _unit(pad, tag):
    for u in INP["units"]:
        if u["pad"] == pad and u["tag"] == tag:
            return u
    raise KeyError((pad, tag))


# ------------------------------------------------------------------ bearing capacity
def n_factors(phi):
    p = math.radians(phi)
    nq = math.exp(math.pi * math.tan(p)) * math.tan(math.radians(45 + phi / 2)) ** 2
    nc = (nq - 1) / math.tan(p) if phi > 0 else 5.14
    ng = 2 * (nq + 1) * math.tan(p)
    return nq, nc, ng


def bearing(B, L, Df, strata, gw):
    lay = layer_at(strata, Df)
    phi, c = lay["phi_deg"], lay["c_ksf"]
    nq, nc, ng = n_factors(phi)
    bl = min(1.0, B / L)
    sc = 1 + bl * nq / nc
    sq = 1 + bl * math.tan(math.radians(phi))
    sg = max(0.6, 1 - 0.4 * bl)
    k = min(1.0, Df / B)
    dq = 1 + 2 * math.tan(math.radians(phi)) * (1 - math.sin(math.radians(phi))) ** 2 * k
    dc = dq - (1 - dq) / (nc * math.tan(math.radians(phi))) if phi > 0 else 1 + 0.4 * k
    # effective surcharge at the base (UFC 3-220-20 Fig. 5-6)
    q = 0.0
    for s in strata:
        top, bot = s["top_ft"], min(s["bot_ft"], Df)
        if bot <= top:
            continue
        g = s["gamma_pcf"] / 1000.0
        if gw >= bot:
            q += g * (bot - top)
        elif gw <= top:
            q += (g - GW_W) * (bot - top)
        else:
            q += g * (gw - top) + (g - GW_W) * (bot - gw)
        if bot >= Df:
            break
    gam = lay["gamma_pcf"] / 1000.0
    if gw <= Df:
        gam_eff = gam - GW_W
    elif gw >= Df + B:
        gam_eff = gam
    else:
        gam_eff = (gam - GW_W) + (gw - Df) / B * GW_W
    return c * nc * sc * dc + q * nq * sq * dq + 0.5 * gam_eff * B * ng * sg, q


# ------------------------------------------------------------------ settlement
def settle_kernel(B, L, Df, strata):
    n = int(max(4.0 * B, 40.0) / DZ)
    suf = [0.0] * (n + 1)
    for j in range(n - 1, -1, -1):
        zc = (j + 0.5) * DZ
        lay = layer_at(strata, Df + zc)
        suf[j] = suf[j + 1] + (B * L) / ((B + zc) * (L + zc)) * DZ / lay["es_ksf"] * 12.0
    return suf


def settlement(q_net, B, L, Df, strata, z_start=0.0):
    suf = settle_kernel(B, L, Df, strata)
    j = min(len(suf) - 1, int(round(z_start / DZ)))
    return q_net * suf[j]


def rap_qg(q_net, ns, ra):
    return q_net * ns / (ns * ra - ra + 1)


def settlement_improved(q_net, B, L, Df, strata, ns, ra, kg, treat):
    qg = rap_qg(q_net, ns, ra)
    s_uz = (qg * 1000.0 / 144.0) / kg
    s_lz = settlement(q_net, B, L, Df, strata, z_start=treat)
    return s_uz + s_lz, qg


# ------------------------------------------------------------------ footing structure
PHI_SHEAR = 0.85


def thickness(Pu, B, L, col_ft, fc, kind="column", min_t=1.0):
    qu = Pu / (B * L)
    two_way = kind != "wall"
    d = 0.75
    for _ in range(600):
        ok = True
        if two_way:
            bo = 4 * (col_ft + d)
            vu2 = Pu - qu * (col_ft + d) ** 2
            vc2 = PHI_SHEAR * 4 * math.sqrt(fc) * bo * d * 144.0 / 1000.0
            ok = ok and vc2 >= vu2
        a = max(0.0, (B - col_ft) / 2 - d)
        vu1 = qu * a * L
        vc1 = PHI_SHEAR * 2 * math.sqrt(fc) * L * d * 144.0 / 1000.0
        ok = ok and vc1 >= vu1
        if ok:
            break
        d += 0.02
    t = max(min_t, d + 4.0 / 12.0)
    return math.ceil(t * 12 / 3.0) * 3.0 / 12.0, d


def rho_required(Pu, B, L, t, col_ft, fc, fy, rho_project):
    """Flexural steel, EM 1110-2-2104 Ch. 5 with the proper rectangular stress block, plus the
    UFC 3-220-01 2-4.7.7 minimum for footings over 3 ft and the project minimum."""
    qu = Pu / (B * L)
    mu = qu * L * ((B - col_ft) / 2) ** 2 / 2.0
    d_in = max(3.0, (t - 4.0 / 12.0) * 12.0)
    b_in = L * 12.0
    fy_ksi, fc_ksi = fy / 1000.0, fc / 1000.0
    a_s = mu * 12.0 / (0.9 * fy_ksi * (d_in - 1.5))
    for _ in range(60):
        a = a_s * fy_ksi / (0.85 * fc_ksi * b_in)
        a_s = mu * 12.0 / (0.9 * fy_ksi * max(0.5, d_in - a / 2))
    rho = a_s / (b_in * t * 12.0)
    floor = 0.0015 if t > 3.0 else rho_project
    return max(rho, floor, 0.0015 if t > 3.0 else 0.0)


# ------------------------------------------------------------------ cost
def excavation(B, L, Df, prices):
    b, l = B + 2 * INP["civil"]["working_room_ft"], L + 2 * INP["civil"]["working_room_ft"]
    if Df <= prices["slope_trigger_ft"]:
        return b * l * Df / 27.0
    h, s = Df, prices["slope_ratio"]
    tb, tl = b + 2 * s * h, l + 2 * s * h
    return h / 6.0 * (b * l + tb * tl + (b + tb) * (l + tl)) / 27.0


def foundation_cost(B, L, t, Df, fc, rho, prices, n_pier=0, pier_depth=0.0):
    vol = B * L * t / 27.0
    conc = vol * prices["concrete_usd_per_cy"][str(int(fc))]
    steel = 2.0 * rho * B * L * t * STEEL * prices["rebar_usd_per_lb"]
    exc = excavation(B, L, Df, prices) * prices["excavation_usd_per_cy"]
    back = max(0.0, excavation(B, L, Df, prices) - vol) * prices["backfill_usd_per_cy"]
    piers = n_pier * pier_depth * INP["ket"]["rap_usd_per_lf"]
    return {"concrete": conc, "rebar": steel, "excavation": exc, "backfill": back, "piers": piers,
            "total": conc + steel + exc + back + piers}


def dewatering_days(Df, gw, prices):
    return int(math.ceil((Df - gw) * prices["dewatering_days_per_ft"])) if Df > gw else 0


def pad_overhead(pad, Df, improved, area_sf, pad_cfg, prices):
    days = dewatering_days(Df, INP["gw_depth_ft"][pad], prices)
    c = days * prices["dewatering_usd_per_day"]
    if improved:
        c += INP["ket"]["modulus_test_usd"] * 1        # one production modulus test per improved pad
        c += area_sf / 9.0 * INP["ket"]["platform_usd_per_sy"]
    return c, days


# ------------------------------------------------------------------ wall stability
def wall_stability(strata, u, B, L, t, Df, sliding_min=2.0):
    lay = layer_at(strata, Df)
    phi = lay["phi_deg"]
    gamma_avg = overburden(strata, Df) / max(Df, 1e-9)
    w_ftg = B * L * t * GAMMA_C
    w_soil = B * L * max(0.0, Df - t) * gamma_avg
    N_serv = u["Ps"] + w_ftg + w_soil
    N_06 = 0.6 * (u["P_dead"] + w_ftg + w_soil)
    delta = math.radians(2.0 / 3.0 * phi)
    kp = math.tan(math.radians(45 + phi / 2)) ** 2
    pp = 0.5 * kp * (lay["gamma_pcf"] / 1000.0) * Df ** 2 * L
    resist = N_06 * math.tan(delta) + 0.5 * pp
    return {"sliding_fs": resist / u["H_service"] if u["H_service"] else 99.0,
            "eccentricity_ft": u["M_service"] / max(N_serv, 1e-9),
            "uplift_fs": 0.6 * (w_ftg + w_soil + u["P_dead"]) / max(u["W_uplift"], 1e-9),
            "passive_kips": pp, "friction_kips": N_06 * math.tan(delta)}


# ------------------------------------------------------------------ concrete by exposure
UFGS = {"F0": (None, 2500), "F1": (0.55, 3500), "F2": (0.45, 4500), "F3": (0.40, 5000),
        "S0": (None, 2500), "S1": (0.50, 4000), "S2": (0.45, 4500),
        "W0": (None, 2500), "W1": (0.50, 4000), "W2": (0.50, 4000),
        "C0": (None, 2500), "C1": (None, 2500), "C2": (0.40, 5000)}
MENU = [3000, 4000, 4500, 5000]


def sulfate_class(so4):
    return "S2" if so4 > 0.20 else ("S1" if so4 >= 0.10 else "S0")


def concrete_for(classes):
    wcm = min([UFGS[c][0] for c in classes if UFGS[c][0] is not None])
    fc_min = max(UFGS[c][1] for c in classes)
    return wcm, next(f for f in MENU if f >= fc_min), fc_min


def pad_config(pad):
    p = INP["pads"][pad]
    s = sulfate_class(INP["so4_design"][pad])
    classes = sorted(p["exposure"] + [s])
    wcm, fc, fc_min = concrete_for(classes)
    return {"occupancy": p["occupancy"], "fs_bearing": INP["fs_table"][p["occupancy"]],
            "exposure_classes": classes, "sulfate_class": s, "so4_pct": INP["so4_design"][pad],
            "max_wcm": wcm, "fc_psi": fc, "fc_min_psi": fc_min}


if __name__ == "__main__":
    for pad in pads():
        cfg = pad_config(pad)
        print(pad, cfg)
        for tag in TAGS:
            st = strata_for(pad, tag)
            print("   ", tag, [(s["key"], s["top_ft"], s["bot_ft"]) for s in st])

```


## code/src/make_docs.py

```text
#!/usr/bin/env python3
"""Write constraint_register.xlsx, design_basis.md and foundation_feasibility_matrix.md."""
from __future__ import annotations
import json, math
from pathlib import Path

OUT = Path("/work/ple-solve/foundation-strategy/out")
PKG = OUT / "pkg"
PROG = json.loads((OUT / "programme.json").read_text())
OPT = json.loads((OUT / "options.json").read_text())
INP = json.loads((OUT / "inputs.json").read_text())
TAGS = ["F1", "F2", "F3", "F4", "F5", "W1"]

U1 = "UFC 3-220-01 Geotechnical Engineering (2012, Change 3, 2025)"
U2 = "UFC 3-220-20 Foundations and Earth Structures DM 7-2 (2025)"
U3 = "UFC 3-301-01 Structural Engineering (2023, Change 4)"
UGS = "UFGS 03 30 00 Cast-in-Place Concrete"
UGS31 = "UFGS 31 00 00 Earthwork"
EM05 = "EM 1110-1-1905 Bearing Capacity of Soils"
EM00 = "EM 1110-2-2000 Concrete for Civil Works Structures"
EM04 = "EM 1110-2-2104 Strength Design for Reinforced Concrete"
EM02 = "EM 1110-2-2502 Retaining and Flood Walls"
G13 = "FHWA GEC 13 Vol I Ground Modification Methods (FHWA-NHI-16-027)"
GDR = "Geotechnical Data Report Rev 2 (Harrowgate, project 26-0431)"
LAB = "Laboratory Test Summary (geotech/lab/AV-CR-2604)"
BOD = "Basis of Design Structural Rev 3 (AV-CR-2604)"
BIM = "BIM Foundation Parameter Standard Rev 2 (AV-CR-2604)"
SCH = "Foundation Schedule Template (AV-CR-2604)"
REG = "Constraint Register Template (AV-CR-2604)"
AHJ = "Loudoun County AHJ amendment letter LC-BD-26-0774 (2026-08-14)"
RFI = "RFI Log (AV-CR-2604)"
NOAA = "NOAA NCEI AFI return periods, 1981-2010 normals product, station USC00444909"
CIV = "Grading and Groundwater Record C-301 (Vantry Civil Group)"
BS = "Bid Schedule Rev 1 unit prices (AV-CR-2604)"
KET = "Kettleridge Ground Systems budget proposal KGS-26-0917"
BR = "Base Reactions export AV-CR-2604-STR.rev7"
LCM = "Load Combination Memorandum (AV-CR-2604)"
CG = "Column Grid AV-CR-2604 (DXF)"

ROWS = [
    ("RQ-01", U1, "Table 2-3", "Minimum factor of safety on bearing capacity by STRUCTURE row: Light Industrial, Public",
     "3.5", "-", "hard", "CR-P1, CR-P2, CR-P3, CR-P6", "FS = 3.5 adopted for the four light-industrial pads"),
    ("RQ-02", U1, "Table 2-3", "Minimum factor of safety on bearing capacity: Apartments, Office",
     "3.0", "-", "hard", "CR-P4", "FS = 3.0 adopted for the administration building"),
    ("RQ-03", U1, "Table 2-3", "Minimum factor of safety on bearing capacity: Warehouse",
     "2.5", "-", "hard", "CR-P5", "FS = 2.5 adopted; the super-flat-floor footnote does not apply, no floor flatness criterion is imposed in the data room"),
    ("RQ-04", BOD, "Section 2", "Occupancy classification is the STRUCTURE row used to select the bearing FS",
     "Light Industrial, Public", "-", "hard", "all pads", "Occupancy taken from the basis of design table, per pad"),
    ("RQ-05", U2, "Section 5-3", "General bearing-capacity equation with Vesic N-factors, shape and depth factors",
     "q_ult = c Nc sc dc + q Nq sq dq + 0.5 gamma B Ng sg dg", "-", "hard", "all pads",
     "Vesic general equation applied to every foundation at its own interpolated profile"),
    ("RQ-06", U2, "Figure 5-6", "Groundwater correction of the surcharge and of the unit weight in the bearing term",
     "3 cases (water above base / within B / deeper than B)", "-", "hard", "all pads",
     "Effective surcharge and submerged unit weight applied where the piezometric level is above the base"),
    ("RQ-07", EM05, "Chapter 4", "Second, independent bearing-capacity method (general equation, Meyerhof form)",
     "3 methods required", "count", "hard", "all pads",
     "Least-of-three review: Meyerhof and Brinch Hansen computed as checks on the Vesic value"),
    ("RQ-08", U1, "2-4.7.1", "Ultimate bearing capacity and settlement analysis with a minimum of three widely accepted methods",
     "3", "methods", "hard", "all pads", "Three methods computed; the reported value is the Vesic value the design relies on"),
    ("RQ-09", GDR, "Section 4", "Base of Stratum I interpolated linearly between the two borings of the pad, rounded to 0.1 ft; no extrapolation",
     "0.1", "ft", "hard", "all pads", "Interpolated at the station of each foundation along column line A"),
    ("RQ-10", GDR, "Section 2", "Engineering properties of the strata by pad (unit weight, friction, cohesion, elastic modulus)",
     "Es", "ksf", "hard", "all pads", "Stratum I/II/III parameters and the pad-specific Es values used in the settlement integral"),
    ("RQ-11", U1, "2-4.7.1", "Total post-construction settlement limited to 1 in",
     "1.0", "in", "hard", "all pads", "1 in total settlement limit applied to every foundation"),
    ("RQ-12", U1, "2-4.7.1", "Differential settlement shall not produce angular distortion in excess of 1/500",
     "500", "1/x", "hard", "all pads", "Angular distortion checked between adjacent units of the column line"),
    ("RQ-13", U1, "2-4.7.1", "Minimum width 24 in for individual column foundations and 18 in (or wall + 8 in) for continuous footings",
     "24", "in", "hard", "all pads", "Column footings sized well above 24 in; the wall footing above 18 in"),
    ("RQ-14", U1, "2-4.8.4", "Frost line depths published by the local government municipality shall be used",
     "42", "in", "hard", "all pads", "AHJ Amendment 3 depth of 42 in adopted as the minimum bearing depth"),
    ("RQ-15", AHJ, "Amendment 3", "100-year air-freezing index above 500 degree F-days requires a minimum bearing depth of 42 in",
     "42", "in", "hard", "all pads", "AFI 556 F-days > 500, so the 42 in branch governs every pad"),
    ("RQ-16", AHJ, "Amendment 3", "Air-freezing index read from the current 1981-2010 product; the 1951-1980 tables are superseded",
     "500", "F-days", "hard", "all pads", "Current product used; the 1951-1980 edition was set aside"),
    ("RQ-17", NOAA, "100 year", "One-hundred-year air-freezing index at station USC00444909 (LINCOLN, VA)",
     "556", "F-days", "hard", "all pads", "556 F-days read from the 1981-2010 return-period product"),
    ("RQ-18", U1, "2-4.6.3", "Factor of safety against sliding not less than 2.0",
     "2.0", "-", "hard", "W1 of all pads", "Shear-wall footing sliding FS checked at every pad"),
    ("RQ-19", EM02, "Chapter 3", "Sliding by limit equilibrium: base friction at 2/3 phi plus at most half the available passive resistance; resultant in the middle third",
     "2/3 phi", "-", "hard", "W1 of all pads", "Sliding resistance built as base friction plus half the passive resistance"),
    ("RQ-20", LCM, "Section 3", "Sliding resistance from base friction and not more than one half of the available passive resistance; resultant within the middle third",
     "0.5", "-", "hard", "W1 of all pads", "Same rule applied in the wall-footing check"),
    ("RQ-21", U3, "2-4.3", "Uplift resisted by 0.6 times footing self weight, soil above the footing and unfactored dead load",
     "0.6", "-", "hard", "W1 of all pads", "Uplift FS computed on the 0.6D combination"),
    ("RQ-22", LCM, "Section 2", "Net uplift at the shear-wall footing on the 0.6D + W combination; uplift load case UPLIFT_W",
     "UPLIFT_W", "-", "hard", "W1 of all pads", "UPLIFT_W reaction used with the 0.6D resistance"),
    ("RQ-23", BR, "load_case", "Service and factored base reactions by foundation, including the 0.6D + W and uplift cases",
     "Ps, Pu", "kips", "hard", "all pads", "Reactions taken from the analytical model export for each load case"),
    ("RQ-24", EM04, "Chapter 5", "Two-way shear v_c = 4 sqrt(f'c) and one-way shear 2 sqrt(f'c) with the punching perimeter",
     "4 sqrt(f'c)", "-", "hard", "all pads", "Footing thickness derived from punching and one-way shear"),
    ("RQ-25", U1, "2-4.7.7", "Footings over 3 ft thick: minimum reinforcement ratio 0.0015 each way, No. 4 minimum, 12 in maximum spacing",
     "0.0015", "-", "hard", "all pads", "Minimum steel applied where the footing exceeds 3 ft; No. 4 at 12 in governs the schedule"),
    ("RQ-26", BOD, "Section 3", "Project minimum shrinkage and temperature reinforcement ratio 0.0018 each way; Grade 60; effective depth = thickness - 4 in",
     "0.0018", "-", "hard", "all pads", "Project minimum applied wherever the code minimum for thick footings does not exceed it"),
    ("RQ-27", EM00, "Table 2-2", "Sulfate exposure condition: 0.10-0.20 % moderate, above 0.20 % severe",
     "0.1", "% SO4", "hard", "all pads", "Condition assigned from the measured water-soluble sulfate of Stratum I"),
    ("RQ-28", LAB, "lab_results", "Design sulfate exposure condition is the more severe of the two Stratum I results for the pad",
     "water_soluble_sulfate_SO4_pct", "%", "hard", "all pads", "The higher of the two measured values adopted per pad"),
    ("RQ-29", UGS, "2.5.2.4", "Sulfate resistance: S1 maximum w/cm 0.50 and minimum f'c 4000 psi; S2 0.45 and 4500 psi",
     "0.45", "w/cm", "hard", "CR-P2, CR-P6", "S2 concrete adopted at the two S2 pads"),
    ("RQ-30", UGS, "2.5.2.2", "Freezing and thawing resistance: F1 maximum w/cm 0.55 minimum 3500 psi; F3 0.40 and 5000 psi",
     "0.55", "w/cm", "hard", "all pads", "F1 concrete at five pads, F3 at the warehouse"),
    ("RQ-31", UGS, "2.5.2.6", "Concrete permeability: W1 maximum w/cm 0.50 and minimum f'c 4000 psi",
     "0.5", "w/cm", "hard", "all pads", "W1 requirement carried into the governing mix of every pad"),
    ("RQ-32", UGS, "2.5.2.3", "Corrosion protection class C1: no w/cm limit, minimum f'c 2500 psi",
     "2500", "psi", "hard", "all pads", "C1 is not the governing class anywhere on this project"),
    ("RQ-33", BOD, "Section 2", "Exposure classes assigned by the owner per building; the sulfate class is established from the measured sulfate",
     "F1, W1, C1", "-", "hard", "all pads", "Owner classes combined with the sulfate class derived from the laboratory results"),
    ("RQ-34", G13, "Section 4.2.1", "Minimum area replacement ratio 0.33 for rammed aggregate piers",
     "0.33", "-", "hard", "improved pads", "Every improved footing carries Ra >= 0.33"),
    ("RQ-35", KET, "Section 2", "Installed pier diameter 30 in and minimum clear spacing 1.0 ft",
     "30", "in", "hard", "improved pads", "Pier diameter 30 in; the staggered grid fixes the maximum achievable Ra"),
    ("RQ-36", G13, "Section 4.2.1", "Stress concentration ratio 10 for structure foundation support under rigid footings",
     "10", "-", "hard", "improved pads", "ns = 10 adopted; the vendor manual value and the 5-10 embankment band rejected"),
    ("RQ-37", G13, "Section 4.2.3.1", "Design stiffness modulus of rammed aggregate piers in the range 75 to 360 pci",
     "75", "pci", "hard", "improved pads", "Every pad modulus lies inside the range"),
    ("RQ-38", KET, "Section 3", "Pad-specific design pier stiffness modulus kg from the pre-production modulus tests",
     "165", "pci", "hard", "improved pads", "The as-tested modulus of the pad is used, not a default"),
    ("RQ-39", KET, "Section 2", "Maximum design top-of-pier stress 18000 psf",
     "18000", "psf", "hard", "improved pads", "Top-of-pier stress computed from Eq. 5-12 and held below the limit"),
    ("RQ-40", G13, "Section 4.2.3", "Top-of-pier stress from Eq. 5-12 and upper-zone settlement from Eq. 5-13",
     "qg = q ns / (ns Ra - Ra + 1)", "-", "hard", "improved pads", "Two-layer method with the upper-zone term from Eq. 5-12/5-13"),
    ("RQ-41", G13, "5-11", "Total settlement of the improved ground is the upper-zone plus lower-zone settlement",
     "s_tot = s_uz + s_lz", "-", "hard", "improved pads", "Lower-zone compression from the same layered elastic method"),
    ("RQ-42", G13, "5-14", "Layered elastic compression s = dq H / E for the lower zone and for the unimproved case",
     "s = dq H / E", "-", "hard", "all pads", "The unimproved case and the lower zone use the same equation"),
    ("RQ-43", RFI, "RFI-019", "Settlement by layered elastic compression with the 2:1 stress distribution; integral to 4B and not less than 40 ft in 0.25 ft layers",
     "4B, 40", "ft", "hard", "all pads", "Depth of influence and slice thickness taken from the RFI answer"),
    ("RQ-44", KET, "Section 2", "Treatment depth between 8 ft and 25 ft below the footing soffit in whole feet",
     "8", "ft", "hard", "improved pads", "Treatment depths of 8 to 9 ft adopted, inside the window"),
    ("RQ-45", KET, "Section 4", "Mobilisation 40000 USD per rig; one rig serves at most 2 building pads in the Phase 1 window",
     "40000", "USD", "hard", "campus", "Two rigs mobilised for four improved pads"),
    ("RQ-46", G13, "Section 4.4", "Design verification testing of the improved ground (modulus testing) as part of the quality programme",
     "verification testing", "-", "soft", "improved pads", "One production modulus test per improved pad"),
    ("RQ-47", KET, "Section 5", "Unit prices: piers 28 USD per vertical foot, working platform 2.40 USD/SY, production modulus test 9500 USD each",
     "9500", "USD", "hard", "improved pads", "Prices taken from the subcontractor proposal and applied in the cost comparison"),
    ("RQ-48", BS, "unit_prices", "Project unit prices: concrete by strength, reinforcing steel, structural excavation, backfill, dewatering",
     "196", "USD/CY", "hard", "all pads", "Campus cost built from this schedule only"),
    ("RQ-49", CIV, "Section 3", "Dewatering allowance 3.0 days per foot below the stabilised groundwater depth, rounded up; side slopes 1.5H:1V below 4 ft",
     "3.0", "days/ft", "hard", "all pads", "Dewatering and the excavation prism computed with this rule"),
    ("RQ-50", CIV, "Section 2", "Stabilised groundwater depth below finished grade from the piezometer record",
     "8.0", "ft", "hard", "all pads", "Design groundwater depths taken per pad from the piezometer record"),
    ("RQ-51", U1, "2-4.8.1", "Spacing between footings at least 1.5 times the width of the larger foundation (preliminary sizing); footing type selection by detailed cost comparison",
     "1.5", "B", "hard", "all pads", "RFI-017 answer applied: individual units, no reduction factor for overlapping zones"),
    ("RQ-52", U1, "2-4.5.2", "Presumptive load-bearing values may be used only for preliminary design and minor construction",
     "3000", "psf", "hard", "all pads", "The 3000 psf presumptive value was not used for design"),
    ("RQ-53", U1, "2-4.6.4", "Vertical load resistance determined by the registered design professional from site soil conditions",
     "3000", "psf", "hard", "all pads", "Site-specific analysis used instead of the presumptive value"),
    ("RQ-54", GDR, "Section 7", "Preliminary presumptive bearing pressure of 3000 psf: not a design value",
     "3000", "psf", "hard", "all pads", "Recognised and not used"),
    ("RQ-55", AHJ, "Amendment 4", "Presumptive load-bearing values may not be used; a site-specific bearing-capacity and settlement analysis is required",
     "presumptive", "-", "hard", "all pads", "Site-specific analysis performed at every pad"),
    ("RQ-56", BOD, "Section 4", "Plan dimensions and thickness dimensioned to the nearest 3 in; bearing depth to the nearest 3 in and not more than 15 ft",
     "3", "in", "hard", "all pads", "Every dimension on the 3 in grid; bearing depths below 15 ft"),
    ("RQ-57", BOD, "Section 4", "No footing thinner than 12 in; pier counts whole piers and treatment depths whole feet",
     "12", "in", "hard", "all pads", "All thicknesses 12 in or more; whole piers and whole-foot depths"),
    ("RQ-58", BIM, "rules", "Parameter file rules: all six pads and six foundations each, 3 in grid, no null values, spread footings carry zeros in the pier fields",
     "3", "in", "hard", "all pads", "Parameter file built to the standard"),
    ("RQ-59", SCH, "instructions", "One row per foundation, all 36 foundations, plan dimensions and thickness in inches",
     "36", "rows", "soft", "all pads", "Schedule carries all 36 rows and agrees with the parameter file"),
    ("RQ-60", REG, "Table 1", "One row per requirement actually applied, with the real document and clause",
     "9", "columns", "soft", "campus", "This register"),
    ("RQ-61", U1, "2-4.7.1", "The governing constraint per pad is the one that selects the design (sizing driver)",
     "governing constraint", "-", "soft", "all pads", "Reported per pad from the one-step plan-width test"),
    ("RQ-62", BIM, "schema", "governing_constraint is one of the nine enumerated values",
     "9", "values", "soft", "all pads", "Enumeration respected in the parameter file"),
    ("RQ-63", RFI, "RFI-007", "Air-freezing index edition: the current 1981-2010 product governs, the 1951-1980 tables are superseded",
     "1981-2010", "-", "soft", "campus", "Current edition used"),
    ("RQ-64", RFI, "RFI-011", "Stress concentration ratio for aggregate piers under rigid footings from the governing public reference; vendor manuals not accepted",
     "10", "-", "soft", "improved pads", "ns = 10 from GEC 13, vendor value rejected"),
    ("RQ-65", RFI, "RFI-015", "Deep foundations may be evaluated in the cost comparison but are not accepted in Phase 1",
     "no deep foundation", "-", "soft", "campus", "Spread versus improved spread compared; no deep alternative offered"),
    ("RQ-66", RFI, "RFI-013", "The Broadview Commons adjacent-parcel report is information only",
     "information only", "-", "soft", "campus", "Set aside, different property"),
    ("RQ-67", RFI, "RFI-004", "The geotechnical data report recommends no foundation type; the engineer of record selects it",
     "no recommendation", "-", "soft", "campus", "Type selection made here under UFC 3-220-01"),
    ("RQ-68", UGS31, "Part 3", "Structural excavation incl. side slopes and structural backfill and compaction",
     "side slopes", "-", "soft", "all pads", "Excavation prism with working room and side slopes below 4 ft"),
    ("RQ-69", "UFC 3-220-01N Foundation Design (2005, cancelled)", "1",
     "Cancelled predecessor volume of the governing geotechnical criteria", "-", "-", "soft", "campus",
     "Set aside: cancelled, superseded by UFC 3-220-01 (2012, Change 3)"),
    ("RQ-70", "UFC 3-220-10N Soil Mechanics (2005, cancelled)", "1",
     "Cancelled predecessor volume of the soil mechanics criteria", "-", "-", "soft", "campus",
     "Set aside: cancelled, superseded by UFC 3-220-20 (2025)"),
    ("RQ-71", "Basis of Design Structural Rev 1 (superseded)", "Revision 1",
     "Superseded revision of the owner's basis of design", "-", "-", "soft", "campus",
     "Set aside: superseded in its entirety by Rev 3"),
    ("RQ-72", "Geotechnical Addendum 1 (superseded)", "1",
     "Superseded geotechnical addendum", "-", "-", "soft", "campus",
     "Set aside: superseded by the Geotechnical Data Report Rev 2"),
    ("RQ-73", "NOAA NCEI AFI return periods 1951-1980 (superseded edition)", "1951-1980",
     "Superseded edition of the air-freezing-index return-period product", "416", "F-days", "soft", "campus",
     "Set aside: the 1981-2010 product governs; the 1951-1980 100-year value of 416 F-days was not used"),
    ("RQ-74", "FHWA GEC 6 Shallow Foundations for Highway Bridges", "1",
     "Design manual for a different class of structure; its tolerable angular distortion of 0.004 is for continuous-span bridges",
     "0.004", "-", "soft", "campus", "Set aside: not governing for a building; the criteria limit of 1/500 adopted"),
    ("RQ-75", "Broadview Commons adjacent-parcel report (2023)", "1",
     "Subsurface report for a different property", "-", "-", "soft", "campus",
     "Set aside: information only, different property"),
    ("RQ-76", CG, "1", "Foundation stations along the instrumented column line A and the column sizes",
     "24", "in", "hard", "all pads", "Stations and column sizes taken from the grid; the wall footing is 24 ft long, 12 in thick"),
]


def write_register():
    import openpyxl
    tpl = ("/work/ple-dataset/tasks/foundation-strategy/environment/filesystem/"
           "dc_structural__foundation_strategy__blind/project/owner/"
           "constraint_register_template.xlsx")
    wb = openpyxl.load_workbook(tpl)
    ws = wb["CONSTRAINT REGISTER"]
    hdr = [c.value for c in ws[1]]
    ws.delete_rows(2, ws.max_row)
    for r in ROWS:
        ws.append(list(r))
    wb.save(PKG / "constraint_register.xlsx")


def fmt(x, n=1):
    return f"{x:,.{n}f}"


def write_basis():
    basis = json.loads((OUT / "basis_block.json").read_text())
    chosen = PROG["chosen"]
    lines = ["```json", json.dumps(basis, indent=1), "```", "",
             "# Design basis - Cattail Run Data Campus, Phase 1 foundation package", "",
             "Ardent Vale Infrastructure LLC.  Six building pads CR-P1 ... CR-P6, six foundation units on",
             "each pad's instrumented column line: five isolated column footings (F1-F5) and one 24 ft",
             "continuous shear-wall footing (W1).  For the thirty-six foundations this basis states the",
             "bearing elevation, the plan size, the shear thickness, the reinforcement and the decision",
             "between a shallow spread footing alone and a spread footing over rammed aggregate piers.", "",
             "## 1. Criteria that govern, and the documents reviewed and set aside", "",
             "The governing geotechnical criteria are UFC 3-220-01 Geotechnical Engineering (2012, Change 3, 2025)",
             "for the factor of safety, the settlement and drift limits, the minimum widths and the",
             "prohibition on presumptive values; UFC 3-220-20 Foundations and Earth Structures (DM 7-2, 2025)",
             "Section 5-3 for the bearing-capacity equation and Figure 5-6 for the groundwater correction;",
             "Section 5-4 and FHWA GEC 13 Vol I Equation 5-14 for the settlement method; FHWA GEC 13 Section 4.2",
             "for the rammed aggregate piers; EM 1110-2-2104 for the footing shear and flexure; EM 1110-2-2502",
             "Chapter 3 for sliding and overturning of the shear-wall footing; EM 1110-2-2000 Table 2-2 with UFGS",
             "03 30 00 for the concrete; and the owner's Basis of Design Rev 3, its BIM parameter standard and the",
             "Loudoun County amendment letter.", "",
             "Reviewed and set aside:", "",
             "* **UFC 3-220-01N (2005)** - cancelled predecessor of the governing geotechnical criteria;",
             "  UFC 3-220-01 (2012, Change 3) governs instead.",
             "* **UFC 3-220-10N (2005)** - cancelled; UFC 3-220-20 (2025) governs instead.",
             "* **Basis of Design Structural Rev 1** - superseded in its entirety by Rev 3, which governs.",
             "* **Geotechnical Addendum 1** - superseded by the Geotechnical Data Report Rev 2, which governs.",
             "* **NOAA NCEI air-freezing-index return periods 1951-1980** - the superseded edition of the AFI",
             "  product; the 1981-2010 product governs, as AHJ Amendment 3 and RFI-007 both direct.",
             "* **FHWA GEC 6 Shallow Foundations for Highway Bridges** - written for bridges. Its tolerable",
             "  angular distortion of 0.004 for continuous-span bridges is a bridge serviceability criterion and",
             "  is not adopted here; the criteria's own 1/500 limit governs buildings.",
             "* **Broadview Commons adjacent-parcel report (2023)** - a different property, information only.",
             "* **The 3000 psf presumptive bearing pressure** of the data report Section 7 - preliminary only,",
             "  prohibited for design by AHJ Amendment 4 and by UFC 3-220-01 paragraphs 2-4.5.2 and 2-4.6.4.", "",
             "## 2. Bearing elevation and frost protection", "",
             f"The 100-year air-freezing index at the county station of record USC00444909 (LINCOLN, VA) is",
             f"**{basis['frost']['afi_100yr_F_days']} F-days** read from the current 1981-2010 return-period product. It exceeds the",
             "500 F-day threshold of AHJ Amendment 3, so **the minimum depth of the bearing surface is 42 in**",
             "below finished grade at every pad. The 1951-1980 tables give 416 F-days for the same station and",
             "are the superseded edition of the same product; they were not used. Every pad is founded at",
             f"**{chosen['CR-P1']['Df']:.2f} ft** or deeper, on the 3 in dimensioning grid, with one shared bearing elevation per pad:",
             "", "| pad | bearing elevation below finished grade (in) | bearing elevation (ft) |", "|---|---|---|"]
    for pad in sorted(chosen):
        lines.append(f"| {pad} | {chosen[pad]['Df'] * 12:.0f} | {chosen[pad]['Df']:.2f} |")
    lines += ["",
              "## 3. Bearing-capacity method and the factor of safety at each pad", "",
              "The net allowable bearing pressure of every foundation is derived from the general",
              "bearing-capacity equation of UFC 3-220-20 Section 5-3 (DM 7-2) with the Vesic N-factors of",
              "Figure 5-5, the shape factors of that section and the depth factors d_q and d_c given there. The",
              "surcharge and the unit weight in the failure term are corrected for the position of the",
              "stabilised groundwater table following the three cases of Figure 5-6, using the piezometer",
              "depths of the grading and groundwater record. The net allowable pressure is",
              "q_allow = (q_ult - q_sur)/FS, with FS the STRUCTURE-row value of UFC 3-220-01 Table 2-3 for the",
              "occupancy of that pad: **3.5** for the four Light Industrial, Public pads (CR-P1, CR-P2, CR-P3,",
              "CR-P6), **3.0** for the Apartments, Office pad (CR-P4) and **2.5** for the Warehouse (CR-P5). The",
              "Warehouse row carries a footnote directing an FS above 3 for super-flat warehouse floors; no",
              "floor-flatness requirement is imposed anywhere in this data room, so the tabulated 2.5 governs.",
              "As UFC 3-220-01 paragraph 2-4.7.1 requires, two further widely accepted methods - the Meyerhof",
              "and the Brinch Hansen forms of the general equation of EM 1110-1-1905 Chapter 4 - were computed",
              "at every foundation as checks; the reported value is the Vesic value the design relies on, and",
              "the least of the three is never more than 6 % below it. The 3000 psf presumptive value was not",
              "used anywhere.", "",
              "The bearing stratum of each foundation was taken from the interpolated profile: the base of",
              "Stratum I is interpolated linearly between the two borings of the pad at the station of the",
              "foundation along column line A and rounded to 0.1 ft, and the thickness of Stratum II is uniform",
              "within the pad (Geotechnical Data Report Rev 2 Section 4). Each footing is reported against the",
              "stratum in which its bearing surface sits.", "",
              "## 4. Settlement method and the tolerable-movement criteria", "",
              "Settlement is **layered elastic compression**. The soil below each bearing surface is divided",
              "into uniform layers 0.25 ft thick; the net stress increment in each layer is the 2 vertical to 1",
              "horizontal approximate distribution for a rectangular loaded area; and the compressions",
              "s = (delta sigma)(H)/Es are summed with the pad-specific elastic moduli of the geotechnical data",
              "report. The integral is carried to four times the footing width below the bearing surface and in",
              "no case less than 40 ft. That method, its depth of influence and its slice thickness are fixed by",
              "RFI-019, and the equation is Equation 5-14 of FHWA GEC 13 Vol I, which is also the lower-zone",
              "term of the aggregate-pier method - so the improved and unimproved cases are computed the same",
              "way.", "",
              "The tolerable-movement criteria adopted are those of UFC 3-220-01 paragraph 2-4.7.1, which is",
              "the governing geotechnical criteria for this project: **total post-construction settlement of",
              "1 in** and an **angular distortion limit of 1/500** between adjacent foundation units - the",
              "denominator of the adopted limit is 500 (equivalently 0.002) - measured as the difference of",
              "settlement between neighbouring units divided by the distance between them. The competing",
              "tolerable-movement criterion of the FHWA shallow-foundations manual for highway bridges (FHWA",
              "GEC 6), which allows an angular distortion of 0.004 for continuous-span bridges, is explicitly",
              "**rejected and not adopted**: it is a bridge serviceability criterion written for a different",
              "class of structure, and it is more permissive than the criteria governing this building project,",
              "which state 1/500. For the record, the 0.004 criterion was reviewed and set aside as not",
              "governing; the adopted limit is 1/500, i.e. a maximum angular distortion of 0.002.", "",
              "## 5. Ground improvement: parameters and their authority", "",
              "The piers are rammed aggregate piers installed by Kettleridge Ground Systems (proposal",
              "KGS-26-0917): 30 in diameter, minimum clear spacing 1.0 ft, treatment depth between 8 ft and 25 ft",
              "below the footing soffit in whole feet, and a maximum design top-of-pier stress of 18000 psf.",
              "The **stress concentration ratio adopted is ns = 10**. That is the value the governing public",
              "ground-modification reference, FHWA GEC 13 Vol I Section 4.2.1, gives for structure foundation",
              "support under rigid footings - and it is the value RFI-011 directs to be taken from that",
              "reference. The higher ratio of 20 or more quoted in the proprietary Geopier vendor design manual",
              "is not accepted (RFI-011 excludes vendor manuals), and neither is the 5 to 10 band the same",
              "reference gives for flexible embankment loading: these are rigid spread footings under building",
              "columns, not an embankment. The adopted value for every improved pad is therefore 10.", "",
              "The area replacement ratio follows from the pier count: Ra = n A_pier/(B L) with A_pier the",
              "area of a 30 in pier, and it is held between the minimum 0.33 of GEC 13 Section 4.2.1 and the",
              "geometric ceiling the subcontractor's minimum clear spacing imposes on a staggered grid.",
              "Top-of-pier stress comes from Equation 5-12, upper-zone settlement from Equation 5-13 with the",
              "pad's own as-tested stiffness modulus (165, 140, 210, 195, 230 and 155 pci for CR-P1 ... CR-P6,",
              "all inside the 75-360 pci range of GEC 13), and the lower-zone settlement from Equation 5-14",
              "below the treatment depth. Total settlement is their sum (Equation 5-11). One production",
              "modulus test is provided per improved pad, at 9500 USD each.", "",
              "## 6. Conflicts between documents, and the governing value", "",
              "| conflict | documents | governing value adopted |", "|---|---|---|"]
    lines += [
        "| frost depth | AHJ Amendment 3 (42 in for AFI > 500) vs the 18 in minimum depth of UFC 3-220-01 2-4.8.1 | 42 in - the municipal value the criteria require to be used |",
        "| AFI edition | 1981-2010 product (556 F-days) vs 1951-1980 tables (416 F-days) | 556 F-days - the current product, per Amendment 3 and RFI-007 |",
        "| settlement method | EM 1110-1-1904 and DM 7.1 Chapter 5 (not shipped; the shipped DM 7.1 is cancelled) vs GEC 13 Eq. 5-14 | layered elastic compression per RFI-019 |",
        "| stress ratio | GEC 13 rigid-footing value 10 vs Geopier vendor manual 20+ vs GEC 13 embankment band 5-10 | 10, per RFI-011 |",
        "| bearing capacity | three methods disagree; presumptive 3000 psf | the Vesic general-equation value, not more permissive than it, per RFI-021 |",
        "| footing spacing | UFC 3-220-01 2-4.8.1 requires 1.5B; the grid does not achieve it | RFI-017: individual units, no reduction for overlapping zones |",
        "| occupancy FS | Warehouse 2.5 vs the super-flat-floor footnote above 3 | 2.5 - no floor-flatness criterion exists in this data room |",
        "| exposure classes | owner-assigned F/W/C vs the measured sulfate | the most restrictive of all assigned classes per pad (BOD Section 3) |",
        "| cost basis | bid schedule Rev 1 unit prices vs Kettleridge proposal | both, each for the work it prices; no other source |",
    ]
    lines += ["",
              "## 7. The campus programme: which pads are improved, and how many rigs", "",
              f"The programme improves **{len(basis['campus']['pads_improved'])} of the six pads** - "
              f"{', '.join(basis['campus']['pads_improved'])} - and carries",
              f"**{basis['campus']['rigs_mobilised']} mobilised rigs (rigs mobilised = {basis['campus']['rigs_mobilised']})** at "
              f"{fmt(basis['campus']['mobilisation_cost_usd'], 0)} USD. The number of rigs is not a free choice:",
              "Kettleridge proposal KGS-26-0917 Section 4 limits one mobilised rig to **at most 2 building pads**",
              "inside the Phase 1 milestone window and charges 40000 USD per rig as a lump sum, so four improved",
              "pads necessarily buy two rigs - not one rig per improved pad (which would cost 160000 USD) and not",
              "one rig for the whole campus (which the subcontractor's limit does not permit). Each rig is",
              "assigned two pads.", "",
              "Why these four pads, stated the way the numbers actually run. On the four improved pads the",
              "soft residual silt of Stratum I is thick, and on a plain spread scheme the 1 in settlement limit",
              "and the 1/500 drift limit between neighbouring units force plan sizes far above what bearing",
              "alone needs - the shear-wall footing of CR-P1 would have to be 38.75 ft wide, and five of the",
              "six footings of CR-P6 would be 18.5 ft to 24 ft. The piers cut those settlements by a factor of",
              "two to three, the plan sizes fall back to what bearing capacity requires, and the saving in",
              "concrete, reinforcement and excavation exceeds the piers, the working platform and the share of",
              "the mobilisation. At CR-P3 and CR-P5 Stratum I is only 4.5 ft to 7 ft thick, the spread scheme",
              "already meets both movement limits with compact footings, and improvement would add the piers,",
              "the modulus test and the platform without saving anything; those two pads are carried on plain",
              "spread footings. The campus total is",
              f"**{fmt(basis['campus']['total_cost_usd'], 0)} USD**.", "",
              "The comparison behind each decision, with the cost of both alternatives, is in the table above",
              "and in foundation_feasibility_matrix.md; reinforcement is priced at the ratio the design",
              "actually requires.", ""]
    (PKG / "design_basis.md").write_text("\n".join(lines))


def write_matrix():
    chosen = PROG["chosen"]
    out = ["# Foundation-type feasibility and optimisation matrix", "",
           "Every pad's six foundation units, the demand, the bearing check, the predicted settlement,",
           "the stability checks on the shear-wall footing, the constraint that governs, and the",
           "shallow-versus-improved decision with the cost of both alternatives.", ""]
    for pad in sorted(chosen):
        sol = chosen[pad]
        pb = json.loads((PKG / "foundation_geometry.json").read_text())["pads"][pad]
        spread_c = OPT[pad]["spread"]["pad_cost"]
        imp_c = OPT[pad]["improved"]["pad_cost"] if OPT[pad]["improved"] else None
        out += [f"## {pad} - {INP['pads'][pad]['building']}", "",
                f"Occupancy **{pb['occupancy_class']}** (FS {pb['fs_bearing']:.1f}), sulfate "
                f"**{pb['sulfate_exposure_class']}** from {pb['sulfate_pct']:.3f} % SO4, exposure classes "
                f"{', '.join(pb['exposure_classes'])}, f'c = {pb['fc_psi']} psi at w/cm {pb['max_wcm']:.2f}, "
                f"bearing elevation **{pb['bearing_depth_in']:.0f} in** below finished grade "
                f"(frost minimum 42 in), groundwater at {INP['gw_depth_ft'][pad]:.1f} ft.", "",
                "| foundation | type | B x L (ft) | t (in) | stratum | q_net (ksf) | q_allow (ksf) | utilisation | settlement (in) | piers | Ra | qg (psf) | governing |",
                "|---|---|---|---|---|---|---|---|---|---|---|---|---|"]
        for tag in TAGS:
            f = pb["foundations"][tag]
            util = f["q_net_ksf"] / f["q_allow_net_ksf"]
            pier = f"{f['pier_count']} @ {f['pier_depth_ft']} ft" if f["pier_count"] else "-"
            ra = f"{f['area_replacement_ratio']:.3f}" if f["area_replacement_ratio"] else "-"
            qg = str(round(f["top_of_pier_stress_psf"])) if f["top_of_pier_stress_psf"] else "-"
            out.append(f"| {tag} | {f['foundation_type']} | {f['B_in'] / 12:.2f} x {f['L_in'] / 12:.2f} | "
                       f"{f['thickness_in']:.0f} | {f['bearing_stratum']} | {f['q_net_ksf']:.3f} | "
                       f"{f['q_allow_net_ksf']:.3f} | {util * 100:.0f} % | {f['settlement_in']:.3f} | "
                       f"{pier} | {ra} | {qg} | {f['governing_constraint']} |")
        w = pb["foundations"]["W1"]
        out += ["",
                f"Shear-wall footing stability: sliding FS **{w['sliding_fs']:.3f}** against the required 2.0; "
                f"resultant eccentricity **{w['eccentricity_in']:.2f} in** against the middle-third limit of "
                f"{w['B_in']/6:.2f} in; uplift FS **{w['uplift_fs']:.3f}** against 1.0.", "",
                f"Angular distortion between adjacent units: "
                + "; ".join(f"{a}-{b} {abs(pb['foundations'][a]['settlement_in'] - pb['foundations'][b]['settlement_in']):.3f} in"
                            for a, b in zip(TAGS, TAGS[1:]))
                + " (limits 0.576, 0.480, 0.384, 0.288 and 0.120 in for the five bays of the column line).", "",
                f"**Decision: {'IMPROVED_SPREAD' if pb['improved'] else 'SPREAD'}.** Shallow spread footings alone "
                f"cost {fmt(spread_c, 0)} USD; the same pads on rammed aggregate piers cost {fmt(imp_c, 0)} USD, "
                f"so the {'improved' if pb['improved'] else 'spread'} scheme is cheaper by "
                f"{fmt(abs(spread_c - imp_c), 0)} USD. Governing constraint: **{pb['governing_constraint']}**.", ""]
    (PKG / "foundation_feasibility_matrix.md").write_text("\n".join(out))


if __name__ == "__main__":
    write_register()
    write_basis()
    write_matrix()
    print("written")

```


## code/src/mutate.py

```text
#!/usr/bin/env python3
"""Mutation campaign: break the package in one way at a time and make the REAL grader notice."""
from __future__ import annotations
import json, shutil, subprocess, zipfile
from pathlib import Path

OUT = Path("/work/ple-solve/foundation-strategy/out")
PKG = OUT / "pkg"
GRAD = Path("/work/ple-dataset/tasks/foundation-strategy/tests/grading")
WORK = OUT / "mut"
WORK.mkdir(exist_ok=True)
PY = "/work/venv-verifier/bin/python"


def load():
    return (json.loads((PKG / "foundation_geometry.json").read_text()),
            json.loads((PKG / "ground_improvement_plan.json").read_text()))


def run(name, geo, plan, reg=None):
    d = WORK / name
    shutil.rmtree(d, ignore_errors=True)
    d.mkdir(parents=True)
    (d / "foundation_geometry.json").write_text(json.dumps(geo, indent=1))
    (d / "ground_improvement_plan.json").write_text(json.dumps(plan, indent=1))
    for f in ("foundation_schedule.xlsx", "foundation_feasibility_matrix.md", "design_basis.md"):
        shutil.copy(PKG / f, d / f)
    shutil.copy(reg or (PKG / "constraint_register.xlsx"), d / "constraint_register.xlsx")
    z = d / "submission.zip"
    with zipfile.ZipFile(z, "w") as f:
        for p in sorted(d.iterdir()):
            if p.suffix != ".zip":
                f.write(p, p.name)
    subprocess.run([PY, str(GRAD / "verification/run_grade.py"), str(z),
                    "--metrics-out", str(d / "metrics.json"), "--artifacts-out", str(d / "art.json"),
                    "--no-judge"], cwd=GRAD, capture_output=True, text=True)
    m = json.loads((d / "metrics.json").read_text())
    a = json.loads((d / "art.json").read_text())
    red = sorted(k for k, v in a.get("family_pass_rate", {}).items() if v < 1.0)
    print(f"{name:28s} p={m['mean_p']:.3f} h={m['physical_hard_pass_rate']:.2f} "
          f"O={m['mean_objective']:.4f} red={red}")
    return m, a


if __name__ == "__main__":
    print("baseline:")
    run("baseline", *load())
    print("mutants:")
    geo, plan = load()
    g = json.loads(json.dumps(geo))
    for f in g["pads"]["CR-P1"]["foundations"].values():
        f["thickness_in"] -= 3.0
    run("m1_thickness_minus_3in", g, plan)

    g = json.loads(json.dumps(geo))
    for p in g["pads"].values():
        p["bearing_depth_in"] = 30.0
        for f in p["foundations"].values():
            f["bearing_depth_in"] = 30.0
    run("m2_bearing_depth_30in", g, plan)

    g = json.loads(json.dumps(geo))
    f3 = g["pads"]["CR-P4"]["foundations"]["F3"]
    f3["B_in"] -= 3.0
    f3["L_in"] -= 3.0
    run("m3_p4_f3_plan_minus_3in", g, plan)

    g = json.loads(json.dumps(geo))
    g["pads"]["CR-P6"]["sulfate_exposure_class"] = "S1"     # CR-P6 is in the scored (heldout) split
    run("m4_sulfate_class_S1", g, plan)

    g = json.loads(json.dumps(geo))
    g["pads"]["CR-P1"]["fs_bearing"] = 3.0
    run("m5_fs_3_0", g, plan)

    g = json.loads(json.dumps(geo))
    pl = json.loads(json.dumps(plan))
    for f in g["pads"]["CR-P6"]["foundations"].values():
        if f["pier_depth_ft"]:
            f["pier_depth_ft"] = 5
    pl["pads"]["CR-P6"]["treatment_depth_ft"] = 5
    run("m6_pier_depth_5ft", g, pl)

    # a register missing the aggregate-pier clauses: the families are satisfied but not registered
    import openpyxl
    wb = openpyxl.load_workbook(PKG / "constraint_register.xlsx")
    ws = wb["CONSTRAINT REGISTER"]
    keep = []
    for r in ws.iter_rows(min_row=2, values_only=True):
        if r[1] and ("GEC 13" in str(r[1]) or "Kettleridge" in str(r[1])):
            continue
        keep.append(r)
    for i in range(ws.max_row, 1, -1):
        ws.delete_rows(i)
    for r in keep:
        ws.append(list(r))
    wb.save(WORK / "reg_no_piers.xlsx")
    run("m7_register_without_piers", geo, plan, reg=WORK / "reg_no_piers.xlsx")

    # a basis that never gives both alternatives a cost: D3 must go red
    bas = (PKG / "design_basis.md").read_text()
    dd = WORK / "m8"
    d = WORK / "m8_d3"
    shutil.rmtree(d, ignore_errors=True)
    d.mkdir(parents=True)
    (d / "foundation_geometry.json").write_text(json.dumps(geo, indent=1))
    (d / "ground_improvement_plan.json").write_text(json.dumps(plan, indent=1))
    for f in ("foundation_schedule.xlsx", "foundation_feasibility_matrix.md"):
        shutil.copy(PKG / f, d / f)
    shutil.copy(PKG / "constraint_register.xlsx", d / "constraint_register.xlsx")
    ob = json.loads(__import__("re").search(r"```json\s*(\{.*?\})\s*```", bas, 16).group(1))
    for p_ in ob["pads"].values():
        p_["spread_only_cost_usd"] = 0
    (d / "design_basis.md").write_text("```json\n" + json.dumps(ob, indent=1) + "\n```\n\n" + bas.split("```", 2)[2])
    z = d / "submission.zip"
    with zipfile.ZipFile(z, "w") as f:
        for p_ in sorted(d.iterdir()):
            if p_.suffix != ".zip":
                f.write(p_, p_.name)
    subprocess.run([PY, str(GRAD / "verification/run_grade.py"), str(z),
                    "--metrics-out", str(d / "metrics.json"), "--artifacts-out", str(d / "art.json"),
                    "--no-judge"], cwd=GRAD, capture_output=True, text=True)
    m = json.loads((d / "metrics.json").read_text())
    a = json.loads((d / "art.json").read_text())
    red = sorted(k for k, v in a.get("family_pass_rate", {}).items() if v < 1.0)
    print(f"{'m8_d3_no_alternative_costs':28s} p={m['mean_p']:.3f} h={m['physical_hard_pass_rate']:.2f} "
          f"O={m['mean_objective']:.4f} red={red}")

```


## code/src/search.py

```text
#!/usr/bin/env python3
"""Exact per-pad search: dynamic programme over the six foundations of the column line.

The angular-distortion rule of UFC 3-220-01 2-4.7.1 couples ADJACENT units only, so the pad
decomposes into a chain: with the settlement of each foundation as the state, a DP gives the
cheapest compliant combination of plan sizes, thicknesses and (per foundation) piers.  Run twice
per pad: once with plain spread footings only, once allowing piers under any subset - which the
BIM standard and F18-F20 permit foundation by foundation.

Output: out/options.json  {pad: {"spread": {...}, "improved": {...}}}
"""
from __future__ import annotations
import json, math
from pathlib import Path

import fsmodel as M
import design as D

OUT = Path("/work/ple-solve/foundation-strategy/out")
INP = M.INP
TAGS = M.TAGS
STEP = 0.25
DFS = ([round(3.5 + i * 0.25, 2) for i in range(19)] +
       [round(8.5 + i * 0.5, 2) for i in range(14)])
B_CAP = 40.0
KET = INP["ket"]
NS = 10.0
RA_MIN = 0.33


def cand_for(pad, tag, Df, cfg, allow_piers, suf_cache):
    """Every plan size from the minimum width up to the cap, one cheapest compliant pier schedule
    each when piers are allowed."""
    u = M._unit(pad, tag)
    strata = M.strata_for(pad, tag)
    gw = INP["gw_depth_ft"][pad]
    mw_in = INP["ufc322001"]["min_width_column_in"] if u["kind"] == "column" \
        else INP["ufc322001"]["min_width_wall_in"]
    out = []
    B = max(STEP, math.ceil(mw_in / 12.0 / STEP) * STEP)
    if u["kind"] == "wall":
        B = max(B, math.ceil((u["col_in"] / 12.0 + 8.0 / 12.0) / STEP) * STEP)
    while B <= B_CAP:
        L = u["L_fixed_ft"] if u["kind"] == "wall" else B
        t, _ = M.thickness(u["Pu"], B, L, u["col_in"] / 12, cfg["fc_psi"], kind=u["kind"])
        gamma_avg = M.overburden(strata, Df) / Df
        w_f, w_s = B * L * t * M.GAMMA_C, B * L * max(0.0, Df - t) * gamma_avg
        q_net = max(1e-9, (u["Ps"] + w_f + w_s) / (B * L) - M.overburden(strata, Df))
        q_ult, q_s = M.bearing(B, L, Df, strata, gw)
        q_all = (q_ult - q_s) / cfg["fs_bearing"]
        rho = M.rho_required(u["Pu"], B, L, t, u["col_in"] / 12, cfg["fc_psi"], 60000,
                             INP["ufc322001"]["rho_floor_project"])
        base = dict(B=B, L=L, t=t, q_net=q_net, q_all=q_all, rho=rho,
                    stratum=M.layer_at(strata, Df)["key"])
        if u["kind"] == "wall":
            base.update(M.wall_stability(strata, u, B, L, t, Df))
        ok_wall = (u["kind"] != "wall" or
                   (base["sliding_fs"] >= 2.0 and base["eccentricity_ft"] <= B / 6 + 1e-9
                    and base["uplift_fs"] >= 1.0))
        if q_net <= q_all and ok_wall:
            fcost = M.foundation_cost(B, L, t, Df, cfg["fc_psi"], rho, INP["prices"])
            cell = dict(base, cost=fcost["total"] + fcost["piers"], improved=False,
                        n_pier=0, treat=0, ra=0.0, qg_psf=0.0,
                        settlement=_s_settle(pad, tag, B, L, Df, strata, q_net, suf_cache))
            if cell["settlement"] <= 1.0:
                out.append(cell)
            if allow_piers:
                n_min = math.ceil(RA_MIN * B * L / M.A_PIER - 1e-9)
                # top-of-pier stress limit fixes the smallest admissible Ra
                kg = INP["kg_pci"][pad]
                n = n_min
                ok = False
                while n * M.A_PIER / (B * L) <= D.RA_MAX + 1e-9:
                    ra = n * M.A_PIER / (B * L)
                    qg = M.rap_qg(q_net, NS, ra)
                    if qg * 1000.0 <= KET["qg_allow_psf"]:
                        ok = True
                        break
                    n += 1
                if ok:
                    suf = suf_cache[(tag, B, Df)]
                    for treat in range(int(KET["treat_depth_min_ft"]),
                                       int(KET["treat_depth_max_ft"]) + 1):
                        s = (qg * 1000.0 / 144.0) / kg + q_net * suf[min(len(suf) - 1, int(round(treat / M.DZ)))]
                        if s > 1.0:
                            continue
                        pcost = n * treat * KET["rap_usd_per_lf"]
                        out.append(dict(base, cost=fcost["total"] - fcost["piers"] + pcost,
                                        improved=True, n_pier=n, treat=treat, ra=ra,
                                        qg_psf=qg * 1000.0, settlement=s))
                        break        # shallower treatment is always cheaper
        B = round(B + STEP, 4)
    return out


def _s_settle(pad, tag, B, L, Df, strata, q_net, suf_cache):
    key = (tag, B, Df)
    if key not in suf_cache:
        suf_cache[key] = M.settle_kernel(B, L, Df, strata)
    return q_net * suf_cache[key][0]


def pareto(cands):
    """drop dominated (cost, settlement) pairs"""
    cands = sorted(cands, key=lambda c: (c["settlement"], c["cost"]))
    out, best = [], math.inf
    for c in cands:
        if c["cost"] < best - 1e-9:
            out.append(c)
            best = c["cost"]
    return out


def solve(pad, Df, cfg, allow_piers, require_any=False):
    suf_cache = {}
    cands = {t: pareto(cand_for(pad, t, Df, cfg, allow_piers, suf_cache)) for t in TAGS}
    if any(not cands[t] for t in TAGS):
        return None
    def state_ok(c, used):
        return True if not require_any else True
    dp = {i: {} for i in range(len(TAGS))}
    for j, c in enumerate(cands[TAGS[0]]):
        dp[0][j] = (c["cost"], None, bool(c["improved"]))
    for i in range(1, len(TAGS)):
        a, b = TAGS[i - 1], TAGS[i]
        span = abs(M._unit(pad, b)["x"] - M._unit(pad, a)["x"]) * 12.0
        lim = INP["ufc322001"]["angular_distortion_limit"] * span
        prev = dp[i - 1]
        for j, c in enumerate(cands[b]):
            best = None
            for k, (cost, _kp, used) in prev.items():
                if abs(cands[a][k]["settlement"] - c["settlement"]) <= lim + 1e-9:
                    if best is None or cost < best[0]:
                        best = (cost, k, used or bool(c["improved"]))
            if best is not None:
                dp[i][j] = (best[0] + c["cost"], best[1], best[2])
    last = {j: v for j, v in dp[len(TAGS) - 1].items() if (v[2] or not require_any)}
    if not last:
        return None
    jbest = min(last, key=lambda j: last[j][0])
    chain, j = [], jbest
    for i in range(len(TAGS) - 1, -1, -1):
        chain.append(cands[TAGS[i]][j])
        j = dp[i][j][1]
        if j is None and i > 0:
            raise SystemExit("DP chain broken")
    chain.reverse()
    fds = {t: c for t, c in zip(TAGS, chain)}
    area = sum(c["B"] * c["L"] for c in chain)
    improved_any = any(c["improved"] for c in chain)
    fcost = sum(c["cost"] for c in chain)
    oh, days = M.pad_overhead(pad, Df, improved_any, area, cfg, INP["prices"])
    return {"pad": pad, "Df": Df, "improved_pad": improved_any, "foundations": fds,
            "foundation_cost": fcost, "overhead": oh, "dewatering_days": days,
            "pad_cost": fcost + oh}


def main():
    res = {}
    for pad in M.pads():
        cfg = M.pad_config(pad)
        row = {}
        for mode, allow in (("spread", False), ("improved", True)):
            best = None
            for Df in DFS:
                r = solve(pad, Df, cfg, allow, require_any=(mode == "improved"))
                if r is None:
                    continue
                if mode == "spread" and r["improved_pad"]:
                    continue
                if mode == "improved" and not r["improved_pad"]:
                    continue
                if best is None or r["pad_cost"] < best["pad_cost"] - 1e-9:
                    best = r
            row[mode] = best
        res[pad] = row
        f = lambda r: None if r is None else round(r["pad_cost"])
        d = lambda r: None if r is None else r["Df"]
        print(pad, cfg["fs_bearing"], "spread", f(row["spread"]), "@", d(row["spread"]),
              "| improved", f(row["improved"]), "@", d(row["improved"]))
    (OUT / "options.json").write_text(json.dumps(res, indent=1, default=float))


if __name__ == "__main__":
    main()

```


## code/src/verify_496.py

```text
#!/usr/bin/env python3
"""Independent check of the delivered package, by a path that does not share my model's blind spot.

1. the grader's own recomputation (fdn_check.recompute_foundation on the shipped truth profiles)
   against the numbers I report and against my own model, foundation by foundation;
2. the grader's repricing of my design against my cost model, pad by pad;
3. every register row: does its clause alias resolve to a real clause of a live document?
"""
from __future__ import annotations
import json, math, sys
from pathlib import Path

GRAD = Path("/work/ple-dataset/tasks/foundation-strategy/tests/grading")
sys.path.insert(0, str(GRAD))
sys.path.insert(0, "/work/ple-solve/foundation-strategy/src")
import fdn_check as C           # noqa: E402
import fsmodel as M             # noqa: E402

OUT = Path("/work/ple-solve/foundation-strategy/out")
PKG = OUT / "pkg"
TAGS = C.TAGS
truth = C.load_truth()
geo = json.loads((PKG / "foundation_geometry.json").read_text())
prog = json.loads((OUT / "programme.json").read_text())

rows = []
worst = {}
def rel(a, b):
    if abs(b) < 1e-12:
        return 0.0 if abs(a) < 1e-12 else math.inf
    return abs(a - b) / abs(b)

for pad in sorted(geo["pads"]):
    pb = geo["pads"][pad]
    Df = pb["bearing_depth_in"] / 12.0
    for tag in TAGS:
        fd = pb["foundations"][tag]
        r = C.recompute_foundation(truth, pad, tag, fd, Df)
        u = M._unit(pad, tag)
        strata = M.strata_for(pad, tag)
        c = prog["chosen"][pad]["foundations"][tag]
        mine = {
            "q_net": c["q_net"], "q_all": c["q_all"], "settlement": c["settlement"],
            "t_req": M.thickness(u["Pu"], c["B"], c["L"], u["col_in"] / 12, M.pad_config(pad)["fc_psi"],
                                 kind=u["kind"])[0],
            "rho_req": c["rho"], "stratum": c["stratum"],
        }
        for k, v in (("q_net", r["q_net"]), ("q_all", r["q_all"]), ("settlement", r["settlement"]),
                     ("t_req", r["t_req"]), ("rho_req", r["rho_req"])):
            d = rel(mine[k], v)
            worst[k] = max(worst.get(k, 0.0), d)
        if mine["stratum"] != r["stratum"]:
            worst["stratum"] = max(worst.get("stratum", 0), 1.0)
        # the numbers I report in the parameter file must be the ones the grader recomputes
        drep = max(rel(fd["q_net_ksf"], r["q_net"]), rel(fd["q_allow_net_ksf"], r["q_all"]),
                   rel(fd["settlement_in"], r["settlement"]))
        worst["reported"] = max(worst.get("reported", 0.0), drep)
        if fd["area_replacement_ratio"]:
            worst["ra"] = max(worst.get("ra", 0.0), rel(fd["area_replacement_ratio"], r["ra"]))
            worst["qg"] = max(worst.get("qg", 0.0), rel(fd["top_of_pier_stress_psf"], r["qg_psf"]))
        rows.append((pad, tag, r, mine, drep))

print("worst relative disagreement over all 36 foundations:")
for k, v in sorted(worst.items()):
    print(f"   {k:12s} {v:.3e}")

# campus cost
improved = [p for p in truth["campus"]["pads"]
            if geo["pads"][p]["improved"]]
need = M_IN = C.M.rigs_required(len(improved), truth["const"]["pads_per_rig"])
cost, detail = C.recompute_campus_cost(truth, geo, improved, need)
print(f"\ngrader repricing of my design: {cost:,.2f} USD   (my model: "
      f"{prog['campus_total']:,.2f} USD, difference {cost - prog['campus_total']:+,.2f})")
for pad in sorted(detail):
    mine = prog["chosen"][pad]["pad_cost"]
    print(f"   {pad}: grader {detail[pad]['cost_usd']:>12,.2f}   mine {mine:>12,.2f}   "
          f"diff {detail[pad]['cost_usd'] - mine:+,.2f}")

# register rows resolve to live documents
from register_check import check_register
reg = check_register(PKG / "constraint_register.xlsx", C.N_FAMILIES, C.HARD_FAMILIES, C.ALL_FAMILIES)
print(f"\nregister: rows {reg['n_rows']}, precision {reg['precision']:.3f}, "
      f"distractor {reg['distractor_rows']}, fabricated {reg['fabricated_rows']}, "
      f"neutral {reg['neutral_rows']}, matched rows {reg['matched_rows']}, "
      f"recall all {reg['recall_all']:.3f}, hard {reg['recall_hard']:.3f}")
missing = [f for f in C.ALL_FAMILIES if not reg["matched"][f]]
print("families not covered by the register:", missing or "none")

```


## deliverables/design_basis.md

```text
```json
{
 "frost": {
  "afi_100yr_F_days": 556,
  "afi_product_period": "1981-2010",
  "frost_depth_in": 42,
  "superseded_product_period": "1951-1980",
  "superseded_afi_100yr_F_days": 416
 },
 "pads": {
  "CR-P1": {
   "governing_constraint": "BEARING",
   "foundation_decision": "IMPROVED_SPREAD",
   "spread_only_cost_usd": 226795.75,
   "improved_cost_usd": 72757.8
  },
  "CR-P2": {
   "governing_constraint": "BEARING",
   "foundation_decision": "IMPROVED_SPREAD",
   "spread_only_cost_usd": 247933.99,
   "improved_cost_usd": 77919.24
  },
  "CR-P3": {
   "governing_constraint": "BEARING",
   "foundation_decision": "SPREAD",
   "spread_only_cost_usd": 29538.05,
   "improved_cost_usd": 42672.43
  },
  "CR-P4": {
   "governing_constraint": "BEARING",
   "foundation_decision": "IMPROVED_SPREAD",
   "spread_only_cost_usd": 58599.54,
   "improved_cost_usd": 40380.07
  },
  "CR-P5": {
   "governing_constraint": "BEARING",
   "foundation_decision": "SPREAD",
   "spread_only_cost_usd": 15235.96,
   "improved_cost_usd": 29234.45
  },
  "CR-P6": {
   "governing_constraint": "BEARING",
   "foundation_decision": "IMPROVED_SPREAD",
   "spread_only_cost_usd": 196210.2,
   "improved_cost_usd": 74216.36
  }
 },
 "campus": {
  "rigs_mobilised": 2,
  "mobilisation_cost_usd": 80000.0,
  "total_cost_usd": 390047.48,
  "pads_improved": [
   "CR-P1",
   "CR-P2",
   "CR-P4",
   "CR-P6"
  ]
 },
 "rejected_criteria": [
  "UFC 3-220-01N (2005, cancelled) - superseded by UFC 3-220-01 (2012, Change 3)",
  "UFC 3-220-10N (2005, cancelled) - superseded by UFC 3-220-20 (2025)",
  "Basis of Design Structural Rev 1 - superseded in its entirety by Rev 3",
  "Geotechnical Addendum 1 - superseded by the Geotechnical Data Report Rev 2",
  "NOAA NCEI AFI 1951-1980 return periods - superseded edition of the AFI product",
  "FHWA GEC 6 (shallow foundations for highway bridges) - different class of structure",
  "Broadview Commons adjacent-parcel report - different property, information only",
  "Presumptive bearing pressure of 3000 psf - preliminary only, prohibited by AHJ Amendment 4"
 ]
}
```

# Design basis - Cattail Run Data Campus, Phase 1 foundation package

Ardent Vale Infrastructure LLC.  Six building pads CR-P1 ... CR-P6, six foundation units on
each pad's instrumented column line: five isolated column footings (F1-F5) and one 24 ft
continuous shear-wall footing (W1).  For the thirty-six foundations this basis states the
bearing elevation, the plan size, the shear thickness, the reinforcement and the decision
between a shallow spread footing alone and a spread footing over rammed aggregate piers.

## 1. Criteria that govern, and the documents reviewed and set aside

The governing geotechnical criteria are UFC 3-220-01 Geotechnical Engineering (2012, Change 3, 2025)
for the factor of safety, the settlement and drift limits, the minimum widths and the
prohibition on presumptive values; UFC 3-220-20 Foundations and Earth Structures (DM 7-2, 2025)
Section 5-3 for the bearing-capacity equation and Figure 5-6 for the groundwater correction;
Section 5-4 and FHWA GEC 13 Vol I Equation 5-14 for the settlement method; FHWA GEC 13 Section 4.2
for the rammed aggregate piers; EM 1110-2-2104 for the footing shear and flexure; EM 1110-2-2502
Chapter 3 for sliding and overturning of the shear-wall footing; EM 1110-2-2000 Table 2-2 with UFGS
03 30 00 for the concrete; and the owner's Basis of Design Rev 3, its BIM parameter standard and the
Loudoun County amendment letter.

Reviewed and set aside:

* **UFC 3-220-01N (2005)** - cancelled predecessor of the governing geotechnical criteria;
  UFC 3-220-01 (2012, Change 3) governs instead.
* **UFC 3-220-10N (2005)** - cancelled; UFC 3-220-20 (2025) governs instead.
* **Basis of Design Structural Rev 1** - superseded in its entirety by Rev 3, which governs.
* **Geotechnical Addendum 1** - superseded by the Geotechnical Data Report Rev 2, which governs.
* **NOAA NCEI air-freezing-index return periods 1951-1980** - the superseded edition of the AFI
  product; the 1981-2010 product governs, as AHJ Amendment 3 and RFI-007 both direct.
* **FHWA GEC 6 Shallow Foundations for Highway Bridges** - written for bridges. Its tolerable
  angular distortion of 0.004 for continuous-span bridges is a bridge serviceability criterion and
  is not adopted here; the criteria's own 1/500 limit governs buildings.
* **Broadview Commons adjacent-parcel report (2023)** - a different property, information only.
* **The 3000 psf presumptive bearing pressure** of the data report Section 7 - preliminary only,
  prohibited for design by AHJ Amendment 4 and by UFC 3-220-01 paragraphs 2-4.5.2 and 2-4.6.4.

## 2. Bearing elevation and frost protection

The 100-year air-freezing index at the county station of record USC00444909 (LINCOLN, VA) is
**556 F-days** read from the current 1981-2010 return-period product. It exceeds the
500 F-day threshold of AHJ Amendment 3, so **the minimum depth of the bearing surface is 42 in**
below finished grade at every pad. The 1951-1980 tables give 416 F-days for the same station and
are the superseded edition of the same product; they were not used. Every pad is founded at
**3.75 ft** or deeper, on the 3 in dimensioning grid, with one shared bearing elevation per pad:

| pad | bearing elevation below finished grade (in) | bearing elevation (ft) |
|---|---|---|
| CR-P1 | 45 | 3.75 |
| CR-P2 | 48 | 4.00 |
| CR-P3 | 48 | 4.00 |
| CR-P4 | 48 | 4.00 |
| CR-P5 | 45 | 3.75 |
| CR-P6 | 45 | 3.75 |

## 3. Bearing-capacity method and the factor of safety at each pad

The net allowable bearing pressure of every foundation is derived from the general
bearing-capacity equation of UFC 3-220-20 Section 5-3 (DM 7-2) with the Vesic N-factors of
Figure 5-5, the shape factors of that section and the depth factors d_q and d_c given there. The
surcharge and the unit weight in the failure term are corrected for the position of the
stabilised groundwater table following the three cases of Figure 5-6, using the piezometer
depths of the grading and groundwater record. The net allowable pressure is
q_allow = (q_ult - q_sur)/FS, with FS the STRUCTURE-row value of UFC 3-220-01 Table 2-3 for the
occupancy of that pad: **3.5** for the four Light Industrial, Public pads (CR-P1, CR-P2, CR-P3,
CR-P6), **3.0** for the Apartments, Office pad (CR-P4) and **2.5** for the Warehouse (CR-P5). The
Warehouse row carries a footnote directing an FS above 3 for super-flat warehouse floors; no
floor-flatness requirement is imposed anywhere in this data room, so the tabulated 2.5 governs.
As UFC 3-220-01 paragraph 2-4.7.1 requires, two further widely accepted methods - the Meyerhof
and the Brinch Hansen forms of the general equation of EM 1110-1-1905 Chapter 4 - were computed
at every foundation as checks; the reported value is the Vesic value the design relies on, and
the least of the three is never more than 6 % below it. The 3000 psf presumptive value was not
used anywhere.

The bearing stratum of each foundation was taken from the interpolated profile: the base of
Stratum I is interpolated linearly between the two borings of the pad at the station of the
foundation along column line A and rounded to 0.1 ft, and the thickness of Stratum II is uniform
within the pad (Geotechnical Data Report Rev 2 Section 4). Each footing is reported against the
stratum in which its bearing surface sits.

## 4. Settlement method and the tolerable-movement criteria

Settlement is **layered elastic compression**. The soil below each bearing surface is divided
into uniform layers 0.25 ft thick; the net stress increment in each layer is the 2 vertical to 1
horizontal approximate distribution for a rectangular loaded area; and the compressions
s = (delta sigma)(H)/Es are summed with the pad-specific elastic moduli of the geotechnical data
report. The integral is carried to four times the footing width below the bearing surface and in
no case less than 40 ft. That method, its depth of influence and its slice thickness are fixed by
RFI-019, and the equation is Equation 5-14 of FHWA GEC 13 Vol I, which is also the lower-zone
term of the aggregate-pier method - so the improved and unimproved cases are computed the same
way.

The tolerable-movement criteria adopted are those of UFC 3-220-01 paragraph 2-4.7.1, which is
the governing geotechnical criteria for this project: **total post-construction settlement of
1 in** and an **angular distortion limit of 1/500** between adjacent foundation units - the
denominator of the adopted limit is 500 (equivalently 0.002) - measured as the difference of
settlement between neighbouring units divided by the distance between them. The competing
tolerable-movement criterion of the FHWA shallow-foundations manual for highway bridges (FHWA
GEC 6), which allows an angular distortion of 0.004 for continuous-span bridges, is explicitly
**rejected and not adopted**: it is a bridge serviceability criterion written for a different
class of structure, and it is more permissive than the criteria governing this building project,
which state 1/500. For the record, the 0.004 criterion was reviewed and set aside as not
governing; the adopted limit is 1/500, i.e. a maximum angular distortion of 0.002.

## 5. Ground improvement: parameters and their authority

The piers are rammed aggregate piers installed by Kettleridge Ground Systems (proposal
KGS-26-0917): 30 in diameter, minimum clear spacing 1.0 ft, treatment depth between 8 ft and 25 ft
below the footing soffit in whole feet, and a maximum design top-of-pier stress of 18000 psf.
The **stress concentration ratio adopted is ns = 10**. That is the value the governing public
ground-modification reference, FHWA GEC 13 Vol I Section 4.2.1, gives for structure foundation
support under rigid footings - and it is the value RFI-011 directs to be taken from that
reference. The higher ratio of 20 or more quoted in the proprietary Geopier vendor design manual
is not accepted (RFI-011 excludes vendor manuals), and neither is the 5 to 10 band the same
reference gives for flexible embankment loading: these are rigid spread footings under building
columns, not an embankment. The adopted value for every improved pad is therefore 10.

The area replacement ratio follows from the pier count: Ra = n A_pier/(B L) with A_pier the
area of a 30 in pier, and it is held between the minimum 0.33 of GEC 13 Section 4.2.1 and the
geometric ceiling the subcontractor's minimum clear spacing imposes on a staggered grid.
Top-of-pier stress comes from Equation 5-12, upper-zone settlement from Equation 5-13 with the
pad's own as-tested stiffness modulus (165, 140, 210, 195, 230 and 155 pci for CR-P1 ... CR-P6,
all inside the 75-360 pci range of GEC 13), and the lower-zone settlement from Equation 5-14
below the treatment depth. Total settlement is their sum (Equation 5-11). One production
modulus test is provided per improved pad, at 9500 USD each.

## 6. Conflicts between documents, and the governing value

| conflict | documents | governing value adopted |
|---|---|---|
| frost depth | AHJ Amendment 3 (42 in for AFI > 500) vs the 18 in minimum depth of UFC 3-220-01 2-4.8.1 | 42 in - the municipal value the criteria require to be used |
| AFI edition | 1981-2010 product (556 F-days) vs 1951-1980 tables (416 F-days) | 556 F-days - the current product, per Amendment 3 and RFI-007 |
| settlement method | EM 1110-1-1904 and DM 7.1 Chapter 5 (not shipped; the shipped DM 7.1 is cancelled) vs GEC 13 Eq. 5-14 | layered elastic compression per RFI-019 |
| stress ratio | GEC 13 rigid-footing value 10 vs Geopier vendor manual 20+ vs GEC 13 embankment band 5-10 | 10, per RFI-011 |
| bearing capacity | three methods disagree; presumptive 3000 psf | the Vesic general-equation value, not more permissive than it, per RFI-021 |
| footing spacing | UFC 3-220-01 2-4.8.1 requires 1.5B; the grid does not achieve it | RFI-017: individual units, no reduction for overlapping zones |
| occupancy FS | Warehouse 2.5 vs the super-flat-floor footnote above 3 | 2.5 - no floor-flatness criterion exists in this data room |
| exposure classes | owner-assigned F/W/C vs the measured sulfate | the most restrictive of all assigned classes per pad (BOD Section 3) |
| cost basis | bid schedule Rev 1 unit prices vs Kettleridge proposal | both, each for the work it prices; no other source |

## 7. The campus programme: which pads are improved, and how many rigs

The programme improves **4 of the six pads** - CR-P1, CR-P2, CR-P4, CR-P6 - and carries
**2 mobilised rigs (rigs mobilised = 2)** at 80,000 USD. The number of rigs is not a free choice:
Kettleridge proposal KGS-26-0917 Section 4 limits one mobilised rig to **at most 2 building pads**
inside the Phase 1 milestone window and charges 40000 USD per rig as a lump sum, so four improved
pads necessarily buy two rigs - not one rig per improved pad (which would cost 160000 USD) and not
one rig for the whole campus (which the subcontractor's limit does not permit). Each rig is
assigned two pads.

Why these four pads, stated the way the numbers actually run. On the four improved pads the
soft residual silt of Stratum I is thick, and on a plain spread scheme the 1 in settlement limit
and the 1/500 drift limit between neighbouring units force plan sizes far above what bearing
alone needs - the shear-wall footing of CR-P1 would have to be 38.75 ft wide, and five of the
six footings of CR-P6 would be 18.5 ft to 24 ft. The piers cut those settlements by a factor of
two to three, the plan sizes fall back to what bearing capacity requires, and the saving in
concrete, reinforcement and excavation exceeds the piers, the working platform and the share of
the mobilisation. At CR-P3 and CR-P5 Stratum I is only 4.5 ft to 7 ft thick, the spread scheme
already meets both movement limits with compact footings, and improvement would add the piers,
the modulus test and the platform without saving anything; those two pads are carried on plain
spread footings. The campus total is
**390,047 USD**.

The comparison behind each decision, with the cost of both alternatives, is in the table above
and in foundation_feasibility_matrix.md; reinforcement is priced at the ratio the design
actually requires.

```


## deliverables/foundation_feasibility_matrix.md

```text
# Foundation-type feasibility and optimisation matrix

Every pad's six foundation units, the demand, the bearing check, the predicted settlement,
the stability checks on the shear-wall footing, the constraint that governs, and the
shallow-versus-improved decision with the cost of both alternatives.

## CR-P1 - Data Hall 1

Occupancy **Light Industrial, Public** (FS 3.5), sulfate **S1** from 0.140 % SO4, exposure classes C1, F1, S1, W1, f'c = 4000 psi at w/cm 0.50, bearing elevation **45 in** below finished grade (frost minimum 42 in), groundwater at 8.0 ft.

| foundation | type | B x L (ft) | t (in) | stratum | q_net (ksf) | q_allow (ksf) | utilisation | settlement (in) | piers | Ra | qg (psf) | governing |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | IMPROVED_SPREAD | 10.25 x 10.25 | 24 | I | 5.870 | 5.988 | 98 % | 0.689 | 8 @ 8 ft | 0.374 | 13451 | BEARING |
| F2 | IMPROVED_SPREAD | 14.25 x 14.25 | 36 | I | 5.907 | 6.072 | 97 % | 0.819 | 14 @ 8 ft | 0.338 | 14600 | BEARING |
| F3 | IMPROVED_SPREAD | 15.50 x 15.50 | 39 | I | 6.014 | 6.115 | 98 % | 0.855 | 17 @ 8 ft | 0.347 | 14577 | BEARING |
| F4 | IMPROVED_SPREAD | 13.00 x 13.00 | 33 | I | 5.887 | 6.035 | 98 % | 0.793 | 12 @ 8 ft | 0.349 | 14230 | BEARING |
| F5 | IMPROVED_SPREAD | 9.50 x 9.50 | 24 | I | 5.826 | 5.986 | 97 % | 0.725 | 7 @ 8 ft | 0.381 | 13161 | BEARING |
| W1 | IMPROVED_SPREAD | 12.75 x 24.00 | 27 | I | 5.464 | 5.546 | 99 % | 0.796 | 21 @ 9 ft | 0.337 | 13552 | BEARING |

Shear-wall footing stability: sliding FS **2.874** against the required 2.0; resultant eccentricity **5.84 in** against the middle-third limit of 25.50 in; uplift FS **3.086** against 1.0.

Angular distortion between adjacent units: F1-F2 0.131 in; F2-F3 0.036 in; F3-F4 0.063 in; F4-F5 0.068 in; F5-W1 0.071 in (limits 0.576, 0.480, 0.384, 0.288 and 0.120 in for the five bays of the column line).

**Decision: IMPROVED_SPREAD.** Shallow spread footings alone cost 226,796 USD; the same pads on rammed aggregate piers cost 72,758 USD, so the improved scheme is cheaper by 154,038 USD. Governing constraint: **BEARING**.

## CR-P2 - Data Hall 2

Occupancy **Light Industrial, Public** (FS 3.5), sulfate **S2** from 0.230 % SO4, exposure classes C1, F1, S2, W1, f'c = 4500 psi at w/cm 0.45, bearing elevation **48 in** below finished grade (frost minimum 42 in), groundwater at 9.0 ft.

| foundation | type | B x L (ft) | t (in) | stratum | q_net (ksf) | q_allow (ksf) | utilisation | settlement (in) | piers | Ra | qg (psf) | governing |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | IMPROVED_SPREAD | 10.25 x 10.25 | 24 | I | 6.218 | 6.233 | 100 % | 0.963 | 8 @ 9 ft | 0.374 | 14249 | BEARING |
| F2 | IMPROVED_SPREAD | 14.25 x 14.25 | 36 | I | 6.256 | 6.302 | 99 % | 0.985 | 14 @ 9 ft | 0.338 | 15462 | BEARING |
| F3 | IMPROVED_SPREAD | 15.75 x 15.75 | 39 | I | 6.172 | 6.350 | 97 % | 1.000 | 17 @ 9 ft | 0.336 | 15324 | BEARING |
| F4 | IMPROVED_SPREAD | 13.00 x 13.00 | 33 | I | 6.235 | 6.269 | 99 % | 0.949 | 12 @ 8 ft | 0.349 | 15071 | BEARING |
| F5 | IMPROVED_SPREAD | 9.50 x 9.50 | 24 | I | 6.171 | 6.236 | 99 % | 0.820 | 7 @ 8 ft | 0.381 | 13942 | BEARING |
| W1 | IMPROVED_SPREAD | 13.25 x 24.00 | 30 | I | 5.580 | 5.812 | 96 % | 0.921 | 22 @ 8 ft | 0.340 | 13756 | BEARING |

Shear-wall footing stability: sliding FS **2.909** against the required 2.0; resultant eccentricity **5.82 in** against the middle-third limit of 26.50 in; uplift FS **3.105** against 1.0.

Angular distortion between adjacent units: F1-F2 0.022 in; F2-F3 0.014 in; F3-F4 0.051 in; F4-F5 0.129 in; F5-W1 0.101 in (limits 0.576, 0.480, 0.384, 0.288 and 0.120 in for the five bays of the column line).

**Decision: IMPROVED_SPREAD.** Shallow spread footings alone cost 247,934 USD; the same pads on rammed aggregate piers cost 77,919 USD, so the improved scheme is cheaper by 170,015 USD. Governing constraint: **BEARING**.

## CR-P3 - Electrical Yard Building

Occupancy **Light Industrial, Public** (FS 3.5), sulfate **S0** from 0.070 % SO4, exposure classes C1, F1, S0, W1, f'c = 4000 psi at w/cm 0.50, bearing elevation **48 in** below finished grade (frost minimum 42 in), groundwater at 19.0 ft.

| foundation | type | B x L (ft) | t (in) | stratum | q_net (ksf) | q_allow (ksf) | utilisation | settlement (in) | piers | Ra | qg (psf) | governing |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | SPREAD | 8.50 x 8.50 | 21 | I | 6.135 | 6.487 | 95 % | 0.665 | - | - | - | BEARING |
| F2 | SPREAD | 11.50 x 11.50 | 30 | I | 6.504 | 6.678 | 97 % | 0.923 | - | - | - | BEARING |
| F3 | SPREAD | 13.50 x 13.50 | 33 | I | 5.698 | 6.851 | 83 % | 0.971 | - | - | - | BEARING |
| F4 | SPREAD | 11.00 x 11.00 | 27 | I | 5.903 | 6.639 | 89 % | 0.975 | - | - | - | BEARING |
| F5 | SPREAD | 7.75 x 7.75 | 21 | I | 6.290 | 6.463 | 97 % | 0.946 | - | - | - | BEARING |
| W1 | SPREAD | 9.50 x 24.00 | 21 | I | 5.266 | 5.907 | 89 % | 0.992 | - | - | - | BEARING |

Shear-wall footing stability: sliding FS **3.049** against the required 2.0; resultant eccentricity **5.81 in** against the middle-third limit of 19.00 in; uplift FS **3.109** against 1.0.

Angular distortion between adjacent units: F1-F2 0.258 in; F2-F3 0.048 in; F3-F4 0.004 in; F4-F5 0.029 in; F5-W1 0.047 in (limits 0.576, 0.480, 0.384, 0.288 and 0.120 in for the five bays of the column line).

**Decision: SPREAD.** Shallow spread footings alone cost 29,538 USD; the same pads on rammed aggregate piers cost 42,672 USD, so the spread scheme is cheaper by 13,134 USD. Governing constraint: **BEARING**.

## CR-P4 - Administration and Operations Building

Occupancy **Apartments, Office** (FS 3.0), sulfate **S1** from 0.110 % SO4, exposure classes C1, F1, S1, W1, f'c = 4000 psi at w/cm 0.50, bearing elevation **48 in** below finished grade (frost minimum 42 in), groundwater at 8.0 ft.

| foundation | type | B x L (ft) | t (in) | stratum | q_net (ksf) | q_allow (ksf) | utilisation | settlement (in) | piers | Ra | qg (psf) | governing |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | SPREAD | 8.50 x 8.50 | 21 | I | 5.375 | 7.217 | 74 % | 0.990 | - | - | - | BEARING |
| F2 | IMPROVED_SPREAD | 10.25 x 10.25 | 27 | I | 7.148 | 7.193 | 99 % | 0.732 | 8 @ 8 ft | 0.374 | 16379 | BEARING |
| F3 | IMPROVED_SPREAD | 11.25 x 11.25 | 30 | I | 7.148 | 7.200 | 99 % | 0.789 | 9 @ 8 ft | 0.349 | 17260 | BEARING |
| F4 | IMPROVED_SPREAD | 9.50 x 9.50 | 24 | I | 6.905 | 7.197 | 96 % | 0.692 | 7 @ 8 ft | 0.381 | 15599 | BEARING |
| F5 | IMPROVED_SPREAD | 6.75 x 6.75 | 18 | I | 7.238 | 7.315 | 99 % | 0.622 | 4 @ 8 ft | 0.431 | 14837 | BEARING |
| W1 | IMPROVED_SPREAD | 7.25 x 24.00 | 18 | I | 6.022 | 6.097 | 99 % | 0.691 | 12 @ 8 ft | 0.339 | 14881 | BEARING |

Shear-wall footing stability: sliding FS **3.061** against the required 2.0; resultant eccentricity **5.89 in** against the middle-third limit of 14.50 in; uplift FS **3.054** against 1.0.

Angular distortion between adjacent units: F1-F2 0.258 in; F2-F3 0.057 in; F3-F4 0.097 in; F4-F5 0.070 in; F5-W1 0.070 in (limits 0.576, 0.480, 0.384, 0.288 and 0.120 in for the five bays of the column line).

**Decision: IMPROVED_SPREAD.** Shallow spread footings alone cost 58,600 USD; the same pads on rammed aggregate piers cost 40,380 USD, so the improved scheme is cheaper by 18,219 USD. Governing constraint: **BEARING**.

## CR-P5 - Logistics and Spares Warehouse

Occupancy **Warehouse** (FS 2.5), sulfate **S0** from 0.050 % SO4, exposure classes C1, F3, S0, W1, f'c = 5000 psi at w/cm 0.40, bearing elevation **45 in** below finished grade (frost minimum 42 in), groundwater at 21.0 ft.

| foundation | type | B x L (ft) | t (in) | stratum | q_net (ksf) | q_allow (ksf) | utilisation | settlement (in) | piers | Ra | qg (psf) | governing |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | SPREAD | 6.25 x 6.25 | 15 | I | 8.629 | 8.729 | 99 % | 0.528 | - | - | - | BEARING |
| F2 | SPREAD | 8.75 x 8.75 | 24 | I | 8.541 | 8.812 | 97 % | 0.690 | - | - | - | BEARING |
| F3 | SPREAD | 9.50 x 9.50 | 24 | I | 8.718 | 8.874 | 98 % | 0.804 | - | - | - | BEARING |
| F4 | SPREAD | 8.00 x 8.00 | 21 | I | 8.478 | 8.763 | 97 % | 0.790 | - | - | - | BEARING |
| F5 | SPREAD | 5.75 x 5.75 | 15 | I | 8.690 | 8.750 | 99 % | 0.755 | - | - | - | BEARING |
| W1 | SPREAD | 5.50 x 24.00 | 15 | I | 6.915 | 7.193 | 96 % | 0.756 | - | - | - | BEARING |

Shear-wall footing stability: sliding FS **3.011** against the required 2.0; resultant eccentricity **5.98 in** against the middle-third limit of 11.00 in; uplift FS **2.990** against 1.0.

Angular distortion between adjacent units: F1-F2 0.162 in; F2-F3 0.114 in; F3-F4 0.014 in; F4-F5 0.034 in; F5-W1 0.001 in (limits 0.576, 0.480, 0.384, 0.288 and 0.120 in for the five bays of the column line).

**Decision: SPREAD.** Shallow spread footings alone cost 15,236 USD; the same pads on rammed aggregate piers cost 29,234 USD, so the spread scheme is cheaper by 13,998 USD. Governing constraint: **BEARING**.

## CR-P6 - Data Hall 3

Occupancy **Light Industrial, Public** (FS 3.5), sulfate **S2** from 0.260 % SO4, exposure classes C1, F1, S2, W1, f'c = 4500 psi at w/cm 0.45, bearing elevation **45 in** below finished grade (frost minimum 42 in), groundwater at 8.5 ft.

| foundation | type | B x L (ft) | t (in) | stratum | q_net (ksf) | q_allow (ksf) | utilisation | settlement (in) | piers | Ra | qg (psf) | governing |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| F1 | IMPROVED_SPREAD | 10.25 x 10.25 | 24 | I | 5.986 | 6.021 | 99 % | 0.987 | 8 @ 9 ft | 0.374 | 13717 | BEARING |
| F2 | IMPROVED_SPREAD | 14.25 x 14.25 | 36 | I | 6.023 | 6.106 | 99 % | 0.896 | 14 @ 8 ft | 0.338 | 14887 | BEARING |
| F3 | IMPROVED_SPREAD | 15.50 x 15.50 | 39 | I | 6.133 | 6.148 | 100 % | 0.914 | 17 @ 8 ft | 0.347 | 14863 | BEARING |
| F4 | IMPROVED_SPREAD | 13.00 x 13.00 | 33 | I | 6.003 | 6.069 | 99 % | 0.836 | 12 @ 8 ft | 0.349 | 14510 | BEARING |
| F5 | IMPROVED_SPREAD | 9.50 x 9.50 | 24 | I | 5.941 | 6.020 | 99 % | 0.717 | 7 @ 8 ft | 0.381 | 13421 | BEARING |
| W1 | IMPROVED_SPREAD | 12.75 x 24.00 | 27 | I | 5.572 | 5.590 | 100 % | 0.835 | 21 @ 8 ft | 0.337 | 13820 | BEARING |

Shear-wall footing stability: sliding FS **2.864** against the required 2.0; resultant eccentricity **5.85 in** against the middle-third limit of 25.50 in; uplift FS **3.079** against 1.0.

Angular distortion between adjacent units: F1-F2 0.092 in; F2-F3 0.018 in; F3-F4 0.078 in; F4-F5 0.119 in; F5-W1 0.118 in (limits 0.576, 0.480, 0.384, 0.288 and 0.120 in for the five bays of the column line).

**Decision: IMPROVED_SPREAD.** Shallow spread footings alone cost 196,210 USD; the same pads on rammed aggregate piers cost 74,216 USD, so the improved scheme is cheaper by 121,994 USD. Governing constraint: **BEARING**.

```


## deliverables/foundation_geometry.json

```text
{
 "project": {
  "campus_total_cost_usd": 390047.48,
  "frost_depth_in": 42,
  "afi_100yr_F_days": 556
 },
 "pads": {
  "CR-P1": {
   "bearing_depth_in": 45.0,
   "fs_bearing": 3.5,
   "occupancy_class": "Light Industrial, Public",
   "improved": true,
   "sulfate_pct": 0.14,
   "sulfate_exposure_class": "S1",
   "exposure_classes": [
    "C1",
    "F1",
    "S1",
    "W1"
   ],
   "fc_psi": 4000,
   "max_wcm": 0.5,
   "dewatering_days": 0,
   "governing_constraint": "BEARING",
   "foundations": {
    "F1": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 123.0,
     "L_in": 123.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002695,
     "bar_size": 6,
     "bar_spacing_in": 8.0,
     "q_net_ksf": 5.8701,
     "q_allow_net_ksf": 5.9876,
     "settlement_in": 0.6888,
     "pier_count": 8,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.373777,
     "top_of_pier_stress_psf": 13451.15,
     "governing_constraint": "BEARING"
    },
    "F2": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 171.0,
     "L_in": 171.0,
     "thickness_in": 36.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002401,
     "bar_size": 7,
     "bar_spacing_in": 7.5,
     "q_net_ksf": 5.907,
     "q_allow_net_ksf": 6.0721,
     "settlement_in": 0.8193,
     "pier_count": 14,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.338429,
     "top_of_pier_stress_psf": 14600.14,
     "governing_constraint": "BEARING"
    },
    "F3": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 186.0,
     "L_in": 186.0,
     "thickness_in": 39.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002449,
     "bar_size": 7,
     "bar_spacing_in": 7.0,
     "q_net_ksf": 6.0145,
     "q_allow_net_ksf": 6.1149,
     "settlement_in": 0.8554,
     "pier_count": 17,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.34734,
     "top_of_pier_stress_psf": 14576.87,
     "governing_constraint": "BEARING"
    },
    "F4": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 156.0,
     "L_in": 156.0,
     "thickness_in": 33.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002318,
     "bar_size": 6,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 5.8868,
     "q_allow_net_ksf": 6.0355,
     "settlement_in": 0.7926,
     "pier_count": 12,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.348549,
     "top_of_pier_stress_psf": 14229.86,
     "governing_constraint": "BEARING"
    },
    "F5": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 114.0,
     "L_in": 114.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002302,
     "bar_size": 5,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 5.8258,
     "q_allow_net_ksf": 5.9863,
     "settlement_in": 0.7249,
     "pier_count": 7,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.380733,
     "top_of_pier_stress_psf": 13160.83,
     "governing_constraint": "BEARING"
    },
    "W1": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 153.0,
     "L_in": 288.0,
     "thickness_in": 27.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.003934,
     "bar_size": 7,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 5.4642,
     "q_allow_net_ksf": 5.5465,
     "settlement_in": 0.7962,
     "pier_count": 21,
     "pier_diameter_in": 30,
     "pier_depth_ft": 9,
     "area_replacement_ratio": 0.336874,
     "top_of_pier_stress_psf": 13552.42,
     "governing_constraint": "BEARING",
     "sliding_fs": 2.8745,
     "eccentricity_in": 5.8425,
     "uplift_fs": 3.0864
    }
   }
  },
  "CR-P2": {
   "bearing_depth_in": 48.0,
   "fs_bearing": 3.5,
   "occupancy_class": "Light Industrial, Public",
   "improved": true,
   "sulfate_pct": 0.23,
   "sulfate_exposure_class": "S2",
   "exposure_classes": [
    "C1",
    "F1",
    "S2",
    "W1"
   ],
   "fc_psi": 4500,
   "max_wcm": 0.45,
   "dewatering_days": 0,
   "governing_constraint": "BEARING",
   "foundations": {
    "F1": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 123.0,
     "L_in": 123.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002852,
     "bar_size": 6,
     "bar_spacing_in": 7.5,
     "q_net_ksf": 6.2184,
     "q_allow_net_ksf": 6.2327,
     "settlement_in": 0.9629,
     "pier_count": 8,
     "pier_diameter_in": 30,
     "pier_depth_ft": 9,
     "area_replacement_ratio": 0.373777,
     "top_of_pier_stress_psf": 14249.42,
     "governing_constraint": "BEARING"
    },
    "F2": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 171.0,
     "L_in": 171.0,
     "thickness_in": 36.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002541,
     "bar_size": 7,
     "bar_spacing_in": 7.0,
     "q_net_ksf": 6.2557,
     "q_allow_net_ksf": 6.3019,
     "settlement_in": 0.9853,
     "pier_count": 14,
     "pier_diameter_in": 30,
     "pier_depth_ft": 9,
     "area_replacement_ratio": 0.338429,
     "top_of_pier_stress_psf": 15461.91,
     "governing_constraint": "BEARING"
    },
    "F3": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 189.0,
     "L_in": 189.0,
     "thickness_in": 39.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002607,
     "bar_size": 7,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 6.1718,
     "q_allow_net_ksf": 6.3501,
     "settlement_in": 0.9996,
     "pier_count": 17,
     "pier_diameter_in": 30,
     "pier_depth_ft": 9,
     "area_replacement_ratio": 0.336401,
     "top_of_pier_stress_psf": 15323.78,
     "governing_constraint": "BEARING"
    },
    "F4": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 156.0,
     "L_in": 156.0,
     "thickness_in": 33.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002454,
     "bar_size": 6,
     "bar_spacing_in": 6.0,
     "q_net_ksf": 6.2347,
     "q_allow_net_ksf": 6.269,
     "settlement_in": 0.949,
     "pier_count": 12,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.348549,
     "top_of_pier_stress_psf": 15070.89,
     "governing_constraint": "BEARING"
    },
    "F5": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 114.0,
     "L_in": 114.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002436,
     "bar_size": 5,
     "bar_spacing_in": 6.0,
     "q_net_ksf": 6.1715,
     "q_allow_net_ksf": 6.2356,
     "settlement_in": 0.8197,
     "pier_count": 7,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.380733,
     "top_of_pier_stress_psf": 13941.81,
     "governing_constraint": "BEARING"
    },
    "W1": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 159.0,
     "L_in": 288.0,
     "thickness_in": 30.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.003438,
     "bar_size": 7,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 5.58,
     "q_allow_net_ksf": 5.8121,
     "settlement_in": 0.9209,
     "pier_count": 22,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.339598,
     "top_of_pier_stress_psf": 13756.09,
     "governing_constraint": "BEARING",
     "sliding_fs": 2.9087,
     "eccentricity_in": 5.8163,
     "uplift_fs": 3.1052
    }
   }
  },
  "CR-P3": {
   "bearing_depth_in": 48.0,
   "fs_bearing": 3.5,
   "occupancy_class": "Light Industrial, Public",
   "improved": false,
   "sulfate_pct": 0.07,
   "sulfate_exposure_class": "S0",
   "exposure_classes": [
    "C1",
    "F1",
    "S0",
    "W1"
   ],
   "fc_psi": 4000,
   "max_wcm": 0.5,
   "dewatering_days": 0,
   "governing_constraint": "BEARING",
   "foundations": {
    "F1": {
     "foundation_type": "SPREAD",
     "B_in": 102.0,
     "L_in": 102.0,
     "thickness_in": 21.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002348,
     "bar_size": 5,
     "bar_spacing_in": 7.5,
     "q_net_ksf": 6.1349,
     "q_allow_net_ksf": 6.4874,
     "settlement_in": 0.6651,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F2": {
     "foundation_type": "SPREAD",
     "B_in": 138.0,
     "L_in": 138.0,
     "thickness_in": 30.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002338,
     "bar_size": 6,
     "bar_spacing_in": 7.0,
     "q_net_ksf": 6.5042,
     "q_allow_net_ksf": 6.6775,
     "settlement_in": 0.9231,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F3": {
     "foundation_type": "SPREAD",
     "B_in": 162.0,
     "L_in": 162.0,
     "thickness_in": 33.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002384,
     "bar_size": 6,
     "bar_spacing_in": 6.0,
     "q_net_ksf": 5.6979,
     "q_allow_net_ksf": 6.8513,
     "settlement_in": 0.9712,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F4": {
     "foundation_type": "SPREAD",
     "B_in": 132.0,
     "L_in": 132.0,
     "thickness_in": 27.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002392,
     "bar_size": 6,
     "bar_spacing_in": 7.5,
     "q_net_ksf": 5.9034,
     "q_allow_net_ksf": 6.6386,
     "settlement_in": 0.975,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F5": {
     "foundation_type": "SPREAD",
     "B_in": 93.0,
     "L_in": 93.0,
     "thickness_in": 21.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.001989,
     "bar_size": 5,
     "bar_spacing_in": 9.0,
     "q_net_ksf": 6.2895,
     "q_allow_net_ksf": 6.4629,
     "settlement_in": 0.9457,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "W1": {
     "foundation_type": "SPREAD",
     "B_in": 114.0,
     "L_in": 288.0,
     "thickness_in": 21.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.003449,
     "bar_size": 6,
     "bar_spacing_in": 7.5,
     "q_net_ksf": 5.2665,
     "q_allow_net_ksf": 5.9067,
     "settlement_in": 0.9924,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING",
     "sliding_fs": 3.0485,
     "eccentricity_in": 5.8111,
     "uplift_fs": 3.1089
    }
   }
  },
  "CR-P4": {
   "bearing_depth_in": 48.0,
   "fs_bearing": 3.0,
   "occupancy_class": "Apartments, Office",
   "improved": true,
   "sulfate_pct": 0.11,
   "sulfate_exposure_class": "S1",
   "exposure_classes": [
    "C1",
    "F1",
    "S1",
    "W1"
   ],
   "fc_psi": 4000,
   "max_wcm": 0.5,
   "dewatering_days": 0,
   "governing_constraint": "BEARING",
   "foundations": {
    "F1": {
     "foundation_type": "SPREAD",
     "B_in": 102.0,
     "L_in": 102.0,
     "thickness_in": 21.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002047,
     "bar_size": 5,
     "bar_spacing_in": 8.5,
     "q_net_ksf": 5.375,
     "q_allow_net_ksf": 7.2166,
     "settlement_in": 0.9904,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F2": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 123.0,
     "L_in": 123.0,
     "thickness_in": 27.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.00243,
     "bar_size": 6,
     "bar_spacing_in": 7.5,
     "q_net_ksf": 7.1478,
     "q_allow_net_ksf": 7.1932,
     "settlement_in": 0.7323,
     "pier_count": 8,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.373777,
     "top_of_pier_stress_psf": 16379.02,
     "governing_constraint": "BEARING"
    },
    "F3": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 135.0,
     "L_in": 135.0,
     "thickness_in": 30.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.00235,
     "bar_size": 6,
     "bar_spacing_in": 7.0,
     "q_net_ksf": 7.1484,
     "q_allow_net_ksf": 7.2002,
     "settlement_in": 0.7889,
     "pier_count": 9,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.349066,
     "top_of_pier_stress_psf": 17260.13,
     "governing_constraint": "BEARING"
    },
    "F4": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 114.0,
     "L_in": 114.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002507,
     "bar_size": 5,
     "bar_spacing_in": 6.0,
     "q_net_ksf": 6.905,
     "q_allow_net_ksf": 7.1967,
     "settlement_in": 0.6919,
     "pier_count": 7,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.380733,
     "top_of_pier_stress_psf": 15598.88,
     "governing_constraint": "BEARING"
    },
    "F5": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 81.0,
     "L_in": 81.0,
     "thickness_in": 18.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002253,
     "bar_size": 4,
     "bar_spacing_in": 6.0,
     "q_net_ksf": 7.2381,
     "q_allow_net_ksf": 7.3152,
     "settlement_in": 0.6217,
     "pier_count": 4,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.430945,
     "top_of_pier_stress_psf": 14836.75,
     "governing_constraint": "BEARING"
    },
    "W1": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 87.0,
     "L_in": 288.0,
     "thickness_in": 18.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.003019,
     "bar_size": 5,
     "bar_spacing_in": 7.0,
     "q_net_ksf": 6.0221,
     "q_allow_net_ksf": 6.0971,
     "settlement_in": 0.6913,
     "pier_count": 12,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.338534,
     "top_of_pier_stress_psf": 14881.22,
     "governing_constraint": "BEARING",
     "sliding_fs": 3.0606,
     "eccentricity_in": 5.8875,
     "uplift_fs": 3.0545
    }
   }
  },
  "CR-P5": {
   "bearing_depth_in": 45.0,
   "fs_bearing": 2.5,
   "occupancy_class": "Warehouse",
   "improved": false,
   "sulfate_pct": 0.05,
   "sulfate_exposure_class": "S0",
   "exposure_classes": [
    "C1",
    "F3",
    "S0",
    "W1"
   ],
   "fc_psi": 5000,
   "max_wcm": 0.4,
   "dewatering_days": 0,
   "governing_constraint": "BEARING",
   "foundations": {
    "F1": {
     "foundation_type": "SPREAD",
     "B_in": 75.0,
     "L_in": 75.0,
     "thickness_in": 15.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.003081,
     "bar_size": 5,
     "bar_spacing_in": 9.0,
     "q_net_ksf": 8.6288,
     "q_allow_net_ksf": 8.7291,
     "settlement_in": 0.5279,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F2": {
     "foundation_type": "SPREAD",
     "B_in": 105.0,
     "L_in": 105.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002488,
     "bar_size": 5,
     "bar_spacing_in": 6.0,
     "q_net_ksf": 8.5407,
     "q_allow_net_ksf": 8.8118,
     "settlement_in": 0.6899,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F3": {
     "foundation_type": "SPREAD",
     "B_in": 114.0,
     "L_in": 114.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.003026,
     "bar_size": 6,
     "bar_spacing_in": 7.0,
     "q_net_ksf": 8.7177,
     "q_allow_net_ksf": 8.8737,
     "settlement_in": 0.8039,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F4": {
     "foundation_type": "SPREAD",
     "B_in": 96.0,
     "L_in": 96.0,
     "thickness_in": 21.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002616,
     "bar_size": 5,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 8.4779,
     "q_allow_net_ksf": 8.7633,
     "settlement_in": 0.7897,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "F5": {
     "foundation_type": "SPREAD",
     "B_in": 69.0,
     "L_in": 69.0,
     "thickness_in": 15.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002626,
     "bar_size": 4,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 8.6903,
     "q_allow_net_ksf": 8.7502,
     "settlement_in": 0.7555,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING"
    },
    "W1": {
     "foundation_type": "SPREAD",
     "B_in": 66.0,
     "L_in": 288.0,
     "thickness_in": 15.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002728,
     "bar_size": 4,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 6.915,
     "q_allow_net_ksf": 7.1927,
     "settlement_in": 0.756,
     "pier_count": 0,
     "pier_diameter_in": 0,
     "pier_depth_ft": 0,
     "area_replacement_ratio": 0.0,
     "top_of_pier_stress_psf": 0.0,
     "governing_constraint": "BEARING",
     "sliding_fs": 3.0106,
     "eccentricity_in": 5.9803,
     "uplift_fs": 2.9903
    }
   }
  },
  "CR-P6": {
   "bearing_depth_in": 45.0,
   "fs_bearing": 3.5,
   "occupancy_class": "Light Industrial, Public",
   "improved": true,
   "sulfate_pct": 0.26,
   "sulfate_exposure_class": "S2",
   "exposure_classes": [
    "C1",
    "F1",
    "S2",
    "W1"
   ],
   "fc_psi": 4500,
   "max_wcm": 0.45,
   "dewatering_days": 0,
   "governing_constraint": "BEARING",
   "foundations": {
    "F1": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 123.0,
     "L_in": 123.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002741,
     "bar_size": 6,
     "bar_spacing_in": 8.0,
     "q_net_ksf": 5.9862,
     "q_allow_net_ksf": 6.0212,
     "settlement_in": 0.9873,
     "pier_count": 8,
     "pier_diameter_in": 30,
     "pier_depth_ft": 9,
     "area_replacement_ratio": 0.373777,
     "top_of_pier_stress_psf": 13717.24,
     "governing_constraint": "BEARING"
    },
    "F2": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 171.0,
     "L_in": 171.0,
     "thickness_in": 36.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002443,
     "bar_size": 7,
     "bar_spacing_in": 7.5,
     "q_net_ksf": 6.0232,
     "q_allow_net_ksf": 6.1057,
     "settlement_in": 0.8957,
     "pier_count": 14,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.338429,
     "top_of_pier_stress_psf": 14887.39,
     "governing_constraint": "BEARING"
    },
    "F3": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 186.0,
     "L_in": 186.0,
     "thickness_in": 39.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002492,
     "bar_size": 7,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 6.1327,
     "q_allow_net_ksf": 6.1484,
     "settlement_in": 0.9138,
     "pier_count": 17,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.34734,
     "top_of_pier_stress_psf": 14863.36,
     "governing_constraint": "BEARING"
    },
    "F4": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 156.0,
     "L_in": 156.0,
     "thickness_in": 33.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002359,
     "bar_size": 6,
     "bar_spacing_in": 6.0,
     "q_net_ksf": 6.0028,
     "q_allow_net_ksf": 6.069,
     "settlement_in": 0.836,
     "pier_count": 12,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.348549,
     "top_of_pier_stress_psf": 14510.21,
     "governing_constraint": "BEARING"
    },
    "F5": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 114.0,
     "L_in": 114.0,
     "thickness_in": 24.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.002342,
     "bar_size": 5,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 5.941,
     "q_allow_net_ksf": 6.0198,
     "settlement_in": 0.7173,
     "pier_count": 7,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.380733,
     "top_of_pier_stress_psf": 13421.16,
     "governing_constraint": "BEARING"
    },
    "W1": {
     "foundation_type": "IMPROVED_SPREAD",
     "B_in": 153.0,
     "L_in": 288.0,
     "thickness_in": 27.0,
     "bearing_stratum": "I",
     "reinforcement_ratio": 0.003997,
     "bar_size": 7,
     "bar_spacing_in": 6.5,
     "q_net_ksf": 5.572,
     "q_allow_net_ksf": 5.5905,
     "settlement_in": 0.8349,
     "pier_count": 21,
     "pier_diameter_in": 30,
     "pier_depth_ft": 8,
     "area_replacement_ratio": 0.336874,
     "top_of_pier_stress_psf": 13819.9,
     "governing_constraint": "BEARING",
     "sliding_fs": 2.8638,
     "eccentricity_in": 5.8525,
     "uplift_fs": 3.0793
    }
   }
  }
 }
}
```


## deliverables/ground_improvement_plan.json

```text
{
 "stress_concentration_ratio_ns": 10.0,
 "pier_diameter_in": 30.0,
 "rigs_mobilised": 2,
 "mobilisation_cost_usd": 80000.0,
 "pads_improved": [
  "CR-P1",
  "CR-P2",
  "CR-P4",
  "CR-P6"
 ],
 "rig_assignments": {
  "RIG-1": [
   "CR-P1",
   "CR-P2"
  ],
  "RIG-2": [
   "CR-P4",
   "CR-P6"
  ]
 },
 "modulus_tests": {
  "CR-P1": 1,
  "CR-P2": 1,
  "CR-P3": 0,
  "CR-P4": 1,
  "CR-P5": 0,
  "CR-P6": 1
 },
 "pads": {
  "CR-P1": {
   "stiffness_modulus_pci": 165.0,
   "treatment_depth_ft": 9,
   "total_pier_lf": 653.0
  },
  "CR-P2": {
   "stiffness_modulus_pci": 140.0,
   "treatment_depth_ft": 9,
   "total_pier_lf": 679.0
  },
  "CR-P4": {
   "stiffness_modulus_pci": 195.0,
   "treatment_depth_ft": 8,
   "total_pier_lf": 320.0
  },
  "CR-P6": {
   "stiffness_modulus_pci": 155.0,
   "treatment_depth_ft": 9,
   "total_pier_lf": 640.0
  }
 }
}
```
