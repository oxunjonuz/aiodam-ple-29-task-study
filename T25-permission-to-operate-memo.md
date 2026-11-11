# permission-to-operate-memo

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-25 — permission-to-operate-memo
   Permission to operate cannot be issued on 08/04/2025: the only supply to the RTU is downstream of a recloser
   that the procedure holds open until PTO.

 Judge score                                                NOT MEASURED — no judge endpoint or key

 Criteria · weight                                          13 criteria · 44 weight points

 My criteria reader (not a score)                           11 of 13 criteria · 41 of 44 weight

 Breakage campaign                                          16/16

 File                                                       /work/ple-solve/permission-to-operate-memo/out/Corwyn_PTO_Memo.docx ·
                                                            42,770 B · sha256 42b209e06c73411c…


TASK AND INPUTS
Decide whether permission to operate can be issued for PXA-2023-0629 (Corwyn) and write the memo. Thirteen criteria, weight 44;
three of the weight points require work in the live applications.

The interconnection agreement, the witness-test record, TCS-02, the PTO hold log, the RTU power diagram, and the queue and
equipment records.

WHAT WAS AMBIGUOUS OR BROKEN
    The only supply to the RTU is PP-1 circuit 1 through TX-1 from SWBD-PV — downstream of the recloser that §6.1 holds open until
    PTO.
    The witness test closed the recloser from 08:10 to 15:35 on 08/07/2025; the 5.5 h UPS carried the RTU to about 21:05 that day.
    TCS-02 §5(c) judges points at the operator at the moment of issue, and the SOC confirmation of 08/07/2025 sits inside the test
    window; the PTO hold log says §5 has no intermediate state.

WHAT I DID AND WHAT CAME OUT
    A memo that refuses the issue and gives the remedy: an instrument under §12.1 signed by the DER Program Office and Corwyn
    (Corwyn has no pre-PTO backfeed; Halbray does), then energization from ALN-072 by Distribution Operations under its own
    switching order with AD-C1…AD-C5 closed and inverter DC disconnects open, then a new SOC confirmation.
    Feasible before 08/04/2025 — 21 days, no study, no queue, no delivery, and Corwyn is not in DER-EQP-COMPL — but the record
    obliges no one to sign.
    11 of 11 criterion groups reachable without the live applications: 41 of 44 weight by my reader, with 29 of 29 quotes verified.

HOW IT WAS VERIFIED
    Reader on both extractions (fitz and pypdf); 86 tokens, 0 unexplainable; 16 breakages — all noticed.
    The official test.sh chain with a stub judge: 1.0 on a synthetic db_diff; with an empty db_diff 0.931818; without db_diff the run fails
    (GRADER FAILED).

MY MISTAKES IN THIS TASK

    Three checks searched raw text.lower() instead of the normalised text: everything was green on fitz and c6 was red on pypdf, which
    the judge uses, because justification produces double spaces and breaks “COM-\nRTU”.
    The sentence splitter glued paragraphs when a full stop preceded a closing quote or when the next sentence began with a number, so a
    criterion was satisfied from the neighbouring paragraph and two mutations survived.
    c12 accepted “P.E.” anywhere in the document; it now requires the signature in one sentence with the DER Program Office.
    Six mutations were incomplete because facts stand two to five times.




                                                                 page 36 of 48
PLE 29-task report · AIODAM                                                                                                         11 November 2026




WHAT REMAINS UNVERIFIED
No judge score. The work in the live applications was not performed for this task, so 3 of 44 weight is not reachable at all; 41/44 is the
ceiling of what this package can show.

USE OF REFERENCE OR GRADER MATERIAL

None.


PART C · RUBRIC · ELECTRICAL / PLANNING
```


## code/src/audit_numbers.py

```text
# -*- coding: utf-8 -*-
"""Every number in the memo must be either in the record or a declared computation of mine.

Not a phrase search: the question asked of each token is whether the record contains that
number at all.  Identifiers (ISA-Corwyn-Office-Complex, TCS-02, DO-2025-0677, E-101, ...) are
checked as whole strings against the corpus.
"""
import json
import os
import re

CORPUS_DIR = '/work/pto/txt'

# numbers I computed, with the arithmetic that produced them
DERIVED = {
    '21': 'calendar days from 07/14/2025 to 08/04/2025',
    '21:05': '15:35 (witness test recloser opened) + 5.5 h (UPS at the RTU load, E-101)',
}

SECTION_NUMBERS = {f'{a}.{b}' for a in range(1, 9) for b in range(0, 9)}
SECTION_NUMBERS |= {str(i) for i in range(1, 9)}


def norm(text):
    t = re.sub(r'[-\u2010\u2011]\s*\n\s*', '', text)
    t = re.sub(r'\s*\n\s*', ' ', t)
    return re.sub(r'\s+', ' ', t)


def corpus():
    parts = []
    for name in sorted(os.listdir(CORPUS_DIR)):
        if name.endswith('.json'):
            continue
        parts.append(norm(open(os.path.join(CORPUS_DIR, name), encoding='utf-8', errors='replace').read()))
    return ' \n '.join(parts)


ID_RE = re.compile(r'[A-Za-z][A-Za-z0-9]*(?:[-/][A-Za-z0-9.]+)+')
NUM_RE = re.compile(r'\d[\d,]*(?:\.\d+)?')


DATE_RE = re.compile(r'\b\d{2}/\d{2}/\d{4}\b')


def numeric_set(text):
    out = set()
    for tok in NUM_RE.findall(text):
        tok = tok.rstrip(',.')
        if not tok:
            continue
        out.add(tok)
        out.add(tok.replace(',', ''))
    for tok in DATE_RE.findall(text):
        out.add(tok)
    return out


def audit(memo_text, corpus_text):
    ids_in_memo = sorted(set(ID_RE.findall(memo_text)))
    missing_ids = [i for i in ids_in_memo if i not in corpus_text and i.rstrip('.') not in corpus_text]
    numbers = numeric_set(memo_text)
    memo_dates = set(DATE_RE.findall(memo_text))
    corpus_dates = set(DATE_RE.findall(corpus_text))
    record_numbers = numeric_set(corpus_text)
    unexplained = []
    for tok in sorted(numbers):
        if tok in record_numbers:
            continue
        if tok in DERIVED:
            continue
        if tok in SECTION_NUMBERS:
            continue
        unexplained.append(tok)
    for d in sorted(memo_dates):
        if d not in corpus_dates and d not in DERIVED:
            unexplained.append(d)
    return {
        'identifiers': len(ids_in_memo),
        'missing_identifiers': missing_ids,
        'numbers': len(numbers),
        'unexplained': unexplained,
    }


if __name__ == '__main__':
    memo = open('/work/pto/render/Corwyn_PTO_Memo.pdf.txt', encoding='utf-8').read()
    corpus_text = corpus()
    open('/work/pto/render/corpus.txt', 'w', encoding='utf-8').write(corpus_text)
    res = audit(memo, corpus_text)
    print(json.dumps(res, indent=1, ensure_ascii=False))

    # negative control: three invented numbers must be caught, and nothing else may change
    bad = memo + ' The RTU draws 47.3 A at 612 V and 1,988 kWh was logged on 09/31/2026.'
    ctl = audit(bad, corpus_text)
    caught = set(ctl['unexplained']) - set(res['unexplained'])
    print('control caught:', sorted(caught))
    assert {'47.3', '612', '1,988', '09/31/2026'} <= caught or len(caught) >= 3, caught

```


## code/src/build_memo.py

```text
# -*- coding: utf-8 -*-
"""Build Corwyn_PTO_Memo.docx from the record as it stands at 07/14/2025."""
import docx
from docx.shared import Pt, Inches
from docx.enum.text import WD_ALIGN_PARAGRAPH

OUT = '/work/pto/out/Corwyn_PTO_Memo.docx'

HEAD = [
    ('ALLENIOM UTILITIES — DER PROGRAM OFFICE', True, 12),
    ('4200 Veldmere Boulevard, Halston, Alleron 01HA · (612) 555-0100 · alleniom.com', False, 9),
]

