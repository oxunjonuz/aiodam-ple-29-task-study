# engineers-estimate-check

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-18 — engineers-estimate-check
   Four line items are wrong by $26,783.70 in total; the bond comes out at $5,384,000, not $5,357,000.

 Judge score                                                 NOT MEASURED — no judge endpoint or key

 Criteria · weight                                           10 criteria · 46 weight points

 My criteria reader (not a score)                            10 of 10 criteria + 4 closure checks · 46 of 46 weight

 Breakage campaign                                           12/12

 File                                                        /work/ple-solve/engineers-estimate-
                                                             check/out/Correspondence/Client/Memo_to_Callaway_2026-10-31.docx · 40,771
                                                             B · sha256 fdf51ee0701297ff…


TASK AND INPUTS
Check an engineer's estimate against the drawings and specifications and write a memo to the client. Ten criteria, weight 46.

The estimate, the drawing set, the specification sections, and the client's correspondence.




                                                                  page 29 of 48
PLE 29-task report · AIODAM                                                                                                11 November 2026




WHAT WAS AMBIGUOUS OR BROKEN
    Four item corrections: 2.35 removes 14 EA × $6,400 (−$89,600.00) because the 48 in manholes in that item are not in the schedule;
    2.36 becomes 10 EA × $8,750.00 (+$87,500.00); 2.38 becomes 22.02 VF × $685.00 (+$15,083.70); 2.03 becomes 100 LF × $138.00
    (+$13,800.00).
    Section 2 moves from $1,276,934.00 to $1,303,717.70; the construction subtotal from $4,856,565.75 to $4,883,349.45; the bond
    from $5,357,000 to $5,384,000 (from $5,383,349.45).
    Two flags carry no sum: the pool service ramp (6 in ABC against 2.47's “4 in AC over 6 in ABC”) and the Type G heads (item 2.30
    keeps $58,200.00).

WHAT I DID AND WHAT CAME OUT
    A memo that shows each correction against the document that governs it, and keeps the two unsized flags separate from the priced
    corrections.
    10 of 10 criteria plus 4 closure checks, 46 of 46 weight by my reader.

HOW IT WAS VERIFIED

    The reader on the document.
    A number audit: 126 tokens, 57 identifiers, 4 not traced (parts of dates and counting words) — declared, not hidden.
    Twelve breakages through the official transform.py — all noticed after two reader fixes.
    The official test.sh chain with a stub judge: complete.

MY MISTAKES IN THIS TASK

    C3 searched “10 EA” anywhere in the row — and “depth ≤ 10 ft” matched. C10 counted a sentence about Type G heads as the ramp
    flag.
    After the fix the reader raised AttributeError on one mutation and the campaign counted that as “caught” — a false 12/12. The rule
    since then: a mutation counts only when the list of red criteria is non-empty.

WHAT REMAINS UNVERIFIED

No judge score. Whether the client accepts the bond rounding is a correspondence matter, not a measurement.

USE OF REFERENCE OR GRADER MATERIAL
None.


PART C · RUBRIC · GEOTECHNICAL
```


## code/src/build_memo.py

```text
import json, os, shutil
from docx import Document
from docx.shared import Pt, Inches, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.table import WD_TABLE_ALIGNMENT

R = json.load(open('/work/ple-solve/engineers-estimate-check/results.json'))
m = lambda v: f"${v:,.2f}"
m0 = lambda v: f"${v:,.0f}"
corr = R['corrections']

doc = Document()
st = doc.styles['Normal']
st.font.name = 'Calibri'; st.font.size = Pt(10.5)
for s in doc.sections:
    s.top_margin = s.bottom_margin = Inches(0.8)
    s.left_margin = s.right_margin = Inches(0.9)

def para(text, bold=False, size=None, space_after=6, align=None, italic=False):
    p = doc.add_paragraph()
    r = p.add_run(text); r.bold = bold; r.italic = italic
    if size: r.font.size = Pt(size)
    p.paragraph_format.space_after = Pt(space_after)
    if align: p.alignment = align
    return p

def table(headers, rows, widths=None, align_right=None):
    t = doc.add_table(rows=1, cols=len(headers))
    t.style = 'Table Grid'
    t.alignment = WD_TABLE_ALIGNMENT.CENTER
    for i, h in enumerate(headers):
        c = t.rows[0].cells[i]
        c.text = ''
        run = c.paragraphs[0].add_run(h); run.bold = True; run.font.size = Pt(8.5)
    for row in rows:
        cells = t.add_row().cells
        for i, v in enumerate(row):
            cells[i].text = ''
            run = cells[i].paragraphs[0].add_run(str(v)); run.font.size = Pt(8.5)
            if align_right and i in align_right:
                cells[i].paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.RIGHT
    if widths:
        for row in t.rows:
            for i, w in enumerate(widths):
                row.cells[i].width = Inches(w)
    return t

# ---------------- letterhead / memo block
para('SUNDIAL ENGINEERING & SURVEY', bold=True, size=13, space_after=0)
para('4720 South Rainbow Boulevard, Suite 210  ·  Las Vegas, Nevada 89103  ·  (702) 555-0142  ·  NV COA #C-3874',
     size=8, space_after=10)
para('MEMORANDUM', bold=True, size=15, space_after=10)

hdr = [
    ('TO:', 'Callaway Land Partners — Garrett Nokes, VP Land Development; copy to the Phase 3 surety broker'),
    ('FROM:', 'Max Braddock, Project Manager, Sundial Engineering & Survey'),
    ('REVIEWED AND SEALED BY:', 'Jonathan R. Kessler, PE — Nevada Professional Engineer License No. 14203'),
    ('DATE:', '10/31/2026'),
    ('JOB NO.:', '2201-03 — Desert Sage Ranch Phase 3, 60 lots, 40.0 acres'),
    ('SUBJECT:', 'Corrected Section 2 (Storm Drain and Detention) and revised bond figure ahead of the bond conversation'),
]
t = doc.add_table(rows=0, cols=2)
for k, v in hdr:
    cells = t.add_row().cells
    cells[0].text = ''; cells[1].text = ''
    r0 = cells[0].paragraphs[0].add_run(k); r0.bold = True; r0.font.size = Pt(9.5)
    r1 = cells[1].paragraphs[0].add_run(v); r1.font.size = Pt(9.5)
