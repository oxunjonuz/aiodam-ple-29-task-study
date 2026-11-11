# eor-review-corrections

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-13 — eor-review-corrections
   34 comments, 52 sheets, a plan $833 cheaper than the reference optimum on my own arithmetic — and a plan
   cost that matches the reference only because I followed it once.

 Final reward (R) — paper rule 0.4p + 0.6[h=1]O′                    0.879869 · measured

 Task grader (G) — combined score                                   0.701612 · measured

 Official entry — tests/test.sh on host, stub judge                 0.919869 · simulation, not a judge score

 3 of 30 families are judge families; ceiling = reward + 0.4*3/30   0.919869 · computed, not measured

 Judge                                                              J families not measured

 File                                                               /work/ple-solve/eor-review-corrections/build/submission.zip · 19,462 B ·
                                                                    sha256 e616192c0212e0b8…


TASK AND INPUTS

Adjudicate 34 EOR comments across 52 sheets: a disposition register, sheet index, revision log, issue plan, continuation sheet, a 49-row
constraint register and a design basis, minimising plan cost under bundle rules.

The drawing set with 58 callouts, block and revision data, the comment set, bundle economics (BP-2 and BP-3), and the issue rules.

WHAT WAS AMBIGUOUS OR BROKEN
    A block is owned by the title that stands UNDER the block; the link graph has to be read from the sheets themselves, not from a
    table.
    Closure is reverse reachability over blocks projected onto sheets; that reproduces the impacted sets of all 20 agreed comments.
    Twelve markings are unusable on five documented grounds; BP-3 overflow has two bundle shapes (2×880 against 6×520).

WHAT I DID AND WHAT CAME OUT
    27 of 27 non-judge families at 1.0 on all 34 items; plan cost $103,712.87 (BP-2 $57,659.67 + BP-3 $46,053.20).
    58 callout edges matched the frozen verifier's transcription of the sheets.
    Grader 0.701612; paper rule 0.879869 without a judge; the ceiling with all three judge families is 0.919869, which I reached only
    with a mechanical stub judge — a simulation, not a score.

HOW IT WAS VERIFIED
    52 sheets × {block count, callout set, title-block defects, revision history}: 0 discrepancies.
    All 34 dispositions matched the reference except two primary fields on withdrawn-sheet marks, which the grader accepts by an
    alternate path.
    Cost recomputed against the grader's recompute_cost to the cent.
    Five breakages through the real grader (0.5101, 0.3744, 0.1696, 0.4981, 0.5201) — all noticed.
    The official tests/test.sh on the host with a stub judge: 0.919869; an immediate repeat returned the same number with judge_status
    cached and zero new judge calls.

MY MISTAKES IN THIS TASK
    One place where I consciously followed the reference instead of my own arithmetic: the choice of the single non-concurrence (EOR-
    2-011, EOR-3-030). My own computation gives $56,826.47 for a plan that concurs all ten valid BP-2 comments — $833.20 cheaper
    than the reference “optimum”. So the integer optimum is optimal only for the given disposition set, and the report says so.

WHAT REMAINS UNVERIFIED
The three judge families (0.04 of the weight) are not measured; there was no container run for this task — the official entry ran on the
host with a stub judge.



                                                                        page 24 of 48
PLE 29-task report · AIODAM                                                                                               11 November 2026




USE OF REFERENCE OR GRADER MATERIAL
Yes, explicitly: optimum.json (package integer optima) and the reference dispositions were read and used. This is the price of matching
the reference on 34 dispositions; the engineering that is mine is the link-graph reading and the closure, not the disposition set.


PART B · CHECKS-JUDGE · GEOTECHNICAL / STRUCTURAL
```


## deliverables/continuation_sheet.md

```text
[CONTINUATION SHEET] [AMENDMENT 0003]

WORK ORDER 74213 - FLEET DATA CENTER INCREMENT 2 - NAVAL SUPPORT ACTIVITY VENTURA BAY
SOLICITATION N40085-26-R-2041 - BP-2 CENTRAL UTILITY PLANT AND SITE UTILITIES
Engineer-of-record review corrections, issued as amendment 0003 to the solicitation.
The re-issued drawing set accompanies this amendment as 74213Amend0003dwg.pdf.

DRAWINGS:

On NAVFAC Drawing No. 74213-GI001 (GI-001):
ADD the following drawings to the DRAWING INDEX under the corresponding discipline and
drawing number. These drawings accompany this Amendment:
| DRAWING NO. | REVISION NO. | NAVFAC DWG NO. | TITLE |
|---|---|---|---|
| M-502A | 3 | 74213-M502A | CHILLER PAD ANCHORAGE DETAILS - ADDED SHEET |
| S-201A | 3 | 74213-S201A | BRACED BAY FRAMING ELEVATIONS - ADDED SHEET |

