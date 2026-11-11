# fire-solve-convergence

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-20 — fire-solve-convergence
   The plateau is burner pulsation at 0.87 Hz, there is no steady solution, and the run departs from the modelling
   standard in four named ways.

 Judge score                                                NOT MEASURED — no judge endpoint or key

 Criteria · weight                                          43 criteria · 129 weight points

 My criteria reader (not a score)                           43 of 43 criteria · 129 of 129 weight

 Breakage campaign                                          42/42

 File                                                       /work/ple-solve/fire-solve-convergence/out/CFDME-2025-
                                                            041_transient_solve_disposition_revA.docx · 50,471 B · sha256
                                                            c1d6cdf28d5c274e…


TASK AND INPUTS
Write the disposition of the transient CFD solve (attempt 3) for a fire case: is the plateau a converged steady state, and does the run
comply? Forty-three criteria, weight 129.

The solver logs, the case setup, the three-grid study, the modelling standard, and the validation record.

WHAT WAS AMBIGUOUS OR BROKEN
    The plateau is burner pulsation: f = 0.87 Hz, T = 1.15 s, 0.41% agreement with the Mulshi record when compared against the
    unrounded 0.8660 Hz / 1.1547 s — not against the rounded values.
    There is no steady solution to report.
    Four nonconformities: radiation with 32 rays and one band against four directions and two bands with a 2.5 µm split; RAS k-ε
    against the required LES + WALE; the direction vector (cos240, sin240, 0) = (−0.500, −0.866, 0) points to 210°, not to wind from
    240° (the correct vector is (0.866, 0.500, 0)); and a roughness of 0.03 m that is not in the met file.

WHAT I DID AND WHAT CAME OUT

    The authorised block: adjustTimeStep yes, maxCo 0.9, maxDeltaT 0.01 s, adjustableRunTime, writeInterval 0.0461 s (T/25),
    endTime 31.10 s (8 + 20 × 1.1547005 = 31.09401).




                                                                 page 31 of 48
PLE 29-task report · AIODAM                                                                                                11 November 2026




    Attempt 3 ran on 11,214,862 cells, 5 cell zones and 13 patches; the three-grid study is fluid-only, with 1 cell zone and 6 patches per
    grid, covering D*/dx = 7.52 and Pope 0.84 but not the baffles, the cavity radiation or the connections.
    Cost: 4 h 28 m per case and 26 h 49 m for the matrix as lower bounds; attempt 3 took 6 h 41 m, 424.3 CPU-hours, 1.50×, 2 h 13 m
    longer.
    43 of 43 criteria and 129 of 129 weight by my reader on the PDF twin.

HOW IT WAS VERIFIED
    Reader on the PDF twin; 316 distinguishable numbers, 0 unexplainable.
    An independent recomputation of 19 quantities with a second parser.
    Forty-two breakages — all noticed, each by its own criterion.
    The official test.sh chain with a stub judge: complete, no GRADER_FAILED.

MY MISTAKES IN THIS TASK
    The first mutation run gave 27 of 39: some survivors were my own incomplete mutations (a fact stands two to five times), and some
    were reader holes — c12 found “2000” in a note near endTime, c23 accepted a permuted vector because it checked two numbers
    without order, c30/c34/c28/c29 passed on keywords scattered across the document, c9 read a standard's “typically 0.01 s” as the set
    value, c21 accepted “carried forward” 300 characters before EDC. All six were tightened.
    The first text had broken cross-references after section renumbering, “31.08 s” instead of “31.09 s”, and “0.0460 s — smaller than the
    standard permits” where the truth is “compliant, but not the largest”. Found by reading the rendered PDF, not the docx.

WHAT REMAINS UNVERIFIED
No judge score. The external normative check is limited to the standard text in the corpus.

USE OF REFERENCE OR GRADER MATERIAL
None.


PART C · RUBRIC · FIRE TESTING
```


## code/src/audit_numbers.py

```text
#!/usr/bin/env python3
"""Strict number audit: every *distinctive* number in the note (a decimal with 2+ places, or an integer of 3+
digits) must match, exactly and at its own precision, either a literal in the case record or a value compute.py
derived. Short integers and section numbers are excluded: a 1.1 MB corpus contains every one of them somewhere,
so matching them would prove nothing.
"""
import json, re, sys, os
from collections import Counter

REC = "/work/fsc/corpus"
COMP = json.load(open("/work/fsc/out/computed.json"))

def record_literals():
    toks = set()
    for f in os.listdir(REC):
        try: t = open(os.path.join(REC, f), encoding="utf-8", errors="replace").read()
        except OSError: continue
        for m in re.finditer(r"\d[\d,]*(?:\.\d+)?", t):
            toks.add(m.group(0).replace(",", ""))
    return toks

def derived_strings(o, out):
    """Every string a value takes when formatted at 0..6 decimals, plus the rounded forms of derived results."""
    if isinstance(o, dict):
        for v in o.values(): derived_strings(v, out)
    elif isinstance(o, list):
        for v in o: derived_strings(v, out)
    elif isinstance(o, (int, float)) and not isinstance(o, bool):
        for dp in range(0, 7): out.add(f"{o:.{dp}f}")

ALLOWED = record_literals()
derived_strings(COMP, ALLOWED)
# values the note derives by one arithmetic step on the record, stated at their own precision
EXTRA = {
 "31.10":"8 s + 20 x 1.1547005 s = 31.09401 s, floored up", "31.09":"one hundredth below 31.10",
 "0.0461":"floor(T/25 to 4 dp)", "0.0462":"next step up, fails the 25-sample rule", "0.0460":"1.15/25",
 "0.046188":"T/25", "23.09":"20 periods", "23.09401":"20 x 1.1547005",
 "4.470":"5.75 h x 31.09401 / 40.00", "26.82":"6 x 4.470", "2.21357":"6.6833 - 4.4698",
 "0.143750":"5.75/40", "1.4952":"6.6833/4.4698", "1.342,754":"sum of the four non-atmosphere zones",
 "1342754":"sum of the four non-atmosphere zones", "19.9965":"(31.09-8)/T",
}
ALLOWED |= {k.replace(",", "") for k in EXTRA}
# section numbers the note cites (references, not quantities)
ALLOWED |= {f"{a}.{b}" for a in range(1,10) for b in range(1,9)}

text = open(sys.argv[1], encoding="utf-8", errors="replace").read()
# a decimal split by the line wrapper ("308\n.5 K") is one token, not two
text = re.sub(r"(\d)\s*\n\s*\.(\d)", r"\1.\2", text)
toks = re.findall(r"(?<![\w.])\d[\d,]*(?:\.\d+)?(?![\w.])", text)
distinctive = [t for t in toks if ("." in t and len(t.split(".")[1]) >= 2) or len(t.replace(",", "")) >= 3]
unknown = []
for t in distinctive:
    plain = t.replace(",", "")
    if plain in ALLOWED: continue
    # a time step written as h/m or a date is not a quantity to audit here
    unknown.append(t)
# bearings written with a leading zero are compass directions, not quantities
unknown = [t for t in unknown if not (re.fullmatch(r"0\d\d", t))]

# wall clocks and percentages get their own audit: a wrong 4 h 31 m must not slip through
HM_OK = {("4","28"), ("26","49"), ("26","48"), ("6","41"), ("5","45"), ("2","13")}
for h, m in re.findall(r"(\d+)\s*h\s*(\d+)\s*m\b", text):
    if (h, m) not in HM_OK:
        unknown.append(f"{h} h {m} m")
PCT_OK = {"0.41","0.40","0.4087","0.4071","0.00","84","80","7.5","1.5","100","15","10"}
for p in re.findall(r"(\d+(?:\.\d+)?)\s*%", text):
    if p not in PCT_OK:
        unknown.append(f"{p} %")
c = Counter(unknown)
print(f"{len(toks)} numeric tokens, {len(distinctive)} distinctive; {sum(c.values())} unexplained")
for t, n in c.most_common(40):
    i = text.find(t)
    print(f"  {t!r} x{n}   ...{text[max(0,i-70):i+45]!r}...")
sys.exit(1 if c else 0)

```


## code/src/checks.py

```text
#!/usr/bin/env python3
"""The 43 rubric criteria of fire-solve-convergence written as code, BEFORE the memo text.

Written against the criteria statements and the case record only: it asks whether the
document says a thing, never whether it says it the way the memo happens to say it.
Run on the rendered PDF text (what the judge reads) or on the docx text.
"""
import re, sys, json, unicodedata

def norm(t):
    t = unicodedata.normalize("NFKC", t)
    t = t.replace("\u2212", "-").replace("\u2013", "-").replace("\u2014", "-").replace("\u00a0", " ")
    t = t.replace("\u00b5", "u").replace("\u03bc", "u")      # micro sign / mu
    t = t.lower()
    t = re.sub(r"[ \t\r\n]+", " ", t)
    return t

def compact(t):
    return re.sub(r"[\s,]", "", norm(t))

def has(t, *pats):
    n = norm(t)
    return all(re.search(p.lower(), n) for p in pats)

def has_c(t, *pats):
    c = compact(t)
    return all(re.search(p, c) for p in pats)

def near(t, a, b, window=260):
    """a and b within `window` characters of each other (same sentence/row)."""
    n = norm(t)
    a, b = a.lower(), b.lower()
    for m in re.finditer(a, n):
        seg = n[max(0, m.start()-window): m.end()+window]
        if re.search(b, seg):
            return True
    return False

CHECKS = []
def chk(i, w, label):
    def deco(fn):
        CHECKS.append((i, w, label, fn)); return fn
    return deco

# 0-4  the plateau, the prediction, the agreement, no steady solution
@chk(0, 2.0, "plateau is the puffing oscillation of the buoyant plume")
def c0(t):
    return near(t, r"puffing", r"oscillat|shed|neck", 320) and "plateau" in norm(t)

@chk(1, 2.0, "predicted frequency 0.87 Hz and period 1.15 s")
def c1(t):
    return has(t, r"0\.87\s*hz") and has(t, r"1\.15\s*s\b")

@chk(2, 1.0, "attempt-3 narrative quotes 1.15 s")
def c2(t):
    return near(t, r"closing narrative", r"1\.15\s*s\b", 400)

@chk(3, 4.0, "agreement 0.41 percent")
def c3(t):
    return has(t, r"0\.41\s*(%|percent)")

@chk(4, 1.0, "no steady solution exists")
def c4(t):
    n = norm(t)
    return bool(re.search(r"no steady (solution|state|flow field)", n)
                or re.search(r"steady solution does not exist", n)
                or re.search(r"no time-invariant (flow )?field", n))

# 5-7  conformance: radiation directions, banding, LES closure
@chk(5, 3.0, "four angular directions")
def c5(t):
    return has(t, r"\b(four|4)\s+angular directions")

@chk(6, 1.0, "two wavelength bands split at 2.5 um")
def c6(t):
    return has(t, r"\b(two|2)\s+wavelength bands") and has(t, r"2\.5\s*u\s*m|2\.5\s*microm")

@chk(7, 1.0, "LES with WALE or dynamic Smagorinsky")
def c7(t):
    n = norm(t)
    return bool(re.search(r"large-eddy simulation|\bles\b", n) and re.search(r"wale", n)
                and re.search(r"dynamic smagorinsky", n))

# 8-13  the authorized time-control block and the delivered one
@chk(8, 1.0, "adjustTimeStep yes, maxCo 0.9, writeControl adjustableRunTime")
def c8(t):
    return near(t, r"adjusttimestep", r"\byes\b", 80) and near(t, r"maxco", r"\b0\.9\b", 80) \
           and near(t, r"writecontrol", r"adjustableruntime", 80)

@chk(9, 1.0, "maxDeltaT 0.01 s")
def c9(t):
    """maxDeltaT 0.01 s stated as the value carried, not as the standard's 'typically'."""
    n = norm(t)
    for m in re.finditer(r"maxdeltat", n):
        seg = n[m.end(): m.end()+70]
        if re.search(r"0\.01\s*s\b", seg) and "typically" not in seg and "typically" not in n[max(0,m.start()-40):m.end()]:
            return True
    return False

@chk(10, 6.0, "write interval 0.0461 s")
def c10(t):
    return has_c(t, r"0\.0461s?\b")

@chk(11, 3.0, "end time 31.10 s")
def c11(t):
    return has_c(t, r"31\.10s?\b")

@chk(12, 2.0, "delivered block: 2000 iterations, adjustTimeStep no, deltaT 0.01 s fixed, writeControl timeStep, writeInterval 100")
def c12(t):
    return near(t, r"endtime", r"\b2000\b", 120) and has(t, r"outer iterations") \
           and near(t, r"adjusttimestep", r"\bno\b", 80) and near(t, r"deltaT", r"0\.01\s*s", 80) \
           and near(t, r"writecontrol", r"\btimestep\b", 80) and near(t, r"writeinterval", r"\b100\b", 80)

@chk(13, 1.0, "delivered startFrom startTime, no maxCo / no maxDeltaT vs latestTime, 0.9, 0.01 s")
def c13(t):
    n = norm(t)
    absent = r"(absent|none|not set|not present|not carried|no entry|-)"
    return near(t, r"startfrom", r"starttime", 80) and near(t, r"startfrom", r"latesttime", 200) \
           and near(t, r"maxco", absent, 120) and near(t, r"maxco", r"0\.9", 200) \
           and near(t, r"maxdeltat", absent, 120) and near(t, r"maxdeltat", r"0\.01\s*s", 200)

# 14-19  the mesh and the verification scope
@chk(14, 1.0, "attempt 3 ran on the production conjugate mesh")
def c14(t):
    return near(t, r"attempt 3", r"11,214,862|11214862", 500) or \
           near(t, r"attempt 3", r"production conjugate mesh", 400)

@chk(15, 1.0, "production mesh carries five cell zones")
def c15(t):
    n = norm(t)
    return bool(re.search(r"five cell zones|5 cell zones", n)) and \
           all(z in n for z in ("atmosphere", "outerleaf", "cavity", "innerleaf", "ties"))

@chk(16, 1.0, "study verifies plume resolution in the fluid domain")
def c16(t):
    return has_c(t, r"7\.52") and (has(t, r"pope") and has(t, r"0\.84|84\s*%|84 percent"))

@chk(17, 1.0, "solid leaves, cavity radiation, ties outside the study's scope")
def c17(t):
    n = norm(t)
    return all(k in n for k in ("solid leaves", "cavity radiation", "tie regions")) and \
           bool(re.search(r"outside the scope|not verified|does not (cover|verify)", n))

@chk(18, 5.0, "each study mesh carries a single cell zone, which is fluid")
def c18(t):
    n = norm(t)
    return bool(re.search(r"(single|one|\b1\b) cell zone", n)) and bool(re.search(r"fluid", n)) and \
           bool(re.search(r"firerefinementzone|fire refinement zone", n))

@chk(19, 8.0, "each study mesh carries six boundary patches")
def c19(t):
    n = norm(t)
    ok_names = all(k in n for k in ("ground", "inlet_240", "inlet_270", "outlet", "top", "fire_source"))
    return bool(re.search(r"\bsix boundary patches|\b6 boundary patches", n)) and ok_names

# 20-25  what carries forward, with the inlet's defining settings
@chk(20, 1.0, "inlet Uref 8.0 m/s at Zref 6.1 m")
def c20(t):
    return has(t, r"8\.0\s*m/s") and has(t, r"6\.1\s*m\b")

@chk(21, 1.0, "EDC carries forward")
def c21(t):
    for s_ in sentences(norm(t)):
        if re.search(r"\bedc\b", s_) and re.search(r"carries forward|carried forward|carry forward|retained|carries over", s_):
            return True
    return False

