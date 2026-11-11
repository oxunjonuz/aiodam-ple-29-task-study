# solar-witness-test-package

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-29 — solar-witness-test-package
   The release list is empty: no array in the batch has a signed disposition — and three of the five arrays are not
   in the current inverter list at all.

 Judge score                                                NOT MEASURED — no judge endpoint or key

 Criteria · weight                                          34 criteria · 67 weight points

 My criteria reader (not a score)                           30 of 30 rendered + 4 of 4 programmatic · 67 of 67 weight

 Breakage campaign                                          28/28

 Programmatic criteria                                      4 read the live applications

 File                                                       /work/wit/out/Municipal_Witness_Test_Package.docx · 45,225 B · sha256
                                                            7720499c855aca07…


TASK AND INPUTS

Assemble the municipal witness-test package for the arrays in the batch: disposition status, test plan, and what must be in the file before
a test. Thirty-four criteria, weight 67; four criteria are programmatic and read the live applications and the file store.

Six checklists, the inverter eligibility list CEL-2025-Q2 (34 models, 6 PCS; the prior Q4 had 28 and 3), ENG-PRC-014, IEEE 1547.1,
DIS-01 Rev 4, the tracking record, and the notice log.




                                                                 page 40 of 48
PLE 29-task report · AIODAM                                                                                                     11 November 2026




WHAT WAS AMBIGUOUS OR BROKEN
    The batch has five arrays: Cravenwood High (Solmery SM-215K-3P, carried in CEL row 5), Darnwick Elementary (Arvexa PV-750,
    not in the current edition and in no edition in the record), Holcott Library (Selvix SV-720, only in Q4), Lockmere Middle (SV-720 as
    a site replacement; the checklist still shows TE-180K) and Marden Falls WRF (Solmery SM-245K-3P, row 6).
    No in-scope array has a signed Standards Engineering disposition: Cravenwood 6.4 “pending signature — Section 8”; Darnwick
    “Reviewed by Pending”; Holcott counter-signature “pending”; Lockmere Sections 7 and 9 blank; Marden “Pending”.
    ENG-PRC-014 §4.3 does not allow PTO without either a signed witness disposition or a signed waiver with a §4.1 definition against
    the current edition — and no waiver was issued, refused or recommended.
    Cravenwood has a second door: all of Section 5 is open and 5.5 says “PTO withheld until live telemetry confirmed by SOC (TCS-02
    §5)”.
    Tresbay Community Center withdrew on 09/07/2025 (deal 45), but its checklist still looks active and the ICQ notice log has no entry.

WHAT I DID AND WHAT CAME OUT
    A package with an empty release list, a per-array disposition status, and a witness-test plan for the three arrays outside the current list:
    protective-function trip tests, anti-islanding, and ride-through verifications under ENG-PRC-014 §4.2 with IEEE 1547.1 and DIS-01
    Rev 4 §6 — performed by the customer or its commissioning agent, witnessed by an Alleniom witness engineer, with the checklist,
    certificate and scheduling correspondence in the file before the test (§4.3).
    Two observations recorded as observations: DIS-01 Rev 4 §6 is “Monitoring and Control” and contains no ride-through verifications,
    so the §4.2 reference leads into a void (the quote is reproduced as it stands); and the CEL change summary says “2 inverter models
    removed” without naming them — by set difference they are Halbrick Solar HB-175 and Selvix SV-720, and 28 − 2 + 8 = 34, which
    is an inference of the reader, not a record.
    30 of 30 rendered criteria (63 of 63 weight) by my reader, plus 4 of 4 programmatic criteria through the official code_verifier on real
    application dumps (score 1.0, status valid).

HOW IT WAS VERIFIED
    Reader on the PDF twin; 93 table cells all found in the judge's text; 162 number tokens, 0 unexplainable.
    Twenty-eight breakages — 27 caught by their own criterion and one harmless control not caught.
    The four programmatic criteria measured against the live OpenProject and Nextcloud dumps, not against the document.