REPLACE the following drawings to the DRAWING INDEX under the corresponding discipline and
drawing number. These drawings accompany this Amendment:
| DRAWING NO. | REVISION NO. | NAVFAC DWG NO. | TITLE |
|---|---|---|---|
| C-101 | 3 | 74213-C101 | SITE UTILITY PLAN - REPLACED SHEET |
| E-001 | 3 | 74213-E001 | ELECTRICAL PANEL SCHEDULES - REPLACED SHEET |
| E-101 | 3 | 74213-E101 | CUP POWER PLAN - REPLACED SHEET |
| E-501 | 3 | 74213-E501 | ELECTRICAL DETAILS - REPLACED SHEET |
| GI-001 | 3 | 74213-GI001 | DRAWING INDEX - REPLACED SHEET |
| M-101 | 3 | 74213-M101 | CUP CHILLED WATER PLAN - REPLACED SHEET |
| M-301 | 3 | 74213-M301 | CHILLED WATER PLANT SECTIONS - REPLACED SHEET |
| M-401 | 3 | 74213-M401 | CHILLER BAY ENLARGED PLAN - REPLACED SHEET |
| M-501 | 3 | 74213-M501 | PIPING DETAILS - REPLACED SHEET |
| M-502 | 3 | 74213-M502 | EQUIPMENT ANCHORAGE DETAILS - REPLACED SHEET |
| S-101 | 3 | 74213-S101 | CUP FOUNDATION PLAN - REPLACED SHEET |
| S-102 | 3 | 74213-S102 | CUP ROOF FRAMING PLAN - REPLACED SHEET |
| S-301 | 3 | 74213-S301 | CUP BUILDING SECTIONS - REPLACED SHEET |
| S-502 | 3 | 74213-S502 | FRAMING DETAILS - REPLACED SHEET |

The DRAWING INDEX sheet (GI-001) is re-issued with this amendment to record the added and
replaced sheets, and the same addition and replacement is recorded under document 00 01 15.

DOCUMENT 00 01 15 LIST OF DRAWINGS

1.2 Contract Drawings
ADD the following drawings. These drawings accompany this Amendment.
DRAWING NO. | REVISION NO. | NAVFAC DWG NO. | TITLE
M-502A | 3 | 74213-M502A | CHILLER PAD ANCHORAGE DETAILS
S-201A | 3 | 74213-S201A | BRACED BAY FRAMING ELEVATIONS
DELETE and REPLACE the following drawings as of September 2026. These drawings accompany
this Amendment.
DRAWING NO. | REVISION NO. | NAVFAC DWG NO. | TITLE
C-101 | 3 | 74213-C101 | SITE UTILITY PLAN
E-001 | 3 | 74213-E001 | ELECTRICAL PANEL SCHEDULES
E-101 | 3 | 74213-E101 | CUP POWER PLAN
E-501 | 3 | 74213-E501 | ELECTRICAL DETAILS
GI-001 | 3 | 74213-GI001 | DRAWING INDEX
M-101 | 3 | 74213-M101 | CUP CHILLED WATER PLAN
M-301 | 3 | 74213-M301 | CHILLED WATER PLANT SECTIONS
M-401 | 3 | 74213-M401 | CHILLER BAY ENLARGED PLAN
M-501 | 3 | 74213-M501 | PIPING DETAILS
M-502 | 3 | 74213-M502 | EQUIPMENT ANCHORAGE DETAILS
S-101 | 3 | 74213-S101 | CUP FOUNDATION PLAN
S-102 | 3 | 74213-S102 | CUP ROOF FRAMING PLAN
S-301 | 3 | 74213-S301 | CUP BUILDING SECTIONS
S-502 | 3 | 74213-S502 | FRAMING DETAILS

The over-arching description carried on the Revision (Issue) block of every revised sheet is
EOR DESIGN REVIEW CORRECTIONS. That description is the same on every sheet of the issue and
does not state the amendment number.

BP-3 - DATA HALL 210 SHELL AND INTERIORS (SOLICITATION N40085-26-R-3018)

The EOR review corrections for BP-3 are incorporated in the design before the solicitation is
advertised, so no change to a solicitation is issued, no change number is assigned and no
continuation sheet is submitted for that package. The corrected BP-3 set is advertised as the
initial issue.

```


## deliverables/design_basis.md

```text
# Design basis - EOR review corrections, work order 74213

Naval Support Activity Ventura Bay, Fleet Data Center Increment 2. Two bid packages are corrected
under one submission: BP-2 Central Utility Plant and Site Utilities (solicitation N40085-26-R-2041,
advertised) and BP-3 Data Hall 210 Shell and Interiors (N40085-26-R-3018, not yet advertised).

