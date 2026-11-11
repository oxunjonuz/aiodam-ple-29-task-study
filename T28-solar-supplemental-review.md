# solar-supplemental-review

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-28 — solar-supplemental-review
   The earlier determination's outcome stands; its reason does not — Screen D passes on the record, and the
   study agreement is required for four other screens.

 Judge score                                               NOT MEASURED — no judge endpoint or key

 Criteria · weight                                         22 criteria · 77 weight points

 My criteria reader (not a score)                          22 of 22 criteria · 77 of 77 weight

 Breakage campaign                                         21/21

 File                                                      /work/ssr/build/Bramwick_Commons_Screen_Memo.docx · 32,204 B · sha256
                                                           198adf5577db161d…


TASK AND INPUTS
Decide the supplemental interconnection review for a solar project (Bramwick Commons, feeder ALN-214) and write the screening
memo. Twenty-two criteria, weight 77.

The interconnection standard SRP-3, DI-01/DIS-01, the tariff, the meter (AMI) export, the feeder map, the queues, and the study
backlog.

WHAT WAS AMBIGUOUS OR BROKEN

    The memo of 25/06/2025 sent the application to a study agreement through a failure of Screen D: 1.8 MW against “a minimum of 1.1
    MW”. On the record Screen D passes: the daytime minimum is 2,400 kW and the request is 1,800 kW.
    A study agreement is still required — under SRP-3 §2, because four other screens are inconclusive.
    Screen A has no threshold to apply: the word “penetration” does not occur in DIS-01 at all, and the tariff sets only a 500 kW scope.
    Screens B, C and E are inconclusive for want of records: no planning study for ALN-214 (a “Last Study Date” of 12/2023 is a date,
    not a study), no impedance data, no interrupting rating or fault contribution, and no certified reverse power for the Larkspur Ridge
    transformer.




                                                                page 39 of 48
PLE 29-task report · AIODAM                                                                                                 11 November 2026




WHAT I DID AND WHAT CAME OUT
    Recomputed Screen D from the AMI export: 01/07/2024–30/06/2025, 15-minute steps, 35,040 intervals without gaps, 8,760
    evaluated (10:00–15:45), no exclusions because no outage, transfer or switching record exists on ALN-214.
    Minimum 2,400 kW at 11:30 on 12/02/2025; the next values are 2,418 and 2,433; only three intervals in the year fall below 2,500
    kW, and no day has a daily minimum at or below 1,800 kW.
    The 1,100 kW figure is the absolute night minimum (03:45 on 23/03/2025), which §4 excludes by name. The memo's window and
    step also differ from the export (“24/06/2024–24/06/2025”, “sixty minutes”).
    22 of 22 criteria and 77 of 77 weight by my reader on the PDF twin.

HOW IT WAS VERIFIED

    Reader on the PDF twin; a second path over 69 checks (raw XLSX XML with hand-decoded Excel serial dates, pdftotext for the
    standards, document.xml for the older memo).
    Render completeness: 117 of 117 paragraphs and cells over 20 characters.
    28 numbers with recorded provenance, 0 unexplainable; a control with three invented numbers catches 3 of 3.
    Twenty-one breakages — all noticed; two harmless controls not noticed.
    The official test.sh chain: reward 1.0 with a stub judge, and the official rendering text is character-identical to mine. Negative
    controls: m01 0.870130 (c0, weight 10), m18 0.961039, m20 0.974026, and no file at all 0.0.

MY MISTAKES IN THIS TASK
    I wrote that there is no existing DER on the feeder. The DER registry and the holds registry have no feeder column at all: their silence
    about ALN-214 is blindness of the key, not evidence. No checker caught this — reading the registry did.
    ALN-214 has two rows in the map (capacity and the pre-application note); my first draft had one.
    The number audit accepted an invented number as a date if a time stood within 40 characters.
    Three mutations were incomplete after I rebuilt the Screen D table.
    c12 stayed green when “8,760” was removed because “9,125” remained; the rubric accepts either counter — the rubric was right.

WHAT REMAINS UNVERIFIED
No judge score. The live applications were not touched, because no criterion in this task reads db_diff.

USE OF REFERENCE OR GRADER MATERIAL
None. The AMI export is project data, not a reference answer.