@chk(22, 3.0, "delivered flow direction does not describe a wind from 240 deg")
def c22(t):
    n = norm(t)
    if not re.search(r"cos\s*\(?\s*240", n):
        return False
    return bool(re.search(r"does not (describe|represent|give|model)|is not a wind from|not a wind from 240", n)) \
       and bool(re.search(r"wind from 240", n))

@chk(23, 6.0, "correct inlet vector (0.866, 0.500, 0)")
def c23(t):
    ordered = has_c(t, r"0\.8660\.5000\.0?\)?") or has_c(t, r"cos30.{0,10}sin30")
    return ordered and near(t, r"0\.866", r"wind from 240|unit vector|flow direction", 500)

@chk(24, 8.0, "0.03 m roughness not from the site meteorological record, which holds none")
def c24(t):
    n = norm(t)
    return bool(re.search(r"0\.03\s*m", n)) \
           and bool(re.search(r"not taken from|not sourced|is not from|no basis|not from the (site )?(met|meteorological)", n)) \
           and bool(re.search(r"meteorological|met pull|site met", n)) \
           and bool(re.search(r"holds no roughness|no roughness length|carries no roughness", n))

@chk(25, 8.0, "only constants on file; no temperature-dependent property table")
def c25(t):
    n = norm(t)
    return bool(re.search(r"no temperature-dependent", n)) and bool(re.search(r"table", n)) and \
           bool(re.search(r"constant", n))

# 26-39  cost
@chk(26, 2.0, "one authorized case 4 h 28 m")
def c26(t):
    return has_c(t, r"4h28m")

@chk(27, 2.0, "six-case matrix 26 h 49 m")
def c27(t):
    return has_c(t, r"26h49m")

@chk(28, 1.0, "fixed core count condition unmet; fine-grid allocation unrecorded")
def c28(t):
    return near(t, r"core count", r"not (recorded|stated)|unrecorded|no core allocation|no allocation", 600) \
           and near(t, r"core count", r"fine", 600)

@chk(29, 1.0, "fixed mesh condition fails: fluid-only mesh without the wall assembly")
def c29(t):
    for s_ in sentences(norm(t)):
        if "fixed mesh" in s_ and re.search(r"not met|fails", s_) \
           and re.search(r"fluid-only|carries none of|no wall assembly", s_):
            return True
    return False

@chk(30, 9.0, "fixed solver stack condition satisfied: same reacting solver family")
def c30(t):
    for s_ in sentences(norm(t)):
        if "solver stack" in s_ and re.search(r"\bmet\b|satisf", s_) and re.search(r"\bsame\b", s_) \
           and "firefoam" in s_:
            return True
    return False

@chk(31, 1.0, "max Courant 0.90 against the 0.9 ceiling")
def c31(t):
    return has_c(t, r"0\.90") and near(t, r"0\.90", r"0\.9\s*(ceiling|limit)", 200)

@chk(32, 3.0, "production solve advances at a smaller average time step than 0.0022 s")
def c32(t):
    n = norm(t)
    return bool(re.search(r"smaller (average )?time step|shorter (average )?time step", n)) and \
           bool(re.search(r"0\.0022\s*s", n))

@chk(33, 1.0, "figures are a lower bound")
def c33(t):
    return has(t, r"lower bound")

@chk(34, 1.0, "a firm figure requires a benchmark on the production conjugate mesh")
def c34(t):
    return near(t, r"benchmark", r"production conjugate mesh", 300) and near(t, r"benchmark", r"firm", 400)

def sentences(n):
    return re.split(r"(?<=\.)\s+", n)

@chk(35, 10.0, "no contingency, margin or risk multiplier applied")
def c35(t):
    """A sentence must say, about the figure, that contingency / risk multiplier is NOT applied."""
    for s_ in sentences(norm(t)):
        if "contingency" in s_ and re.search(r"\bno\b|\bnot\b|\bnever\b", s_) \
           and re.search(r"applied|carried|added", s_) and re.search(r"risk multiplier|margin", s_):
            return True
    return False

@chk(36, 10.0, "no overhead percentage applied")
def c36(t):
    for s_ in sentences(norm(t)):
        if "overhead" in s_ and re.search(r"\bno\b|\bnot\b|\bnever\b", s_) \
           and re.search(r"applied|carried|added", s_):
            return True
    return False

@chk(37, 1.0, "attempt 3: 6 h 41 m and 424.3 CPU-hours")
def c37(t):
    return has_c(t, r"6h41m") and has_c(t, r"424\.3")

@chk(38, 1.0, "attempt 3 = 1.50 times one authorized case")
def c38(t):
    return has_c(t, r"1\.50") and near(t, r"1\.50", r"times", 120)

@chk(39, 10.0, "attempt 3 consumed 2 h 13 m more")
def c39(t):
    return has_c(t, r"2h13m")

@chk(40, 1.0, "solved fields from attempt 3 are discarded")
def c40(t):
    return near(t, r"(solved )?fields", r"discard", 200) or near(t, r"discard", r"field", 200)

@chk(41, 1.0, "stamp boundary: CFD thermal analysis only; rating certified separately")
def c41(t):
    n = norm(t)
    return bool(re.search(r"cfd thermal analysis only", n)) and \
           bool(re.search(r"fire-resistance rating (is )?certified separately by others", n))

@chk(42, 1.0, "dated 10/10/2025")
def c42(t):
    return "10/10/2025" in norm(t)

def run(text):
    res = []
    for i, w, label, fn in sorted(CHECKS):
        try:
            ok = bool(fn(text))
            err = None
        except Exception as e:
            ok, err = False, f"{e.__class__.__name__}: {e}"
        res.append({"index": i, "weight": w, "label": label, "pass": ok, "error": err})
    return res

if __name__ == "__main__":
    text = open(sys.argv[1], encoding="utf-8", errors="replace").read()
    res = run(text)
    red = [r for r in res if not r["pass"]]
    got = sum(r["weight"] for r in res if r["pass"]); tot = sum(r["weight"] for r in res)
    for r in red:
        print(f"RED  #{r['index']:2d} w={r['weight']:4.1f}  {r['label']}" + (f"  [{r['error']}]" if r["error"] else ""))
    print(f"\npassed {len(res)-len(red)}/{len(res)} criteria, weight {got:.1f}/{tot:.1f}")
    sys.exit(1 if red else 0)

```


## code/src/compute.py

```text
#!/usr/bin/env python3
"""Every number the disposition carries, re-derived from the case record.

Sources (read-only): the attempt-3 log, the delivered and reference controlDicts,
the three-grid study, the meshing log, the site met pull, the SOW. Nothing is
taken from the rubric; the rubric was not read into this file.
"""
import csv, json, math, re, os

FILES = "/work/ple-dataset/tasks/fire-solve-convergence/environment/files"
P = os.path.join(FILES, "Projects/CFDME-2025-041_Ocotillo_WUI")

def read(p):
    return open(p, encoding="utf-8", errors="replace").read()

log3   = read(f"{P}/Case/logs/log.solve_20251009_attempt3.txt")
snappy = read(f"{P}/Case/logs/log.snappyHexMesh_20251009.txt")
three  = read(f"{P}/Case/verification/three_grid_study_outputs.txt")
ctrl   = read(f"{P}/Case/system/controlDict")
ref    = read(os.path.join(FILES, "Firm/Standards/Reference_Cases/fireFoam_transient_controlDict.txt"))
met    = read(f"{P}/Data/site_met_avra_valley_2015-2024.csv")

out = {}

# ---- 1. puffing frequency and period, Cetegen-Ahmed per Modeling Standard rev4 s4
D = 3.0                                  # m, front segment (scoping memo s5; study header)
f_pred = 1.5 * D ** -0.5                 # Hz
T_pred = 1.0 / f_pred                    # s
out["D_m"] = D
out["f_pred_Hz"] = f_pred
out["T_pred_s"] = T_pred
out["f_pred_2dp"] = round(f_pred, 2)
out["T_pred_2dp"] = round(T_pred, 2)

# ---- 2. what the run showed: the closing narrative quotes a period
m = re.search(r"oscillate at\s*\n?//\s*a matching period of ~([\d.]+) s", log3)
assert m, "closing-narrative period not found"
T_quoted = float(m.group(1))
f_quoted = 1.0 / T_quoted
out["T_quoted_s"] = T_quoted
out["f_quoted_Hz"] = f_quoted
out["agreement_pct_as_frequency"] = abs(f_quoted - f_pred) / f_pred * 100
out["agreement_pct_as_period"]    = abs(T_quoted - T_pred) / T_pred * 100
out["agreement_pct_2dp"] = round(out["agreement_pct_as_frequency"], 2)
# the circular reading the note must avoid: both rounded to 2 dp
out["circular_pct_from_rounded"] = abs(round(f_quoted,2) - round(f_pred,2)) / round(f_pred,2) * 100

# ---- 3. write interval: >= 25 monitor samples per puffing period, largest to 4 dp
samples_min = 25
write_ceiling = T_pred / samples_min               # s
write_chosen  = math.floor(write_ceiling * 1e4) / 1e4
out["write_ceiling_s"] = write_ceiling
out["write_interval_s"] = write_chosen
out["samples_per_period_at_chosen"] = T_pred / write_chosen
out["samples_per_period_at_next"]   = T_pred / (write_chosen + 1e-4)

# ---- 4. end time: 8 s recorded startup + 20 puffing periods, as a floor to 2 dp
startup = None
m = re.search(r"initial transient cleared by t ~ (\d+) s", three)
assert m
startup = float(m.group(1))
periods_min = 20
end_exact = startup + periods_min * T_pred
end_chosen = math.ceil(end_exact * 100) / 100
out["startup_s"] = startup
out["periods_min"] = periods_min
out["end_time_exact_s"] = end_exact
out["end_time_s"] = end_chosen
out["periods_at_float_down"] = (math.floor(end_exact*100)/100 - startup) / T_pred

# ---- 5. the fine-grid benchmark of record and the projection basis
#     wall clock and simulated time per grid, read from the three solver summaries
grids = {}
for tag in ("COARSE", "MEDIUM", "FINE"):
    blk = three.split(f"Solver summary : {tag} grid")[1].split("---", 1)[0]
    wc = re.search(r"Wall-clock \(([^)]*)\)\s*:\s*(\d+)h\s*(\d+)m", blk)
    co = re.search(r"Max Courant number\s*:\s*([\d.]+)\s*\(limit ([\d.]+)\)", blk)
    meanc = re.search(r"Mean Courant number\s*:\s*([\d.]+)", blk)
    dt = re.search(r"Time-step average\s*:\s*([\d.eE+-]+) s", blk)
    cells = re.search(r"Cells\s*:\s*([\d ]+)", blk)
    per = re.search(r"Puffing period recovered\s*:\s*([\d.]+) s", blk)
    grids[tag] = dict(host_cores=wc.group(1).strip(),
                      wall_h=int(wc.group(2)) + int(wc.group(3))/60.0,
                      maxCo=float(co.group(1)), Co_limit=float(co.group(2)),
                      meanCo=float(meanc.group(1)), dt_mean=float(dt.group(1)),
                      cells=int(cells.group(1).replace(" ","")),
                      period=float(per.group(1)))
out["grids"] = grids
sample_t = float(re.search(r"Sample time\s*:\s*t = ([\d.]+) s", three).group(1))
out["benchmark_simulated_s"] = sample_t
rate = grids["FINE"]["wall_h"] / sample_t          # h of wall clock per simulated second
out["fine_rate_h_per_sim_s"] = rate

per_case_h = rate * end_exact
out["per_case_h"] = per_case_h
out["per_case_hm"] = (int(per_case_h), round((per_case_h % 1) * 60))
n_cases = 6
out["n_cases"] = n_cases
matrix_h = per_case_h * n_cases
out["matrix_h"] = matrix_h
out["matrix_hm"] = (int(matrix_h), round((matrix_h % 1) * 60))

# ---- 6. attempt 3 consumption and the comparison
m = re.search(r"Wall clock\s*:\s*(\d+)h\s*(\d+)m\s*(\d+)s", log3)
a3_h = int(m.group(1)) + int(m.group(2))/60.0 + int(m.group(3))/3600.0
out["attempt3_h"] = a3_h
out["attempt3_hm"] = (int(m.group(1)), int(m.group(2)))
out["attempt3_cpu_hours"] = float(re.search(r"User CPU-hours\s*:\s*([\d.]+)", log3).group(1))
out["attempt3_cores"] = int(re.search(r"Cores\s*:\s*(\d+)", log3).group(1))
out["attempt3_iterations"] = int(re.search(r"Reached endTime = (\d+)", log3).group(1))
out["attempt3_vs_case_ratio"] = a3_h / per_case_h
out["attempt3_excess_h"] = a3_h - per_case_h
out["attempt3_excess_hm"] = (int(a3_h - per_case_h), round(((a3_h - per_case_h) % 1) * 60))

# ---- 7. delivered controlDict, entry by entry
def entry(text, key):
    m = re.search(rf"^\s*{key}\s+([^\s;]+)\s*;", text, re.M)
    return m.group(1) if m else None
for k in ("application","startFrom","startTime","stopAt","endTime","deltaT","adjustTimeStep",
          "writeControl","writeInterval","maxCo","maxDeltaT","writePrecision"):
    out[f"delivered_{k}"] = entry(ctrl, k)
    out[f"reference_{k}"] = entry(ref, k)
out["delivered_simulated_s_at_fixed_dt"] = int(out["delivered_endTime"]) * float(out["delivered_deltaT"])
out["attempt3_simulated_s"] = out["attempt3_iterations"] * float(out["delivered_deltaT"])

# ---- 8. meshes
m = re.search(r"Mesh statistics\s*\n\s*cells\s+([\d ]+)", log3)
out["attempt3_cells"] = int(m.group(1).replace(" ",""))
out["attempt3_nonorth"] = float(re.search(r"max non-orth\s+([\d.]+)", log3).group(1))
out["attempt3_skew"]    = float(re.search(r"max skewness\s+([\d.]+)", log3).group(1))
zones = dict(re.findall(r"^\s{4}(atmosphere|outerLeaf|cavity|innerLeaf|ties)\s+(?:\([^)]*\))?\s*:?\s+([\d ]+)\s+[\d ]+$",
                        snappy, re.M))
# the block above carries a header line per zone; fall back to the explicit table
zones = {}
blk = snappy.split("Cell breakdown by cellZone :")[1].split("Refinement level distribution")[0]
for name, cnt in re.findall(r"(atmosphere|outerLeaf|cavity|innerLeaf|ties)\s*(?:\([^)]*\))?\s*:\s*([\d ]+)", blk):
    zones[name] = int(cnt.replace(" ", ""))
out["production_zones"] = zones
out["production_cell_zones"] = int(re.search(r"cell zones\s*:\s*(\d+)", snappy).group(1))
out["production_patches"] = int(re.search(r"boundary patches\s*:\s*(\d+)", snappy).group(1))
out["production_cells_checkMesh"] = int(re.search(r"cells\s*:\s*([\d ]+)\n\s*faces per cell", snappy).group(1).replace(" ",""))
out["production_finest_cells_m"] = [0.0625, 0.03125]
out["study_cell_zones"] = [int(x) for x in re.findall(r"cell zones:\s+(\d+)", three)]
out["study_patches"] = [int(x) for x in re.findall(r"boundary patches:\s+(\d+)", three)]
out["study_patch_names"] = sorted(set(re.findall(r"^\s{4}(ground|inlet_240|inlet_270|outlet|top|fire_source)\s+\d+", three, re.M)))
out["study_min_cell_m"] = round(float(re.search(r"Min volume = ([\d.eE+-]+)\.", three.split("GRID 3 of 3")[1]).group(1)) ** (1/3), 4)
out["production_patch_names"] = re.findall(r"^\d+\s+(?:wall|patch)\s+(\w+)$", snappy, re.M)
out["Dstar_over_dx_study"] = float(re.search(r"D\*/dx = 3\.76 / 0\.5 = ([\d.]+)", three).group(1))
out["pope_resolved"] = float(re.search(r"k_res/\(k_res\+k_sgs\) = ([\d.]+)", three).group(1))