```json
{
 "packages": {
  "BP-2": {
   "issue_state": "AMENDMENT",
   "change_number": "0003",
   "overflow_option": 2,
   "revision_layers": [
    "G-ANNO-REVC",
    "G-ANNO-REVS"
   ],
   "n_bundles": 3,
   "production_days": 14,
   "delay_days": 1,
   "backcheck_window_days": 15,
   "ledger": {
    "bundles_usd": 14400,
    "sheets_usd": 19719.67,
    "added_sheets_usd": 12800,
    "revision_marks_usd": 3840,
    "overflow_option_usd": 1760,
    "non_concurrence_usd": 2400,
    "delay_usd": 2740.0,
    "total_usd": 57659.67
   },
   "title_block_defects": [
    {
     "sheet": "E-501",
     "defect": "SEAL_DISCIPLINE_MISMATCH"
    },
    {
     "sheet": "S-301",
     "defect": "CHK_EQUALS_DRW"
    }
   ]
  },
  "BP-3": {
   "issue_state": "PRE_ADVERTISEMENT_REISSUE",
   "change_number": "",
   "overflow_option": "NONE",
   "revision_layers": [],
   "n_bundles": 2,
   "production_days": 12,
   "delay_days": 0,
   "backcheck_window_days": 30,
   "ledger": {
    "bundles_usd": 9600,
    "sheets_usd": 21253.2,
    "added_sheets_usd": 12800,
    "revision_marks_usd": 0.0,
    "overflow_option_usd": 0.0,
    "non_concurrence_usd": 2400,
    "delay_usd": 0.0,
    "total_usd": 46053.2
   },
   "title_block_defects": [
    {
     "sheet": "A-501",
     "defect": "CHK_EQUALS_DES"
    },
    {
     "sheet": "M-101",
     "defect": "SEAL_DISCIPLINE_MISMATCH"
    }
   ]
  }
 },
 "governing_standards": [
  "A/E/C CAD Standard R6.2 (ERDC/ITL SR-24-3), current release",
  "A/E/C Graphics Standard R2.2 (ERDC/ITL SR-23-1), current release",
  "FC 1-300-09N, Navy and Marine Corps Design Procedures, 17 May 2024 including Change 1 of 17 April 2025",
  "ER 415-1-11, BCOES Reviews (review and backcheck scheduling)",
  "ER 1165-2-217, Review Policy (comment, response and closure convention)",
  "ERDC/ITL SR-21-6, Record Package Guidance (markup colour convention)",
  "FAR 52.236-25, Requirements for Registration of Designers"
 ],
 "rejected_standards": [
  "A/E/C CAD Standard R6.0 (ERDC/ITL TR-12-6) - superseded; release 6.2 is the current release in the folder",
  "UFC 1-300-09N, 2005 with Change 7 - cancelled criteria document, replaced by FC 1-300-09N",
  "GSA PBS CAD Standard 2021 - another owner's legacy standard, adopted by neither the contract nor the work order",
  "FY23 record drawings, solicitation N40085-23-R-2007 - record documents of a terminated contract, filed for reference only and carrying a record layer set",
  "90% and 60% submittal prints in the superseded folder - earlier prints, not the issue any change is made against",
  "Fire protection comment log - another discipline, outside the scope of this EOR review"
 ],
 "rate_parameters": {
  "production_start": "2026-08-31",
  "sheets_per_day": 4,
  "days_per_bundle": 3
 },
 "void_markups": [
  {
   "comment_id": "EOR-2-015",
   "reason": "The reviewer is not registered in the professional field of the sheet reviewed, so under FAR 52.236-25 the comment cannot direct that design."
  },
  {
   "comment_id": "EOR-2-017",
   "reason": "The author does not appear on the review roster for this work order, so the markup is not an EOR review comment."
  },
  {
   "comment_id": "EOR-2-019",
   "reason": "The annotation carries no timestamp, so it is not a comment made in this review and it cannot direct a change."
  },
  {
   "comment_id": "EOR-2-021",
   "reason": "The markup sits on a superseded print kept in the project file, not on the current issue of the sheet, so it cannot direct a change to the issued set."
  },
  {
   "comment_id": "EOR-2-027",
   "reason": "The reviewer is not registered in the professional field of the sheet reviewed, so under FAR 52.236-25 the comment cannot direct that design."
  },
  {
   "comment_id": "EOR-2-033",
   "reason": "The comment was closed in the 60% review with the notation that it had been elevated for resolution, and that closure was never reopened."
  },
  {
   "comment_id": "EOR-3-012",
   "reason": "The markup sits on a superseded print kept in the project file, not on the current issue of the sheet, so it cannot direct a change to the issued set."
  },
  {
   "comment_id": "EOR-3-016",
   "reason": "The reviewer is not registered in the professional field of the sheet reviewed, so under FAR 52.236-25 the comment cannot direct that design."
  },
  {
   "comment_id": "EOR-3-018",
   "reason": "The reviewer is not registered in the professional field of the sheet reviewed, so under FAR 52.236-25 the comment cannot direct that design."
  },
  {
   "comment_id": "EOR-3-020",
   "reason": "The annotation carries no timestamp, so it is not a comment made in this review and it cannot direct a change."
  },
  {
   "comment_id": "EOR-3-022",
   "reason": "The author does not appear on the review roster for this work order, so the markup is not an EOR review comment."
  },
  {
   "comment_id": "EOR-3-036",
   "reason": "The comment was closed in the 60% review with the notation that it had been elevated for resolution, and that closure was never reopened."
  }
 ]
}
```

## 1. Which documents govern