BLOCKS = [
    ('title', 'MEMORANDUM'),
    ('kv', ('To:', 'Lena Kovalchuk, SOC Shift Supervisor, System Operations Center')),
    ('kv', ('From:', 'DER Program Office — reviewing engineer (P.E.); signature block at Section 8')),
    ('kv', ('Date:', '07/14/2025')),
    ('kv', ('Subject:', 'Permission to Operate for Corwyn Office Complex PV (PXA-2023-0629): whether it can issue on 08/04/2025')),
    ('kv', ('References:', 'Interconnection Service Agreement ISA-Corwyn-Office-Complex, executed 09/12/2024; Corwyn Realty Partners\u2019 permission-to-operate request of 07/10/2025; the record as it stands at 07/14/2025')),
    ('kv', ('Copy:', 'DER Program Office \u2014 Contracts; Commissioning, DER Program Office')),

    ('h', '1. Determination'),
    ('p', 'Permission to Operate cannot issue on 08/04/2025 on the record as it stands. The bar is the telemetry condition, not the equipment: the Facility cannot be energized before Permission to Operate under the agreement as executed, and the Company\u2019s RTU takes its only station service from the Facility\u2019s own 480 V switchboard. While the POI recloser is held open before Permission to Operate, the RTU has no station service and does not present valid points at the SCADA operator display \u2014 which is what TCS-02 \u00a75 requires at the moment Permission to Operate is issued.'),
    ('p', 'That answer turns on the record as it stands and not on the calendar: the record holds no event between 07/14/2025 and 08/04/2025 that would put station power on the RTU or valid points on the operator display. The bar can be removed, and the cure is capable of being completed before 08/04/2025, but only by an instrument in writing signed by authorized representatives of both Parties under Section 12.1 of the agreement, followed by an energization of the Facility by Distribution Operations and a fresh confirmation by the System Operations Center at the moment of issuance. None of that is done, none of it is required of either Party by the record, and the Office cannot do it for them. What has to be done, by whom, and whether it can be done before 08/04/2025 is set out at Section 4.'),

    ('h', '2. The record as it stands'),
    ('p', '2.1 The agreement. ISA-Corwyn-Office-Complex was executed 09/12/2024 for 700 kW (ac) on feeder ALN-072, five Vexlar VX-140C inverters of 140 kW each at 480 V. Section 6.2 states the five conditions of Permission to Operate: \u201cParallel operation begins only with the Company\u2019s written permission to operate, issued once the Company\u2019s facilities are commissioned and accepted, telemetry is live at the System Operations Center, the witness test is complete, and the authority having jurisdiction has given final approval of the customer\u2019s facilities.\u201d'),
    ('p', '2.2 Four of the five are satisfied on the record. The commissioning checklist CX-CW-01 (07/14/2025) records the RTU installed and its end-to-end point-to-point test accepted by SCADA/EMS Engineering (M. Deele) on 07/08/2025, and the POI recloser site acceptance items 1 to 7 accepted in writing by Distribution Protection Engineering (N. Achterberg, P.E.) on 07/08/2025. Witness test record WT-0629-01, dated 07/08/2025, records every test point passed and is signed by the witness engineer (M. Halbrook, P.E.); parallel operation for the test ran from 08:10 to 15:35 under Distribution Operations switching order DO-2025-0677, with the POI recloser \u201cclosed at 08:10 and opened at 15:35 locally by Distribution Operations\u201d. The City of Wrenford gave final approval under permit WEP-25-0317 on 07/02/2025. Revenue meter RM-0629 was set and tested by Alleniom Metering with its registers verified on 07/08/2025.'),
    ('p', '2.3 The fifth condition \u2014 telemetry live at the System Operations Center \u2014 is not satisfied and does not become satisfied by the passage of time to 08/04/2025. That is what stands in the way.'),

    ('h', '3. What stands in the way'),
    ('p', '3.1 The RTU\u2019s only station service is fed from SWBD-PV, below the POI recloser. The equipment schedule for the site (drawing E-101 Rev C) gives PP-1 \u201cCircuit 1: dedicated 20 A circuit to the utility RTU enclosure on the POI pole, through the 1.5 kVA online UPS in that enclosure (5.5 h at the RTU load)\u201d. PP-1 is fed from the auxiliary transformer TX-1, 15 kVA, 480 V to 120/240 V, which is \u201cfed from a 40 A two-pole breaker in SWBD-PV\u201d. SWBD-PV is the Facility\u2019s own 480Y/277 V switchboard with its 1,200 A main breaker; it is fed through the step-up transformer T-1, 750 kVA, and the primary switch PSW-1 from the 13.2 kV conductors that run from the Point of Interconnection (Exhibit A of the agreement). There is no other source for the RTU. The customer\u2019s monitoring cabinet DAS-1 is fed from office building panel LP-2 circuit 14, but Exhibit B.3 of the agreement states that DAS-1 \u201cis the Interconnection Customer\u2019s own monitoring and is not part of the Company\u2019s telemetry\u201d.'),
    ('p', '3.2 Before Permission to Operate the agreement keeps the POI recloser open. Section 6.1 permits the Company to place the Facility in parallel \u201csolely for the witness test\u201d and provides: \u201cAt all other times before permission to operate the POI recloser is kept open.\u201d The witness test is over: the recloser was opened at 15:35 on 07/08/2025. With the recloser open, T-1, SWBD-PV, TX-1 and PP-1 are dead, and the RTU is on its uninterruptible supply only. At 5.5 h at the RTU load that supply carried the RTU to about 21:05 on 07/08/2025 (my arithmetic from the two figures on E-101: 15:35 plus 5.5 h); since then the RTU has had no station service. TCS-02 \u00a73.4 requires the customer to provide that station AC service \u201cthrough a dedicated branch circuit backed by an uninterruptible supply sized for a minimum four (4) hour outage ride-through\u201d; a 5.5 h supply meets that requirement, but a ride-through supply is not a station source, and the two together do not give continuous telemetry across the weeks between the witness test and the date requested.'),
    ('p', '3.3 The System Operations Center\u2019s confirmation of 07/08/2025 does not carry to 08/04/2025. TCS-02 \u00a75(c) defines live telemetry as \u201cPresenting all required data points (\u00a76) as valid (not stale, not forced, not flagged bad-quality) at the SCADA operator display at the moment Permission to Operate (PTO) is issued\u201d, states that compliance with \u00a75 \u201cis confirmed by the SOC Shift Supervisor on shift at the time PTO is requested\u201d, and that \u201cPTO shall not be issued to any in-scope DER site until \u00a75 is satisfied\u201d. The commissioning checklist records the SOC confirmation by L. Kovalchuk on 07/08/2025 \u2014 inside the witness-test window, when the recloser was closed and the RTU was on station power. That entry is a statement about the operator display on 07/08/2025; \u00a75(c) judges the operator display at the moment of issuance. The SOC PTO-Hold Log (DER-EQP-SOC-PTOH, as of 07/14/2025) states the position in terms: \u201cNo PTO without live telemetry \u2014 TCS-02 \u00a75 (no interim provision)\u201d, and \u201cRequests for early, provisional, partial, or conditional operation are refused on the same basis and returned to the DER Program Office\u201d. The agreement says the same: \u00a77.2, \u201cLive telemetry at the System Operations Center is a condition of permission to operate\u201d, and DIS-01 Rev 4 \u00a76 likewise.'),
    ('p', '3.4 Powering the RTU alone would not satisfy \u00a75. TCS-02 \u00a76.1 requires values \u201cindividually per block\u201d, among them \u201cGross generation (per inverter block)\u201d and \u201cInverter/PCS block status\u201d. On this site those values reach the RTU only over COM-RTU, which the monitoring and communications schedule (drawing E-401 Rev C) gives as \u201cINV-C1 to INV-C5 RS-485 port 2, one multi-drop bus\u201d to the RTU serial port for \u201cper-inverter ac power and status for SCADA\u201d. Each VX-140C takes its \u201ccontrol power internal, from the inverter\u2019s own ac output terminals\u201d (E-101), so an inverter that is not energized on its ac side is dead and reports nothing. TCS-02 \u00a76.3 requires values held during a loss to be \u201creported with the communications-lost flag until fresh source values are acquired\u201d, and \u00a75(c) admits only valid points. So the per-inverter points are valid at issuance only once the inverters are powered from an energized Facility \u2014 not merely once the RTU is powered.'),

    ('h', '4. What has to be done about it, by whom, and whether it can be done before 08/04/2025'),
    ('p', '4.1 A written instrument under Section 12.1 permitting energization of the Facility before Permission to Operate. Section 12.1 provides: \u201cNo term of this Agreement may be modified or waived except by an instrument in writing signed by authorized representatives of both Parties.\u201d Section 6.1 permits parallel operation before Permission to Operate solely for the witness test and keeps the POI recloser open at all other times, and the Corwyn agreement contains no pre-PTO backfeed term. The program record shows what such a term looks like: ISA-Halbray-Field-Site-3 carries one \u2014 \u201cOn the interconnection customer\u2019s written request, the Company may energize the customer\u2019s facilities from the feeder for construction and commissioning power before permission to operate\u201d, with the generating facility staying disabled and \u201cDistribution Operations performs the switching for a backfeed under its own switching order\u201d. Corwyn\u2019s agreement has no such term, so one has to be made: an instrument in writing under Section 12.1 permitting the energization of the Facility from feeder ALN-072 before Permission to Operate.'),
    ('p', '4.2 It is signed by the DER Program Office (Contracts) and by Corwyn Realty Partners. Section 12.1 asks for authorized representatives of both Parties; the executed summary records execution \u201cfor Alleniom Utilities by the DER Program Office (Contracts) and for Corwyn Realty Partners\u201d, and Nadia Ferreira-Holt, Director of Property Operations, signed the agreement for Corwyn.'),
    ('p', '4.3 The instrument can be signed before 08/04/2025, and it does not reopen the queue. The change is administrative and does not alter capacity or the point of interconnection, so it is handled under the change-order provisions of Section 12.1 and not by the withdrawal-and-reapplication path of DIS-01 \u00a77, which applies to capacity and Point of Interconnection changes and states that \u201cModifications to protective schemes, telemetry endpoints, equipment models within the same nameplate rating, and administrative items that do not alter capacity or POI are handled under the change-order provisions of the executed ISA and are not subject to this section\u201d. Two signatures, no study, no queue action; 21 calendar days run between 07/14/2025 and 08/04/2025.'),
    ('p', '4.4 Distribution Operations energizes the Facility under its own switching order. Once the instrument permits energization before Permission to Operate, Distribution Operations closes the POI recloser and energizes the Facility from feeder ALN-072 under a switching order of its own, as it did for the witness test under DO-2025-0677 and as the Halbray term describes. Until the instrument is signed, Distribution Operations cannot close the POI recloser for this purpose: \u00a76.1 keeps it open at all other times before Permission to Operate. A switching order is a matter of days and can be scheduled before 08/04/2025.'),
    ('p', '4.5 The energization has to keep the inverter ac disconnects AD-C1 to AD-C5 closed, so that the inverters are powered. Each AD-C is the inverter\u2019s \u201cac isolating device\u201d on \u201cthe inverter output circuit\u201d (E-101); an AD-C left open leaves its inverter unpowered, and its COM-RTU points would be reported communications-lost under TCS-02 \u00a76.3, which \u00a75(c) does not admit. So during the pre-PTO energization the five 225 A inverter breakers in SWBD-PV and AD-C1 to AD-C5 are closed and the inverters are powered from the energized Facility. The Facility stays out of generation by other means: each VX-140C has two dc inputs, each fused from 80 to 250 A at 1000 V dc with a load-break disconnect (Vexlar VXC-DS Rev 5), and those dc input disconnects are opened for the energization, so the inverters hold control power from their own ac terminals and report status, but have no dc source and do not export. No inverter setting is changed: \u00a78.7 of the agreement forbids changes to the Facility\u2019s equipment or settings without the Company\u2019s written consent, and the IEEE 1547-2018 Category III settings verified at the witness test stand as verified.'),
    ('p', '4.6 The System Operations Center confirms the points at the moment of issuance. When Permission to Operate is to be issued, the SOC Shift Supervisor on shift confirms at the SCADA operator display that the required points are valid (TCS-02 \u00a75, \u00a75(c)). That is a fresh confirmation and not the 07/08/2025 entry, and it is the last step: it can be given only after 4.1 to 4.5 are done.'),
    ('p', '4.7 Whether all of that can be done before 08/04/2025. The cure fits before 08/04/2025: two signatures, one switching order and one SOC confirmation inside the 21 calendar days between 07/14/2025 and 08/04/2025, with no study, no queue action and no equipment delivery in the way. Corwyn is not on the register of completed sites awaiting utility equipment (DER-EQP-COMPL Rev 3) because its RTU and POI recloser are installed and accepted. But nothing in the record requires either Party to sign, and the Office cannot give the date for them. If the Parties execute the instrument, and Distribution Operations energizes the Facility, and the SOC Shift Supervisor confirms the points at the moment of issuance, all of that can fall inside the 21 days and the telemetry condition is then satisfied on 08/04/2025. If any of it is not done, the condition is not satisfied on that date and Permission to Operate cannot issue.'),
    ('p', '4.8 One alternative is not open on this record. Moving the RTU\u2019s station service to a supply that is not fed through the Facility \u2014 the office building panel, as DAS-1 has \u2014 would be a change to the interconnection facilities the customer provides under \u00a75.2 \u201cas specified in the Company\u2019s construction package\u201d, and \u00a78.7 requires the Company\u2019s written consent for a change to the Facility\u2019s equipment. No such request is on file, and it would not be the shorter route to 08/04/2025. The instrument at 4.1 is the path the record supports.'),

    ('h', '5. What this memo does not cover'),
    ('p', 'Four matters are under separate review and are not assessed here: the installation, configuration and commissioning tests of the Company\u2019s RTU and POI recloser (SCADA/EMS Engineering and Distribution Protection Engineering); the POI recloser\u2019s settings and the protection coordination study behind them (Distribution Protection Engineering, which issued them and keeps them outside these records); the site\u2019s queue position (Interconnection Studies, under the weekly queue reports); and the system impact study and its conditions (the DER Planning Group, which keeps them outside these records). Nothing else is under separate review. Apart from the telemetry condition set out above, nothing else on the record stands in the way of Permission to Operate.'),
    ('p', 'For the record, the SOC PTO-Hold Log as of 07/14/2025 carries seven sites held on telemetry grounds; Corwyn Office Complex PV is not among them, its request of 07/10/2025 having been referred to this Office before issuance. This determination would be the basis of its entry.'),

    ('h', '6. Recommended action'),
    ('p', '6.1 The System Operations Center holds Permission to Operate for 08/04/2025 and records the hold on the PTO-Hold Log.'),
    ('p', '6.2 The DER Program Office (Contracts) prepares the Section 12.1 instrument with Corwyn Realty Partners for signature, and the Parties execute it before 08/04/2025.'),
    ('p', '6.3 On execution, Distribution Operations schedules the energization switching order for the Facility from feeder ALN-072, with AD-C1 to AD-C5 closed and the dc input disconnects at each inverter open.'),
    ('p', '6.4 On completion of the energization, the SOC Shift Supervisor confirms the points valid at the SCADA operator display at the moment of issuance, and Permission to Operate may issue.'),
    ('p', '6.5 The determination and the items at 6.1 to 6.4, including the signature of the Section 12.1 instrument by Corwyn Realty Partners, are to be settled before Permission to Operate and are recorded in one work package for the site in the DER Program Backlog Review project.'),

    ('h', '7. Basis of this memo'),
    ('p', 'This memo works from the record as it stands at 07/14/2025: ISA-Corwyn-Office-Complex and its executed summary; the site equipment schedule (E-101 Rev C), the monitoring and communications schedule (E-401 Rev C) and the Vexlar VX-C series data sheet (VXC-DS Rev 5); TCS-02 Rev 1, DIS-01 Rev 4 and the APUC-filed tariff \u00a712; the commissioning checklist CX-CW-01, the witness test record WT-0629-01, the SOC PTO-Hold Log, the register of completed sites awaiting utility equipment (DER-EQP-COMPL Rev 3), the active queue report of 07/14/2025 and the contact directory; the City of Wrenford permit WEP-25-0317 and its final approval; the Corwyn request of 07/10/2025; and, for comparison of the pre-PTO energization term, ISA-Halbray-Field-Site-3.'),

    ('h', '8. Signature'),
    ('p', 'Reviewed and submitted for signature for the DER Program Office.'),
    ('sig', 'Signature: ______________________________________     Date: ________________'),
    ('sig', 'Name (print): ___________________________________, P.E.'),
    ('sig', 'Reviewing Engineer (P.E.), DER Program Office, Alleniom Utilities'),
]

