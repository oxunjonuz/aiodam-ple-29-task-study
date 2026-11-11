# les-mesh-resolution

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-24 — les-mesh-resolution
   On the grid the report actually rests on, the wake is resolved at 29.9 — not 7.52 — and the acceptance
   survives only through the Pope criterion.

 Judge score                                                NOT MEASURED — no judge endpoint or key

 Criteria · weight                                          14 criteria · 82 weight points

 My criteria reader (not a score)                           14 of 14 criteria · 82 of 82 weight

 Breakage campaign                                          19/19

 File                                                       /work/ple-solve/les-mesh-resolution/out/CFDME-2025-
                                                            041_resolution_note_revA.docx · 43,314 B · sha256 a0155983ffa83ae4…


TASK AND INPUTS
Write the note on the LES mesh-resolution basis for the same fire case. Fourteen criteria, weight 82.

The three-grid study, the case setup, the QA standard, the met data and the run logs.

WHAT WAS AMBIGUOUS OR BROKEN
    The wake-resolution index on the fine grid of the three-grid study is 29.9 against a reported 7.52: D* = 3.74 m (the record says 3.76
    — a last-step error, since 27.08^(2/5) = 3.742), the fire cell is 0.125 m (two snappy levels over a 0.500 m background; confirmed by a
    minimum volume of 0.001936 m³ → 0.1246 m and a zone volume of ≈2,454 m³ on all three grids), and the 5–10 band is missed by a
    factor of three.
    Acceptance rests on Pope alone (0.84 ≥ 0.80). §5 allows either criterion, so no escalation is needed and no decision on Marguerite's
    edge length is required; requiring both is the default only for AHJ or life-safety cases, which this phase is not.
    The refinement zone's radius and height are recorded nowhere — the volume fixes only πr²h — so the probe landing at 2.81 m inside
    the zone is an inference.
    The ambient behind D* is the reference atmosphere (1.204 kg/m³, 293.15 K), not the site (35–42 °C, maximum 41.8 °C in the met
    export), and D* at site conditions cannot be computed from the files: there is no pressure.
    Checklist item 4 stays open: QA §3(a) requires naming the SGS closure, and it is in no file (the only solver log is a steady k-ε bring-
    up; the only case export is rhoSimpleFoam + kOmegaSST from DWA). Closing it needs a case-setup memo — the Mixed-convection
    LES setup memo, SOW §4 deliverable 2 — not a new run.

WHAT I DID AND WHAT CAME OUT
    A note that separates what the record supports from what it implies, and states the one open item with the deliverable that closes it.
    14 of 14 criteria and 82 of 82 weight by my reader on the PDF twin.

HOW IT WAS VERIFIED
    Reader on the PDF twin; 218 tokens, 0 unexplainable; an independent recomputation of 12 quantities in Decimal.
    Nineteen breakages — all noticed, plus one harmless control that was correctly not noticed.
    Render completeness: 0 of 105 paragraphs and cells lost.
    The official test.sh chain with a stub judge: 1.0 by construction; negative controls on the same path: 0.914634 (c0, weight 7) and
    0.890244 (c3, weight 9).

MY MISTAKES IN THIS TASK
    The number audit initially accepted an invented number as a date if a time appeared within 40 characters — a control with 7,300
    went unnoticed; the window was narrowed to the token itself.
    Three mutations were marked incomplete after I rebuilt a table, so the pairs no longer applied.
    c12 stayed green when “8,760” was removed because “9,125” remained — but the rubric accepts either counter, so the rubric was
    right and my reader was too narrow.




                                                                 page 35 of 48
PLE 29-task report · AIODAM                                                                                                 11 November 2026




WHAT REMAINS UNVERIFIED
No judge score. The docx was not rendered inside the verifier image; the PDF twin was produced by the official transform.py locally.

USE OF REFERENCE OR GRADER MATERIAL
None.


PART C · RUBRIC · ELECTRICAL / INTERCONNECTION
```


## code/src/audit_numbers.py

```text
#!/usr/bin/env python3
"""Number audit: every figure in the note must be derivable from the record or from the recomputation.

A different mechanism from checks.py: it does not look for phrases at all. It pulls every
number out of the rendered note and asks whether that number exists in the files on the file
server (all of them, text-extracted) or is one of the recomputed values in computed.json.
Anything else is reported with its context.

    python3 audit_numbers.py out/note_pdf.txt [--control]
"""
from __future__ import annotations

import csv
import json
import re
import sys
from pathlib import Path

BASE = Path("/work/ple-dataset/tasks/les-mesh-resolution/environment/files")
C = json.loads(Path("/work/les/computed.json").read_text())


def norm(t: str) -> str:
    t = t.replace("\u00b3", "3").replace("\u00b2", "2")
    t = t.replace("\u2013", "-").replace("\u2014", "-").replace("\u2212", "-")
    t = re.sub(r"-\s*\n\s*", "", t)
    return re.sub(r"\s+", " ", t)


def canon(v: str) -> str:
    """canonical form of a numeric token: no thousands separators, trailing zeros kept."""
    return v.replace(",", "").replace(" ", "")


def extract_numbers(text: str) -> list[tuple[str, str]]:
    out = []
    for m in re.finditer(r"\d{1,3}(?:,\d{3})+(?:\.\d+)?|\d+(?:\.\d+)?", text):
        tok = m.group(0)
        ctx = text[max(0, m.start() - 55):m.end() + 20]
        out.append((tok, ctx))
    return out


def record_corpus() -> set[str]:
    """every numeric token in every file on the file server, canonically."""
    nums: set[str] = set()
    docs = []

    def add(t: str) -> None:
        for m in re.finditer(r"\d{1,3}(?:,\d{3})+(?:\.\d+)?|\d+(?:\.\d+)?", t):
            nums.add(canon(m.group(0)))
            # a space-grouped thousands form ("11 214 862") also counts as its joined value
            nums.add(canon(m.group(0)).rstrip("."))

    for p in sorted(BASE.rglob("*")):
        if not p.is_file():
            continue
        suf = p.suffix.lower()
        try:
            if suf == ".pdf":
                import pymupdf
                d = pymupdf.open(p)
                docs.append("\n".join(pg.get_text() for pg in d))
            elif suf == ".docx":
                import docx
                d = docx.Document(p)
                parts = [q.text for q in d.paragraphs]
                for t in d.tables:
                    for row in t.rows:
                        parts.append(" ".join(c.text for c in row.cells))
                docs.append("\n".join(parts))
            elif suf == ".xlsx":
                import openpyxl
                wb = openpyxl.load_workbook(p, data_only=True)
                parts = []
                for ws in wb.worksheets:
                    for row in ws.iter_rows(values_only=True):
                        parts.append(" ".join(str(v) for v in row if v is not None))
                docs.append("\n".join(parts))
            elif suf in (".txt", ".csv", ".eml", ".json", ".md"):
                docs.append(p.read_text(errors="replace"))
            elif suf == ".fcstd":
                continue
        except Exception as exc:                     # a file the audit cannot read is stated, not ignored
            print(f"  [audit] could not read {p.name}: {exc}")
    joined = norm("\n".join(docs))
    # space-grouped thousands ("11 214 862") -> also their joined form
    for m in re.finditer(r"\d{1,3}(?: \d{3})+", joined):
        nums.add(m.group(0).replace(" ", ""))
    add(joined)
    return nums


def recomputed() -> dict[str, str]:
    """the values this analysis computed, in the form the note prints them."""
    vals = {
        "3.742": C["Dstar_3dp"], "3.7423": C["Dstar_m"], "3.74": C["Dstar_reported"],
        "29.94": round(C["index"], 2), "29.9": C["index_1dp"], "29.938": round(C["index"], 3),
        "0.125": C["dx_fire"], "0.1246": C["dx_fire_minvol_4dp"], "0.001936": C["min_vol_fine"],
        "2.81": C["probe_height_2dp"], "27.09": round(C["ratio"], 2), "27.08": C["ratio_record"],
        "2450": C["zone_vol_by_cells"][0], "2455": C["zone_vol_by_cells"][1], "2454": C["zone_vol_by_cells"][2],
        "2453": round(C["zone_vol_mean"]), "3.61": C["zone_r_min_in_domain_2dp"],
        "16.7": C["zone_r_if_h_is_probe_1dp"], "5.6": C["zone_r_over_segment_1dp"],
        "30.08": round(C["index_with_record_D"], 2), "30.1": C["index_with_record_D_1dp"],
        "7.52": C["index_on_background"], "3.0": C["index_over_band_1dp"],
    }
    return {k: str(v) for k, v in vals.items()}


def audit(path: Path) -> tuple[int, list]:
    text = norm(path.read_text(errors="replace"))
    corpus = record_corpus()
    mine = set(recomputed())
    # physical constants this note uses, each with an external source (not project data):
    # 101 325 Pa = 1 standard atmosphere, 10th CGPM 1954 -- src_cb451907e479
    constants = {"101325"}
    rows, unknown = [], []
    for tok, ctx in extract_numbers(text):
        c = canon(tok)
        if c in corpus or c in mine or c in constants or c.rstrip("0").rstrip(".") in corpus or c in ("1", "2", "3", "4", "5"):
            rows.append((tok, "record" if c in corpus else "recomputed", ctx))
            continue
        unknown.append((tok, ctx))
    return len(rows), unknown


def main() -> int:
    path = Path(sys.argv[1])
    ok, unknown = audit(path)
    print(f"numeric tokens resolved: {ok}")
    print(f"unresolved: {len(unknown)}")
    for tok, ctx in unknown[:25]:
        print(f"  {tok!r:>12}  ...{ctx}...")
    if "--control" in sys.argv:
        # the audit must not be vacuous: three invented numbers, injected into a copy
        import tempfile
        t = norm(path.read_text(errors="replace"))
        t2 = t.replace("2,450", "2,477").replace("3.61", "3.93").replace("101,325", "98,742")
        with tempfile.NamedTemporaryFile("w", suffix=".txt", delete=False) as fh:
            fh.write(t2)
            tmp = Path(fh.name)
        _, ctrl = audit(tmp)
        print(f"\ncontrol (three numbers replaced): {len(ctrl)} unresolved")
        for tok, ctx in ctrl[:6]:
            print(f"  {tok!r:>12}  ...{ctx}...")
        if len(ctrl) < 3:
            print("CONTROL FAILED: the audit did not notice invented numbers")
            return 1
    return 0 if not unknown else 1


if __name__ == "__main__":
    sys.exit(main())

```


## code/src/build_note.py

```text
#!/usr/bin/env python3
"""Build the LES resolution note (CFDME-2025-041, rev A) from computed.json.

Every number in the document is interpolated from the recomputation in compute.py;
nothing numeric is typed here except the fixed document furniture (dates, section
numbers, standard section references).
"""
from __future__ import annotations

import json
import os
from pathlib import Path

import docx
from docx.enum.section import WD_SECTION
from docx.enum.table import WD_TABLE_ALIGNMENT
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml.ns import qn
from docx.shared import Pt, Inches, RGBColor