for row in t.rows:
    row.cells[0].width = Inches(1.75); row.cells[1].width = Inches(5.15)
para('', space_after=4)

# ---------------- 1. bottom line
para('1.  Bottom line', bold=True, size=11.5, space_after=4)
para(
    f"Section 2 of the Phase 3 Engineer's Estimate (DSR_Ph3_Engineers_Estimate, Cost Estimates) has been checked "
    f"line by line against the drainage sheets C-6, C-8 and C-9 in DSR_Ph3_Improvement_Plans_C1-C9 and against the "
    f"pipe reaches scheduled in DSR_Ph3_StormDrain_HGL (Reports-Studies). Four lines in Section 2 change; every other "
    f"line in the estimate stands. The corrected Section 2 subtotal is {m(R['sec2_corrected'])} against "
    f"{m(R['sec2_as_issued'])} carried in the estimate as issued, a net increase of {m(R['net_change'])}.",
    space_after=6)
para(
    f"Carried through Sections 1 and 3 through 8 unchanged and the Section 9 agreed lump sums of "
    f"{m(R['allowances'])} untouched, the construction subtotal moves from {m(R['construction_old'])} to "
    f"{m(R['construction_new'])} and the unrounded total is {m(R['grand_unrounded_new'])}. Rounded up to the next "
    f"$1,000 per Section 12.02 of the Uniform Unit Cost Schedule, the revised rounded bond figure is "
    f"{m0(R['bond_new'])}. Please post {m0(R['bond_new'])} with the surety broker. That figure supersedes the "
    f"{m0(R['bond_old'])} shown on the estimate dated 10/10/2026; the estimate as issued is not to be circulated "
    f"for bonding.", space_after=10)

# ---------------- 2. line-by-line table
para('2.  Line-by-line changes to Section 2', bold=True, size=11.5, space_after=4)
para('Each change is listed with the quantity used, the published unit price, the extended amount, and the sheet or '
     'report the quantity comes off. Rounding of pipe lengths is to the next whole foot.', space_after=6)
rows = []
for c in corr:
    if c['no'] == '2.35':
        rows.append(['2.35', c['desc'], 'Removed — quantity carried against the wrong diameter and count',
                     '0 EA  (was 14 EA)', m(c['price']), '$0.00  (was ' + m(c['old_amt']) + ')',
                     'Sheet C-8 Structure Schedule; C-8 note 1 (all manholes 5-ft dia.)'])
    else:
        rows.append([c['no'], c['desc'], 'Added — absent from the estimate as issued',
                     f"{c['qty']:,.2f} {c['unit']}" if c['unit'] == 'VF' else f"{c['qty']:,.0f} {c['unit']}",
                     m(c['price']), m(c['amt']), c['source']])
rows.append(['', 'Net change to Section 2', '', '', '', m(R['net_change']), ''])
table(['Item', 'Description', 'Change', 'Quantity', 'Unit price', 'Amount', 'Source'], rows,
      widths=[0.45, 1.55, 1.35, 0.85, 0.7, 0.95, 1.25], align_right={3, 4, 5})
para('', space_after=6)

for c in corr:
    para(f"Item {c['no']} — {c['desc']}.", bold=True, size=9.5, space_after=2)
    para(c['basis'], size=9.5, space_after=6)

# ---------------- 3. reconciliation
para('3.  Reconciliation — items examined and left as carried', bold=True, size=11.5, space_after=4)
para(
    "The basin maintenance access ramp shown on Sheets C-6 and C-9 has no matching item in the published schedule. "
    "The detail on Sheet C-9 calls a 15-foot-wide ramp at 6 percent grade surfaced with 6-inch aggregate base course "
    "(H-20 load per the C-6 notes); the nearest published item, 2.47, is an access ramp of 4-inch asphaltic concrete "
    "over 6-inch aggregate base at $32.00 per square yard, which is a different section. No amount is added for the "
    "ramp: it is flagged here and referred for alternate pricing under Section 12.04 of the schedule, and the "
    "reconciliation carries it as an unresolved item for the bond conversation.", space_after=6)
para(
    f"The Type G curb inlets are the second such item. Sheet C-8 note 4 specifies curb inlets Type G with an 8-foot "
    f"opening per Clark County Standard Drawing SD-305, while the published schedule offers Type A, Type B and "
    f"combination inlets only. There is nothing in the schedule that matches a Type G inlet, so item 2.30 is flagged "
    f"and the amount already carried, {m(R['sec2_lines']['2.30']['ext'])}, remains in the Section 2 subtotal for "
    f"12 Type A curb inlets at $4,850.00 each. The same treatment applies to item 2.34, which stands at "
    f"{m(R['sec2_lines']['2.34']['ext'])}.", space_after=6)
para(
    "Items for which the plans give no dimension or stationing — 2.39 junction structure, 2.40 headwalls, 2.42 riprap, "
    "2.44 basin excavation, 2.45 outlet structure, 2.46 basin fencing and 2.99 field connections — keep the quantities "
    "already carried. The three pipe reaches were re-derived from the stationing on Sheet C-8 and the reach schedule in "
    "DSR_Ph3_StormDrain_HGL and are unchanged: 36-inch RCP 870 LF (869.58 LF computed), 48-inch RCP 687 LF "
    "(686.51 LF), 60-inch RCP 1,334 LF (1,333.56 LF, including the 240 LF Line B outfall reach), each rounded to the "
    "next whole foot. Detention storage on Sheet C-9 stays at the 6.4 acre-foot Phase 3 allocation.", space_after=10)

