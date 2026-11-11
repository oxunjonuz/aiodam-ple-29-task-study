# footing-settlement-verification

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-22 — footing-settlement-verification
   The settlement calculation stands on a footing elevation the drawings do not have — and the consolidation it
   omits is 5.4 in on its own.

 Judge score                                                NOT MEASURED — no judge endpoint or key

 Criteria · weight                                          17 criteria · 62 weight points

 My criteria reader (not a score)                           17 of 17 criteria + 5 closure checks · 62 of 62 weight

 Breakage campaign                                          20/20

 File                                                       /work/foot/out/Thelbridge_Verification_Memo_Item051_BlockC.docx · 17,638 B
                                                            · sha256 54fd2aa41d8471f9…


TASK AND INPUTS
Verify the settlement of Item 051 Block C and write the memo. Seventeen criteria, weight 62.

The geotechnical report, the foundation details, the settlement calculation CB-GEO-007, the survey, the monitoring rounds and the
laboratory results.

WHAT WAS AMBIGUOUS OR BROKEN
    CB-GEO-007 stands on footing El. 740.5, while Detail 3/S-201 Rev.4 gives El. 746.0 (5.5 ft higher), DBM Rev.2 §9 says “4,000 psf
    at El. 748 ± 2 ft”, and S-101 gives no elevation at all.
    Depth to mid-clay is 11.5 ft against 17.0 ft, and Δσ is 900 against 571 psf — so the calculation is held and the 0.90 in is not
    confirmed.
    Only immediate settlement is computed (Sheet 4 says “Not computed” for consolidation). By the record's own parameters: 0.28 ×
    216 / 1.92 × log10(2750/1850) = 5.42 in; total 6.32 in against a 1.0 in allowance.
    Art. 3.6.B is triggered: the load is already applied and no notice was given — the memo itself is the notice.
    Round 15 gives 0.61 in at MP-7, 61% of the allowance, and accelerating (0.03 → 0.07 in per round); the distribution omits the SER.
    The calculation cites “S-101 Rev.2 (09/12/2025)” while the revision block issues Rev.2 on 12/12/2025; LL 34–39 against laboratory
    40 and 41; PI 15–19 against 21 on all 25 CL samples.

WHAT I DID AND WHAT CAME OUT
    Disposition NOT VERIFIED, with the consolidation analysis and the elevation reconciliation assigned to the SER, indices to
    Kelbrook, monitoring transfer to Rowan Bay, and the hold point of 22/07/2026 moved.
    A side note to Item 052: the readiness letter cites A615, while Spec 03 20 00 Art 2.1 and S-201 forbid A615 and require A706.
    17 of 17 criteria plus 5 closure checks, 62 of 62 weight by my reader on the PDF twin.



                                                                 page 33 of 48
PLE 29-task report · AIODAM                                                                                                 11 November 2026




HOW IT WAS VERIFIED
    Reader on the PDF twin; 63 number tokens, 0 unexplainable; 52 of 52 citations checked against their source; 20 breakages — all
    noticed.
    The official test.sh chain with a stub judge: rendering, staging, 17 indices and a reward.json, with GRADER_FAILED empty.

MY MISTAKES IN THIS TASK
    The first mutation run gave 2 of 15 — and that was true of the document, not of the reader: every fact stands two to six times. One
    genuine reader failure was found (c13 accepted “the Contractor is the party required to produce it” because the SER stood in another
    sentence); the reader now requires the obligation in one sentence and not the Contractor.
    cite_check.py first reported 44 of 52; all eight were my own typos inside quotations, not errors of the memo.

WHAT REMAINS UNVERIFIED
No judge score.

USE OF REFERENCE OR GRADER MATERIAL
None.


PART C · RUBRIC · LANDSCAPE / PLANNING
```


## deliverables/Thelbridge_Verification_Memo_Item051_BlockC.txt

```text
THELBRIDGE PROJECT MANAGEMENT SERVICES

500 South High Street, Suite 720, Columbus, OH 43215  |  (614) 555-0125  |  thelbridgepm.com

VERIFICATION MEMORANDUM

To:	Adrian Vance, Project Engineer (Structural / Geotechnical), Owner's Representative — Thelbridge Project Management Services

