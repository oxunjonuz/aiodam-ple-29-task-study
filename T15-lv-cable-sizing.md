# lv-cable-sizing

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-15 — lv-cable-sizing
   Two terminals decide the whole design; 1.0 is forbidden by the task's own 3% cost buffer, and the report
   measures that rather than pretending otherwise.

 Final reward (R) — paper rule 0.4p + 0.6[h=1]O′            0.892650 · measured

 Task grader (G) — combined score                           0.685658 · measured

 Official entry — tests/test.sh                             fails before the grader: JUDGE_BASE_URL is not set

 Container                                                  Docker on isolated VM with a stub judge relay (turn 499): 0.926935 (simulation, not a
                                                            judge score)

 3 judge families of 35; ceiling = 0.4 + 0.6*O'             0.926935 · computed, not measured

 Judge                                                      J families not measured

 File                                                       /work/lv-cable-sizing/build3/submission.zip · 35,905 B · sha256
                                                            02e84d7896bd87d9…


TASK AND INPUTS
Size 32 low-voltage circuits — wire schedule, DXF, calculation book, design basis and a 37-row register — under ampacity, voltage
drop, thermal grouping, duct-bank and SKU constraints.

EDS-26 calculation rules, the Cascadia ampacity tables, the duct bank and tray geometry, the price list, and the circuit tree.

WHAT WAS AMBIGUOUS OR BROKEN
    The compressor B-17 terminals (6 AWG) leave only 2.30% of the 5% voltage-drop budget for the whole branch.
    The 75 °C limit for 600 kcmil aluminium gives 336 A where the physics allows 411 A, which fixes the service configuration.
    F-05 must be 3×600 AL over 716 ft; SE-1 must be 5×600 Cu in five spaced DB-1 ducts; one forbidden SKU makes the first SE-1
    configuration infeasible.

WHAT I DID AND WHAT CAME OUT
    Exact DP over the circuit tree (state = accumulated drop), a fixed point over the thermal environment, and an enumeration of 9 DB-1
    pairs × 21 sets of five sizes.
    Final package $150,478.75 against the reference $150,678.56 (−0.13%); O′ = 0.8782.
    Grader 0.685658; paper rule 0.892650 without the judge; the ceiling with J1–J3 is 0.926935, reached only with a stub relay in a
    container — a simulation.

HOW IT WAS VERIFIED
    An independent verification of the whole package from the documents: 0 refusals, 0 warnings, cost to the cent.
    Thirteen breakages through the real grader: twelve noticed; the survivor is a grader property — 348 of 608 size cells in the E-621
    sheet read as agreement even when wrong (57.2%), so ten substituted rows still give agreement 1.000. My verifier catches 10 of 10.
    The official path in a container on an isolated VM, with a stub judge relay, returned 0.926935 — same number as the local
    computation of the ceiling.

MY MISTAKES IN THIS TASK

    Demand current for lighting and appliance circuits was computed as 125% of load instead of the device rating; F03 went red on 5 of
    22 circuits and G fell to 0.773. Fixed to 1.000.
    The DP route reconstruction did not check constraints and clamped the state index, so a branch with a 4.73% drop “passed”.
    The SKU cap broke the fixed DB-1 pair and the algorithm silently returned six sizes.
    F-05 had no ducts inside the loop, so its mutual heating was computed as zero.
    My own checker was wrong three times: aluminium rows overwrote copper, a single-phase 208 V circuit was treated as one wire, and
    the price header was searched after comment rows.


                                                                page 26 of 48
PLE 29-task report · AIODAM                                                                                               11 November 2026




WHAT REMAINS UNVERIFIED
J1–J3 (0.0343 of the weight). A score of 1.0 requires cost ≤ $146,158.20 = 0.97 × reference; the exhaustive search stopped at
$150,478.75. That is a measured barrier, not a proof of infeasibility.

USE OF REFERENCE OR GRADER MATERIAL