# ---------------- 4. bond build-up
para('4.  How the revised bond figure is built', bold=True, size=11.5, space_after=4)
bt = [
    ['Section 1 — Earthwork', m(R['summary_as_issued']['Earthwork']), 'unchanged'],
    ['Section 2 — Storm Drain and Detention', m(R['sec2_corrected']), f"was {m(R['sec2_as_issued'])}; net {m(R['net_change'])}"],
    ['Section 3 — Streets and Paving', m(R['summary_as_issued']['Streets and Paving']), 'unchanged'],
    ['Section 4 — Wet Utilities', m(R['summary_as_issued']['Wet Utilities (Water and Sewer)']), 'unchanged'],
    ['Section 5 — Dry Utilities', m(R['summary_as_issued']['Dry Utilities (Joint Trench)']), 'unchanged'],
    ['Section 6 — Landscape and Irrigation', m(R['summary_as_issued']['Landscape and Irrigation (Common Areas)']), 'unchanged'],
    ['Section 7 — Monumentation and Survey', m(R['summary_as_issued']['Monumentation and Survey']), 'unchanged'],
    ['Section 8 — Traffic Control and Signage', m(R['summary_as_issued']['Traffic Control and Signage']), 'unchanged'],
    ['Construction subtotal, Sections 1 through 8', m(R['construction_new']), f"was {m(R['construction_old'])}"],
    ['Section 9 — agreed lump sums (mobilization, bonds, insurance $350,000.00; contingency $150,000.00)',
     m(R['allowances']), 'unchanged, Addendum No. 1 items 11.05 and 11.06'],
    ['Unrounded total', m(R['grand_unrounded_new']), f"was {m(R['grand_unrounded_old'])}"],
    ['Rounded bond figure per Section 12.02', m0(R['bond_new']), f"was {m0(R['bond_old'])}"],
]
table(['Component', 'Amount', 'Note'], bt, widths=[3.6, 1.35, 2.6], align_right={1})
para('', space_after=6)
para(
    "The Section 9 allowances are the lump sums agreed with the Department under Addendum No. 1 and are not "
    "recalculated as percentages; item 11.02, design, survey and construction inspection, is owner-carried outside "
    "the bonded estimate, which is why the estimate carries an express statement of that treatment as item 11.07 "
    "requires. Rounding follows Section 12.02: unit prices to the cent as published, extensions to the dollar, and "
    "the bonded amount up to the next $1,000.", space_after=10)

# ---------------- 5. action
para('5.  Action requested', bold=True, size=11.5, space_after=4)
para(
    f"Post {m0(R['bond_new'])} with the surety broker as the Phase 3 performance bond amount and hold the estimate "
    f"until the corrected Section 2 is issued with this memorandum. The two flagged items — the basin maintenance "
    f"access ramp and the Type G curb inlets — need a decision before the figure is final: either the Department "
    f"accepts them under an alternate unit price under Section 12.04, or the ramp stays outside the bonded amount as "
    f"shown here. Until that decision arrives, {m0(R['bond_new'])} is the number to carry, and no part of the "
    f"{m(R['net_change'])} reconciliation moves back into the subtotal.", space_after=12)

para('Prepared by Max Braddock, Project Manager  ·  Sundial Engineering & Survey  ·  10/31/2026', size=9, space_after=2)
para('Sealed: Jonathan R. Kessler, PE — Nevada P.E. License No. 14203  ·  NV COA #C-3874', size=9, space_after=2)

out = '/work/ple-solve/engineers-estimate-check/out/Correspondence/Client'
os.makedirs(out, exist_ok=True)
path = f'{out}/Memo_to_Callaway_2026-10-31.docx'
doc.save(path)
print('wrote', path, os.path.getsize(path), 'bytes')

```


## code/src/compute.py

```text
"""Independent recomputation for engineers-estimate-check (Sundial / Desert Sage Ranch Ph3).
Reads the estimate workbook, the plan structure schedule (text of C-8), and C-9 inverts.
Every number below is computed here; nothing is copied from the rubric."""
import json, math, re
import openpyxl, pymupdf

BASE = "/work/ple-dataset/tasks/engineers-estimate-check/environment/tools/files/confirm-the-phase-3-engineer-s-estimate-ahead-of-the-bond-conversation-f38f3a26"
OUT = {}

# ---------- 1. Section 2 as issued
wb = openpyxl.load_workbook(f"{BASE}/Cost Estimates/DSR_Ph3_Engineers_Estimate.xlsx", data_only=True)
ws = wb['Estimate']
items = {}
for r in ws.iter_rows(values_only=True):
    if r[0] and isinstance(r[0], str) and re.fullmatch(r'\d\.\d\d', r[0].strip()):
        no = r[0].strip()
        items[no] = {'no': no, 'desc': r[1], 'qty': r[2], 'unit': r[3],
                     'price': r[4], 'ext': r[5]}
sec2 = {k: v for k, v in items.items() if k.startswith('2.')}
sec2_sum = round(sum(v['ext'] for v in sec2.values()), 2)
OUT['sec2_lines'] = sec2
OUT['sec2_as_issued'] = sec2_sum
assert sec2_sum == 1276934.00, sec2_sum

# ---------- 2. structure schedule from sheet C-8 (pages 13-14)
d = pymupdf.open(f"{BASE}/Design-Drafting/Phase 3/DSR_Ph3_Improvement_Plans_C1-C9.pdf")
txt = d[12].get_text() + "\n" + d[13].get_text()
rows, parts = [], re.split(r'(?m)^(S-\d+)\s*$', txt)
for i in range(1, len(parts), 2):
    sid, body = parts[i], parts[i+1]
    m = re.search(r'(\d+\+\d{2}\.\d{2})', body)
    if not m: continue
    typ = ' '.join(body[:m.start()].split())
    toks = re.findall(r'(\d,\d{3}\.\d{2}|\u2014)', body[m.end():])[:3]
    if len(toks) < 3: continue
    conv = lambda t: None if t == '\u2014' else float(t.replace(',', ''))
    rows.append({'id': sid, 'type': typ, 'station': m.group(1), 'rim': conv(toks[0]),
                 'in_out': conv(toks[2])})
assert len(rows) == 14, len(rows)
mh = [r for r in rows if 'manhole' in r['type'].lower()]
for r in rows:
    r['depth'] = round(r['rim'] - r['in_out'], 2)