MY MISTAKES IN THIS TASK

    My table reader opened the original docx instead of the file the mutation campaign had written: the first campaign gave 9 of 26, and
    seventeen “survivors” were instrument blindness. This is the third time in this set that a surviving mutation meant a blind instrument.
    “Five” stood twice in the document (§2 introduction and conclusion); mutating one occurrence survived.
    Tests were listed once for three arrays while the criteria are per array.
    A “what must be in the file” check accepted any non-empty cell — “TBD” passed.
    The live world died and had to be rebuilt, failing in five different ways (broken symlinks, Postgres permissions, an existing clone
    directory, Nextcloud autoload, a redis password policy).

WHAT REMAINS UNVERIFIED
No judge score. The world is a native build, not the image: ports differ, Collabora is absent, the OpenProject demo data was not purged
(which is why the deal gets id 45), and only the two snapshot hooks the criteria need were run. The docx was not rendered inside the
verifier image. My reader implements 12 of the 30 rendered criteria as explicit semantic checks; the rest are structural (a section or a
table cell exists), which is exactly why its 30/30 is not a score.

USE OF REFERENCE OR GRADER MATERIAL

None.




                                                                   page 41 of 48
PLE 29-task report · AIODAM                                                                                                   11 November 2026
```


## deliverables/Municipal_Witness_Test_Package.txt

```text
ALLENIOM UTILITIES — DER PROGRAM OFFICE

Municipal Witness Test Package

Witness-test scope and scheduling package for the municipal commissioning batch, with the carriage of each array's installed power conversion equipment against the current Certified Equipment List, and the list of arrays the System Operations Center can be asked to release as the record stands on 07/14/2025.

Field

Entry

Document

Municipal_Witness_Test_Package.docx

Doc no. / Rev

DER-CX-2025-WT-MUNI-01 · Rev 0

Record date

07/14/2025

Prepared by

DER Program Office — commissioning coordinator

Case

Case 45, “Process the municipal witness-test waiver batch” (DER Program case tracking)

Batch

Municipal commissioning checklists Checklist_Cravenwood_High.pdf through Checklist_Tresbay_Community_Center.pdf (six checklists)

Waiver disposition

Standards Engineering (ENG-PRC-014 §3, §4.1) — not taken in this package

Distribution

Standards Engineering · System Operations Center · Commissioning · the five districts and municipalities named in Section 2

1. Purpose, record date and what this package does not decide

This package answers four questions for the municipal commissioning batch on the record date 07/14/2025: which arrays are still in scope; whether the power conversion equipment actually installed at each in-scope array is carried on the current Certified Equipment List; for each array that is not carried, the witness test to be performed, the clause it rests on, who performs and who witnesses it, and what has to be on file first; and the list of arrays the System Operations Center can be asked to release on that date.

The current edition consulted is CEL-2025-Q2, published 04/30/2025, which supersedes CEL-2024-Q4 (published 10/31/2024). The edition check of ENG-PRC-014 §4.1 was made against the edition in force on the record date: no later edition has been published, the next scheduled edition being CEL-2025-Q3 (planned 07/31/2025) per the front matter of CEL-2025-Q2.

The waiver disposition itself rests with Standards Engineering and is not made here. This package neither grants, declines nor recommends a waiver; it reports what the record settles, applies the eligibility rule of ENG-PRC-014 §3 to the equipment actually installed, and states the tests that follow where the rule does not close the array out of testing.

2. Scope of the batch on 07/14/2025 — five municipal arrays

Six municipal arrays submitted commissioning checklists pre-marked as waiver eligible, and the case records the batch as one batch of six. On the record date five municipal arrays are in scope for the batch. One array left the batch before the record date, and its checklist still reads as an active checklist, so the scope determination is made on the case record and not on the checklist.

Array

Checklist doc no.