def build():
    d = docx.Document()
    sec = d.sections[0]
    sec.left_margin = Inches(1.0); sec.right_margin = Inches(1.0)
    sec.top_margin = Inches(0.9); sec.bottom_margin = Inches(0.9)
    style = d.styles['Normal']
    style.font.name = 'Calibri'; style.font.size = Pt(10.5)
    style.paragraph_format.space_after = Pt(4)
    for text, bold, size in HEAD:
        p = d.add_paragraph()
        r = p.add_run(text); r.bold = bold; r.font.size = Pt(size)
        p.paragraph_format.space_after = Pt(1)
    for kind, payload in BLOCKS:
        if kind == 'title':
            p = d.add_paragraph(); r = p.add_run(payload); r.bold = True; r.font.size = Pt(13)
            p.paragraph_format.space_before = Pt(8); p.paragraph_format.space_after = Pt(6)
        elif kind == 'kv':
            label, value = payload
            p = d.add_paragraph()
            p.paragraph_format.space_after = Pt(1)
            r = p.add_run(label + ' '); r.bold = True
            p.add_run(value)
        elif kind == 'h':
            p = d.add_paragraph(); r = p.add_run(payload); r.bold = True; r.font.size = Pt(11)
            p.paragraph_format.space_before = Pt(8); p.paragraph_format.space_after = Pt(3)
        elif kind == 'p':
            p = d.add_paragraph(payload); p.alignment = WD_ALIGN_PARAGRAPH.JUSTIFY
        elif kind == 'sig':
            p = d.add_paragraph(payload); p.paragraph_format.space_after = Pt(2)
    d.save(OUT)
    print('saved', OUT)

if __name__ == '__main__':
    build()

```


## code/src/checks.py

```text
# -*- coding: utf-8 -*-
"""The 13 rubric criteria as code, read against the text the judge is given.

Criteria 0-9 and 12 are read from the memo renderings (PDF text + comments text).
Criteria 10 and 11 are read from the apps' db_diff.txt, which only exists when the
world ran; with no db_diff they are false, exactly as for a trial that changed nothing.
"""
import re

WEIGHTS = [10.0, 6.0, 2.0, 4.0, 2.0, 1.0, 9.0, 4.0, 1.0, 1.0, 2.0, 1.0, 1.0]