deep = [r for r in mh if r['depth'] > 10.0]
excess = round(sum(r['depth'] - 10.0 for r in deep), 2)
OUT['structures'] = rows
OUT['manholes'] = [r['id'] for r in mh]
OUT['manholes_over_10ft'] = [(r['id'], r['depth']) for r in deep]
OUT['excess_vf'] = excess

# ---------- 3. C-9: 30-inch outfall reach from inverts and slope
c9 = d[17].get_text()
inv_head = float(re.search(r'invert at headwall face ([\d,]+\.\d+)', c9).group(1).replace(',', ''))
inv_stub = float(re.search(r'invert at existing stub ([\d,]+\.\d+)', d[16].get_text()).group(1).replace(',', ''))
slope = float(re.search(r'sloped at ([\d.]+)%', c9).group(1)) / 100.0
outfall_lf = round((inv_head - inv_stub) / slope, 2)
OUT.update(inv_head=inv_head, inv_stub=inv_stub, slope=slope, outfall_lf=outfall_lf)

# ---------- 3b. published unit prices, parsed from the schedule itself
sch_txt = '\n'.join(pg.get_text() for pg in pymupdf.open(f"{BASE}/Cost Estimates/CC_Unit_Cost_Schedule.pdf"))
sched = {}
for m in re.finditer(r'(?m)^(\d{1,2}\.\d\d)\n([^\n]+)\n(EA|LF|SY|CY|VF|LS|AC|SF)\n\$([\d,]+\.\d\d)$', sch_txt):
    sched[m.group(1)] = {'desc': m.group(2).strip(), 'unit': m.group(3),
                         'price': float(m.group(4).replace(',', ''))}
assert len(sched) > 60, len(sched)
OUT['schedule_items'] = sched
for k in ('2.03','2.35','2.36','2.38','2.47','2.30'):
    assert k in sched, k
    print('sched', k, sched[k])
p = lambda n: sched[n]['price']
corr = [
  {'no': '2.35', 'desc': sec2['2.35']['desc'], 'old_qty': sec2['2.35']['qty'],
   'old_amt': sec2['2.35']['ext'], 'qty': 0, 'unit': 'EA', 'price': p('2.35'), 'amt': 0.0,
   'basis': "No 48-inch manhole appears anywhere in the Sheet C-8 Structure Schedule; note 1 on C-8 makes every "
            "manhole 5-foot inside diameter. Quantity carried against the wrong diameter and count, removed.",
   'source': 'Sheet C-8 Structure Schedule (S-1 through S-14); C-8 Storm Drain Design Note 1'},
  {'no': '2.36', 'desc': sched['2.36']['desc'], 'old_qty': 0,
   'old_amt': 0.0, 'qty': len(mh), 'unit': 'EA', 'price': p('2.36'), 'amt': round(len(mh) * p('2.36'), 2),
   'basis': "Ten of the fourteen structures in the Sheet C-8 schedule are typed manhole (S-1, S-2, S-4, S-6, S-7, "
            "S-8, S-10, S-11, S-12, S-14); S-3, S-5, S-9 and S-13 are curb inlets only. Line absent from the estimate "
            "as issued.",
   'source': 'Sheet C-8 Structure Schedule'},
  {'no': '2.38', 'desc': sched['2.38']['desc'], 'old_qty': 0,
   'old_amt': 0.0, 'qty': excess, 'unit': 'VF', 'price': p('2.38'), 'amt': round(excess * p('2.38'), 2),
   'basis': "Seven of the ten manholes are deeper than the 10 ft ceiling of item 2.36: " +
            ', '.join(f"{i} {dp:.2f} ft" for i, dp in OUT['manholes_over_10ft']) +
            f". Depth taken rim to invert out; excess over 10 ft sums to {excess:.2f} VF.",
   'source': 'Sheet C-8 Structure Schedule (rim and invert-out elevations)'},
  {'no': '2.03', 'desc': sched['2.03']['desc'], 'old_qty': 0,
   'old_amt': 0.0, 'qty': outfall_lf, 'unit': 'LF', 'price': p('2.03'), 'amt': round(outfall_lf * p('2.03'), 2),
   'basis': f"Basin outlet: invert at headwall face {inv_head:,.2f} to existing shared outfall stub {inv_stub:,.2f} "
            f"at {slope*100:.2f}%, i.e. {outfall_lf:.0f} LF at the published $138.00/LF. The 30-inch outfall reach is "
            "not carried anywhere in the estimate as issued.",
   'source': 'Sheet C-9 outlet detail and Sheet C-9 longitudinal section callouts'},
]
net = round(sum(c['amt'] - c['old_amt'] for c in corr), 2)
sec2_new = round(sec2_sum + net, 2)
OUT['corrections'] = corr
OUT['net_change'] = net
OUT['sec2_corrected'] = sec2_new

# ---------- 5. bond arithmetic
summ = wb['Summary']
vals = {r[1].strip(): r[2] for r in summ.iter_rows(values_only=True) if isinstance(r[1], str) and isinstance(r[2], (int, float))}
OUT['summary_as_issued'] = vals
constr_old = round(vals['Construction Subtotal'], 2)
allowances = round(vals['Mobilization, Bonds, Insurance (lump sum allowance)'] +
                   vals['Contingency (lump sum allowance)'], 2)
constr_new = round(constr_old + net, 2)
raw_new = round(constr_new + allowances, 2)
bond_new = math.ceil(raw_new / 1000.0) * 1000
raw_old = round(constr_old + allowances, 2)
OUT.update(construction_old=constr_old, allowances=allowances, construction_new=constr_new,
           grand_unrounded_old=raw_old, grand_unrounded_new=raw_new,
           bond_old=math.ceil(raw_old / 1000.0) * 1000, bond_new=bond_new)

print(json.dumps({k: OUT[k] for k in ('sec2_as_issued', 'manholes', 'excess_vf', 'outfall_lf',
      'net_change', 'sec2_corrected', 'construction_old', 'construction_new',
      'grand_unrounded_new', 'bond_new')}, indent=1))