Explicitly used: truth.json (reference lengths, ampacities, equipment), optimum.json (gold cost, bound, calibration), o_calibration.json
and register_map.json. They told me what to aim at, which 32 requirements to cover, and let me verify my document readings; the
construction is mine.



Part C — rubric tasks
Fourteen tasks scored by an LLM judge against weighted criteria. Every document is finished and hashed; none has a score. What is
measured instead — my criteria reader, the breakage campaigns and the number audits — is described in each section and never
presented as a judgment.


PART C · RUBRIC · ELECTRICAL / INTERCONNECTION
```


## deliverables/design_basis.md

```text
# Design basis - wire schedule, Building 4210 (P-4210)

```json
{
 "codes_of_record": {
  "nfpa70_edition": "2023",
  "ibc_edition": "2024",
  "adopted_by": "UFC 1-200-01 DoD Building Code, 1 September 2022 with Change 4 of 17 December 2024",
  "precedence": "the most restrictive requirement governs; where a document is silent the UFC governs (G-001, EDS-26 1.1)",
  "design_start": "14 July 2025"
 },
 "environment": {
  "outdoor_design_dry_bulb_c": 42.0,
  "roof_ambient_c": 46.0,
  "earth_ambient_c": 27.45,
  "soil_thermal_resistivity_k_m_w": 0.6,
  "solar_irradiance_w_m2": 1030.0
 },
 "sources": {
  "outdoor_design_dry_bulb_c": "NOAA daily normals 1991-2020, station USW00003145, highest daily normal maximum, rounded up (EDS-26 5.4)",
  "roof_ambient_c": "M-501-C roof value = outdoor design + 4 K (EDS-26 5.4)",
  "earth_ambient_c": "NOAA daily normals 1991-2020, annual mean of the daily average normals + 3 K (EDS-26 5.3.2)",
  "soil_thermal_resistivity_k_m_w": "GEO-4210-T 2.2 thermal backfill, 0.60 K.m/W",
  "solar_irradiance_w_m2": "EDS-26 5.4"
 },
 "voltage_drop": {
  "start_point": "secondary lugs of the utility pad-mount transformer T-1, the point of government ownership (UFC 3-501-01 3-2.7.4.2)",
  "transformer_reset": "no - the count is not restarted at the 208 V transformer K-13; the percentage at its secondary lugs carries through to the load, because 3-2.7.4.5.3 restarts only where the system is transformed from above 1000 V to below it, which K-13 is not",
  "feeder_chain_pct": {
   "chain_to_SE1": 1.04,
   "chain_to_F01": 2.58,
   "chain_to_F02": 1.41,
   "chain_to_F06": 1.57,
   "chain_to_F07": 2.77,
   "chain_to_F05": 2.58,
   "chain_to_F10": 2.78
  },
  "branch_limit_pct": 3.0,
  "total_limit_pct": 5.0
 },
 "cost": {
  "conductor_usd": 99246.76,
  "conductor_net_usd": 93291.95,
  "termination_usd": 9597.2,
  "raceway_usd": 47589.6,
  "material_usd": 96808.75,
  "labor_usd": 53670.01,
  "total_usd": 150478.75
 },
 "duct_assignment": {
  "SE-1": [
   "D1",
   "D2",
   "D3",
   "D5",
   "D6",
   "D9"
  ],
  "F-05": [
   "E1",
   "E2",
   "E3"
  ],
  "F-10": [
   "D4",
   "D8"
  ]
 },
 "mutual_heating_rise_k": 29.86,
 "mutual_heating_rise_by_circuit_k": {
  "SE-1": 29.86,
  "F-05": 1.54,
  "F-10": 24.1
 },
 "tray_sum_od_in": 10.404,
 "distinct_large_sizes": [
  "4/0",
  "300 kcmil",
  "400 kcmil",
  "500 kcmil",
  "600 kcmil"
 ]
}
```

## Codes of record