MUTATIONS = {
    # M01 the record's own slip: dividing by the background cell
    "M01": [("0.125 m", "0.500 m")],
    "M02": [("3.74 m", "3.76 m")],
    "M03": [("the band is not met", "the band is met"),
            ("criterion is therefore not met", "criterion is therefore met"),
            ("The plume resolution criterion is not met", "The plume resolution criterion is met"),
            ("and it is not the quantity", "and it is the quantity")],
    "M04": [("rests on the resolved turbulent kinetic energy criterion alone",
             "rests on the plume resolution criterion alone")],
    "M05": [("requires no escalation to you as Reviewing Engineer", "requires escalation to you as Reviewing Engineer")],
    "M06": [("neither its radius nor its height", "both its radius and its height")],
    "M07": [("are reference-atmosphere values", "are site design values"),
            ("standard reference-atmosphere conditions", "standard site design conditions")],
    "M08": [("cannot be evaluated from the files on the file server", "can be evaluated from the files on the file server")],
    "M09": [("remains open", "can now be closed")],
    "M10": [("Mixed-convection LES setup memo", "setup record")],
    "M11": [("without any new simulation run", "only after a new simulation run")],
    "M12": [("No ruling from you as Reviewing Engineer is needed", "A ruling from you as Reviewing Engineer is needed")],
    "M13": [("does not apply to Phase 1 rev B", "applies to Phase 1 rev B")],
    "M14": [("29.9", "7.5")],
    "M15": [("101,325 Pa", "101,325")],
    "M16": [("2.81 m", "3.76 m")],
    "M17": [("two additional snappy refinement levels", "one additional snappy refinement level"),
            ("0.125 m", "0.250 m")],
    "M18": [("84 %", "72 %")],
    "M19": [("5 to 10", "5 to 40")],
    "M20": [("10/10/2025", "10/11/2025")],
}
MUT = os.environ.get("MUT", "")


def mut(text):
    for a, b in MUTATIONS.get(MUT, []):
        text = text.replace(a, b)
    return text


C = json.loads(Path("/work/les/computed.json").read_text())
OUT = Path("/work/les/out/CFDME-2025-041_resolution_note_revA.docx")


def f(x, nd=0):
    """Number with thousands separators."""
    if nd:
        return f"{x:,.{nd}f}"
    return f"{x:,.0f}"


n = {
    "Q_MW": f"{C['Q_MW_study']:.1f}",
    "rho": f"{C['rho_inf']:.3f}",
    "T": f"{C['T_inf_K']:.2f}",
    "cp": f"{C['cp_inf']:.0f}",
    "g": f"{C['g']:.2f}",
    "ratio": f"{C['ratio']:.2f}",
    "ratio_rec": f"{C['ratio_record']:.2f}",
    "D3": f"{C['Dstar_3dp']:.3f}",
    "D2": f"{C['Dstar_reported']:.2f}",
    "D_rec": f"{C['claimed_Dstar_m']:.2f}",
    "D_rec_to_D": f"{C['ratio_to_D']:.3f}",
    "dx_bg": f"{C['dx_bg_fine']:.3f}",
    "dx_fire": f"{C['dx_fire']:.3f}",
    "dx_minvol": f"{C['dx_fire_minvol_4dp']:.4f}",
    "minvol": f"{C['min_vol_fine']:.6f}",
    "zone_cells": f(C["grids"][2]["zone_cells"]),
    "zone_cells_1": f(C["grids"][0]["zone_cells"]),
    "zone_cells_2": f(C["grids"][1]["zone_cells"]),
    "v1": f(C["zone_vol_by_cells"][0], 0),
    "v2": f(C["zone_vol_by_cells"][1], 0),
    "v3": f(C["zone_vol_by_cells"][2], 0),
    "vmean": f(C["zone_vol_mean"], 0),
    "index": f"{C['index_1dp']:.1f}",
    "index_2dp": f"{C['index']:.2f}",
    "index_recD": f"{C['index_with_record_D_1dp']:.1f}",
    "index_rec": f"{C['claimed_index_rounded']:.1f}",
    "index_rec2": f"{C['claimed_index']:.2f}",
    "index_bg": f"{C['index_on_background']:.2f}",
    "over": f"{C['index_over_band_1dp']:.1f}",
    "band_lo": f"{C['band_lo']:.0f}",
    "band_hi": f"{C['band_hi']:.0f}",
    "pope": f"{C['pope_fine']*100:.0f}",
    "pope_min": f"{C['pope_min']*100:.0f}",
    "probe_h": f"{C['probe_height_2dp']:.2f}",
    "probe_factor": f"{C['probe_factor']:.2f}",
    "r_min": f"{C['zone_r_min_in_domain_2dp']:.2f}",
    "r_disc": f"{C['zone_r_if_h_is_probe_1dp']:.1f}",
    "r_over_seg": f"{C['zone_r_over_segment_1dp']:.1f}",
    "seg": f"{C['segment_m']:.1f}",
    "site_lo": f"{C['site_T_lo_C']:.0f}",
    "site_hi": f"{C['site_T_hi_C']:.0f}",
    "met_hot": f(C["met_hot_hours"]),
    "met_max": f"{C['met_Tmax_C']:.1f}",
    "met_rows": f(C["met_rows"]),
    "elev": f"{C['nominal_elev_m']:.0f}",
    "rho_wind": f"{C['nominal_density_wind']:.1f}",
    "p_ref": f(C["press_ref_Pa"]),
    "prod_cells": f(C["prod_cells"]),
    "levels": f"{C['levels_above_bg']:.0f}",
    "domain_top": f"{C['domain_top_m']:.0f}",
    "I": f(C["I_kW_m"]),
}

doc = docx.Document()

# page and base style
sec = doc.sections[0]
sec.top_margin = Inches(0.8)
sec.bottom_margin = Inches(0.8)
sec.left_margin = Inches(0.9)
sec.right_margin = Inches(0.9)
st = doc.styles["Normal"]
st.font.name = "Calibri"
st.font.size = Pt(10.5)
st.paragraph_format.space_after = Pt(5)
st.paragraph_format.line_spacing = 1.05


def para(text, bold=False, size=None, space_after=5, align=None, italic=False, style=None):
    p = doc.add_paragraph(style=style)
    r = p.add_run(mut(text))
    r.bold = bold
    r.italic = italic
    if size:
        r.font.size = Pt(size)
    p.paragraph_format.space_after = Pt(space_after)
    if align is not None:
        p.alignment = align
    return p


def head(text):
    p = doc.add_paragraph()
    r = p.add_run(mut(text))
    r.bold = True
    r.font.size = Pt(11.5)
    p.paragraph_format.space_before = Pt(10)
    p.paragraph_format.space_after = Pt(3)
    return p


def bullet(text, lead=None):
    p = doc.add_paragraph(style="List Bullet")
    if lead:
        r = p.add_run(mut(lead))
        r.bold = True
    p.add_run(mut(text))
    p.paragraph_format.space_after = Pt(2)
    return p


def table(rows, widths=None, header=True, size=9.5):
    t = doc.add_table(rows=0, cols=len(rows[0]))
    t.style = "Table Grid"
    t.alignment = WD_TABLE_ALIGNMENT.CENTER
    for i, row in enumerate(rows):
        cells = t.add_row().cells
        for j, val in enumerate(row):
            cells[j].text = ""
            p = cells[j].paragraphs[0]
            r = p.add_run(mut(str(val)))
            r.font.size = Pt(size)
            if header and i == 0:
                r.bold = True
            p.paragraph_format.space_after = Pt(0)
            if widths:
                cells[j].width = Inches(widths[j])
    doc.add_paragraph().paragraph_format.space_after = Pt(2)
    return t


# ------------------------------------------------------------------ letterhead
para("THE CFD ME  |  Tucson, Arizona  |  CFDME-2025-041 rev A", bold=True, size=9.5,
     align=WD_ALIGN_PARAGRAPH.LEFT, space_after=2)
para("MEMORANDUM — LES RESOLUTION BASIS", bold=True, size=15, space_after=8)

table([
    ["To", "Marguerite Falk, PE — Technical Director & Reviewing Engineer (AZ PE #42798)"],
    ["From", "Gordon Selby, PE — Owner & Principal Engineer (AZ PE #48213)"],
    ["Project", "CFDME-2025-041 — Ocotillo WUI Wildfire, Phase 1"],
    ["Deliverable under review", "Phase 1 Report rev B; peer-review checklist Item 4 (resolution basis), currently open"],
    ["Date", "10/10/2025"],
    ["Subject", "The plume resolution index on the fine grid of the three-grid study, and what the resolution "
                "acceptance for Phase 1 rev B rests on"],
    ["Basis", "Documents on the file server as of 10/09/2025. No new simulation has been run for this note."],
], widths=[1.7, 4.9], header=False, size=9.5)

# ------------------------------------------------------------------ 1
head("1.  The answer, before the reasoning")
para(f"You have Item 4 open on the checklist and will not sign rev B until it closes, so the note leads with the "
     f"figures and then shows where each one comes from. On the grid the Phase 1 report rests on — the fine grid of "
     f"the three-grid mesh study, background cell size {n['dx_bg']} m — the plume resolution index is {n['index']}, "
     f"not the {n['index_rec']} the record carries. It is computed on a representative fire-region cell edge length of "
     f"{n['dx_fire']} m and a plume characteristic diameter of {n['D2']} m, and it is outside the band of "
     f"{n['band_lo']} to {n['band_hi']}: the band is not met. The resolution acceptance for rev B therefore rests on "
     f"the resolved turbulent kinetic energy criterion alone, whose {n['pope']} % on the plume axis clears the "
     f"{n['pope_min']} % minimum. Resting it there requires no escalation to you and no ruling from you on which cell "
     f"edge length the index takes. What keeps Item 4 open is something else: the sub-grid-scale closure of the "
     f"simulation that produced the reported monitors is not recorded anywhere on the file server, so the item "
     f"remains open, and it closes by recording the closure in the Mixed-convection LES setup memo — without any new "
     f"simulation run.")

table([
    ["Quantity", "Value", "Where it comes from"],
    ["Fire heat release rate, Q", f"{n['Q_MW']} MW", "Scoping memo §5.1"],
    ["Ambient density, rho_inf", f"{n['rho']} kg/m³", "Scoping memo §5.1 (reference atmosphere)"],
    ["Ambient temperature, T_inf", f"{n['T']} K", "Scoping memo §5.1 (reference atmosphere)"],
    ["Specific heat, cp", f"{n['cp']} J/(kg·K)", "Scoping memo §5.1"],
    ["Plume characteristic diameter, D*", f"{n['D2']} m", f"Recomputed here ({n['D3']} m); the record carries {n['D_rec']} m"],
    ["Background cell size, fine grid of the study", f"{n['dx_bg']} m", "Three-grid study, GRID 3 of 3"],
    ["Refinement of the fire-source zone above background", f"{n['levels']} levels", "Three-grid study, head note"],
    ["Representative fire-region cell edge length, dx", f"{n['dx_fire']} m",
     f"{n['dx_bg']} m divided by 2^{n['levels']}"],
    ["Plume resolution index, D*/dx", f"{n['index']}", f"{n['D3']} m / {n['dx_fire']} m"],
    ["Required band", f"{n['band_lo']} to {n['band_hi']}", "QA Verification Standard rev 3, §5.1"],
    ["Band met?", "No", f"{n['index']} is {n['over']} times the upper limit"],
    ["Pope resolved-TKE fraction, k_res/(k_res+k_sgs)", f"{n['pope']} %", "Fine-grid solver summary, on-axis probe"],
    ["Required minimum", f"{n['pope_min']} %", "QA Verification Standard rev 3, §5.2"],
    ["Pope criterion met?", "Yes", f"{n['pope']} % against {n['pope_min']} %"],
], widths=[2.5, 1.0, 3.1], size=9)