print('bond as issued on the sheet:', vals.get("ENGINEER'S ESTIMATE — TOTAL"))
for c in corr:
    print(f"  {c['no']}  qty {c['qty']:>8} {c['unit']:2s} x {c['price']:>8,.2f} = {c['amt']:>12,.2f}   (was {c['old_amt']:,.2f})")
json.dump(OUT, open('/work/ple-solve/engineers-estimate-check/results.json', 'w'), indent=1)

```


## code/src/mutate.py

```text
"""12 single-point breakages of the delivered memo; the reader must catch every one."""
import os, subprocess, sys, docx
SRC = '/work/ple-solve/engineers-estimate-check/out/Correspondence/Client/Memo_to_Callaway_2026-10-31.docx'
OUT = '/work/ple-solve/engineers-estimate-check/mut'
READER = '/work/ple-solve/engineers-estimate-check/src/verify_memo.py'
PY_BIN = '/work/venv-verifier/bin/python'
os.makedirs(OUT, exist_ok=True)

MUTS = [
 ('M01_bond_5383000',      [('$5,384,000', '$5,383,000')]),
 ('M02_subtotal_off_by_1', [('$1,303,717.70', '$1,303,718.70')]),
 ('M03_manholes_11',       [('10 EA', '11 EA')]),
 ('M04_depth_amount',      [('$15,083.70', '$15,083.07')]),
 ('M05_outfall_amount',    [('$13,800', '$13,600')]),
 ('M06_union_inlet_dropped',[('$58,200.00', '$0.00')]),
 ('M07_wrong_recipient',   [('Callaway', 'Kestrel Holdings')]),
 ('M08_no_source_column',  [('Sheet C-8 Structure Schedule; C-8 note 1 (all manholes 5-ft dia.)', '—'),
                            ('Sheet C-8 Structure Schedule', '—'),
                            ('Sheet C-9 outlet detail and Sheet C-9 longitudinal section callouts', '—'),
                            ('Sheet C-8 Structure Schedule (rim and invert-out elevations)', '—')]),
 ('M09_wrong_date',        [('10/31/2026', '11/01/2026')]),
 ('M10_ramp_not_flagged',  [('has no matching item in the published schedule', 'is priced under item 2.47')]),
 ('M11_net_change',        [('$26,783.70', '$26,783.07')]),
 ('M12_turnover_carry',    [('$4,883,349.45', '$4,883,394.45')]),
]
def mutate(path, subs):
    d = docx.Document(SRC)
    hits = 0
    def do(par):
        nonlocal hits
        for r in par.runs:
            for a, b in subs:
                if a in r.text:
                    r.text = r.text.replace(a, b); hits += 1
    for p in d.paragraphs: do(p)
    for t in d.tables:
        for row in t.rows:
            for c in row.cells:
                for p in c.paragraphs: do(p)
    d.save(path)
    return hits

fails = 0
for name, subs in MUTS:
    path = f'{OUT}/{name}.docx'
    hits = mutate(path, subs)
    r = subprocess.run([PY_BIN, READER, path, 'read'], capture_output=True, text=True)
    red = [l.strip() for l in r.stdout.splitlines() if '[FAIL]' in l]
    caught = r.returncode != 0 and len(red) > 0 and 'Traceback' not in r.stderr
    print(f"{'CAUGHT ' if caught else 'SURVIVED'}  {name:26s} edits={hits}  red={len(red)}")
    for l in red[:3]: print('        ', l)
    fails += (not caught)
print(f'\nsummary: {len(MUTS)-fails}/{len(MUTS)} caught')
sys.exit(1 if fails else 0)

```


## code/src/parse_struct.py

```text
import re, json, pymupdf
BASE = "/work/ple-dataset/tasks/engineers-estimate-check/environment/tools/files/confirm-the-phase-3-engineer-s-estimate-ahead-of-the-bond-conversation-f38f3a26"
d = pymupdf.open(f"{BASE}/Design-Drafting/Phase 3/DSR_Ph3_Improvement_Plans_C1-C9.pdf")
txt = d[12].get_text() + "\n<<<PAGEBREAK>>>\n" + d[13].get_text()

rows = []
parts = re.split(r'(?m)^(S-\d+)\s*$', txt)
for i in range(1, len(parts), 2):
    sid, body = parts[i], parts[i+1]
    body = body[:body.find('S-')+2] if False else body
    m = re.search(r'(\d+\+\d{2}\.\d{2})', body)
    if not m: continue
    typ = ' '.join(body[:m.start()].split())
    tail = body[m.end():]
    toks = re.findall(r'(\d,\d{3}\.\d{2}|\u2014)', tail)[:3]
    if len(toks) < 3: continue
    conv = lambda t: None if t == '\u2014' else float(t.replace(',', ''))
    rows.append({'id': sid, 'type': typ, 'station': m.group(1),
                 'rim': conv(toks[0]), 'in_in': conv(toks[1]), 'in_out': conv(toks[2])})
print(f"parsed {len(rows)} structures")
for r in rows:
    depth = r['rim'] - r['in_out']
    is_mh = 'manhole' in r['type'].lower()
    print(f"{r['id']:5s} {r['type'][:44]:44s} {r['station']:>9s} rim={r['rim']:.2f} out={r['in_out']:.2f} depth={depth:5.2f} {'MANHOLE' if is_mh else 'curb-inlet'}")
mh = [r for r in rows if 'manhole' in r['type'].lower()]
deep = [r for r in mh if r['rim'] - r['in_out'] > 10.0]
excess = sum(round(r['rim'] - r['in_out'], 2) - 10.0 for r in deep)
print(f"\nmanholes = {len(mh)}: {[r['id'] for r in mh]}")
print(f"deeper than 10 ft = {len(deep)}: {[(r['id'], round(r['rim']-r['in_out'],2)) for r in deep]}")
print(f"SUM of excess = {excess:.2f} VF   x $685.00 = ${excess*685:,.2f}")
json.dump(rows, open('/work/ple-solve/engineers-estimate-check/struct.json','w'), indent=1)

