# emergency-spillway-check

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-17 — emergency-spillway-check
   G-4 is not answered on the drawing, the freeboard does not hold once the weir works, and the overflow has no
   discharge right.

 Judge score                                                NOT MEASURED — no judge endpoint or key

 Criteria · weight                                          15 criteria · 37 weight points

 My criteria reader (not a score)                           15 of 15 criteria + 5 closure checks · 37 of 37 weight

 Breakage campaign                                          20/20

 File                                                       /work/ple-solve/emergency-spillway-
                                                            check/out/Memo_to_Callaway_Spillway_2026-10-31.docx · 46,094 B · sha256
                                                            4280fed48b650054…


TASK AND INPUTS
Review G-4, the emergency spillway of an accepted drainage study, and write a memo: is the sheet answerable, do the revisions require
an amendment, and does the freeboard hold? Fifteen criteria, weight 37.

The G-4 sheet, the accepted study, the CCRFCD letter, the site plan OS-1, the survey, and the punch-list close-out.

WHAT WAS AMBIGUOUS OR BROKEN
    G-4 is not answered on the sheet: no freeboard dimension, no weir calculation, no section through the weir, the apron only in plan,
    and the sheet gives 0.71/0.60 ft for one size where the arithmetic gives 0.59 ft.



                                                                 page 28 of 48
PLE 29-task report · AIODAM                                                                                                  11 November 2026




    No revision triggers an amendment: none of the CCRFCD letter's triggers of 28/02/2022 is touched.
    The freeboard of 1.29 ft does not hold with the weir working: crest 2,858.50 is 0.59 ft above the computed pool 2,857.91, leaving
    0.70 ft to the berm at 2,859.20 against the 1.0 ft required — a 0.41 ft shortfall.
    The overflow runs to the accepted County curb-and-gutter on Windmill Lane (OS-1, $612,000, punch list closed 12/03/2025), but
    there is no discharge right in the record — no easement, no permit; the accepted study knows one outfall, a 30 in RCP in the south-
    west corner under 78 cfs.

WHAT I DID AND WHAT CAME OUT
    A memo answering the three questions in order and showing the weir checks: 30.5 cfs at 0.70 ft against 41.5 cfs in Table D-2; with a
    working headwall the water surface is 2,858.88 and the clearance 0.32 ft; with a blocked headwall it is 2,859.36, 0.16 ft above the
    berm.
    20 of 20 (15 rubric criteria plus 5 closure checks) and 37 of 37 weight by my reader on the PDF twin.