# ------------------------------------------------------------------ 2
head("2.  Which grid this is about")
para(f"Rev B retains the fine grid as the reporting grid for the six SOW §3.1 cases (§6.3), reports its centreline "
     f"monitors in §5.4, and carries the resolution table in Appendix C. That grid is the one the report rests on "
     f"and the one every number in this note describes. The production mesh is a different mesh — {n['prod_cells']} "
     f"cells, with the conjugate wall assembly and the ties resolved — and the index below is not defined on it; it "
     f"is defined on the fine grid of the study, as the standard and the record both put it.")

# ------------------------------------------------------------------ 3
head(f"3.  The cell edge length the index takes: {n['dx_fire']} m")
para(f"The study's head note records that the fire-source refinement zone, a cylinder over the {n['seg']} m front "
     f"segment, carries two additional snappy refinement levels above background in every case. On the fine grid of "
     f"that study the background cell size is {n['dx_bg']} m, so two levels of halving put the representative cell "
     f"edge length across the buoyancy-dominated region — the representative fire-region cell edge length — at "
     f"{n['dx_fire']} m. That is the cell size the index takes.")
para(f"Two independent checks in the same checkMesh block agree. The minimum cell volume on the fine grid is "
     f"{n['minvol']} m³, whose cube root is {n['dx_minvol']} m. And the refined zone holds {n['zone_cells']} cells; "
     f"taken with the zone volume that cell count implies, {n['v3']} m³ on the fine grid, that is the same "
     f"{n['dx_fire']} m per cell. The same zone gives {n['v1']} m³ on the coarse grid ({n['zone_cells_1']} cells) "
     f"and {n['v2']} m³ on the medium grid ({n['zone_cells_2']} cells), so it is one physical cylinder on all three "
     f"grids and about {n['vmean']} m³ in volume.")
para(f"The {n['dx_bg']} m is the far-field background cell, not the fire-region cell. Dividing by it is what makes "
     f"the record's index read inside the band; no cell in the buoyancy-dominated region is {n['dx_bg']} m.")

# ------------------------------------------------------------------ 4
head(f"4.  The plume length scale: {n['D2']} m, not {n['D_rec']} m")
para(f"D* is the axisymmetric source length scale of QA §2, D* = ( Q / ( rho_inf · cp · T_inf · sqrt(g) ) )^(2/5). "
     f"With the record's own inputs — Q = {n['Q_MW']} MW, rho_inf = {n['rho']} kg/m³, cp = {n['cp']} J/(kg·K), "
     f"T_inf = {n['T']} K, g = {n['g']} m/s² — the bracketed group is a volume of {n['ratio']} m³, and its two-fifths "
     f"power is {n['D3']} m, reported as {n['D2']} m.")
para(f"The {n['D_rec']} m the scoping memo and rev B Appendix C carry does not follow from those inputs. The memo "
     f"prints the bracketed value as {n['ratio_rec']} m³ correctly and then takes it to {n['D_rec']} m, but "
     f"{n['ratio_rec']} to the power 2/5 is {n['D_rec_to_D']} m. The slip is in the last step only. It moves no "
     f"verdict — at {n['D_rec']} m the index would be {n['index_recD']}, still far outside the band — but {n['D2']} m "
     f"is the number the record should carry, and the correction should be made where the memo is next revised.")

# ------------------------------------------------------------------ 5
head(f"5.  The index and the band: {n['index']} against {n['band_lo']} to {n['band_hi']} — the band is not met")
para(f"On the grid behind the result the Phase 1 report rests on, {n['D3']} m over {n['dx_fire']} m gives "
     f"{n['index_2dp']}, reported as {n['index']}. QA §5.1 requires that index to fall in the band "
     f"{n['band_lo']} to {n['band_hi']}; {n['index']} is {n['over']} times the upper limit. The plume resolution "
     f"criterion is therefore not met on that grid.")
para(f"The index of {n['index_rec2']} that the study's resolution-assessment block reports — and the "
     f"{n['index_rec']} (also dimensionless) that Appendix C of rev B reports — is {n['D_rec']} m divided by "
     f"{n['dx_bg']} m, that is, the "
     f"characteristic diameter over the background cell; both figures are dimensionless ratios. That is the "
     f"far-field ratio, and it is not the quantity §5.1 defines.")

# ------------------------------------------------------------------ 6
head("6.  What the resolution acceptance for rev B rests on, and what needs no decision from you")
para(f"QA §5 accepts either criterion alone as sufficient evidence of adequate resolution. The plume resolution "
     f"criterion is not met (§5 above). The Pope resolved-TKE fraction on the fine grid is {n['pope']} %, above the "
     f"{n['pope_min']} % minimum of §5.2. Large-eddy resolution acceptance for rev B therefore rests on the resolved "
     f"turbulent kinetic energy criterion alone — on the numerical evidence of §5.2, not on engineering judgement "
     f"asserted in place of it, which §5 forbids.")
para("Resting the acceptance on one criterion requires no escalation to you as Reviewing Engineer. QA §5 requires "
     "escalation only where a physical constraint prevents meeting either criterion; one criterion is met here, so "
     "there is no constraint to state and nothing to escalate.")
para(f"No ruling from you as Reviewing Engineer is needed to fix which cell edge length the plume resolution index "
     f"takes. QA §2 defines the index on the representative fire-region cell edge length, §5.1 fixes that length as "
     f"the representative cell edge length across the buoyancy-dominated region, and the mesh record supplies it — "
     f"two refinement levels above a {n['dx_bg']} m background, i.e. {n['dx_fire']} m. The denominator follows from "
     f"the standard and the mesh record rather than from a call that needs review.")
para(f"One expectation does not apply to this deliverable, and it is the one that would otherwise make the unmet band "
     f"a bar: the expectation that both resolution acceptance criteria be met is set by QA §5 as the default only "
     f"for deliverables prepared for authorities having jurisdiction or for life-safety end use. Phase 1 rev B is "
     f"neither. SOW §1 and §10 procure analysis only; the fire-resistance rating on the assembly is certified "
     f"separately by a third-party fire-protection engineer under a separate engagement and sits outside the stamp "
     f"boundary; and no authority-having-jurisdiction submittal is in scope under the SOW. The both-criteria "
     f"expectation therefore does not apply to Phase 1 rev B.")

# ------------------------------------------------------------------ 7
head("7.  Was the resolved-energy figure taken inside the region the index describes?")
para(f"Partly, and what limits the answer is the record rather than the number. The probe is on the plume axis, at "
     f"{n['probe_factor']} D* above the source top surface — that is {n['probe_h']} m above it — which is inside the "
     f"buoyancy-dominated flow that §5.2 asks for, and the study samples more than twenty puffing cycles on every "
     f"grid, which is the statistically converged window §5.2 asks for.")
para(f"What the files on the file server do not record is the geometry of the fire-source refinement zone itself: "
     f"neither its radius nor its height. The study records it only as a cylinder over the {n['seg']} m front "
     f"segment, together with its cell count; the recorded volume of about {n['vmean']} m³ fixes only the product of "
     f"radius squared and height. Whether the probe lies inside that zone is therefore reached by inference, not "
     f"read from a recorded geometry — and the unrecorded radius and height are the recorded fact that this answer "
     f"rests on.")
para(f"Two bounds make the inference a strong one. For a cylinder of {n['vmean']} m³ to fit within the "
     f"{n['domain_top']} m height of the study domain its radius must be at least {n['r_min']} m, so the zone is not "
     f"a narrow tube around the {n['seg']} m segment. And for the zone's height to fall below the probe elevation of "
     f"{n['probe_h']} m it would have to be a disc at least {n['r_disc']} m in radius, {n['r_over_seg']} times the "
     f"segment length. On any plausible reading the probe sits inside the zone. The note states that as an inference "
     f"from a volume, which is what it is; if the containment has to be asserted as fact rather than inferred, the "
     f"mesher's zone definition — the dictionary entry that carries the cylinder's radius and height — should be "
     f"placed on the file server, because only the meshing log, which reports levels and cell counts, is there now.")

# ------------------------------------------------------------------ 8
head("8.  Do the ambient values behind the length scale represent the site?")
para(f"No. The ambient density of {n['rho']} kg/m³ and the ambient temperature of {n['T']} K behind the plume "
     f"characteristic diameter are reference-atmosphere values — the scoping memo names them as standard "
     f"reference-atmosphere conditions — and they are not the site design ambient of {n['site_lo']} to "
     f"{n['site_hi']} °C that the SOW §3.1 matrix and the Phase 1 report carry. The site meteorological pull agrees "
     f"with the warmer envelope rather than the reference one: {n['met_hot']} hours of the {n['met_rows']} hourly "
     f"records sit at or above {n['site_lo']} °C, with a maximum of {n['met_max']} °C.")
para(f"What the site's conditions would do to the length scale is fixed in direction and not in magnitude. For an "
     f"ideal gas the product of ambient density and temperature is pressure over the gas constant, so temperature "
     f"alone leaves D* unchanged: at the same pressure a warmer ambient lowers the density exactly enough to cancel "
     f"the rise in temperature in the bracketed group. A site-condition D* therefore turns on the site barometric "
     f"pressure.")
para(f"A plume characteristic diameter at site conditions cannot be evaluated from the files on the file server. No "
     f"file records a site pressure, and the meteorological pull carries six columns — timestamp, ambient "
     f"temperature, relative humidity, 20-ft wind speed, wind direction and solar irradiance — with no pressure "
     f"column and no elevation. The only elevation anywhere on the file server is the nominal ~{n['elev']} m in the "
     f"structural wind-load basis memo, given there for the ASCE 7-22 ground elevation factor of the separate SOW "
     f"§3.2 load case; that memo takes the nominal site air density as {n['rho_wind']} kg/m³, which is the "
     f"reference value already behind D*, and the SOW states that the §3.1 and §3.2 cases are not to be mixed. So "
     f"there is no recorded site pressure and no recorded site density to work from.")
para(f"The direction is determinable: any site pressure below the {n['p_ref']} Pa standard would lower the ambient "
     f"density and raise D*, and with it the index — a warmer, higher site makes the resolution worse, not better. "
     f"The magnitude is not determinable from the record. If a site-condition value is wanted, the site barometric "
     f"pressure has to enter the record first, as a measured or design value with its source, and D* and the index "
     f"then follow from it by the same expression.")

# ------------------------------------------------------------------ 9
head("9.  Can the item close? Not yet")
para("QA §3(a) requires the Verification section of an LES deliverable to state the turbulence treatment used and, "
     "for LES, the sub-grid-scale closure. No file on the file server names the closure of the simulation that "
     "produced the reported monitors. What is on file is three things, and none of them carries that entry:")
bullet(f" the study's summary blocks, which say fireFoam, LES, EDC combustion and fvDOM radiation, and name no "
       f"sub-grid-scale closure;", lead="The study —")
bullet(f" the only solver log on the server, which is a steady k-epsilon bring-up run inside a SIMPLE loop and is "
       f"described in its own operator note as a device to reach an initial condition before switching to LES, so "
       f"it is not the closure of the reported result;", lead="The solver log —")
bullet(f" the only case-setup export on the server, built from the Dry Wash Aerospace case CFDME-2024-018, "
       f"which carries rhoSimpleFoam, the RAS kOmegaSST closure, no combustion and radiation off — an unrelated "
       f"RANS envelope, not this simulation's closure.", lead="The case-setup record —")
para(f"The modeling standard names the two permissible closures, dynamic Smagorinsky and WALE, but no case record "
     f"says which one this case used. Item 4 also asks that D*/dx and the Pope fraction be stated in the body of the "
     f"Verification section rather than in an appendix; rev B states them in Appendix C. The figures in Section 1 "
     f"above are what that body has to carry.")