From:Rahul Nimkar, Geotechnical Reviewer — Thelbridge Project Management Services

Date:07/17/2026

Subject:Item 051 — Block C Spread-Footing Long-Term Settlement — Verification Memorandum (Package A, Birchwood Village Expansion Program)

Copy:Rachel Marlbeck, University Program Director, Thornbury State University, Office of Capital Programs

1.  PURPOSE

This memorandum addresses Item 051 of the Thelbridge Review Comment and Verification Register — Package A (Discipline: Geotechnical; raised by R. Nimkar 07/10/2026; verification due 07/20/2026). Item 051 requires the Block C spread-footing foundations to be verified for long-term settlement over the compressible CL clay aquitard; that the settlement analysis be complete and reconciled against the settlement tolerances of Specification Section 31 20 00 Article 3.6 (1.0 in total, 0.5 in differential); and that the calculation be correlated with the monitoring reading of 0.61 in cumulative settlement recorded on 06/26/2026.

What is determined here is whether the project record supports a verification of Block C long-term settlement; and, because it does not, what has to be produced before the item can be closed, who is required to produce it, and what the item's disposition means for the 07/22/2026 Level-3 hold-point.

Determination in brief. Item 051 is NOT VERIFIED. The calculation in the record computes the immediate settlement only; the consolidation settlement of the clay, which the geotechnical report identifies as the governing mechanism and places on the Structural Engineer of Record, was never computed; the stress increase the calculation rests on cannot be traced to a settled founding elevation; and on the parameters in the record the total settlement at the governing footing exceeds the 1.0 in tolerance by a factor of four to six. The Level-3 placement scheduled for 0530 on 07/22/2026 must itself be delayed or its hold-point moved until Item 051 is verified.

2.  DOCUMENTS REVIEWED

Each document below was read in the form issued, at the revision and date stated.

1.	Thelbridge Review Comment and Verification Register — Package A, workbook last updated 07/15/2026 — Item 051, the REGISTER, REVIEWER ASSIGNMENTS and SCHEDULE tabs.

2.	Geotechnical Investigation Report, Kelbrook Geotechnical, Inc., Report No. KGI-2024-0417 Rev.2, 02/14/2025 — Executive Summary recommendation 1; §5.1, §5.2, §6.3, §8.1, §9.1.

3.	Laboratory Consolidation Test Report, Vantree Materials Testing, LLC, Report No. VMT-2025-0109-CON, 01/09/2025 (ASTM D2435; samples B-06 U3 and B-08 U2).

4.	Geotechnical Laboratory Index Test Data, Vantree Materials Testing, LLC, Report No. VMT-2024-0917-PkgA-INDEX, 01/09/2025 — INDEX TESTS and SUMMARY sheets.

5.	Design Calculation CB-GEO-007, "Block C Spread Footing Settlement", Calloway Braddock Structural Engineers, LLC, Rev.0 (issued), 09/30/2025 — Sheets 1 to 5.

6.	Sheet S-101, "Foundation Plan — Blocks A/B/C", Calloway Braddock, Rev.4, 05/29/2026, and its revision block.

7.	Sheet S-201, "Foundation Sections and Details", Calloway Braddock, Rev.4, 05/29/2026 — Detail 3/S-201 (typical spread footing, Block C) and the revision block.

8.	Design Basis Memorandum CB-DBM-001 Rev.2, Calloway Braddock, 11/14/2025 — §7.3 (shallow foundations, Block C) and §9 (geotechnical design parameters summary).

9.	Specification Section 31 20 00, "Earth Moving", Rev.3, 03/06/2026 — Article 3.6, Settlement Tolerance.

10.	Settlement Monitoring Records — Block C, workbook SM-BLKC-2026-Rev14, Rowan Bay Constructors, Survey Crew 3 — rounds 1 to 15, baseline 12/12/2025, latest reading 06/26/2026 (Monitoring Points, Readings, Trend Chart and Notes sheets).

11.	Field Observation Report — Excavation and Groundwater, Kelbrook Geotechnical, Inc., Report No. KGI-FO-2026-0528 — §2.2, strata exposed in the cut face.