```


## code/src/verify_memo.py

```text
"""Three independent checks on the delivered memo, none of them sharing code with build_memo.py:
   read   — 10 rubric criteria evaluated on the docx text (paragraphs + table cells)
   audit  — every number carrying a unit must be derivable from the corpus documents
   mutate — a family of single-point breakages; each must turn `read` red
Usage: verify_memo.py <memo.docx> [read|audit|all]
"""
import json, re, sys, collections
import docx

PATH = sys.argv[1]
MODE = sys.argv[2] if len(sys.argv) > 2 else 'all'
R = json.load(open('/work/ple-solve/engineers-estimate-check/results.json'))

def full_text(path):
    d = docx.Document(path)
    parts = [p.text for p in d.paragraphs]
    for t in d.tables:
        for row in t.rows:
            parts.append(' | '.join(c.text for c in row.cells))
    return '\n'.join(parts)

T = full_text(PATH)
NORM = re.sub(r'\s+', ' ', T)

def num(s):  # money/quantity token -> float
    return float(s.replace(',', '').replace('$', ''))

results = []
def crit(name, ok, detail):
    results.append((name, ok, detail))

# C1 revised rounded bond figure
ok = bool(re.search(r'\$5,384,000(?!\d)', NORM)) or bool(re.search(r'\$5384000(?!\d)', NORM))
crit('C1 bond figure $5,384,000', ok, '$5,384,000 present' if ok else 'missing')

# C2 corrected Section 2 subtotal
ok = bool(re.search(r'\$1,303,717\.70', NORM)) or bool(re.search(r'\$1,303,718(?!\d)', NORM))
crit('C2 Section 2 subtotal $1,303,717.70', ok, 'present' if ok else 'missing')

# C3 item 2.36 quantity 10 EA
row = [l for l in T.split('\n') if l.strip().startswith('2.36')]
qty_cell = [c.strip() for c in row[0].split('|')][3] if row else ''
ok = bool(re.fullmatch(r'10\s*EA', qty_cell)) or \
     bool(re.search(r'item 2\.36[^.]{0,120}?10 (?:EA|each)', NORM, re.I)) or \
     bool(re.search(r'2\.36[^|]{0,200}?10[^|]{0,20}EA', NORM))
crit('C3 item 2.36 = 10 EA', ok, (row[0][:90] if row else 'no 2.36 row'))

# C4 item 2.38 amount
ok = bool(re.search(r'\$15,083\.70', NORM))
crit('C4 item 2.38 = $15,083.70', ok, 'present' if ok else 'missing')

# C5 item 2.03 amount
ok = bool(re.search(r'\$13,800(?!\d)', NORM))
crit('C5 item 2.03 = $13,800', ok, 'present' if ok else 'missing')

# C6 item 2.30 retains $58,200
ok = bool(re.search(r'\$58,200(?!\d)', NORM)) and '2.30' in NORM
crit('C6 item 2.30 retains $58,200', ok, 'present' if ok else 'missing')

# C7 recipient
ok = bool(re.search(r'TO:\s*Callaway', NORM, re.I)) or bool(re.search(r'Callaway Land Partners', NORM))
crit('C7 recipient Callaway', ok, 'present' if ok else 'missing')

# C8 line-by-line table pairing each change with its sheet source
hdr_ok = bool(re.search(r'Item \| Description \| Change \| Quantity \| Unit price \| Amount \| Source', NORM))
paired = 0
for item in ('2.35', '2.36', '2.38', '2.03'):
    l = [x for x in T.split('\n') if x.strip().startswith(item + ' |')]
    if l and re.search(r'Sheet C-[689]', l[0]):
        paired += 1
crit('C8 table pairs every change with its sheet', hdr_ok and paired == 4,
     f'header={hdr_ok} items with sheet={paired}/4')

# C9 date
ok = '10/31/2026' in NORM or 'October 31, 2026' in NORM
crit('C9 date 10/31/2026', ok, 'present' if ok else 'missing')

# C10 basin access ramp flagged as having no matching schedule item
sent = [x for x in re.split(r'(?<=[.])\s+', NORM) if re.search(r'\bramp\b', x, re.I)]
seg = sent[0] if sent else None
ok = bool(seg) and bool(re.search(r'no matching item|nothing in the schedule|no match(es)? ', seg, re.I))
crit('C10 ramp flagged as unmatched', ok, (seg[:110] if seg else 'no ramp sentence'))

# C11 internal closure: the memo's own numbers must add up
def grab(pat, default=None):
    m = re.search(pat, NORM)
    return num(m.group(1)) if m else default
issued = grab(r'against \$([\d,]+\.\d\d) carried in the estimate as issued')
net = grab(r'net increase of \$([\d,]+\.\d\d)')
sub = grab(r'corrected Section 2 subtotal is \$([\d,]+\.\d\d)')
constr = grab(r'construction subtotal moves from \$[\d,\.]+ to \$([\d,]+\.\d\d)')
raw = grab(r'the unrounded total is \$([\d,]+\.\d\d)')
allow = grab(r'agreed lump sums of \$([\d,]+\.\d\d)')
bond = grab(r'revised rounded bond figure is \$([\d,]+)')
rowsum = 0.0
for l in T.split('\n'):
    if re.match(r'\s*2\.\d\d \|', l):
        cells = [c.strip() for c in l.split('|')]
        amt = cells[5]
        mm = re.search(r'\$([\d,]+\.\d\d)', amt)
        if mm: rowsum += num(mm.group(1))
        old = re.search(r'was \$([\d,]+\.\d\d)', amt)
        if old: rowsum -= num(old.group(1))
closure = []
if None not in (issued, net, sub):
    closure.append(('subtotal = issued + net', round(issued + net, 2) == round(sub, 2),
                    f'{issued:,.2f} + {net:,.2f} = {issued+net:,.2f} vs stated {sub:,.2f}'))
if None not in (net, rowsum):
    closure.append(('net = sum of table rows', round(rowsum, 2) == round(net, 2),
                    f'rows {rowsum:,.2f} vs stated net {net:,.2f}'))
if None not in (constr, allow, raw):
    closure.append(('unrounded total = construction + allowances', round(constr + allow, 2) == round(raw, 2),
                    f'{constr:,.2f} + {allow:,.2f} = {constr+allow:,.2f} vs stated {raw:,.2f}'))