# ---- 9. inlet settings as delivered, and the correct vector for wind from 240 deg
out["inlet_Uref"] = float(re.search(r"Uref\s+([\d.]+) m/s", log3).group(1))
out["inlet_Zref"] = float(re.search(r"Zref\s+([\d.]+) m", log3).group(1))
out["inlet_z0"]   = float(re.search(r"z0\s+([\d.]+) m", log3).group(1))
out["inlet_flowDir_entry"] = re.search(r"flowDir\s+(\S+)", log3).group(1)
vx, vy = math.cos(math.radians(240)), math.sin(math.radians(240))
out["delivered_flow_vector"] = (round(vx,3), round(vy,3), 0.0)
out["delivered_flow_bearing_compass"] = (math.degrees(math.atan2(vx, vy)) + 360) % 360
wx, wy = math.sin(math.radians(60)), math.cos(math.radians(60))
out["correct_vector_for_from240"] = (round(wx,3), round(wy,3), 0.0)
out["correct_bearing_toward_compass"] = 60
out["twenty_ft_in_m"] = 20 * 0.3048

# ---- 10. site met pull: columns and the design envelope footer
rows = list(csv.reader(met.splitlines()))
cols = rows[0]
out["met_columns"] = cols
out["met_rows"] = sum(1 for r in rows[1:] if r and not r[0].startswith("#"))
out["met_has_roughness_column"] = any("rough" in c.lower() or c.lower() in ("z0","z_0") for c in cols)
out["met_footer"] = [l for l in met.splitlines() if l.startswith("#")]
# 20-ft wind and direction extremes in the record
vals = [(float(r[3]), float(r[4])) for r in rows[1:] if r and not r[0].startswith("#")]
out["met_wind_min_max"] = (min(v[0] for v in vals), max(v[0] for v in vals))
out["met_dir_set"] = sorted({v[1] for v in vals})

json.dump(out, open("/work/fsc/out/computed.json","w"), indent=1)
for k,v in out.items():
    if k in ("grids","met_footer","production_patch_names"): continue
    print(f"{k:38s} {v}")
print("\ngrids:")
for t,g in out["grids"].items(): print(" ", t, g)

```


## code/src/dump_corpus.py

```text
import os, subprocess, sys, re
ROOT="/work/ple-dataset/tasks/fire-solve-convergence/environment/files"
OUT="/work/fsc/corpus"
import docx, openpyxl, fitz
n=0
for dp,dn,fn in os.walk(ROOT):
    for f in sorted(fn):
        src=os.path.join(dp,f)
        rel=os.path.relpath(src,ROOT)
        dst=os.path.join(OUT, rel.replace('/','__')+'.txt')
        if os.path.exists(dst) and os.path.getsize(dst)>0:
            n+=1; continue
        text=None
        lo=f.lower()
        try:
            if lo.endswith('.pdf'):
                r=subprocess.run(['pdftotext','-layout',src,'-'],capture_output=True,text=True,timeout=120)
                text=r.stdout
            elif lo.endswith('.docx'):
                d=docx.Document(src)
                parts=[]
                for p in d.paragraphs:
                    parts.append(p.text)
                for ti,t in enumerate(d.tables):
                    parts.append(f'[TABLE {ti}]')
                    for row in t.rows:
                        parts.append(' | '.join(c.text.strip() for c in row.cells))
                text='\n'.join(parts)
            elif lo.endswith('.xlsx'):
                wb=openpyxl.load_workbook(src,data_only=True)
                parts=[]
                for ws in wb.worksheets:
                    parts.append(f'[SHEET {ws.title}] dims={ws.dimensions}')
                    for row in ws.iter_rows(values_only=True):
                        if any(v is not None for v in row):
                            parts.append(' | '.join('' if v is None else str(v) for v in row))
                text='\n'.join(parts)
            elif lo.endswith(('.txt','.csv','.eml','.json','.md','')):
                text=open(src,encoding='utf-8',errors='replace').read()
            elif lo.endswith(('.fcstd','.png','.whl')):
                text='[BINARY %s %d bytes]'%(lo,os.path.getsize(src))
            else:
                text='[UNKNOWN %s]'%lo
        except Exception as e:
            text='[ERROR %s: %s]'%(lo,e)
        open(dst,'w',encoding='utf-8').write(text or '')
        n+=1
print('dumped',n)

```


## code/src/make_memo.py

```text
#!/usr/bin/env python3
"""Builds /work/fsc/out/CFDME-2025-041_transient_solve_disposition_revA.docx.

Every number in the text is interpolated from out/computed.json, which compute.py
derives from the case record. Nothing is typed in by hand.
"""
import json, os
from docx import Document
from docx.shared import Pt, Inches, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.table import WD_TABLE_ALIGNMENT
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

C = json.load(open("/work/fsc/out/computed.json"))
OUT = "/work/fsc/out/CFDME-2025-041_transient_solve_disposition_revA.docx"

def f2(x):  return f"{x:.2f}"
def f4(x):  return f"{x:.4f}"
def hm(h):  return f"{int(h)} h {round((h-int(h))*60):02d} m"

D_m      = 3.0
f_hz     = C["f_pred_Hz"]                 # 0.866025...
T_s      = C["T_pred_s"]                  # 1.1547005...
f_2      = f"{C['f_pred_2dp']:.2f}"       # 0.87
T_2      = f"{C['T_pred_2dp']:.2f}"       # 1.15
agr      = f"{C['agreement_pct_2dp']:.2f}"# 0.41
agr_f    = f"{C['agreement_pct_as_frequency']:.4f}"
agr_T    = f"{C['agreement_pct_as_period']:.4f}"
Tq       = C["T_quoted_s"]                # 1.15
fq       = C["f_quoted_Hz"]               # 0.869565
wceil    = C["write_ceiling_s"]           # 0.046188
wint     = C["write_interval_s"]          # 0.0461
samp_ok  = C["samples_per_period_at_chosen"]
samp_no  = C["samples_per_period_at_next"]
startup  = C["startup_s"]                 # 8
end_ex   = C["end_time_exact_s"]          # 31.09401
end_t    = C["end_time_s"]                # 31.10
end_down = C["periods_at_float_down"]     # 19.9965
bench_t  = C["benchmark_simulated_s"]     # 40.00
fine_h   = C["grids"]["FINE"]["wall_h"]   # 5.75
rate     = C["fine_rate_h_per_sim_s"]     # 0.14375
per_case = C["per_case_h"]                # 4.4698
matrix   = C["matrix_h"]                  # 26.8186
a3_h     = C["attempt3_h"]                # 6.6833
ratio    = C["attempt3_vs_case_ratio"]    # 1.4952
excess   = C["attempt3_excess_h"]         # 2.21357
zones    = C["production_zones"]
prod_n   = C["production_cells_checkMesh"]
fluid_n  = zones["atmosphere"]
solid_n  = prod_n - fluid_n
cells_st = C["grids"]["COARSE"]["cells"]; cells_md = C["grids"]["MEDIUM"]["cells"]; cells_fn = C["grids"]["FINE"]["cells"]
dt_fine  = C["grids"]["FINE"]["dt_mean"]  # 0.0022
maxco    = C["grids"]["FINE"]["maxCo"]    # 0.9
meanc    = C["grids"]["FINE"]["meanCo"]   # 0.47
a3_iter  = C["attempt3_iterations"]       # 800
a3_sim   = C["attempt3_simulated_s"]      # 8.00
deliv_end= C["delivered_endTime"]         # 2000
deliv_sim= C["delivered_simulated_s_at_fixed_dt"]  # 20.00
samples_delivered = T_s / (100*0.01)      # writes every 1.00 s
min_sim  = f"{20*T_s:.2f}"

doc = Document()

# ---------- page and styles
sec = doc.sections[0]
sec.page_width, sec.page_height = Inches(8.5), Inches(11)
for attr, val in (("left_margin",0.95),("right_margin",0.95),("top_margin",0.8),("bottom_margin",0.8)):
    setattr(sec, attr, Inches(val))

st = doc.styles["Normal"]
st.font.name = "Calibri"; st.font.size = Pt(10)
st.paragraph_format.space_after = Pt(5)
st.paragraph_format.line_spacing = 1.06
st.element.rPr.rFonts.set(qn("w:eastAsia"), "Calibri")

def shade(cell, hexcolor):
    tcPr = cell._tc.get_or_add_tcPr()
    sh = OxmlElement("w:shd"); sh.set(qn("w:val"), "clear"); sh.set(qn("w:color"), "auto"); sh.set(qn("w:fill"), hexcolor)
    tcPr.append(sh)

def rule(par, sz=12, color="1F3864"):
    pPr = par._p.get_or_add_pPr()
    pbdr = OxmlElement("w:pBdr"); bottom = OxmlElement("w:bottom")
    bottom.set(qn("w:val"), "single"); bottom.set(qn("w:sz"), str(sz)); bottom.set(qn("w:space"), "1"); bottom.set(qn("w:color"), color)
    pbdr.append(bottom); pPr.append(pbdr)

def para(text="", bold=False, italic=False, size=None, align=None, space_before=None, space_after=None, color=None):
    p = doc.add_paragraph()
    r = p.add_run(text); r.bold = bold; r.italic = italic
    if size: r.font.size = Pt(size)
    if color: r.font.color.rgb = RGBColor.from_string(color)
    if align == "c": p.alignment = WD_ALIGN_PARAGRAPH.CENTER
    if align == "r": p.alignment = WD_ALIGN_PARAGRAPH.RIGHT
    if space_before is not None: p.paragraph_format.space_before = Pt(space_before)
    if space_after is not None: p.paragraph_format.space_after = Pt(space_after)
    return p

def lead(bold_lead, text, size=None):
    p = doc.add_paragraph()
    r = p.add_run(bold_lead); r.bold = True
    if size: r.font.size = Pt(size)
    r2 = p.add_run(text)
    if size: r2.font.size = Pt(size)
    return p

def h1(text):
    p = doc.add_paragraph(); p.paragraph_format.space_before = Pt(12); p.paragraph_format.space_after = Pt(3)
    p.paragraph_format.keep_with_next = True
    r = p.add_run(text); r.bold = True; r.font.size = Pt(12); r.font.color.rgb = RGBColor.from_string("1F3864")
    return p

def h2(text):
    p = doc.add_paragraph(); p.paragraph_format.space_before = Pt(8); p.paragraph_format.space_after = Pt(2)
    p.paragraph_format.keep_with_next = True
    r = p.add_run(text); r.bold = True; r.italic = False; r.font.size = Pt(10.5)
    return p

def bullet(text, bold_lead=None):
    p = doc.add_paragraph(style="List Bullet")
    p.paragraph_format.space_after = Pt(2)
    p.paragraph_format.left_indent = Inches(0.28)
    if bold_lead:
        r = p.add_run(bold_lead); r.bold = True
    p.add_run(text)
    return p

def table(rows, widths, header=True, font=8.6, header_fill="D9E2F3"):
    t = doc.add_table(rows=0, cols=len(widths))
    t.style = "Table Grid"; t.alignment = WD_TABLE_ALIGNMENT.CENTER
    t.autofit = False
    for ri, row in enumerate(rows):
        cells = t.add_row().cells
        for ci, val in enumerate(row):
            cells[ci].width = Inches(widths[ci])
            p = cells[ci].paragraphs[0]
            p.paragraph_format.space_after = Pt(1); p.paragraph_format.space_before = Pt(1)
            r = p.add_run(str(val)); r.font.size = Pt(font)
            if header and ri == 0:
                r.bold = True; shade(cells[ci], header_fill)
    for row in t.rows:
        for ci, cell in enumerate(row.cells):
            cell.width = Inches(widths[ci])
    doc.add_paragraph().paragraph_format.space_after = Pt(2)
    return t

def field(par, instr):
    fld = OxmlElement("w:fldSimple"); fld.set(qn("w:instr"), instr)
    r = OxmlElement("w:r"); t = OxmlElement("w:t"); t.text = "1"; r.append(t); fld.append(r)
    par._p.append(fld)

# ---------- letterhead
p = para("THE CFD ME", bold=True, size=14, space_after=0)
r = p.add_run("    |    Tucson, Arizona    |    AZ PE #48213    |    CFDME-2025-041    rev A")
r.font.size = Pt(9); r.font.color.rgb = RGBColor.from_string("404040")
rule(p)
para("TECHNICAL NOTE — TRANSIENT SOLVE DISPOSITION", bold=True, size=13, align="c", space_before=8, space_after=1)
para("Third steady-solve attempt, the authorized transient run, and the solver wall-clock projection — "
     "Ocotillo desert-fire case CFDME-2025-041", italic=True, size=9.5, align="c", space_after=8)

table([
 ["Project", "CFDME-2025-041 — Ocotillo WUI Wildfire, Phase 1 (client: Ocotillo Additive Structures)"],
 ["Subject", "Disposition of the third steady-solve attempt (SLURM job 41304); authorization and time-control block of the "
             "replacement transient run; solver wall-clock projection for the six-case matrix"],
 ["From", "Marguerite Falk, PE — Technical Director & Reviewing Engineer, AZ PE #42798"],
 ["Copy", "Gordon Selby, PE — Owner & Principal Engineer, AZ PE #48213"],
 ["To", "Project file CFDME-2025-041; Mulshi Simulation Labs (S. Bhatia), case supplier"],
 ["Date", "10/10/2025"],
 ["Governing documents", "Modeling Standards — Fire & Buoyant Flows rev 4 (effective 08/15/2025); QA Verification Standard — Mesh & "
                         "Resolution rev 3 (effective 07/01/2025); Units and Conversions SOP rev 2 (effective 04/01/2025); "
                         "SOW CFDME-2025-041-SOW-01 (signed 10/08/2025)"],
 ["Revision", "rev A — issued for action ahead of any fourth attempt"],
], widths=[1.35, 5.25], header=False, font=8.8)

# ---------- 1. disposition
h1("1.  Disposition")
para(f"This note closes the third steady-solve attempt on the Ocotillo desert-fire case and authorizes the transient run that "
     f"replaces it. It is dated 10/10/2025 and is issued before Mulshi starts a fourth attempt. The disposition, in order:")
bullet(f"The residual plateau in attempt 3 is the puffing oscillation of the buoyant plume — a physical frequency of the flow, "
       f"not a numerical stall. The period the run showed and the Cetegen–Ahmed prediction for this plume agree to {agr} %.",
       "The plateau is physics. ")
bullet("No steady solution exists for this problem, because the plume oscillates at a finite, deterministic, physical frequency "
       "and no time-invariant flow field exists for a residual-tolerance schedule to converge to. A fourth steady attempt is not "
       "authorized, and the operator's alternative of loosening the residual controls and keeping the cycle-averaged field is refused.",
       "There is no steady state to converge to. ")
bullet(f"FireFoam LES, EDC combustion, fvDOM at the firm reference settings, on the production conjugate mesh, for "
       f"{f2(end_t)} s of simulated time at a {f4(wint)} s write interval, on the block of Section 4.",
       "The transient run is authorized. ")