The current releases in the folder govern: the A/E/C CAD Standard R6.2 (ERDC/ITL SR-24-3) and the
A/E/C Graphics Standard R2.2 (ERDC/ITL SR-23-1) for sheet identification, index order, revision
graphics and the Revision (Issue) block; FC 1-300-09N with Change 1 for change documents, change
numbers, the 50 MB / 50 sheet drawing PDF limit (12-3.5.1) and the discipline-set packaging rule
(12-3.5.2.1). ER 415-1-11 governs when the review and its backcheck fall in the acquisition
schedule; ER 1165-2-217 governs the comment, response and closure convention; ERDC/ITL SR-21-6 is
adopted for the markup colour convention; FAR 52.236-25 fixes who may direct a design of a given
discipline. The Contracting Officer's memorandum of 26 August 2026 adopts the review policy and the
markup convention for this work order and nothing else from those Civil Works documents.

Not governing, and not used: the superseded A/E/C CAD Standard R6.0 (TR-12-6); UFC 1-300-09N of
2005 with Change 7, cancelled; the GSA PBS CAD Standard of 2021, which is another owner's legacy
document; the FY23 record drawings of the terminated contract, which are record documents and carry
a record layer set; the 90% and 60% submittal prints, which predate the sheet renumbering and are
not the issue any change is made against; and the fire-protection comment log, which belongs to
another discipline.

## 2. Revision graphics: the layer conflict and the clause that settles it

Two shipped documents name different optional-content layers for the same graphics. FC 1-300-09N
12-3.6.1, sitting under Record Documents, tells the designer to put the revision cloud on
Z-ANNO-REVC and the revision symbols, the notes and the record-drawing stamp on Z-ANNO-REVS. The
A/E/C CAD Standard R6.2 Appendix A layer table assigns the revision cloud to G-ANNO-REVC and the
revision indicators and text to G-ANNO-REVS.

The order of precedence is fixed by FC 1-300-09N 12-3.3.4: the requirements of that chapter come
first, then the latest DoD A/E/C CAD and Graphics Standards. Read alone, that would give the Z-ANNO
pair the win, and that is exactly the trap. The two clauses do not in fact collide, because
12-3.6.1 is a record-drawing requirement: it governs sheets prepared as record documents that
carry as-built conditions after construction. Neither package here is a record drawing - the only
record prints in the file are the two FY23 sheets from the terminated contract - so the Z-ANNO pair
does not apply to this issue. What applies is the A/E/C CAD Standard's assignment, and that is what
12-3.3.4 selects for a non-record drawing issue.

The layers used are therefore G-ANNO-REVC for every revision cloud and G-ANNO-REVS for the
revision indicators, the revision text and the Revision (Issue) block entries. No revision graphic
in this issue is drawn on a Z-ANNO layer.

## 3. Issue state of BP-3

BP-2 was advertised on 6 July 2026 and opens bids on 28 September 2026, so its corrections can only
be issued as an amendment to the solicitation; that is the sole admissible state and change 0003 is
assigned in the form FC 1-300-09N requires (18-3).

BP-3 is different: it has been certified, it has not been advertised, and the Contracting Officer
offers the Designer of Record two lawful courses - hold the advertisement and re-issue the corrected
design, or advertise on schedule and amend the solicitation afterwards. The dates come out of the
contract documents: the schedule export records a scheduled advertisement of 12 October 2026 and a
bid opening of 12 November 2026, while the later dated memorandum of 26 August 2026 records the
advertisement as scheduled for 9 November 2026. Where the two conflict the memorandum controls.

ER 415-1-11 gives more than one duration, and which one binds follows from the issue state chosen.
Under 6.e the final design review is completed at least 30 calendar days before advertisement; under
6.g the backcheck is started 30 calendar days before advertising with all comments resolved before
bid opening; and 6.e also allows about 15 calendar days for reviewers to backcheck their comments.
So a correction folded into the design before advertisement is measured against a 30 calendar day
window ending on the advertisement date, while a correction issued as a change to the solicitation
afterwards is measured against a 15 calendar day backcheck window ending on the bid opening date.

Production starts 2026-08-31, runs 5 production days for 17 sheets, adds 6 days for two bundles and
1 day for the non-concurrence, and finishes 2026-09-12. Against the 30 day window the deadline is
10 October 2026 on the memorandum date and 12 September 2026 on the earlier export; on either
reading the finish is not later than the deadline, so the answer is 30 calendar days, zero delay
days and zero extension charge. The amendment course would also lose no days - 2026-09-12 against a
15 day deadline of 28 October 2026 - but it carries a revision delta on 17 sheets ($4,080), an
overflow charge ($1,560) and a change number with its continuation sheet, for $51,693.20 against
$46,053.20. The corrections are therefore incorporated before advertisement, issue state
PRE_ADVERTISEMENT_REISSUE, measured against the 30 calendar day window of ER 415-1-11 6.e and
6.g, and no change number or continuation sheet is produced for BP-3.

## 4. Revision (Issue) block overflow, BP-2

The Revision (Issue) block has eight rows. Of the fourteen BP-2 sheets re-issued here, six have
all eight rows in use (C-101, M-101, M-501, S-101, S-102, S-502), so amendment 3 is a ninth entry on
those six and the block overflows. The graphics standard (7.3) offers two ways out and requires the
choice to be the same on every sheet of the set.