PART C · RUBRIC · ELECTRICAL / INTERCONNECTION
```


## deliverables/Bramwick_Commons_Screen_Memo.txt

```text
Alleniom Utilities — DER Program Office

4200 Veldmere Boulevard, Halston, Alleron 01HA  ·  (612) 555-0100  ·  alleniom.com

SUPPLEMENTAL REVIEW MEMO — SCREEN DETERMINATIONS AND DISPOSITION

Doc no.

Rev

Date

Owner

IS-BRAM-2025-07-14

A

07/14/2025

Interconnection Studies

To:      Meera Vellanki, Manager, Interconnection Studies

From:   Interconnection Studies — supplemental review of IS-BRAM-2025-06-25

Date:   07/14/2025

Subject: Bramwick Commons Solar (1,800 kW requested, Feeder ALN-214) — supplemental review screen determinations and disposition

Supersedes: IS-BRAM-2025-06-25, dated 06/25/2025 — Bramwick_Commons_Screening_Memo_v1.docx. This memo supersedes the earlier memo of record; the statement is repeated in section 7, as SRP-3 §6(f) requires.

Application record

Item

Value

Application identifier

IS-BRAM-2025-06-25 — the Interconnection Studies file number carried on the application record for Bramwick Commons Solar. No PXA number appears on any queue, register or study record for this application as of 07/14/2025.

Applicant

Bramwick Commons Solar LLC

Project

Bramwick Commons Solar

Subject feeder

ALN-214 (Larkspur Ridge substation, 12.47 kV)

Requested capacity

1,800 kW (1.8 MW)

Resource type

Solar photovoltaic

Record date

07/14/2025

1.  Purpose and effect of this memo

This memo records the supplemental review of the Bramwick Commons Solar application on feeder ALN-214, conducted under SRP-3 Rev 3 (effective 02/10/2025) against the DER Program record as it stands at 07/14/2025. It records one determination for each of the five screens in SRP-3 §3 — A, B, C, D and E — with the basis relied on for each, and the disposition SRP-3 §6(d) requires.

The memo it replaces was a Screen D determination dated 06/25/2025, countersigned the same day, and it recorded a single figure as the feeder minimum load. SRP-3 §4 anticipates exactly that situation: “Where a prior supplemental review memo on the same feeder has applied a different minimum-load basis, the reviewer shall recompute the value under this section and, if the two disagree, supersede the earlier memo in the file.” The two disagree on the minimum-load value. This memo recomputes the value on the AMI record and supersedes the earlier memo; it does not revisit the rest of that memo's contents, which the file does not need to re-argue.

2.  Record reviewed

SRP-3 §5 names the sources authoritative for supplemental review. Everything below is taken from them; where a source is silent about ALN-214, that silence is recorded, because for four of the five screens it is the finding.

Source (as of 07/14/2025)

What it holds for ALN-214 / Bramwick Commons

ALN-214 AMI Interval Load Export — feeder-aggregate standing extract, as of 07/2025; 15-minute intervals; 35,040 values from 07/01/2024 00:00 through 06/30/2025 23:45

Sole authoritative source for the Screen D minimum-load determination under SRP-3 §5(a). Recomputed in section 3.4 below.

Feeder Hosting Capacity Map, HCM-2024-Q2 (published 05/15/2024, Distribution Planning)

One capacity row for ALN-214: Larkspur Ridge substation, 12.47 kV, published available capacity 4.9 MW, DER category Pure Generation, Last Study Date 12/2023 — and beneath it a note row recording that ALN-214 is the point of interconnection for the Bramwick Commons Solar pre-application dialogue tracked under Distribution Planning correspondence. No planning study document is attached to either row.

2025-07-14 Active Queue Report

380 entries, each carrying a feeder. None on feeder ALN-214, and none for Bramwick Commons Solar.

2025-07-14 On-Hold Queue Report

140 held applications, listed by project ID, customer, site and capacity. The report carries no feeder column, and no entry for Bramwick Commons Solar.

Existing-DER Register Extract (as of 07/14/2025; 500 kW reporting floor)