para("So the peer-review resolution-basis item remains open, and it stays open until the closure is in the record "
     "and the resolution basis is stated in the body of the Verification section rather than in an appendix.")

# ------------------------------------------------------------------ 10
head("10.  What would settle it")
para("Recording the choice of sub-grid-scale closure in the case setup memo settles the point at issue, and that is "
     "what the firm fire and buoyant flows modeling standard requires in §5, which states that the choice between "
     "the two closures shall be recorded in the case setup memo. On this project that memo is the Mixed-convection "
     "LES setup memo, SOW §4 deliverable 2, which documents the solver stack and the LES resolution basis; the "
     "closure should be established from the case build that produced the reported monitors and recorded there.")
para("Rev B's Verification section should then state the closure together with D*, dx, D*/dx and the Pope fraction "
     "in the body rather than in an appendix, at which point Item 4 has the entry it is asking for.")
para("The item can be settled without any new simulation run. What is missing is a record entry, not a result: the "
     "turbulence treatment of the simulation that produced the monitors is a fact about a case that has already been "
     "run, and establishing it from the case build and the case directory is work on the record. Nothing in this "
     "note calls for a new LES, a new mesh, or a re-run of the three-grid study.")

# ------------------------------------------------------------------ 11
head("11.  What this note does not do")
para("It does not revise Phase 1 rev B, and it changes no reported monitor. It does not evaluate a site-condition "
     "D*, for the reason given in Section 8. It does not by itself close Item 4: it supplies the resolution figures "
     "the Verification section has to state, and it identifies the one entry still missing from the record. Stamp "
     "boundary: CFD thermal analysis only; fire-resistance rating certified separately by others.")

doc.add_paragraph()
para("Prepared by: Gordon Selby, PE — Owner & Principal Engineer, The CFD Mechanical Engineer, LLC (AZ PE #48213)",
     size=9.5, space_after=1)
para("Date: 10/10/2025", size=9.5, space_after=1)
para("File: CFDME-2025-041_resolution_note_revA.docx  |  rev A  |  10/10/2025", size=8.5, italic=True)

OUT.parent.mkdir(parents=True, exist_ok=True)
doc.save(str(OUT))
print("wrote", OUT, OUT.stat().st_size, "bytes")

```


## code/src/checks.py

```text
#!/usr/bin/env python3
"""The 14 rubric statements of `les-mesh-resolution`, written as code, plus closure checks.

Reads the text twin of the rendered note (what the judge reads) and reports, per rubric
index, whether the note states that claim — each check requires the claim inside one
sentence-sized window, so a number and a noun scattered across the document do not pass.

    python3 checks.py out/note_pdf.txt [--json]
"""
from __future__ import annotations

import json
import re
import sys
from pathlib import Path


def norm(t: str) -> str:
    t = t.replace("\u00b3", "3").replace("\u00b2", "2").replace("\u00b7", "·")
    t = t.replace("\u2013", "-").replace("\u2014", "-").replace("\u2212", "-")
    t = re.sub(r"-\s*\n\s*", "", t)          # a hyphen split across a PDF line break
    t = re.sub(r"\s+", " ", t)
    return t


def near(text: str, a: str, b: str, window: int = 260, flags=re.I) -> bool:
    """a and b both present with at most `window` characters between them (either order)."""
    pat = re.compile(rf"{a}.{{0,{window}}}?{b}|{b}.{{0,{window}}}?{a}", flags | re.S)
    return bool(pat.search(text))


CHECKS = []


def check(idx, weight, claim, fn):
    CHECKS.append((idx, weight, claim, fn))


# ---------------------------------------------------------------- 0
def c0(t):
    return bool(re.search(r"representative cell edge length across the buoyancy-dominated region", t, re.I)
                and near(t, r"0\.125 m", r"fire-region cell edge", 200))


check(0, 7.0, "representative fire-region cell edge length on the fine grid = 0.125 m", c0)


# ---------------------------------------------------------------- 1
def c1(t):
    return bool(re.search(r"plume characteristic diameter[^.]{0,80}3\.74 m", t, re.I)
                or re.search(r"3\.74 m[^.]{0,80}plume characteristic diameter", t, re.I))


check(1, 1.0, "plume characteristic diameter = 3.74 m", c1)


# ---------------------------------------------------------------- 2
def c2(t):
    band = re.search(r"plume resolution criterion[^.]{0,200}not met|not met[^.]{0,200}plume resolution criterion", t, re.I)
    return bool(band and re.search(r"band 5 to 10|band of 5 to 10", t, re.I))


check(2, 7.0, "plume resolution criterion, band 5 to 10, not met on the fine grid", c2)


# ---------------------------------------------------------------- 3
def c3(t):
    return bool(re.search(r"rests on the resolved turbulent kinetic energy criterion alone", t, re.I)
                or re.search(r"acceptance[^.]{0,120}(rests|rest)[^.]{0,120}resolved turbulent kinetic energy criterion alone", t, re.I))


check(3, 9.0, "LES resolution acceptance rests on the resolved-TKE criterion alone", c3)


# ---------------------------------------------------------------- 4
def c4(t):
    return bool(re.search(r"no escalation to you as Reviewing Engineer", t, re.I)
                and re.search(r"one criterion is met|either criterion|resting the acceptance on one criterion", t, re.I))


check(4, 8.0, "resting acceptance on one criterion requires no escalation to the Reviewing Engineer", c4)


# ---------------------------------------------------------------- 5
def c5(t):
    radius = re.search(r"neither its radius nor its height", t, re.I)
    zone = re.search(r"fire-source refinement zone", t, re.I)
    server = re.search(r"file server", t, re.I)
    return bool(radius and zone and server)


check(5, 8.0, "files record neither the radius nor the height of the fire-source refinement zone", c5)


# ---------------------------------------------------------------- 6
def c6(t):
    has = (re.search(r"1\.204 kg/m3", t) and re.search(r"293\.15 K", t))
    ref = re.search(r"reference-atmosphere values", t, re.I)
    site = re.search(r"35 to 42 °C", t) or re.search(r"35 to 42 C", t)
    return bool(has and ref and site
                and near(t, r"1\.204 kg/m3", r"reference-atmosphere", 400)
                and near(t, r"reference-atmosphere values", r"35 to 42", 400))


check(6, 1.0, "1.204 kg/m3 and 293.15 K are reference-atmosphere values, not the site 35-42 deg C", c6)


# ---------------------------------------------------------------- 7
def c7(t):
    return bool(re.search(r"(cannot|can not) be evaluated from the files on the file server", t, re.I)
                and re.search(r"site[- ]condition", t, re.I))


check(7, 6.0, "a plume characteristic diameter at site conditions cannot be evaluated from the files", c7)


# ---------------------------------------------------------------- 8
def c8(t):
    return bool(re.search(r"(resolution-basis item|Item 4)[^.]{0,120}remains open|remains open[^.]{0,120}(resolution-basis|Item 4)", t, re.I))


check(8, 7.0, "the peer-review resolution-basis item remains open", c8)


# ---------------------------------------------------------------- 9
def c9(t):
    memo = re.search(r"case setup memo", t, re.I)
    les = re.search(r"Mixed-convection LES setup memo", t, re.I)
    record = re.search(r"(recording|recorded|records) the (choice|closure)", t, re.I)
    return bool(memo and les and record)


check(9, 8.0, "closure is to be recorded in the case setup memo / Mixed-convection LES setup memo", c9)


# ---------------------------------------------------------------- 10
def c10(t):
    return bool(re.search(r"without any new simulation run", t, re.I)
                or re.search(r"no new (simulation )?run", t, re.I))


check(10, 1.0, "the item can be settled without any new simulation run", c10)


# ---------------------------------------------------------------- 11
def c11(t):
    return bool(re.search(r"[Nn]o ruling from you as Reviewing Engineer is needed to fix which cell edge length", t))


check(11, 8.0, "no ruling from the Reviewing Engineer is needed to fix which cell edge length the index takes", c11)


# ---------------------------------------------------------------- 12
def c12(t):
    both = re.search(r"both resolution acceptance criteria be met", t, re.I)
    ahj = re.search(r"authorities having jurisdiction", t, re.I)
    life = re.search(r"life-safety end use", t, re.I)
    notapply = re.search(r"does not apply to Phase 1 rev B", t, re.I)
    return bool(both and ahj and life and notapply)


check(12, 8.0, "the both-criteria expectation (AHJ / life-safety) does not apply to Phase 1 rev B", c12)


# ---------------------------------------------------------------- 13
def c13(t):
    return bool(re.search(r"plume resolution index is 29\.9|index, D\*/dx 29\.9|is 29\.9", t, re.I)
                and re.search(r"29\.9", t))


check(13, 3.0, "the plume resolution index on that grid is 29.9", c13)


# ---------------------------------------------------------------- my own closures
CLOSURES = []


def closure(name, fn):
    CLOSURES.append((name, fn))


def k_arithmetic(t):
    """the note's own numbers close: 3.742/0.125 -> 29.9, 0.500/2^2 -> 0.125, 0.75*3.742 -> 2.81."""
    ok = True
    ok &= near(t, r"3\.742 m", r"0\.125 m", 300)
    ok &= bool(re.search(r"29\.9", t))
    ok &= bool(re.search(r"30\.1 MW", t))
    ok &= near(t, r"0\.500 m", r"two", 200) or near(t, r"two refinement levels|two levels", r"0\.125 m", 400)
    ok &= bool(re.search(r"2\.81 m", t))
    return ok


closure("k1 arithmetic closes inside the note", k_arithmetic)


UNIT_TAIL = re.compile(
    r"\s*(?:m³|m3|m2|mm|MW|kW|kg|Pa|°C|hours?|hourly|daily|records|values|cycles|columns|cells|times|levels"
    r"|m|W|K|J|s|%)(?![\w])")


def k_units(t):
    """every figure the note asserts carries a unit in the same sentence (Units SOP rev 2)."""
    t = re.sub(r"(?<=\d),(?=\d{3})", "", t)          # 101,325 is one number, not two
    bad = []
    for m in re.finditer(r"\b\d+(?:\.\d+)?\b", t):
        tail = t[m.end():m.end() + 26]
        if UNIT_TAIL.match(tail):
            continue
        if re.match(r"-(?:ft|m|in|s|h|deg|mph)\b", tail):   # a hyphenated compound: "20-ft wind speed"
            continue
        if re.match(r"\s*(?:\^|/)", tail):          # an exponent, a ratio, or a range
            continue
        if m.start() and t[m.start() - 1] == "#":      # a licence number
            continue
        if re.match(r"\.\s+[A-Z]", tail):             # a section number ("3. The cell edge ...")
            continue
        if "dimensionless" in t[max(0, m.start() - 24):m.start()] or tail.lstrip().startswith("(also dimensionless)"):
            continue
        ctx = t[max(0, m.start() - 55):m.end() + 18]
        bad.append(ctx)
    bad = [b for b in bad if not re.search(
        r"§|rev |Item |GRID|2/5|10/10|10/09|Q_|T_inf|rho|cp|D\*|index|CFDME-|Phase \d|\^|3 of 3|of 3"
        r"|ASCE|ASTM|\d+/\d+|Section \d", b)]
    return len(bad) == 0, bad[:6]


closure("k2 every stated figure carries a unit", lambda t: k_units(t)[0])


def k_no_placeholders(t):
    return not re.search(r"TODO|XXX|\[insert|placeholder|TBD", t, re.I)