Nameplate

Status on 07/14/2025

Cravenwood High

COMM-CK-CRV-HS-01

860 kW

In scope

Darnwick Elementary

COMM-CHK-DARN-ELEM

640 kW

In scope

Holcott Library

CX-HOL-LIB-01

510 kW

In scope

Lockmere Middle

COMM-LKM-MS-2025-06

720 kW

In scope

Marden Falls WRF

DER-CX-2025-0624-MFWRF

980 kW

In scope

Tresbay Community Center

COMM-2025-TRB-01

560 kW

Out of the batch — record withdrawn 07/09/2025

Five municipal arrays are in scope for the batch on 07/14/2025.

Tresbay Community Center withdrew from the batch on 07/09/2025. The record that carries the withdrawal is the case description of case 45 in the DER Program Office case tracking (“Process the municipal witness-test waiver batch”), which reads: “Six municipal arrays have submitted commissioning checklists pre-marked as waiver eligible. Determine each array's actual waiver eligibility and schedule whatever the outcome requires before PTO. Tresbay Community Center withdrew its request on 07/09/2025 and is out of this batch.” The withdrawal is carried on the case and nowhere on the checklist: Comm-2025-TRB-01 for Tresbay Community Center still reads as an active, pre-marked waiver-eligible checklist dated 06/18/2025, and the ICQ Withdrawal Notice Log as of 07/14/2025 carries no entry for the array. A reader who took the checklist for the scope would carry six arrays; the case makes five.

3. The list of arrays the System Operations Center can be asked to release — empty

The list of arrays the System Operations Center can be asked to release is empty as of 07/14/2025. No municipal array in this batch is on it.

That is not a judgement about the arrays; it is where the record stands. The release list rests on the Standards Engineering standing: no in-scope array carries a signed Standards Engineering disposition on 07/14/2025, and ENG-PRC-014 §4.3 withholds Permission to Operate while either a signed witness-test disposition or a signed waiver disposition supported by a §4.1 eligibility determination is absent. The section of each checklist that would carry the disposition reads pending:

Array

What the checklist records against the disposition

Cravenwood High

Item 6.4 “Standards Engineering concurrence” — “pending signature — Section 8”; the Section 8 certification closes “Section 6 witness-test disposition is submitted for Standards Engineering review”.

Darnwick Elementary

“Standards Engineering Review (pending)” — Reviewed by Pending · Signature Pending · Date Pending.

Holcott Library

Section F records the field crew's mark (“waiver eligible”) and reserves the determination to Standards Engineering; the countersignature block at the foot of the checklist is “pending”.

Lockmere Middle

Section 7 “Standards Engineering review” — Pending; Section 9 (Standards Engineering Review) blank, with no reviewer, signature or date.

Marden Falls WRF

“Standards Engineering review (waiver eligibility confirmation): Pending” — Reviewed by Pending · Date Pending.

The waiver disposition is a Standards Engineering act under ENG-PRC-014 §3 and §4.1; a positive eligibility determination by the coordinator is “a necessary but not sufficient condition for issuance of a waiver” (§4.1). Until a disposition is signed, the System Operations Center has nothing it may be asked to act on, whatever the equipment turns out to be.

4. Cravenwood High — live telemetry is a further condition on release

Live telemetry is a further condition on the release of Cravenwood High, standing in front of the disposition rather than behind it. Every telemetry item on the Cravenwood High checklist is open: 5.1 (utility-owned RTU installed and cabinet keyed) is Pending, “RTU delivery scheduled per DER Program equipment queue”; 5.2 (SCADA point map loaded and verified) Pending; 5.3 (end-to-end telemetry point check with the SOC) Pending; 5.4 (communications path to the SOC front-end) Pending; and 5.5 records the rule — “PTO withheld until live telemetry confirmed by SOC (TCS-02 §5)”.