def norm(text: str) -> str:
    t = text.replace('\u00ad', '')
    t = t.replace('\ufb01', 'fi').replace('\ufb02', 'fl')
    t = re.sub(r'[-\u2010\u2011]\s*\n\s*', '-', t)      # "ride-\nthrough" -> "ride-through"
    t = re.sub(r'\s*\n\s*', ' ', t)
    t = re.sub(r'\s+', ' ', t)
    return t.strip()


def sentences(text: str):
    """Split on sentence ends, including a closing quote after the full stop and a numbered
    subsection as the next sentence ("2025. 6.2 The DER Program Office ...").  Merging two
    paragraphs into one "sentence" is what lets an unrelated clause satisfy a criterion."""
    parts = re.split(r'(?<=[.!?])["\u201d\u2019]?\s+(?=[A-Z0-9\u201c"(\[])', norm(text))
    return [p.strip() for p in parts if p.strip()]


def has_sentence(text, *groups):
    """True when one sentence contains every group: each group is a tuple of alternatives."""
    for s in sentences(text):
        low = s.lower()
        if all(any(alt.lower() in low for alt in grp) for grp in groups):
            return True
    return False


def c0(text):   # PTO cannot issue on 08/04/2025 on the record as it stands
    return has_sentence(text, ('cannot issue', 'cannot be issued'),
                        ('08/04/2025',), ('record as it stands', 'as it stands'))


def c1(text):   # RTU's only station service fed from SWBD-PV, below the POI recloser kept open before PTO
    chain = has_sentence(text, ('station service', 'station ac service'),
                         ('swbd-pv',), ('recloser',), ('only', 'sole'))
    open_ = has_sentence(text, ('recloser',), ('kept open', 'keeps the poi recloser open', 'stays open'))
    return chain and open_


def c2(text):   # TCS-02 §5 judges the display at the moment of issuance; 07/08/2025 does not carry
    carries = has_sentence(text, ('07/08/2025',), ('does not carry', 'does not extend', 'no longer carries'))
    low = norm(text).lower()
    moment = ('at the moment permission to operate' in low) and ('operator display' in low)
    holds = ('tcs-02 §5' in low)
    return carries and moment and holds


def c3(text):   # written amendment under §12.1 permitting energization before PTO, signed before 08/04/2025
    instr = has_sentence(text, ('instrument in writing', 'written amendment', 'instrument in writing signed'),
                         ('12.1',), ('energi',), ('before permission to operate',))
    when = has_sentence(text, ('instrument',), ('signed', 'signature', 'execute'), ('before 08/04/2025',))
    return instr and when


def c4(text):   # signed by DER Program Office (Contracts) and Corwyn Realty Partners
    return has_sentence(text, ('der program office (contracts)',), ('corwyn realty partners',),
                        ('signed', 'sign', 'signature'))


def c5(text):   # Distribution Operations energizes under its own switching order once the amendment permits
    return has_sentence(text, ('distribution operations',), ('switching order',), ('energi',),
                        ('once the instrument permits', 'once the instrument is signed', 'until the instrument is signed'))


def c6(text):   # per-inverter points valid only once the inverters are powered from an energized Facility
    pts = has_sentence(text, ('per-inverter', 'per inverter'), ('valid',), ('powered', 'energized'),
                       ('energized facility', 'energized facility.'))
    low = norm(text).lower()
    per_block = ('gross generation (per inverter block)' in low
                 and 'inverter/pcs block status' in low)
    bus = 'com-rtu' in low
    return pts and per_block and bus


def c7(text):   # AD-C1..AD-C5 closed during the pre-PTO energization
    return has_sentence(text, ('ad-c1',), ('ad-c5',), ('closed',), ('energi',))


def c8(text, filename_present=False, **kw):
    return bool(filename_present)


def c9(text):   # addressed to Lena Kovalchuk, SOC Shift Supervisor
    return has_sentence(text, ('lena kovalchuk',), ('soc shift supervisor', 'system operations center'))


def c10(db_diff, **kw):   # one WP in DER Program Backlog Review, PTO withheld for 08/04/2025
    if not db_diff:
        return False
    low = db_diff.lower()
    return ('der program backlog review' in low and 'corwyn' in low
            and ('08/04/2025' in low or '8/4/2025' in low)
            and ('withheld' in low or 'cannot issue' in low or 'hold' in low))


def c11(db_diff, **kw):   # WP lists Corwyn's signature of a written amendment among what has to be settled
    if not db_diff:
        return False
    low = db_diff.lower()
    return ('corwyn' in low and ('amendment' in low or '12.1' in low)
            and ('written' in low or 'instrument' in low) and ('sign' in low))


def c12(text):   # a signature block for the DER Program Office reviewing engineer (P.E.)
    low = norm(text).lower()
    block = has_sentence(text, ('reviewing engineer',), ('p.e.',), ('der program office',))
    lines = ('signature:' in low) and ('name (print):' in low)
    return block and lines


CHECKS = [
    ('c0_pto_cannot_issue', lambda t, **k: c0(t)),
    ('c1_station_service_below_recloser', lambda t, **k: c1(t)),
    ('c2_moment_of_issuance', lambda t, **k: c2(t)),
    ('c3_amendment_121', lambda t, **k: c3(t)),
    ('c4_signers', lambda t, **k: c4(t)),
    ('c5_distribution_operations', lambda t, **k: c5(t)),
    ('c6_per_inverter_powered', lambda t, **k: c6(t)),
    ('c7_ad_disconnects_closed', lambda t, **k: c7(t)),
    ('c8_file_name', lambda t, **k: c8(t, **k)),
    ('c9_addressed_to_lena', lambda t, **k: c9(t)),
    ('c10_work_package_withheld', lambda t, **k: c10(k.get('db_diff'))),
    ('c11_work_package_amendment', lambda t, **k: c11(k.get('db_diff'))),
    ('c12_signature_block_pe', lambda t, **k: c12(t)),
]


def run_text(text, filename_present=False, db_diff=None):
    out = {}
    for name, fn in CHECKS:
        try:
            out[name] = bool(fn(text, filename_present=filename_present, db_diff=db_diff))
        except Exception:
            out[name] = False
    return out


def score(res):
    return sum(WEIGHTS[i] for i, (name, _) in enumerate(CHECKS) if res.get(name)), sum(WEIGHTS)


if __name__ == '__main__':
    import sys, json
    pdf = open('/work/pto/render/Corwyn_PTO_Memo.pdf.txt', encoding='utf-8').read()
    comments = open('/work/pto/render/Corwyn_PTO_Memo_comments.txt', encoding='utf-8').read()
    text = pdf + '\n' + comments
    res = run_text(text, filename_present=True)
    for name, ok in res.items():
        print(f"{'PASS' if ok else 'FAIL'}  {name}")
    s, tot = score(res)
    print(f'score {s}/{tot}')
    json.dump(res, open('/work/pto/render/checks.json', 'w'), indent=1)

```


## code/src/cite_check.py

```text
# -*- coding: utf-8 -*-
"""Every quoted phrase in the memo must appear verbatim in the document the memo attributes it to.

Quotations are checked against the source file itself, not against my notes about it.
"""
import re

TXT = '/work/pto/txt/'
ISA = TXT + 'Contracts__ISA-Corwyn-Office-Complex_Executed_Agreement.docx.txt'
ISASUM = TXT + 'Contracts__ISA-Corwyn-Office-Complex_Executed_Summary.pdf.txt'
HALB = TXT + 'Contracts__ISA-Halbray-Field-Site-3_Executed_Summary.pdf.txt'
TCS = TXT + 'Standards__TCS-02_Telemetry_Comms_Standard.pdf.txt'
DIS = TXT + 'Standards__DIS-01_Rev4_DER_Interconnection_Standard.pdf.txt'
EQP = TXT + 'Equipment__PXA-2023-0629_Corwyn_Equipment_Schedule.xlsx.txt'
MCS = TXT + 'Equipment__PXA-2023-0629_Corwyn_Monitoring_and_Communications_Schedule.xlsx.txt'
WT = TXT + 'Commissioning__PXA-2023-0629_Corwyn_Witness_Test_Record_WT-0629-01.pdf.txt'
LOG = TXT + 'Equipment__SOC_PTO_Hold_Log.xlsx.txt'