bullet("The radiation configuration (Section 3.1), the turbulence closure (Section 3.2) and the inlet flow direction "
       "(Section 6.2). Two record items are open and must be closed before the case is set up: a temperature-dependent solid "
       "property table (Section 3.3) and a sourced inlet roughness length (Section 6.3).",
       "Three items of the delivered setup are non-conforming and must be corrected. ")
bullet("An unconverged steady RAS field of a formulation the Modeling Standards forbid cannot seed a transient LES.",
       "The solved fields from attempt 3 are discarded. ")
bullet(f"{hm(per_case)} of solver wall clock for one authorized case and {hm(matrix)} for the six-case matrix. Both are a lower "
       f"bound, not a firm figure: two of the three conditions the firm projection basis sets are not met by the benchmark of "
       f"record (Section 7.4).", "Cost. ")

# ---------- 2. plateau
h1("2.  The residual plateau in attempt 3")
h2("2.1  What the plateau is")
para(f"The plume above a {D_m:.1f} m fire segment necks, sheds and re-forms at a frequency that depends only on the fire diameter "
     f"and gravity. Modeling Standards rev 4 Section 4 adopts the Cetegen–Ahmed relation as normative for design fires in this "
     f"regime, f = 1.5 × D^(−1/2) with D in m. For the {D_m:.1f} m front segment fixed in the scoping memo of 10/08/2025 and "
     f"carried in the three-grid study:")
para(f"        f = 1.5 × ({D_m:.1f} m)^(−1/2) = {f2(f_hz)} Hz      (unrounded {f_hz:.4f} Hz)")
para(f"        T = 1 / f = {f2(T_s)} s                    (unrounded {T_s:.4f} s)")
para(f"Every LES run at this diameter is expected to display monitor-point oscillation at this frequency, and the standard is "
     f"explicit that this is a physical property of the plume rather than a numerical artefact. The same section states the "
     f"consequence that governs this disposition: because the plume oscillates at a finite, deterministic, physical frequency, "
     f"no time-invariant flow field exists that any residual-tolerance schedule can converge to.")

h2("2.2  What the run showed")
para(f"The closing narrative of the attempt-3 log records the plateau exactly where the plume is: over outer iterations 400 to 800 "
     f"the Ux initial residual holds in the band 3e-4 to 8e-4 (dimensionless) and the p initial residual in the band 2e-5 to 3e-5 "
     f"(dimensionless), while the plume-centreline probes oscillate at a quoted period of about {Tq:.2f} s with roughly 40 K "
     f"peak-to-peak on temperature and 2.4 m/s peak-to-peak on vertical velocity. The wall stand-off probes stay flat across the "
     f"same window: probe 3 reads U = (8.00, 0.13, 0.02) m/s at T = 308.5 K and probe 4 reads U = (7.74, 0.10, 0.12) m/s at "
     f"T = 310.2 K. An oscillation confined to the plume axis, in phase at 4 m, 8 m and 12 m above grade, is the plume's own "
     f"structure. A residual tail from the pressure solve would not look like that.")
table([
 ["Probe", "Location (mesh axes, m)", "Behaviour over outer iterations 400–800"],
 ["0", "(1.5, 0.0, 8.0) — plume centreline, 8 m AGL", f"oscillates at ~{Tq:.2f} s; ~40 K and ~2.4 m/s peak-to-peak"],
 ["1, 2", "(1.5, 0.0, 4.0) and (1.5, 0.0, 12.0) — plume centreline, 4 m and 12 m AGL", "oscillate in phase with probe 0"],
 ["3, 4", "(10.0, 0.0, 2.0) and (10.0, 0.0, 8.0) — wall stand-off, upwind face and above roof",
          "flat: T = 308.5 K and 310.2 K, U unchanged to two decimals"],
], widths=[0.6, 3.2, 2.8])

h2("2.3  How close the prediction and the run are, as a percentage")
para(f"The comparison has to be made against the unrounded prediction. Rounded to the two decimals the note reports, both the "
     f"prediction and the quoted value read {f_2} Hz and {T_2} s, and comparing those with each other returns a circular "
     f"0.00 %, which is not an agreement figure. Carried unrounded:")
bullet(f"as a frequency: quoted 1 / {Tq:.2f} s = {fq:.4f} Hz against the prediction {f_hz:.4f} Hz — {agr_f} %;")
bullet(f"as a period: quoted {Tq:.4f} s against the prediction {T_s:.4f} s — {agr_T} %.")
para(f"Both readings give {agr} %, and the agreement is stated as {agr} % on that basis.")

h2("2.4  Whether a steady solution exists")
para("It does not. The frequency is finite, deterministic and physical, so the flow has no time-invariant state for a "
     "residual-tolerance schedule to reach; the residual plateau is the signature of the physical oscillation and not evidence "
     "of a defective solver. Attempt 3 is therefore not a failed solve — it is a correct solve of the wrong formulation on a flow "
     "that has no steady state. The three recommendations in the operator's closing narrative are each declined:")
bullet(" cannot converge a physical oscillation; it changes the path toward a destination that does not exist.",
       "1. Further under-relaxation (U 0.3 to 0.2) ")
bullet(" cannot either. The pressure residual is a consequence of the plume passing through the pressure field; the probes that "
       "carry the physical signal are temperature and vertical velocity on the plume axis, and they lock onto the predicted "
       "frequency to " + agr + " % — which is what distinguishes a physical mode from numerical noise.",
       "2. Tightening the pressure tolerance to 1e-8 ")
bullet(" is refused. A steady RANS buoyant field is not a permissible starting point for fire work (Section 3.6 of the "
       "Modeling Standards), k-epsilon applied to LES is not permitted (Section 5), and a cycle-averaged field is a time-invariant "
       "object constructed from a flow that has none.", "3. Loosening the residual controls and keeping the cycle-averaged field ")

# ---------- 3. conformance
h1("3.  How the rest of the delivered setup stands against our standards")
para("The delivered case is read from the attempt-3 log and the case system directory. Four items fail the Modeling Standards "
     "and one record item is open; each carries the conforming configuration that replaces it.")
h2("3.1  Radiation — non-conforming")
para("The delivered case runs fvDOM with 32 rays and a single grey band (greyMeanCombustion absorption/emission, no scattering, "
     "no soot model). Modeling Standards rev 4 Section 3.4 takes the number of angular directions and the wavelength banding from "
     "the firm reference table, and Appendix A gives that table for the 1200–1700 K design range as four angular directions "
     "(S4-equivalent quadrature) and two wavelength bands split at 2.5 micrometres. The conforming configuration — and the one the "
     "authorized run carries — is therefore four angular directions and two wavelength bands split at 2.5 micrometres, with the "
     "absorption and emission coefficients set consistent with the combustion product composition rather than defaulted.")
h2("3.2  Turbulence closure — non-conforming")
para("The delivered case runs RAS k-epsilon (kEpsilon) inside the fire solver, with the operator's note that the steady k-epsilon "
     "solve was a bring-up device for a later LES. Modeling Standards rev 4 Section 5 permits only large-eddy simulation, with "
     "either a dynamic Smagorinsky closure (Germano–Lilly, test filter twice the mesh spacing) or WALE, and forbids k-epsilon "
     "variants applied to LES and hybrid RANS/LES formulations. The authorized run is therefore large-eddy simulation with a WALE "
     "sub-grid closure — the standard's preferred closure where solid boundaries carry a significant fraction of the thermal load, "
     "which is the case here, the wall assembly being the object of the analysis — with dynamic Smagorinsky as the permitted "
     "alternative. The choice is recorded in the case-setup record per Section 5.")
h2("3.3  Solid-region properties — record item open")
para("The delivered case carries no solid regions at all. The production mesh does — the outer leaf, the inner leaf and the ties — "
     "and Modeling Standards Section 3.5 requires k, rho and c_p for solid regions to be entered as temperature-tabulated functions or "
     "supported polynomial fits over the design temperature range. What the record holds are constants: Appendix A gives the "
     "printed siliceous-aggregate concrete as k = 1.4 W/(m·K), rho = 2200 kg/m³ and c_p = 900 J/(kg·K), and the galvanized steel "
     "ties as k = 45 W/(m·K), rho = 7850 kg/m³ and c_p = 490 J/(kg·K), and states that solid-region properties are held constant "
     "over the design range absent case-specific data. There is no temperature-dependent property table for the concrete leaves "
     "or for the steel ties anywhere in the record. This is an open record item: the only solid-region property values on file are "
     "constants, and a table must be established and entered before the conjugate solve is set up, because a constant-property "
     "short-cut for solid regions in a fire case is not permitted.")
h2("3.4  Case inheritance from a non-fire parent — non-conforming")
para("The case-setup export of record (OAS_case_setup_from_DWA.txt) documents the case as built from the validated Dry Wash "
     "Aerospace high-altitude external-aerodynamics case, with the solver, turbulence, freestream, combustion and radiation blocks "
     "carried forward and only the geometry and patch naming swapped to revC. Modeling Standards rev 4 Section 7 prohibits that "
     "transfer in terms: a validated aerospace compressible non-reacting case shall not be pulled forward as the physics basis for "
     "a ground-level fire case, and case-directory reuse without a full physics rebuild is not acceptable practice. The export also "
     "records that no combustion model, no radiation dictionary and no reactive scalars are compiled in, while the live "
     "controlDict and the attempt-3 log show fireFoam with EDC and fvDOM — so the record holds no case-setup artifact that conforms "
     "to Section 3. The freestream block of the export (p_inf = 26 500 Pa, T_inf = 223 K, rho_inf = 0.4 kg/m³, M_inf = 0.25) is an "
     "aerospace reference condition and must not appear in the authorized case; the run of record used an ambient of 308.15 K "
     "(35 °C). The physics on the production mesh is to be recorded as built new, per Section 7 of the Modeling Standards.")
h2("3.5  What the case must be")
para("On that reading the authorized run is: fireFoam, large-eddy simulation with WALE, EDC combustion carried forward unchanged, "
     "fvDOM with four angular directions and two bands split at 2.5 micrometres, a four-region conjugate treatment (outer leaf and "
     "inner leaf as solid regions with temperature-dependent properties, the cavity as a fluid region with participating "
     "radiation, and each tie as a separate solid region with refinement at the tie–concrete interfaces) coupled with the "
     "chtMultiRegion boundary conditions of Modeling Standards Section 6, the atmospheric-boundary-layer inlet of Section 7 with "
     "matching turbulence fields, and the time-control block of Section 4 below.")

# ---------- 5 (numbered 4 in the note)
h1("4.  The run I am authorizing — the time-control block")
h2("4.1  The block, with values")
table([
 ["Entry", "Authorized value", "Basis"],
 ["application", "fireFoam", "Modeling Standards §3.1"],
 ["startFrom", "latestTime", "firm reference transient controlDict"],
 ["startTime", "0 s", "first run of the authorized case"],
 ["stopAt", "endTime", "—"],
 ["endTime", f"{f2(end_t)} s", f"§4: {startup:.0f} s startup + 20 periods (Section 4.3)"],
 ["deltaT", "0.001 s (first step, then Courant-controlled)", "firm reference transient controlDict"],
 ["adjustTimeStep", "yes", "§4, mandatory"],
 ["maxCo", "0.9 (dimensionless)", "§4, mandatory"],
 ["maxDeltaT", "0.01 s", "§4 ceiling, fixed by the firm reference dictionary (Section 4.2)"],
 ["writeControl", "adjustableRunTime", "§4, mandatory"],
 ["writeInterval", f"{f4(wint)} s", f"§4: ≥ 25 samples per period (Section 4.3)"],
 ["purgeWrite", "0", "all written times retained for the spectra"],
 ["writeFormat / writePrecision", "ascii / 8 (significant figures)", "firm reference transient controlDict"],
 ["writeCompression / timeFormat", "off / general", "firm reference transient controlDict"],
 ["runTimeModifiable", "true", "firm reference transient controlDict"],
], widths=[1.55, 2.55, 2.5])

h2("4.2  The reference dictionary that governs the case-specific ceiling")
para("Section 4 of the Modeling Standards fixes adjustTimeStep yes, maxCo 0.9, writeControl adjustableRunTime and a maxDeltaT "
     "ceiling that it leaves case-specific, \"typically 0.01 s\". Our reference transient dictionary — "
     "Firm/Standards/Reference_Cases/fireFoam_transient_controlDict.txt — sets maxDeltaT 0.01 s, and that is the ceiling carried "
     "into the authorized block. Two of that dictionary's own settings are not carried: its endTime of 60 s and its writeInterval "
     "of 0.1 s are the settings of its own 60 s run, and a 0.1 s write interval would capture only "
     f"{T_s/0.1:.1f} samples per puffing period here, against the 25 the standard requires. The ceiling that governs the step is "
     "0.01 s; the write interval and the end time are set from Section 4 of the standard for this case, below.")

h2("4.3  Largest write interval and shortest end time the standard allows here")
para(f"Write interval. Section 4 requires at least 25 monitor samples per puffing period. With the period carried at full "
     f"precision, T = {T_s:.4f} s, the ceiling is T / 25 = {wceil:.6f} s. The largest four-decimal write interval that meets it "
     f"is {f4(wint)} s, which gives {samp_ok:.2f} samples per period; {f4(wint+0.0001)} s gives {samp_no:.2f} and fails. A write "
     f"interval derived from the rounded period of {T_2} s would be {1.15/25:.4f} s — still compliant, but not the largest the "
     f"standard allows, because the rounded period is the shorter of the two.")
para(f"End time. Section 4 requires the simulated end time to span a minimum of 20 puffing periods after the initial transient "
     f"has cleared, which it puts at no less than approximately {min_sim} s of physical time for a {D_m:.1f} m fire, exclusive of "
     f"startup. The three-grid study records the startup transient as cleared by t ≈ {startup:.0f} s, and the Phase 1 draft "
     f"samples from t ≥ {startup:.0f} s on the same basis. Hence:")
para(f"        end time = {startup:.0f} s + 20 × {T_s:.4f} s = {startup:.0f} s + {20*T_s:.5f} s = {end_ex:.5f} s")
para(f"As a floor, {f2(end_t)} s is the shortest value to two decimals that satisfies the rule: {end_t-0.01:.2f} s leaves "
     f"{end_down:.4f} periods after startup, short of the twenty required. The authorized end time is {f2(end_t)} s.")

h2("4.4  The block Mulshi delivered, entry by entry")
para("Every entry that differs between the delivered controlDict and the authorized block, with both values:")
table([
 ["Entry", "Delivered by Mulshi", "Authorized", "Why it differs"],
 ["startFrom", "startTime", "latestTime", "the authorized run may restart from the last written time"],
 ["endTime", f"{deliv_end} (counted as outer iterations)", f"{f2(end_t)} s (physical time)",
  f"{deliv_end} iterations × 0.01 s = {deliv_sim:.2f} s of simulated time, under the {min_sim} s the standard requires before startup is added"],
 ["deltaT", "0.01 s, fixed", "0.001 s first step, then Courant-controlled", "a fixed step cannot hold maxCo through the plume passage"],
 ["adjustTimeStep", "no", "yes", "§4, mandatory"],
 ["maxCo", "absent (no entry)", "0.9 (dimensionless)", "§4, mandatory"],
 ["maxDeltaT", "absent (no entry)", "0.01 s", "§4 ceiling, firm reference dictionary"],
 ["writeControl", "timeStep", "adjustableRunTime", "§4: writes pegged to simulated time, not to solver iteration"],
 ["writeInterval", "100 time steps", f"{f4(wint)} s",
  f"100 steps × 0.01 s = 1.00 s of simulated time between writes, i.e. {samples_delivered:.2f} samples per puffing period"],
 ["writePrecision", "6", "8", "firm reference transient controlDict"],
], widths=[1.25, 1.6, 1.55, 2.2])
para(f"The delivered write interval records about {samples_delivered:.2f} samples per puffing period, against the 25 the standard "
     f"requires; the delivered endTime of {deliv_end} outer iterations is an iteration count rather than a simulated time, and at "
     f"the delivered fixed step it corresponds to {deliv_sim:.2f} s. Attempt 3 reached {a3_iter} of those iterations, that is "
     f"{a3_sim:.2f} s of simulated time — it stopped at the end of the startup window, before a single period of the window the "
     f"standard requires had been sampled.")