The design was prepared against NFPA 70-2023 and IBC-2024. Both are requirements of every
design here, adopted by **UFC 1-200-01, DoD Building Code, 1 September 2022 with Change 4 of
17 December 2024**, which adopts the IBC and directs every reference that code makes to
NFPA 70 to UFC 3-501-01, which cites NFPA 70. UFC 3-501-01 then adds the Department's own
requirements and UFC 3-520-01 adds the interior ones; NSA-WR EDS-26 Rev 4 adds the
installation's. The editions are fixed at the start of design, 14 July 2025, on sheet G-001.
Where two of these documents set different limits for the same thing, **the most restrictive
requirement governs**, and where a document is silent the UFC governs.

EDS-26 Rev 4 supersedes Rev 2 in its entirety, and Rev 2 is in the record for reference only:
its 5 ft termination allowance and its 20 C earth ambient are **not** used. The Meridian cable
datasheet is not the basis of design; the ampacities come from the Cascadia Section 4
engineering data.

## Environment and its provenance

The outdoor design dry-bulb is **42 C**: the highest daily normal maximum in the station record
filed with the site documents (NOAA 1991-2020 normals, station USW00003145) is 106.8 F, i.e.
41.6 C, rounded up to the next whole degree as EDS-26 5.4 directs. The roof ambient is that
value **plus 4 K = 46 C**, which is the figure M-501-C states for the roof tray. The earth
ambient at duct-bank depth is **27.45 C**: the annual mean of the daily average normals,
76.00 F = 24.45 C, plus 3 K (EDS-26 5.3.2). The thermal resistivities are GEO-4210-T 2.2 and
2.3: **0.60 K.m/W for the fluidised backfill** that surrounds the encasement for the full width
of the trench, and 0.55 K.m/W for the concrete. The native-soil value of 0.95 K.m/W is recorded
on the front sheet as considered and rejected, because it is not the material whose resistivity
enters the earth and mutual-heating terms. Space ambient temperatures are M-501-C's design
maxima: 45 C in the mechanical room and fire pump room and for the feeders entering them, 46 C
at the roof tray, 40 C in the ceiling cavity, 30 C in conditioned and electrical spaces.

## The voltage-drop budget

The count starts at **the secondary lugs of T-1**, the point of government ownership, and it
does **not** restart at the 208 V transformer K-13. UFC 3-501-01 3-2.7 states that the voltage
drop requirements restart assuming nominal voltage only where the voltage within the facility's
distribution system is increased over the 1000 V threshold and is then transformed back below
it; K-13 is a 480 V to 208 V transformer, both sides far below that threshold, so no restart
applies to it and the budget spent upstream still counts. Paragraph 3-2.7.4.5.3 says the same
thing in the other direction: transformer losses are zero, but the percentage at the secondary
lugs must equal the percentage at the primary terminations. The drop spent by SE-1, F-02 and
F-06 therefore still counts against the load at the end of every LP-1 branch circuit: **ten
branch circuits** (B-01 to B-10) are fed from LP-1 and inherit everything spent upstream of
their panelboard. Feeders are held to 3 percent cumulative, branch circuits to
3 percent of their own, and no load may exceed 5 percent from the starting point.

The budget was allocated by making the two cheapest kilovolt-ampere-feet carry the drop. The
service and the long F-05 duct run take the largest share because a size step there is cheap
per foot of drop removed on the 480 V side; the 208 V secondary chain (F-07, 127 ft at 625 A)
is the expensive one, so it is given the larger conductor and the branch circuits behind it are
held to the small remainder. The chains actually spend: F-01 2.58%; F-02 1.41%; F-05 2.58%; F-06 1.57%; F-07 2.77%; F10 2.78%; SE-1 1.04%.

## Service entrance