21 sites, listed by service address, customer, site name, nameplate, commissioning date and telemetry status. The extract carries no feeder column, so it cannot be searched for ALN-214; it holds no entry for Bramwick Commons Solar. Systems below 500 kW are outside it.

SIS Backlog Status 2025-07

44 study rows, each carrying a feeder. No system impact study for Bramwick Commons Solar, and no row for any application on ALN-214.

DIS-01 Rev 4 (03/01/2024), DPS-07 Rev 2 (09/15/2023), SRP-3 Rev 3 (02/10/2025)

The standards applied in this review.

Bramwick_Commons_Screening_Memo_v1.docx (IS-BRAM-2025-06-25, 06/25/2025)

The memo of record that this memo supersedes; compared in section 4.

Distribution planning record for ALN-214

No feeder planning study; no conductor or transformer impedance data; no short-circuit study; no protection coordination study; no feeder outage, load-transfer or planned-switching record. The only planning study in the record is 2024-10-15_ALN-108_Planning_Study.docx, which is for feeder ALN-108.

3.  Screen determinations

SRP-3 §3 requires the screens to be evaluated in the order given, each documented with its underlying data. Where the record cannot support either a pass or a fail, the determination recorded is inconclusive, and SRP-3 §2 governs what follows.

Screen

Determination

Basis in one line

A — Aggregate DER penetration

Inconclusive

No penetration threshold is set in DIS-01, and no record carries the feeder's existing and queued DER penetration against one.

B — Voltage regulation and flicker

Inconclusive

No feeder planning study for ALN-214 is on file, and no impedance data exists for the screening calculation the screen falls back to.

C — Short-circuit contribution

Inconclusive

No interrupting rating, short-circuit contribution or coordination margin for ALN-214 is on file.

D — 100%-of-minimum-load (solar)

Pass

Daytime-window minimum 2,400 kW; requested 1,800 kW is below it.

E — Reverse power at substation

Inconclusive

No certified reverse-power capability for the Larkspur Ridge transformer is on file.

3.1  Screen A — Aggregate DER penetration on the feeder

Requirement (SRP-3 §3, Screen A). “Total nameplate DER (existing plus queued plus requested) on the subject feeder shall not exceed the applicable penetration threshold set in DIS-01. Queue positions are those on record with the DER Program Office at the date of screen evaluation.”

DIS-01 sets no aggregate DER penetration threshold for the feeder. The word penetration does not appear anywhere in DIS-01 Rev 4, and the standard contains no numeric limit on the DER a feeder may carry. Its thresholds are of other kinds and cannot stand in for the Screen A threshold: the 500 kW program-scope threshold in §3 classifies a facility and fixes its review path; the TCS-02 threshold in §6 triggers telemetry; §3.4 defines aggregation at a point of common coupling; §5 sets protective equipment; §7 governs post-execution changes to an executed agreement. Screen A's phrase “the applicable penetration threshold set in DIS-01” therefore points at a provision that was never written, and the APUC tariff excerpt at §12 does not supply one either — it fixes the 500 kW program scope and delegates classification to DIS-01.

What the record holds. Requested capacity is 1,800 kW. What the feeder already carries cannot be established from the record. The existing-DER register extract lists 21 sites by service address and carries no feeder column at all, so it cannot be searched for ALN-214 and it holds no entry for Bramwick Commons Solar; systems below its 500 kW reporting floor are outside it in any case. The on-hold register is no better: 140 held applications, listed by project, customer, site and capacity, no feeder column, and no Bramwick entry. The active queue is the one register of the three that identifies feeders, and it carries 380 entries, none on ALN-214 and none for this application. Against that, no record carries the feeder's existing and queued DER penetration against a threshold — no published penetration figure for ALN-214 exists in any record, and with no threshold in DIS-01 there is nothing the 1,800 kW could be tested against.

The 4.9 MW figure. The one record that prints a number for ALN-214 is the hosting-capacity map, and the number it prints is 4.9 MW. That figure is the map's published available capacity — estimated headroom for new DER as of 05/15/2024 — and it is used in no screen in this memo: it is not a penetration threshold, it is not a certified reverse-power capability, and it is not a minimum-load value, so Screen A cannot be passed or failed against it and Screen E cannot be tested against it either. The map's own title block disclaims queue activity recorded after its publication date, and the figure is fourteen months old at the record date.