12.	Owner directive — R. Marlbeck (Thornbury State University) to A. Vance, e-mail of 07/06/2026, "Package A — Written Verification Required Before Permanent Work — Level-3 Pour and Related Items".

13.	Contractor notification — S. Kendrow (Rowan Bay Constructors) to A. Vance, e-mail of 07/14/2026, "Package A — Level-3 Pour Readiness Notification — Placement 0530 on 07/22/2026".

14.	Memorandum form, Thelbridge Project Management Services, template v1 — sections retained as issued.

3.  BASIS OF REVIEW

3.1  Contractual requirements

(a)	Specification Section 31 20 00 Rev.3, Article 3.6.A — total settlement 1.0 in maximum; differential settlement 0.5 in maximum over 30 ft.

(b)	Article 3.6.B — where predicted or monitored settlements exceed the tolerances of Paragraph A, the Structural Engineer of Record shall be notified in writing and no additional load shall be applied until a written disposition is issued.

(c)	Article 3.6.C — monitoring readings shall be taken and transmitted to the Owner's Representative Engineer at the frequency indicated, and cumulative settlement approaching either tolerance of Paragraph A shall be reported immediately.

(d)	Register conventions and SCHEDULE tab (07/15/2026) — hold-point releases are issued in writing by Adrian Vance; no permanent work may proceed past a hold-point without a Thelbridge written verification; the 07/22/2026 Level-3 placement is the next contractual hold-point, conditioned on closure of Items 052, 053 and 057 at minimum, with all ten open items carrying verifications due 07/20/2026 in advance of it.

(e)	Owner directive of 07/06/2026 — no permanent work on Package A proceeds until Thelbridge has issued a written verification; the list to be closed by 07/20/2026 includes Block C long-term settlement; if any item cannot be verified in time the Owner would rather delay the pour than release non-conforming work, and the 07/22/2026 hold-point is either released or moved.

3.2  Geotechnical requirements

(f)	Geotechnical Investigation Report Rev.2, Executive Summary recommendation 1 and §9.1 — consolidation of the clay under the column-load stress increase at mid-depth of the clay must be explicitly computed by the Structural Engineer of Record, using the parameters provided in the report, and the computed settlement must be reconciled against the total-and-differential settlement tolerances specified in the project structural criteria.