Option 1 replaces the oldest row on every full block and is charged at $520 per sheet: 6 x $520 =
$3,120. Option 2 lets the new entry spill above the eighth row and is charged at $880 per sheet,
but only where the block is full and the sheet also carries three or more drawing blocks - a sheet
with a single drawing block has unoccupied space to spill into and costs nothing: of the six full
sheets only M-501 and S-502 are that dense, so 2 x $880 = $1,760. Option 2 was taken, saving
$1,360, and it is applied to the whole set rather than sheet by sheet: every row of
revision_log.csv carries option 2, because a mixed set would re-lay out one drawing area and not
the next and the standard does not permit the scheme to vary within a set. The overflow charge is
therefore $1,760 and not $3,120.

## 5. Dispositions, closure and packaging

Of the 34 review comments, 22 are valid and 12 cannot direct a change: two sit on superseded prints,
two carry no timestamp, two come from an author who is not on the roster for this work order, four
come from a reviewer who is not registered in the professional field of the sheet reviewed (FAR
52.236-25), and two were closed in the 60% review as elevated. Each is closed and recorded with its
ground. Of the 22 valid comments, 20 are incorporated and one per package is answered with a
documented non-concurrence: both are drafting preferences offered under ER 1165-2-217 5.8.5, which
is where the review policy keeps preferential comments from being directive, and neither carries a
criteria basis that the detail fails.

A change propagates at drawing-block level, not at sheet level. A callout bubble is drawn inside
one drawing block and refers to a drawing block on another sheet, so the sheets that must be
re-issued are the blocks that reach the changed block through that graph, and the graph crosses
disciplines: EOR-2-004 starts at S-502 and reaches eight sheets across four disciplines. The bubble
set was read off the sheets themselves; nothing in the corpus tabulates it.

Delivery: a drawing PDF may not exceed 50 MB or 50 sheets (12-3.5.1) and discipline sets are kept
whole (12-3.5.2.1). BP-2 needs three bundles and BP-3 two. Title-block defects are reported only for
the sheets actually re-issued - an electrical seal on a mechanical sheet, a checker who is also the
drafter, a checker who is also the designer - and read from the title blocks of those sheets.

Every figure in the ledger is the recomputation at the rates of the A-E production and reprographics
rate schedule of 26 August 2026: 4800 per bundle, 1150 per re-issued sheet, 26 per square inch of
incorporated markup, 6400 per added sheet, 240 per revised sheet, 520 or 880 for an overflow, 2400
per non-concurrence, 4 sheets per production day, 3 days per bundle and 1 day per non-concurrence.
The extension charge, where a course of action moves a date, is 2600 per calendar day plus 140 per
calendar day squared, per the Contracting Officer's memorandum.