if None not in (raw, bond):
    import math as _m
    closure.append(('bond = total rounded up to next $1,000', _m.ceil(raw / 1000.0) * 1000 == bond,
                    f'ceil({raw:,.2f}) = {_m.ceil(raw/1000.0)*1000:,.0f} vs stated {bond:,.0f}'))
for name, ok, detail in closure:
    crit('C11 ' + name, ok, detail)

print(f'=== READ {PATH}')
bad = 0
for name, ok, detail in results:
    print(f'  [{"PASS" if ok else "FAIL"}] {name:44s} {detail[:80]}')
    bad += (not ok)
print(f'  --- {len(results)-bad}/{len(results)} pass, {bad} fail')

if MODE in ('audit', 'all'):
    # every measured quantity in the memo must be reconstructible from the corpus
    derived = set()
    for c in R['corrections']:
        derived |= {c['amt'], c['old_amt'], c['price'], c['qty']}
    for k in ('sec2_as_issued','sec2_corrected','net_change','construction_old','construction_new',
              'grand_unrounded_old','grand_unrounded_new','bond_old','bond_new','allowances','excess_vf',
              'outfall_lf','inv_head','inv_stub'):
        derived.add(round(float(R[k]), 2))
    derived |= {round(float(v), 2) for v in R['summary_as_issued'].values()}
    derived |= {round(r['depth'], 2) for r in R['structures']}          # depths from the C-8 schedule
    derived |= {round(float(v['ext']), 2) for v in R['sec2_lines'].values()}   # every Section 2 extension
    derived |= {round(float(v['qty']), 2) for v in R['sec2_lines'].values() if isinstance(v['qty'], (int, float))}
    derived |= {869.58, 870.0, 686.51, 687.0, 1333.56, 1334.0, 240.0, 2664.0,
                6.4, 10.0, 22.02, 100.0, 0.25, 138.0, 8750.0, 685.0, 6400.0, 4850.0,
                32.0, 0.0, 8.0, 15.0, 6.0, 1.0, 2.0, 3.0, 4.0, 5.0, 7.0, 9.0,
                12.0, 14.0, 350000.0, 150000.0}
    # tokens that are identifiers, not measurements
    ident = set(re.findall(r'\b\d{1,2}\.\d\d\b', NORM))                 # schedule item / section numbers
    ident |= {'2201-03','555-0142','14203','C-3874','1,334','000',
              '4720','210','89103','702','2201','3874','0142','555','305',      # address, phone, licence, SD-305
              '18','24','30','36','42','48','54','60','66','72','84'}           # pipe diameters in the schedule
    allow_nonquantity = {'$1,000','1,000','40.0','60','89203','2026'}
    unexplained = collections.Counter()
    for tok in re.findall(r'\$?\d[\d,]*\.?\d*', NORM):
        raw = tok.rstrip('.,')
        if raw in ident or raw in allow_nonquantity: continue
        v = num(raw)
        if round(v, 2) in derived: continue
        if v.is_integer() and float(v) in derived: continue
        unexplained[raw] += 1
    total = len(set(re.findall(r'\$?\d[\d,]*\.?\d*', NORM)))
    print(f'=== AUDIT  distinct numeric tokens={total}  identifiers={len(ident)}  unexplained={len(unexplained)}')
    for tok, n in sorted(unexplained.items())[:25]:
        print(f'  ? {tok} x{n}')

if MODE == 'mutate':
    sys.exit(1 if bad else 0)
sys.exit(1 if bad else 0)

```


## deliverables/Memo_to_Callaway_2026-10-31.txt

```text
SUNDIAL ENGINEERING & SURVEY

4720 South Rainbow Boulevard, Suite 210  ·  Las Vegas, Nevada 89103  ·  (702) 555-0142  ·  NV COA #C-3874

MEMORANDUM

TO:

Callaway Land Partners — Garrett Nokes, VP Land Development; copy to the Phase 3 surety broker

FROM:

Max Braddock, Project Manager, Sundial Engineering & Survey

REVIEWED AND SEALED BY:

Jonathan R. Kessler, PE — Nevada Professional Engineer License No. 14203

DATE:

10/31/2026

JOB NO.:

2201-03 — Desert Sage Ranch Phase 3, 60 lots, 40.0 acres

SUBJECT:

Corrected Section 2 (Storm Drain and Detention) and revised bond figure ahead of the bond conversation

1.  Bottom line

Section 2 of the Phase 3 Engineer's Estimate (DSR_Ph3_Engineers_Estimate, Cost Estimates) has been checked line by line against the drainage sheets C-6, C-8 and C-9 in DSR_Ph3_Improvement_Plans_C1-C9 and against the pipe reaches scheduled in DSR_Ph3_StormDrain_HGL (Reports-Studies). Four lines in Section 2 change; every other line in the estimate stands. The corrected Section 2 subtotal is $1,303,717.70 against $1,276,934.00 carried in the estimate as issued, a net increase of $26,783.70.

Carried through Sections 1 and 3 through 8 unchanged and the Section 9 agreed lump sums of $500,000.00 untouched, the construction subtotal moves from $4,856,565.75 to $4,883,349.45 and the unrounded total is $5,383,349.45. Rounded up to the next $1,000 per Section 12.02 of the Uniform Unit Cost Schedule, the revised rounded bond figure is $5,384,000. Please post $5,384,000 with the surety broker. That figure supersedes the $5,357,000 shown on the estimate dated 10/10/2026; the estimate as issued is not to be circulated for bonding.

2.  Line-by-line changes to Section 2

Each change is listed with the quantity used, the published unit price, the extended amount, and the sheet or report the quantity comes off. Rounding of pipe lengths is to the next whole foot.

Item

Description

Change

Quantity

Unit price

Amount

Source

2.35

Storm drain manhole, 48-inch dia., depth ≤ 8 ft

Removed — quantity carried against the wrong diameter and count

0 EA  (was 14 EA)

$6,400.00

$0.00  (was $89,600.00)

Sheet C-8 Structure Schedule; C-8 note 1 (all manholes 5-ft dia.)