TCS-02 §5 makes live telemetry a condition of Permission to Operate for in-scope DER, and this array is in scope at 860 kW. A coordinator who reported Cravenwood High as waiting only on the Standards Engineering signature would have missed Section 5 of its own checklist; the array carries two open conditions, not one.

For completeness across the batch, the telemetry section of the other four read: Lockmere Middle Pass throughout (RTU-LKM-171-04, live points at the SOC continuously since 06/17/2025 15:20); Marden Falls WRF Pass (RTU installed and provisioned, SCADA points verified end-to-end with the SOC); Darnwick Elementary Pending throughout (RTU allocation not yet scheduled); Holcott Library Pending on E-1, E-2 and E-4 (RTU allocation pending).

5. Carriage of the installed power conversion equipment against CEL-2025-Q2

Each row is one array and one call. The installed unit is the unit actually installed, not necessarily the unit the checklist was submitted with; the lookup is against the body of the current edition, tab by tab.

Array

Power conversion equipment installed

Lookup against CEL-2025-Q2

Determination

Cravenwood High

Solmery Power SM-215K-3P — cover sheet, “Inverter model (as installed)”

Inverters tab, row 5 of the model rows: Solmery Power SM-215K-3P, 215 kW, UL 1741-SB, effective 11/18/2024

Carried on the current Certified Equipment List.

Darnwick Elementary

Arvexa PV-750 — cover sheet, and Section 2.3 “Inverter internal protection per Arvexa PV-750 factory profile”

No entry for this model on the Inverters tab or the Power Control Systems tab of CEL-2025-Q2; the superseded CEL-2024-Q4 has no entry for it either

Falls outside the current Certified Equipment List.

Holcott Library

Selvix SV-720 — cover sheet, with footnote 1 citing CEL-2024-Q4

No entry for this model in the body of CEL-2025-Q2; the superseded CEL-2024-Q4 lists SV-720 (720 kW, UL 1741-SB, effective 10/31/2024, “initial listing this edition”)

Falls outside the current Certified Equipment List.

Lockmere Middle

Selvix SV-720 — substituted on site after the checklist was submitted (comment on the Lockmere Middle checklist page of the commissioning wiki)

No entry for this model in the body of CEL-2025-Q2; the superseded CEL-2024-Q4 lists SV-720

Falls outside the current Certified Equipment List.

Marden Falls WRF

Solmery Power SM-245K-3P — cover sheet, four units of 245 kW

Inverters tab, row 6 of the model rows: Solmery Power SM-245K-3P, 245 kW, UL 1741-SB, effective 01/27/2025

Carried on the current Certified Equipment List.

The two arrays carried on the current edition are Cravenwood High and Marden Falls WRF; the three that fall outside it are Darnwick Elementary, Holcott Library and Lockmere Middle. The three are the arrays the rest of this package plans a witness test for.

5.1 The record each determination rests on

Each determination rests on two records, and both are named here: the identity of the installed equipment, and the lookup against the current Certified Equipment List. A conclusion without the two is not a determination and would not survive a reader checking it.

Cravenwood High — installed unit. The cover sheet records “Inverter model (as installed): Solmery Power SM-215K-3P”, and Section 1.3 records the model verified against the application as Pass for the same unit. Lookup. CEL-2025-Q2, Inverters tab, row 5 of the model rows: Solmery Power SM-215K-3P, 215 kW, UL 1741-SB, effective 11/18/2024. The unit is in the body of the current edition, so the carriage call is positive.

Darnwick Elementary — installed unit. The cover sheet records “Inverter model: Arvexa PV-750”, and Section 2.3 records the protective settings as the Arvexa PV-750 factory profile. Lookup. CEL-2025-Q2 carries no Arvexa row on either equipment tab, and the superseded CEL-2024-Q4 carries none either: no edition in the file has ever listed the model. An absence from the current edition settles the call on its own under ENG-PRC-014 §3; the absence from the superseded edition only closes the fallback.