(g)	Geotechnical Investigation Report Rev.2, §8.1 — one-dimensional consolidation settlement Sc = Cc · H / (1 + e0) · log10[(σ'0 + q) / σ'0], with Cc = 0.28, e0 = 0.92, H = 18 ft and σ'0 = 1,850 psf at mid-depth; because σ'p = 1,900 psf exceeds σ'0 by only about 50 psf, any stress increase of engineering interest is virgin compression on the Cc slope.

(h)	Design Basis Memorandum CB-DBM-001 Rev.2 §7.3 — settlement of the clay layer is a serviceability consideration checked in CB-GEO-007 under service-level combinations against the Article 3.6 tolerances (total 1.0 in, differential 0.5 in).

3.3  Standard of review applied

(i) completeness — whether the analysis the geotechnical report requires is in the record at all; (ii) traceability — whether every input the calculation uses can be traced to an issued document, revision and date; (iii) internal consistency — the calculation against the drawings and against the design basis; (iv) reconciliation — the computation against the project tolerances and against the monitoring record. A document that restates a requirement does not satisfy it; the requirement is satisfied only where the record contains the work.

4.  FINDINGS

Finding 1 — The computed settlement turns on which of two founding elevations on the record governs, and the record does not settle it. CB-GEO-007 Sheet 2 takes the underside of the footing at El. 740.5 ft, and describes the Block C footings as "isolated square spread footings on Stratum 1 sand, ~2.5 ft above top of clay" (top of Stratum 2 at El. 738.0 ft). Detail 3/S-201 Rev.4, issued 05/29/2026, dimensions the typical Block C spread footing with the top of footing at El. 748.0 ft and a thickness of 2 ft 0 in, placing the underside at El. 746.0 ft — 5.5 ft above the elevation the calculation uses. Design Basis Memorandum CB-DBM-001 Rev.2 §9 records the allowable bearing pressure of 4,000 psf (net) "at El. 748 ± 2 ft", which brackets 746.0 ft and excludes 740.5 ft, and S-101 Rev.4 gives no bottom-of-footing elevation of its own, deferring it to the typical spread footing on S-201/3. Both elevations are physically possible on this profile — KGI-FO-2026-0528 §2.2 logs roughly 4 ft of topsoil and reworked material between El. 750.0 and El. 746.0 above the SP-SM sand — which is exactly why the record, and not the reviewer, has to decide between them.

The difference is quantitative, not formal. The depth from the footing base to mid-clay at El. 729.0 — the depth that generates the stress increase on which the whole calculation rests — is 11.5 ft on CB-GEO-007's basis and 17.0 ft on Detail 3/S-201. The 900 psf at mid-depth reproduces from CB-GEO-007's own basis: the governing footing F-C6 carries 432 kips service dead plus live over 10.5 ft × 10.5 ft, and a 2:1 spread to 11.5 ft gives 432,000 / (22.0 × 22.0) = 893 psf, which is the 900 psf of Sheet 2. On the 17.0 ft depth the same method gives 571 psf. The settlement computation is therefore held open: the figure is not verifiable until the founding elevation is reconciled, and this memorandum does not endorse the value computed in CB-GEO-007.

Finding 2 — What CB-GEO-007 computes is the immediate settlement; the consolidation settlement is expressly not computed. Sheet 3 works the Schmertmann–Hartman–Brown immediate (elastic) settlement in sublayers to a depth of 2B and reports a maximum of 0.90 in at F-C6 (Sheet 3, F34 and G50). Sheet 4, headed "Consolidation", carries every parameter as "—" and closes with "Not computed. Serviceability taken from the immediate settlement result on Sheet 3." The 0.90 in that Sheet 3 compares against the 1.0 in tolerance is thus the immediate component alone — the elastic settlement of the sand profile together with the undrained distortion of the clay within the influence zone — while the long-term consolidation of the 18 ft clay aquitard, which the geotechnical report identifies as the governing mechanism for Block C, is absent from the calculation. The "SATISFACTORY" conclusion on Sheet 3 is drawn against a total that is incomplete on the face of the same workbook.

Finding 3 — Primary consolidation of the clay, computed from the parameters in the record, is 5.42 in. The derivation follows the report's own §8.1 expression, with every input taken from the record: Cc = 0.28 (VMT-2025-0109-CON, Summary — Design Params), e0 = 0.92 (same), H = 18 ft = 216 in (GIR §6.3; CB-GEO-007 Sheet 5), σ'0 = 1,850 psf at mid-depth El. 729.0 (same), σ'p = 1,900 psf with OCR 1.03 (essentially normally consolidated), and Δσ = 900 psf at mid-depth (CB-GEO-007 Sheet 2, F12).

Sc = Cc · H / (1 + e0) · log10[(σ'0 + Δσ) / σ'0]

Sc = 0.28 × 216 in / (1 + 0.92) × log10[(1,850 + 900) / 1,850]

Sc = 31.50 in × log10(1.48649) = 31.50 in × 0.17216 = 5.42 in.

The reported primary consolidation settlement is therefore 5.42 in. Two refinements are noted and neither changes the conclusion. First, if the 50 psf band between σ'0 and σ'p is taken on the recompression index (Cr = 0.045) and only the remainder on Cc, the figure reduces to 5.12 in; 5.42 in is reported because that is the expression §8.1 prescribes for any q greater than 50 psf, and because the recompression band contributes 0.06 in of the difference. Second, the bounding case on the S-201/3 founding elevation, with Δσ ≈ 571 psf at 17.0 ft, gives 3.68 in. Both are reported as sensitivities; the governing figure on the calculation's own inputs is 5.42 in.

Finding 4 — Total settlement at the governing Block C footing exceeds the 1.0 in tolerance of Article 3.6.A. Adding the two components at the governing footing F-C6: 0.90 in immediate (CB-GEO-007 Sheet 3) + 5.42 in primary consolidation = 6.32 in against a total-settlement tolerance of 1.0 in, a utilisation of 6.3. On the S-201/3 founding elevation the total is 0.90 + 3.68 = 4.58 in, a utilisation of 4.6. The exceedance holds on either founding elevation; only its magnitude depends on the elevation that Finding 1 leaves open.

Finding 5 — Article 3.6.B is engaged, and no written notification or disposition is in the record. Predicted settlement exceeds the tolerances of Paragraph A by a factor of four to six, so the Structural Engineer of Record is to be notified in writing and no additional load is to be applied to Block C until a written disposition is issued. Loading of Block C is already well advanced — the monitoring record's loading notes put Level-1 column and Level-2 slab framing in 01/2026 to 02/2026, Level-3 framing and roof steel in 03/2026 to 05/2026, and the podium slab-on-grade in 06/2026 — and the register holds no Article 3.6.B notification and no disposition against Item 051. This memorandum is written notification to the Structural Engineer of Record under Article 3.6.B, and it is copied to the Owner.

Finding 6 — The Round 15 monitoring readings were routed to the Contractor's own staff. Round 15 (06/26/2026) records a maximum cumulative settlement of 0.61 in at MP-7 against the 1.0 in tolerance of Article 3.6.A — 61 per cent of it — with cumulative values spread from 0.31 in to 0.61 in across the twelve points. The field notes record the rate as not decreasing and not yet stabilised: at MP-7 the round-to-round increment has grown from 0.03 in per 14-day round over Rounds 1 to 3 to 0.07 in over Round 15, a rate that has risen from 0.0021 to 0.0050 in per day, and the trend annotation records that the slope is increasing over the observation window with downward (concave) curvature that has not entered a decelerating phase. That is cumulative settlement approaching a tolerance of Paragraph A, and Article 3.6.C requires it to be transmitted to the Owner's Representative Engineer and reported immediately. It was not: the transmittal line of the monitoring record names Marcus Whitfield, Superintendent, and Steven Kendrow, Project Executive — both Rowan Bay Constructors personnel, that is, the Contractor's own staff — with a copy to the Owner's Representative file, and the Structural Engineer of Record appears nowhere on the transmittal or in the round. Register Item 041 logged the reading on 06/27/2026 as a document receipt only. The reading that triggered the immediate reporting duty under Article 3.6.C never reached the Owner's Representative Engineer or the Structural Engineer of Record through the channel the Specification names.

Finding 7 — The monitored settlement is already consistent with a total well above the calculated 0.90 in. The 0.61 in observed on 06/26/2026 is 68 per cent of the entire settlement CB-GEO-007 predicts, with the structure still under erection, and the cycle is early: with cv = 0.042 ft²/day (CB-GEO-007 Sheet 5) and a 9 ft drainage path in the double-drained 18 ft layer, 30 per cent of primary consolidation is reached in about 137 days and 90 per cent in about 1,635 days, that is about 4.5 years. A monitored value already at 68 per cent of the predicted total within five months of first structural load is not consistent with a total of 0.90 in; it is consistent with the several-inch consolidation settlement Finding 3 computes. This is a corroboration of the order of magnitude, not a substitute for the missing analysis, which is why the Item 051 disposition does not rest on it.

Finding 8 — The calculation declares a layout basis that cannot be traced to an issued drawing. Sheet 2 of CB-GEO-007 states: "Column layout follows structural drawing S-101 Foundation Plan Rev.2 (09/12/2025); grid spacing 24 ft × 26 ft typ." The revision block of S-101 issues Rev.0 on 08/22/2025, Rev.1 on 10/03/2025, Rev.2 on 12/12/2025, Rev.3 on 03/06/2026 and Rev.4 on 05/29/2026. No revision of S-101 carries the date 09/12/2025; the calculation, dated 09/30/2025, cites a revision of a drawing that on its own cover sheet did not exist at that date, and the date it cites is the actual issue date of Rev.2 with the day and month transposed. The calculation therefore declares a basis that cannot be traced to an issued document, and the column layout it relies on cannot be confirmed against the issue of S-101 that governs the work.

Finding 9 — The laboratory index data place the liquid limit of the CL clay at 40 to 41, above the 34 to 39 the investigation report's narrative gives for the same clay. The Geotechnical Investigation Report Rev.2, §5.1, characterises the CL clay in narrative as LL = 34 to 39, PL = 17 to 20, PI = 15 to 19. The Vantree index workbook of the same date reports every CL sample taken in the investigation — twenty-five samples across all nine borings — at LL 40 (14 samples) and LL 41 (11 samples). The two ranges do not overlap at a single sample. Both documents are named in the Item 051 reference list, and the narrative range is the one a designer carries forward when judging compressibility; CB-GEO-007 Sheet 5 itself records "LL / PL / PI = 41 / 20 / 21 (B-06)", that is, the laboratory value rather than the report's. The discrepancy has not been reconciled by either author.

Finding 10 — The laboratory index data place the plasticity index of the CL clay at 21, above the 15 to 19 the investigation report's narrative gives for the same clay. Measured and reported separately from the liquid limit under ASTM D4318, and therefore recorded separately here: the narrative gives PI = 15 to 19, while every one of the twenty-five CL samples in the laboratory workbook returns PI = 21 (with PL of 19 to 20). The ranges do not overlap. Plasticity index is the property on which the report's own characterisation of the clay as of medium plasticity rests, and at PI = 21 the clay sits at the upper bound of the CL range; the parameter that governs the choice between Cc and Cr and the compressibility judgement generally is therefore stated differently in the two documents that carry it.

Finding 11 — Item 051 cannot be closed: the consolidation settlement analysis the report requires is not in the record, and the Structural Engineer of Record is the party required to produce it. Executive Summary recommendation 1, §8.1 and §9.1 of the Geotechnical Investigation Report place the computation of the stress increase at mid-depth of the clay for each governing footing, and its reconciliation against the total and differential settlement tolerances, on the Structural Engineer of Record. CB-GEO-007 Sheet 4 is blank on exactly that computation, so the analysis that Item 051 exists to verify is the analysis that is missing. Before Item 051 can be closed there must be produced: the stress increase at mid-depth of the clay for each governing Block C footing on the reconciled founding elevation; the one-dimensional consolidation settlement from Cc, e0, H and the effective stresses, with the OCR margin stated; and the reconciliation of the resulting total and differential settlements against the 1.0 in and 0.5 in tolerances, with the remedy adopted where they are exceeded (resized footings, a combined mat, or a deep-foundation alternative, as §9.1 sets out). That analysis is to be produced by Calloway Braddock Structural Engineers, LLC — Gregory Nash, PE, Structural Engineer of Record, the party the report obliges — sealed and issued as a revision to CB-GEO-007.

Finding 12 — Further observations recorded on the face of the documents. (a) CB-GEO-007 Sheet 5 cites the tested samples as "B-06 U3 (El. 727.5) and B-08 U2 (El. 731.0)", whereas GIR §4.2 and VMT-2025-0109-CON record them at El. 729.5 and El. 728.7; the parameters are not affected but the traceability of the input block is not clean. (b) The footing schedule in CB-GEO-007 (8.5 ft, 9.5 ft and 10.5 ft square footings) does not match the "typical interior" footing of Detail 3/S-201 (7 ft 6 in × 7 ft 6 in × 2 ft 0 in), while the as-built monitoring points record footings of 8.5 ft to 10.5 ft plan dimensions — so the drawing that supplies the only founding elevation in the record is itself not the as-built configuration. (c) The Contractor's pour-readiness notification of 07/14/2026 states that the Level-3 rebar transmittal confirms conformance to ASTM A615/A615M Grade 60, whereas Specification Section 03 20 00 Rev.3 Article 2.1 and General Note 2 of S-201 specify ASTM A706/A706M Grade 60 and prohibit A615 substitution; this falls outside Item 051 and is referred to Item 052, not decided here.

5.  CONCLUSION AND RECOMMENDATION

5.1  Determination. The Block C spread-footing foundations are not verified for long-term settlement. The settlement analysis in the record is incomplete (Finding 2), the stress increase it relies on rests on a founding elevation the record does not reconcile (Finding 1), the total settlement exceeds the 1.0 in tolerance of Specification 31 20 00 Article 3.6.A on either founding elevation (Findings 3 and 4), and the monitored settlement is already 61 per cent of that tolerance and accelerating (Findings 6 and 7).

5.2  Disposition recommended for the register. Item 051 is to be recorded as NOT VERIFIED, held open pending the consolidation settlement analysis and the founding-elevation reconciliation, with no verification date entered until the items in 5.3 are received and reviewed. The item is not approved and is not verified with comment; the settlement question is open.

5.3  What must be produced before Item 051 can be closed, and by whom.

(a)	The consolidation settlement analysis required by GIR Executive Summary recommendation 1, §8.1 and §9.1, computed for each governing footing and reconciled against the 1.0 in total and 0.5 in differential tolerances — to be produced by Calloway Braddock Structural Engineers, LLC (Gregory Nash, PE, Structural Engineer of Record), sealed, as a revision to CB-GEO-007.

(b)	Reconciliation of the Block C founding elevation — El. 740.5 ft in CB-GEO-007 Sheet 2 against El. 746.0 ft in Detail 3/S-201 Rev.4 and the El. 748 ± 2 ft bearing level in DBM Rev.2 §9 — with the governing elevation stated on both the calculation and the footing detail; and reissue of CB-GEO-007 with a layout basis that traces to an issued revision of S-101, the cited 09/12/2025 not being one (Finding 8). Produced by the Structural Engineer of Record.

(c)	Reconciliation of the CL index properties — narrative LL 34 to 39 and PI 15 to 19 against laboratory LL 40 to 41 and PI 21 — with a statement of which values govern the compressibility assessment. To be produced by Kelbrook Geotechnical, Inc., the geotechnical consultant of record, with Vantree Materials Testing as the laboratory.

(d)	Transmission of the Block C monitoring readings to the Owner's Representative Engineer under Article 3.6.C, with immediate reporting of cumulative settlement approaching either tolerance, and continuation of the rounds — to be effected by Rowan Bay Constructors (Marcus Whitfield, Superintendent), with the Structural Engineer of Record added to the distribution.

5.4  Consequence for the Level-3 placement of 07/22/2026. The Level-3 concrete placement scheduled for 0530 on 07/22/2026 must itself be delayed, or its hold-point moved, until Item 051 is verified. The Owner's directive of 07/06/2026 states that no permanent work on Package A proceeds until Thelbridge has issued a written verification, that the position applies equally to the Level-3 pour and to every other operation on the current list, that Block C long-term settlement is one of the items to be closed by 07/20/2026, and that if any item cannot be verified in time the Owner would rather delay the pour than release non-conforming work. The register says the same in its own terms: the 07/22/2026 placement is the next contractual hold-point, the schedule tab conditions it on closure of Items 052, 053 and 057 at minimum, the register summary records that all ten open items carry verifications due 07/20/2026 in advance of that hold-point, and the schedule conventions provide that no permanent work may proceed past a hold-point without a Thelbridge written verification. Item 051 is one of those ten items and is not verified. The Contractor's pour-readiness notification of 07/14/2026 asks for the release on four materials items — the Level-2 cylinder reports, the slab-on-grade compaction report SF-26-0708, rebar lot 2026-0710 and the pre-pour cover inspection — and does not mention Block C settlement at all; those items are Items 052 to 054 and 057 on the same register, they close nothing that Item 051 leaves open, and the phrase "at minimum" in the schedule tab is there because the materials items are the floor of the condition, not the whole of it. Whether Block C receives concrete in the 07/22/2026 placement is a question of physical scope; the condition the Owner set is contractual, and physical separation from the Block A/B pour does not answer it. The hold-point is accordingly to be moved, with the new date set once the analysis in 5.3(a) and the reconciliation in 5.3(b) are in hand, and the written status due to the Owner by end of day 07/20/2026 is to record Item 051 as not cleared.

5.5  Referral and closing. The rebar conformance point in Finding 12(c) is referred to Item 052 and is not decided here. Nothing in this memorandum relieves the Structural Engineer of Record of the analysis the geotechnical report requires, and nothing in it releases the 07/22/2026 hold-point. I am available to review the reconciliation as soon as it is issued; the settlement figures set out in Findings 3 and 4 are offered so that the reconciliation can be checked against them, not in place of it.

NameRahul Nimkar

TitleGeotechnical Reviewer

Thelbridge Project Management Services - Owner's Representative

Date07/17/2026
```