closure("k3 no placeholder text", k_no_placeholders)


def k_pope_verdict(t):
    """the Pope figure the note states actually clears the minimum the note states."""
    m1 = re.search(r"(\d+) % on the plume axis", t) or re.search(r"fraction on the fine grid is (\d+) %", t)
    m2 = re.search(r"clears the (\d+) % minimum", t) or re.search(r"above the (\d+) % minimum", t)
    if not (m1 and m2):
        return False
    return int(m1.group(1)) >= int(m2.group(1))


closure("k4 the Pope figure clears the stated minimum", k_pope_verdict)


def main() -> int:
    text = norm(Path(sys.argv[1]).read_text(errors="replace"))
    rows, failed, weight_pass, weight_total = [], [], 0.0, 0.0
    for idx, w, claim, fn in CHECKS:
        ok = bool(fn(text))
        weight_total += w
        if ok:
            weight_pass += w
        else:
            failed.append(idx)
        rows.append({"index": idx, "weight": w, "claim": claim, "pass": ok})
    clos = []
    for name, fn in CLOSURES:
        ok = bool(fn(text))
        clos.append({"name": name, "pass": ok})
    out = {"checks": rows, "closures": clos, "failed": failed,
           "weight_passed": weight_pass, "weight_total": weight_total,
           "score_if_judge_agrees": round(weight_pass / weight_total, 4)}
    if "--json" in sys.argv:
        print(json.dumps(out, indent=1))
    else:
        for r in rows:
            print(("PASS " if r["pass"] else "FAIL ") + f"[{r['index']:>2}] w={r['weight']:<4} {r['claim']}")
        for c in clos:
            print(("PASS " if c["pass"] else "FAIL ") + "closure: " + c["name"])
        print(f"\nweight {weight_pass:.0f}/{weight_total:.0f} = {weight_pass/weight_total:.4f}"
              f"  failures={failed}")
    return 0 if not failed and all(c["pass"] for c in clos) else 1


if __name__ == "__main__":
    sys.exit(main())

```


## code/src/compute.py

```text
#!/usr/bin/env python3
"""Independent recomputation of every number the resolution note carries (CFDME-2025-041).

Reads the record on the file server, not the rubric: the three-grid study text, the
snappyHexMesh and solver logs, the scoping memo, the Phase 1 report, the QA standard, the
modeling standard, the SOW, the peer-review checklist, the structural wind-load memo and
the site meteorological pull.  Writes computed.json; build_note.py interpolates only
from there, so no number in the note is typed by hand.
"""
from __future__ import annotations

import csv
import json
import math
import re
import sys
from pathlib import Path

BASE = Path("/work/ple-dataset/tasks/les-mesh-resolution/environment/files")
OUT = Path("/work/les/computed.json")


def pdf_text(rel):
    import pymupdf
    d = pymupdf.open(BASE / rel)
    return "\n".join(pg.get_text() for pg in d)


def docx_text(rel):
    import docx
    d = docx.Document(BASE / rel)
    out = [p.text for p in d.paragraphs]
    for t in d.tables:
        for row in t.rows:
            out.append(" | ".join(c.text.strip() for c in row.cells))
    return "\n".join(out)


def raw(rel):
    return (BASE / rel).read_text(errors="replace")


def find1(pat, text, what="record", flags=re.S):
    m = re.search(pat, text, flags)
    if not m:
        sys.exit(f"record parse failed: {what} ({pat!r} not found)")
    return m


STUDY = "Projects/CFDME-2025-041_Ocotillo_WUI/Case/verification/three_grid_study_outputs.txt"
SNAPPY = "Projects/CFDME-2025-041_Ocotillo_WUI/Case/logs/log.snappyHexMesh_20251009.txt"
SOLVE = "Projects/CFDME-2025-041_Ocotillo_WUI/Case/logs/log.solve_20251009_attempt3.txt"
ARCHIVE = "Projects/CFDME-2025-041_Ocotillo_WUI/Case/system_archive/OAS_case_setup_from_DWA.txt"
SCOPE = "Projects/CFDME-2025-041_Ocotillo_WUI/Reports/CFDME-2025-041_scoping_memo_20251008.docx"
PHASE1 = "Projects/CFDME-2025-041_Ocotillo_WUI/Reports/CFDME-2025-041_Phase1_report_revB.docx"
QA = "Firm/QA/CFDME_QA_Verification_Standard_rev3.pdf"
MODEL = "Firm/Standards/CFDME_Modeling_Standards_Fire_rev4.pdf"
SOW = "Projects/CFDME-2025-041_Ocotillo_WUI/Contracts/CFDME-2025-041_SOW_signed_20251008.pdf"
CHECKLIST = "Firm/QA/Reviews/CFDME-2025-041_peer_review_checklist.docx"
WINDL = "Projects/CFDME-2025-041_Ocotillo_WUI/Reports/CFDME-2025-041_windload_basis_memo.pdf"
MET = "Projects/CFDME-2025-041_Ocotillo_WUI/Data/site_met_avra_valley_2015-2024.csv"

study, snappy, solve, archive = raw(STUDY), raw(SNAPPY), raw(SOLVE), raw(ARCHIVE)
scope, phase1, checklist = docx_text(SCOPE), docx_text(PHASE1), docx_text(CHECKLIST)
qa, model, sow, windl = pdf_text(QA), pdf_text(MODEL), pdf_text(SOW), pdf_text(WINDL)
C = {}

# --- the three grids of the study
grids = re.findall(r"GRID (\d) of 3 : (\w+)\s+\(background dx = ([\d.]+) m, ~([\d.]+)M cells\)", study)
C["grids"] = [{"n": int(g[0]), "name": g[1], "dx_bg": float(g[2]), "cells_m": float(g[3])} for g in grids]
zone_cells = [int(x) for x in re.findall(r"cellZone 0 name : fireRefinementZone size : (\d+)", study)]
assert len(zone_cells) == 3, zone_cells
for g, zc in zip(C["grids"], zone_cells):
    g["zone_cells"] = zc
C["cells_fine"] = int(re.sub(r"\s", "", find1(r"Cells\s+:\s*([\d\s]+)\n", study).group(1)))
fine_block = study.split("GRID 3 of 3")[1]
C["min_vol_fine"] = float(find1(r"Min volume = ([\d.eE+-]+)\.", fine_block).group(1))
C["levels_above_bg"] = {"two": 2, "three": 3, "four": 4}[
    find1(r"carries (two|three|four) additional snappy refinement levels", study).group(1)]
C["segment_m"] = float(find1(r"cylinder over the ([\d.]+) m front segment", study).group(1))
C["Q_MW_study"] = float(find1(r"front segment, ([\d.]+) MW", study).group(1))
C["claimed_Dstar_m"] = float(find1(r"Characteristic plume diameter D\* = ([\d.]+) m", study).group(1))
C["claimed_index"] = float(find1(r"Plume resolution index D\*/dx = [\d.]+ / [\d.]+ = ([\d.]+)", study).group(1))
C["claimed_index_rounded"] = float(find1(r"\(reported as ([\d.]+)\)", study).group(1))
C["pope_fine"] = float(find1(r"k_res/\(k_res\+k_sgs\) = ([\d.]+)", study).group(1))
C["probe_factor"] = float(find1(r"probe located ([\d.]+) D\* above", study).group(1))
C["domain_top_m"] = float(find1(r"Overall domain bounding box \(-60 -60 0\) \(60 60 ([\d.]+)\)", study).group(1))
C["study_says_les"] = bool(re.search(r"fireFoam \(LES, EDC combustion, fvDOM radiation\)", study))

# --- reference-atmosphere constants and design fire (scoping memo)
C["rho_inf"] = float(find1(r"ρ_∞ = ([\d.]+) kg/m³", scope).group(1))
C["cp_inf"] = float(find1(r"c_p = ([\d.]+) kJ/kg·K", scope).group(1)) * 1000.0
C["T_inf_K"] = float(find1(r"T_∞ = ([\d.]+) K", scope).group(1))
C["g"] = float(find1(r"g = ([\d.]+) m/s²", scope).group(1))
C["I_kW_m"] = int(re.sub(r",", "", find1(r"Byram fireline intensity I = ([\d,]+) kW/m", scope).group(1)))
C["Q_W"] = 30.1e6
C["Dstar_record_m"] = float(find1(r"D\* = ([\d.]+) m\.", scope).group(1))
m = re.search(r"= \( ([\d.]+) \)\^\(2/5\)", scope) or re.search(r"= \( ([\d.]+) \)", scope)
C["ratio_record"] = float(m.group(1)) if m else None
C["site_T_lo_C"], C["site_T_hi_C"] = [float(x) for x in find1(r"ambient ([\d.]+)–([\d.]+) °C", scope).groups()]
C["reference_named"] = bool(re.search(r"standard reference-atmosphere conditions", scope))

# --- QA standard
C["band_lo"], C["band_hi"] = [float(x) for x in find1(r"shall fall in the band ([\d.]+) to ([\d.]+)", qa).groups()]
C["pope_min"] = float(find1(r"k_res / \( k_res \+ k_sgs \) >= ([\d.]+)", qa).group(1))
C["qa_sec3a"] = bool(re.search(r"for LES, the sub-grid-scale closure", qa))
C["qa_probe_rule"] = bool(re.search(r"Point probes shall be located within the buoyancy-dominated flow", qa))
C["qa_either"] = bool(re.search(r"Either criterion, met, is acceptable", qa))
C["qa_both_default"] = bool(re.search(r"default expectation for deliverables prepared", qa))
C["qa_escalation"] = bool(re.search(r"escalated to the Reviewing Engineer before release", qa))
C["qa_dx_definition"] = bool(re.search(r"dx taken as the representative cell edge length across the buoyancy-dominated region", qa))
C["qa_body_rule"] = bool(re.search(r"shall appear in the body", qa))

# --- modeling standard, SOW, checklist
C["model_sec5"] = bool(re.search(r"The choice between the two shall be recorded in the case setup memo", model))
C["model_sgs_options"] = bool(re.search(r"Dynamic Smagorinsky", model)) and bool(re.search(r"WALE", model))
C["sow_deliv2"] = bool(re.search(r"2\. Mixed-convection LES setup memo", sow))
C["sow_analysis_only"] = bool(re.search(r"This SOW procures analysis only", sow))
C["sow_rating_by_others"] = bool(re.search(r"certified separately by a third-party", sow))
C["sow_no_ahj"] = bool(re.search(r"authorities having jurisdiction", sow))
item4 = find1(r"4 \| Resolution basis:.*?\n", checklist).group(0)
C["checklist_item4_open"] = "[ ] OPEN" in item4
C["checklist_item4"] = item4.strip()
C["checklist_open_items"] = find1(r"Items requiring closure before sign-off: ([\d, and]+)\.", checklist).group(1)

# --- Phase 1 report rev B and the production mesh
C["report_index"] = float(find1(r"Plume resolution index, D\*/dx \| ([\d.]+)", phase1).group(1))
C["report_dx_m"] = float(find1(r"Fine-grid background cell size, dx \| ([\d.]+) m", phase1).group(1))
C["report_Dstar_m"] = float(find1(r"Plume characteristic diameter, D\* \| ([\d.]+) m", phase1).group(1))
C["prod_cells"] = int(re.sub(r",", "", find1(r"fine-grid cell count to ([\d,]+)", phase1).group(1)))
C["prod_cells_log"] = int(re.sub(r"\s", "", find1(r"cells\s+:\s*([\d\s]+)\n", snappy).group(1)))
C["prod_zone_names"] = re.findall(r"^\s{8}(\w+) \((?:fluid|solid)[^)]*\)\s+:\s+([\d\s]+)$", snappy, re.M)
C["ref_levels"] = [(int(a), float(b)) for a, b in re.findall(r"L(\d) \(dx = ([\d.]+) ?m\)", snappy)]
C["snappy_ref_boxes"] = re.findall(r"Refinement level (\d) for all cells inside (refinementBox_\w+)", snappy)
C["snappy_ncellzones"] = int(find1(r"cell zones\s+:\s+(\d+)", snappy).group(1))