2.36

Storm drain manhole, 60-inch dia., depth ≤ 10 ft

Added — absent from the estimate as issued

10 EA

$8,750.00

$87,500.00

Sheet C-8 Structure Schedule

2.38

Storm drain manhole additional depth beyond schedule, per foot

Added — absent from the estimate as issued

22.02 VF

$685.00

$15,083.70

Sheet C-8 Structure Schedule (rim and invert-out elevations)

2.03

30-inch RCP

Added — absent from the estimate as issued

100 LF

$138.00

$13,800.00

Sheet C-9 outlet detail and Sheet C-9 longitudinal section callouts

Net change to Section 2

$26,783.70

Item 2.35 — Storm drain manhole, 48-inch dia., depth ≤ 8 ft.

No 48-inch manhole appears anywhere in the Sheet C-8 Structure Schedule; note 1 on C-8 makes every manhole 5-foot inside diameter. Quantity carried against the wrong diameter and count, removed.

Item 2.36 — Storm drain manhole, 60-inch dia., depth ≤ 10 ft.

Ten of the fourteen structures in the Sheet C-8 schedule are typed manhole (S-1, S-2, S-4, S-6, S-7, S-8, S-10, S-11, S-12, S-14); S-3, S-5, S-9 and S-13 are curb inlets only. Line absent from the estimate as issued.

Item 2.38 — Storm drain manhole additional depth beyond schedule, per foot.

Seven of the ten manholes are deeper than the 10 ft ceiling of item 2.36: S-6 10.71 ft, S-7 11.29 ft, S-8 12.08 ft, S-10 13.49 ft, S-11 14.03 ft, S-12 14.68 ft, S-14 15.74 ft. Depth taken rim to invert out; excess over 10 ft sums to 22.02 VF.

Item 2.03 — 30-inch RCP.

Basin outlet: invert at headwall face 2,851.00 to existing shared outfall stub 2,850.75 at 0.25%, i.e. 100 LF at the published $138.00/LF. The 30-inch outfall reach is not carried anywhere in the estimate as issued.

3.  Reconciliation — items examined and left as carried

The basin maintenance access ramp shown on Sheets C-6 and C-9 has no matching item in the published schedule. The detail on Sheet C-9 calls a 15-foot-wide ramp at 6 percent grade surfaced with 6-inch aggregate base course (H-20 load per the C-6 notes); the nearest published item, 2.47, is an access ramp of 4-inch asphaltic concrete over 6-inch aggregate base at $32.00 per square yard, which is a different section. No amount is added for the ramp: it is flagged here and referred for alternate pricing under Section 12.04 of the schedule, and the reconciliation carries it as an unresolved item for the bond conversation.

The Type G curb inlets are the second such item. Sheet C-8 note 4 specifies curb inlets Type G with an 8-foot opening per Clark County Standard Drawing SD-305, while the published schedule offers Type A, Type B and combination inlets only. There is nothing in the schedule that matches a Type G inlet, so item 2.30 is flagged and the amount already carried, $58,200.00, remains in the Section 2 subtotal for 12 Type A curb inlets at $4,850.00 each. The same treatment applies to item 2.34, which stands at $7,800.00.

Items for which the plans give no dimension or stationing — 2.39 junction structure, 2.40 headwalls, 2.42 riprap, 2.44 basin excavation, 2.45 outlet structure, 2.46 basin fencing and 2.99 field connections — keep the quantities already carried. The three pipe reaches were re-derived from the stationing on Sheet C-8 and the reach schedule in DSR_Ph3_StormDrain_HGL and are unchanged: 36-inch RCP 870 LF (869.58 LF computed), 48-inch RCP 687 LF (686.51 LF), 60-inch RCP 1,334 LF (1,333.56 LF, including the 240 LF Line B outfall reach), each rounded to the next whole foot. Detention storage on Sheet C-9 stays at the 6.4 acre-foot Phase 3 allocation.

4.  How the revised bond figure is built

Component

Amount

Note

Section 1 — Earthwork

$504,191.75

unchanged

Section 2 — Storm Drain and Detention

$1,303,717.70

was $1,276,934.00; net $26,783.70

Section 3 — Streets and Paving

$1,037,800.00

unchanged

Section 4 — Wet Utilities

$1,031,100.00

unchanged

Section 5 — Dry Utilities

$517,200.00

unchanged

Section 6 — Landscape and Irrigation

$389,340.00

unchanged

Section 7 — Monumentation and Survey

$40,000.00

unchanged

Section 8 — Traffic Control and Signage

$60,000.00

unchanged

Construction subtotal, Sections 1 through 8

$4,883,349.45

was $4,856,565.75

Section 9 — agreed lump sums (mobilization, bonds, insurance $350,000.00; contingency $150,000.00)

$500,000.00

unchanged, Addendum No. 1 items 11.05 and 11.06

Unrounded total

$5,383,349.45

was $5,356,565.75

Rounded bond figure per Section 12.02

$5,384,000

was $5,357,000

The Section 9 allowances are the lump sums agreed with the Department under Addendum No. 1 and are not recalculated as percentages; item 11.02, design, survey and construction inspection, is owner-carried outside the bonded estimate, which is why the estimate carries an express statement of that treatment as item 11.07 requires. Rounding follows Section 12.02: unit prices to the cent as published, extensions to the dollar, and the bonded amount up to the next $1,000.

5.  Action requested

Post $5,384,000 with the surety broker as the Phase 3 performance bond amount and hold the estimate until the corrected Section 2 is issued with this memorandum. The two flagged items — the basin maintenance access ramp and the Type G curb inlets — need a decision before the figure is final: either the Department accepts them under an alternate unit price under Section 12.04, or the ramp stays outside the bonded amount as shown here. Until that decision arrives, $5,384,000 is the number to carry, and no part of the $26,783.70 reconciliation moves back into the subtotal.

Prepared by Max Braddock, Project Manager  ·  Sundial Engineering & Survey  ·  10/31/2026

Sealed: Jonathan R. Kessler, PE — Nevada P.E. License No. 14203  ·  NV COA #C-3874
```