Determination: INCONCLUSIVE. Screen A is inconclusive because no record carries the feeder's existing and queued DER penetration against a threshold, and no threshold is set in DIS-01 to carry it against. Under SRP-3 §2 an inconclusive screen result is advanced to a study agreement rather than passed or failed; the disposition is recorded in section 5.

3.2  Screen B — Voltage regulation and flicker

Requirement (SRP-3 §3, Screen B). “Reviewed against the feeder's most recent planning study or, where none is on file for the subject feeder within twenty-four (24) months, against a screening voltage-rise calculation performed using conductor and transformer impedance from the distribution planning record.”

No feeder planning study for ALN-214 is on file. The only planning study in the record is for feeder ALN-108 and evaluates that feeder; no study report for ALN-214 appears anywhere in the record. The hosting-capacity map's Last Study Date of 12/2023 against ALN-214 is a date and not a study: it names no document, no study underlies it in the file, and Distribution Planning holds no planning study for this feeder that could be read against the screen. A date on a capacity map is not the feeder's most recent planning study.

Nor is there data to compute the alternative. Screen B falls back to a screening voltage-rise calculation using conductor and transformer impedance from the distribution planning record where no study is on file. No voltage-rise, flicker or impedance data for ALN-214 exists in the record: there is no conductor impedance table, no transformer impedance, no voltage profile, and no flicker measurement or study for this feeder. The fallback route is therefore closed as well, and closing it is the finding, not an omission.

Determination: INCONCLUSIVE. Screen B is inconclusive because no voltage-rise, flicker or impedance data for ALN-214 exists in the record, and the planning study the screen would otherwise be reviewed against is not on file for this feeder. SRP-3 §2 applies; see section 5.

3.3  Screen C — Short-circuit contribution

Requirement (SRP-3 §3, Screen C). “DER short-circuit contribution shall not cause any feeder-protective device to exceed 87.5% of its interrupting rating, nor cause any downstream protection coordination margin to fall below the values specified in DPS-07.”

What the record holds. No interrupting rating, short-circuit contribution or protection coordination margin for ALN-214 is on file. There is no feeder protection schedule, no device interrupting rating, no short-circuit study and no coordination study for this feeder, and nothing in the record states the DER's contribution to fault current on ALN-214. DPS-07 §5.1(c) places the coordination time-current documentation in the site's protection coordination study before energization, and no such study for this site or this feeder exists in the record.

Determination: INCONCLUSIVE. Screen C is inconclusive. The three quantities the screen tests — interrupting rating, short-circuit contribution and coordination margin — are all absent from the record for ALN-214, so neither the 87.5% test nor the DPS-07 margin test can be run in either direction, and none of the three may be assumed. SRP-3 §2 applies; see section 5.

3.4  Screen D — 100%-of-minimum-load screen for solar

Requirement (SRP-3 §4). “For solar DER, the minimum load used in this screen shall be the minimum load within the daytime window 10:00–16:00, determined from feeder AMI interval data. The absolute overnight minimum shall NOT be used. Requested DER capacity must be ≤ this daytime-window minimum for the site to pass the screen.”

Parameter

Value

Basis

Data source

ALN-214 AMI Interval Load Export, feeder-aggregate standing extract as of 07/2025

SRP-3 §5(a)

Data window

07/01/2024 to 06/30/2025, the twelve complete months

SRP-3 §4

Interval cadence

15 minutes

the export's timestamps

Intervals in the export

35,040

365 days x 96

Window evaluated

start times 10:00 through 15:45

SRP-3 §4

Intervals evaluated

8,760 fifteen-minute intervals

365 days x 24

Interval exclusions

None applied

SRP-3 §4, §6(c)

Daytime-window minimum load

2,400 kW, at 11:30 on 02/12/2025

minimum of the 8,760 intervals

Requested capacity

1,800 kW

application record

Screen D result

PASS: 1,800 kW ≤ 2,400 kW

SRP-3 §4