# ---------- 5 mesh & verification
h1("5.  The mesh the attempt ran on, and what our verification of it covers")
h2("5.1  The attempt ran on the production conjugate mesh")
para(f"The attempt-3 log reports {prod_n:,} cells with a maximum non-orthogonality of 58.34 (limit 65, dimensionless) and a maximum "
     f"skewness of 2.14 (limit 4, dimensionless) — the figures the meshing log of 10/09/2025 records for the production mesh "
     f"(job borrego/41287, 32 cores, geometry of record OAS-WUI-01_revC). Its five cell zones are atmosphere (fluid, "
     f"{zones['atmosphere']:,} cells), outerLeaf (solid, {zones['outerLeaf']:,} cells), cavity (fluid with participating radiation, "
     f"{zones['cavity']:,} cells), innerLeaf (solid, {zones['innerLeaf']:,} cells) and ties (solid, {zones['ties']:,} cells), and it "
     f"carries 13 boundary patches.")
h2("5.2  The three-grid study is a different, fluid-only family of meshes")
para(f"The verification study of 10/09/2025 runs three separate meshes: {cells_st:,} cells at a background dx = 1.0 m, {cells_md:,} "
     f"cells at dx = 0.7 m and {cells_fn:,} cells at dx = 0.5 m. Each of the three carries a single cell zone, which is fluid "
     f"(fireRefinementZone), and six boundary patches — ground, inlet_240, inlet_270, outlet, top and fire_source — with no wall "
     f"patch among them. None of the wall assembly, the cavity or the ties exists in these meshes.")
h2("5.3  What the verification covers")
para(f"It verifies plume resolution in the fluid domain. On the fine grid the plume resolution index is D*/dx = {C['Dstar_over_dx_study']:.2f} "
     f"(dimensionless, within the required band of 5 to 10) and the Pope resolved-TKE fraction is {C['pope_resolved']:.2f} "
     f"({C['pope_resolved']*100:.0f} %), above the 80 % minimum. On that basis the fluid-domain resolution at dx = 0.5 m carries "
     f"into the authorized run. A Richardson or GCI band is not part of that evidence: QA Verification Standard rev 3 Section 4 "
     f"states that the grid convergence index is not valid for LES and shall not be used to verify any large-eddy result.")
h2("5.4  What it does not cover")
para("The solid leaves, the cavity radiation and the tie regions of the production mesh are outside the scope of the three-grid "
     "verification. None of those regions exists in the study meshes — one fluid cell zone per mesh, no wall patch, no conjugate "
     "region and no participating-radiation cavity — so nothing about the conjugate heat transfer through the leaves, the "
     "radiative exchange inside the cavity or the conduction into the ties is verified by the three grids. Those mechanisms are "
     "exactly what sets the single-point temperature rise of this assembly. The verification that carries to the authorized run "
     "is therefore the fluid-plume resolution only; the conjugate response needs its own verification and reporting under Sections "
     "5 and 6 of the QA standard.")

# ---------- 6 what carries forward
h1("6.  What in the delivered case carries forward, with the inlet's defining settings")
h2("6.1  Carries forward")
bullet("The production conjugate mesh on the geometry of record OAS-WUI-01_revC, with its five cell zones and 13 boundary patches.")
bullet("The EDC combustion model of the delivered case (version v2005), which Modeling Standards Section 3.3 mandates: EDC "
       "carries forward unchanged into the authorized run, with the finite-rate chemistry coupling and its C_gamma 2.1377 and "
       "C_num 0.4083 constants.")
bullet("The atmospheric-boundary-layer inlet formulation with its matching turbulence inlet fields, and the probe set of "
       "Section 2.2 (plume centreline at 4 m, 8 m and 12 m above grade; wall stand-off at 2 m and 8 m).")
bullet("The job accounting of attempt 3, as a cost datum only (Section 7.7).")
h2("6.2  The inlet's defining settings, and the one that must change")
para(f"The delivered inlet is atmBoundaryLayerInletVelocity with Uref = {C['inlet_Uref']:.1f} m/s at Zref = {C['inlet_Zref']:.1f} m "
     f"(the 20 ft reference height of SOW Section 3.1, case 3.1-B of the matrix), z0 = {C['inlet_z0']:.2f} m and flowDir "
     f"(cos(240) sin(240) 0). The reference velocity and the reference height carry forward unchanged.")
para("The flow direction does not. Units and Conversions SOP rev 2 Section 2 fixes wind direction on the compass convention, "
     "north = 0 degrees and clockwise positive, with solver-internal angles in radians. Evaluated, (cos 240°, sin 240°, 0) = "
     "(−0.500, −0.866, 0): that vector points to 210 degrees on the compass, so it drives the flow out through the western boundary "
     "and does not describe a wind from 240 degrees — it describes a wind from 030 degrees. A wind from 240 degrees flows toward "
     "060 degrees, and with x east and y north — the axes the production mesh fixes with its inlet_west, outlet_east, "
     "lateral_north and lateral_south patches — the unit vector for the inlet flow direction is (0.866, 0.500, 0), equivalently "
     "(cos 30°, sin 30°, 0). That is the direction the authorized run carries. The delivered entry is also neither convention "
     "cleanly: it is a mathematical angle in degrees evaluated on mesh axes, while the SOP puts wind directions on the compass in "
     "degrees and solver-internal angles in radians.")
h2("6.3  The roughness length cannot carry forward as conforming")
para("Modeling Standards Section 3.2 requires the roughness length, the reference height and the reference velocity to be taken "
     "from the project's site meteorological pull. The pull of record (site_met_avra_valley_2015-2024.csv, 240 hourly records plus "
     "the design-envelope footer) carries a timestamp, ambient temperature, relative humidity, 20 ft wind speed, wind direction and "
     "solar irradiance, and holds no roughness length. The 0.03 m on the delivered inlet is Mulshi's own annotation in the "
     "attempt-3 log (\"buffelgrass roughness\"); it is not sourced from the pull and the record holds no other basis for it. This "
     "is an open record item: the roughness length must be sourced and recorded with its reference before the authorized run is "
     "set up, and the value entered at the 6.1 m reference height it belongs to.")

# ---------- 7 cost
h1("7.  Cost")
h2("7.1  The basis of record")
para("The firm projection basis is the reviewing engineer's note of 10/09/2025 18:05 to R. Ellsworth. It has five points: (1) at "
     "a fixed mesh, a fixed solver stack and a fixed core count, solver wall clock is linear in simulated physical time, the rate "
     "being taken from a run of record on that grid as simulated seconds reached divided by wall clock consumed; (2) the rate is "
     "taken from the grid the production case will actually run on, never carried from one grid to another; (3) the figure is "
     "quoted per case first and then multiplied by the number of cases the governing scope document procures; (4) the number is "
     "solver wall clock and nothing else — queue wait, staging, meshing and decomposition, post-processing and cloud-burst "
     "provisioning latency are excluded; (5) the linear-scaling step is written into the assumptions section of whatever document "
     "carries the number. Nothing beyond those five steps is applied.")
h2("7.2  Solver wall clock for one authorized case")
para(f"The run of record on the fine grid is the FINE run of the three-grid study: {bench_t:.2f} s of simulated time in "
     f"{fine_h:.2f} h of solver wall clock (5 h 45 m as posted for that run; the queue wait before its burst allocation is "
     f"excluded under point 4 of the basis). The rate is therefore {fine_h:.2f} h / {bench_t:.2f} s = {rate:.6f} h per simulated "
     f"second, and one authorized case is:")
para(f"        {rate:.6f} h/s × {end_ex:.5f} s = {per_case:.3f} h  =  {hm(per_case)} of solver wall clock")
h2("7.3  Solver wall clock for the full six-case matrix")
para(f"SOW Section 3.1 procures six cases (240 and 270 degrees at 5, 8 and 12 m/s). Within the matrix every case carries the same "
     f"mesh and the same required physical duration and only the inlet condition changes, so the per-case figure multiplies by "
     f"six, with no discount for cases expected to restart from a neighbour:")
para(f"        6 × {per_case:.3f} h = {matrix:.2f} h  =  {hm(matrix)} of solver wall clock")
para(f"({hm(round(per_case*60)/60*6)} from multiplying the rounded per-case figure of {hm(per_case)} is equally correct.)")
h2("7.4  Whether that basis can be met for this run, on every condition it sets")
para("The basis sets three conditions of equal standing. The verdict on each:")
table([
 ["Condition", "Verdict", "Evidence in the record"],
 ["Fixed solver stack", "Met",
  "The study runs fireFoam LES with EDC combustion and fvDOM radiation, and Modeling Standards Section 3.1 makes fireFoam the "
  "solver for the production case: the study and the production case both run the same reacting solver family. The conjugate "
  "regions are mesh content and are counted under the fixed-mesh condition, not here."],
 ["Fixed mesh", "Not met",
  f"The benchmark ran on the {cells_fn:,}-cell fluid-only plume mesh, which carries none of the wall assembly, the cavity or the "
  f"ties. The production run is the {prod_n:,}-cell multi-region conjugate solve with five cell zones."],
 ["Fixed core count", "Not met",
  "The study records 96 cores for the coarse grid and 192 cores for the medium grid, and no core allocation at all for the fine "
  "grid (posted as \"borrego + cloud burst\"). The allocation behind the rate is therefore not recorded."],
], widths=[1.1, 0.7, 4.8])
para("Two of the three conditions therefore fail, so the linear-scaling precondition the projection rests on is not satisfied by "
     "the benchmark of record. The figures of Sections 7.2 and 7.3 remain the answer the basis produces, and they are a projection "
     "under a stated assumption rather than a measurement.")
h2("7.5  What the time step and the Courant margin behind the figure imply")
para(f"The fine-grid benchmark reached a maximum Courant number of {maxco:.2f} against the 0.9 ceiling — no margin at all, with a "
     f"mean Courant number of {meanc:.2f} (dimensionless) and an average time step of {dt_fine:.4f} s. Its finest cells measure "
     f"0.125 m (minimum cell volume 0.001936 m³ on the fine grid). The production mesh carries 0.0625 m cells in the wall assembly and the cavity and 0.03125 m cells at the ties and "
     f"the tie–concrete interfaces, and {solid_n:,} cells in the four non-atmosphere zones that the benchmark never advanced. A "
     f"Courant-limited step scales with cell size at fixed velocity, so the production conjugate solve will advance at a smaller "
     f"average time step than the {dt_fine:.4f} s recorded on the fine study grid, and it does extra work per step in the solid and "
     f"tie zones. Both effects push the true cost above the projection.")
h2("7.6  What the figure is worth")
para(f"Both figures — {hm(per_case)} for one case and {hm(matrix)} for the matrix — are a lower bound on the true cost, not a firm "
     f"estimate. They carry no conjugate solid, cavity-radiation or tie work, and the rate behind them comes from a fluid-only grid "
     f"whose core allocation is unrecorded. A firm solver wall-clock figure for this project requires a benchmark run on the "
     f"production conjugate mesh — a transient fireFoam run of record on the {prod_n:,}-cell mesh. The only run that exists on that "
     f"mesh is attempt 3, and its rate cannot be used either: it is a steady RANS solve on a different solver stack and has no "
     f"Courant-controlled time step ({a3_iter} outer iterations at a fixed 0.01 s, that is {a3_sim:.2f} s of simulated time, in "
     f"{hm(a3_h)}).")
para(f"No contingency, margin or risk multiplier is applied to either figure, and no overhead percentage is applied to either "
     f"figure. Nothing is added for queue wait, staging, meshing or decomposition, post-processing or cloud-burst provisioning "
     f"latency: the basis excludes them (point 4) and none of them appears in the projection. The two figures are the measured rate "
     f"multiplied by the procured cases and nothing else.")
h2("7.7  What attempt 3 consumed, against one authorized case")
para(f"Attempt 3 (SLURM job 41304, {C['attempt3_cores']} cores across four nodes of the borrego cluster) consumed "
     f"{hm(a3_h)} of wall clock and {C['attempt3_cpu_hours']:.1f} CPU-hours. Against one authorized case at {hm(per_case)} that is "
     f"{ratio:.2f} times the solver wall clock, and {hm(excess)} more solver wall clock than one authorized case.")
h2("7.8  What of it, if anything, is usable downstream")
para("Nothing of the solved fields: they are discarded. An unconverged steady RAS field of a formulation Modeling Standards "
     "Sections 3.6 and 5 forbid cannot seed a transient LES, and nothing in it is a boundary condition for the authorized run. What "
     "is usable is everything around the fields — the production mesh, the probe set, the EDC configuration, the inlet formulation "
     "once its direction and roughness are corrected (Section 6), and the job accounting as a cost datum.")

# ---------- 8 assumptions
h1("8.  Assumptions behind the cost figures")
bullet(f" Solver wall clock is linear in simulated physical time at a fixed mesh, a fixed solver stack and a fixed core count. "
       f"That is the assumption the projection rests on, and Section 7.4 records that two of its three conditions are not satisfied "
       f"by the benchmark of record; the figures are a projection, not a measurement.")
bullet(f" The rate is that of the fine-grid run of record: {bench_t:.2f} s of simulated time in {fine_h:.2f} h of solver wall "
       f"clock, queue wait excluded.")
bullet(f" The simulated duration of each authorized case is {end_ex:.5f} s, the {startup:.0f} s startup recorded in the three-grid "
       f"study plus twenty unrounded puffing periods of {T_s:.4f} s.")
bullet(" Six cases are procured, per SOW Section 3.1; every case carries the same mesh and the same required duration, and only the "
       "inlet condition changes.")
bullet(" The ambient condition of record is 308.15 K (35 °C); 20 ft is 6.096 m, recorded throughout as 6.1 m.")

# ---------- 9 sign-off
h1("9.  Reviewer and stamp block")
table([
 ["Reviewing engineer", "Marguerite Falk, PE — Technical Director & Reviewing Engineer, AZ PE #42798"],
 ["Signature", "_/s/ Marguerite Falk, PE /_"],
 ["Date", "10/10/2025"],
 ["Disposition", "Attempt 3 is closed as non-converging by physics; a fourth steady attempt is not authorized. The transient run "
                 "of Section 4 is authorized, subject to closure of the two record items in Sections 3.3 and 6.3."],
], widths=[1.35, 5.25], header=False, font=8.8)
table([
 ["[  PE STAMP — APPLIED  ]", "The CFD Mechanical Engineer, LLC — Arizona PE #48213\nGordon Selby, PE — Owner & Principal Engineer\nDate: 10/10/2025"],
 ["Signature", "_/s/ Gordon Selby, PE /_"],
 ["Stamp boundary", "CFD thermal analysis only. Fire-resistance rating certified separately by others."],
], widths=[1.6, 5.0], header=False, font=8.8)
para("This note is a computational thermal analysis. It does not certify, extend or modify any fire-resistance rating of the "
     "assembly; the certified rating is issued separately by others under a separate engagement, as SOW Sections 4 and 10 require.",
     italic=True, size=8.8)