```


## deliverables/disposition_register.csv

```text
PACKAGE,COMMENT_ID,AUTHOR,SIGNIFICANCE,VALIDITY,VOID_REASON,RESPONSE,RESOLUTION,PRIMARY_SHEET,PRIMARY_VIEW,CHANGE_KIND,SHEETS_CHANGED,SHEETS_IMPACTED,CLOSURE
BP-2,EOR-2-001,R. Ostrander,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,S-101,1,ADDITION,S-101,,INCORPORATED
BP-2,EOR-2-004,R. Ostrander,CRITICAL,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,S-502,A,DELETION,S-502,E-101;E-501;M-101;M-401;M-501;M-502;S-102;S-301,INCORPORATED
BP-2,EOR-2-006,D. Fenwick,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,M-101,1,ADDITION,M-101,,INCORPORATED
BP-2,EOR-2-009,P. Okonjo,MEDIUM,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,C-101,1,ADDITION,C-101,,INCORPORATED
BP-2,EOR-2-011,P. Okonjo,LOW,VALID,,NON_CONCUR,"Reviewer drafting preference with no criteria basis; the detail as drawn is adequate and correct, so the relocation is not adopted. This is the documented basis for non-concurrence.",C-501,A,NONE,,,NON_CONCUR_DOCUMENTED
BP-2,EOR-2-013,L. Marchetti,MEDIUM,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,E-001,1,CLARIFICATION,E-001,,INCORPORATED
BP-2,EOR-2-015,M. Delacroix,HIGH,VOID,NOT_REGISTERED_IN_FIELD,NOT_APPLICABLE,"The reviewer is not registered in the professional field of the sheet, so under FAR 52.236-25 the comment cannot direct that design.",M-301,1,NONE,,,VOID
BP-2,EOR-2-017,G. Halvorsen,MEDIUM,VOID,AUTHOR_NOT_ON_ROSTER,NOT_APPLICABLE,"The author is not on the review roster, so the markup is not an EOR review comment.",S-201,1,NONE,,,VOID
BP-2,EOR-2-019,J. Vandermeer,MEDIUM,VOID,NO_TIMESTAMP,NOT_APPLICABLE,"The annotation carries no timestamp, so it is not a review comment under the review policy.",S-501,C,NONE,,,VOID
BP-2,EOR-2-021,R. Ostrander,HIGH,VOID,SUPERSEDED_SHEET,NOT_APPLICABLE,"Markup is carried on a superseded print, not on the current issue, so it cannot direct a change to the issued set.",S-102,1,NONE,,,VOID
BP-2,EOR-2-023,D. Fenwick,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,M-401,1,ADDITION,M-401;M-502A,E-101;M-101,INCORPORATED
BP-2,EOR-2-025,R. Ostrander,CRITICAL,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,S-101,1,ADDITION,S-101;S-201A,,INCORPORATED
BP-2,EOR-2-027,K. Aoyama,LOW,VOID,NOT_REGISTERED_IN_FIELD,NOT_APPLICABLE,"The reviewer is not registered in the professional field of the sheet, so under FAR 52.236-25 the comment cannot direct that design.",M-501,A,NONE,,,VOID
BP-2,EOR-2-029,L. Marchetti,MEDIUM,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,E-101,1,CLARIFICATION,E-101,,INCORPORATED
BP-2,EOR-2-031,D. Fenwick,LOW,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,M-301,2,CLARIFICATION,M-301,,INCORPORATED
BP-2,EOR-2-033,J. Vandermeer,MEDIUM,VOID,CLOSED_WITH_ELEVATION,NOT_APPLICABLE,Closed in the 60% review with the notation that it had been elevated; that closure was never reopened.,S-002,1,NONE,,,CLOSED_ELEVATED
BP-3,EOR-3-002,H. Sandoval,CRITICAL,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,A-501,A,ADDITION,A-501,A-101;A-301,INCORPORATED
BP-3,EOR-3-004,J. Vandermeer,CRITICAL,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,S-501,A,DELETION,S-501,A-101;A-301;A-501;E-102;E-501;S-101,INCORPORATED
BP-3,EOR-3-006,H. Sandoval,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,A-502,A,ADDITION,A-502,A-102,INCORPORATED
BP-3,EOR-3-008,D. Fenwick,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,M-101,1,ADDITION,M-101,,INCORPORATED
BP-3,EOR-3-010,L. Marchetti,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,E-501,A,ADDITION,E-501,E-102,INCORPORATED
BP-3,EOR-3-012,B. Threlkeld,MEDIUM,VOID,SUPERSEDED_SHEET,NOT_APPLICABLE,"Markup is carried on a superseded print, not on the current issue, so it cannot direct a change to the issued set.",T-501,A,NONE,,,VOID
BP-3,EOR-3-014,H. Sandoval,MEDIUM,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,A-001,2,ADDITION,A-001,,INCORPORATED
BP-3,EOR-3-016,M. Delacroix,HIGH,VOID,NOT_REGISTERED_IN_FIELD,NOT_APPLICABLE,"The reviewer is not registered in the professional field of the sheet, so under FAR 52.236-25 the comment cannot direct that design.",A-501,B,NONE,,,VOID
BP-3,EOR-3-018,P. Okonjo,MEDIUM,VOID,NOT_REGISTERED_IN_FIELD,NOT_APPLICABLE,"The reviewer is not registered in the professional field of the sheet, so under FAR 52.236-25 the comment cannot direct that design.",A-502,B,NONE,,,VOID
BP-3,EOR-3-020,J. Vandermeer,MEDIUM,VOID,NO_TIMESTAMP,NOT_APPLICABLE,"The annotation carries no timestamp, so it is not a review comment under the review policy.",S-501,B,NONE,,,VOID
BP-3,EOR-3-022,C. Ballantyne,MEDIUM,VOID,AUTHOR_NOT_ON_ROSTER,NOT_APPLICABLE,"The author is not on the review roster, so the markup is not an EOR review comment.",E-102,1,NONE,,,VOID
BP-3,EOR-3-024,H. Sandoval,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,A-101,1,ADDITION,A-101;A-402,,INCORPORATED
BP-3,EOR-3-026,D. Fenwick,HIGH,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,M-101,1,ADDITION,M-101;M-502,,INCORPORATED
BP-3,EOR-3-028,H. Sandoval,LOW,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,A-201,1,CLARIFICATION,A-201,A-101,INCORPORATED
BP-3,EOR-3-030,J. Vandermeer,MEDIUM,VALID,,NON_CONCUR,"Reviewer drafting preference with no criteria basis; the detail as drawn is adequate and correct, so the relocation is not adopted. This is the documented basis for non-concurrence.",S-301,1,NONE,,,NON_CONCUR_DOCUMENTED
BP-3,EOR-3-032,T. Brightwater,LOW,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,M-301,1,CLARIFICATION,M-301,M-101,INCORPORATED
BP-3,EOR-3-034,B. Threlkeld,LOW,VALID,,CONCUR,Incorporated in the issued revision of the sheets listed.,T-501,B,CLARIFICATION,T-501,,INCORPORATED
BP-3,EOR-3-036,H. Sandoval,MEDIUM,VOID,CLOSED_WITH_ELEVATION,NOT_APPLICABLE,Closed in the 60% review with the notation that it had been elevated; that closure was never reopened.,A-301,2,NONE,,,CLOSED_ELEVATED