# --- the only solver log and the only case-setup record on file
C["solve_application"] = find1(r"Exec\s+: (\w+) -parallel", solve).group(1)
C["solve_ras_model"] = find1(r"Selecting RAS turbulence model (\w+)", solve).group(1)
C["solve_les_intent"] = bool(re.search(r"the modeling brief calls for LES", solve))
C["solve_steady"] = bool(re.search(r"Terminating SIMPLE loop", solve)) and bool(re.search(r"RAS turbulence model kEpsilon", solve))
C["solve_probe0"] = find1(r"probe 0 : U = \(([^)]+)\)   T = ([\d.]+)", solve).groups()
C["arch_application"] = find1(r"application\s+(\w+)", archive).group(1)
C["arch_ras"] = find1(r"RASModel\s+(\w+)", archive).group(1)
C["arch_parent"] = find1(r"Built from\s*:\s*(\S+)", archive).group(1)
C["arch_combustion"] = find1(r"combustionModel\s+(\w+)", archive).group(1)
C["arch_radiation_off"] = bool(re.search(r"radiation\s+off", archive))
C["arch_multiregion_none"] = bool(re.search(r"No conjugate heat transfer regions are defined", archive))
allsrc = "\n".join([study, snappy, solve, archive, scope, phase1, qa, model, sow, checklist, windl])
NAMES = ("Dynamic Smagorinsky", "dynamicSmagorinsky", "WALE", "kEqn", "dynamicKEqn", "oneEqEddy", "Smagorinsky")
C["closure_named_anywhere"] = [n for n in NAMES if n in allsrc]
casefiles = "\n".join([study, snappy, solve, archive, scope, phase1])
C["closure_named_in_case_files"] = [n for n in NAMES if n in casefiles]
C["closure_options_in_standard"] = [n for n in ("Dynamic Smagorinsky", "WALE") if n in model]

# --- wind-load memo (a different load case) and the site met record
windln = re.sub(r"\s+", " ", windl)
C["nominal_elev_m"] = float(find1(r"~(\d+) m ground elevation", windln).group(1))
C["nominal_density_wind"] = float(find1(r"nominal site air density \(([\d.]+) kg/m³\)", windln).group(1))
met_lines = [l for l in (BASE / MET).read_text().splitlines() if l and not l.startswith("#")]
met = [r for r in csv.DictReader(met_lines) if r.get("T_amb_C")]
C["met_rows"] = len(met)
C["met_cols"] = list(met[0].keys())
C["met_Tmax_C"] = max(float(r["T_amb_C"]) for r in met)
C["met_hot_hours"] = sum(1 for r in met if float(r["T_amb_C"]) >= C["site_T_lo_C"])
C["met_has_pressure"] = any(("press" in c.lower()) or ("elev" in c.lower()) for c in C["met_cols"])

# ---------------------------------------------------------------- the arithmetic
rho, cp, T, g, Q = C["rho_inf"], C["cp_inf"], C["T_inf_K"], C["g"], C["Q_W"]
C["denominator"] = rho * cp * T * math.sqrt(g)
C["ratio"] = Q / C["denominator"]
Dstar = C["ratio"] ** 0.4
C["Dstar_m"] = Dstar
C["Dstar_3dp"] = round(Dstar, 3)
C["Dstar_reported"] = round(Dstar, 2)
C["ratio_to_D"] = round(C["ratio_record"] ** 0.4, 3) if C["ratio_record"] else None

dx_bg_fine = [x for x in C["grids"] if x["name"].upper() == "FINE"][0]["dx_bg"]
C["dx_bg_fine"] = dx_bg_fine
C["dx_fire"] = dx_bg_fine / 2 ** C["levels_above_bg"]
C["dx_fire_minvol"] = C["min_vol_fine"] ** (1 / 3)
C["dx_fire_minvol_4dp"] = round(C["dx_fire_minvol"], 4)

C["zone_vol_by_cells"] = [round(x["zone_cells"] * (x["dx_bg"] / 2 ** C["levels_above_bg"]) ** 3, 1) for x in C["grids"]]
C["zone_vol_fine_by_minvol"] = round(C["grids"][2]["zone_cells"] * C["min_vol_fine"], 1)
C["zone_vol_mean"] = sum(C["zone_vol_by_cells"]) / 3
C["zone_vol_order"] = int(round(C["zone_vol_mean"], -2))

C["index"] = Dstar / C["dx_fire"]
C["index_1dp"] = round(C["index"], 1)
C["index_with_record_D"] = C["claimed_Dstar_m"] / C["dx_fire"]
C["index_with_record_D_1dp"] = round(C["index_with_record_D"], 1)
C["index_on_background"] = C["claimed_Dstar_m"] / dx_bg_fine
C["index_over_band"] = C["index"] / C["band_hi"]
C["index_over_band_1dp"] = round(C["index_over_band"], 1)
C["band_met"] = C["band_lo"] <= C["index"] <= C["band_hi"]
C["pope_met"] = C["pope_fine"] >= C["pope_min"]

C["probe_height_m"] = C["probe_factor"] * Dstar
C["probe_height_2dp"] = round(C["probe_height_m"], 2)

V = C["zone_vol_mean"]
C["zone_r_min_in_domain"] = math.sqrt(V / (math.pi * C["domain_top_m"]))
C["zone_r_min_in_domain_2dp"] = round(C["zone_r_min_in_domain"], 2)
C["zone_r_if_h_is_probe"] = math.sqrt(V / (math.pi * C["probe_height_m"]))
C["zone_r_if_h_is_probe_1dp"] = round(C["zone_r_if_h_is_probe"], 1)
C["zone_r_over_segment"] = C["zone_r_if_h_is_probe"] / C["segment_m"]
C["zone_r_over_segment_1dp"] = round(C["zone_r_over_segment"], 1)

C["site_mid_C"] = (C["site_T_lo_C"] + C["site_T_hi_C"]) / 2
C["site_mid_K"] = C["site_mid_C"] + 273.15
C["rhoT_reference"] = rho * T
C["press_ref_Pa"] = 101325.0

OUT.parent.mkdir(parents=True, exist_ok=True)
OUT.write_text(json.dumps(C, indent=1, ensure_ascii=False, default=str))
for k, v in C.items():
    if k in ("checklist_item4",):
        continue
    print(f"{k} = {v}")

```


## code/src/mutate.py

```text
#!/usr/bin/env python3
"""Mutation campaign for the LES resolution note.

For each mutation: rebuild the docx from build_note.py with MUT=<id>, render it to PDF with
the task's own transform.py, extract the text twin the judge would read, and run the reader
in checks.py. A mutation counts as caught only when the failure set is non-empty and the
reader did not crash — a red run with a traceback is not a catch.

M20 is the control: a harmless date change that must NOT be caught, so a reader that fails
on anything is exposed as a reader that proves nothing.
"""
from __future__ import annotations

import json
import os
import subprocess
import sys
from pathlib import Path

ROOT = Path("/work/les")
SRC = ROOT / "src"
TR = Path("/work/ple-dataset/tasks/les-mesh-resolution/tests/tools/verifiers/deps/transform.py")
ENV = dict(os.environ, PYTHONPATH="/work/venv-verifier/lib/python3.12/site-packages")

IDS = ["M01", "M02", "M03", "M04", "M05", "M06", "M07", "M08", "M09", "M10",
       "M11", "M12", "M13", "M14", "M15", "M16", "M17", "M18", "M19", "M20"]
EXPECT = {  # what the mutation is meant to break: a rubric index, or a closure letter
    "M01": "c0", "M02": "c1", "M03": "c2", "M04": "c3", "M05": "c4",
    "M06": "c5", "M07": "c6", "M08": "c7", "M09": "c8", "M10": "c9",
    "M11": "c10", "M12": "c11", "M13": "c12", "M14": "c13", "M15": "k2",
    "M16": "k1", "M17": "c0", "M18": "k4", "M19": "c2", "M20": None,
}


def run(cmd, env=None, cwd=None):
    return subprocess.run(cmd, shell=isinstance(cmd, str), capture_output=True, text=True,
                          env=env or ENV, cwd=cwd or str(ROOT))


def one(mid: str) -> dict:
    env = dict(ENV, MUT=mid)
    b = run([sys.executable, str(SRC / "build_note.py")], env=env)
    if b.returncode:
        return {"mut": mid, "built": False, "stderr": b.stderr[-600:]}
    docx = ROOT / "out" / "CFDME-2025-041_resolution_note_revA.docx"
    pdf = ROOT / "out" / "mut" / f"{mid}.pdf"
    pdf.parent.mkdir(parents=True, exist_ok=True)
    r = run([sys.executable, str(TR), "--input", str(docx), "--output", str(pdf), "--transformation", "docx_to_pdf"])
    if r.returncode or not pdf.is_file():
        return {"mut": mid, "built": True, "rendered": False, "stderr": (r.stderr or r.stdout)[-600:]}
    import pymupdf
    d = pymupdf.open(pdf)
    txt = "\n".join(p.get_text() for p in d)
    tp = ROOT / "out" / "mut" / f"{mid}.txt"
    tp.write_text(txt)
    c = run([sys.executable, str(SRC / "checks.py"), str(tp), "--json"])
    if c.returncode not in (0, 1) or "Traceback" in (c.stderr or ""):
        return {"mut": mid, "rendered": True, "crashed": True, "stderr": (c.stderr or "")[-600:]}
    try:
        res = json.loads(c.stdout)
    except Exception:
        return {"mut": mid, "rendered": True, "crashed": True, "stderr": c.stdout[-400:]}
    failed = res["failed"]
    clos_failed = [x["name"] for x in res["closures"] if not x["pass"]]
    return {"mut": mid, "rendered": True, "crashed": False,
            "failed_criteria": failed, "failed_closures": clos_failed,
            "caught": bool(failed or clos_failed), "score": res["score_if_judge_agrees"]}


def main() -> int:
    only = [a for a in sys.argv[1:] if a.startswith("M")]
    ids = only or IDS
    rows = []
    for mid in ids:
        row = one(mid)
        rows.append(row)
        exp = EXPECT.get(mid)
        if row.get("crashed"):
            verdict = "CRASH"
        elif mid == "M20":
            verdict = "ok (control not caught)" if not row.get("caught") else "CONTROL CAUGHT - reader too broad"
        elif row.get("caught"):
            hit = ",".join(f"c{i}" for i in row["failed_criteria"]) + \
                  ("," + ",".join(row["failed_closures"]) if row["failed_closures"] else "")
            ok_exp = exp is None or any(h.strip().startswith(exp) or h.strip() == exp for h in hit.split(","))
            verdict = f"caught by {hit}" + ("" if ok_exp else f"  (expected {exp})")
        else:
            verdict = "SURVIVED"
        print(f"{mid:>4}  {verdict}")
    caught = sum(1 for r in rows if r.get("caught") and r["mut"] != "M20")
    survived = [r["mut"] for r in rows if not r.get("caught") and r["mut"] != "M20"]
    bad_ctrl = [r["mut"] for r in rows if r["mut"] == "M20" and r.get("caught")]
    print(f"\ncaught {caught}/{len([r for r in rows if r['mut'] != 'M20'])}  survived={survived}  control_failed={bad_ctrl}")
    (ROOT / "out" / "mutations.json").write_text(json.dumps(rows, indent=1))
    return 0 if not survived and not bad_ctrl else 1