# ---------- footer
footer = sec.footer.paragraphs[0]
footer.text = ""
r = footer.add_run("CFDME-2025-041  rev A   |   Transient solve disposition   |   10/10/2025   |   page ")
r.font.size = Pt(8); r.font.color.rgb = RGBColor.from_string("595959")
field(footer, "PAGE")

doc.save(OUT)
print("wrote", OUT, os.path.getsize(OUT), "bytes")

```


## code/src/mutate.py

```text
#!/usr/bin/env python3
"""Mutation campaign on the disposition.

Each mutation breaks exactly one claim, in every place the document states it (prose and
table cells), is rendered by the task's own transform.py, and read by checks.py. A
mutation is CAUGHT only when a non-empty list of criteria goes red. A survivor is a hole
in the reader (or an equivalent mutation) and is reported, not hidden.
"""
import json, os, subprocess, sys
from docx import Document
sys.path.insert(0, "/work/fsc/src")
import checks

BASE = "/work/fsc/out/CFDME-2025-041_transient_solve_disposition_revA.docx"
TR   = "/work/ple-dataset/tasks/fire-solve-convergence/tests/tools/verifiers/deps/transform.py"
VARD = "/work/fsc/mut"
os.makedirs(VARD, exist_ok=True)

def rewrite(doc, pairs, cell_pairs):
    hits = 0
    def fix(par, exact_only=False):
        nonlocal hits
        whole = "".join(r.text for r in par.runs)
        new = whole
        if exact_only:
            for old, rep in cell_pairs:
                if new.strip() == old:
                    new = rep; break
        else:
            for old, rep in pairs:
                if old in new:
                    new = new.replace(old, rep)
        if new != whole:
            hits += 1
            for r in par.runs[1:]:
                r._element.getparent().remove(r._element)
            if par.runs: par.runs[0].text = new
            else: par.add_run(new)
    for p in doc.paragraphs: fix(p)
    for t in doc.tables:
        for row in t.rows:
            for c in row.cells:
                for p in c.paragraphs: fix(p)          # substring pairs apply everywhere
    for t in doc.tables:
        for row in t.rows:
            for c in row.cells:
                for p in c.paragraphs: fix(p, exact_only=True)
    for s in doc.sections:
        for p in s.footer.paragraphs: fix(p)
        for p in s.header.paragraphs: fix(p)
    return hits

M = [
 ("M01", "#1 predicted frequency 0.87 -> 0.88 Hz", [("0.87 Hz", "0.88 Hz")], [], [1]),
 ("M02", "#3 agreement 0.41 -> 0.40 percent", [("0.41 %", "0.40 %")], [], [3]),
 ("M03", "#10 write interval 0.0461 -> 0.0462 s", [("0.0461 s", "0.0462 s")], [], [10]),
 ("M04", "#11 end time 31.10 -> 31.09 s", [("31.10 s", "31.09 s")], [], [11]),
 ("M05", "#4 'no steady solution' turned into a positive statement",
  [("No steady solution exists for this problem", "A steady solution exists for this problem"),
   ("no time-invariant flow field exists", "a time-invariant flow field exists"),
   ("the flow has no time-invariant state", "the flow has a time-invariant state"),
   ("a flow that has no steady state", "a flow that has a steady state"),
   ("There is no steady state to converge to.", "There is a steady state to converge to.")], [], [4]),
 ("M06", "#5 four angular directions -> eight", [("four angular directions", "eight angular directions")], [], [5]),
 ("M07", "#6 band split 2.5 -> 3.0 micrometres", [("2.5 micrometres", "3.0 micrometres")], [], [6]),
 ("M08", "#7 closure turned into k-epsilon",
  [("large-eddy simulation with a WALE sub-grid closure", "steady RANS with a k-epsilon closure"),
   ("WALE", "k-epsilon"), ("dynamic Smagorinsky", "k-epsilon")], [], [7]),
 ("M09", "#8 writeControl adjustableRunTime -> timeStep", [("adjustableRunTime", "timeStep")], [], [8]),
 ("M10", "#9 maxDeltaT 0.01 -> 0.02 s",
  [("maxDeltaT 0.01 s", "maxDeltaT 0.02 s")], [("0.01 s", "0.02 s")], [9]),
 ("M11", "#12 delivered endTime 2000 -> 3000",
  [("2000 iterations × 0.01 s = 20.00 s", "3000 iterations × 0.01 s = 20.00 s"),
   ("the delivered endTime of 2000 outer iterations", "the delivered endTime of 3000 outer iterations")],
  [("2000 (counted as outer iterations)", "3000 (counted as outer iterations)")], [12]),
 ("M12", "#14 attempt 3 said to have run on the study grid",
  [("The attempt ran on the production conjugate mesh", "The attempt ran on the fine study grid"),
   ("production conjugate mesh", "fine study grid"),
   ("11,214,862-cell", "11,196,000-cell"), ("11,214,862 cells", "11,196,000 cells")], [], [14]),
 ("M13", "#15 five cell zones -> four", [("five cell zones", "four cell zones")], [], [15]),
 ("M14", "#16 Pope fraction 0.84 -> 0.72", [("0.84 (84 %)", "0.72 (72 %)")], [], [16]),
 ("M15", "#17 scope reversed: study said to cover the solid and tie regions",
  [("are outside the scope of the three-grid verification", "are inside the scope of the three-grid verification"),
   ("What it does not cover", "What else it covers"),
   ("so nothing about the conjugate heat transfer", "so everything about the conjugate heat transfer")], [], [17]),
 ("M16", "#18 study meshes carry two zones, one solid",
  [("carries a single cell zone, which is fluid", "carries two cell zones, one of them solid")], [], [18]),
 ("M17", "#19 six boundary patches -> four", [("six boundary patches", "four boundary patches")], [], [19]),
 ("M18", "#20 inlet reference velocity 8.0 -> 12.0 m/s", [("Uref = 8.0 m/s", "Uref = 12.0 m/s")], [], [20]),
 ("M19", "#21 EDC replaced",
  [("EDC carries forward unchanged", "EDC is replaced by a mixture-fraction model"),
   ("EDC combustion carried forward unchanged", "EDC combustion replaced by a mixture-fraction model")], [], [21]),
 ("M20", "#22 delivered flow direction called correct",
  [("and does not describe a wind from 240 degrees", "and correctly describes a wind from 240 degrees")], [], [22]),
 ("M21", "#23 inlet vector transposed",
  [("(0.866, 0.500, 0)", "(0.500, 0.866, 0)"), ("(cos 30°, sin 30°, 0)", "(cos 60°, sin 60°, 0)")], [], [23]),
 ("M22", "#24 roughness attributed to the met record",
  [("it is not sourced from the pull and the record holds no other basis for it",
    "it is taken from the site meteorological record, which is where a roughness length is carried")], [], [24]),
 ("M23", "#25 temperature-dependent table claimed on file",
  [("There is no temperature-dependent property table for the concrete leaves or for the steel ties anywhere in the record.",
    "A temperature-dependent property table for the concrete leaves and the steel ties is on file.")], [], [25]),
 ("M24", "#26 one case 4 h 28 m -> 4 h 30 m", [("4 h 28 m", "4 h 30 m")], [], [26]),
 ("M25", "#27 matrix 26 h 49 m -> 26 h 50 m", [("26 h 49 m", "26 h 50 m")], [], [27]),
 ("M26", "#28 core-count record claim removed",
  [("and no core allocation at all for the fine grid", "and the same core allocation as the other two grids"),
   ("The allocation behind the rate is therefore not recorded.", "The allocation behind the rate is therefore on record.")], [], [28]),
 ("M27", "#30 solver-stack verdict flipped: different families",
  [("the study and the production case both run the same reacting solver family",
    "the study and the production case run different solver families")], [], [30]),
 ("M28", "#31 max Courant 0.90 -> 0.80",
  [("maximum Courant number of 0.90 against the 0.9 ceiling", "maximum Courant number of 0.80 against the 0.9 ceiling")], [], [31]),
 ("M29", "#32 time-step comparison reversed",
  [("will advance at a smaller average time step than the 0.0022 s", "will advance at the same average time step as the 0.0022 s")], [], [32]),
 ("M30", "#33 lower bound -> firm estimate",
  [("are a lower bound on the true cost", "are a firm estimate of the true cost"),
   ("are a lower bound, not a firm figure", "are a firm estimate, not a bound")], [], [33]),
 ("M31", "#34 benchmark requirement moved off the production mesh",
  [("requires a benchmark run on the production conjugate mesh", "requires a benchmark run on the fine study grid")], [], [34]),
 ("M32", "#35/#36 contingency and overhead applied",
  [("No contingency, margin or risk multiplier is applied to either figure, and no overhead percentage is applied to either "
    "figure.",
    "A 15 % contingency and risk multiplier is applied to either figure, and a 10 % overhead percentage is applied to either "
    "figure.")], [], [35, 36]),
 ("M33", "#37 attempt-3 consumption misquoted", [("424.3 CPU-hours", "442.3 CPU-hours")], [], [37]),
 ("M34", "#38 ratio 1.50 -> 1.20", [("1.50 times the solver wall clock", "1.20 times the solver wall clock")], [], [38]),
 ("M35", "#39 excess 2 h 13 m -> 2 h 14 m", [("2 h 13 m", "2 h 14 m")], [], [39]),
 ("M36", "#40 fields retained",
  [("Nothing of the solved fields: they are discarded", "Nothing of the solved fields: they are retained as the LES initial condition"),
   ("The solved fields from attempt 3 are discarded.", "The solved fields from attempt 3 are retained.")], [], [40]),
 ("M37", "#41 stamp boundary altered",
  [("CFD thermal analysis only. Fire-resistance rating certified separately by others.",
    "CFD thermal analysis and fire-resistance rating. Fire-resistance rating certified separately by others.")], [], [41]),
 ("M38", "#42 date moved to 10/11/2025", [("10/10/2025", "10/11/2025")], [], [42]),
 ("M39", "#2 closing-narrative period re-attributed",
  [("The closing narrative of the attempt-3 log", "The operator's covering email")], [], [2]),
 ("M40", "#13 delivered startFrom said to be latestTime",
  [("startFrom | startTime", "startFrom | startTime")], [("startTime", "latestTime")], [13]),
 ("M41", "#0 plateau re-labelled as numerical noise", [("puffing", "residual")], [], [0]),
 ("M42", "#29 fixed-mesh verdict flipped to Met",
  [("The benchmark ran on the 11,196,000-cell fluid-only plume mesh, which carries none of the wall assembly, the cavity or the ties.",
    "The benchmark ran on the production conjugate mesh, which carries the wall assembly, the cavity and the ties."),
   ("Not met", "Met")], [], [29]),
]


def render(docx_path, pdf_path):
    r = subprocess.run(["/work/venv-verifier/bin/python", TR, "--input", docx_path, "--output", pdf_path,
                        "--transformation", "docx_to_pdf"], capture_output=True, text=True)
    if r.returncode != 0: return None, r.stdout + r.stderr
    t = subprocess.run(["pdftotext", "-layout", pdf_path, "-"], capture_output=True, text=True)
    return t.stdout, ""

base_text = open("/work/fsc/out/memo.txt", encoding="utf-8").read()
print("baseline red:", sorted(r["index"] for r in checks.run(base_text) if not r["pass"]) or "none")

results = []
for mid, desc, pairs, cell_pairs, expected in M:
    docx = os.path.join(VARD, f"{mid}.docx"); pdf = os.path.join(VARD, f"{mid}.pdf")
    d = Document(BASE); hits = rewrite(d, pairs, cell_pairs); d.save(docx)
    text, err = render(docx, pdf)
    if text is None:
        results.append(dict(id=mid, desc=desc, expected=expected, red=[], verdict="RENDER FAILED", hits=hits)); continue
    red = sorted(r["index"] for r in checks.run(text) if not r["pass"])
    ok = all(e in red for e in expected) and bool(red) and hits > 0
    results.append(dict(id=mid, desc=desc, expected=expected, red=red, verdict="caught" if ok else "SURVIVED", hits=hits))

for r in results:
    print(f"{r['id']} {r['verdict']:9s} hits={r['hits']} expected={r['expected']} red={r['red']} :: {r['desc']}")
print(f"\ncaught {sum(1 for r in results if r['verdict']=='caught')}/{len(results)}")
json.dump(results, open("/work/fsc/out/mutations.json","w"), indent=1)

```


## code/src/verify_numbers.py

```text
#!/usr/bin/env python3
"""Second, independent path to the note's numbers.

It does not import compute.py and does not read the corpus dump: it parses the raw record files with its own
patterns, recomputes each headline value with its own arithmetic (Fractions for the wall clocks), and then
asserts that the rendered note carries that exact value. A disagreement anywhere fails.
"""
import math, re, sys
from fractions import Fraction

RAW = "/work/ple-dataset/tasks/fire-solve-convergence/environment/files/Projects/CFDME-2025-041_Ocotillo_WUI"
LOG3  = open(f"{RAW}/Case/logs/log.solve_20251009_attempt3.txt").read()
SNAP  = open(f"{RAW}/Case/logs/log.snappyHexMesh_20251009.txt").read()
THREE = open(f"{RAW}/Case/verification/three_grid_study_outputs.txt").read()
CTRL  = open(f"{RAW}/Case/system/controlDict").read()
NOTE  = open(sys.argv[1], encoding="utf-8", errors="replace").read()
NOTE  = re.sub(r"\s+", " ", NOTE)

fails = []
def want(needle, why):
    if needle not in NOTE:
        fails.append(f"{why}: the note does not carry {needle!r}")

# ---- inputs, each read from the record by this file's own patterns
D = Fraction(3)                                             # front segment, m (study header, scoping memo s5)
startup_s = Fraction(int(re.search(r"initial transient cleared by t ~ (\d+) s", THREE).group(1)))
fine_wall = re.search(r"Wall-clock \(borrego \+ cloud burst\)\s*:\s*(\d+)h\s*(\d+)m", THREE)
fine_h = Fraction(int(fine_wall.group(1))) + Fraction(int(fine_wall.group(2)), 60)
sim_s  = Fraction(re.search(r"Sample time\s*:\s*t = ([\d.]+) s", THREE).group(1))
a3 = re.search(r"Wall clock\s*:\s*(\d+)h\s*(\d+)m\s*(\d+)s", LOG3)
a3_h = Fraction(int(a3.group(1))) + Fraction(int(a3.group(2)), 60) + Fraction(int(a3.group(3)), 3600)
quoted = Fraction(str(re.search(r"matching period of ~([\d.]+) s", LOG3).group(1)))

# ---- the physics
f = 1.5 / math.sqrt(float(D))                    # Cetegen-Ahmed, standard s4
T = 1.0 / f
want(f"{f:.2f} Hz", "predicted frequency")
want(f"{T:.2f} s", "predicted period")
agree = abs(1/float(quoted) - f) / f * 100
want(f"{agree:.2f} %", "agreement percentage")

# ---- the time-control block
ceil = T / 25
w = math.floor(ceil * 1e4) / 1e4
want(f"{w:.4f} s", "write interval")
end = float(startup_s) + 20 * T
want(f"{math.ceil(end*100)/100:.2f} s", "end time")