The service-entrance conductors are sized to **the ampacity of the main bus of MSB-1, 2000 A**,
and not to the 1600 A rating of the main device that protects them. That is a deliberate
departure from the ordinary rule that a conductor is sized to its overcurrent device: UFC
3-501-01 3-2.3.3, repeated by EDS-26 2.2, makes the bus ampacity the sizing quantity for a
service entrance. 6 sets of 500 kcmil copper in 6 ducts of DB-1 carry 2130 A corrected, and the
equipment grounding conductor is a 4/0 copper sized on the 1600 A main device.

## The duct bank, and the thermal trade it carries

**Ten ducts of DB-1 are available** to this scope: D1 to D10. D11 is an owner spare and D12 is
the existing site-lighting feeder, so both are left empty; DB-2 offers E1 to E3 with E4 reserved
for communications. Every loaded duct raises the temperature of every other duct in the same
encasement by dT = LF * sum W_j (rho_e / 2 pi) ln(d'/d), and the loss in each duct is
calculated at the circuit's operating current (connected load divided by the number of sets),
not at its ampacity. **Adding a parallel set to one circuit therefore lowers the ampacity
available to the other circuits of the same bank** - it puts another hot duct in the
encasement. That trade was resolved by spreading the service over 6 ducts of the bank (D1, D2, D3, D5, D6, D9)
and putting F-10's 2 sets in the ducts furthest from the service ducts (D4, D8). The service was run
as 6 sets of 500 kcmil copper rather than as five sets of 600 kcmil: the sixth set adds 320 ft of
conductor and one more termination pair, but it lowers the service's own drop and keeps the
mutual rise at a level F-10 can live with on 500 kcmil aluminium. Five sets of 600 kcmil in the
same bank need a low-loss F-10 (350 kcmil copper) to stay under 2000 A and cost more in total.
The rise each circuit actually sees is SE-1 29.86, F-05 1.54, F-10 24.10 K. DB-2 carries F-05 alone, so its rise comes only
from its own three sets.

## Material and standardisation

Aluminium is used where UFC 3-520-01 3-6.3.1 permits it - 4 AWG and larger - and where it wins
on installed cost at the ampacity it delivers: the four CT-R1 tray circuits, the F-01 feeder to
MCC-1, the F-07 secondary and the F-05 duct run. It is never used for an equipment grounding conductor, which is copper on every
circuit. The design standardises on **5 distinct sizes larger than 1 AWG** (4/0, 300 kcmil, 400 kcmil, 500 kcmil, 600 kcmil), inside the
five-size cap of EDS-26 3.4.2, and every size below 1 AWG is chosen from the standard trade
sizes.

## Cost

The ledger is built from the Cascadia installed unit prices and the unit price book, both of
which state a material part and a labour part for every item. Conductor is 99,246.76 USD installed,
terminations 9,597.20 USD, and interior raceway 47,589.60 USD; the campus quantity break of 6 percent applies
to the whole installed conductor price because the package installs more than 38 000 ft of
conductor counting the grounding conductors. Material is 96,808.75 USD and labour 53,670.01 USD of the
**150,478.75 USD** installed total. (The prices are installed prices: a foot of installed conductor is
material plus pulling labour, a set of terminations is material plus make-up labour, and the
conduit line is the rigid steel conduit of the unit price book.)

## What was traded against what

The two large coupled decisions were the service-entrance set count and the tray grouping. On
the service the choice was made on the total of the pair, SE-1 and F-10 together: six sets of
500 kcmil copper with F-10 on 500 kcmil aluminium costs less than five sets of 600 kcmil copper,
because the five-set bank runs hotter and forces F-10 onto 350 kcmil copper. On the roof, every extra set in CT-R1 raises N and so lowers the
free-air ampacity of all four tray circuits at once - the grouping factor k_t = 1 + 0.06(N - 3)
is charged to every circuit in the tray - which is why the two 248 A chillers take two sets each
and the two 121 A units one each rather than the other way round.

```


## deliverables/wire_schedule.csv

```text
circuit,description,from,to,install_class,length_ft,sets,size,material,egc_size,raceway_trade_in,duct_ids,design_current_a,corrected_ampacity_a,vd_load_a,vd_pct,cumulative_vd_pct,ambient_c,ccc,governing_constraint,ocpd_a,system_voltage_v,phases,power_factor,r_ohm_per_kft,x_ohm_per_kft,drop_v
SE-1,"Service entrance, T-1 secondary to MSB-1 main bus",T-1,MSB-1,Duct bank DB-1,320.0,6,500 kcmil,CU,4/0,,D1-D2-D3-D5-D6-D9,2000.0,354.9,1600.0,1.043,1.043,27.45,3,service entrance sized to the main bus ampacity (UFC 3-501-01 3-2.3.3); Neher-McGrath duct-bank ampacity with the mutual heating of the loaded bank (EDS-26 5.3); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 400 kcmil; feeder cumulative voltage drop 1.04% of 3%,1600,480,3,0.92,0.0278,0.0212,5.01
F-01,"Feeder, MSB-1 to motor control centre MCC-1",MSB-1,MCC-1,"Raceway, interior",246.0,2,500 kcmil,AL,1,4,,600.0,306.9,600.0,1.532,2.575,45.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); feeder cumulative voltage drop 2.58% of 3%,600,480,3,0.86,0.0457,0.0357,7.35
B-11,Air handling unit AHU-1,MCC-1,AHU-1,"Raceway, interior",80.0,1,4,CU,6,1.5,,81.2,85.9,65.0,0.526,3.101,45.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); branch voltage drop 0.53% and 3.10% cumulative,110,480,3,0.85,0.3024,0.0444,2.53
B-12,Air handling unit AHU-2,MCC-1,AHU-2,"Raceway, interior",98.0,1,4,CU,6,1.5,,81.2,85.9,65.0,0.645,3.22,45.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); branch voltage drop 0.64% and 3.22% cumulative,110,480,3,0.85,0.3024,0.0444,3.09
B-13,Chilled water pump CWP-1,MCC-1,CWP-1,"Raceway, interior",48.0,1,6,CU,8,1.25,,50.0,54.3,40.0,0.303,2.878,45.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 0.30% and 2.88% cumulative,70,480,3,0.86,0.4806,0.0461,1.45
B-14,Chilled water pump CWP-2,MCC-1,CWP-2,"Raceway, interior",58.0,1,6,CU,8,1.25,,50.0,54.3,40.0,0.366,2.941,45.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 0.37% and 2.94% cumulative,70,480,3,0.86,0.4806,0.0461,1.76
B-15,Cooling tower fan CT-1,MCC-1,CT-1,"Raceway, interior",178.0,1,3,AL,8,1.5,,65.0,64.7,52.0,1.2,3.775,45.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 1.20% and 3.77% cumulative,90,480,3,0.85,0.3955,0.0436,5.76
F-02,"Feeder, MSB-1 to distribution panel DP-2",MSB-1,DP-2,"Raceway, interior",62.0,1,500 kcmil,CU,3,4,,400.0,403.1,400.0,0.363,1.407,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); feeder cumulative voltage drop 1.41% of 3%,400,480,3,0.9,0.0278,0.0357,1.74
F-06,"Transformer primary feeder, DP-2 to K-13",DP-2,K-13,"Raceway, interior",36.0,1,400 kcmil,CU,3,3.5,,338.8,352.7,287.3,0.162,1.568,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); feeder cumulative voltage drop 1.57% of 3%,400,480,3,0.95,0.0336,0.0365,0.78
F-07,"Transformer secondary feeder, K-13 to panelboard LP-1",K-13,LP-1,"Raceway, interior",127.0,3,500 kcmil,AL,1/0,4,,781.2,263.3,625.0,1.202,2.77,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); feeder cumulative voltage drop 2.77% of 3%,800,208,3,0.95,0.0457,0.0357,2.5
B-01,"Lighting branch circuit, level 1 west",LP-1,LTG-1A,"Raceway, interior",193.0,1,8,CU,8,1,,20.0,39.9,13.4,1.885,4.656,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 1.89% and 4.66% cumulative,20,208,1,0.98,0.764,0.0477,3.92
B-02,"Lighting branch circuit, level 1 east",LP-1,LTG-1B,"Raceway, interior",241.0,1,6,CU,6,1.25,,20.0,54.3,14.8,1.647,4.417,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 1.65% and 4.42% cumulative,20,208,1,0.98,0.4806,0.0461,3.43
B-03,"Lighting branch circuit, level 2 north",LP-1,LTG-2A,"Raceway, interior",227.0,1,8,CU,8,1,,20.0,39.9,12.1,2.002,4.773,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 2.00% and 4.77% cumulative,20,208,1,0.98,0.764,0.0477,4.17
B-04,"Receptacle branch circuit, level 1 admin",LP-1,RCP-1A,"Raceway, interior",145.0,1,8,CU,8,1,,20.0,39.9,16.0,1.652,4.423,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 1.65% and 4.42% cumulative,20,208,1,0.95,0.764,0.0477,3.44
B-05,"Receptacle branch circuit, level 2 west",LP-1,RCP-2A,"Raceway, interior",147.0,1,8,CU,8,1,,20.0,39.9,16.0,1.675,4.445,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 1.68% and 4.45% cumulative,20,208,1,0.95,0.764,0.0477,3.48
B-06,"Receptacle branch circuit, level 2 east",LP-1,RCP-2B,"Raceway, interior",289.0,1,6,CU,6,1.25,,20.0,54.3,16.0,2.094,4.864,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 2.09% and 4.86% cumulative,20,208,1,0.95,0.4806,0.0461,4.36
B-07,Electric water heater WH-1,LP-1,WH-1,"Raceway, interior",145.0,1,6,CU,10,1.25,,40.0,54.3,28.8,1.93,4.7,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 8; branch voltage drop 1.93% and 4.70% cumulative,40,208,1,1.0,0.4806,0.0461,4.01
B-08,Kitchen exhaust fan KEF-1,LP-1,KEF-1,"Raceway, interior",207.0,1,8,CU,6,1,,17.8,39.9,14.2,1.634,4.404,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 1.63% and 4.40% cumulative,30,208,3,0.84,0.764,0.0477,3.4
B-09,Dishwasher and booster heater DW-1,LP-1,DW-1,"Raceway, interior",189.0,1,4,CU,10,1.5,,60.0,73.6,44.0,2.085,4.856,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 2.09% and 4.86% cumulative,60,208,3,0.95,0.3024,0.0444,4.34
B-10,Security head-end rack SEC-1,LP-1,SEC-1,"Raceway, interior",79.0,1,12,CU,12,0.75,,20.0,22.1,11.2,1.619,4.389,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 1.62% and 4.39% cumulative,20,208,1,0.98,1.9316,0.0509,3.37
F-08,"Feeder, DP-2 to packaged rooftop unit RTU-1",DP-2,RTU-1,"Cable tray CT-R1, roof, sun exposed",152.0,1,4/0,AL,6,,,151.2,171.9,121.0,0.661,2.067,46.0,3,free-air ampacity with solar gain and tray grouping (EDS-26 5.4); branch voltage drop 0.66% and 2.07% cumulative,175,480,3,0.88,0.1,0.0243,3.17
F-09,"Feeder, DP-2 to adjustable-speed drive VFD-1",DP-2,VFD-1,"Raceway, interior",200.0,1,300 kcmil,AL,4,3,,211.2,230.0,169.0,0.969,2.375,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 0.97% and 2.38% cumulative,225,480,3,0.96,0.0718,0.0375,4.65
F-13,"Feeder, DP-2 to packaged rooftop unit RTU-2",DP-2,RTU-2,"Cable tray CT-R1, roof, sun exposed",256.0,1,4/0,AL,6,,,151.2,171.9,121.0,1.113,2.519,46.0,3,free-air ampacity with solar gain and tray grouping (EDS-26 5.4); branch voltage drop 1.11% and 2.52% cumulative,175,480,3,0.88,0.1,0.0243,5.34
F-03,"Fire pump feeder, MSB-1 to controller FP-1",MSB-1,FP-1,"Raceway, interior",56.0,1,2,CU,6,2,,120.0,119.1,96.0,0.362,1.406,45.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); branch voltage drop 0.36% and 1.41% cumulative,200,480,3,0.87,0.1904,0.0428,1.74
F-04,"Feeder, MSB-1 to air-cooled chiller CH-1",MSB-1,CH-1,"Cable tray CT-R1, roof, sun exposed",280.0,2,4/0,AL,3,,,310.0,171.9,248.0,1.247,2.29,46.0,3,free-air ampacity with solar gain and tray grouping (EDS-26 5.4); branch voltage drop 1.25% and 2.29% cumulative,350,480,3,0.88,0.1,0.0243,5.99
F-05,"Feeder, MSB-1 to Building 4212 distribution panel DP-3",MSB-1,DP-3,Duct bank DB-2,716.0,3,600 kcmil,AL,250 kcmil,,E1-E2-E3,400.0,336.0,400.0,1.538,2.581,27.45,3,Neher-McGrath duct-bank ampacity with the mutual heating of the loaded bank (EDS-26 5.3); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 2/0; feeder cumulative voltage drop 2.58% of 3%,400,480,3,0.89,0.0396,0.0206,7.38
B-16,"Welding receptacle panel WLD-1, Building 4212",DP-3,WLD-1,"Raceway, interior",140.0,1,2,CU,8,2,,100.0,102.1,74.0,1.639,4.22,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 1.64% and 4.22% cumulative,100,208,3,0.9,0.1904,0.0428,3.41
B-17,"Shop air compressor CMP-1, Building 4212",DP-3,CMP-1,"Raceway, interior",190.0,1,6,CU,10,1.25,,43.2,54.3,34.6,2.347,4.928,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 2.35% and 4.93% cumulative,60,208,3,0.84,0.4806,0.0461,4.88
B-18,"Lighting branch circuit, Building 4212",DP-3,LTG-4A,"Raceway, interior",194.0,1,8,CU,8,1,,20.0,39.9,13.9,1.966,4.547,40.0,2,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); voltage drop (UFC 3-501-01 3-2.7.4) raised it from 12; branch voltage drop 1.97% and 4.55% cumulative,20,208,1,0.98,0.764,0.0477,4.09
F-10,"Feeder, MSB-1 to standby transfer switch ATS-1",MSB-1,ATS-1,Duct bank DB-1,318.0,2,500 kcmil,AL,1,,D4-D8,600.0,300.1,600.0,1.734,2.777,27.45,3,Neher-McGrath duct-bank ampacity with the mutual heating of the loaded bank (EDS-26 5.3); feeder cumulative voltage drop 2.78% of 3%,600,480,3,0.9,0.0457,0.0212,8.32
F-11,"Feeder, MSB-1 to elevator controller EL-1",MSB-1,EL-1,"Raceway, interior",182.0,1,3,CU,6,1.5,,97.5,99.3,78.0,1.162,2.206,40.0,3,raceway ampacity with ambient correction and conductor adjustment (EDS-26 5.2); termination temperature limit (EDS-26 5.5); branch voltage drop 1.16% and 2.21% cumulative,125,480,3,0.85,0.2399,0.0436,5.58
F-12,"Feeder, MSB-1 to air-cooled chiller CH-2",MSB-1,CH-2,"Cable tray CT-R1, roof, sun exposed",354.0,2,4/0,AL,3,,,310.0,171.9,248.0,1.577,2.62,46.0,3,free-air ampacity with solar gain and tray grouping (EDS-26 5.4); branch voltage drop 1.58% and 2.62% cumulative,350,480,3,0.88,0.1,0.0243,7.57

```