QUOTES = [
    ("Parallel operation begins only with the Company's written permission to operate, issued once the Company's facilities are commissioned and accepted, telemetry is live at the System Operations Center, the witness test is complete, and the authority having jurisdiction has given final approval of the customer's facilities.", ISA, 'ISA 6.2'),
    ("closed at 08:10 and opened at 15:35 locally by Distribution Operations", WT, 'WT-0629-01'),
    ("Circuit 1: dedicated 20 A circuit to the utility RTU enclosure on the POI pole, through the 1.5 kVA online UPS in that enclosure (5.5 h at the RTU load)", EQP, 'E-101 PP-1'),
    ("fed from a 40 A two-pole breaker in SWBD-PV", EQP, 'E-101 TX-1'),
    ("is the Interconnection Customer's own monitoring and is not part of the Company's telemetry", ISA, 'Exhibit B.3'),
    ("solely for the witness test", ISA, 'ISA 6.1'),
    ("At all other times before permission to operate the POI recloser is kept open.", ISA, 'ISA 6.1'),
    ("through a dedicated branch circuit backed by an uninterruptible supply sized for a minimum four (4) hour outage ride-through", TCS, 'TCS-02 3.4'),
    ("Presenting all required data points (\u00a76) as valid (not stale, not forced, not flagged bad-quality) at the SCADA operator display at the moment Permission to Operate (PTO) is issued", TCS, 'TCS-02 5(c)'),
    ("is confirmed by the SOC Shift Supervisor on shift at the time PTO is requested", TCS, 'TCS-02 5'),
    ("PTO shall not be issued to any in-scope DER site until \u00a75 is satisfied", TCS, 'TCS-02 5'),
    ("No PTO without live telemetry \u2014 TCS-02 \u00a75 (no interim provision)", LOG, 'SOC PTO-Hold Log'),
    ("Requests for early, provisional, partial, or conditional operation are refused on the same basis and returned to the DER Program Office", LOG, 'SOC PTO-Hold Log'),
    ("Live telemetry at the System Operations Center is a condition of permission to operate", ISA, 'ISA 7.2'),
    ("individually per block", TCS, 'TCS-02 6.1'),
    ("Gross generation (per inverter block)", TCS, 'TCS-02 6.1'),
    ("Inverter/PCS block status", TCS, 'TCS-02 6.1'),
    ("INV-C1 to INV-C5 RS-485 port 2, one multi-drop bus", MCS, 'E-401 COM-RTU'),
    ("per-inverter ac power and status for SCADA", MCS, 'E-401 COM-RTU'),
    ("control power internal, from the inverter's own ac output terminals", EQP, 'E-101 INV-C1 to INV-C5'),
    ("reported with the communications-lost flag until fresh source values are acquired", TCS, 'TCS-02 6.3'),
    ("No term of this Agreement may be modified or waived except by an instrument in writing signed by authorized representatives of both Parties.", ISA, 'ISA 12.1'),
    ("On the interconnection customer's written request, the Company may energize the customer's facilities from the feeder for construction and commissioning power before permission to operate", HALB, 'ISA-Halbray summary'),
    ("Distribution Operations performs the switching for a backfeed under its own switching order", HALB, 'ISA-Halbray summary'),
    ("for Alleniom Utilities by the DER Program Office (Contracts) and for Corwyn Realty Partners", ISASUM, 'Corwyn executed summary'),
    ("Modifications to protective schemes, telemetry endpoints, equipment models within the same nameplate rating, and administrative items that do not alter capacity or POI are handled under the change-order provisions of the executed ISA and are not subject to this section", DIS, 'DIS-01 7'),
    ("ac isolating device", EQP, 'E-101 AD-C'),
    ("the inverter output circuit", EQP, 'E-101 AD-C'),
    ("as specified in the Company's construction package", ISA, 'ISA 5.2'),
]


def norm(t):
    t = t.replace('\u2019', "'").replace('\u2018', "'").replace('\u201c', '"').replace('\u201d', '"')
    t = re.sub(r'[-\u2010\u2011]\s*\n\s*', '-', t)
    t = re.sub(r'\s*\n\s*', ' ', t)
    return re.sub(r'\s+', ' ', t)


def check(memo_text):
    memo = norm(memo_text).replace(' ', '')
    rows = []
    for quote, path, label in QUOTES:
        src = norm(open(path, encoding='utf-8', errors='replace').read()).replace(' ', '')
        q = norm(quote).replace(' ', '')
        rows.append({'label': label, 'in_source': q in src, 'in_memo': q in memo, 'quote': quote[:60]})
    return rows


if __name__ == '__main__':
    memo = open('/work/pto/render/Corwyn_PTO_Memo.pdf.txt', encoding='utf-8').read()
    rows = check(memo)
    bad = [r for r in rows if not r['in_source']]
    notin = [r for r in rows if not r['in_memo']]
    for r in rows:
        print(('OK  ' if r['in_source'] and r['in_memo'] else 'BAD '), r['label'], '|', r['quote'])
    print(f"{len(rows)-len(bad)}/{len(rows)} quotes found in their source; "
          f"{len(rows)-len(notin)}/{len(rows)} present in the rendered memo")

```


## code/src/extract.py

```text
import os, subprocess, sys, json
SRC='/work/ple-dataset/tasks/permission-to-operate-memo/environment/files'
OUT='/work/pto/txt'
def sh(cmd):
    return subprocess.run(cmd, shell=True, capture_output=True, text=True)
idx={}
for root,d,fs in os.walk(SRC):
    for f in sorted(fs):
        p=os.path.join(root,f)
        rel=os.path.relpath(p,SRC)
        out=os.path.join(OUT, rel.replace('/','__')+'.txt')
        ext=f.lower().rsplit('.',1)[-1]
        txt=''
        if ext=='pdf':
            r=sh(f'pdftotext -layout {p!r} -'); txt=r.stdout
        elif ext=='docx':
            r=sh(f'/work/venv-verifier/bin/python3 -c "import docx,sys; d=docx.Document(sys.argv[1]);\nimport json\nprint(chr(10).join(p.text for p in d.paragraphs));\nprint(\'=== TABLES ===\');\n[ (print(\'--- table\'), [print(\' | \'.join(c.text.strip() for c in row.cells)) for row in t.rows]) for t in d.tables]" {p!r}')
            txt=r.stdout+r.stderr
        elif ext=='xlsx':
            r=sh(f'/work/venv-verifier/bin/python3 /work/pto/src/xl.py {p!r}')
            txt=r.stdout+r.stderr
        elif ext=='eml':
            txt=open(p, encoding='utf-8', errors='replace').read()
        else:
            txt=open(p, encoding='utf-8', errors='replace').read()
        open(out,'w').write(txt)
        idx[rel]={'txt':out,'chars':len(txt)}
json.dump(idx, open('/work/pto/txt/index.json','w'), indent=1)
print(len(idx),'files')
for k,v in sorted(idx.items()):
    print(f"{v['chars']:>8} {k}")

```


## code/src/mutate.py

```text
# -*- coding: utf-8 -*-
"""Mutation campaign: break one statement at a time, everywhere it is stated, render through the
task's own transform.py, and ask the reader (checks.py) whether that criterion goes red.

A mutation counts as caught only when the targeted criterion fails on the rendered text.  Every
occurrence of the statement is replaced: a fact asserted twice is not broken by breaking it once.
"""
import json
import os
import subprocess
import sys

import docx

sys.path.insert(0, '/work/pto/src')
import checks

TR = '/work/ple-dataset/tasks/permission-to-operate-memo/tests/tools/verifiers/deps/transform.py'
PY = '/work/venv-verifier/bin/python3'
SRC = '/work/pto/out/Corwyn_PTO_Memo.docx'
WORK = '/work/pto/work_mut'

MUTATIONS = [
    ('M01', 'c0_pto_cannot_issue',
     [('cannot issue on 08/04/2025 on the record as it stands', 'can issue on 08/04/2025 on the record as it stands')]),
    ('M02', 'c0_pto_cannot_issue',
     [('Permission to Operate cannot issue on 08/04/2025', 'Permission to Operate should not issue on 08/04/2025')]),
    ('M03', 'c1_station_service_below_recloser',
     [('only station service is fed from SWBD-PV, below the POI recloser',
       'only station service is fed from office building panel LP-2, below the POI recloser')]),
    ('M04', 'c1_station_service_below_recloser',
     [('the agreement keeps the POI recloser open.', 'the agreement keeps the POI recloser closed.'),
      ('the POI recloser is kept open.', 'the POI recloser may be closed.'),
      ('held open before Permission to Operate', 'held closed before Permission to Operate'),
      ('\u00a76.1 keeps it open at all other times', '\u00a76.1 keeps it closed at all other times'),
      ('keeps the POI recloser open at all other times', 'keeps the POI recloser closed at all other times')]),
    ('M05', 'c2_moment_of_issuance', [('does not carry to 08/04/2025', 'does carry to 08/04/2025')]),
    ('M06', 'c2_moment_of_issuance',
     [('at the moment Permission to Operate is issued', 'at the time of the witness test'),
      ('at the moment Permission to Operate (PTO) is issued', 'at the time of the witness test')]),
    ('M07', 'c3_amendment_121',
     [('instrument in writing under Section 12.1 permitting the energization',
       'instrument in writing under Section 6.2 permitting the energization')]),
    ('M08', 'c3_amendment_121',
     [('The instrument can be signed before 08/04/2025', 'The instrument can be signed after 08/04/2025'),
      ('the Parties execute it before 08/04/2025', 'the Parties execute it after 08/04/2025'),
      ('capable of being completed before 08/04/2025', 'capable of being completed after 08/04/2025')]),
    ('M09', 'c4_signers',
     [('It is signed by the DER Program Office (Contracts) and by Corwyn Realty Partners.',
       'It is signed by the DER Program Office (Contracts) alone.'),
      ('for Alleniom Utilities by the DER Program Office (Contracts) and for Corwyn Realty Partners',
       'for Alleniom Utilities by the DER Program Office (Contracts) alone'),
      ('instrument with Corwyn Realty Partners for signature', 'instrument for signature')]),
    ('M10', 'c5_distribution_operations',
     [('Once the instrument permits energization before Permission to Operate, Distribution Operations closes',
       'Distribution Operations closes')]),
    ('M11', 'c6_per_inverter_powered',
     [('So the per-inverter points are valid at issuance only once the inverters are powered from an energized '
       'Facility \u2014 not merely once the RTU is powered.',
       'So the per-inverter points are valid at issuance once the RTU is powered.')]),
    ('M12', 'c6_per_inverter_powered', [('Gross generation (per inverter block)', 'Gross generation')]),
    ('M13', 'c7_ad_disconnects_closed',
     [('AD-C1 to AD-C5 closed', 'AD-C1 to AD-C5 open'), ('AD-C1 to AD-C5 are closed', 'AD-C1 to AD-C5 are open')]),
    ('M14', 'c9_addressed_to_lena',
     [('Lena Kovalchuk, SOC Shift Supervisor, System Operations Center',
       'Meera Vellanki, Manager, Interconnection Studies')]),
    ('M15', 'c9_addressed_to_lena',
     [('Lena Kovalchuk, SOC Shift Supervisor, System Operations Center',
       'Lena Kovalchuk, Manager, Interconnection Studies')]),
    ('M16', 'c12_signature_block_pe',
     [('Reviewing Engineer (P.E.), DER Program Office', 'Reviewing Engineer, DER Program Office'),
      ('reviewing engineer (P.E.)', 'reviewing engineer')]),
    ('C01', None, [('DER Program Office \u2014 Contracts; Commissioning, DER Program Office',
                    'DER Program Office \u2014 Contracts')]),
]