The source, the window and the cadence. SRP-3 §5(a) makes the AMI interval data the sole authoritative source for this determination, and the standing export for ALN-214 is the record: feeder-aggregate interval load in kW, fifteen-minute cadence, 35,040 values running from 07/01/2024 00:00 through 06/30/2025 23:45, continuous and complete, with no gap and no missing value. The data window used is the twelve complete months 07/01/2024 to 06/30/2025. Twenty-four intervals a day have a start time between 10:00 and 15:45, which gives the 8,760 fifteen-minute intervals evaluated.

Interval exclusions and their reason. None applied. No interval was excluded, and the reason is that no feeder outage, load-transfer or planned-switching record for ALN-214 exists in the DER Program record for the window, so no interval is identified as affected by one. The exclusion is recorded here because SRP-3 §6(c) requires any exclusions with reason, and a review that applied none says so and gives the reason.

The minimum is not a lone spike on a flat record. The next-lowest reading in the window is 2,418 kW at 10:00 on 01/25/2025, then 2,433 kW at 10:00 on 01/26/2025; only three of the 8,760 intervals fall below 2,500 kW. Setting the 02/12/2025 reading aside would not change the result, because the requested 1,800 kW sits below every interval in the window, not only below the minimum — no day in the twelve months has a daytime-window minimum at or below 1,800 kW. The seasonal shape is the usual one: the window minimum falls to 2,400 kW in February 2025 and is 4,996 kW in July 2024.

Neither boundary reading changes it. Reading the upper bound the other way, with start times 10:00 through 16:00 taken as inside the window, adds the 365 readings at 16:00 and gives 9,125 intervals evaluated; the lowest of those 365 readings is 3,080 kW, so the minimum stays at 2,400 kW. Nor does the cadence carry the result: sampling the same export on the hour would give a window minimum of 2,418 kW, still far above the requested 1,800 kW.

The absolute overnight minimum. The lowest reading anywhere in the export is 1,100 kW, at 03:45 on 03/23/2025. That is the absolute overnight minimum, which SRP-3 §4 excludes by name, and so it is not a valid Screen D basis. It is also the figure the earlier memo used (section 4 below). The same day's daytime readings are ordinary — 3,689 kW at 11:15 on 03/23/2025 — which is the point of the exclusion: an overnight trough is not the load a solar facility's output coincides with.

Determination: PASS. The daytime-window minimum load for ALN-214 under SRP-3 §4 is 2,400 kW, determined on the AMI interval record over the twelve complete months 07/01/2024 to 06/30/2025 across 8,760 fifteen-minute intervals, with no interval exclusions. The requested 1,800 kW is below it, and Screen D passes.

3.5  Screen E — Reverse power flow at substation

Requirement (SRP-3 §3, Screen E). “Where the aggregate DER on the subject feeder exceeds the substation transformer's certified reverse-power capability, or where reverse-power protective settings are not in place, the application shall be routed to study.”

What the record holds. No certified reverse-power capability for the Larkspur Ridge substation transformer is on file. The record holds no transformer nameplate, no certified reverse-power rating, and no reverse-power study for that substation or for ALN-214; the hosting-capacity map identifies Larkspur Ridge as the substation and 12.47 kV as the nominal voltage, and prints nothing about the transformer. No reverse-power protective settings for the substation are on file either, so the second leg of the test — settings in place — cannot be closed from the record. The map's 4.9 MW is the map's published available capacity and is used in no screen here; it is not a reverse-power capability of the transformer.

Determination: INCONCLUSIVE. Screen E is inconclusive. The test compares aggregate DER on the feeder against a certified reverse-power capability, and no such capability is on file for the Larkspur Ridge transformer; with no settings on file, the alternative leg of the test is equally unanswerable. SRP-3 §2 applies; see section 5.

4.  The memo of 06/25/2025 — what changes and what does not

The memo of record reached a study agreement through a Screen D failure. On the AMI record Screen D passes; the study agreement is reached instead through four inconclusive screens. The comparison, figure by figure:

Item

IS-BRAM-2025-06-25 (06/25/2025)

This memo (07/14/2025)

Screens addressed

Screen D only

Screens A, B, C, D and E

Data window stated

06/24/2024 – 06/24/2025

07/01/2024 – 06/30/2025

Interval cadence stated

Sixty (60) minutes

15 minutes

Intervals evaluated

Not stated

8,760 (9,125 with the 16:00 start read as inside the window)

Minimum load stated