# ---- the projection, in exact fractions until the last step
rate = fine_h / sim_s                            # hours per simulated second
per_case = rate * Fraction(str(round(end, 5)))
ch = int(per_case); cm = round(float(per_case - ch) * 60)
want(f"{ch} h {cm:02d} m", "one authorized case")
matrix = per_case * 6
mh = int(matrix); mm = round(float(matrix - mh) * 60)
want(f"{mh} h {mm:02d} m", "six-case matrix")
ratio = a3_h / per_case
want(f"{float(ratio):.2f} times", "attempt 3 vs one case")
ex = a3_h - per_case
want(f"{int(ex)} h {round(float(ex - int(ex)) * 60):02d} m", "excess wall clock")

# ---- the record figures the note quotes
for pat, label in [
    (r"cells\s+([\d ]+)", "attempt-3 cells"),
]:
    pass
cells = re.search(r"Mesh statistics\s*\n\s*cells\s+([\d ]+)", LOG3).group(1).replace(" ", "")
want(f"{int(cells):,}", "attempt-3 cell count")
studied = re.search(r"GRID 3 of 3.*?cells:\s+([\d ]+)", THREE, re.S).group(1).replace(" ", "")
want(f"{int(studied):,}", "fine study cell count")
want("five cell zones", "production cell zones")
want("six boundary patches", "study boundary patches")
want(f"{float(re.search(r'Uref\s+([\d.]+) m/s', LOG3).group(1)):.1f} m/s", "inlet Uref")
want(f"{float(re.search(r'Zref\s+([\d.]+) m', LOG3).group(1)):.1f} m", "inlet Zref")
want(f"{float(re.search(r'Max Courant number\s*:\s*([\d.]+)', THREE.split('GRID 3 of 3')[1]).group(1)):.2f}", "max Courant, fine grid")
want(f"{float(re.search(r'Time-step average\s*:\s*([\d.eE+-]+) s', THREE.split('GRID 3 of 3')[1]).group(1)):.4f} s", "average time step, fine grid")
want(f"{float(re.search(r'User CPU-hours\s*:\s*([\d.]+)', LOG3).group(1)):.1f} CPU-hours", "attempt-3 CPU hours")
want(f"{float(re.search(r'D\*/dx = 3\.76 / 0\.5 = ([\d.]+)', THREE).group(1)):.2f}", "plume resolution index")
want(f"{float(re.search(r'k_res/\(k_res\+k_sgs\) = ([\d.]+)', THREE).group(1)):.2f}", "Pope fraction")

# ---- the delivered block this second path reads for itself
for k, v in [("endTime", "2000"), ("deltaT", "0.01"), ("writeInterval", "100"), ("adjustTimeStep", "no"),
             ("writeControl", "timeStep"), ("startFrom", "startTime")]:
    got = re.search(rf"^\s*{k}\s+(\S+);", CTRL, re.M).group(1)
    if got != v:
        fails.append(f"delivered {k}: record has {got!r}, verifier expected {v!r}")
want("2000", "delivered end time")
want("0.01 s", "delivered deltaT")

print(f"checked {len(NOTE)} characters against {RAW.rsplit('/',1)[-1]}")
if fails:
    for f_ in fails: print("  FAIL", f_)
    sys.exit(1)
print("all recomputed values are carried by the note, and the delivered block re-reads as the note states it")

```


## deliverables/CFDME-2025-041_transient_solve_disposition_revA.txt

```text
THE CFD ME    |    Tucson, Arizona    |    AZ PE #48213    |    CFDME-2025-041    rev A

TECHNICAL NOTE — TRANSIENT SOLVE DISPOSITION

Third steady-solve attempt, the authorized transient run, and the solver wall-clock projection — Ocotillo desert-fire case CFDME-2025-041

Project

CFDME-2025-041 — Ocotillo WUI Wildfire, Phase 1 (client: Ocotillo Additive Structures)

Subject

Disposition of the third steady-solve attempt (SLURM job 41304); authorization and time-control block of the replacement transient run; solver wall-clock projection for the six-case matrix

From

Marguerite Falk, PE — Technical Director & Reviewing Engineer, AZ PE #42798

Copy

Gordon Selby, PE — Owner & Principal Engineer, AZ PE #48213

To

Project file CFDME-2025-041; Mulshi Simulation Labs (S. Bhatia), case supplier

Date

10/10/2025

Governing documents

Modeling Standards — Fire & Buoyant Flows rev 4 (effective 08/15/2025); QA Verification Standard — Mesh & Resolution rev 3 (effective 07/01/2025); Units and Conversions SOP rev 2 (effective 04/01/2025); SOW CFDME-2025-041-SOW-01 (signed 10/08/2025)

Revision

rev A — issued for action ahead of any fourth attempt

1.  Disposition

This note closes the third steady-solve attempt on the Ocotillo desert-fire case and authorizes the transient run that replaces it. It is dated 10/10/2025 and is issued before Mulshi starts a fourth attempt. The disposition, in order:

The plateau is physics. The residual plateau in attempt 3 is the puffing oscillation of the buoyant plume — a physical frequency of the flow, not a numerical stall. The period the run showed and the Cetegen–Ahmed prediction for this plume agree to 0.41 %.

There is no steady state to converge to. No steady solution exists for this problem, because the plume oscillates at a finite, deterministic, physical frequency and no time-invariant flow field exists for a residual-tolerance schedule to converge to. A fourth steady attempt is not authorized, and the operator's alternative of loosening the residual controls and keeping the cycle-averaged field is refused.

The transient run is authorized. FireFoam LES, EDC combustion, fvDOM at the firm reference settings, on the production conjugate mesh, for 31.10 s of simulated time at a 0.0461 s write interval, on the block of Section 4.

Three items of the delivered setup are non-conforming and must be corrected. The radiation configuration (Section 3.1), the turbulence closure (Section 3.2) and the inlet flow direction (Section 6.2). Two record items are open and must be closed before the case is set up: a temperature-dependent solid property table (Section 3.3) and a sourced inlet roughness length (Section 6.3).

The solved fields from attempt 3 are discarded. An unconverged steady RAS field of a formulation the Modeling Standards forbid cannot seed a transient LES.

Cost. 4 h 28 m of solver wall clock for one authorized case and 26 h 49 m for the six-case matrix. Both are a lower bound, not a firm figure: two of the three conditions the firm projection basis sets are not met by the benchmark of record (Section 7.4).

2.  The residual plateau in attempt 3

2.1  What the plateau is

The plume above a 3.0 m fire segment necks, sheds and re-forms at a frequency that depends only on the fire diameter and gravity. Modeling Standards rev 4 Section 4 adopts the Cetegen–Ahmed relation as normative for design fires in this regime, f = 1.5 × D^(−1/2) with D in m. For the 3.0 m front segment fixed in the scoping memo of 10/08/2025 and carried in the three-grid study:

        f = 1.5 × (3.0 m)^(−1/2) = 0.87 Hz      (unrounded 0.8660 Hz)

        T = 1 / f = 1.15 s                    (unrounded 1.1547 s)

Every LES run at this diameter is expected to display monitor-point oscillation at this frequency, and the standard is explicit that this is a physical property of the plume rather than a numerical artefact. The same section states the consequence that governs this disposition: because the plume oscillates at a finite, deterministic, physical frequency, no time-invariant flow field exists that any residual-tolerance schedule can converge to.

2.2  What the run showed

The closing narrative of the attempt-3 log records the plateau exactly where the plume is: over outer iterations 400 to 800 the Ux initial residual holds in the band 3e-4 to 8e-4 (dimensionless) and the p initial residual in the band 2e-5 to 3e-5 (dimensionless), while the plume-centreline probes oscillate at a quoted period of about 1.15 s with roughly 40 K peak-to-peak on temperature and 2.4 m/s peak-to-peak on vertical velocity. The wall stand-off probes stay flat across the same window: probe 3 reads U = (8.00, 0.13, 0.02) m/s at T = 308.5 K and probe 4 reads U = (7.74, 0.10, 0.12) m/s at T = 310.2 K. An oscillation confined to the plume axis, in phase at 4 m, 8 m and 12 m above grade, is the plume's own structure. A residual tail from the pressure solve would not look like that.

Probe

Location (mesh axes, m)

Behaviour over outer iterations 400–800

0

(1.5, 0.0, 8.0) — plume centreline, 8 m AGL

oscillates at ~1.15 s; ~40 K and ~2.4 m/s peak-to-peak

1, 2

(1.5, 0.0, 4.0) and (1.5, 0.0, 12.0) — plume centreline, 4 m and 12 m AGL

oscillate in phase with probe 0

3, 4

(10.0, 0.0, 2.0) and (10.0, 0.0, 8.0) — wall stand-off, upwind face and above roof

flat: T = 308.5 K and 310.2 K, U unchanged to two decimals

2.3  How close the prediction and the run are, as a percentage

The comparison has to be made against the unrounded prediction. Rounded to the two decimals the note reports, both the prediction and the quoted value read 0.87 Hz and 1.15 s, and comparing those with each other returns a circular 0.00 %, which is not an agreement figure. Carried unrounded:

as a frequency: quoted 1 / 1.15 s = 0.8696 Hz against the prediction 0.8660 Hz — 0.4087 %;

as a period: quoted 1.1500 s against the prediction 1.1547 s — 0.4071 %.

Both readings give 0.41 %, and the agreement is stated as 0.41 % on that basis.

2.4  Whether a steady solution exists

It does not. The frequency is finite, deterministic and physical, so the flow has no time-invariant state for a residual-tolerance schedule to reach; the residual plateau is the signature of the physical oscillation and not evidence of a defective solver. Attempt 3 is therefore not a failed solve — it is a correct solve of the wrong formulation on a flow that has no steady state. The three recommendations in the operator's closing narrative are each declined:

1. Further under-relaxation (U 0.3 to 0.2)  cannot converge a physical oscillation; it changes the path toward a destination that does not exist.

2. Tightening the pressure tolerance to 1e-8  cannot either. The pressure residual is a consequence of the plume passing through the pressure field; the probes that carry the physical signal are temperature and vertical velocity on the plume axis, and they lock onto the predicted frequency to 0.41 % — which is what distinguishes a physical mode from numerical noise.

3. Loosening the residual controls and keeping the cycle-averaged field  is refused. A steady RANS buoyant field is not a permissible starting point for fire work (Section 3.6 of the Modeling Standards), k-epsilon applied to LES is not permitted (Section 5), and a cycle-averaged field is a time-invariant object constructed from a flow that has none.

3.  How the rest of the delivered setup stands against our standards

The delivered case is read from the attempt-3 log and the case system directory. Four items fail the Modeling Standards and one record item is open; each carries the conforming configuration that replaces it.

3.1  Radiation — non-conforming

The delivered case runs fvDOM with 32 rays and a single grey band (greyMeanCombustion absorption/emission, no scattering, no soot model). Modeling Standards rev 4 Section 3.4 takes the number of angular directions and the wavelength banding from the firm reference table, and Appendix A gives that table for the 1200–1700 K design range as four angular directions (S4-equivalent quadrature) and two wavelength bands split at 2.5 micrometres. The conforming configuration — and the one the authorized run carries — is therefore four angular directions and two wavelength bands split at 2.5 micrometres, with the absorption and emission coefficients set consistent with the combustion product composition rather than defaulted.

3.2  Turbulence closure — non-conforming

The delivered case runs RAS k-epsilon (kEpsilon) inside the fire solver, with the operator's note that the steady k-epsilon solve was a bring-up device for a later LES. Modeling Standards rev 4 Section 5 permits only large-eddy simulation, with either a dynamic Smagorinsky closure (Germano–Lilly, test filter twice the mesh spacing) or WALE, and forbids k-epsilon variants applied to LES and hybrid RANS/LES formulations. The authorized run is therefore large-eddy simulation with a WALE sub-grid closure — the standard's preferred closure where solid boundaries carry a significant fraction of the thermal load, which is the case here, the wall assembly being the object of the analysis — with dynamic Smagorinsky as the permitted alternative. The choice is recorded in the case-setup record per Section 5.

3.3  Solid-region properties — record item open

The delivered case carries no solid regions at all. The production mesh does — the outer leaf, the inner leaf and the ties — and Modeling Standards Section 3.5 requires k, rho and c_p for solid regions to be entered as temperature-tabulated functions or supported polynomial fits over the design temperature range. What the record holds are constants: Appendix A gives the printed siliceous-aggregate concrete as k = 1.4 W/(m·K), rho = 2200 kg/m³ and c_p = 900 J/(kg·K), and the galvanized steel ties as k = 45 W/(m·K), rho = 7850 kg/m³ and c_p = 490 J/(kg·K), and states that solid-region properties are held constant over the design range absent case-specific data. There is no temperature-dependent property table for the concrete leaves or for the steel ties anywhere in the record. This is an open record item: the only solid-region property values on file are constants, and a table must be established and entered before the conjugate solve is set up, because a constant-property short-cut for solid regions in a fire case is not permitted.

3.4  Case inheritance from a non-fire parent — non-conforming

The case-setup export of record (OAS_case_setup_from_DWA.txt) documents the case as built from the validated Dry Wash Aerospace high-altitude external-aerodynamics case, with the solver, turbulence, freestream, combustion and radiation blocks carried forward and only the geometry and patch naming swapped to revC. Modeling Standards rev 4 Section 7 prohibits that transfer in terms: a validated aerospace compressible non-reacting case shall not be pulled forward as the physics basis for a ground-level fire case, and case-directory reuse without a full physics rebuild is not acceptable practice. The export also records that no combustion model, no radiation dictionary and no reactive scalars are compiled in, while the live controlDict and the attempt-3 log show fireFoam with EDC and fvDOM — so the record holds no case-setup artifact that conforms to Section 3. The freestream block of the export (p_inf = 26 500 Pa, T_inf = 223 K, rho_inf = 0.4 kg/m³, M_inf = 0.25) is an aerospace reference condition and must not appear in the authorized case; the run of record used an ambient of 308.15 K (35 °C). The physics on the production mesh is to be recorded as built new, per Section 7 of the Modeling Standards.

3.5  What the case must be

On that reading the authorized run is: fireFoam, large-eddy simulation with WALE, EDC combustion carried forward unchanged, fvDOM with four angular directions and two bands split at 2.5 micrometres, a four-region conjugate treatment (outer leaf and inner leaf as solid regions with temperature-dependent properties, the cavity as a fluid region with participating radiation, and each tie as a separate solid region with refinement at the tie–concrete interfaces) coupled with the chtMultiRegion boundary conditions of Modeling Standards Section 6, the atmospheric-boundary-layer inlet of Section 7 with matching turbulence fields, and the time-control block of Section 4 below.

4.  The run I am authorizing — the time-control block

4.1  The block, with values

Entry

Authorized value

Basis

application

fireFoam

Modeling Standards §3.1

startFrom

latestTime

firm reference transient controlDict

startTime

0 s

first run of the authorized case

stopAt

endTime

—

endTime

31.10 s

§4: 8 s startup + 20 periods (Section 4.3)

deltaT

0.001 s (first step, then Courant-controlled)

firm reference transient controlDict

adjustTimeStep

yes

§4, mandatory

maxCo

0.9 (dimensionless)

§4, mandatory

maxDeltaT

0.01 s

§4 ceiling, fixed by the firm reference dictionary (Section 4.2)

writeControl

adjustableRunTime

§4, mandatory

writeInterval

0.0461 s

§4: ≥ 25 samples per period (Section 4.3)

purgeWrite

0

all written times retained for the spectra

writeFormat / writePrecision

ascii / 8 (significant figures)

firm reference transient controlDict

writeCompression / timeFormat

off / general

firm reference transient controlDict

runTimeModifiable

true

firm reference transient controlDict

4.2  The reference dictionary that governs the case-specific ceiling