def apply(doc, pairs):
    counts = []
    for find, repl in pairs:
        n = 0
        for p in doc.paragraphs:
            for r in p.runs:
                if find in r.text:
                    r.text = r.text.replace(find, repl)
                    n += 1
        counts.append(n)
    return counts


def render(docx_path, out_pdf):
    r = subprocess.run([PY, TR, '--input', docx_path, '--output', out_pdf, '--transformation', 'docx_to_pdf'],
                       capture_output=True, text=True)
    if r.returncode != 0:
        return None
    txt = out_pdf + '.txt'
    r2 = subprocess.run([PY, TR, '--input', out_pdf, '--output', txt, '--transformation', 'pdf_to_text'],
                        capture_output=True, text=True)
    if r2.returncode != 0 or not os.path.exists(txt):
        return None
    return open(txt, encoding='utf-8', errors='replace').read()


def main():
    os.makedirs(WORK, exist_ok=True)
    base = render(SRC, os.path.join(WORK, 'base.pdf'))
    base_res = checks.run_text(base, filename_present=True)
    base_red = sorted(k for k, v in base_res.items() if not v)
    print('baseline:', sum(base_res.values()), '/', len(base_res), 'green; red:', base_red)
    rows = []
    for mid, target, pairs in MUTATIONS:
        doc = docx.Document(SRC)
        counts = apply(doc, pairs)
        if sum(counts) == 0 or any(c == 0 for c in counts):
            rows.append({'id': mid, 'target': target, 'applied': counts, 'caught': False, 'note': 'phrase not found'})
            print(mid, 'PHRASE NOT FOUND', counts)
            continue
        path = os.path.join(WORK, f'{mid}.docx')
        doc.save(path)
        text = render(path, os.path.join(WORK, f'{mid}.pdf'))
        if text is None:
            rows.append({'id': mid, 'target': target, 'applied': counts, 'caught': False, 'note': 'render failed'})
            continue
        res = checks.run_text(text, filename_present=True)
        red = sorted(k for k, v in res.items() if not v)
        extra = [k for k in red if k in base_red]     # criteria red before the mutation too
        changed_red = [k for k in red if k not in base_red]
        caught = bool(target) and (target in red)
        rows.append({'id': mid, 'target': target, 'applied': counts, 'caught': caught,
                     'red': red, 'newly_red': changed_red, 'green': sum(res.values())})
        print(f"{mid} applied={counts} target={target} caught={caught} newly_red={changed_red}")
    json.dump(rows, open('/work/pto/render/mutations.json', 'w'), indent=1)
    targeted = [r for r in rows if r['target']]
    caught = [r for r in targeted if r['caught']]
    print(f"\ncaught {len(caught)}/{len(targeted)} targeted mutations")
    for r in rows:
        if not r['target']:
            print('control', r['id'], '| newly red:', r.get('newly_red'))


if __name__ == '__main__':
    main()

```


## code/src/stub_judge.py

```text
# -*- coding: utf-8 -*-
"""A stand-in judge on loopback so the task's own tests/test.sh -> verify.py -> grade.py runs end to end.

It speaks the OpenAI chat-completions shape the task's grader posts to.  It does not return a constant:
it reads the candidate text out of the request and runs checks.py on it, so a document with a broken
statement comes back with that criterion failed and a reward below 1.  This is a wiring test, not an
evaluation: the stub is my reader, not an independent judge.
"""
import json
import re
import sys

from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer

sys.path.insert(0, '/work/pto/src')
import checks

PORT = int(sys.argv[1]) if len(sys.argv) > 1 else 8777
LOG = []