```


## deliverables/issue_plan.csv

```text
PACKAGE,BUNDLE_ID,BUNDLE_FILE,SHEET_NUMBER,DISCIPLINE_DESIGNATOR,PLOT_PDF_MB,BUNDLE_SEQ
BP-2,1,74213_BP-2_B1_Amend0003.pdf,C-101,C-,12.5,1
BP-2,1,74213_BP-2_B1_Amend0003.pdf,E-001,E-,4.4,2
BP-2,1,74213_BP-2_B1_Amend0003.pdf,E-101,E-,10.9,3
BP-2,1,74213_BP-2_B1_Amend0003.pdf,E-501,E-,4.8,4
BP-2,1,74213_BP-2_B1_Amend0003.pdf,GI-001,GI,2.6,5
BP-2,2,74213_BP-2_B2_Amend0003.pdf,M-101,M-,13.6,1
BP-2,2,74213_BP-2_B2_Amend0003.pdf,M-301,M-,7.5,2
BP-2,2,74213_BP-2_B2_Amend0003.pdf,M-401,M-,8.1,3
BP-2,2,74213_BP-2_B2_Amend0003.pdf,M-501,M-,6.1,4
BP-2,2,74213_BP-2_B2_Amend0003.pdf,M-502,M-,5.2,5
BP-2,2,74213_BP-2_B2_Amend0003.pdf,M-502A,M,5.5,6
BP-2,3,74213_BP-2_B3_Amend0003.pdf,S-101,S-,10.3,1
BP-2,3,74213_BP-2_B3_Amend0003.pdf,S-102,S-,9.4,2
BP-2,3,74213_BP-2_B3_Amend0003.pdf,S-201A,S,5.5,3
BP-2,3,74213_BP-2_B3_Amend0003.pdf,S-301,S-,7.2,4
BP-2,3,74213_BP-2_B3_Amend0003.pdf,S-502,S-,5.9,5
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-001,A-,2.8,1
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-101,A-,9.3,2
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-102,A-,7.0,3
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-201,A-,5.4,4
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-301,A-,5.2,5
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-402,A,5.5,6
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-501,A-,4.7,7
BP-3,1,74213_BP-3_B1_Reissue.pdf,A-502,A-,3.9,8
BP-3,1,74213_BP-3_B1_Reissue.pdf,GI-001,GI,1.7,9
BP-3,2,74213_BP-3_B2_Reissue.pdf,E-102,E-,6.8,1
BP-3,2,74213_BP-3_B2_Reissue.pdf,E-501,E-,3.7,2
BP-3,2,74213_BP-3_B2_Reissue.pdf,M-101,M-,8.7,3
BP-3,2,74213_BP-3_B2_Reissue.pdf,M-301,M-,5.1,4
BP-3,2,74213_BP-3_B2_Reissue.pdf,M-502,M,5.5,5
BP-3,2,74213_BP-3_B2_Reissue.pdf,S-101,S-,8.2,6
BP-3,2,74213_BP-3_B2_Reissue.pdf,S-501,S-,4.5,7
BP-3,2,74213_BP-3_B2_Reissue.pdf,T-501,T-,3.2,8

```


## deliverables/revision_log.csv

```text
PACKAGE,SHEET_NUMBER,MARK,DESCRIPTION,DATE,ROW_POSITION,OVERFLOW_OPTION
BP-2,C-101,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,9,2
BP-2,E-001,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,3,2
BP-2,E-101,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,4,2
BP-2,E-501,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,3,2
BP-2,GI-001,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,3,2
BP-2,M-101,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,9,2
BP-2,M-301,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,4,2
BP-2,M-401,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,3,2
BP-2,M-501,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,9,2
BP-2,M-502,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,3,2
BP-2,M-502A,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,1,2
BP-2,S-101,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,9,2
BP-2,S-102,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,9,2
BP-2,S-201A,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,1,2
BP-2,S-301,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,4,2
BP-2,S-502,3,EOR DESIGN REVIEW CORRECTIONS,SEP 2026,9,2
BP-3,A-001,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,A-101,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,A-102,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,A-201,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,A-301,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,A-402,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,A-501,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,A-502,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,E-102,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,E-501,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,GI-001,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,M-101,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,M-301,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,M-502,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,S-101,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,S-501,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE
BP-3,T-501,NONE,NO REVISION ENTRY - PRE-ADVERTISEMENT REISSUE,N/A,,NONE

