# fire-test-validation-memo

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-21 — fire-test-validation-memo
   The run does not validate the design: the prediction is 13 minutes past a criterion whose own uncertainty is ±9
   minutes.

 Judge score                                                 NOT MEASURED — no judge endpoint or key

 Criteria · weight                                           48 criteria · 208 weight points

 My criteria reader (not a score)                            48 of 48 criteria · 208 of 208 weight

 Breakage campaign                                           12/12

 File                                                        /work/ple-solve/fire-test-validation-memo/output/CFDME-2025-
                                                             041_Validation_Memo_Section7.docx · 46,265 B · sha256 69daa3950ad8c928…


TASK AND INPUTS
Write Section 7 of the validation memo for a fire test: does the run validate the design? Forty-eight criteria, weight 208 — the largest
single weight in the set.

The test report, the certified assembly, the client's report, the validation procedure, and the simulation predictions.

WHAT WAS AMBIGUOUS OR BROKEN
    Not validated: E = +13 min against a criterion of 11.8 min, with u_val = ±9 min; the maximum passing prediction is 129.8 min.
    D = 118 min comes from the certified report; the client's 129 min is not accepted under §6 of the procedure.
    The run assembly equals the certificate assembly (50/205/305 mm, siliceous, 9-gauge 3.8 mm, 600 mm o.c., 25 mm) — so the
    comparison is like for like.

WHAT I DID AND WHAT CAME OUT
    A memo that states the determination, shows the uncertainty band, and explains why 129 min cannot replace 118 min.
    48 of 48 criteria and 208 of 208 weight by my reader; plus two structural checks.



                                                                  page 32 of 48
PLE 29-task report · AIODAM                                                                                              11 November 2026




HOW IT WAS VERIFIED
    Reader; 116 numbers and units traced to sources, 0 unexplainable.
    Twelve breakages through verify.py — all noticed (M01 0.9952, M06 0.9519, M10 0.9423).
    The world against tests/world.md5: 50 files, 0 changed, 0 extra.
    Wiring: verify.py + grade.py with an all-pass stub, a blind stub and the reader stub gave 1.0 / 0.0 / 1.0 — the path is alive and the
    weights arrive.

MY MISTAKES IN THIS TASK
    My error here was an expectation, not a number: I assumed a rubric task could be scored offline. It cannot. This task established the
    boundary that shaped the next fourteen.

WHAT REMAINS UNVERIFIED
No judge score — and with 208 weight this is the largest unmeasured quantity in the set. The docx was not rendered inside the verifier
image; this rubric has no separate rendering, so the judge reads the document itself.

USE OF REFERENCE OR GRADER MATERIAL

None.


PART C · RUBRIC · GEOTECHNICAL
```


## deliverables/CFDME-2025-041_Validation_Memo_Section7.txt

```text
THE CFD MECHANICAL ENGINEER, LLC  ·  Tucson, Arizona  ·  AZ PE #48213

PROJECT CFDME-2025-041 — Ocotillo Additive Structures, WUI Wildfire, Phase 1

Validation Memorandum — Section 7, Companion ASTM E119 Simulation

Memorandum no. CFDME-2025-041-VAL-001  ·  Issued 10/10/2025  ·  Prepared from the file server as it stands on 10/10/2025; no new runs and no new tests.

To