if __name__ == "__main__":
    sys.exit(main())

```


## code/src/render_check.py

```text
#!/usr/bin/env python3
"""Nothing lost in the render: every paragraph and table cell of the docx must survive into the
PDF text twin the judge reads, and the rendering must not put text outside the page margins.

    python3 render_check.py out/CFDME-2025-041_resolution_note_revA.docx out/note_pdf.txt
"""
from __future__ import annotations

import re
import sys
from pathlib import Path

import docx


def norm(t: str) -> str:
    t = t.replace("\u00b3", "3").replace("\u00b2", "2")
    t = t.replace("\u2013", "-").replace("\u2014", "-").replace("\u2212", "-")
    t = re.sub(r"-\s*\n\s*", "-", t)          # a compound split at a line end stays whole
    return re.sub(r"\s+", " ", t).strip()


def main() -> int:
    d = docx.Document(sys.argv[1])
    pdf = norm(Path(sys.argv[2]).read_text(errors="replace"))
    units = [p.text for p in d.paragraphs]
    for t in d.tables:
        for row in t.rows:
            for c in row.cells:
                units.append(c.text)
    units = [norm(u) for u in units if norm(u)]
    missing = [u for u in units if u not in pdf]
    # a paragraph split across a page break still appears, but with a hyphen or a space inserted:
    def loose(u: str) -> bool:
        parts = [p for p in re.split(r"\s+", u) if p]
        return all(p in pdf for p in parts) if parts else True
    missing_soft = [u for u in missing if not loose(u)]
    print(f"docx text units: {len(units)}   missing from the PDF text: {len(missing)} (loosely: {len(missing_soft)})")
    for u in missing_soft[:5]:
        print("  MISSING:", u[:160])
    print(f"pdf chars {len(pdf)}")
    return 0 if not missing_soft else 1


if __name__ == "__main__":
    sys.exit(main())

```


## code/src/stub_judge.py

```text
#!/usr/bin/env python3
"""A stub judge for the official verifier path: OpenAI-compatible, on loopback.

It answers /v1/chat/completions with one grade per criterion, computed by the reader in
checks.py on the candidate text that actually arrived in the request — not on any file of
ours — so a run through test.sh -> verify.py -> grade.py proves the wiring (staging,
rendering, schema, weighting) on real bytes. Its score is the reader's, so a 1.0 here is a
statement about the reader and the pipeline, never about the real judge's opinion.
"""
from __future__ import annotations

import json
import re
import sys
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer

sys.path.insert(0, "/work/les/src")
import checks  # noqa: E402


def candidate_text(prompt: str) -> str:
    """The staged candidate: everything from the first '===== <file> =====' header on."""
    m = re.search(r"\n===== ", prompt)
    return prompt[m.start():] if m else ""


def grades_for(text: str) -> list:
    t = checks.norm(text)
    rows = []
    for idx, w, claim, fn in checks.CHECKS:
        ok = bool(fn(t))
        rows.append({"index": idx, "score": 1.0 if ok else 0.0, "passed": ok,
                     "evidence": "the staged rendering carries the statement" if ok else "no sentence in the staged rendering states it",
                     "reason": claim})
    return rows


class H(BaseHTTPRequestHandler):
    def do_POST(self):
        n = int(self.headers.get("Content-Length", 0))
        body = json.loads(self.rfile.read(n) or b"{}")
        msgs = body.get("messages") or []
        prompt = "\n".join(m.get("content", "") for m in msgs if m.get("role") == "user")
        text = candidate_text(prompt)
        rows = grades_for(text)
        with open("/work/les/out/stub_last_request.txt", "w") as fh:
            fh.write(text)
        payload = {"choices": [{"index": 0, "finish_reason": "stop",
                                "message": {"role": "assistant", "content": json.dumps({"grades": rows})}}],
                   "usage": {"completion_tokens": len(rows)}}
        data = json.dumps(payload).encode()
        self.send_response(200)
        self.send_header("Content-Type", "application/json")
        self.send_header("Content-Length", str(len(data)))
        self.end_headers()
        self.wfile.write(data)

    def log_message(self, *a):
        pass


if __name__ == "__main__":
    port = int(sys.argv[1]) if len(sys.argv) > 1 else 8731
    ThreadingHTTPServer(("127.0.0.1", port), H).serve_forever()

```


## code/src/verify_numbers.py

```text
#!/usr/bin/env python3
"""Second, independent recomputation: different arithmetic (Decimal, not float), different
parsing of the record, and the numbers compared against computed.json.

Nothing here imports compute.py; if the two agree, the agreement is between two paths.
"""
from __future__ import annotations

import csv
import json
import re
from decimal import Decimal, getcontext
from pathlib import Path

getcontext().prec = 40

BASE = Path("/work/ple-dataset/tasks/les-mesh-resolution/environment/files")
C = json.loads(Path("/work/les/computed.json").read_text())
study = (BASE / "Projects/CFDME-2025-041_Ocotillo_WUI/Case/verification/three_grid_study_outputs.txt").read_text()

fails = []


def close(name, mine, theirs, tol=Decimal("0.0005")):
    ok = abs(Decimal(str(mine)) - Decimal(str(theirs))) <= tol
    print(f"{'OK  ' if ok else 'FAIL'} {name}: independent={mine} computed.json={theirs}")
    if not ok:
        fails.append(name)


# --- the record, parsed a second way
blocks = re.split(r"// =+\n// GRID ", study)
assert len(blocks) == 4, len(blocks)
dx_bg = []
zone = []
for b in blocks[1:]:
    dx_bg.append(Decimal(re.search(r"background dx = ([\d.]+) m", b).group(1)))
    zone.append(int(re.search(r"fireRefinementZone size : (\d+)", b).group(1)))
fine = blocks[3]
minvol = Decimal(re.search(r"Min volume = ([\d.eE+-]+)\.", fine).group(1))
levels = Decimal(2) if "carries two additional snappy refinement levels" in study else None
assert levels is not None

D = Decimal("30.1e6")                                   # joules per second, from the memo's 30.1 MW
rho = Decimal("1.204")
cp = Decimal("1005")
T = Decimal("293.15")
g = Decimal("9.81")
grp = D / (rho * cp * T * g.sqrt())
Dstar = (Decimal(2) / Decimal(5) * grp.ln()).exp()      # exp(0.4 ln x), not x**0.4
dx_fire = dx_bg[2] / Decimal(2) ** levels
index = Dstar / dx_fire
probe = Decimal("0.75") * Dstar

close("D* (m)", round(Dstar, 6), C["Dstar_m"], tol=Decimal("0.000002"))
close("fire-region dx (m)", dx_fire, C["dx_fire"], tol=Decimal("0"))
close("index", round(index, 4), round(C["index"], 4), tol=Decimal("0.0001"))
close("probe height (m)", round(probe, 4), round(C["probe_height_m"], 4), tol=Decimal("0.0001"))
close("cube root of min cell volume", round(minvol ** (Decimal(1) / Decimal(3)), 5),
      round(C["dx_fire_minvol"], 5), tol=Decimal("0.00002"))
for i, (zc, dxb) in enumerate(zip(zone, dx_bg)):
    vol = Decimal(zc) * (dxb / Decimal(2) ** levels) ** 3
    close(f"zone volume grid {i+1} (m3)", round(vol, 1), C["zone_vol_by_cells"][i], tol=Decimal("0.1"))

# what the record's own bracket value gives, by the same expression
grp_rec = Decimal("27.08")
D_rec = (Decimal(2) / Decimal(5) * grp_rec.ln()).exp()
close("D* from the memo's own bracket", round(D_rec, 3), C["ratio_to_D"], tol=Decimal("0.0005"))
bound_flag = "band met" if Decimal(5) <= index <= Decimal(10) else "band NOT met"
print(f"     verdict from the independent path: index {index:.3f} -> {bound_flag}")

# the site record, read a second way (csv module, not a regex)
rows = [r for r in csv.DictReader(
    [l for l in (BASE / "Projects/CFDME-2025-041_Ocotillo_WUI/Data/site_met_avra_valley_2015-2024.csv")
     .read_text().splitlines() if l and not l.startswith("#")]) if r.get("T_amb_C")]
close("met record rows", len(rows), C["met_rows"], tol=Decimal("0"))
close("met max temperature (degC)", max(Decimal(r["T_amb_C"]) for r in rows), C["met_Tmax_C"], tol=Decimal("0"))
close("hours at or above 35 degC", sum(1 for r in rows if Decimal(r["T_amb_C"]) >= Decimal(35)), C["met_hot_hours"],
      tol=Decimal("0"))
print("     met record columns:", list(rows[0].keys()))

print(f"\nFAILURES: {len(fails)}" + (f" {fails}" if fails else ""))
raise SystemExit(1 if fails else 0)

```


## deliverables/CFDME-2025-041_resolution_note_revA.txt

```text
THE CFD ME  |  Tucson, Arizona  |  CFDME-2025-041 rev A

MEMORANDUM — LES RESOLUTION BASIS

To