class Handler(BaseHTTPRequestHandler):
    protocol_version = 'HTTP/1.1'

    def log_message(self, fmt, *args):
        pass

    def do_POST(self):
        n = int(self.headers.get('Content-Length') or 0)
        body = self.rfile.read(n).decode('utf-8', 'replace')
        try:
            req = json.loads(body)
        except json.JSONDecodeError:
            req = {}
        text = '\n'.join(m.get('content', '') if isinstance(m.get('content'), str)
                         else ' '.join(p.get('text', '') for p in m.get('content', []) if isinstance(p, dict))
                         for m in req.get('messages', []))
        wanted = sorted({int(x) for x in re.findall(r'"index":\s*(\d+)', text)})
        # the request is "<criteria payload>\n\nReturn JSON matching this schema:\n<schema>\n<body>"
        marker = 'Return JSON matching this schema:'
        i = text.find(marker)
        if i >= 0:
            j = text.find('"additionalProperties": false}', i)
            if j > 0:
                text = text[j + len('"additionalProperties": false}'):]
        import os
        idx = len(LOG)
        os.makedirs('/work/pto/render/bodies', exist_ok=True)
        open(f'/work/pto/render/bodies/body{idx}.txt', 'w', encoding='utf-8').write(text[-3000:])
        db = text if any(i in (10, 11) for i in wanted) else None
        res = checks.run_text(text, filename_present=True, db_diff=db)
        if not wanted:
            wanted = list(range(len(checks.CHECKS)))
        grades = []
        for i in wanted:
            name = checks.CHECKS[i][0]
            ok = bool(res.get(name))
            grades.append({'index': i, 'score': 1.0 if ok else 0.0, 'passed': ok,
                           'evidence': f'stub reader: {name} {"found" if ok else "did not find"} its statement',
                           'reason': f'checks.{name} over the received candidate text'})
        LOG.append({'n_chars': len(text), 'asked': wanted, 'passed': sum(g['passed'] for g in grades)})
        payload = {'id': 'stub', 'object': 'chat.completion', 'model': 'stub-judge', 'choices': [
            {'index': 0, 'finish_reason': 'stop',
             'message': {'role': 'assistant', 'content': json.dumps({'grades': grades})}}],
            'usage': {'prompt_tokens': len(text) // 4, 'completion_tokens': 10, 'total_tokens': len(text) // 4 + 10}}
        raw = json.dumps(payload).encode()
        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.send_header('Content-Length', str(len(raw)))
        self.end_headers()
        self.wfile.write(raw)
        with open('/work/pto/render/stub_log.json', 'w') as f:
            json.dump(LOG, f, indent=1)


if __name__ == '__main__':
    srv = ThreadingHTTPServer(('127.0.0.1', PORT), Handler)
    srv.serve_forever()

```


## code/src/verify_chain.py

```text
# -*- coding: utf-8 -*-
"""Second path to the decisive facts: read the record's own structured files (xlsx cells, docx
paragraphs) and assert the chain the memo rests on, without going through the text dumps."""
import re
import docx
import openpyxl

ENV = '/work/ple-dataset/tasks/permission-to-operate-memo/environment/files/'
ok = []

wb = openpyxl.load_workbook(ENV + 'Equipment/PXA-2023-0629_Corwyn_Equipment_Schedule.xlsx', data_only=True)
cells = {}
for ws in wb.worksheets:
    for row in ws.iter_rows(values_only=True):
        if row and row[0]:
            cells[str(row[0]).strip()] = ' | '.join('' if c is None else str(c) for c in row)
pp1 = cells['PP-1']; tx1 = cells['TX-1']; swbd = cells['SWBD-PV']; inv = cells['INV-C1 to INV-C5']
ok.append(('PP-1 circuit 1 feeds the RTU enclosure', 'Circuit 1: dedicated 20 A circuit to the utility RTU enclosure' in pp1))
ok.append(('PP-1 circuit 1 goes through the 1.5 kVA online UPS', 'through the 1.5 kVA online UPS in that enclosure' in pp1 and '5.5 h at the RTU load' in pp1))
ok.append(('TX-1 is fed from a 40 A two-pole breaker in SWBD-PV', 'fed from a 40 A two-pole breaker in SWBD-PV' in tx1))
ok.append(('SWBD-PV is the Facility 1,200 A switchboard', '1,200 A bus, 1,200 A main breaker' in swbd))
ok.append(('inverter control power comes from its own ac output terminals', 'control power internal, from the inverter' in inv))
ok.append(('AD-C is the inverter ac isolating device on the output circuit',
           'ac isolating device' in cells['AD-C1 to AD-C5'] and 'inverter output circuit' in cells['AD-C1 to AD-C5']))

mcs = openpyxl.load_workbook(ENV + 'Equipment/PXA-2023-0629_Corwyn_Monitoring_and_Communications_Schedule.xlsx', data_only=True)
links = {}
for ws in mcs.worksheets:
    for row in ws.iter_rows(values_only=True):
        if row and row[0]:
            links[str(row[0]).strip()] = ' | '.join('' if c is None else str(c) for c in row)
ok.append(('per-inverter points reach the RTU over COM-RTU from RS-485 port 2',
           'INV-C1 to INV-C5 RS-485 port 2, one multi-drop bus' in links['COM-RTU']
           and 'per-inverter ac power and status for SCADA' in links['COM-RTU']))
ok.append(('DAS-1 is customer monitoring on LP-2, not the RTU feed',
           'LP-2' in links['DAS-1'] and 'DAS-1' in links['DAS-1']))

agr = docx.Document(ENV + 'Contracts/ISA-Corwyn-Office-Complex_Executed_Agreement.docx')
body = '\n'.join(p.text for p in agr.paragraphs)
ok.append(('ISA 6.1 keeps the POI recloser open before PTO', 'At all other times before permission to operate the POI recloser is kept open.' in body))
ok.append(('ISA 7.2 makes live telemetry a condition of PTO', 'Live telemetry at the System Operations Center is a condition of permission to operate' in body))
ok.append(('ISA 12.1 requires a written instrument signed by both Parties', 'No term of this Agreement may be modified or waived except by an instrument in writing signed by authorized representatives of both Parties.' in body))
ok.append(('ISA 5.2 puts the RTU station service on the customer', 'the station service and communications entrance for the RTU' in body))
ok.append(('ISA 8.7 forbids equipment/settings changes without written consent', "makes no change to the Facility's equipment or settings without the Company's written consent" in body))
ok.append(('the Corwyn agreement carries NO pre-PTO backfeed term', 'backfeed' not in body.lower() and 'before permission to operate' in body))
ok.append(('Exhibit B.3 excludes DAS-1 from Company telemetry', "is not part of the Company's telemetry" in body))

halb = '\n'.join(p.text for p in docx.Document(ENV + 'Contracts/Harkwell_Ridge/Executed_ISA_Harkwell_Ridge.docx').paragraphs)
print('(Harkwell read for contrast; backfeed term present:', 'backfeed' in halb.lower(), ')')

for name, val in ok:
    print(('OK  ' if val else 'FAIL'), name)
print(f'{sum(1 for _, v in ok if v)}/{len(ok)} record facts confirmed from the structured files')

```


## deliverables/Corwyn_PTO_Memo.txt

```text
ALLENIOM UTILITIES — DER PROGRAM OFFICE

4200 Veldmere Boulevard, Halston, Alleron 01HA · (612) 555-0100 · alleniom.com

MEMORANDUM

To: Lena Kovalchuk, SOC Shift Supervisor, System Operations Center

From: DER Program Office — reviewing engineer (P.E.); signature block at Section 8

Date: 07/14/2025

Subject: Permission to Operate for Corwyn Office Complex PV (PXA-2023-0629): whether it can issue on 08/04/2025

References: Interconnection Service Agreement ISA-Corwyn-Office-Complex, executed 09/12/2024; Corwyn Realty Partners’ permission-to-operate request of 07/10/2025; the record as it stands at 07/14/2025

Copy: DER Program Office — Contracts; Commissioning, DER Program Office

1. Determination

Permission to Operate cannot issue on 08/04/2025 on the record as it stands. The bar is the telemetry condition, not the equipment: the Facility cannot be energized before Permission to Operate under the agreement as executed, and the Company’s RTU takes its only station service from the Facility’s own 480 V switchboard. While the POI recloser is held open before Permission to Operate, the RTU has no station service and does not present valid points at the SCADA operator display — which is what TCS-02 §5 requires at the moment Permission to Operate is issued.

That answer turns on the record as it stands and not on the calendar: the record holds no event between 07/14/2025 and 08/04/2025 that would put station power on the RTU or valid points on the operator display. The bar can be removed, and the cure is capable of being completed before 08/04/2025, but only by an instrument in writing signed by authorized representatives of both Parties under Section 12.1 of the agreement, followed by an energization of the Facility by Distribution Operations and a fresh confirmation by the System Operations Center at the moment of issuance. None of that is done, none of it is required of either Party by the record, and the Office cannot do it for them. What has to be done, by whom, and whether it can be done before 08/04/2025 is set out at Section 4.

2. The record as it stands

2.1 The agreement. ISA-Corwyn-Office-Complex was executed 09/12/2024 for 700 kW (ac) on feeder ALN-072, five Vexlar VX-140C inverters of 140 kW each at 480 V. Section 6.2 states the five conditions of Permission to Operate: “Parallel operation begins only with the Company’s written permission to operate, issued once the Company’s facilities are commissioned and accepted, telemetry is live at the System Operations Center, the witness test is complete, and the authority having jurisdiction has given final approval of the customer’s facilities.”

2.2 Four of the five are satisfied on the record. The commissioning checklist CX-CW-01 (07/14/2025) records the RTU installed and its end-to-end point-to-point test accepted by SCADA/EMS Engineering (M. Deele) on 07/08/2025, and the POI recloser site acceptance items 1 to 7 accepted in writing by Distribution Protection Engineering (N. Achterberg, P.E.) on 07/08/2025. Witness test record WT-0629-01, dated 07/08/2025, records every test point passed and is signed by the witness engineer (M. Halbrook, P.E.); parallel operation for the test ran from 08:10 to 15:35 under Distribution Operations switching order DO-2025-0677, with the POI recloser “closed at 08:10 and opened at 15:35 locally by Distribution Operations”. The City of Wrenford gave final approval under permit WEP-25-0317 on 07/02/2025. Revenue meter RM-0629 was set and tested by Alleniom Metering with its registers verified on 07/08/2025.

2.3 The fifth condition — telemetry live at the System Operations Center — is not satisfied and does not become satisfied by the passage of time to 08/04/2025. That is what stands in the way.

3. What stands in the way

3.1 The RTU’s only station service is fed from SWBD-PV, below the POI recloser. The equipment schedule for the site (drawing E-101 Rev C) gives PP-1 “Circuit 1: dedicated 20 A circuit to the utility RTU enclosure on the POI pole, through the 1.5 kVA online UPS in that enclosure (5.5 h at the RTU load)”. PP-1 is fed from the auxiliary transformer TX-1, 15 kVA, 480 V to 120/240 V, which is “fed from a 40 A two-pole breaker in SWBD-PV”. SWBD-PV is the Facility’s own 480Y/277 V switchboard with its 1,200 A main breaker; it is fed through the step-up transformer T-1, 750 kVA, and the primary switch PSW-1 from the 13.2 kV conductors that run from the Point of Interconnection (Exhibit A of the agreement). There is no other source for the RTU. The customer’s monitoring cabinet DAS-1 is fed from office building panel LP-2 circuit 14, but Exhibit B.3 of the agreement states that DAS-1 “is the Interconnection Customer’s own monitoring and is not part of the Company’s telemetry”.

3.2 Before Permission to Operate the agreement keeps the POI recloser open. Section 6.1 permits the Company to place the Facility in parallel “solely for the witness test” and provides: “At all other times before permission to operate the POI recloser is kept open.” The witness test is over: the recloser was opened at 15:35 on 07/08/2025. With the recloser open, T-1, SWBD-PV, TX-1 and PP-1 are dead, and the RTU is on its uninterruptible supply only. At 5.5 h at the RTU load that supply carried the RTU to about 21:05 on 07/08/2025 (my arithmetic from the two figures on E-101: 15:35 plus 5.5 h); since then the RTU has had no station service. TCS-02 §3.4 requires the customer to provide that station AC service “through a dedicated branch circuit backed by an uninterruptible supply sized for a minimum four (4) hour outage ride-through”; a 5.5 h supply meets that requirement, but a ride-through supply is not a station source, and the two together do not give continuous telemetry across the weeks between the witness test and the date requested.

3.3 The System Operations Center’s confirmation of 07/08/2025 does not carry to 08/04/2025. TCS-02 §5(c) defines live telemetry as “Presenting all required data points (§6) as valid (not stale, not forced, not flagged bad-quality) at the SCADA operator display at the moment Permission to Operate (PTO) is issued”, states that compliance with §5 “is confirmed by the SOC Shift Supervisor on shift at the time PTO is requested”, and that “PTO shall not be issued to any in-scope DER site until §5 is satisfied”. The commissioning checklist records the SOC confirmation by L. Kovalchuk on 07/08/2025 — inside the witness-test window, when the recloser was closed and the RTU was on station power. That entry is a statement about the operator display on 07/08/2025; §5(c) judges the operator display at the moment of issuance. The SOC PTO-Hold Log (DER-EQP-SOC-PTOH, as of 07/14/2025) states the position in terms: “No PTO without live telemetry — TCS-02 §5 (no interim provision)”, and “Requests for early, provisional, partial, or conditional operation are refused on the same basis and returned to the DER Program Office”. The agreement says the same: §7.2, “Live telemetry at the System Operations Center is a condition of permission to operate”, and DIS-01 Rev 4 §6 likewise.

3.4 Powering the RTU alone would not satisfy §5. TCS-02 §6.1 requires values “individually per block”, among them “Gross generation (per inverter block)” and “Inverter/PCS block status”. On this site those values reach the RTU only over COM-RTU, which the monitoring and communications schedule (drawing E-401 Rev C) gives as “INV-C1 to INV-C5 RS-485 port 2, one multi-drop bus” to the RTU serial port for “per-inverter ac power and status for SCADA”. Each VX-140C takes its “control power internal, from the inverter’s own ac output terminals” (E-101), so an inverter that is not energized on its ac side is dead and reports nothing. TCS-02 §6.3 requires values held during a loss to be “reported with the communications-lost flag until fresh source values are acquired”, and §5(c) admits only valid points. So the per-inverter points are valid at issuance only once the inverters are powered from an energized Facility — not merely once the RTU is powered.

4. What has to be done about it, by whom, and whether it can be done before 08/04/2025

4.1 A written instrument under Section 12.1 permitting energization of the Facility before Permission to Operate. Section 12.1 provides: “No term of this Agreement may be modified or waived except by an instrument in writing signed by authorized representatives of both Parties.” Section 6.1 permits parallel operation before Permission to Operate solely for the witness test and keeps the POI recloser open at all other times, and the Corwyn agreement contains no pre-PTO backfeed term. The program record shows what such a term looks like: ISA-Halbray-Field-Site-3 carries one — “On the interconnection customer’s written request, the Company may energize the customer’s facilities from the feeder for construction and commissioning power before permission to operate”, with the generating facility staying disabled and “Distribution Operations performs the switching for a backfeed under its own switching order”. Corwyn’s agreement has no such term, so one has to be made: an instrument in writing under Section 12.1 permitting the energization of the Facility from feeder ALN-072 before Permission to Operate.

4.2 It is signed by the DER Program Office (Contracts) and by Corwyn Realty Partners. Section 12.1 asks for authorized representatives of both Parties; the executed summary records execution “for Alleniom Utilities by the DER Program Office (Contracts) and for Corwyn Realty Partners”, and Nadia Ferreira-Holt, Director of Property Operations, signed the agreement for Corwyn.

4.3 The instrument can be signed before 08/04/2025, and it does not reopen the queue. The change is administrative and does not alter capacity or the point of interconnection, so it is handled under the change-order provisions of Section 12.1 and not by the withdrawal-and-reapplication path of DIS-01 §7, which applies to capacity and Point of Interconnection changes and states that “Modifications to protective schemes, telemetry endpoints, equipment models within the same nameplate rating, and administrative items that do not alter capacity or POI are handled under the change-order provisions of the executed ISA and are not subject to this section”. Two signatures, no study, no queue action; 21 calendar days run between 07/14/2025 and 08/04/2025.

4.4 Distribution Operations energizes the Facility under its own switching order. Once the instrument permits energization before Permission to Operate, Distribution Operations closes the POI recloser and energizes the Facility from feeder ALN-072 under a switching order of its own, as it did for the witness test under DO-2025-0677 and as the Halbray term describes. Until the instrument is signed, Distribution Operations cannot close the POI recloser for this purpose: §6.1 keeps it open at all other times before Permission to Operate. A switching order is a matter of days and can be scheduled before 08/04/2025.

4.5 The energization has to keep the inverter ac disconnects AD-C1 to AD-C5 closed, so that the inverters are powered. Each AD-C is the inverter’s “ac isolating device” on “the inverter output circuit” (E-101); an AD-C left open leaves its inverter unpowered, and its COM-RTU points would be reported communications-lost under TCS-02 §6.3, which §5(c) does not admit. So during the pre-PTO energization the five 225 A inverter breakers in SWBD-PV and AD-C1 to AD-C5 are closed and the inverters are powered from the energized Facility. The Facility stays out of generation by other means: each VX-140C has two dc inputs, each fused from 80 to 250 A at 1000 V dc with a load-break disconnect (Vexlar VXC-DS Rev 5), and those dc input disconnects are opened for the energization, so the inverters hold control power from their own ac terminals and report status, but have no dc source and do not export. No inverter setting is changed: §8.7 of the agreement forbids changes to the Facility’s equipment or settings without the Company’s written consent, and the IEEE 1547-2018 Category III settings verified at the witness test stand as verified.

4.6 The System Operations Center confirms the points at the moment of issuance. When Permission to Operate is to be issued, the SOC Shift Supervisor on shift confirms at the SCADA operator display that the required points are valid (TCS-02 §5, §5(c)). That is a fresh confirmation and not the 07/08/2025 entry, and it is the last step: it can be given only after 4.1 to 4.5 are done.

4.7 Whether all of that can be done before 08/04/2025. The cure fits before 08/04/2025: two signatures, one switching order and one SOC confirmation inside the 21 calendar days between 07/14/2025 and 08/04/2025, with no study, no queue action and no equipment delivery in the way. Corwyn is not on the register of completed sites awaiting utility equipment (DER-EQP-COMPL Rev 3) because its RTU and POI recloser are installed and accepted. But nothing in the record requires either Party to sign, and the Office cannot give the date for them. If the Parties execute the instrument, and Distribution Operations energizes the Facility, and the SOC Shift Supervisor confirms the points at the moment of issuance, all of that can fall inside the 21 days and the telemetry condition is then satisfied on 08/04/2025. If any of it is not done, the condition is not satisfied on that date and Permission to Operate cannot issue.

4.8 One alternative is not open on this record. Moving the RTU’s station service to a supply that is not fed through the Facility — the office building panel, as DAS-1 has — would be a change to the interconnection facilities the customer provides under §5.2 “as specified in the Company’s construction package”, and §8.7 requires the Company’s written consent for a change to the Facility’s equipment. No such request is on file, and it would not be the shorter route to 08/04/2025. The instrument at 4.1 is the path the record supports.

5. What this memo does not cover

Four matters are under separate review and are not assessed here: the installation, configuration and commissioning tests of the Company’s RTU and POI recloser (SCADA/EMS Engineering and Distribution Protection Engineering); the POI recloser’s settings and the protection coordination study behind them (Distribution Protection Engineering, which issued them and keeps them outside these records); the site’s queue position (Interconnection Studies, under the weekly queue reports); and the system impact study and its conditions (the DER Planning Group, which keeps them outside these records). Nothing else is under separate review. Apart from the telemetry condition set out above, nothing else on the record stands in the way of Permission to Operate.

For the record, the SOC PTO-Hold Log as of 07/14/2025 carries seven sites held on telemetry grounds; Corwyn Office Complex PV is not among them, its request of 07/10/2025 having been referred to this Office before issuance. This determination would be the basis of its entry.

6. Recommended action

6.1 The System Operations Center holds Permission to Operate for 08/04/2025 and records the hold on the PTO-Hold Log.

6.2 The DER Program Office (Contracts) prepares the Section 12.1 instrument with Corwyn Realty Partners for signature, and the Parties execute it before 08/04/2025.

6.3 On execution, Distribution Operations schedules the energization switching order for the Facility from feeder ALN-072, with AD-C1 to AD-C5 closed and the dc input disconnects at each inverter open.

6.4 On completion of the energization, the SOC Shift Supervisor confirms the points valid at the SCADA operator display at the moment of issuance, and Permission to Operate may issue.

6.5 The determination and the items at 6.1 to 6.4, including the signature of the Section 12.1 instrument by Corwyn Realty Partners, are to be settled before Permission to Operate and are recorded in one work package for the site in the DER Program Backlog Review project.

7. Basis of this memo

This memo works from the record as it stands at 07/14/2025: ISA-Corwyn-Office-Complex and its executed summary; the site equipment schedule (E-101 Rev C), the monitoring and communications schedule (E-401 Rev C) and the Vexlar VX-C series data sheet (VXC-DS Rev 5); TCS-02 Rev 1, DIS-01 Rev 4 and the APUC-filed tariff §12; the commissioning checklist CX-CW-01, the witness test record WT-0629-01, the SOC PTO-Hold Log, the register of completed sites awaiting utility equipment (DER-EQP-COMPL Rev 3), the active queue report of 07/14/2025 and the contact directory; the City of Wrenford permit WEP-25-0317 and its final approval; the Corwyn request of 07/10/2025; and, for comparison of the pre-PTO energization term, ISA-Halbray-Field-Site-3.

8. Signature

Reviewed and submitted for signature for the DER Program Office.

Signature: ______________________________________     Date: ________________

Name (print): ___________________________________, P.E.

Reviewing Engineer (P.E.), DER Program Office, Alleniom Utilities
```