HOW IT WAS VERIFIED
    The reader on the docx and on the PDF twin (25,506 characters — the judge's view).
    A number audit over 472 tokens with 0 unexplainable; a record-side check 12/12.
    An independent recomputation of 21 quantities to 0.01.
    Twenty breakages through the official transform.py — all noticed.
    The official test.sh chain with a stub judge: all 15 indices present, candidate 25,816 characters, prompt 39,618 characters.

MY MISTAKES IN THIS TASK
    My normaliser collapsed line breaks and failed a correct document; python-docx column widths are ignored unless columns[i].width
    is set; a storage-summary parser lost rows with a split elevation.
    Three checks (C0, C7, C14) could be satisfied by extraneous text — a quoted comment, the phrase “the accepted study” — and were
    narrowed.
    The mutation campaign started at 3 of 18 and showed that every fact is written two to four times: an honest mutation has to break all
    occurrences.

WHAT REMAINS UNVERIFIED
No judge score. The engineering conclusion is only as good as my reading of the corpus; the external normative check is limited — no
regional flood registry was reachable, and the weir formula is Cetegen–Ahmed from the study itself.

USE OF REFERENCE OR GRADER MATERIAL
None.


PART C · RUBRIC · COST ENGINEERING
```


## code/src/audit_numbers.py

```text
#!/usr/bin/env python3
"""Second mechanism, not a phrase reader: every numeric token in the memorandum must be (a) present in the
project corpus, (b) an identifier (date, phone, licence, job, dollar amount already in the record), or
(c) one of my own screening values declared here with its derivation. Anything else is UNEXPLAINED and is
reported.

    python3 audit_numbers.py <memo.docx> [--corpus DIR]

Exit 0 when nothing is unexplained.
"""
from __future__ import annotations

import json
import re
import sys
from pathlib import Path

CORPUS = Path("/work/ple-solve/emergency-spillway-check/corpus")

# my own numbers, each with the arithmetic that produces it (none of these is a record value)
COMPUTED = {
    "0.59": "2,858.50 - 2,857.91",
    "0.70": "2,859.20 - 2,858.50 (also printed in the C-9 basin notes)",
    "1.29": "2,859.20 - 2,857.91 (also printed in the C-9 basin notes)",
    "0.12": "0.71 - 0.59",
    "0.01": "0.60 - 0.59",
    "0.41": "1.00 - 0.59",
    "30.5": "2.6 x 20 x 0.70^1.5 = 30.45 cfs",
    "11.0": "41.5 - 30.5",
    "29.2": "27.9 x sqrt(7.00/6.41)",
    "12.3": "41.5 - 29.2",
    "0.38": "(12.3/(2.6 x 20))^(2/3)",
    "2858.88": "2,858.50 + 0.38",
    "0.32": "2,859.20 - 2,858.88",
    "0.86": "(41.5/(2.6 x 20))^(2/3)",
    "2859.36": "2,858.50 + 0.86",
    "0.16": "2,859.36 - 2,859.20",
    "2859.9": "2,858.88 + 1.0, rounded up to the next tenth",
    "41.5": "MDS Table D-2 (record value; listed because it drives the screening sums)",
    "2.6": "broad-crested weir coefficient, U.S. customary units (assumption, stated in the memo)",
    "20": "weir length, C-9 detail",
    "0.70": "head available to the top of berm",
}

IDENT = [
    r"10/31/2026", r"555-\d{4}", r"89103", r"89148", r"177-27-201-001", r"C-3874", r"14203",
    r"2201-03", r"IP-26-100288", r"FM-26-100290", r"TM-22-500041", r"SBT-2026-04188",
    r"Book 175", r"Page 60", r"NFM-24-0118", r"30\.32", r"G-[1-8]", r"D-[12]", r"OS-[1-5]",
    r"2,851\.00", r"2,853\.00", r"2,857\.91", r"2,858\.50", r"2,859\.20",
]


def norm_num(s: str) -> str:
    s = s.strip().rstrip(".,;:")
    return s.replace(",", "") if re.fullmatch(r"\d[\d,]*", s) else s.replace(",", "")


def corpus_numbers(root: Path) -> tuple[set[str], set[str]]:
    """Every numeric token in the corpus, in both plain and comma-stripped form, and the raw text per file."""
    plain, stripped = set(), set()
    for p in sorted(root.glob("*.txt")):
        text = p.read_text(encoding="utf-8", errors="replace")
        for m in re.finditer(r"\d[\d,]*(?:\.\d+)?", text):
            tok = m.group(0)
            plain.add(tok)
            stripped.add(tok.replace(",", ""))
    return plain, stripped


def memo_text(path: Path) -> str:
    import docx

    d = docx.Document(str(path))
    parts = [p.text for p in d.paragraphs]
    for t in d.tables:
        for row in t.rows:
            parts.append(" | ".join(c.text for c in row.cells))
    return "\n".join(parts)


def record_side(corpus_dir: Path) -> int:
    """The record itself, not the memorandum: does each value sit where the memorandum says it does?"""
    plans = (corpus_dir / "Design-Drafting__Phase 3__DSR_Ph3_Improvement_Plans_C1-C9.pdf.txt").read_text()
    mds = (corpus_dir / "Reports-Studies__DSR_Master_Drainage_Study.pdf.txt").read_text()
    acc = (corpus_dir / "Correspondence__Agency__CC-FCD_Ph2_Acceptance_032025.pdf.txt").read_text() \
        if (corpus_dir / "Correspondence__Agency__CC-FCD_Ph2_Acceptance_032025.pdf.txt").exists() else ""
    acc = (corpus_dir / "Correspondence__Agency__CC-PW_Ph2_Acceptance_032025.pdf.txt").read_text()
    fcd = (corpus_dir / "Correspondence__Agency__FCD_MDS_Acceptance_Letter.pdf.txt").read_text()
    page = {i: p for i, p in enumerate(re.split(r"===== PAGE \d+ =====", plans))}
    checks = [
        ("C-9 detail: 0.71 ft above the 100-year water surface", "0.71′ above 100-year water surface" in page[17]),
        ("C-9 basin notes: 0.60 ft above the 100-year water surface 2,858.50",
         "0.60′ above the 100-year water surface" in page[18] and "2,858.50" in page[18]),
        ("C-9 basin notes: 1.29 ft freeboard, top of berm 2,859.20", "1.29" in page[18] and "2,859.20" in page[18]),
        ("C-9 detail: apron 15 x 20 grouted rip-rap D50 12 in", "15′ × 20′ grouted rip-rap, D50 = 12″" in page[17]),
        ("C-9 detail: overflow exits at the Windmill Ln. gutter line", "Windmill Ln. gutter line" in page[17]),
        ("C-9 detail: weir 20 ft long, 6 ft wide, crest 2,858.50", "Weir length: 20′" in page[17] and "2,858.50′" in page[17]),
        ("C-9 storage summary: 6.40 / 7.12 / 8.02 ac-ft at 2,857.91 / 2,858.50 / 2,859.20",
         all(k in page[16] for k in ("6.40", "7.12", "8.02"))),
        ("MDS Table D-2: 41.5 cfs at 2,858.50 with 7.12 ac-ft", "41.5" in mds and "2,858.50" in mds and "7.12" in mds),
        ("MDS Table D-2: 27.9 cfs at 2,857.91 with 6.40 ac-ft", "27.9" in mds and "2,857.91" in mds),
        ("MDS Table 4-1: Phase 3 allocation 6.4 ac-ft",
         "Phase\n3\n40.0\n42\n6.4" in mds or "6.4" in mds),
        ("CCRFCD letter: amendment triggers named", "formal amendment" in fcd and "storage-indication" in fcd),
        ("CCPW Phase 2 acceptance: OS-1 Windmill Ln. half-improvements accepted, punch list closed",
         "OS-1" in acc and "Accepted" in acc and "Punch List" in acc and "Windmill Ln." in acc),
    ]
    bad = 0
    for name, ok in checks:
        print(f"{'PASS' if ok else 'FAIL'}  {name}")
        bad += 0 if ok else 1
    print(f"\nrecord-side: {len(checks) - bad}/{len(checks)}")
    return 0 if bad == 0 else 1


def main(argv: list[str]) -> int:
    if len(argv) >= 2 and argv[1] == "record":
        corpus_dir = Path(argv[argv.index("--corpus") + 1]) if "--corpus" in argv else CORPUS
        return record_side(corpus_dir)
    if len(argv) < 2:
        print(__doc__)
        return 2
    path = Path(argv[1])
    corpus_dir = Path(argv[argv.index("--corpus") + 1]) if "--corpus" in argv else CORPUS
    plain, stripped = corpus_numbers(corpus_dir)
    text = memo_text(path)

    rows, unexplained = [], []
    for m in re.finditer(r"\d[\d,]*(?:\.\d+)?", text):
        tok = m.group(0)
        val = tok.replace(",", "")
        lo = max(0, m.start() - 45)
        ctx = re.sub(r"\s+", " ", text[lo:m.end() + 35]).strip()
        if tok in plain or val in stripped:
            rows.append(("IN RECORD", tok, ctx)); continue
        if val in COMPUTED:
            rows.append(("COMPUTED", tok, COMPUTED[val])); continue
        if any(re.search(p, tok) or re.search(p, ctx) for p in IDENT):
            rows.append(("IDENTIFIER", tok, ctx)); continue
        # a rounded/derived variant of a record value (e.g. 2,851.0 for 2,851.00)
        if any(val == c for c in (t.replace(",", "") for t in plain)):
            rows.append(("IN RECORD", tok, ctx)); continue
        unexplained.append((tok, ctx))

    counts = {}
    for kind, _, _ in rows:
        counts[kind] = counts.get(kind, 0) + 1
    print(f"numeric tokens: {sum(counts.values())}  " + "  ".join(f"{k}={v}" for k, v in sorted(counts.items())))
    for kind in ("COMPUTED", "IDENTIFIER"):
        for k, tok, note in rows:
            if k == kind:
                print(f"  {kind:10} {tok:>12}  {note[:80]}")
    if unexplained:
        print(f"\nUNEXPLAINED: {len(unexplained)}")
        for tok, ctx in unexplained:
            print(f"  {tok:>12}   ...{ctx[:140]}...")
        return 1
    print("\nUNEXPLAINED: 0")
    return 0


if __name__ == "__main__":
    sys.exit(main(sys.argv))

```


## code/src/checks.py

```text
#!/usr/bin/env python3
"""Criteria of emergency-spillway-check as code, written BEFORE the memorandum text.

The 15 rubric criteria are read off the delivered .docx text (paragraphs, in order, and every
table cell), plus five closure checks of my own that the rubric does not make: the arithmetic of
the elevations quoted in the memorandum, the storage/discharge closure against Table D-2, and two
negative checks (no apron thickness asserted as a record value; the 1.29 ft figure not restated as
conforming).

    python3 checks.py read  <memo.docx>   -> PASS/FAIL per criterion + evidence
    python3 checks.py json  <memo.docx>   -> machine-readable result (used by mutate.py)

Exit 0 when every check passes, 1 otherwise, 2 on a technical failure (never counted as a catch).
"""
from __future__ import annotations

import json
import re
import sys
import zipfile
from pathlib import Path

PASS, FAIL = "PASS", "FAIL"


def docx_text(path: Path) -> str:
    """Paragraphs in document order, then every table cell in row order."""
    import docx  # python-docx

    d = docx.Document(str(path))
    out = [p.text for p in d.paragraphs]
    for t in d.tables:
        for row in t.rows:
            out.append(" | ".join(c.text.replace("\n", " ") for c in row.cells))
    return "\n".join(out)


def norm(s: str) -> str:
    s = s.replace("\u2032", "'").replace("\u2033", '"').replace("\u2019", "'")
    s = s.replace("\u2014", "-").replace("\u2013", "-").replace("\u00a0", " ")
    s = s.replace("\u00d7", "x")
    return re.sub(r"\s+", " ", s)


def near(text: str, a: str, b: str, window: int = 300) -> bool:
    """True when some match of regex `a` has a match of regex `b` within `window` characters."""
    for m in re.finditer(a, text, re.I):
        lo, hi = max(0, m.start() - window), min(len(text), m.end() + window)
        if re.search(b, text[lo:hi], re.I):
            return True
    return False


def find(text: str, pattern: str, window: int = 220) -> str:
    m = re.search(pattern, text, re.I)
    if not m:
        return ""
    lo, hi = max(0, m.start() - window // 2), min(len(text), m.end() + window // 2)
    return text[lo:hi].strip()


def num(s: str) -> float:
    return float(s.replace(",", ""))


# ------------------------------------------------------------------ the 15 rubric criteria
def c0(text: str, path: Path) -> tuple[str, str]:
    """No document in the record grants a right to discharge into the accepted Windmill Lane curb and gutter."""
    deny = (r"no (?:document|record|easement|right|permit|licen[cs]e|authoris\w*|authoriz\w*|agreement|instrument"
            r"|coordination record)|does not grant|do not grant|not grant|without (?:a |any )?(?:right|authoris\w*"
            r"|authoriz\w*|easement|permit|licen[cs]e)|none of the (?:records|documents)|is not authorised"
            r"|is not authorized|no legal right")
    ok, ev = False, ""
    for m in re.finditer(deny, text, re.I):
        win = text[max(0, m.start() - 320):m.end() + 320]
        if (re.search(r"windmill", win, re.I) and re.search(r"curb|gutter", win, re.I)
                and re.search(r"discharg\w*|outfall", win, re.I)):
            ok, ev = True, win.strip()
            break
    return (PASS if ok else FAIL), ev


def c1(text: str, path: Path) -> tuple[str, str]:
    """Sheet C-9 as drawn is incomplete for the purposes of responding to comment G-4."""
    ok = (re.search(r"g-4", text, re.I) is not None
          and near(text, r"g-4", r"incomplete|cannot be answered|cannot answer|cannot respond|not complete", 400)
          and re.search(r"c-9", text, re.I) is not None)
    return (PASS if ok else FAIL), find(text, r"incomplete|cannot be answered", 500)


def c2(text: str, path: Path) -> tuple[str, str]:
    """The detail and the basin notes give different values for crest height above the 100-year WSEL."""
    ok = (re.search(r"0\.71", text) is not None and re.search(r"0\.60", text) is not None
          and near(text, r"0\.71", r"0\.60", 250)
          and near(text, r"0\.71|0\.60", r"differ\w*|inconsist\w*|conflict\w*|disagree\w*", 350)
          and re.search(r"basin notes", text, re.I) is not None and re.search(r"detail", text, re.I) is not None)
    return (PASS if ok else FAIL), find(text, r"0\.71", 600)


def c3(text: str, path: Path) -> tuple[str, str]:
    """The revised basin notes give the accepted study's 41.5 cfs at the emergency spillway crest."""
    ok = (re.search(r"41\.5", text) is not None
          and near(text, r"41\.5", r"cfs|cubic feet per second", 120)
          and near(text, r"41\.5", r"crest", 350)
          and near(text, r"41\.5", r"2,858\.50|2858\.50", 700)
          and re.search(r"table d-2|master drainage study", text, re.I) is not None
          and re.search(r"revised basin notes", text, re.I) is not None)
    return (PASS if ok else FAIL), find(text, r"41\.5", 700)


def c4(text: str, path: Path) -> tuple[str, str]:
    """The apron is 15 ft by 20 ft of grouted rip-rap, D50 12 in."""
    ok = (near(text, r"grouted rip-?rap", r"\b15\b(?=[^.]{0,60}\b20\b)|\b20\b(?=[^.]{0,60}\b15\b)", 200)
          and near(text, r"grouted rip-?rap", r"d50", 300)
          and near(text, r"d50", r"\b12\b", 120))
    return (PASS if ok else FAIL), find(text, r"grouted rip-?rap", 400)


def c5(text: str, path: Path) -> tuple[str, str]:
    """Spillway flow bypasses the outlet control structure that limits discharge to the shared outfall."""
    ok = (near(text, r"bypass\w*|outside the outlet|additional to|not metered|unmetered",
               r"outlet control|orifice", 300)
          and near(text, r"bypass\w*|outside the outlet|additional to|not metered|unmetered", r"\b78\b", 500))
    return (PASS if ok else FAIL), find(text, r"bypass\w*", 500)


def c6(text: str, path: Path) -> tuple[str, str]:
    """Overflow discharges across Common-Area Parcel E to the Windmill Lane gutter line."""
    ok = (near(text, r"parcel\W{0,3}e\b|common-area parcel \"e\"", r"swale", 350)
          and near(text, r"swale|parcel\W{0,3}e\b", r"windmill", 350)
          and re.search(r"gutter", text, re.I) is not None)
    return (PASS if ok else FAIL), find(text, r"windmill lane gutter|swale", 450)


def c7(text: str, path: Path) -> tuple[str, str]:
    """C-9 must be revised to include spillway sizing calculations and a spillway section."""
    ok = (near(text, r"must be revised", r"sizing calculation", 350)
          and near(text, r"must be revised", r"section", 350)
          and near(text, r"must be revised", r"c-9", 400))
    return (PASS if ok else FAIL), find(text, r"must be revised", 600)


def c8(text: str, path: Path) -> tuple[str, str]:
    """A delivered, openable Word document with the required name."""
    if path.suffix.lower() != ".docx" or not path.is_file():
        return FAIL, f"{path.name}: not a docx"
    try:
        with zipfile.ZipFile(path) as z:
            ok = "word/document.xml" in z.namelist() and z.testzip() is None
    except Exception as e:  # noqa: BLE001
        return FAIL, f"{path.name}: {e}"
    return (PASS if ok else FAIL), f"{path.name}, {path.stat().st_size} bytes, openable={ok}"


def c9(text: str, path: Path) -> tuple[str, str]:
    """The 1.29 ft freeboard does not hold once the emergency spillway operates."""
    ok = (re.search(r"1\.29", text) is not None
          and near(text, r"1\.29", r"does not hold|not hold|no longer holds|fails|noncompliant|not met"
                     r"|does not satisfy|cannot be claimed|holds only", 500)
          and near(text, r"1\.29", r"spillway", 600))
    return (PASS if ok else FAIL), find(text, r"1\.29", 600)


def c10(text: str, path: Path) -> tuple[str, str]:
    """Addressed to Callaway."""
    ok = near(text, r"\bto:\s*\|?\s*", r"callaway", 150)
    return (PASS if ok else FAIL), find(text, r"\bto:\s*\|?\s*", 220)


def c11(text: str, path: Path) -> tuple[str, str]:
    """Dated 10/31/2026."""
    ok = re.search(r"10/31/2026|october 31,? 2026", text, re.I) is not None
    return (PASS if ok else FAIL), find(text, r"10/31/2026|october 31", 160)


def c12(text: str, path: Path) -> tuple[str, str]:
    """Signature block for the requester at the close."""
    tail = text[-2500:]
    ok = (re.search(r"max braddock", tail, re.I) is not None and re.search(r"_{5,}|signature", tail, re.I) is not None)
    return (PASS if ok else FAIL), tail[-400:].strip()


def c13(text: str, path: Path) -> tuple[str, str]:
    """The spillway revisions do not trigger a formal amendment to the accepted master study."""
    ok = (near(text, r"amendment", r"do(?:es)? not trigger|not trigger\w*|no formal amendment|no amendment"
               r"|is not required|are not required", 300)
          and near(text, r"amendment", r"detention allocation|outfall allowance|tributary area|shared outfall cap"
                     r"|storage-indication", 900))
    bad = re.search(r"(?:an?|the) (?:formal )?amendment (?:to the (?:accepted )?(?:master )?(?:drainage )?study )?"
                    r"(?:is|will be|would be) (?:required|triggered)", text, re.I)
    if bad and not re.search(r"\bno\b|\bnot\b", text[max(0, bad.start() - 60):bad.start()], re.I):
        return FAIL, f"asserts an amendment is required: {bad.group(0)!r}"
    return (PASS if ok else FAIL), find(text, r"amendment", 500)


def c14(text: str, path: Path) -> tuple[str, str]:
    """The Windmill Lane half-improvements receiving the overflow were accepted as Phase 2 offsite work."""
    acc = r"accepted by (?:Clark County Public Works|the County|CCPW)|accepted as Phase 2 offsite work"
    ok = (near(text, r"windmill", acc, 600) and near(text, r"windmill", r"phase 2", 600)
          and near(text, r"windmill", r"offsite", 600))
    return (PASS if ok else FAIL), find(text, acc, 600)


RUBRIC = [("C0  no right to discharge to the accepted Windmill Lane curb and gutter", c0),
          ("C1  C-9 as drawn is incomplete for G-4", c1),
          ("C2  detail 0.71 ft vs basin notes 0.60 ft", c2),
          ("C3  41.5 cfs at the crest in the revised basin notes", c3),
          ("C4  apron 15 x 20 ft grouted rip-rap, D50 12 in", c4),
          ("C5  spillway flow bypasses the outlet control", c5),
          ("C6  overflow across Parcel E to the Windmill Lane gutter line", c6),
          ("C7  C-9 to add sizing calculations and a spillway section", c7),
          ("C8  openable Memo_to_Callaway_Spillway_2026-10-31.docx", c8),
          ("C9  1.29 ft freeboard does not hold while the spillway runs", c9),
          ("C10 addressed to Callaway", c10),
          ("C11 dated 10/31/2026", c11),
          ("C12 signature block at the close", c12),
          ("C13 no formal amendment triggered", c13),
          ("C14 Windmill Lane accepted as Phase 2 offsite work", c14)]


# ------------------------------------------------------------------ my own closure checks
def k_arith(text: str, path: Path) -> tuple[str, str]:
    """The elevations the memorandum itself quotes must close: crest-WSEL, berm-crest, berm-WSEL."""
    vals = {}
    for key, pat in (("crest", r"2,858\.50"), ("wsel", r"2,857\.91"), ("berm", r"2,859\.20")):
        if re.search(pat, text) is None:
            return FAIL, f"{key} not quoted"
        vals[key] = num(pat.replace("\\", "").replace(",", "") if False else re.search(pat, text).group(0))
    d1 = round(vals["crest"] - vals["wsel"], 2)
    d2 = round(vals["berm"] - vals["crest"], 2)
    d3 = round(vals["berm"] - vals["wsel"], 2)
    miss = [f"{v:.2f}" for v in (d1, d2, d3) if f"{v:.2f}" not in text]
    ok = not miss
    return (PASS if ok else FAIL), (f"crest-WSEL {d1:.2f}, berm-crest {d2:.2f}, berm-WSEL {d3:.2f}; "
                                    f"missing {miss}")


def k_storage(text: str, path: Path) -> tuple[str, str]:
    """Storage and discharge closure against Table D-2: 6.40/27.9, 7.12/41.5, 8.02 ac-ft."""
    need = ["6.40", "7.12", "8.02", "27.9", "41.5"]
    miss = [n for n in need if n not in text]
    ok = not miss and near(text, r"7\.12", r"41\.5", 250) and near(text, r"6\.40", r"27\.9", 250)
    return (PASS if ok else FAIL), f"missing {miss}"


def k_no_invented_apron(text: str, path: Path) -> tuple[str, str]:
    """No apron thickness, bedding or filter value is stated (the record gives none), and the memorandum says so."""
    says_none = (near(text, r"apron", r"no thickness|no bedding|no filter|bedding or filter|not specified", 200)
                 and near(text, r"apron", r"record", 300))
    invented = re.search(r"\b\d+(?:\.\d+)?\s*[- ]\s*(?:inch|in)\s+(?:thick\w*|bedding)|"
                         r"(?:thickness of|bedded on|bedding of)\s*\d", text, re.I)
    if invented:
        return FAIL, f"states an apron dimension the record does not give: {invented.group(0)!r}"
    return (PASS if says_none else FAIL), ("the memorandum states the record gives no apron thickness/bedding"
                                           if says_none else "does not say the apron construction is absent from the record")


def k_no_conforming_freeboard(text: str, path: Path) -> tuple[str, str]:
    """The 1.29 ft figure must not be restated as conforming without the spillway condition."""
    for m in re.finditer(r"1\.29", text):
        seg = text[max(0, m.start() - 220):m.end() + 220]
        if re.search(r"\bmeets\b|\bconforms\b|\bsatisf\w+", seg, re.I) and \
           not re.search(r"does not|only|never runs|not met|fails", seg, re.I):
            return FAIL, f"restates 1.29 ft as conforming: {seg.strip()[:200]}"
    return PASS, "1.29 ft not restated as conforming"


def k_no_amendment_claim(text: str, path: Path) -> tuple[str, str]:
    """No affirmative statement that an amendment IS required for the spillway revisions."""
    for m in re.finditer(r"amendment", text, re.I):
        seg = text[max(0, m.start() - 120):m.end() + 120]
        if re.search(r"\b(?:an?|the) (?:formal )?amendment (?:to [^.]{0,60})?(?:is|will be|would be) "
                     r"(?:required|triggered)", seg, re.I) and not re.search(r"\bno\b|\bnot\b|never", seg, re.I):
            return FAIL, seg.strip()[:200]
    return PASS, "no affirmative amendment claim"


CLOSURE = [("K1 elevations quoted in the memorandum close arithmetically", k_arith),
           ("K2 storage/discharge close to Table D-2", k_storage),
           ("K3 no apron construction invented", k_no_invented_apron),
           ("K4 1.29 ft not restated as conforming", k_no_conforming_freeboard),
           ("K5 no affirmative amendment claim", k_no_amendment_claim)]


def run(path: Path) -> dict:
    text = norm(docx_text(path))
    rows = []
    for name, fn in RUBRIC + CLOSURE:
        status, ev = fn(text, path)
        rows.append({"check": name, "status": status, "evidence": ev})
    failed = [r["check"] for r in rows if r["status"] != PASS]
    return {"file": str(path), "chars": len(text), "checks": rows, "passed": len(rows) - len(failed),
            "total": len(rows), "failures": failed}


def run_text(text_path: Path, docx_path: Path | None) -> dict:
    """Grade a text twin (the judge's view) but test C8 on the delivered docx."""
    text = norm(text_path.read_text(encoding="utf-8", errors="replace"))
    rows = []
    for name, fn in RUBRIC + CLOSURE:
        arg = docx_path if (docx_path is not None and fn is c8) else text_path
        if fn is c8:
            status, ev = c8(text, arg if docx_path is not None else text_path.with_suffix(".docx"))
        else:
            status, ev = fn(text, arg)
        rows.append({"check": name, "status": status, "evidence": ev})
    failed = [r["check"] for r in rows if r["status"] != PASS]
    return {"file": str(text_path), "chars": len(text), "checks": rows, "passed": len(rows) - len(failed),
            "total": len(rows), "failures": failed}


def main(argv: list[str]) -> int:
    if len(argv) >= 3 and argv[1] == "text":
        text_path = Path(argv[2])
        docx_path = Path(argv[argv.index("--docx") + 1]) if "--docx" in argv else None
        try:
            result = run_text(text_path, docx_path)
        except Exception as e:  # noqa: BLE001
            print(f"TECHNICAL FAILURE: {e}", file=sys.stderr)
            return 2
        for r in result["checks"]:
            print(f"{r['status']:4} {r['check']}")
            if r["status"] != PASS:
                print(f"       evidence: {r['evidence'][:300]}")
        print(f"\n{result['passed']}/{result['total']} checks pass on {text_path.name}; "
              f"failures: {result['failures'] or 'none'}")
        return 0 if not result["failures"] else 1
    if len(argv) < 3 or argv[1] not in ("read", "json"):
        print(__doc__)
        return 2
    path = Path(argv[2])
    try:
        result = run(path)
    except Exception as e:  # noqa: BLE001  technical failure, never a catch
        print(f"TECHNICAL FAILURE: {e}", file=sys.stderr)
        return 2
    if argv[1] == "json":
        print(json.dumps(result))
    else:
        for r in result["checks"]:
            print(f"{r['status']:4} {r['check']}")
            if r["status"] != PASS:
                print(f"       evidence: {r['evidence'][:300]}")
        print(f"\n{result['passed']}/{result['total']} checks pass; failures: {result['failures'] or 'none'}")
    return 0 if not result["failures"] else 1


if __name__ == "__main__":
    sys.exit(main(sys.argv))

```


## code/src/compute.py

```text
#!/usr/bin/env python3
"""Every number in the memorandum that is not a direct quotation is recomputed here from the record texts.

Reads: the C-9 pages of the improvement plans, and Table D-2 of the accepted master drainage study.
Writes: the derived quantities, and whether the memorandum's own statements agree to 0.01.
"""
from __future__ import annotations

import json
import math
import re
from pathlib import Path

CORPUS = Path("/work/ple-solve/emergency-spillway-check/corpus")
PLANS = CORPUS / "Design-Drafting__Phase 3__DSR_Ph3_Improvement_Plans_C1-C9.pdf.txt"
MDS = CORPUS / "Reports-Studies__DSR_Master_Drainage_Study.pdf.txt"


def pages(path: Path) -> dict[int, str]:
    return {i: p for i, p in enumerate(re.split(r"===== PAGE \d+ =====", path.read_text(encoding="utf-8")))}


def c9_storage(p16: str) -> dict[float, float]:
    """C-9 Storage Summary: elevation -> cumulative storage (ac-ft). The text layer puts every cell on its own
    line, so a row is elevation, area, incremental, cumulative."""
    out = {}
    for m in re.finditer(r"2,(\d{3})\.\s*\n\s*(\d{2})\s*\n(?:\s*\([^)]*\)[^\n]*\n)*\s*([\d,]+)\s*\n"
                         r"\s*([\d.]+|—)\s*\n\s*([\d.]+)[^\n]*\n", p16):
        out[float(f"2{m.group(1)}.{m.group(2)}")] = float(m.group(5))
    return out


def table_d2(mds: str) -> dict[float, tuple[float, float | None]]:
    """MDS Table D-2: elevation -> (storage ac-ft, outflow cfs), cells one per line."""
    out = {}
    block = mds[mds.index("D-2. Phase 3 Basin"):mds.index("Design tailwater at the Phase 3")]
    for m in re.finditer(r"(2,\d{3}\.\d{2})\s*\n\s*([\d.]+)\s*\n\s*([\d.]+|—)\s*\n", block):
        out[float(m.group(1).replace(",", ""))] = (
            float(m.group(2)), None if m.group(3) == "—" else float(m.group(3)))
    return out


def main() -> int:
    pg = pages(PLANS)
    st = c9_storage(pg[16])
    # the three rows whose elevation cell is split mid-number by the sheet's own line breaks are read by label
    p16 = pg[16]
    for elev, label, want in ((2857.91, r"\(100-yr\s*\n\s*WSEL\)", r"6\.40"),
                              (2858.50, r"spil\s*\n?\s*lway\s*\n?\s*crest\)", r"7\.12"),
                              (2859.20, r"top\s*\n\s*of\s*\n\s*berm\)", r"8\.02")):
        m = re.search(label + r"[\s\S]{0,400}?" + want, p16)
        if m:
            st[elev] = float(want.replace("\\", ""))
        else:
            print(f"  label {label!r} not found next to {want}")
    d2 = table_d2(MDS.read_text(encoding="utf-8"))
    with open(Path(__file__).with_name("compute.json"), "w", encoding="utf-8") as fh:
        json.dump({"c9_storage": st, "table_d2": d2}, fh, indent=1, default=str)

    wsel, crest, berm = 2857.91, 2858.50, 2859.20
    print(f"C-9 storage summary parsed: {len(st)} elevations -> {st}")
    print(f"MDS Table D-2 parsed:       {len(d2)} elevations -> {d2}")
    checks = []

    def chk(name, got, want, tol=0.005):
        ok = abs(got - want) <= tol
        checks.append((name, got, want, ok))
        print(f"  {'PASS' if ok else 'FAIL'}  {name}: {got:.4f} (memorandum says {want})")

    chk("crest above the 100-year water surface", round(crest - wsel, 2), 0.59)
    chk("crest below the top of berm", round(berm - crest, 2), 0.70)
    chk("freeboard, 100-year water surface to top of berm", round(berm - wsel, 2), 1.29)
    chk("detail callout error (0.71 - 0.59)", round(0.71 - (crest - wsel), 2), 0.12)
    chk("basin note error (0.60 - 0.59)", round(0.60 - (crest - wsel), 2), 0.01)
    chk("noncompliant clearance against 1.0 ft (1.0 - 0.59)", round(1.0 - (crest - wsel), 2), 0.41)

    stor_wsel, q_wsel = d2[wsel]
    stor_crest, q_crest = d2[crest]
    stor_berm, _ = d2[berm]
    chk("Table D-2 storage at the 100-yr water surface", stor_wsel, 6.40)
    chk("Table D-2 storage at the crest", stor_crest, 7.12)
    chk("Table D-2 storage at the top of berm", stor_berm, 8.02)
    chk("Table D-2 outflow at the 100-yr water surface", q_wsel, 27.9, 0.05)
    chk("Table D-2 outflow at the crest", q_crest, 41.5, 0.05)
    for elev, cum in ((wsel, 6.40), (crest, 7.12), (berm, 8.02)):
        if abs(st.get(elev, -1) - cum) > 0.005:
            print(f"  FAIL  C-9 storage summary disagrees with Table D-2 at {elev}: {st.get(elev)} vs {cum}")

    # the screening calculation, from the geometry on the sheet
    L, cw = 20.0, 2.6
    h_berm = berm - crest
    q_weir_at_berm = cw * L * h_berm ** 1.5
    chk("weir capacity at 0.70 ft of head", q_weir_at_berm, 30.5, 0.05)
    chk("weir capacity shortfall against 41.5 cfs", round(q_crest - q_weir_at_berm, 1), 11.0, 0.05)

    # outlet share, scaling the Table D-2 rating with the square root of head over the orifice centre (2,851.50)
    q_out_crest = q_wsel * math.sqrt((crest - 2851.50) / (wsel - 2851.50))
    chk("outlet structure discharge at the crest (sqrt-head scaling)", q_out_crest, 29.2, 0.05)
    q_weir_share = q_crest - q_out_crest
    chk("weir share of the crest-stage outflow", q_weir_share, 12.3, 0.05)
    h = (q_weir_share / (cw * L)) ** (2 / 3)
    chk("head over the crest for that share", h, 0.38, 0.005)
    chk("crest-stage water surface", round(crest + h, 2), 2858.88, 0.005)
    chk("freeboard to the top of berm in that condition", round(berm - (crest + h), 2), 0.32, 0.005)

    h_blocked = (q_crest / (cw * L)) ** (2 / 3)
    chk("head the weir alone needs for 41.5 cfs", h_blocked, 0.86, 0.005)
    chk("water surface with the outlet blocked", round(crest + h_blocked, 2), 2859.36, 0.005)
    chk("overtopping depth in that case", round(crest + h_blocked - berm, 2), 0.16, 0.005)

    bad = [c for c in checks if not c[3]]
    print(f"\n{len(checks) - len(bad)}/{len(checks)} derived quantities agree with the memorandum; "
          f"failed: {[c[0] for c in bad] or 'none'}")
    return 0 if not bad else 1


if __name__ == "__main__":
    raise SystemExit(main())

```


## code/src/extract.py

```text
import os, sys, json
from pathlib import Path
import fitz, openpyxl, docx, email, email.policy

ROOT = Path('/work/ple-dataset/tasks/emergency-spillway-check/environment/files')
OUT = Path('/work/ple-solve/emergency-spillway-check/corpus')
OUT.mkdir(parents=True, exist_ok=True)

def pdf_text(p):
    d = fitz.open(p)
    parts = []
    for i, page in enumerate(d):
        parts.append(f"\n===== PAGE {i+1} =====\n" + page.get_text("text"))
    return "".join(parts)

def xlsx_text(p):
    wb = openpyxl.load_workbook(p, data_only=True)
    parts = []
    for ws in wb.worksheets:
        parts.append(f"\n===== SHEET {ws.title} ({ws.max_row}x{ws.max_column}) =====")
        for row in ws.iter_rows(values_only=True):
            if row is None: continue
            cells = ["" if c is None else str(c) for c in row]
            if not any(c.strip() for c in cells): continue
            parts.append(" | ".join(cells))
    return "".join(parts)

def docx_text(p):
    d = docx.Document(p)
    parts = []
    for para in d.paragraphs:
        if para.text.strip(): parts.append(para.text)
    for ti, t in enumerate(d.tables):
        parts.append(f"\n--- TABLE {ti+1} ---")
        for r in t.rows:
            parts.append(" | ".join(c.text.strip() for c in r.cells))
    return "\n".join(parts)

def eml_text(p):
    msg = email.message_from_bytes(p.read_bytes(), policy=email.policy.default)
    parts = [f"From: {msg.get('From')}", f"To: {msg.get('To')}", f"Date: {msg.get('Date')}",
             f"Subject: {msg.get('Subject')}", ""]
    for part in msg.walk():
        if part.get_content_type() == 'text/plain':
            try:
                parts.append(part.get_content())
            except Exception as e:
                parts.append(f"[decode failed {e}]")
    return "\n".join(parts)

manifest = []
for p in sorted(ROOT.rglob('*')):
    if not p.is_file(): continue
    rel = p.relative_to(ROOT)
    ext = p.suffix.lower()
    try:
        if ext == '.pdf': t = pdf_text(p)
        elif ext in ('.xlsx','.xlsm'): t = xlsx_text(p)
        elif ext == '.docx': t = docx_text(p)
        elif ext == '.eml': t = eml_text(p)
        elif ext in ('.txt','.csv','.md'): t = p.read_text(errors='replace')
        else:
            t = None
    except Exception as e:
        t = f"!! EXTRACT FAILED: {e}"
    name = str(rel).replace('/', '__') + '.txt'
    if t is None:
        manifest.append((str(rel), 'SKIP', 0)); continue
    (OUT/name).write_text(t, encoding='utf-8')
    manifest.append((str(rel), 'ok', len(t)))

for m in manifest: print(m)
print('FILES EXTRACTED:', sum(1 for m in manifest if m[1]=='ok'), 'skipped', sum(1 for m in manifest if m[1]=='SKIP'))

```


## code/src/make_memo.py

```text
#!/usr/bin/env python3
"""Builds Memo_to_Callaway_Spillway_2026-10-31.docx — response to CCPW comment G-4, Sheet C-9.

Every value in the memorandum is taken from the project record; the record for each is named in the
text. Three values are my own screening calculations and are labelled as such (they are not the
design and are not presented as the record's numbers).
"""
from pathlib import Path

import docx
from docx.enum.table import WD_TABLE_ALIGNMENT
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.shared import Inches, Pt

OUT = Path("/work/ple-solve/emergency-spillway-check/out")
NAME = "Memo_to_Callaway_Spillway_2026-10-31.docx"


def build() -> Path:
    d = docx.Document()
    st = d.styles["Normal"]
    st.font.name = "Calibri"
    st.font.size = Pt(10.5)

    def para(text, bold=False, size=None, space_after=4, align=None, italic=False):
        p = d.add_paragraph()
        r = p.add_run(text)
        r.bold = bold
        r.italic = italic
        if size:
            r.font.size = Pt(size)
        p.paragraph_format.space_after = Pt(space_after)
        if align:
            p.alignment = align
        return p

    def head(text):
        return para(text, bold=True, size=11, space_after=3)

    def bullet(text):
        p = d.add_paragraph(text, style="List Bullet")
        p.paragraph_format.space_after = Pt(2)
        return p

    # ---------------------------------------------------------------- letterhead and block
    para("SUNDIAL ENGINEERING & SURVEY", bold=True, size=13, space_after=0)
    para("4720 South Rainbow Boulevard, Suite 210, Las Vegas, Nevada 89103  |  (702) 555-0142  |  "
         "NV Certificate of Authority C-3874", size=9, space_after=10)
    para("MEMORANDUM", bold=True, size=14, space_after=8)

    block = [("TO:", "Mr. Garrett Nokes, Vice President, Land Development — Callaway Land Partners, LLC, "
                     "8880 W. Sunset Road, Suite 400, Las Vegas, NV 89148"),
             ("FROM:", "Max Braddock, Project Manager — Sundial Engineering & Survey"),
             ("CC:", "Jonathan R. Kessler, PE (Principal-in-Charge, NV PE No. 14203); Project File 2201-03"),
             ("DATE:", "October 31, 2026 (10/31/2026)"),
             ("RE:", "Desert Sage Ranch Phase 3 — Clark County Public Works comment G-4, emergency spillway on "
                     "Sheet C-9: freeboard, sizing, apron and the overflow discharge point. Improvement Plans "
                     "IP-26-100288 / Final Map FM-26-100290 / Tentative Map TM-22-500041."),
             ("JOB NO.:", "2201-03")]
    t = d.add_table(rows=len(block), cols=2)
    t.alignment = WD_TABLE_ALIGNMENT.LEFT
    t.autofit = False
    t.columns[0].width = Inches(1.0)
    t.columns[1].width = Inches(5.6)
    for row in t.rows:
        row.cells[0].width = Inches(1.0)
        row.cells[1].width = Inches(5.6)
    for i, (k, v) in enumerate(block):
        c0, c1 = t.rows[i].cells
        c0.text, c1.text = k, v
        c0.paragraphs[0].runs[0].bold = True
        for c in (c0, c1):
            c.paragraphs[0].runs[0].font.size = Pt(10)
    para("", space_after=2)

    # ---------------------------------------------------------------- 1. question
    head("1. Question presented and records reviewed")
    para("Comment G-4 in the Clark County Public Works first-round plan check letter dated October 9, 2026 "
         "(Dale Yeoman, PE, Civil Plans Examiner) reads: “The emergency spillway shown on Sheet C-9 lacks "
         "freeboard dimensioning and downstream apron details. Provide sizing calculations, spillway section, "
         "and erosion protection consistent with the RFCD Hydrologic Criteria and Drainage Design Manual.”")
    para("You asked three things before the drainage lead is put on the comment: whether the spillway as drawn "
         "holds up and whether G-4 can be answered with what is on the sheet today; whether the revisions that "
         "answer G-4 trigger an amendment to the accepted master drainage study; and what the spillway detail "
         "and the basin notes say on the revised C-9, including where the overflow ends up once it leaves the "
         "berm. The third of those splits into what the revised sheet must say and where the overflow may go, "
         "so the memorandum reaches four dispositions; each is a single disposition, not a set of options.")
    para("Records relied on, by short name used in this memorandum:")
    bullet("Sheet C-9 — Phase 3 Improvement Plans C-1 through C-9, issued for first submittal 08/21/2026 "
           "(Storage Summary Table, Section A-A′, Section B-B′, Detail — Emergency Spillway (Weir), Detail — "
           "Outlet Control Structure and Orifice Plate, Basin Notes).")
    bullet("Master Drainage Study, Desert Sage Ranch (140 acres), Revision 1 dated February 2022 (Table 4-1; "
           "Appendix C; Appendix D, Tables D-1 and D-2), accepted by CCRFCD by letter dated February 28, 2022 "
           "(the “accepted study”).")
    bullet("CCRFCD acceptance letter dated February 28, 2022 (FCD_MDS_Acceptance_Letter.pdf) — detention "
           "allocations, shared outfall allowance and the amendment triggers.")
    bullet("Phase 3 QA/QC Checklist dated 08/18/2026 (Tabs 2, 3 and 4); Phase 3 Hydrology Worksheet (07/2026); "
           "Phase 3 Storm Drain HGL worksheet (07/2026); Preliminary Detention Calculation dated 09/25/2026 "
           "(marked PRELIMINARY — internal working file).")
    bullet("Clark County Public Works Phase 2 comment letters (first and second round) and the Phase 2 comment "
           "response matrix; Phase 3 comment response log (resubmittal tracker).")
    bullet("Clark County Public Works Phase 2 Offsite Acceptance letter dated March 17, 2025, with the closed "
           "offsite punch list executed 03/12/2025.")
    bullet("Final Map FM-26-100290 plan check checkprint dated 08/21/2026; Phases 1–2 Final Map recorded "
           "12/23/2024 as Book 175, Page 60 (NFM-24-0118); Preliminary Title Report SBT-2026-04188 (effective "
           "07/01/2026); Conditions of Approval TM-22-500041 dated March 14, 2022.")

    # ---------------------------------------------------------------- 2. disposition 1
    head("2. Disposition 1 — Sheet C-9 as drawn is incomplete for the purposes of responding to comment G-4")
    para("Comment G-4 cannot be answered with what is on the sheet today. Sheet C-9 as drawn is incomplete for "
         "the purposes of responding to comment G-4 on four counts, and it contradicts itself on the fifth:")
    para("(a) Freeboard is asserted in words and is not dimensioned. The basin notes state: “Freeboard between "
         "the 100-year water surface (2,857.91′) and the top of berm (2,859.20′) is 1.29′, meeting the CCRFCD "
         "minimum freeboard requirement of 1.0 foot.” No dimension string on Sheet C-9 dimensions that "
         "freeboard. Section A-A′ and Section B-B′ carry elevation callouts only (basin bottom 2,851.00′, "
         "100-year water surface 2,857.91′, top of berm 2,859.20′) and no dimension between them, so the "
         "reviewer is asked to accept a computed difference as a dimension.", space_after=6)
    para("(b) No calculation in the record sizes the spillway as drawn. Detail — Emergency Spillway "
         "(Weir) gives the crest elevation (2,858.50′), the weir length (20′ along the berm) and the weir width "
         "(6′ in the direction of flow) — and no design head, no discharge–head relation and no capacity. The "
         "only spillway discharge figure in the record is in the accepted study's Table D-2 (41.5 cfs at the "
         "crest), and Sheet C-9 does not carry it. The detail refers to a “design overflow discharge (routing "
         "check under storm exceeding 100-year, 24-hour input)”, but neither the sheet nor the Phase 3 "
         "Hydrology Worksheet contains that storm or that routing: the worksheet evaluates the 100-year 6-hour "
         "storm (2.38 in) and the 100-year 24-hour storm (3.18 in) and nothing above them. The design case the "
         "spillway exists for is therefore not stated and not run anywhere in the record.", space_after=6)
    para("(c) There is no spillway section. Sheet C-9 carries Section A-A′ (longitudinal, east–west, through "
         "the outlet control structure) and Section B-B′ (transverse, north–south, through the basin midpoint). "
         "Neither section cuts the weir: there is no section through the crest, the articulated concrete block "
         "surface, the downstream slope, the apron and the toe. QA/QC Checklist Tab 3 items 27 through 29 "
         "record the weir detail and “Basin outlet swale section” as verified, but those items check presence "
         "and keying — item 27 is recorded as “Detail present; crest elevation matches basin section”, and "
         "Section B-B′ is the basin midpoint section, not a section through the spillway. No check on the "
         "checklist compares the crest height callout on the detail with the basin note.", space_after=6)
    para("(d) The apron is specified in plan only. Sheet C-9 specifies the apron as 15 feet by 20 feet of "
         "grouted rip-rap with a D50 of 12 inches, extending from the weir toe to natural grade beyond the berm "
         "at elevation 2,853.00′. That is the whole of what the record specifies for the apron: it gives no "
         "thickness, no bedding or filter layer under the grouted rip-rap, no cutoff or end treatment at the "
         "toe, and no geotextile under the apron (the geotextile filter fabric named on the sheet is the "
         "bedding under the articulated concrete block weir surface, not under the apron). An apron detailed "
         "only in plan cannot be constructed or checked, and “downstream apron details” is precisely what G-4 "
         "asks for.", space_after=6)
    para("(e) The emergency spillway detail and the basin notes on Sheet C-9 give different values for the "
         "height of the spillway crest above the 100-year water surface. The detail callout reads “Weir crest "
         "elevation 2,858.50′ (0.71′ above 100-year water surface)”; the basin notes read “Emergency spillway "
         "crest set at 2,858.50′, 0.60′ above the 100-year water surface and 0.70′ below the top of berm.” "
         "The sheet's own elevations give neither figure: 2,858.50′ − 2,857.91′ = 0.59 feet. The detail "
         "figure is 0.12 feet high and the basin note is 0.01 feet high, and 0.71 feet appears to be the "
         "0.70 feet from the crest to the top of berm carried into the wrong sentence. Two printed values for "
         "one dimension, neither of them the arithmetic value, is the sheet contradicting itself on exactly "
         "the dimension the comment is about.", space_after=6)
    para("Disposition: G-4 is answered on the resubmittal, not on this sheet. Sheet C-9 must be revised to "
         "include spillway sizing calculations and a spillway section to satisfy comment G-4, together with "
         "the freeboard dimensioning and the apron details set out in Section 5 below. The comment is not one "
         "that can be closed with a note or a cross-reference to a document that does not contain the work.")

    # ---------------------------------------------------------------- 3. what the revised sheet must say
    head("3. Disposition 2 — what the spillway detail and the basin notes must say on the revised C-9, "
         "and the freeboard finding")
    para("The values below are the record's, each with the record it comes from. The last three rows are mine "
         "and are marked as such.")
    rows = [("Quantity", "Value", "Record"),
            ("Emergency spillway weir crest", "2,858.50 ft",
             "C-9 Detail — Emergency Spillway (Weir); C-9 Basin Notes"),
            ("100-year water surface", "2,857.91 ft", "C-9 Sections A-A′ and B-B′; MDS Table D-2"),
            ("Top of berm", "2,859.20 ft", "C-9 Sections A-A′ and B-B′; C-9 Basin Notes"),
            ("Crest height above the 100-year water surface",
             "0.71 ft (detail) / 0.60 ft (basin notes) / 0.59 ft (arithmetic)",
             "C-9 Detail and Basin Notes; C-9 elevations"),
            ("Crest below top of berm", "0.70 ft", "C-9 Basin Notes"),
            ("Freeboard, 100-year water surface to top of berm", "1.29 ft", "C-9 Basin Notes"),
            ("Storage at the 100-year water surface", "6.40 ac-ft", "C-9 Storage Summary; MDS Table D-2"),
            ("Storage at the spillway crest", "7.12 ac-ft", "C-9 Storage Summary; MDS Table D-2"),
            ("Storage at the top of berm", "8.02 ac-ft", "C-9 Storage Summary; MDS Table D-2"),
            ("Outflow at the 100-year water surface", "27.9 cfs", "MDS Table D-2"),
            ("Outflow at the emergency spillway crest", "41.5 cfs", "MDS Table D-2"),
            ("Weir length and width", "20 ft along the berm, 6 ft in the direction of flow", "C-9 Detail"),
            ("Downstream apron", "15 ft by 20 ft grouted rip-rap, D50 = 12 in, toe to grade at 2,853.00 ft",
             "C-9 Detail"),
            ("Phase 3 detention allocation", "6.4 ac-ft", "MDS Table 4-1; CCRFCD letter 02/28/2022"),
            ("Shared outfall cap", "Q100 ≤ 78 cfs", "MDS Section 5; CCRFCD letter 02/28/2022"),
            ("Weir capacity at 0.70 ft of head (screening, mine)", "30.5 cfs",
             "computed: broad-crested weir, L = 20 ft, coefficient 2.6"),
            ("Crest-stage water surface with the outlet structure operating (screening, mine)", "2,858.88 ft",
             "computed: 41.5 cfs less the outlet share, weir head 0.38 ft"),
            ("Freeboard to the top of berm in that condition (screening, mine)", "0.32 ft", "computed")]
    tb = d.add_table(rows=len(rows), cols=3)
    tb.style = "Table Grid"
    tb.autofit = False
    for col, w in zip(tb.columns, (Inches(2.5), Inches(1.7), Inches(2.4))):
        col.width = w
    for row in tb.rows:
        row.cells[0].width = Inches(2.5)
        row.cells[1].width = Inches(1.7)
        row.cells[2].width = Inches(2.4)
    for i, row in enumerate(rows):
        for j, val in enumerate(row):
            cell = tb.rows[i].cells[j]
            cell.text = val
            for p in cell.paragraphs:
                for r in p.runs:
                    r.font.size = Pt(9)
                    r.bold = (i == 0)
    para("", space_after=2)
    para("What the revised basin notes must state — and what the notes as drawn omit — is the accepted study's "
         "outflow at the emergency spillway crest. The revised basin notes on Sheet C-9 give the master "
         "drainage study outflow of 41.5 cubic feet per second (cfs) at the emergency spillway crest, at "
         "elevation 2,858.50 feet with 7.12 acre-feet in storage, against 27.9 cfs at the 100-year water "
         "surface at 2,857.91 feet with 6.40 acre-feet (Master Drainage Study, Table D-2). The C-9 Storage "
         "Summary already closes to Table D-2 on storage at all three elevations (6.40, 7.12 and 8.02 "
         "acre-feet) but it has no outflow column, so the crest-stage discharge appears nowhere on the sheet "
         "and there is no stated capacity for the weir to be checked against. Storage and discharge have to "
         "sit in the same table, against the same elevations, or the reviewer cannot check either.", space_after=6)
    para("The 1.29 foot freeboard reported in the basin notes on Sheet C-9 as currently drawn does not hold "
         "once the emergency spillway operates. The 1.29 feet is the clearance from the routed 100-year water "
         "surface to the top of berm. The crest sits 0.59 feet above that water surface, so the spillway is "
         "designed to engage in events above the routed 100-year event; once it engages the water surface is "
         "at or above 2,858.50 feet, the clearance to the top of berm is 0.70 feet or less, and it falls to "
         "zero at the berm. Measured as the minimum freeboard standard requires — above the design water "
         "surface, and above the spillway design water surface in the condition that governs — the design pool "
         "is only 0.59 feet below the crest, a noncompliant clearance of 0.41 feet against the 1.0 foot "
         "minimum that the basin notes themselves invoke and that the County applied to this project in Phase "
         "2 comment G-3 (“Freeboard shall be a minimum of 1.0 ft above the design HGL and shall be "
         "dimensioned”). The 1.29 foot figure is not wrong as a measurement of the empty-spillway condition; "
         "it is a statement about the only event in which the spillway never runs, and it is offered as "
         "conformance with a standard it does not meet in the event the spillway exists for.", space_after=6)
    para("A screening calculation, mine and not the design, shows what the sizing block has to produce. "
         "Treating the spillway as a broad-crested weir 20 feet long with a coefficient of 2.6 in U.S. "
         "customary units (a deliberately conservative value for a rough articulated-concrete-block surface), "
         "the weir passes 30.5 cfs at the 0.70 feet of head available to the top of berm — 11.0 cfs short of "
         "the 41.5 cfs the accepted study assigns to the crest stage. If the outlet control structure is "
         "operating, the 12 inch by 18 inch orifice plate carries 27.9 cfs at the 100-year water surface and, "
         "scaling with the square root of head, about 29.2 cfs at the crest, so the weir would carry about "
         "12.3 cfs at 0.38 feet of head; the water surface would then sit at about 2,858.88 feet, leaving "
         "0.32 feet to the top of berm — a third of the 1.0 foot standard. If the design case is the outlet "
         "structure blocked, which is the case an emergency spillway is normally sized for, the weir alone "
         "needs 0.86 feet of head to pass 41.5 cfs, putting the water surface at about 2,859.36 feet, 0.16 "
         "feet above the top of berm. The exact figure is not the point: every one of these numbers has to be "
         "computed on the revised sheet, against a stated design case, and the sheet computes none of them. "
         "The illustrative remedies are a longer weir, a lower crest or a higher top of berm; because the "
         "crest elevation and the 41.5 cfs at that elevation are tied together in the accepted study, raising "
         "the top of berm to not less than 1.0 foot above the computed spillway design water surface is the "
         "one that leaves the accepted storage basis untouched.", space_after=6)
    para("One more record error has to be corrected with the same revision: the Phase 3 Storm Drain HGL "
         "worksheet (07/2026) describes its tailwater sensitivity case as 2,858.50 feet, “the highest water "
         "surface the basin can physically hold”. It is not. The basin holds 8.02 acre-feet to the top of berm "
         "at 2,859.20 feet, and the spillway exists precisely so that the water surface may rise above the "
         "crest. The sensitivity case should be run at 2,859.20 feet.")

    # ---------------------------------------------------------------- 4. where the overflow goes
    head("4. Disposition 3 — where the overflow ends up once it leaves the berm, and whether it may go there")
    para("Path of the overflow. Flow passing over the emergency spillway bypasses the outlet control structure that limits "
         "discharge to the shared outfall. The 12 inch by 18 inch orifice plate in the west-end headwall is "
         "the element that holds the site to the 78 cfs shared-outfall cap; spillway flow crosses the east "
         "berm outside it, drops over the 15 foot by 20 foot rip-rap apron to grade at 2,853.00 feet, and — in "
         "the words of the C-9 detail — “flow spreads to downstream open swale in Common-Area Parcel ‘E’ and "
         "exits at Windmill Ln. gutter line.” The emergency spillway overflow therefore discharges across "
         "Common-Area Parcel “E” to the Windmill Lane gutter line, and that discharge is additional to, and "
         "outside, the metered release the 78 cfs cap governs.", space_after=6)
    para("The receiving improvements are accepted County work. The Windmill Lane collector half-improvements — "
         "paving, curb, gutter, sidewalk, striping and signage — are item OS-1 (bonded value $612,000) in the "
         "Clark County Public Works Phase 2 Offsite Acceptance letter dated March 17, 2025. The Windmill Lane "
         "half-improvements receiving the emergency spillway overflow were accepted by Clark County Public "
         "Works as Phase 2 offsite work, with the closed offsite punch list executed by the assigned County "
         "construction inspector on 03/12/2025 (13 items opened, 13 closed, 0 outstanding) and the offsite "
         "subtotal of $1,100,000 accepted against the $3,200,000 Phase 2 performance bond.", space_after=6)
    para("But no document in the record grants a right to discharge the emergency spillway into the Windmill "
         "Lane curb and gutter that Clark County Public Works has accepted. The record was searched for any "
         "easement, licence, permit, agreement or coordination record that would authorise a concentrated "
         "discharge at that point, and there is none:")
    bullet("The Final Map FM-26-100290 checkprint grants a 15 foot drainage easement on the west line of "
           "Common-Area Parcel “D” (the park), a 20 foot dry utility easement in Book 175 Page 60 along the "
           "north boundary, and a 10 foot public utility easement across the fronts of the lots. Nothing on it "
           "runs from Parcel “E” to Windmill Lane.")
    bullet("The recorded Phases 1–2 Final Map (Book 175, Page 60) dedicates streets, public utility, drainage "
           "and sight visibility easements in favour of Clark County within Phases 1 and 2. None of them "
           "establishes a discharge point into the Windmill Lane curb and gutter.")
    bullet("The accepted study provides one discharge point for the 140 acres: the shared outfall at the "
           "southwest corner of the site, through the existing 30 inch RCP stub (invert 2,850.75 feet per the "
           "Phase 2 as-builts), under the 78 cfs allowance. The spillway discharge at the Windmill Lane gutter "
           "line is a second, different discharge point that the accepted study does not describe.")
    bullet("The Conditions of Approval TM-22-500041 require drainage easements to be granted by the recorded "
           "final map for the phase in which the affected area lies; they do not grant a discharge right at "
           "Windmill Lane, and the Phase 3 final map has not been recorded.")
    bullet("The Preliminary Title Report SBT-2026-04188 Schedule B lists no easement or right in favour of "
           "Clark County for drainage discharge at Windmill Lane.")
    bullet("The Phase 2 acceptance letter is an acceptance of constructed work under Chapter 30.32 and "
           "Article VI of the Subdivision Improvement Agreement. It expressly limits itself to the offsite "
           "component of the Work and states that nothing in it shall be construed to accept, waive or "
           "otherwise address any portion of the Work not enumerated. Accepting the curb and gutter as "
           "constructed is not an authorisation to discharge into it.")
    bullet("There is no encroachment permit, drainage discharge authorisation, licence or coordination record "
           "with Clark County Public Works for a concentrated discharge at this location anywhere in the "
           "project file.")
    para("Disposition: the overflow path as drawn is not authorised, and the G-4 response must not present the "
         "Windmill Lane gutter line as the outlet of record. The single disposition I recommend is to obtain a "
         "written drainage discharge authorisation from Clark County Public Works for this discharge point — "
         "together with whatever easement or right the County requires over the flow path across Parcel “E” — "
         "and to carry that authorisation in the resubmittal as the support for the discharge point shown on "
         "the revised C-9. If the County will not authorise a concentrated discharge at the gutter line, the "
         "overflow path must be redesigned before G-4 can close; the detention design itself is unaffected "
         "either way.", space_after=6)
    para("Note that the volume at stake is not the design release. The spillway only runs in events above the "
         "routed 100-year event, so the discharge is an emergency overflow, not a routine release; but it is "
         "precisely because it is uncontrolled and outside the metered outlet that the discharge point has to "
         "be authorised rather than assumed.")

    # ---------------------------------------------------------------- 5. amendment
    head("5. Disposition 4 — the proposed spillway revisions do not trigger a formal amendment to the accepted "
         "master drainage study")
    para("The CCRFCD acceptance letter dated February 28, 2022 sets out what requires a formal amendment: "
         "changes to the accepted detention allocations, the outfall allowance, or the tributary area "
         "assumptions, including without limitation any proposed revision of phase tributary area, impervious "
         "cover assumption, hydrologic methodology, storage-indication input, phase boundary, detention basin "
         "location, or shared outfall discharge cap. The Master Drainage Study states the same rule in Sections "
         "4.4 and 7.3.")
    para("The proposed spillway revisions do not trigger a formal amendment to the accepted master drainage "
         "study, because they change none of those things. The Phase 3 allocation stays at 6.4 acre-feet; the "
         "100-year water surface stays at 2,857.91 feet with 6.40 acre-feet of storage; the tributary area "
         "stays at 40.0 acres at 42 percent impervious; the basin stays in Common-Area Parcel “E”; the "
         "storage-indication input is unchanged — the 41.5 cfs at the crest is already in the accepted study's "
         "Table D-2, so the revision carries the accepted rating onto the sheet rather than introducing a new "
         "one; and the shared outfall cap stays at Q100 ≤ 78 cfs. What the resubmittal adds is freeboard "
         "dimensioning, sizing calculations, a section and apron details. Those are calculations and details "
         "of a structure whose governing elevations the District has already accepted, not a departure from "
         "the accepted allocation or allowance. No amendment is required and no District re-review is "
         "triggered before construction plan approval.", space_after=6)
    para("One internal file must not travel with the resubmittal. The Preliminary Detention Calculation dated "
         "09/25/2026 (marked PRELIMINARY — internal working file) assumes 65 percent impervious cover and a "
         "2.42 inch six-hour depth and derives 8.1 acre-feet of required storage. Neither input is the "
         "accepted basis (42 percent impervious, 2.38 inches), and that file is not the basis of the plans: "
         "the Phase 3 Hydrology Worksheet (07/2026) and Sheet C-9 both carry the accepted basis. It should be "
         "reconciled or withdrawn so that it cannot be mistaken for the design basis of record.")

    # ---------------------------------------------------------------- 6. action list
    head("6. Action list for the 11/30/2026 resubmittal")
    acts = [
        "C-9, Detail — Emergency Spillway (Weir): correct “0.71′ above 100-year water surface” to 0.59 ft "
        "(2,858.50 − 2,857.91).",
        "C-9, Basin Notes: correct “0.60′ above the 100-year water surface” to 0.59 ft, and restate the "
        "freeboard statement in the governing condition (spillway operating), not against the routed "
        "100-year pool alone.",
        "C-9, Section A-A′: add the freeboard dimension strings — 0.59 ft (100-year water surface to crest), "
        "0.70 ft (crest to top of berm) and 1.29 ft (100-year water surface to top of berm).",
        "C-9: add the spillway sizing calculation block — weir rating (crest 2,858.50 ft, L = 20 ft, "
        "W = 6 ft, design head, coefficient and its source), outlet control rating, and a stage–storage–"
        "discharge table reconciled to Table D-2 (6.40 ac-ft / 27.9 cfs at 2,857.91 ft; 7.12 ac-ft / "
        "41.5 cfs at 2,858.50 ft; 8.02 ac-ft at 2,859.20 ft).",
        "C-9: state and route the design overflow event. The detail calls for a routing check under a storm "
        "exceeding the 100-year, 24-hour input; that storm, its depth and its routing belong on the sheet or "
        "in a worksheet referenced by it.",
        "C-9: add a section through the spillway — crest, articulated concrete block surface, cutoff, "
        "downstream slope, apron and toe at 2,853.00 ft.",
        "C-9: add the apron details — thickness, bedding, filter/geotextile, cutoff or end treatment at the "
        "toe, and the sizing basis for the D50 = 12 in stone — consistent with the RFCD Hydrologic Criteria "
        "and Drainage Design Manual.",
        "C-9, Basin Notes: carry the accepted study's crest-stage outflow (41.5 cfs at 2,858.50 ft) in the "
        "basin summary, so storage and discharge are read against the same elevations.",
        "C-9, Basin Notes: provide the 1.0 ft minimum freeboard by calculation — on the screening values in "
        "Section 3, raise the top of berm to not less than 1.0 ft above the computed spillway design water "
        "surface (about 2,859.9 ft) with the crest held at 2,858.50 ft.",
        "Correspondence: request written authorisation from Clark County Public Works for the discharge point "
        "at the Windmill Lane gutter line, and carry the authorisation in the G-4 response; until it is in "
        "hand, do not show the gutter line as the outlet of record.",
        "Storm Drain HGL worksheet: correct the tailwater sensitivity note — the highest water surface the "
        "basin can physically hold is the top of berm at 2,859.20 ft, not the spillway crest — and re-run the "
        "sensitivity at 2,859.20 ft.",
        "QA/QC: re-check Tab 3 items 27 through 29 against the revised sheet, including the dimension values "
        "and not only the presence of the detail and sections.",
        "Internal: reconcile or withdraw DSR_Ph3_Detention_Calc_prelim.xlsx (65 percent impervious, "
        "2.42 in) against the accepted basis.",
    ]
    for i, a in enumerate(acts, 1):
        p = d.add_paragraph(f"{i}. {a}")
        p.paragraph_format.space_after = Pt(2)

    # ---------------------------------------------------------------- 7. closing
    head("7. Conclusion")
    para("Sheet C-9 as drawn is incomplete for the purposes of responding to comment G-4: the freeboard is "
         "not dimensioned, the sizing calculations do not exist, the spillway section is absent, the apron is "
         "detailed in plan only, and the sheet states the crest height above the 100-year water surface as two "
         "different values, neither of which is the arithmetic difference of its own elevations. The 1.29 foot "
         "freeboard does not hold once the spillway operates: the crest is 0.59 feet above the design pool, so "
         "the clearance to the top of berm is 0.70 feet at the crest and falls to zero at the berm, against a "
         "1.0 foot minimum. The overflow leaves the berm outside the outlet control structure and crosses "
         "Common-Area Parcel “E” to the Windmill Lane gutter line — improvements the County has accepted as "
         "Phase 2 offsite work, but a discharge point for which no document in the record grants a right. The "
         "revisions needed to answer G-4 are calculations and details of an already-accepted basin: they do "
         "not trigger a formal amendment to the accepted master drainage study.")
    para("Nothing in this memorandum alters the accepted Master Drainage Study, the 6.4 acre-foot Phase 3 "
         "allocation, the 100-year water surface at 2,857.91 feet, the basin location in Common-Area Parcel "
         "“E”, or the Q100 ≤ 78 cfs cap at the shared outfall.", space_after=10)

    para("Respectfully submitted,")
    para("SUNDIAL ENGINEERING & SURVEY", bold=True)
    para("")
    para("_________________________________________          Date: October 31, 2026")
    para("Max Braddock, Project Manager")
    para("(702) 555-0144  |  mbraddock@sundialensv.com")
    para("")
    para("Reviewed: ____________________________________")
    para("Jonathan R. Kessler, PE — Principal-in-Charge, Nevada PE No. 14203")

    OUT.mkdir(parents=True, exist_ok=True)
    path = OUT / NAME
    d.save(str(path))
    return path


if __name__ == "__main__":
    p = build()
    print(f"wrote {p} ({p.stat().st_size} bytes)")

```


## code/src/mutate.py

```text
#!/usr/bin/env python3
"""Mutation campaign on the delivered .docx.

Each mutant breaks ONE thing the rubric or my closure checks read, everywhere it is stated in the
memorandum (a fact stated twice is only broken when both statements are broken — otherwise the mutant is
testing redundancy, not the checks). A mutant counts as CAUGHT only when checks.py returns a non-empty
failure list; a crash or a technical failure (exit 2) is never a catch.

    python3 mutate.py <memo.docx> [--out DIR]
"""
from __future__ import annotations

import json
import re
import subprocess
import sys
from pathlib import Path

import docx

HERE = Path(__file__).resolve().parent
CHECKS = HERE / "checks.py"

# (name, [(pattern, replacement), ...]) — every pattern is applied with re.sub over every paragraph and cell
MUTATIONS = [
    ("M01_crest_outflow_37.5", [(r"41\.5", "37.5")]),
    ("M02_detail_0.71_to_0.70", [(r"0\.71", "0.70")]),
    ("M03_notes_0.60_to_0.59", [(r"0\.60", "0.59")]),
    ("M04_freeboard_conforms", [
        (r"The 1\.29 foot freeboard reported in the basin notes on Sheet C-9 as currently drawn does not hold "
         r"once the emergency spillway operates\.",
         "The 1.29 foot freeboard reported in the basin notes on Sheet C-9 meets the 1.0 foot minimum and conforms."),
        (r"does not hold once the emergency spillway operates", "holds once the emergency spillway operates"),
        (r"a noncompliant clearance of 0\.41 feet", "a compliant clearance of 0.41 feet"),
        (r"a standard it does not meet", "a standard it meets")]),
    ("M06_apron_12x20", [(r"15 feet by 20 feet", "12 feet by 20 feet"), (r"15 ft by 20 ft", "12 ft by 20 ft")]),
    ("M07_apron_d50_18", [(r"D50 of 12 inches", "D50 of 18 inches"), (r"D50 = 12 in", "D50 = 18 in")]),
    ("M08_discharge_right_asserted", [
        (r"But no document in the record grants a right to discharge the emergency spillway into the Windmill "
         r"Lane curb and gutter that Clark County Public Works has accepted\.",
         "The record grants a right to discharge the emergency spillway into the Windmill Lane curb and gutter "
         "that Clark County Public Works has accepted."),
        (r"there is none", "each of them does"),
        (r"lists no easement or right in favour of Clark County", "lists an easement and right in favour of Clark County"),
        (r"None of them establishes a discharge point", "Each of them establishes a discharge point"),
        (r"they do not grant a discharge right at Windmill Lane", "they grant a discharge right at Windmill Lane"),
        (r"Nothing on it runs from Parcel .E. to Windmill Lane", "It runs from Parcel E to Windmill Lane"),
        (r"There is no encroachment permit", "The encroachment permit is in the file:"),
        (r"the overflow path as drawn is not authorised", "the overflow path as drawn is authorised"),
        (r"but a discharge point for which no document in the record grants a right",
         "and a discharge point for which the record grants a right")]),
    ("M09_amendment_required", [
        (r"(?i)do not trigger a formal amendment", "do trigger a formal amendment"),
        (r"change none of those things", "change the accepted basis"),
        (r"No amendment is required and no District re-review is triggered",
         "A formal amendment is required and District re-review is triggered")]),
    ("M10_phase2_accepted_stripped", [
        (r"were accepted by Clark County Public Works as Phase 2 offsite work", "are shown on the Phase 1-2 final map"),
        (r"The receiving improvements are accepted County work", "The receiving improvements are as-built County work"),
        (r"Phase 2 Offsite Acceptance letter", "Phase 2 closeout correspondence"),
        (r"accepted against the \$3,200,000", "bonded against the $3,200,000"),
        (r"improvements the County has accepted as Phase 2 offsite work",
         "improvements the County has constructed under the Phase 2 contract")]),
    ("M11_wrong_date", [(r"October 31, 2026", "November 1, 2026"), (r"10/31/2026", "11/01/2026")]),
    ("M12_wrong_recipient", [(r"Mr\. Garrett Nokes, Vice President, Land Development — Callaway Land Partners, LLC",
                              "Mr. Dale Yeoman, PE — Clark County Public Works")]),
    ("M13_no_signature", [(r"Max Braddock, Project Manager", "The Project Manager")]),
    ("M14_through_outlet", [(r"bypasses the outlet control structure", "passes through the outlet control structure"),
                            (r"additional to, and outside, the metered release", "part of the metered release")]),
    ("M15_parcel_d", [(r"Common-Area Parcel .E.", "Common-Area Parcel D"),
                      (r"Common-Area Parcel .E. and exits", "Common-Area Parcel D and exits")]),
    ("M16_no_section_required", [
        (r"Sheet C-9 must be revised to include spillway sizing calculations and a spillway section to satisfy "
         r"comment G-4",
         "Sheet C-9 may be supported by a note referring the reviewer to the master drainage study to satisfy "
         "comment G-4"),
        (r"\d+\. C-9: add the spillway sizing calculation block[^\n]*", ""),
        (r"\d+\. C-9: add a section through the spillway[^\n]*", ""),
        (r"Sheet C-9 as drawn is incomplete for the purposes[^\n]*", "Sheet C-9 as drawn is substantially complete for the purposes of responding to comment G-4.")]),
    ("M17_wsel_2857.88", [(r"2,857\.91", "2,857.88")]),
    ("M18_apron_thickness_invented", [
        (r"it gives no thickness, no bedding or filter layer under the grouted rip-rap",
         "it gives a 12-inch thickness with 6-inch bedding under the grouted rip-rap")]),
    ("M19_storage_7.20", [(r"7\.12", "7.20")]),
    ("M20_outflow_29.7", [(r"27\.9 cfs", "29.7 cfs")]),
    ("M21_crest_2858.60", [(r"2,858\.50", "2,858.60")]),
]


def rewrite(path_in: Path, path_out: Path, pairs: list[tuple[str, str]]) -> int:
    d = docx.Document(str(path_in))
    hits = 0

    def apply(par):
        nonlocal hits
        text = par.text
        new = text
        for pat, rep in pairs:
            new, n = re.subn(pat, rep, new)
            hits += n
        if new != text:
            if par.runs:
                par.runs[0].text = new
                for r in par.runs[1:]:
                    r.text = ""
            else:
                par.add_run(new)

    for p in d.paragraphs:
        apply(p)
    for t in d.tables:
        for row in t.rows:
            for cell in row.cells:
                for p in cell.paragraphs:
                    apply(p)
    d.save(str(path_out))
    return hits


def main(argv: list[str]) -> int:
    if len(argv) < 2:
        print(__doc__)
        return 2
    src = Path(argv[1])
    out = Path(argv[argv.index("--out") + 1]) if "--out" in argv else HERE.parent / "mut"
    out.mkdir(parents=True, exist_ok=True)

    base = subprocess.run([sys.executable, str(CHECKS), "json", str(src)], capture_output=True, text=True)
    if base.returncode != 0 or not base.stdout.strip():
        print(f"BASELINE NOT GREEN (exit {base.returncode}) — refusing to run: {base.stderr[:300]}")
        return 2
    b = json.loads(base.stdout)
    print(f"baseline: {b['passed']}/{b['total']} checks pass\n")

    survived, caught, broken = 0, 0, 0
    for name, pairs in MUTATIONS:
        dst = out / f"{name}.docx"
        hits = rewrite(src, dst, pairs)
        if not hits:
            print(f"{name:32} NOT APPLIED — the text to mutate was not found")
            broken += 1
            continue
        r = subprocess.run([sys.executable, str(CHECKS), "json", str(dst)], capture_output=True, text=True)
        if r.returncode == 2 or not r.stdout.strip():
            print(f"{name:32} TECHNICAL FAILURE (exit {r.returncode}) — not a catch: {r.stderr[:140]}")
            broken += 1
            continue
        result = json.loads(r.stdout)
        if result["failures"]:
            caught += 1
            print(f"{name:32} CAUGHT   [{hits} edit(s)] red: {', '.join(f.split()[0] for f in result['failures'])}")
        else:
            survived += 1
            print(f"{name:32} SURVIVED [{hits} edit(s)] — no check noticed")
    print(f"\ncaught {caught}/{len(MUTATIONS)}; survived {survived}; not applied or broken {broken}")
    return 0 if caught == len(MUTATIONS) else 1


if __name__ == "__main__":
    sys.exit(main(sys.argv))

```


## deliverables/Memo_to_Callaway_Spillway_2026-10-31.txt

```text
SUNDIAL ENGINEERING & SURVEY

4720 South Rainbow Boulevard, Suite 210, Las Vegas, Nevada 89103  |  (702) 555-0142  |  NV Certificate of Authority C-3874

MEMORANDUM

TO:

Mr. Garrett Nokes, Vice President, Land Development — Callaway Land Partners, LLC, 8880 W. Sunset Road, Suite 400, Las Vegas, NV 89148

FROM:

Max Braddock, Project Manager — Sundial Engineering & Survey

CC:

Jonathan R. Kessler, PE (Principal-in-Charge, NV PE No. 14203); Project File 2201-03

DATE:

October 31, 2026 (10/31/2026)

RE:

Desert Sage Ranch Phase 3 — Clark County Public Works comment G-4, emergency spillway on Sheet C-9: freeboard, sizing, apron and the overflow discharge point. Improvement Plans IP-26-100288 / Final Map FM-26-100290 / Tentative Map TM-22-500041.

JOB NO.:

2201-03

1. Question presented and records reviewed

Comment G-4 in the Clark County Public Works first-round plan check letter dated October 9, 2026 (Dale Yeoman, PE, Civil Plans Examiner) reads: “The emergency spillway shown on Sheet C-9 lacks freeboard dimensioning and downstream apron details. Provide sizing calculations, spillway section, and erosion protection consistent with the RFCD Hydrologic Criteria and Drainage Design Manual.”

You asked three things before the drainage lead is put on the comment: whether the spillway as drawn holds up and whether G-4 can be answered with what is on the sheet today; whether the revisions that answer G-4 trigger an amendment to the accepted master drainage study; and what the spillway detail and the basin notes say on the revised C-9, including where the overflow ends up once it leaves the berm. The third of those splits into what the revised sheet must say and where the overflow may go, so the memorandum reaches four dispositions; each is a single disposition, not a set of options.

Records relied on, by short name used in this memorandum:

Sheet C-9 — Phase 3 Improvement Plans C-1 through C-9, issued for first submittal 08/21/2026 (Storage Summary Table, Section A-A′, Section B-B′, Detail — Emergency Spillway (Weir), Detail — Outlet Control Structure and Orifice Plate, Basin Notes).

Master Drainage Study, Desert Sage Ranch (140 acres), Revision 1 dated February 2022 (Table 4-1; Appendix C; Appendix D, Tables D-1 and D-2), accepted by CCRFCD by letter dated February 28, 2022 (the “accepted study”).

CCRFCD acceptance letter dated February 28, 2022 (FCD_MDS_Acceptance_Letter.pdf) — detention allocations, shared outfall allowance and the amendment triggers.

Phase 3 QA/QC Checklist dated 08/18/2026 (Tabs 2, 3 and 4); Phase 3 Hydrology Worksheet (07/2026); Phase 3 Storm Drain HGL worksheet (07/2026); Preliminary Detention Calculation dated 09/25/2026 (marked PRELIMINARY — internal working file).

Clark County Public Works Phase 2 comment letters (first and second round) and the Phase 2 comment response matrix; Phase 3 comment response log (resubmittal tracker).

Clark County Public Works Phase 2 Offsite Acceptance letter dated March 17, 2025, with the closed offsite punch list executed 03/12/2025.

Final Map FM-26-100290 plan check checkprint dated 08/21/2026; Phases 1–2 Final Map recorded 12/23/2024 as Book 175, Page 60 (NFM-24-0118); Preliminary Title Report SBT-2026-04188 (effective 07/01/2026); Conditions of Approval TM-22-500041 dated March 14, 2022.

2. Disposition 1 — Sheet C-9 as drawn is incomplete for the purposes of responding to comment G-4

Comment G-4 cannot be answered with what is on the sheet today. Sheet C-9 as drawn is incomplete for the purposes of responding to comment G-4 on four counts, and it contradicts itself on the fifth:

(a) Freeboard is asserted in words and is not dimensioned. The basin notes state: “Freeboard between the 100-year water surface (2,857.91′) and the top of berm (2,859.20′) is 1.29′, meeting the CCRFCD minimum freeboard requirement of 1.0 foot.” No dimension string on Sheet C-9 dimensions that freeboard. Section A-A′ and Section B-B′ carry elevation callouts only (basin bottom 2,851.00′, 100-year water surface 2,857.91′, top of berm 2,859.20′) and no dimension between them, so the reviewer is asked to accept a computed difference as a dimension.

(b) No calculation in the record sizes the spillway as drawn. Detail — Emergency Spillway (Weir) gives the crest elevation (2,858.50′), the weir length (20′ along the berm) and the weir width (6′ in the direction of flow) — and no design head, no discharge–head relation and no capacity. The only spillway discharge figure in the record is in the accepted study's Table D-2 (41.5 cfs at the crest), and Sheet C-9 does not carry it. The detail refers to a “design overflow discharge (routing check under storm exceeding 100-year, 24-hour input)”, but neither the sheet nor the Phase 3 Hydrology Worksheet contains that storm or that routing: the worksheet evaluates the 100-year 6-hour storm (2.38 in) and the 100-year 24-hour storm (3.18 in) and nothing above them. The design case the spillway exists for is therefore not stated and not run anywhere in the record.

(c) There is no spillway section. Sheet C-9 carries Section A-A′ (longitudinal, east–west, through the outlet control structure) and Section B-B′ (transverse, north–south, through the basin midpoint). Neither section cuts the weir: there is no section through the crest, the articulated concrete block surface, the downstream slope, the apron and the toe. QA/QC Checklist Tab 3 items 27 through 29 record the weir detail and “Basin outlet swale section” as verified, but those items check presence and keying — item 27 is recorded as “Detail present; crest elevation matches basin section”, and Section B-B′ is the basin midpoint section, not a section through the spillway. No check on the checklist compares the crest height callout on the detail with the basin note.

(d) The apron is specified in plan only. Sheet C-9 specifies the apron as 15 feet by 20 feet of grouted rip-rap with a D50 of 12 inches, extending from the weir toe to natural grade beyond the berm at elevation 2,853.00′. That is the whole of what the record specifies for the apron: it gives no thickness, no bedding or filter layer under the grouted rip-rap, no cutoff or end treatment at the toe, and no geotextile under the apron (the geotextile filter fabric named on the sheet is the bedding under the articulated concrete block weir surface, not under the apron). An apron detailed only in plan cannot be constructed or checked, and “downstream apron details” is precisely what G-4 asks for.

(e) The emergency spillway detail and the basin notes on Sheet C-9 give different values for the height of the spillway crest above the 100-year water surface. The detail callout reads “Weir crest elevation 2,858.50′ (0.71′ above 100-year water surface)”; the basin notes read “Emergency spillway crest set at 2,858.50′, 0.60′ above the 100-year water surface and 0.70′ below the top of berm.” The sheet's own elevations give neither figure: 2,858.50′ − 2,857.91′ = 0.59 feet. The detail figure is 0.12 feet high and the basin note is 0.01 feet high, and 0.71 feet appears to be the 0.70 feet from the crest to the top of berm carried into the wrong sentence. Two printed values for one dimension, neither of them the arithmetic value, is the sheet contradicting itself on exactly the dimension the comment is about.

Disposition: G-4 is answered on the resubmittal, not on this sheet. Sheet C-9 must be revised to include spillway sizing calculations and a spillway section to satisfy comment G-4, together with the freeboard dimensioning and the apron details set out in Section 5 below. The comment is not one that can be closed with a note or a cross-reference to a document that does not contain the work.

3. Disposition 2 — what the spillway detail and the basin notes must say on the revised C-9, and the freeboard finding

The values below are the record's, each with the record it comes from. The last three rows are mine and are marked as such.

Quantity

Value

Record

Emergency spillway weir crest

2,858.50 ft

C-9 Detail — Emergency Spillway (Weir); C-9 Basin Notes

100-year water surface

2,857.91 ft

C-9 Sections A-A′ and B-B′; MDS Table D-2

Top of berm

2,859.20 ft

C-9 Sections A-A′ and B-B′; C-9 Basin Notes

Crest height above the 100-year water surface

0.71 ft (detail) / 0.60 ft (basin notes) / 0.59 ft (arithmetic)

C-9 Detail and Basin Notes; C-9 elevations

Crest below top of berm

0.70 ft

C-9 Basin Notes

Freeboard, 100-year water surface to top of berm

1.29 ft

C-9 Basin Notes

Storage at the 100-year water surface

6.40 ac-ft

C-9 Storage Summary; MDS Table D-2

Storage at the spillway crest

7.12 ac-ft

C-9 Storage Summary; MDS Table D-2

Storage at the top of berm

8.02 ac-ft

C-9 Storage Summary; MDS Table D-2

Outflow at the 100-year water surface

27.9 cfs

MDS Table D-2

Outflow at the emergency spillway crest

41.5 cfs

MDS Table D-2

Weir length and width

20 ft along the berm, 6 ft in the direction of flow

C-9 Detail

Downstream apron

15 ft by 20 ft grouted rip-rap, D50 = 12 in, toe to grade at 2,853.00 ft

C-9 Detail

Phase 3 detention allocation

6.4 ac-ft

MDS Table 4-1; CCRFCD letter 02/28/2022

Shared outfall cap

Q100 ≤ 78 cfs

MDS Section 5; CCRFCD letter 02/28/2022

Weir capacity at 0.70 ft of head (screening, mine)

30.5 cfs

computed: broad-crested weir, L = 20 ft, coefficient 2.6

Crest-stage water surface with the outlet structure operating (screening, mine)

2,858.88 ft

computed: 41.5 cfs less the outlet share, weir head 0.38 ft

Freeboard to the top of berm in that condition (screening, mine)

0.32 ft

computed

What the revised basin notes must state — and what the notes as drawn omit — is the accepted study's outflow at the emergency spillway crest. The revised basin notes on Sheet C-9 give the master drainage study outflow of 41.5 cubic feet per second (cfs) at the emergency spillway crest, at elevation 2,858.50 feet with 7.12 acre-feet in storage, against 27.9 cfs at the 100-year water surface at 2,857.91 feet with 6.40 acre-feet (Master Drainage Study, Table D-2). The C-9 Storage Summary already closes to Table D-2 on storage at all three elevations (6.40, 7.12 and 8.02 acre-feet) but it has no outflow column, so the crest-stage discharge appears nowhere on the sheet and there is no stated capacity for the weir to be checked against. Storage and discharge have to sit in the same table, against the same elevations, or the reviewer cannot check either.

The 1.29 foot freeboard reported in the basin notes on Sheet C-9 as currently drawn does not hold once the emergency spillway operates. The 1.29 feet is the clearance from the routed 100-year water surface to the top of berm. The crest sits 0.59 feet above that water surface, so the spillway is designed to engage in events above the routed 100-year event; once it engages the water surface is at or above 2,858.50 feet, the clearance to the top of berm is 0.70 feet or less, and it falls to zero at the berm. Measured as the minimum freeboard standard requires — above the design water surface, and above the spillway design water surface in the condition that governs — the design pool is only 0.59 feet below the crest, a noncompliant clearance of 0.41 feet against the 1.0 foot minimum that the basin notes themselves invoke and that the County applied to this project in Phase 2 comment G-3 (“Freeboard shall be a minimum of 1.0 ft above the design HGL and shall be dimensioned”). The 1.29 foot figure is not wrong as a measurement of the empty-spillway condition; it is a statement about the only event in which the spillway never runs, and it is offered as conformance with a standard it does not meet in the event the spillway exists for.

A screening calculation, mine and not the design, shows what the sizing block has to produce. Treating the spillway as a broad-crested weir 20 feet long with a coefficient of 2.6 in U.S. customary units (a deliberately conservative value for a rough articulated-concrete-block surface), the weir passes 30.5 cfs at the 0.70 feet of head available to the top of berm — 11.0 cfs short of the 41.5 cfs the accepted study assigns to the crest stage. If the outlet control structure is operating, the 12 inch by 18 inch orifice plate carries 27.9 cfs at the 100-year water surface and, scaling with the square root of head, about 29.2 cfs at the crest, so the weir would carry about 12.3 cfs at 0.38 feet of head; the water surface would then sit at about 2,858.88 feet, leaving 0.32 feet to the top of berm — a third of the 1.0 foot standard. If the design case is the outlet structure blocked, which is the case an emergency spillway is normally sized for, the weir alone needs 0.86 feet of head to pass 41.5 cfs, putting the water surface at about 2,859.36 feet, 0.16 feet above the top of berm. The exact figure is not the point: every one of these numbers has to be computed on the revised sheet, against a stated design case, and the sheet computes none of them. The illustrative remedies are a longer weir, a lower crest or a higher top of berm; because the crest elevation and the 41.5 cfs at that elevation are tied together in the accepted study, raising the top of berm to not less than 1.0 foot above the computed spillway design water surface is the one that leaves the accepted storage basis untouched.

One more record error has to be corrected with the same revision: the Phase 3 Storm Drain HGL worksheet (07/2026) describes its tailwater sensitivity case as 2,858.50 feet, “the highest water surface the basin can physically hold”. It is not. The basin holds 8.02 acre-feet to the top of berm at 2,859.20 feet, and the spillway exists precisely so that the water surface may rise above the crest. The sensitivity case should be run at 2,859.20 feet.

4. Disposition 3 — where the overflow ends up once it leaves the berm, and whether it may go there

Path of the overflow. Flow passing over the emergency spillway bypasses the outlet control structure that limits discharge to the shared outfall. The 12 inch by 18 inch orifice plate in the west-end headwall is the element that holds the site to the 78 cfs shared-outfall cap; spillway flow crosses the east berm outside it, drops over the 15 foot by 20 foot rip-rap apron to grade at 2,853.00 feet, and — in the words of the C-9 detail — “flow spreads to downstream open swale in Common-Area Parcel ‘E’ and exits at Windmill Ln. gutter line.” The emergency spillway overflow therefore discharges across Common-Area Parcel “E” to the Windmill Lane gutter line, and that discharge is additional to, and outside, the metered release the 78 cfs cap governs.

The receiving improvements are accepted County work. The Windmill Lane collector half-improvements — paving, curb, gutter, sidewalk, striping and signage — are item OS-1 (bonded value $612,000) in the Clark County Public Works Phase 2 Offsite Acceptance letter dated March 17, 2025. The Windmill Lane half-improvements receiving the emergency spillway overflow were accepted by Clark County Public Works as Phase 2 offsite work, with the closed offsite punch list executed by the assigned County construction inspector on 03/12/2025 (13 items opened, 13 closed, 0 outstanding) and the offsite subtotal of $1,100,000 accepted against the $3,200,000 Phase 2 performance bond.

But no document in the record grants a right to discharge the emergency spillway into the Windmill Lane curb and gutter that Clark County Public Works has accepted. The record was searched for any easement, licence, permit, agreement or coordination record that would authorise a concentrated discharge at that point, and there is none:

The Final Map FM-26-100290 checkprint grants a 15 foot drainage easement on the west line of Common-Area Parcel “D” (the park), a 20 foot dry utility easement in Book 175 Page 60 along the north boundary, and a 10 foot public utility easement across the fronts of the lots. Nothing on it runs from Parcel “E” to Windmill Lane.

The recorded Phases 1–2 Final Map (Book 175, Page 60) dedicates streets, public utility, drainage and sight visibility easements in favour of Clark County within Phases 1 and 2. None of them establishes a discharge point into the Windmill Lane curb and gutter.

The accepted study provides one discharge point for the 140 acres: the shared outfall at the southwest corner of the site, through the existing 30 inch RCP stub (invert 2,850.75 feet per the Phase 2 as-builts), under the 78 cfs allowance. The spillway discharge at the Windmill Lane gutter line is a second, different discharge point that the accepted study does not describe.

The Conditions of Approval TM-22-500041 require drainage easements to be granted by the recorded final map for the phase in which the affected area lies; they do not grant a discharge right at Windmill Lane, and the Phase 3 final map has not been recorded.

The Preliminary Title Report SBT-2026-04188 Schedule B lists no easement or right in favour of Clark County for drainage discharge at Windmill Lane.

The Phase 2 acceptance letter is an acceptance of constructed work under Chapter 30.32 and Article VI of the Subdivision Improvement Agreement. It expressly limits itself to the offsite component of the Work and states that nothing in it shall be construed to accept, waive or otherwise address any portion of the Work not enumerated. Accepting the curb and gutter as constructed is not an authorisation to discharge into it.

There is no encroachment permit, drainage discharge authorisation, licence or coordination record with Clark County Public Works for a concentrated discharge at this location anywhere in the project file.

Disposition: the overflow path as drawn is not authorised, and the G-4 response must not present the Windmill Lane gutter line as the outlet of record. The single disposition I recommend is to obtain a written drainage discharge authorisation from Clark County Public Works for this discharge point — together with whatever easement or right the County requires over the flow path across Parcel “E” — and to carry that authorisation in the resubmittal as the support for the discharge point shown on the revised C-9. If the County will not authorise a concentrated discharge at the gutter line, the overflow path must be redesigned before G-4 can close; the detention design itself is unaffected either way.

Note that the volume at stake is not the design release. The spillway only runs in events above the routed 100-year event, so the discharge is an emergency overflow, not a routine release; but it is precisely because it is uncontrolled and outside the metered outlet that the discharge point has to be authorised rather than assumed.

5. Disposition 4 — the proposed spillway revisions do not trigger a formal amendment to the accepted master drainage study

The CCRFCD acceptance letter dated February 28, 2022 sets out what requires a formal amendment: changes to the accepted detention allocations, the outfall allowance, or the tributary area assumptions, including without limitation any proposed revision of phase tributary area, impervious cover assumption, hydrologic methodology, storage-indication input, phase boundary, detention basin location, or shared outfall discharge cap. The Master Drainage Study states the same rule in Sections 4.4 and 7.3.

The proposed spillway revisions do not trigger a formal amendment to the accepted master drainage study, because they change none of those things. The Phase 3 allocation stays at 6.4 acre-feet; the 100-year water surface stays at 2,857.91 feet with 6.40 acre-feet of storage; the tributary area stays at 40.0 acres at 42 percent impervious; the basin stays in Common-Area Parcel “E”; the storage-indication input is unchanged — the 41.5 cfs at the crest is already in the accepted study's Table D-2, so the revision carries the accepted rating onto the sheet rather than introducing a new one; and the shared outfall cap stays at Q100 ≤ 78 cfs. What the resubmittal adds is freeboard dimensioning, sizing calculations, a section and apron details. Those are calculations and details of a structure whose governing elevations the District has already accepted, not a departure from the accepted allocation or allowance. No amendment is required and no District re-review is triggered before construction plan approval.

One internal file must not travel with the resubmittal. The Preliminary Detention Calculation dated 09/25/2026 (marked PRELIMINARY — internal working file) assumes 65 percent impervious cover and a 2.42 inch six-hour depth and derives 8.1 acre-feet of required storage. Neither input is the accepted basis (42 percent impervious, 2.38 inches), and that file is not the basis of the plans: the Phase 3 Hydrology Worksheet (07/2026) and Sheet C-9 both carry the accepted basis. It should be reconciled or withdrawn so that it cannot be mistaken for the design basis of record.

6. Action list for the 11/30/2026 resubmittal

1. C-9, Detail — Emergency Spillway (Weir): correct “0.71′ above 100-year water surface” to 0.59 ft (2,858.50 − 2,857.91).

2. C-9, Basin Notes: correct “0.60′ above the 100-year water surface” to 0.59 ft, and restate the freeboard statement in the governing condition (spillway operating), not against the routed 100-year pool alone.

3. C-9, Section A-A′: add the freeboard dimension strings — 0.59 ft (100-year water surface to crest), 0.70 ft (crest to top of berm) and 1.29 ft (100-year water surface to top of berm).

4. C-9: add the spillway sizing calculation block — weir rating (crest 2,858.50 ft, L = 20 ft, W = 6 ft, design head, coefficient and its source), outlet control rating, and a stage–storage–discharge table reconciled to Table D-2 (6.40 ac-ft / 27.9 cfs at 2,857.91 ft; 7.12 ac-ft / 41.5 cfs at 2,858.50 ft; 8.02 ac-ft at 2,859.20 ft).

5. C-9: state and route the design overflow event. The detail calls for a routing check under a storm exceeding the 100-year, 24-hour input; that storm, its depth and its routing belong on the sheet or in a worksheet referenced by it.

6. C-9: add a section through the spillway — crest, articulated concrete block surface, cutoff, downstream slope, apron and toe at 2,853.00 ft.

7. C-9: add the apron details — thickness, bedding, filter/geotextile, cutoff or end treatment at the toe, and the sizing basis for the D50 = 12 in stone — consistent with the RFCD Hydrologic Criteria and Drainage Design Manual.

8. C-9, Basin Notes: carry the accepted study's crest-stage outflow (41.5 cfs at 2,858.50 ft) in the basin summary, so storage and discharge are read against the same elevations.

9. C-9, Basin Notes: provide the 1.0 ft minimum freeboard by calculation — on the screening values in Section 3, raise the top of berm to not less than 1.0 ft above the computed spillway design water surface (about 2,859.9 ft) with the crest held at 2,858.50 ft.

10. Correspondence: request written authorisation from Clark County Public Works for the discharge point at the Windmill Lane gutter line, and carry the authorisation in the G-4 response; until it is in hand, do not show the gutter line as the outlet of record.

11. Storm Drain HGL worksheet: correct the tailwater sensitivity note — the highest water surface the basin can physically hold is the top of berm at 2,859.20 ft, not the spillway crest — and re-run the sensitivity at 2,859.20 ft.

12. QA/QC: re-check Tab 3 items 27 through 29 against the revised sheet, including the dimension values and not only the presence of the detail and sections.

13. Internal: reconcile or withdraw DSR_Ph3_Detention_Calc_prelim.xlsx (65 percent impervious, 2.42 in) against the accepted basis.

7. Conclusion

Sheet C-9 as drawn is incomplete for the purposes of responding to comment G-4: the freeboard is not dimensioned, the sizing calculations do not exist, the spillway section is absent, the apron is detailed in plan only, and the sheet states the crest height above the 100-year water surface as two different values, neither of which is the arithmetic difference of its own elevations. The 1.29 foot freeboard does not hold once the spillway operates: the crest is 0.59 feet above the design pool, so the clearance to the top of berm is 0.70 feet at the crest and falls to zero at the berm, against a 1.0 foot minimum. The overflow leaves the berm outside the outlet control structure and crosses Common-Area Parcel “E” to the Windmill Lane gutter line — improvements the County has accepted as Phase 2 offsite work, but a discharge point for which no document in the record grants a right. The revisions needed to answer G-4 are calculations and details of an already-accepted basin: they do not trigger a formal amendment to the accepted master drainage study.

Nothing in this memorandum alters the accepted Master Drainage Study, the 6.4 acre-foot Phase 3 allocation, the 100-year water surface at 2,857.91 feet, the basin location in Common-Area Parcel “E”, or the Q100 ≤ 78 cfs cap at the shared outfall.

Respectfully submitted,

SUNDIAL ENGINEERING & SURVEY

_________________________________________          Date: October 31, 2026

Max Braddock, Project Manager

(702) 555-0144  |  mbraddock@sundialensv.com

Reviewed: ____________________________________

Jonathan R. Kessler, PE — Principal-in-Charge, Nevada PE No. 14203
```