Marguerite Falk, PE — Technical Director & Reviewing Engineer (AZ PE #42798)

From

Gordon Selby, PE — Owner & Principal Engineer (AZ PE #48213)

Project

CFDME-2025-041 — Ocotillo WUI Wildfire, Phase 1

Deliverable under review

Phase 1 Report rev B; peer-review checklist Item 4 (resolution basis), currently open

Date

10/10/2025

Subject

The plume resolution index on the fine grid of the three-grid study, and what the resolution acceptance for Phase 1 rev B rests on

Basis

Documents on the file server as of 10/09/2025. No new simulation has been run for this note.

1.  The answer, before the reasoning

You have Item 4 open on the checklist and will not sign rev B until it closes, so the note leads with the figures and then shows where each one comes from. On the grid the Phase 1 report rests on — the fine grid of the three-grid mesh study, background cell size 0.500 m — the plume resolution index is 29.9, not the 7.5 the record carries. It is computed on a representative fire-region cell edge length of 0.125 m and a plume characteristic diameter of 3.74 m, and it is outside the band of 5 to 10: the band is not met. The resolution acceptance for rev B therefore rests on the resolved turbulent kinetic energy criterion alone, whose 84 % on the plume axis clears the 80 % minimum. Resting it there requires no escalation to you and no ruling from you on which cell edge length the index takes. What keeps Item 4 open is something else: the sub-grid-scale closure of the simulation that produced the reported monitors is not recorded anywhere on the file server, so the item remains open, and it closes by recording the closure in the Mixed-convection LES setup memo — without any new simulation run.

Quantity

Value

Where it comes from

Fire heat release rate, Q

30.1 MW

Scoping memo §5.1

Ambient density, rho_inf

1.204 kg/m³

Scoping memo §5.1 (reference atmosphere)

Ambient temperature, T_inf

293.15 K

Scoping memo §5.1 (reference atmosphere)

Specific heat, cp

1005 J/(kg·K)

Scoping memo §5.1

Plume characteristic diameter, D*

3.74 m

Recomputed here (3.742 m); the record carries 3.76 m

Background cell size, fine grid of the study

0.500 m

Three-grid study, GRID 3 of 3

Refinement of the fire-source zone above background

2 levels

Three-grid study, head note

Representative fire-region cell edge length, dx

0.125 m

0.500 m divided by 2^2

Plume resolution index, D*/dx

29.9

3.742 m / 0.125 m

Required band

5 to 10

QA Verification Standard rev 3, §5.1

Band met?

No

29.9 is 3.0 times the upper limit

Pope resolved-TKE fraction, k_res/(k_res+k_sgs)

84 %

Fine-grid solver summary, on-axis probe

Required minimum

80 %

QA Verification Standard rev 3, §5.2

Pope criterion met?

Yes

84 % against 80 %

2.  Which grid this is about

Rev B retains the fine grid as the reporting grid for the six SOW §3.1 cases (§6.3), reports its centreline monitors in §5.4, and carries the resolution table in Appendix C. That grid is the one the report rests on and the one every number in this note describes. The production mesh is a different mesh — 11,214,862 cells, with the conjugate wall assembly and the ties resolved — and the index below is not defined on it; it is defined on the fine grid of the study, as the standard and the record both put it.

3.  The cell edge length the index takes: 0.125 m

The study's head note records that the fire-source refinement zone, a cylinder over the 3.0 m front segment, carries two additional snappy refinement levels above background in every case. On the fine grid of that study the background cell size is 0.500 m, so two levels of halving put the representative cell edge length across the buoyancy-dominated region — the representative fire-region cell edge length — at 0.125 m. That is the cell size the index takes.

Two independent checks in the same checkMesh block agree. The minimum cell volume on the fine grid is 0.001936 m³, whose cube root is 0.1246 m. And the refined zone holds 1,256,640 cells; taken with the zone volume that cell count implies, 2,454 m³ on the fine grid, that is the same 0.125 m per cell. The same zone gives 2,450 m³ on the coarse grid (156,800 cells) and 2,455 m³ on the medium grid (458,080 cells), so it is one physical cylinder on all three grids and about 2,453 m³ in volume.

The 0.500 m is the far-field background cell, not the fire-region cell. Dividing by it is what makes the record's index read inside the band; no cell in the buoyancy-dominated region is 0.500 m.

4.  The plume length scale: 3.74 m, not 3.76 m

D* is the axisymmetric source length scale of QA §2, D* = ( Q / ( rho_inf · cp · T_inf · sqrt(g) ) )^(2/5). With the record's own inputs — Q = 30.1 MW, rho_inf = 1.204 kg/m³, cp = 1005 J/(kg·K), T_inf = 293.15 K, g = 9.81 m/s² — the bracketed group is a volume of 27.09 m³, and its two-fifths power is 3.742 m, reported as 3.74 m.

The 3.76 m the scoping memo and rev B Appendix C carry does not follow from those inputs. The memo prints the bracketed value as 27.08 m³ correctly and then takes it to 3.76 m, but 27.08 to the power 2/5 is 3.742 m. The slip is in the last step only. It moves no verdict — at 3.76 m the index would be 30.1, still far outside the band — but 3.74 m is the number the record should carry, and the correction should be made where the memo is next revised.

5.  The index and the band: 29.9 against 5 to 10 — the band is not met

On the grid behind the result the Phase 1 report rests on, 3.742 m over 0.125 m gives 29.94, reported as 29.9. QA §5.1 requires that index to fall in the band 5 to 10; 29.9 is 3.0 times the upper limit. The plume resolution criterion is therefore not met on that grid.

The index of 7.52 that the study's resolution-assessment block reports — and the 7.5 (also dimensionless) that Appendix C of rev B reports — is 3.76 m divided by 0.500 m, that is, the characteristic diameter over the background cell; both figures are dimensionless ratios. That is the far-field ratio, and it is not the quantity §5.1 defines.

6.  What the resolution acceptance for rev B rests on, and what needs no decision from you

QA §5 accepts either criterion alone as sufficient evidence of adequate resolution. The plume resolution criterion is not met (§5 above). The Pope resolved-TKE fraction on the fine grid is 84 %, above the 80 % minimum of §5.2. Large-eddy resolution acceptance for rev B therefore rests on the resolved turbulent kinetic energy criterion alone — on the numerical evidence of §5.2, not on engineering judgement asserted in place of it, which §5 forbids.

Resting the acceptance on one criterion requires no escalation to you as Reviewing Engineer. QA §5 requires escalation only where a physical constraint prevents meeting either criterion; one criterion is met here, so there is no constraint to state and nothing to escalate.

No ruling from you as Reviewing Engineer is needed to fix which cell edge length the plume resolution index takes. QA §2 defines the index on the representative fire-region cell edge length, §5.1 fixes that length as the representative cell edge length across the buoyancy-dominated region, and the mesh record supplies it — two refinement levels above a 0.500 m background, i.e. 0.125 m. The denominator follows from the standard and the mesh record rather than from a call that needs review.

One expectation does not apply to this deliverable, and it is the one that would otherwise make the unmet band a bar: the expectation that both resolution acceptance criteria be met is set by QA §5 as the default only for deliverables prepared for authorities having jurisdiction or for life-safety end use. Phase 1 rev B is neither. SOW §1 and §10 procure analysis only; the fire-resistance rating on the assembly is certified separately by a third-party fire-protection engineer under a separate engagement and sits outside the stamp boundary; and no authority-having-jurisdiction submittal is in scope under the SOW. The both-criteria expectation therefore does not apply to Phase 1 rev B.

7.  Was the resolved-energy figure taken inside the region the index describes?

Partly, and what limits the answer is the record rather than the number. The probe is on the plume axis, at 0.75 D* above the source top surface — that is 2.81 m above it — which is inside the buoyancy-dominated flow that §5.2 asks for, and the study samples more than twenty puffing cycles on every grid, which is the statistically converged window §5.2 asks for.

What the files on the file server do not record is the geometry of the fire-source refinement zone itself: neither its radius nor its height. The study records it only as a cylinder over the 3.0 m front segment, together with its cell count; the recorded volume of about 2,453 m³ fixes only the product of radius squared and height. Whether the probe lies inside that zone is therefore reached by inference, not read from a recorded geometry — and the unrecorded radius and height are the recorded fact that this answer rests on.

Two bounds make the inference a strong one. For a cylinder of 2,453 m³ to fit within the 60 m height of the study domain its radius must be at least 3.61 m, so the zone is not a narrow tube around the 3.0 m segment. And for the zone's height to fall below the probe elevation of 2.81 m it would have to be a disc at least 16.7 m in radius, 5.6 times the segment length. On any plausible reading the probe sits inside the zone. The note states that as an inference from a volume, which is what it is; if the containment has to be asserted as fact rather than inferred, the mesher's zone definition — the dictionary entry that carries the cylinder's radius and height — should be placed on the file server, because only the meshing log, which reports levels and cell counts, is there now.

8.  Do the ambient values behind the length scale represent the site?

No. The ambient density of 1.204 kg/m³ and the ambient temperature of 293.15 K behind the plume characteristic diameter are reference-atmosphere values — the scoping memo names them as standard reference-atmosphere conditions — and they are not the site design ambient of 35 to 42 °C that the SOW §3.1 matrix and the Phase 1 report carry. The site meteorological pull agrees with the warmer envelope rather than the reference one: 75 hours of the 240 hourly records sit at or above 35 °C, with a maximum of 41.8 °C.

What the site's conditions would do to the length scale is fixed in direction and not in magnitude. For an ideal gas the product of ambient density and temperature is pressure over the gas constant, so temperature alone leaves D* unchanged: at the same pressure a warmer ambient lowers the density exactly enough to cancel the rise in temperature in the bracketed group. A site-condition D* therefore turns on the site barometric pressure.

A plume characteristic diameter at site conditions cannot be evaluated from the files on the file server. No file records a site pressure, and the meteorological pull carries six columns — timestamp, ambient temperature, relative humidity, 20-ft wind speed, wind direction and solar irradiance — with no pressure column and no elevation. The only elevation anywhere on the file server is the nominal ~750 m in the structural wind-load basis memo, given there for the ASCE 7-22 ground elevation factor of the separate SOW §3.2 load case; that memo takes the nominal site air density as 1.2 kg/m³, which is the reference value already behind D*, and the SOW states that the §3.1 and §3.2 cases are not to be mixed. So there is no recorded site pressure and no recorded site density to work from.

The direction is determinable: any site pressure below the 101,325 Pa standard would lower the ambient density and raise D*, and with it the index — a warmer, higher site makes the resolution worse, not better. The magnitude is not determinable from the record. If a site-condition value is wanted, the site barometric pressure has to enter the record first, as a measured or design value with its source, and D* and the index then follow from it by the same expression.

9.  Can the item close? Not yet

QA §3(a) requires the Verification section of an LES deliverable to state the turbulence treatment used and, for LES, the sub-grid-scale closure. No file on the file server names the closure of the simulation that produced the reported monitors. What is on file is three things, and none of them carries that entry:

The study — the study's summary blocks, which say fireFoam, LES, EDC combustion and fvDOM radiation, and name no sub-grid-scale closure;

The solver log — the only solver log on the server, which is a steady k-epsilon bring-up run inside a SIMPLE loop and is described in its own operator note as a device to reach an initial condition before switching to LES, so it is not the closure of the reported result;

The case-setup record — the only case-setup export on the server, built from the Dry Wash Aerospace case CFDME-2024-018, which carries rhoSimpleFoam, the RAS kOmegaSST closure, no combustion and radiation off — an unrelated RANS envelope, not this simulation's closure.

The modeling standard names the two permissible closures, dynamic Smagorinsky and WALE, but no case record says which one this case used. Item 4 also asks that D*/dx and the Pope fraction be stated in the body of the Verification section rather than in an appendix; rev B states them in Appendix C. The figures in Section 1 above are what that body has to carry.

So the peer-review resolution-basis item remains open, and it stays open until the closure is in the record and the resolution basis is stated in the body of the Verification section rather than in an appendix.

10.  What would settle it

Recording the choice of sub-grid-scale closure in the case setup memo settles the point at issue, and that is what the firm fire and buoyant flows modeling standard requires in §5, which states that the choice between the two closures shall be recorded in the case setup memo. On this project that memo is the Mixed-convection LES setup memo, SOW §4 deliverable 2, which documents the solver stack and the LES resolution basis; the closure should be established from the case build that produced the reported monitors and recorded there.

Rev B's Verification section should then state the closure together with D*, dx, D*/dx and the Pope fraction in the body rather than in an appendix, at which point Item 4 has the entry it is asking for.

The item can be settled without any new simulation run. What is missing is a record entry, not a result: the turbulence treatment of the simulation that produced the monitors is a fact about a case that has already been run, and establishing it from the case build and the case directory is work on the record. Nothing in this note calls for a new LES, a new mesh, or a re-run of the three-grid study.

11.  What this note does not do

It does not revise Phase 1 rev B, and it changes no reported monitor. It does not evaluate a site-condition D*, for the reason given in Section 8. It does not by itself close Item 4: it supplies the resolution figures the Verification section has to state, and it identifies the one entry still missing from the record. Stamp boundary: CFD thermal analysis only; fire-resistance rating certified separately by others.

Prepared by: Gordon Selby, PE — Owner & Principal Engineer, The CFD Mechanical Engineer, LLC (AZ PE #48213)

Date: 10/10/2025

File: CFDME-2025-041_resolution_note_revA.docx  |  rev A  |  10/10/2025
```