1.1 MW

2,400 kW

Basis of that minimum

The lowest reading in the export — the 03:45 reading of 03/23/2025

The SRP-3 §4 daytime window, start times 10:00 through 15:45

Screen D result

FAILED (1.8 MW > 1.1 MW)

PASS (1,800 kW ≤ 2,400 kW)

Disposition

Study Agreement required

Advanced to a study agreement — on SRP-3 §2, not on a Screen D failure

The 1,100 kW reading is in the export: 03:45 on 03/23/2025. The figure is real; the basis is not. The earlier memo's 1,100 kW minimum is the absolute overnight minimum that SRP-3 §4 excludes by name, and so it is not a valid Screen D basis — the same exclusion that makes the figure unusable makes the screen fail on paper and pass on the record. The window and cadence the earlier memo states (06/24/2024 – 06/24/2025; sixty minutes) do not match the standing export (07/01/2024 – 06/30/2025; fifteen minutes), and because the memo stated no count of intervals evaluated, its figure cannot be reconstructed from the memo itself.

What does not change is the application's route. The earlier memo reached a study agreement; this memo reaches a study agreement. The difference is the ground under it: four screens are inconclusive on the record, and SRP-3 §2 sends the application to a study agreement for that reason.

5.  Disposition

The rule. SRP-3 §2 provides: “Where a supplemental-review screen result is inconclusive on the record before the engineer, the application shall be advanced to a study agreement rather than passed or failed under this procedure.”

The call. Screens A, B, C and E are inconclusive on the record at 07/14/2025; Screen D passes on recomputation. Under SRP-3 §2, the Bramwick Commons Solar application is advanced to a study agreement.

The earlier disposition stands. The disposition reached in IS-BRAM-2025-06-25 stands: the application does not proceed on the supplemental-review track, and a study agreement is required before it advances. What does not stand is the Screen D basis given for it. The earlier memo recorded a Screen D failure on a 1,100 kW minimum; on the AMI record Screen D passes at 2,400 kW, and the study agreement is required because the other four screens are inconclusive on the record, not because Screen D fails. The outcome holds; only its ground moves.

The other two dispositions are not available. SRP-3 §6(d) allows three. Advancing without a study agreement is not available while four screens are inconclusive. Returning the application to the applicant with a specified deficiency is not available either: nothing this application omitted is what blocks Screens A, B, C and E — the feeder record is what is missing, and the applicant cannot cure the Company's record.

Scope of the study agreement. It should cover the four conditions the record cannot close: the feeder's aggregate DER penetration, once a threshold exists to test it against; the feeder planning study and the voltage-rise and flicker analysis for ALN-214; short-circuit contribution and protection coordination against DPS-07; and the Larkspur Ridge transformer's certified reverse-power capability and reverse-power settings.

Filing. This memo is filed to the interconnection application record within five business days of screen completion (§6), and the memo it supersedes is flagged in the file with a cross-reference to this memo added to its cover (§6(f)).

6.  Reviewer signature and date

Reviewer (Interconnection Studies, DER Program Office): ________________________________      Date: 07/14/2025

Manager review (Interconnection Studies): ________________________________      Date: ______________

7.  Record of the superseded memo

Superseded memo: Bramwick_Commons_Screening_Memo_v1.docx — doc no. IS-BRAM-2025-06-25, dated 06/25/2025, countersigned by the Manager, Interconnection Studies on 06/25/2025. This memo supersedes it, as SRP-3 §6(f) requires; the superseded memo is flagged in the file and a cross-reference to this memo is added to its cover. Its disposition stands, on the ground given in section 5 of this memo.

What the memo it supersedes lacked. It carried no per-screen determinations beyond Screen D — Screens A, B, C and E were not addressed at all — where SRP-3 §6(b) requires the screens applied and the pass/fail determination for each. It stated no count of the intervals evaluated, which SRP-3 §6(c) requires, and its data window and cadence do not match the standing AMI export. And it carried the absolute overnight minimum into Screen D, which SRP-3 §4 excludes by name.

Those gaps are why the earlier determination could not simply be left in the file: as it stood, the file recorded a failed screen on a figure the procedure forbids, and recorded nothing at all on the four screens that decide what happens to this application.
```