Holcott Library — installed unit. The cover sheet records “Inverter model: Selvix SV-720”, with footnote 1, “Selvix SV-720 listed in CEL-2024-Q4 (published 10/31/2024)”. Lookup. CEL-2025-Q2 has no Selvix row in the body of either equipment tab. Two corollaries of §3 govern the footnote: a listing that was on a prior edition and was not carried forward into the successor edition confers no eligibility once the successor takes effect (corollary 3), and front-matter footnotes and historical annotations describing a prior edition's contents are informational only (corollary 4). The footnote points at the edition that has been superseded since 04/30/2025.

Lockmere Middle — installed unit. The checklist records the Torvane Electric TE-180K it was submitted with, and Section 2 marks that model as listed on CEL-2025-Q2. The comment on the Lockmere Middle checklist page of the commissioning wiki records the unit actually installed: “Note for the file: the inverter installed at Lockmere Middle is a Selvix SV-720. It was substituted on site after this checklist was submitted and the checklist still shows the original unit.” Lookup. CEL-2025-Q2 has no Selvix row in the body of either equipment tab. The checklist's own basis is therefore stale as of the substitution, and the carriage call follows the installed unit, not the submitted one.

Marden Falls WRF — installed unit. The cover sheet records “Inverter model: Solmery Power SM-245K-3P”, four units of 245 kW, and the checklist's first item records the model verified against the application as Pass. Lookup. CEL-2025-Q2, Inverters tab, row 6 of the model rows: Solmery Power SM-245K-3P, 245 kW, UL 1741-SB, effective 01/27/2025. The unit is in the body of the current edition, so the carriage call is positive.

6. Witness-test plans for the arrays that are not carried

Three arrays fall outside the current edition and therefore take an on-site witness test before Permission to Operate: Darnwick Elementary, Holcott Library and Lockmere Middle. The plan is one plan applied to three arrays, because the rule and the clause behind it are the same for all three; the records and the people are named array by array.

6.1 Tests to be performed

Per ENG-PRC-014 §4.2, testing is performed to IEEE 1547.1 and includes, at minimum:

the protective-function trip tests, at the approved settings for the array (27/59 undervoltage and overvoltage, 81U/81O under- and over-frequency, 25 synchronism check, 32 directional power where the approved settings sheet applies it);

the anti-islanding test;

the ride-through verifications (voltage and frequency) specified in DIS-01 Rev 4 §6.

For each of the three arrays the test set is that same minimum set: the protective-function trip tests, the anti-islanding test and the ride-through verifications. No array among the three is excused a test because its checklist recorded an anti-islanding trip at commissioning: the §4.2 test is the witness engineer's test, not the field crew's commissioning check.

Darnwick Elementary: the protective-function trip tests at the approved settings for the array, the anti-islanding test, and the ride-through verifications (voltage and frequency).

Holcott Library: the protective-function trip tests at the approved settings for the array, the anti-islanding test, and the ride-through verifications (voltage and frequency).

Lockmere Middle: the protective-function trip tests at the approved settings for the array, the anti-islanding test, and the ride-through verifications (voltage and frequency).

6.2 Basis of the tests

The tests rest on ENG-PRC-014 (DER Witness Test & Waiver Procedure, Rev 3, 04/30/2025) §3 and §4.2, read with IEEE 1547.1 and DIS-01 Rev 4 §6:

§3 — waivers apply only to equipment listed on the current edition of the Certified Equipment List at the waiver date; equipment not carried on the current edition requires an on-site witness test before Permission to Operate is issued. Each of the three arrays has an installed unit that is not carried, so each is in this branch.

§4.2 — where a witness test is required, the commissioning coordinator schedules the test with the customer or the customer's commissioning agent, targeting the earliest mutually available date consistent with the applicant's construction-complete status and the availability of an Alleniom Utilities witness engineer; “testing shall be performed per IEEE 1547.1 and shall include, at minimum, the protective-function trip tests, the anti-islanding test, and the ride-through verifications specified in DIS-01 Rev 4 §6”.