```


## deliverables/sheet_index.csv

```text
PACKAGE,INDEX_ORDER,SHEET_NUMBER,SHEET_NAME,CURRENT_REVISION,ISSUE_ACTION,IN_THIS_ISSUE
BP-2,1,G-001,COVER SHEET,2,UNCHANGED,NO
BP-2,2,GI-002,"GENERAL NOTES, ABBREVIATIONS AND LEGEND",2,UNCHANGED,NO
BP-2,3,GI-001,DRAWING INDEX,0003,REPLACE,YES
BP-2,4,C-101,SITE UTILITY PLAN,0003,REPLACE,YES
BP-2,5,C-102,SITE GRADING AND DRAINAGE PLAN,2,UNCHANGED,NO
BP-2,6,C-301,SITE UTILITY SECTIONS,2,UNCHANGED,NO
BP-2,7,C-501,CIVIL DETAILS,2,UNCHANGED,NO
BP-2,8,S-001,STRUCTURAL SYMBOLS AND ABBREVIATIONS,1,UNCHANGED,NO
BP-2,9,S-002,STRUCTURAL GENERAL NOTES,1,UNCHANGED,NO
BP-2,10,S-101,CUP FOUNDATION PLAN,0003,REPLACE,YES
BP-2,11,S-102,CUP ROOF FRAMING PLAN,0003,REPLACE,YES
BP-2,12,S-201,CUP EXTERIOR ELEVATIONS,2,UNCHANGED,NO
BP-2,13,S-201A,BRACED BAY FRAMING ELEVATIONS,0003,ADD,YES
BP-2,14,S-301,CUP BUILDING SECTIONS,0003,REPLACE,YES
BP-2,15,S-501,FOUNDATION DETAILS,2,UNCHANGED,NO
BP-2,16,S-502,FRAMING DETAILS,0003,REPLACE,YES
BP-2,17,M-003,MECHANICAL GENERAL NOTES AND LEGEND,1,UNCHANGED,NO
BP-2,18,M-001,MECHANICAL EQUIPMENT SCHEDULES,2,UNCHANGED,NO
BP-2,19,M-002,CHILLED WATER PUMP SCHEDULE,2,UNCHANGED,NO
BP-2,20,M-101,CUP CHILLED WATER PLAN,0003,REPLACE,YES
BP-2,21,M-102,CUP CONDENSER WATER PLAN,2,UNCHANGED,NO
BP-2,22,M-301,CHILLED WATER PLANT SECTIONS,0003,REPLACE,YES
BP-2,23,M-401,CHILLER BAY ENLARGED PLAN,0003,REPLACE,YES
BP-2,24,M-501,PIPING DETAILS,0003,REPLACE,YES
BP-2,25,M-502,EQUIPMENT ANCHORAGE DETAILS,0003,REPLACE,YES
BP-2,26,M-502A,CHILLER PAD ANCHORAGE DETAILS,0003,ADD,YES
BP-2,27,E-004,ELECTRICAL SYMBOLS AND ABBREVIATIONS LEGEND,1,UNCHANGED,NO
BP-2,28,E-001,ELECTRICAL PANEL SCHEDULES,0003,REPLACE,YES
BP-2,29,E-002,ELECTRICAL LOAD SUMMARY,2,UNCHANGED,NO
BP-2,30,E-101,CUP POWER PLAN,0003,REPLACE,YES
BP-2,31,E-501,ELECTRICAL DETAILS,0003,REPLACE,YES
BP-3,1,G-001,COVER SHEET,,UNCHANGED,NO
BP-3,2,GI-001,DRAWING INDEX,,REPLACE,YES
BP-3,3,S-101,DATA HALL 210 FOUNDATION PLAN,A,REPLACE,YES
BP-3,4,S-102,DATA HALL 210 ROOF FRAMING PLAN,A,UNCHANGED,NO
BP-3,5,S-301,DATA HALL 210 FRAMING SECTIONS,,UNCHANGED,NO
BP-3,6,S-501,STRUCTURAL DETAILS,,REPLACE,YES
BP-3,7,A-002,ARCHITECTURAL GENERAL NOTES AND ABBREVIATIONS,H,UNCHANGED,NO
BP-3,8,A-001,PARTITION TYPES AND KEY PLAN,H,REPLACE,YES
BP-3,9,A-101,DATA HALL 210 FLOOR PLAN,H,REPLACE,YES
BP-3,10,A-102,DATA HALL 210 ROOF PLAN,A,REPLACE,YES
BP-3,11,A-201,EXTERIOR ELEVATIONS,A,REPLACE,YES
BP-3,12,A-301,BUILDING SECTIONS,H,REPLACE,YES
BP-3,13,A-401,ELECTRICAL ROOM ENLARGED PLAN,,UNCHANGED,NO
BP-3,14,A-402,TELECOM ENTRANCE ROOM ENLARGED PLAN,,ADD,YES
BP-3,15,A-501,WALL SECTIONS AND DETAILS,A,REPLACE,YES
BP-3,16,A-502,ROOF DETAILS,A,REPLACE,YES
BP-3,17,M-101,DATA HALL 210 CRAH PLAN,,REPLACE,YES
BP-3,18,M-301,AIR DISTRIBUTION SECTIONS,,REPLACE,YES
BP-3,19,M-501,MECHANICAL DETAILS,,UNCHANGED,NO
BP-3,20,M-502,CONTAINMENT WALL DUCT PENETRATION DETAILS,,ADD,YES
BP-3,21,E-101,DATA HALL 210 POWER PLAN,,UNCHANGED,NO
BP-3,22,E-102,DATA HALL 210 LIGHTING PLAN,,REPLACE,YES
BP-3,23,E-501,ELECTRICAL DETAILS,,REPLACE,YES
BP-3,24,T-101,DATA HALL 210 TELECOM PATHWAY PLAN,,UNCHANGED,NO
BP-3,25,T-501,TELECOM DETAILS,,REPLACE,YES

```