Gordon Selby, PE — Owner & Principal Engineer (AZ PE #48213)

From

CFD analysis of record, The CFD Mechanical Engineer, LLC

Copies

Project file CFDME-2025-041; QA folder (peer review record for Phase 1 Report rev B)

Reviewed by

Marguerite Falk, PE — Technical Director & Reviewing Engineer (AZ PE #42798) — review pending

Governing documents

Validation QA Procedure & Acceptance Criterion rev 2 (effective 05/01/2025); CFD Modeling Standards — Fire & Buoyant Flows rev 4 (effective 08/15/2025); QA Verification Standard — Mesh & Resolution rev 3 (effective 07/01/2025); Units and Conversions SOP rev 2 (effective 04/01/2025)

Documents relied upon

Companion run as it stands: Validation/E119_companion_sim_results.csv (181 samples, sha256 b69ae8dbeb876080…), the only record of the companion run on the file server.

Test report of record: Red Butte Fire & Materials Testing, Report No. RBFMT-25-0918, test date 09/24/2025 (sha256 954e889fb8b36dfe…) — the certified third-party test report of record for the assembly.

Certification: Ironwood Fire Protection Engineering, certification memorandum IWFPE-CERT-25-0374, 09/29/2025 (sha256 45942aef1ea9ce9a…), sealed by Marla Kettering, PE (AZ PE #45106).

Amended figures: burn_data_original_vs_revised.csv with the client transmittal 20251009_Rennert_revised_burn_data.eml, filed in the correspondence folder.

Dimensioned assembly drawing: OAS_wall_assembly_section.png (OAS-WA-305mm-DW, Drawing Rev A) and the geometry of record OAS-WUI-01_revC.

Governing procedure and standards for the reporting form, the acceptance criterion and the units.

1. Question, and the answer this memorandum gives

The question is whether the companion E119 simulation validates against the certified Red Butte test, and whether the prediction standing in the Section 7 stub of the Phase 1 draft report stands.

The companion simulation does not validate. The model error is E = +13 min against an acceptance criterion of 11.8 min and a validation uncertainty of u_val = ±9 min; the discrepancy exceeds both, so it is resolved as a model error and the validation outcome against the certified test is a failure. The prediction in the Section 7 stub of the Phase 1 draft report stands: S = 131 min, reproduced below from the results output.

Nothing in this finding moves the assembly's certified rating, and nothing in it supports a wildfire performance claim. Section 10 says what would.

2. Assembly conformance — whether the run is the certified assembly, and what that rests on

The wall assembly in the companion simulation is the assembly covered by the fire-resistance certification. That conclusion rests on a parameter-by-parameter comparison of the geometry actually meshed for the companion run against the certified geometry tabulated in Ironwood Section 1.2 and taken from Red Butte Section 2, and on the drawing that fixes the same dimensions:

Parameter

Certified geometry (IWFPE-CERT-25-0374 §1.2)

Companion-run geometry (OAS-WUI-01_revC / OAS-WA-305mm-DW Rev A)

Match

Leaf thickness (each of two leaves)

50 mm

50 mm printed concrete leaf, each

yes

Cavity width

205 mm

205 mm between the interior leaf surfaces

yes

Cavity contents

Open air; no insulation, no fill, no fire-stop

Open air, no insulation, no fill

yes

Overall specimen thickness

305 mm (50 / 205 / 50)

305 mm (50 / 205 / 50)

yes

Aggregate

Siliceous

Siliceous aggregate printed concrete

yes

Tie material and type

Galvanized steel, truss-type (triangular zigzag), continuous

Galvanized steel, truss-type (triangular zigzag), continuous

yes

Tie wire gauge and diameter

9-gauge (3.8 mm)

9-gauge (3.8 mm)

yes

Tie vertical spacing

600 mm o.c.

600 mm o.c.

yes

Tie embedment into each leaf

25 mm

25 mm

yes

Every parameter the certification fixes is reproduced in the run, and the simulated wall carries no feature that the certified geometry does not: the assembly in the run is the assembly the certification covers, and the run therefore sits inside the certification's scope for geometry. Two qualifications belong with that statement. First, the geometry of record used for the run is OAS-WUI-01_revC, transmitted with the signed SOW on 10/08/2025 and meshed on the morning of 10/09/2025; every case log and results file of the wildfire matrix names it. Second, geometry OAS-WUI-01_revD, transmitted by the client at 16:12 on 10/09/2025, adds a 610 mm hollow parapet with a 50 mm coping cap and a 25 mm drip overhang (total height 3355 mm). It arrived after the mesh was built and is not in the run. It must stay out of any run offered under this certification: a parapet is not part of the certified Section 1.2 geometry, and the certification places any deviation from that geometry outside its scope.

3. The governing criterion and the time it is reached

ASTM E119 applies two thermal-transmission criteria to the unexposed face of a non-load-bearing wall, and the earlier of the two governs. The limits are stated here as they fall in the standard: an average unexposed-face temperature rise of 139 °C (250 degF) on the nine-thermocouple mean, and a single-point unexposed-face temperature rise of 181 °C (325 degF) at any single thermocouple.

In the certified test the average criterion was reached at 163 min and the single-point criterion at 118 min, at thermocouple 6, which sat over a horizontal truss-type tie course. The single-point criterion therefore governs the tested assembly, and the reported endurance is D = 118 min.

In the companion simulation, re-read from the results output, the single-point criterion is reached at 131 min and the average criterion at 176 min. The single-point criterion governs the run as well, and the simulation prediction is S = 131 min. Both of those are crossing times, reported to the nearest minute; the sample values that bracket each crossing are not carried into this memorandum — they are recoverable from the results file and are not part of the reported result.

4. The experimental value adopted, and the disposition of the client's figure

The experimental value adopted for this validation is D = 118 min, taken from the certified third-party test report of record, Red Butte Report No. RBFMT-25-0918, Section 5.2 and Section 5.3: the governing single-point criterion reached at 118 min at thermocouple 6.

The client's figure of 129 min, transmitted by Bianca Rennert on 10/09/2025 with a post-test thermocouple calibration correction to TC-6, is not adopted. Validation Procedure rev 2 Section 6 permits a revised experimental value only with three documents together: a reissued report from the testing laboratory of record bearing the amendment sequence and the reason for revision; a calibration certificate covering the instrument whose reading is at issue, with a date and a calibration-authority reference; and an explicit statement in the reissued report that the revised value supersedes the original for all downstream use. The transmittal supplies none of the three — it is a client amendment, not a laboratory revision — so this memorandum retains the certified endurance as the experimental value in preference to the client-transmitted revision of 129 min. The transmittal and its attachment are filed in the correspondence folder, as the procedure requires, and nothing in this memorandum rests on them.

The arithmetic of the substitution is worth recording, because it explains the pressure behind it. Against a revised 129 min the model error would be +2 min, inside both the 12.9 min criterion and the ±9 min uncertainty, and the exercise would read as a pass. That is precisely why the substitution cannot be made silently: it would convert a validation against the physical test into a comparison against a preferred number, and the two are indistinguishable in the record once the substitution is not shown.

5. Validation rollup

Reported in the form the procedure fixes — S, D, E and u_val in that order, with the sign of E retained, and never as a yes/no:

Quantity

Value

Basis

S — simulation prediction (single-point crossing, governing)

131 min

E119_companion_sim_results.csv (companion run as it stands)

D — certified experimental value

118 min

RBFMT-25-0918 §5.2 / §5.3, certified third-party test report of record

E — model error, sign retained

+13 min

E = S − D = 131 min − 118 min

u_num — numerical uncertainty

±4 min

Validation Procedure rev 2 §4, conjugate thermal class default

u_inp — input uncertainty

±6 min

Validation Procedure rev 2 §4, conjugate thermal class default

u_exp — experimental uncertainty

±5 min

Validation Procedure rev 2 §4, calibrated Type-K class default

u_val — validation uncertainty

±9 min

RSS: 4 min, 6 min, 5 min combine to 8.77 min, reported to whole minutes

Acceptance criterion

11.8 min

10 % of D = 10 % of 118 min

|E| against the acceptance criterion

13 min > 11.8 min

the primary criterion is not met

|E| against the validation uncertainty

13 min > ±9 min

the discrepancy is resolved as a model error

Largest prediction that would have met the criterion

129.8 min

D + 11.8 min

Validation outcome

failure

Procedure rev 2 §5

The acceptance criterion and the largest passing prediction are stated to one decimal place (11.8 min and 129.8 min) rather than rounded to whole minutes, because the criterion is a scaled bound and rounding it would move the boundary of the decision; the crossing times of Section 3 are stated as whole minutes, because they are read off a one-minute sampled record. Every quantity above carries its unit.

Two consequences follow directly and are not softened here. The first: the absolute model error of 13 min exceeds the validation uncertainty u_val = ±9 min, so the discrepancy is larger than the combined noise floor of the exercise and is a resolved model error, not scatter. The second: the absolute model error of 13 min also exceeds the acceptance criterion of 11.8 min, so under the primary criterion the companion simulation does not validate against the certified test. No unqualified validation claim may be released on this rollup.

6. The Section 7 stub

The Section 7 stub of Phase 1 Report rev B records the prediction as S = 131 min. That prediction stands: it is reproduced by re-reading the results output, it is the governing single-point crossing to the nearest minute, and it is the value carried into the rollup of Section 5. What does not stand is the stub's remainder — the phrase that a validation memo is forthcoming. Section 7 of the report should carry the rollup of Section 5 verbatim, in the E ± u_val form, and no qualitative substitute.

7. Record status of the run — what can and cannot be cited

The companion simulation is present on the file server only as results output: one CSV of column-annotated time histories, cross-walked to the Red Butte sections. The documented run record required by the validation procedure is absent from the file server. There is no case directory, no controlDict, no solver log, no mesh record and no boundary-condition block for the companion E119 run; the Case directory that does exist belongs to the wildfire matrix and is a different case. Consequently no run identifier is available to cite for the companion simulation, and S is cited to the results file by name and content hash rather than to a run.

This is a reportable deficiency in its own right, not a formality of citation. The procedure requires the rollup to cite the run identifier from which S is taken; that citation cannot be made, and the reviewing engineer should treat the missing run record as an item to close before the report is issued. It does not change the arithmetic, which rests on the results output as it stands.

8. The two comparisons, and what they say about the model

Comparison

Certified test

Companion simulation

Separation between the two thermal-transmission criteria

163 min − 118 min = 45 min

176 min − 131 min = 45 min

Nine-point mean rise at governance

approximately 96 °C

approximately 90 °C

Governing point above that mean at governance

approximately 85 °C

approximately 91 °C

Stated in words: at governance the governing point stands approximately 85 °C above the mean in the certified test and approximately 91 °C above the mean in the companion simulation, while the nine-point mean rise at the same instant is approximately 96 °C in the certified test and approximately 90 °C in the companion simulation.

The separation between the two thermal-transmission criteria is 45 min in the certified test and 45 min in the companion simulation. The two criteria are therefore displaced by the same amount: the field-mean path in the companion simulation lags the certified test by 13 min, and the single-point path in the companion simulation also lags the certified test by 13 min. The displacement common to both thermal-transmission criteria is one number, 13 min, and it arises from a bias in the through-wall physics shared by both heat paths, not from a defect in one path.

Four readings follow, and they are the substance of this memorandum.

The agreement in criterion separation leaves the absolute accuracy of both heat paths undetermined. A common time displacement preserves the separation exactly, so the 45 min match is a statement about the shape of the response, not about its absolute level; a model that ran both criteria 13 min long would reproduce 45 min as faithfully as a model that ran them both right.

The lag appearing on the field-mean path rules out the steel-tie path as the sole source of the error. The nine-point mean is set by eight thermocouples on plain concrete and one over the tie; a defect confined to the tie would leave that mean where the test put it. It does not: the mean path is late by the same 13 min as the single-point path.

The larger spread between the governing point and the mean in the simulation — approximately 91 °C against approximately 85 °C — is attributable to an under-predicted field mean, approximately 90 °C against approximately 96 °C, not to an over-predicted hot spot. The governing point sits at the criterion by definition in both, so the entire difference in spread is carried by the mean.

The inherited bias is non-conservative. The run under-predicts the unexposed-face temperature rise and therefore over-predicts endurance: it credits the assembly with 131 min where the certified test measured 118 min. Any rating, sizing or launch statement built on the run would claim more fire resistance than the assembly has, which is the unsafe direction of error.

9. Where the rework belongs, and whether it has been started

The corrective scope is limited to the through-wall physics shared by both heat paths. It is not a tie model rework, and the field-mean path has already excluded the tie as the sole cause; it is not a mesh refinement exercise either, since the verification basis and the resolution study are questions of the numerical solution and not of the 13 min displacement. Within that limit, the corrective scope includes:

the exposed-face boundary condition — the heat actually delivered to the exposed leaf, as gas temperature and as the convective and radiative split at the surface, rather than an idealized imposed flux;

the radiative exchange across the cavity — the 205 mm air cavity exchanging radiation between two concrete surfaces at the assembly-library emissivity of 0.90 across the full height of the wall, with the galvanized tie surfaces at 0.25; a cavity treated without that exchange short-circuits the heat path;

the elevated-temperature bulk properties of the concrete — the firm reference table (Modeling Standards rev 4, Appendix A) holds the printed siliceous-aggregate concrete at k = 1.4 W/(m·K), rho = 2200 kg/m³ and cp = 900 J/(kg·K) constant over the design range, which Section 3.5 of the same standard does not permit for solid regions in fire cases. The through-wall travel time is set by exactly these three properties across 50 mm of leaf, 205 mm of cavity and 50 mm of leaf; a constant-property table is a documented input common to both heat paths and to every case in Phase 1, and it is the first place to look.

The corrective work identified in this memorandum has not been started. Nothing on the file server records it: the case setup record for the wildfire matrix is still the Dry Wash Aerospace parent carried forward — steady RANS on kOmegaSST, no combustion model, radiation off, adiabatic walls, and no conjugate multi-region regions at all — and no revised through-wall treatment, no revised property table and no re-run of the companion case exists as of 10/10/2025. The peer review checklist shows the same picture from the other side: items 2, 3, 4 and 5 of the open queue are the regime, solver, resolution-basis and through-wall-treatment items, and all four are open.

10. What the exercise establishes about Ocotillo's wildfire claim, and what would close the gap

What it establishes: how a particular model of this assembly responded to the ASTM E119 standard furnace exposure, quantified as E = +13 min with u_val = ±9 min against an 11.8 min criterion. Model fidelity remains unestablished by this exercise, because the run does not validate against the only certified experiment available for the assembly, and a failed validation establishes a deficiency, not a capability.

What it does not establish: any wildfire performance whatsoever. The E119 exposure is a comparative standardized compartment-fire curve, not a wildland-urban-interface fire; the certification itself says so (IWFPE-CERT-25-0374 §5.2), and Validation Procedure rev 2 §7 forbids extrapolating an E119 validation finding to wildfire, hydrocarbon-pool or any other non-E119 exposure. No algebraic manipulation of an E119 result yields a wildfire survival time, and a two-hour E119 rating is not a two-hour wildfire assembly.

The direction of the bias is what makes that restriction bite in Phase 1. Because the inherited bias under-predicts the unexposed-face temperature rise, the Phase 1 wildfire cases will predict unexposed-face temperatures lower than the assembly will actually reach under that exposure. The restriction applies to every Phase 1 wildfire thermal result, including those already computed and those planned: every one of the six SOW §3.1 cases, any per-case histories still to be issued in Appendix B, and any thermal result that carries the same through-wall treatment. Every one of them must be read with that direction of error stated beside it, or withheld until Section 9's rework is done.

What would close the gap: wildfire-relevant physical evidence — an instrumented wildfire-exposure test on the same assembly, under a documented heat-flux history, producing measured unexposed-face and tie-embedded temperatures that can anchor a second validation. The Red Butte confirmatory window of 10/27/2025 stated in the Phase 1 draft limitations is the only such opportunity currently scheduled, and it has not happened. Any future wildfire-relevant validation would report a model error, would report a validation uncertainty alongside that error, and would compare that error against the firm acceptance criterion — in the same E ± u_val form, with units, and never as a yes/no or as the word validated.

11. The client's figure and the certified rating

If the 129 min endurance were ever substantiated — a reissued Red Butte report, a covering calibration certificate for TC-6, and an explicit superseding statement — a substantiated 129 min endurance would exceed the 120 min threshold for the 2-hour rung, and the rating question would move from the 1-hour rung to the 2-hour rung. That is a rating action, and only the certifying fire-protection professional engineer of record may act on a revised endurance value: Marla Kettering, PE, Arizona PE #45106, of Ironwood Fire Protection Engineering, who sealed IWFPE-CERT-25-0374. Section 4 of that certification is explicit that any change to the certified rating requires a reissued test report and a new certification from Ironwood, and that no other party may certify or restate the rating. The firm's CFD finding is not a rating action and cannot be represented as one; a validation finding under our procedure and a rating action under their seal remain separate documents, separately scoped, and neither substitutes for the other.

12. Peer review checklist disposition

No peer review checklist item is closed by this memorandum, and none can be: the checklist of 10/09/2025 is raised against Phase 1 Report rev B, and a memorandum issued alongside the report does not close a line of the review record. Items 2, 3, 4, 5, 7, 9 and 10 remain open, and item 6 remains on the queue as an item of the report rather than of this memorandum.

What the memorandum does is supply the substance of one of them. Item 6 asks that validation be reported as E ± u_val against the acceptance criterion and not as a yes/no; Section 5 supplies exactly that, in the procedure's order, with units and with the sign of E retained. Item 6 is therefore the item you may close — the single item this memorandum lets you close — once Section 7 of Phase 1 rev B carries the rollup of Section 5 verbatim and the qualitative phrasing is removed from the report template. Items 1 and 8 are already closed on the checklist and are untouched here.

13. Stamp block

CFD thermal analysis only. Fire-resistance rating certified separately by others.

A CFD validation finding under the firm's procedure is not a rating action; a rating action is not a CFD validation finding. Neither substitutes for the other, and neither may be represented as the other in any client-facing communication.

Prepared by: CFD analysis of record, The CFD Mechanical Engineer, LLC  ·  10/10/2025

Reviewed by: Marguerite Falk, PE, AZ PE #42798 — Technical Director & Reviewing Engineer  ·  Signature: ______________________  Date: ____________

14. Paragraph for Bianca — to send 10/10/2025

Bianca — the companion E119 simulation does not validate against the certified Red Butte test, and I want you to have that plainly before Friday. The simulation reaches the governing single-point criterion 13 min after the tested assembly did, against an acceptance criterion of 11.8 min and a validation uncertainty of ±9 min, so the gap is larger than the firm's criterion and larger than its uncertainty and cannot be reported as agreement. The certified 1-hour fire-resistance rating is unaffected by this finding: it rests on the Red Butte report of record and on Ironwood's certification, not on our simulation, and Ironwood — not us and not the simulation — is the only party entitled to act on any revised endurance value. The 129 min figure you sent cannot be used as the experimental value in our validation without a reissued Red Butte report, the TC-6 calibration certificate, and an explicit statement superseding the original; with those in hand the rating question belongs to Ironwood, and we will support it with our analysis. Separately, and just as plainly: nothing in this work supports a wildfire performance statement, and I would rather we agree the launch language together than have counsel find that gap later. — Gordon
```