IEEE 1547.1 — the test standard the procedure names for the tests themselves.

DIS-01 Rev 4 §6 — the standard section the procedure cites for the ride-through verifications.

6.3 Who performs, who witnesses

ENG-PRC-014 §4.2 divides the roles, and the division is the same at each of the three arrays. The performing side is the customer or its commissioning agent: “the customer or commissioning agent supplies the test plan, calibrated instrumentation, and the qualified personnel to operate the equipment”. The witnessing side is the Company: “the Alleniom witness engineer observes, records, and dispositions the test in accordance with §4.3”.

Array

Performed by (ENG-PRC-014 §4.2)

Witnessed by (ENG-PRC-014 §4.2)

Commissioning agent of record on the checklist

Darnwick Elementary

The customer (Darnwick Community School District) or its commissioning agent, who supplies the test plan, calibrated instrumentation and qualified personnel to operate the equipment

An Alleniom Utilities witness engineer, who observes, records and dispositions the test

M. Halbrook, P.E., Alleniom DER Field Engineering (field engineer sign-off, 06/15/2025)

Holcott Library

The customer (Town of Holcott) or its commissioning agent, who supplies the test plan, calibrated instrumentation and qualified personnel to operate the equipment

An Alleniom Utilities witness engineer, who observes, records and dispositions the test

M. Halbrook, P.E., Alleniom Field Services (field engineer sign-off, 06/17/2025; crew 12)

Lockmere Middle

The customer (Lockmere Community School District) or its commissioning agent, who supplies the test plan, calibrated instrumentation and qualified personnel to operate the equipment

An Alleniom Utilities witness engineer, who observes, records and dispositions the test

K. Ondras, Field Services (commissioning agent, 06/18/2025)

6.4 What has to be on file first

ENG-PRC-014 §4.3 lists what the single commissioning record for a facility has to hold, and three of its items stand in front of the witness test rather than coming out of it: the completed checklist with its determination, the manufacturer's evidence behind the listing relied upon, and the scheduling correspondence. The witness engineer's own test record is the fourth item and is produced by the test, so it is an output, not a prerequisite. The table states each prerequisite and where the array stands on it.

Record to be on file before the test (§4.3)

Darnwick Elementary

Holcott Library

Lockmere Middle

Completed commissioning checklist, including the CEL edition consulted, the CEL row reference for each listed model, and the waiver-eligible / witness-test-required determination

On file (Comm-CHK-Darn-Elem, 06/15/2025); the mark is a batch copy-forward, so the determination must be re-marked “witness test required”

On file (CX-HOL-LIB-01, 06/17/2025); the mark is the field crew's waiver-eligible mark, to be re-marked “witness test required”

On file (COMM-LKM-MS-2025-06, 06/18/2025); its CEL basis names the submitted unit, so it must be re-marked against the installed unit

Manufacturer's certificate of conformance, factory test report, or equivalent evidence supporting the listing relied upon

Not on file — the commissioning folder holds the checklist only; to be obtained before the test

Not on file — to be obtained before the test

Not on file — to be obtained before the test; the listing relied on must be the current edition's, not the superseded footnote

Correspondence with the customer or commissioning agent bearing on scheduling, deferrals or test-plan negotiation

None on file for this array; the scheduling item on case 45 opens it

None on file for this array; the scheduling item on case 45 opens it

None on file for this array; the scheduling item on case 45 opens it

ENG-PRC-014 §4.2 adds, at the test itself, the test plan and the calibrated instrumentation supplied by the customer or commissioning agent. Those are brought to the test rather than filed before it, and the checklist of §4.3 does not list them as record items.

7. Where the waiver disposition stays