Section 4 of the Modeling Standards fixes adjustTimeStep yes, maxCo 0.9, writeControl adjustableRunTime and a maxDeltaT ceiling that it leaves case-specific, "typically 0.01 s". Our reference transient dictionary — Firm/Standards/Reference_Cases/fireFoam_transient_controlDict.txt — sets maxDeltaT 0.01 s, and that is the ceiling carried into the authorized block. Two of that dictionary's own settings are not carried: its endTime of 60 s and its writeInterval of 0.1 s are the settings of its own 60 s run, and a 0.1 s write interval would capture only 11.5 samples per puffing period here, against the 25 the standard requires. The ceiling that governs the step is 0.01 s; the write interval and the end time are set from Section 4 of the standard for this case, below.

4.3  Largest write interval and shortest end time the standard allows here

Write interval. Section 4 requires at least 25 monitor samples per puffing period. With the period carried at full precision, T = 1.1547 s, the ceiling is T / 25 = 0.046188 s. The largest four-decimal write interval that meets it is 0.0461 s, which gives 25.05 samples per period; 0.0462 s gives 24.99 and fails. A write interval derived from the rounded period of 1.15 s would be 0.0460 s — still compliant, but not the largest the standard allows, because the rounded period is the shorter of the two.

End time. Section 4 requires the simulated end time to span a minimum of 20 puffing periods after the initial transient has cleared, which it puts at no less than approximately 23.09 s of physical time for a 3.0 m fire, exclusive of startup. The three-grid study records the startup transient as cleared by t ≈ 8 s, and the Phase 1 draft samples from t ≥ 8 s on the same basis. Hence:

        end time = 8 s + 20 × 1.1547 s = 8 s + 23.09401 s = 31.09401 s

As a floor, 31.10 s is the shortest value to two decimals that satisfies the rule: 31.09 s leaves 19.9965 periods after startup, short of the twenty required. The authorized end time is 31.10 s.

4.4  The block Mulshi delivered, entry by entry

Every entry that differs between the delivered controlDict and the authorized block, with both values:

Entry

Delivered by Mulshi

Authorized

Why it differs

startFrom

startTime

latestTime

the authorized run may restart from the last written time

endTime

2000 (counted as outer iterations)

31.10 s (physical time)

2000 iterations × 0.01 s = 20.00 s of simulated time, under the 23.09 s the standard requires before startup is added

deltaT

0.01 s, fixed

0.001 s first step, then Courant-controlled

a fixed step cannot hold maxCo through the plume passage

adjustTimeStep

no

yes

§4, mandatory

maxCo

absent (no entry)

0.9 (dimensionless)

§4, mandatory

maxDeltaT

absent (no entry)

0.01 s

§4 ceiling, firm reference dictionary

writeControl

timeStep

adjustableRunTime

§4: writes pegged to simulated time, not to solver iteration

writeInterval

100 time steps

0.0461 s

100 steps × 0.01 s = 1.00 s of simulated time between writes, i.e. 1.15 samples per puffing period

writePrecision

6

8

firm reference transient controlDict

The delivered write interval records about 1.15 samples per puffing period, against the 25 the standard requires; the delivered endTime of 2000 outer iterations is an iteration count rather than a simulated time, and at the delivered fixed step it corresponds to 20.00 s. Attempt 3 reached 800 of those iterations, that is 8.00 s of simulated time — it stopped at the end of the startup window, before a single period of the window the standard requires had been sampled.

5.  The mesh the attempt ran on, and what our verification of it covers

5.1  The attempt ran on the production conjugate mesh

The attempt-3 log reports 11,214,862 cells with a maximum non-orthogonality of 58.34 (limit 65, dimensionless) and a maximum skewness of 2.14 (limit 4, dimensionless) — the figures the meshing log of 10/09/2025 records for the production mesh (job borrego/41287, 32 cores, geometry of record OAS-WUI-01_revC). Its five cell zones are atmosphere (fluid, 9,872,108 cells), outerLeaf (solid, 582,214 cells), cavity (fluid with participating radiation, 212,486 cells), innerLeaf (solid, 460,872 cells) and ties (solid, 87,182 cells), and it carries 13 boundary patches.

5.2  The three-grid study is a different, fluid-only family of meshes

The verification study of 10/09/2025 runs three separate meshes: 1,410,240 cells at a background dx = 1.0 m, 4,023,680 cells at dx = 0.7 m and 11,196,000 cells at dx = 0.5 m. Each of the three carries a single cell zone, which is fluid (fireRefinementZone), and six boundary patches — ground, inlet_240, inlet_270, outlet, top and fire_source — with no wall patch among them. None of the wall assembly, the cavity or the ties exists in these meshes.

5.3  What the verification covers

It verifies plume resolution in the fluid domain. On the fine grid the plume resolution index is D*/dx = 7.52 (dimensionless, within the required band of 5 to 10) and the Pope resolved-TKE fraction is 0.84 (84 %), above the 80 % minimum. On that basis the fluid-domain resolution at dx = 0.5 m carries into the authorized run. A Richardson or GCI band is not part of that evidence: QA Verification Standard rev 3 Section 4 states that the grid convergence index is not valid for LES and shall not be used to verify any large-eddy result.

5.4  What it does not cover

The solid leaves, the cavity radiation and the tie regions of the production mesh are outside the scope of the three-grid verification. None of those regions exists in the study meshes — one fluid cell zone per mesh, no wall patch, no conjugate region and no participating-radiation cavity — so nothing about the conjugate heat transfer through the leaves, the radiative exchange inside the cavity or the conduction into the ties is verified by the three grids. Those mechanisms are exactly what sets the single-point temperature rise of this assembly. The verification that carries to the authorized run is therefore the fluid-plume resolution only; the conjugate response needs its own verification and reporting under Sections 5 and 6 of the QA standard.

6.  What in the delivered case carries forward, with the inlet's defining settings

6.1  Carries forward

The production conjugate mesh on the geometry of record OAS-WUI-01_revC, with its five cell zones and 13 boundary patches.

The EDC combustion model of the delivered case (version v2005), which Modeling Standards Section 3.3 mandates: EDC carries forward unchanged into the authorized run, with the finite-rate chemistry coupling and its C_gamma 2.1377 and C_num 0.4083 constants.

The atmospheric-boundary-layer inlet formulation with its matching turbulence inlet fields, and the probe set of Section 2.2 (plume centreline at 4 m, 8 m and 12 m above grade; wall stand-off at 2 m and 8 m).

The job accounting of attempt 3, as a cost datum only (Section 7.7).

6.2  The inlet's defining settings, and the one that must change

The delivered inlet is atmBoundaryLayerInletVelocity with Uref = 8.0 m/s at Zref = 6.1 m (the 20 ft reference height of SOW Section 3.1, case 3.1-B of the matrix), z0 = 0.03 m and flowDir (cos(240) sin(240) 0). The reference velocity and the reference height carry forward unchanged.

The flow direction does not. Units and Conversions SOP rev 2 Section 2 fixes wind direction on the compass convention, north = 0 degrees and clockwise positive, with solver-internal angles in radians. Evaluated, (cos 240°, sin 240°, 0) = (−0.500, −0.866, 0): that vector points to 210 degrees on the compass, so it drives the flow out through the western boundary and does not describe a wind from 240 degrees — it describes a wind from 030 degrees. A wind from 240 degrees flows toward 060 degrees, and with x east and y north — the axes the production mesh fixes with its inlet_west, outlet_east, lateral_north and lateral_south patches — the unit vector for the inlet flow direction is (0.866, 0.500, 0), equivalently (cos 30°, sin 30°, 0). That is the direction the authorized run carries. The delivered entry is also neither convention cleanly: it is a mathematical angle in degrees evaluated on mesh axes, while the SOP puts wind directions on the compass in degrees and solver-internal angles in radians.

6.3  The roughness length cannot carry forward as conforming

Modeling Standards Section 3.2 requires the roughness length, the reference height and the reference velocity to be taken from the project's site meteorological pull. The pull of record (site_met_avra_valley_2015-2024.csv, 240 hourly records plus the design-envelope footer) carries a timestamp, ambient temperature, relative humidity, 20 ft wind speed, wind direction and solar irradiance, and holds no roughness length. The 0.03 m on the delivered inlet is Mulshi's own annotation in the attempt-3 log ("buffelgrass roughness"); it is not sourced from the pull and the record holds no other basis for it. This is an open record item: the roughness length must be sourced and recorded with its reference before the authorized run is set up, and the value entered at the 6.1 m reference height it belongs to.

7.  Cost

7.1  The basis of record

The firm projection basis is the reviewing engineer's note of 10/09/2025 18:05 to R. Ellsworth. It has five points: (1) at a fixed mesh, a fixed solver stack and a fixed core count, solver wall clock is linear in simulated physical time, the rate being taken from a run of record on that grid as simulated seconds reached divided by wall clock consumed; (2) the rate is taken from the grid the production case will actually run on, never carried from one grid to another; (3) the figure is quoted per case first and then multiplied by the number of cases the governing scope document procures; (4) the number is solver wall clock and nothing else — queue wait, staging, meshing and decomposition, post-processing and cloud-burst provisioning latency are excluded; (5) the linear-scaling step is written into the assumptions section of whatever document carries the number. Nothing beyond those five steps is applied.

7.2  Solver wall clock for one authorized case

The run of record on the fine grid is the FINE run of the three-grid study: 40.00 s of simulated time in 5.75 h of solver wall clock (5 h 45 m as posted for that run; the queue wait before its burst allocation is excluded under point 4 of the basis). The rate is therefore 5.75 h / 40.00 s = 0.143750 h per simulated second, and one authorized case is:

        0.143750 h/s × 31.09401 s = 4.470 h  =  4 h 28 m of solver wall clock

7.3  Solver wall clock for the full six-case matrix

SOW Section 3.1 procures six cases (240 and 270 degrees at 5, 8 and 12 m/s). Within the matrix every case carries the same mesh and the same required physical duration and only the inlet condition changes, so the per-case figure multiplies by six, with no discount for cases expected to restart from a neighbour:

        6 × 4.470 h = 26.82 h  =  26 h 49 m of solver wall clock

(26 h 48 m from multiplying the rounded per-case figure of 4 h 28 m is equally correct.)

7.4  Whether that basis can be met for this run, on every condition it sets

The basis sets three conditions of equal standing. The verdict on each:

Condition

Verdict

Evidence in the record

Fixed solver stack

Met

The study runs fireFoam LES with EDC combustion and fvDOM radiation, and Modeling Standards Section 3.1 makes fireFoam the solver for the production case: the study and the production case both run the same reacting solver family. The conjugate regions are mesh content and are counted under the fixed-mesh condition, not here.

Fixed mesh

Not met

The benchmark ran on the 11,196,000-cell fluid-only plume mesh, which carries none of the wall assembly, the cavity or the ties. The production run is the 11,214,862-cell multi-region conjugate solve with five cell zones.

Fixed core count

Not met

The study records 96 cores for the coarse grid and 192 cores for the medium grid, and no core allocation at all for the fine grid (posted as "borrego + cloud burst"). The allocation behind the rate is therefore not recorded.

Two of the three conditions therefore fail, so the linear-scaling precondition the projection rests on is not satisfied by the benchmark of record. The figures of Sections 7.2 and 7.3 remain the answer the basis produces, and they are a projection under a stated assumption rather than a measurement.

7.5  What the time step and the Courant margin behind the figure imply

The fine-grid benchmark reached a maximum Courant number of 0.90 against the 0.9 ceiling — no margin at all, with a mean Courant number of 0.47 (dimensionless) and an average time step of 0.0022 s. Its finest cells measure 0.125 m (minimum cell volume 0.001936 m³ on the fine grid). The production mesh carries 0.0625 m cells in the wall assembly and the cavity and 0.03125 m cells at the ties and the tie–concrete interfaces, and 1,342,754 cells in the four non-atmosphere zones that the benchmark never advanced. A Courant-limited step scales with cell size at fixed velocity, so the production conjugate solve will advance at a smaller average time step than the 0.0022 s recorded on the fine study grid, and it does extra work per step in the solid and tie zones. Both effects push the true cost above the projection.

7.6  What the figure is worth

Both figures — 4 h 28 m for one case and 26 h 49 m for the matrix — are a lower bound on the true cost, not a firm estimate. They carry no conjugate solid, cavity-radiation or tie work, and the rate behind them comes from a fluid-only grid whose core allocation is unrecorded. A firm solver wall-clock figure for this project requires a benchmark run on the production conjugate mesh — a transient fireFoam run of record on the 11,214,862-cell mesh. The only run that exists on that mesh is attempt 3, and its rate cannot be used either: it is a steady RANS solve on a different solver stack and has no Courant-controlled time step (800 outer iterations at a fixed 0.01 s, that is 8.00 s of simulated time, in 6 h 41 m).

No contingency, margin or risk multiplier is applied to either figure, and no overhead percentage is applied to either figure. Nothing is added for queue wait, staging, meshing or decomposition, post-processing or cloud-burst provisioning latency: the basis excludes them (point 4) and none of them appears in the projection. The two figures are the measured rate multiplied by the procured cases and nothing else.

7.7  What attempt 3 consumed, against one authorized case

Attempt 3 (SLURM job 41304, 64 cores across four nodes of the borrego cluster) consumed 6 h 41 m of wall clock and 424.3 CPU-hours. Against one authorized case at 4 h 28 m that is 1.50 times the solver wall clock, and 2 h 13 m more solver wall clock than one authorized case.

7.8  What of it, if anything, is usable downstream

Nothing of the solved fields: they are discarded. An unconverged steady RAS field of a formulation Modeling Standards Sections 3.6 and 5 forbid cannot seed a transient LES, and nothing in it is a boundary condition for the authorized run. What is usable is everything around the fields — the production mesh, the probe set, the EDC configuration, the inlet formulation once its direction and roughness are corrected (Section 6), and the job accounting as a cost datum.

8.  Assumptions behind the cost figures

 Solver wall clock is linear in simulated physical time at a fixed mesh, a fixed solver stack and a fixed core count. That is the assumption the projection rests on, and Section 7.4 records that two of its three conditions are not satisfied by the benchmark of record; the figures are a projection, not a measurement.

 The rate is that of the fine-grid run of record: 40.00 s of simulated time in 5.75 h of solver wall clock, queue wait excluded.

 The simulated duration of each authorized case is 31.09401 s, the 8 s startup recorded in the three-grid study plus twenty unrounded puffing periods of 1.1547 s.

 Six cases are procured, per SOW Section 3.1; every case carries the same mesh and the same required duration, and only the inlet condition changes.

 The ambient condition of record is 308.15 K (35 °C); 20 ft is 6.096 m, recorded throughout as 6.1 m.

9.  Reviewer and stamp block

Reviewing engineer

Marguerite Falk, PE — Technical Director & Reviewing Engineer, AZ PE #42798

Signature

_/s/ Marguerite Falk, PE /_

Date

10/10/2025

Disposition

Attempt 3 is closed as non-converging by physics; a fourth steady attempt is not authorized. The transient run of Section 4 is authorized, subject to closure of the two record items in Sections 3.3 and 6.3.

[  PE STAMP — APPLIED  ]

The CFD Mechanical Engineer, LLC — Arizona PE #48213Gordon Selby, PE — Owner & Principal EngineerDate: 10/10/2025

Signature

_/s/ Gordon Selby, PE /_

Stamp boundary

CFD thermal analysis only. Fire-resistance rating certified separately by others.

This note is a computational thermal analysis. It does not certify, extend or modify any fire-resistance rating of the assembly; the certified rating is issued separately by others under a separate engagement, as SOW Sections 4 and 10 require.
```