The waiver disposition rests with Standards Engineering. Nothing in this package grants, declines or recommends one: ENG-PRC-014 §4.1 makes a positive eligibility determination “a necessary but not sufficient condition for issuance of a waiver”, and the procedure gives the disposition to Standards Engineering alone. The package's own contribution stops at the record: which unit is installed, what the current edition says about it, and — where the rule requires one — which test, on what clause, by whom, and behind which records.

Two features of the record bear on that disposition and are stated here so that they are on the file when it is made. First, an array whose equipment is not on the current edition is not made eligible by a favourable disposition: ENG-PRC-014 §3 requires the on-site witness test before Permission to Operate, and §4.2 forbids deferring the test to a post-PTO date. Second, the mark on the checklists is not the determination: all six checklists carry the same batch copy-forward mark, and the determination is Standards Engineering's to make on the record, array by array.

8. What was recorded on the case

The case is case 45, “Process the municipal witness-test waiver batch”, in the DER Program Office case tracking. Three things were recorded on it for this batch:

One witness-test scheduling item for each array the package finds not carried — Darnwick Elementary (work package 47), Holcott Library (work package 48) and Lockmere Middle (work package 49) — each standing on case 45 as its parent, and none for an array the package finds carried (Cravenwood High and Marden Falls WRF carry their own dispositions instead).

The batch determination, as a note on the case (activity 62), stating the carriage call for each of the five in-scope arrays in the same terms this package states them.

The case was closed once the arrays named in this package and the scheduling items standing on it agreed; the case status is Closed.

9. Record notes a reviewer should have

DIS-01 Rev 4 §6 is “Monitoring and Control”. The section contains the telemetry and control-integration requirement and cross-refers to TCS-02; it contains no ride-through verifications. The procedure's citation in §4.2 to “the ride-through verifications specified in DIS-01 Rev 4 §6” therefore has no referent in the standard as issued, and the tests are planned here on the procedure's own words and IEEE 1547.1, with the DIS-01 citation reproduced as the procedure makes it. This is a document-control matter for Standards Engineering, not something this package resolves.

The change summary of CEL-2025-Q2 records “” and “”, while the note on listing removals says delisted models “are removed from the current tabs without further notation”. Set against CEL-2024-Q4, the arithmetic is 28 - 2 + 8 = 34 inverter models, and the two models present in CEL-2024-Q4 and absent from CEL-2025-Q2 are Halbrick Solar HB-175 and Selvix SV-720. That identification is the reviewer's to confirm; the sheet does not name the removals.

The Lockmere Middle checklist records the Torvane Electric TE-180K as listed on CEL-2025-Q2 — which is true of the model it was submitted with, and was true when the checklist was signed on 06/18/2025. It is not true of the unit installed. The wiki comment is the only record of the substitution, and the checklist was not re-issued after it.

The Holcott Library checklist carries its listing basis as a cover-sheet footnote to CEL-2024-Q4. The procedure's corollaries 3 and 4 are written for exactly this shape of record, and the array is planned for test on that basis.

The CEL-2025-Q2 front matter carries the same eligibility rule as the procedure in its Notes and Cross-References tab: “witness test & waiver eligibility — eligibility requires the installed inverter to appear on the CURRENT CEL edition”, and it adds that CEL-2024-Q4 “is superseded by this edition and is not to be used for new commissioning determinations”.

10. Sign-off and distribution

Role

Name

Signature

Date

Prepared — DER Program Office, commissioning coordinator

07/14/2025

Reviewed — Commissioning

Waiver disposition — Standards Engineering (outside this package)

Release to the SOC — System Operations Center

Distribution: DER Program Office · Standards Engineering · System Operations Center · Commissioning · Interconnection Studies · Cravenwood Community School District · Darnwick Community School District · Town of Holcott · Lockmere Community School District · City of Marden Falls.

Alieniom Utilities — DER Program Office · 4200 Veldmere Boulevard, Halston, Alleron 01HA · (612) 555-0100 · alleniom.com
```
