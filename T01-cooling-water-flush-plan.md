# cooling-water-flush-plan

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-01 — cooling-water-flush-plan
   A flush programme 18% cheaper than the first buildable one — and a 1.0 that the task's own lower bound
   makes unreachable.

 Final reward (R) — paper rule 0.4p + 0.6[h=1]O′            0.949233 · measured

 Task grader (G) — combined score                           0.923849 · measured

 Official entry — tests/test.sh on this host                0.949233 · measured

 Container                                                  task verifier image in Docker on isolated VM (turn 509): 0.949233

 Judge                                                      no LLM judge in this task

 File                                                       /work/cw/ws/output/submission.zip · 10,349 B · sha256 ae3b5b76f6b3e500…


TASK AND INPUTS
Plan the pre-commissioning chemical flush of a data-centre chilled-water loop: eight piping chains, six hose-and-valve kits, a 14-day
membrane cure, water-volume and effluent limits, and a cost model. The deliverable is a flush procedure, a shift schedule and a cost
ledger, scored on 24 constraint families across all eight chains.

Project specifications (including UFC 3-230-13 water treatment), the laboratory water analysis, the equipment catalogue (FS-1200 and
FS-600 kits with their flow and hose limits), the chemical price list, and the project schedule window.

WHAT WAS AMBIGUOUS OR BROKEN
    The “certified lower bound” the objective is measured against is an LP relaxation, not a buildable plan: it charges a flat $430/day for
    every chain, allows continuous shifts, and does not price spent-solution disposal ($686 for chain C7 alone). A score of 1.0 is therefore


                                                                 page 11 of 48
PLE 29-task report · AIODAM                                                                                                 11 November 2026




    not reachable by construction.
    Partial softening is permitted for top-up water (UFC 3-230-13 6-2.1/3-2.1). My first model assumed full softening and overpriced
    every top-up.
    The 14 skid-days and 22 shifts looked improvable. A batch of 110 re-planned schedulers found nothing better, which is how the floor
    was measured rather than asserted.

WHAT I DID AND WHAT CAME OUT
    Cascaded the chains C1→C5→C4→C8→C6→C7 and C3→C2, reusing displaced water and cutting discharge from 200,150 to
    173,687 gal.
    Scheduled 22 twelve-hour shifts at full loading; all eight top-ups run in one shift on an FS-600-A with 25% partial softening (softener
    load 13,260 → 3,315 gal).
    Kept C1–C3 on FS-1200: an FS-600 cannot deliver 132 ft at 624 gpm through its narrower hose.
    Moved four chains to a polysilicate-azole programme (PS-330 at $139 against MO-500 at $172), which required changing the TA-90
    rows, lowering the turnover threshold from 100 to 80 ppm and placing an 18-day order on day 9.19.
    Programme cost $71,554.66 (verified in the result JSON), 14 unit-days, 22 shifts; all 24 families green on all eight chains.
    The remaining distance to the bound sits in the relaxation itself, not in a plan I failed to find.

HOW IT WAS VERIFIED
    The task's real grader (run_grade.py) on the delivered package.
    An independent recomputation (verify501.py) of discharge, effluent, unit-days, shifts and chemistry — to the cent, and the total to
    0.4%.
    Seven deliberate breakages of the package, each run through the real grader; all seven were noticed.
    The official tests/test.sh, run natively on the host, returned the same reward; the task's verifier image in Docker on an isolated VM,
    and a chroot unpacking of the same image, had already returned that number earlier (three paths, one number).

MY MISTAKES IN THIS TASK
    My first schedule put the top-ups inside the 14-day membrane cure. No checker saw it, because the schedule never reached the
    evaluation; it was caught by reading the acceptance criteria myself. Rebuilding took the score from 0.762 to 0.919.
    An independent money check caught me twice on which chain loses displacement in the cascade.
    At the final consistency pass the output path still held the previous package (0.919489) while the recorded score belonged to a later
    one. Fixed by copying the later package onto the output path and re-measuring through both entries.

WHAT REMAINS UNVERIFIED
Nothing about the score. What is not claimed: that the plan is optimal — only that the search stopped at a measured floor and that the
bound is a relaxation.

USE OF REFERENCE OR GRADER MATERIAL
No reference construction was used. The LP bound is part of the grader's scoring and was read to interpret the residual gap, not to build
the plan.


PART A · CHECKS · ARCHITECTURAL / SITE
```


## deliverables/MOP.md

```text
# Method of Procedure - PCW cleaning, flushing and passivation

Mobilisation hour 0. Twelve-hour crew shifts; calendar days are [24k, 24k+24).
The turnover fill is drawn through the site softener SF-1 at SV-2: the makeup is 288 mg/L total hardness, above the 250 mg/L a closed chilled system may take.

## C1 PCW primary header and pump gallery
Flushed at 1182.6 gpm on FS-1200-A; the passivation and turnover circulate at 1183 and 1183 gpm behind the reinstated components; 6 rinse displacements; molybdate-azole programme with molybdate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C1: rinse adequacy (margin 0.0198)
Other margins: skid capability 0.2601, rinse adequacy 0.0198, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.5000.

## C2 PCW heat exchanger bay A
Flushed at 624.3 gpm on FS-1200-A; the passivation and turnover circulate at 624 and 624 gpm behind the reinstated components; 6 rinse displacements; molybdate-azole programme with molybdate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C2: rinse adequacy (margin 0.0198)
Other margins: skid capability 0.5415, rinse adequacy 0.0198, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.5000.

## C3 PCW heat exchanger bay B
Flushed at 721.1 gpm on FS-1200-A; the passivation and turnover circulate at 664 and 664 gpm behind the reinstated components; 6 rinse displacements; polysilicate-azole programme with polysilicate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C3: rinse adequacy (margin 0.0198)
Other margins: skid capability 0.4901, rinse adequacy 0.0198, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.2105.

## C4 PCW CDU distribution north
Flushed at 360.6 gpm on FS-600-A; the passivation and turnover circulate at 361 and 324 gpm behind the reinstated components; 8 rinse displacements; polysilicate-azole programme with polysilicate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C4: rinse adequacy (margin 0.0107)
Other margins: skid capability 0.0495, rinse adequacy 0.0107, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.6667.

## C5 PCW CDU distribution south
Flushed at 476.6 gpm on FS-600-A; the passivation and turnover circulate at 390 and 320 gpm behind the reinstated components; 8 rinse displacements; polysilicate-azole programme with polysilicate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C5: rinse adequacy (margin 0.0107)
Other margins: skid capability 0.0353, rinse adequacy 0.0107, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.6667.

## C6 PCW CDU distribution west
Flushed at 360.6 gpm on FS-600-A; the passivation and turnover circulate at 324 and 318 gpm behind the reinstated components; 8 rinse displacements; polysilicate-azole programme with polysilicate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C6: rinse adequacy (margin 0.0289)
Other margins: skid capability 0.0494, rinse adequacy 0.0289, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.6667.

## C7 PCW fan-wall coil branch
Flushed at 295.7 gpm on FS-600-A; the passivation and turnover circulate at 273 and 273 gpm behind the reinstated components; 8 rinse displacements; polysilicate-azole programme with polysilicate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C7: rinse adequacy (margin 0.0107)
Other margins: skid capability 0.0998, rinse adequacy 0.0107, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.3333.

## C8 PCW makeup and fill riser
Flushed at 151.3 gpm on FS-600-A; the passivation and turnover circulate at 151 and 151 gpm behind the reinstated components; 6 rinse displacements; polysilicate-azole programme with polysilicate passivation; RETAIN_AND_TOP_UP.
GOVERNING CONSTRAINT C8: rinse adequacy (margin 0.0359)
Other margins: skid capability 0.6195, rinse adequacy 0.0359, discharge volume cap 0.2783, water budget 0.0611, temporary power 0.7500.

## Safety and hazard communication

This section discharges 29 CFR 1910.1200. Every product on site carries an SDS (1910.1200(g)): CL-100, CL-210, MO-500, PS-330, TA-90, PD-45.
Containers are labelled (1910.1200(f)). A written hazard communication program (1910.1200(e)) is held on site. Workers are trained (1910.1200(h)).

```


## deliverables/acceptance_matrix.csv

```text
circuit_id,gate,parameter,method,sample_point_tag,threshold_op,threshold_value,units,basis_doc,basis_clause
C1,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C1,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C1,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C1,PASSIVATION,inhibitor residual Mo,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C1,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C1,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C1,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C1,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C1,TURNOVER,inhibitor residual Mo,colorimetric,DW-1,>=,100,mg/L,UFC 3-230-13,6-2.2.3
C1,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3
C2,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C2,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C2,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C2,PASSIVATION,inhibitor residual Mo,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C2,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C2,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C2,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C2,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C2,TURNOVER,inhibitor residual Mo,colorimetric,DW-1,>=,100,mg/L,UFC 3-230-13,6-2.2.3
C2,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3
C3,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C3,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C3,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C3,PASSIVATION,inhibitor residual SiO2,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C3,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C3,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C3,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C3,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C3,TURNOVER,inhibitor residual SiO2,colorimetric,DW-1,>=,80,mg/L,UFC 3-230-13,6-2.2.5
C3,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3
C4,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C4,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C4,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C4,PASSIVATION,inhibitor residual SiO2,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C4,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C4,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C4,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C4,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C4,TURNOVER,inhibitor residual SiO2,colorimetric,DW-1,>=,80,mg/L,UFC 3-230-13,6-2.2.5
C4,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3
C5,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C5,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C5,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C5,PASSIVATION,inhibitor residual SiO2,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C5,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C5,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C5,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C5,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C5,TURNOVER,inhibitor residual SiO2,colorimetric,DW-1,>=,80,mg/L,UFC 3-230-13,6-2.2.5
C5,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3
C6,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C6,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C6,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C6,PASSIVATION,inhibitor residual SiO2,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C6,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C6,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C6,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C6,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C6,TURNOVER,inhibitor residual SiO2,colorimetric,DW-1,>=,80,mg/L,UFC 3-230-13,6-2.2.5
C6,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3
C7,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C7,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C7,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C7,PASSIVATION,inhibitor residual SiO2,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C7,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C7,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C7,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C7,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C7,TURNOVER,inhibitor residual SiO2,colorimetric,DW-1,>=,80,mg/L,UFC 3-230-13,6-2.2.5
C7,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3
C8,RINSE,conductivity,meter,DW-1,<=,676.5,microS/cm,UFC 3-230-13,5-7.3.2.2(e)
C8,RINSE,pH lower limit,meter,DW-1,>=,7.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C8,RINSE,pH upper limit,meter,DW-1,<=,8.5,pH,UFC 3-230-13,5-7.3.2.2(e)
C8,PASSIVATION,inhibitor residual SiO2,colorimetric,DW-1,>=,40,mg/L,UFC 3-230-13,Table 5-15
C8,PASSIVATION,pH,meter,DW-1,>=,7.5,pH,UFC 3-230-13,Table 5-15
C8,PASSIVATION,film formation hold,log,DW-1,>=,14,days,UFC 3-230-13,Table 5-15
C8,DISCHARGE,pH,meter,MH-14,>=,6.0,pH,SW-03 IWD-2026-0417,cl.2 local limits
C8,DISCHARGE,temperature,meter,MH-14,<=,104,degF,SW-03 IWD-2026-0417,cl.2 local limits
C8,TURNOVER,inhibitor residual SiO2,colorimetric,DW-1,>=,80,mg/L,UFC 3-230-13,6-2.2.5
C8,TURNOVER,total aerobic bacteria,plate count,DW-1,<=,1000,CFU/ml,UFC 3-230-13,Table 6-3

```


## deliverables/chemistry_plan.csv

```text
circuit_id,stage,program,product_id,active_species,target_ppm,basis_volume_gal,product_quantity,product_unit,packages,ph_target_lo,ph_target_hi,hold_days,source_doc,source_clause
C1,CHEM_CLEAN,,CL-100,,,32983.4,49.4751,gal,10,9,11,,VD-06 Rev B,Note 4a
C1,CHEM_CLEAN,,PD-45,polymer,4,32983.4,0.3023,gal,1,9,11,,VD-06 Rev B,Application Rate
C1,PASSIVATE,molybdate-azole,MO-500,Mo,40,32983.4,10.0947,gal,3,8.5,8.5,14,VD-06 Rev B,Application Rate
C1,FINAL_FILL,molybdate-azole,MO-500,Mo,100,32983.4,25.2368,gal,6,8.5,9.5,,VD-06 Rev B,Application Rate
C2,CHEM_CLEAN,,CL-100,,,7946.0,11.9190,gal,3,9,11,,VD-06 Rev B,Note 4a
C2,CHEM_CLEAN,,PD-45,polymer,4,7946.0,0.0728,gal,1,9,11,,VD-06 Rev B,Application Rate
C2,PASSIVATE,molybdate-azole,MO-500,Mo,40,7946.0,2.4319,gal,1,8.5,8.5,14,VD-06 Rev B,Application Rate
C2,FINAL_FILL,molybdate-azole,MO-500,Mo,100,7946.0,6.0798,gal,2,8.5,9.5,,VD-06 Rev B,Application Rate
C3,CHEM_CLEAN,,CL-100,,,8553.2,12.8298,gal,3,9,11,,VD-06 Rev B,Note 4a
C3,CHEM_CLEAN,,PD-45,polymer,4,8553.2,0.0784,gal,1,9,11,,VD-06 Rev B,Application Rate
C3,PASSIVATE,polysilicate-azole,PS-330,SiO2,40,8553.2,2.4263,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C3,FINAL_FILL,polysilicate-azole,PS-330,SiO2,80,8553.2,4.8526,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C4,CHEM_CLEAN,,CL-100,,,8536.6,12.8049,gal,3,9,11,,VD-06 Rev B,Note 4a
C4,CHEM_CLEAN,,PD-45,polymer,4,8536.6,0.0782,gal,1,9,11,,VD-06 Rev B,Application Rate
C4,PASSIVATE,polysilicate-azole,PS-330,SiO2,40,8536.6,2.4216,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C4,FINAL_FILL,polysilicate-azole,PS-330,SiO2,80,8536.6,4.8432,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C4,PASSIVATE,polysilicate-azole,TA-90,TTA,10,8536.6,0.1600,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C4,FINAL_FILL,polysilicate-azole,TA-90,TTA,10,8536.6,0.1600,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C5,CHEM_CLEAN,,CL-100,,,8985.8,13.4787,gal,3,9,11,,VD-06 Rev B,Note 4a
C5,CHEM_CLEAN,,PD-45,polymer,4,8985.8,0.0824,gal,1,9,11,,VD-06 Rev B,Application Rate
C5,PASSIVATE,polysilicate-azole,PS-330,SiO2,40,8985.8,2.5490,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C5,FINAL_FILL,polysilicate-azole,PS-330,SiO2,80,8985.8,5.0981,gal,2,7.5,8.5,,VD-06 Rev B,Application Rate
C5,PASSIVATE,polysilicate-azole,TA-90,TTA,10,8985.8,0.1684,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C5,FINAL_FILL,polysilicate-azole,TA-90,TTA,10,8985.8,0.1684,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C6,CHEM_CLEAN,,CL-210,,,5677.1,11.3542,gal,3,7.5,8.5,,VD-06 Rev B,Note 4a
C6,CHEM_CLEAN,,PD-45,polymer,4,5677.1,0.0520,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C6,PASSIVATE,polysilicate-azole,PS-330,SiO2,40,5677.1,1.6104,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C6,FINAL_FILL,polysilicate-azole,PS-330,SiO2,80,5677.1,3.2209,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C6,PASSIVATE,polysilicate-azole,TA-90,TTA,10,5677.1,0.1064,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C6,FINAL_FILL,polysilicate-azole,TA-90,TTA,10,5677.1,0.1064,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C7,CHEM_CLEAN,,CL-100,,,6036.0,9.0540,gal,2,9,11,,VD-06 Rev B,Note 4a
C7,CHEM_CLEAN,,PD-45,polymer,4,6036.0,0.0553,gal,1,9,11,,VD-06 Rev B,Application Rate
C7,PASSIVATE,polysilicate-azole,PS-330,SiO2,40,6036.0,1.7123,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C7,FINAL_FILL,polysilicate-azole,PS-330,SiO2,80,6036.0,3.4245,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C7,PASSIVATE,polysilicate-azole,TA-90,TTA,10,6036.0,0.1131,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C7,FINAL_FILL,polysilicate-azole,TA-90,TTA,10,6036.0,0.1131,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C8,CHEM_CLEAN,,CL-210,,,5790.8,11.5816,gal,3,7.5,8.5,,VD-06 Rev B,Note 4a
C8,CHEM_CLEAN,,PD-45,polymer,4,5790.8,0.0531,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate
C8,PASSIVATE,polysilicate-azole,PS-330,SiO2,40,5790.8,1.6427,gal,1,7.5,8.5,14,VD-06 Rev B,Application Rate
C8,FINAL_FILL,polysilicate-azole,PS-330,SiO2,80,5790.8,3.2854,gal,1,7.5,8.5,,VD-06 Rev B,Application Rate

```


## deliverables/constraint_register.csv

```text
id,document,clause,requirement,value,items_bound
R01,UFC 3-230-13,5-3.7,Water velocity sufficient to keep foulants in suspension during flushing and cleaning,3.28 ft/s (1 m/s) minimum,all circuits
R02,23 25 16,3.3,Project minimum flushing velocity by circuit class,"4.0 ft/s for circuits serving a plate heat exchanger or cold-plate device, 2.0 ft/s otherwise",all circuits
R03,DOE-HDBK-1012/3-92,Laminar and Turbulent Flow,Flow is turbulent above a Reynolds number of 3500,Re > 3500,all circuits
R04,DOE-HDBK-1012/3-92,Darcy's Equation; Minor Losses; Centrifugal Pumps in Parallel,Head loss by the Darcy equation over the straight length plus the equivalent length of the fittings; parallel pumps share the flow and each develops the head at its own flow,"h = f (L+Leq)/D v^2/2g; equal head, added flow",all circuits
R05,M-601 and M-602,Pipe Schedule; Equipment Schedule,"Circuit volume from the line inside diameters, lengths and parallel leg counts plus the component hold-up volumes",per schedule,all circuits
R06,UFC 3-230-13,6-2.2.2,Nitrite and nitrite-blend programs are excluded above a chloride plus sulfate concentration of 800 ppm,(Cl + SO4) > 800 ppm excludes nitrite,all circuits
R07,UFC 3-230-13,6-2.2.5,The polysilicate-azole program is the closed-system program where aluminium metal is present,polysilicate-azole,all circuits
R08,UFC 3-230-13,5-7.3.2,Galvanized steel is passivated with a phosphate or polysilicate treatment; conventional high-pH programs initiate white rust,phosphate or polysilicate; pH above 8.5 produces white rust,all circuits
R09,UFC 3-230-13,10-1.1,Pre-operational cleaning agents have a pH of 9 to 11; systems containing galvanized steel or aluminium must not be subjected to a pH above 8.5,pH 9 to 11; not above 8.5 with galvanized or aluminium,all circuits
R10,UFC 3-230-13,5-3.7,"Polymer dispersant at 4 to 5 ppm active polymer, to 20 ppm for heavily fouled systems",4 to 5 ppm active polymer,all circuits
R11,VD-06,"Quotation Rev B, application rate","Cleaner application rate per 1,000 gallons of the volume dosed","1.5 gal/1,000 gal (CL-100); 2.0 gal/1,000 gal (CL-210)",all circuits
R12,UFC 3-230-13,6-2.2.3,Molybdate-azole maintenance: 100 to 125 ppm as Mo at pH 8.5 to 9.5,"100-125 ppm Mo, pH 8.5-9.5",all circuits
R13,UFC 3-230-13,6-2.2.5,Polysilicate-azole maintenance: 80 to 100 ppm as SiO2 at pH 7.5 to 8.9,"80-100 ppm SiO2, pH 7.5-8.9",all circuits
R14,UFC 3-230-13,6-2.2.3 step 4,"Where copper is present, maintain a minimum of 10 ppm tolyltriazole",10 ppm TTA minimum,all circuits
R15,UFC 3-230-13,Table 5-14 and Table 5-15,Passivation film formation takes 14 to 21 days at the tabulated dosage and pH,14 to 21 days,all circuits
R16,UFC 3-230-13,5-7.3.2.2(b) and (d),"Circulate the cleaning formulation for 24 hours; after refilling, circulate 30 minutes before sampling",24 hours; 30 minutes,all circuits
R17,UFC 3-230-13,5-7.3.2.2(e),"Rinse until the system water matches the makeup supply for clarity and suspended solids, conductivity within 10 percent and pH within 0.3 units from 7.5 to 8.5","conductivity +/-10 %, pH +/-0.3 within 7.5 to 8.5",all circuits
R18,23 25 16,3.6,Rinse displacement model and mixing efficiency,C_n = C_mk + (C_0 - C_mk) exp(-eta n); eta 0.62 / 0.45,all circuits
R19,UFC 3-230-13,Table 5-15,"Molybdate passivation at 40 to 60 ppm as Mo, pH 7.5 to 8.5","40-60 ppm Mo, pH 7.5-8.5",all circuits
R20,UFC 3-230-13,Table 5-15,"Polysilicate passivation at 40 to 50 ppm as SiO2, pH 7.5 to 8.5","40-50 ppm SiO2, pH 7.5-8.5",all circuits
R21,40 CFR 403,403.5(b)(2),No discharge to the POTW with a pH lower than 5.0,pH not lower than 5.0,all circuits
R22,40 CFR 403,403.5(b)(5),No heat in quantities that take the POTW treatment plant above 40 C (104 F),104 degF (40 degC),all circuits
R23,SW-03,Local limits,"Local limits at the authorised discharge point: pH 6.0 to 9.0, total molybdenum 0.5 mg/L, total zinc 1.0 mg/L and total copper 1.0 mg/L",pH 6.0-9.0; Mo 0.5 mg/L; Zn 1.0 mg/L; Cu 1.0 mg/L,all circuits
R24,SW-03 Rev B,2. Authorized volume,The discharge authorisation caps the programme total sewered volume,"500,000 gallons for the programme",all circuits
R25,SW-01,Sections 1 and 2,"Construction water is limited per calendar day, with trucked water available up to a further daily limit","62,000 gal/day metered; 30,000 gal/day trucked",all circuits
R26,SW-01,Section 3,"Three flush skids are available to this scope: two FS-600 units (FS-600-A, FS-600-B) and one FS-1200 unit (FS-1200-A); no unit may serve two circuits at the same time","FS-600-A, FS-600-B, FS-1200-A",all circuits
R27,SW-04,Temporary Power,Simultaneous connected load of flush equipment at a location must not exceed the available kW,45 kW at ROOF; 90 kW at the Level 3 locations,all circuits
R28,"M-602 with VD-03, VD-04, VD-05",Isolation and bypass provision,"Every component with a temporary bypass provision is bypassed for the initial flush and the chemical cleaning, and a temporary strainer is fitted and cleaned at every drain",full-size bypass spool; 40 mesh temporary strainer,all circuits
R29,23 21 16,3.3,"Hydrostatic test at 1.5 times the operating pressure, not less than 125 psig, held 120 minutes","1.5 x 85 psig, minimum 125 psig, 120 minutes",all circuits
R30,VD-05,Temporary hose; temporary strainer,Temporary hose and strainer are rated 150 psig and the lowest rating in the assembled temporary circuit governs,150 psig,all circuits
R31,M-601 Notes / M-PID-103 Rev C,Note 5,Rev C sheets govern; M-PID-103 Rev A is superseded and must not be used,Rev C governs,all circuits
R32,CX-01,TM-4,"No flushing, cleaning or passivation activity may finish after the turnover milestone",day 47 from mobilisation,all circuits
R33,VD-06,Lead Time,Each product must be ordered at least its quoted lead time before first use,5 to 21 calendar days by product,all circuits
R34,23 25 16,1.5,"Flushing, testing and water-treatment log content required for turnover","velocity, strainer, circulation time, rinse results, residual and hold, discharge events",all circuits
R35,23 25 16,1.6,"Submit the register of the requirements applied, with the document and clause of each",design basis and register,all circuits
R36,29 CFR 1910.1200,"(e), (f), (g), (h)","Safety data sheets readily accessible, container labels, a written hazard communication program and employee training","SDS, labels, written program, training",all circuits
R37,23 25 16,1.6,State the governing constraint and its relative margin for each circuit in the design basis,GOVERNING CONSTRAINT <circuit>: <family> (margin <value>),all circuits
R40,01 91 13,1.3,A test result on a commissioning record is accepted only together with the evidence it cites (pressure chart) on file; a CX-02 entry whose chart is not on file is not an accepted test,"charts on file for C1, C2, C4, C8 only",all circuits
R41,UFC 3-230-13,Table 6-1; 6-2.2.5 step 1,Makeup to a closed chilled system under 250 ppm total hardness as CaCO3; makeup harder than that is softened,250 ppm as CaCO3,all circuits
R42,C-101,SV-2 / SF-1,"Softened water from SV-2 (no blending connection) is bounded by softener SF-1: 88,000 grains between its once-a-day regenerations, at most about 5,620 gal a calendar day on the 288 mg/L supply","88,000 grains per regeneration",all circuits
R43,VD-04,Limits,Aluminium cold plates: fluid pH 7.0 to 8.5 in service,pH 7.0 to 8.5,all circuits

```


## deliverables/flush_plan.csv

```text
circuit_id,stage,skid_id,skids_parallel,flow_gpm,governing_segment,governing_id_in,velocity_fps,reynolds,required_head_ft,skid_head_avail_ft,duration_h,circuit_volume_gal,system_volume_gal,passes,water_source,water_volume_gal,discharge_route,discharge_volume_gal,discharge_ph,discharge_temp_f,test_pressure_psig,protection,turnover_method
C1,INITIAL_FLUSH,FS-1200-A,1,1182.6,1043,6.065,3.283,153650,88.37,119.44,0.465,32983.4,32983.4,1,MAKEUP,32983.4,SEWER,32983.4,7.6,68,,temporary bypass on ; 40 mesh strainer,
C1,CHEM_CLEAN,FS-1200-A,1,1182.6,1043,6.065,3.283,153650,88.37,119.44,24.000,32983.4,32983.4,,MAKEUP,32983.4,SEWER,32983.4,8.4,68,,temporary bypass on ; 40 mesh strainer,
C1,DRAIN_RINSE,FS-1200-A,1,1182.6,1043,6.065,3.283,153650,88.37,119.44,2.789,32983.4,32983.4,6,MAKEUP,197900.4,SEWER,164917.0,8.4,68,,,
C1,PASSIVATE,FS-1200-A,1,1182.6,1043,6.065,3.283,153650,88.37,119.44,0.965,32983.4,32983.4,,NONE,0.0,NONE,0.0,,,,,
C1,HOLD,NONE,,,,,,,,,336.000,32983.4,32983.4,,NONE,0.0,NONE,0.0,,,,,
C1,FINAL_FILL,FS-600-A,1,500.0,1043,6.065,1.388,64963,90.99,129.33,0.500,32983.4,32983.4,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,0.0,NONE,0.0,,,,5 micron fine filtration through the temporary filter skid at ,RETAIN_AND_TOP_UP
C2,INITIAL_FLUSH,FS-1200-A,1,624.3,2011,7.981,4.004,246559,48.84,185.65,0.149,5596.0,7946.0,1,MAKEUP,5596.0,SEWER,5596.0,7.6,68,,temporary bypass on HX-201A; 40 mesh strainer,
C2,CHEM_CLEAN,FS-1200-A,1,624.3,2011,7.981,4.004,246559,48.84,185.65,24.000,5596.0,7946.0,,MAKEUP,5596.0,SEWER,5596.0,8.4,68,,temporary bypass on HX-201A; 40 mesh strainer,
C2,DRAIN_RINSE,FS-1200-A,1,624.3,2011,7.981,4.004,246559,48.84,185.65,0.896,5596.0,7946.0,6,CASCADE_FROM:C3,33576.0,SEWER,27980.0,8.4,68,,,
C2,PASSIVATE,FS-1200-A,1,624.3,2011,7.981,4.004,246559,48.84,185.65,0.649,5596.0,7946.0,,NONE,0.0,NONE,0.0,,,,,
C2,HOLD,NONE,,,,,,,,,336.000,5596.0,7946.0,,NONE,0.0,NONE,0.0,,,,,
C2,FINAL_FILL,FS-600-A,1,500.0,2011,7.981,3.207,197468,123.89,129.33,0.563,5596.0,7946.0,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,2350.0,NONE,0.0,,,,5 micron fine filtration through the temporary filter skid at ,RETAIN_AND_TOP_UP
C3,HYDRO,NONE,,,,,,,,,2.000,6203.2,8553.2,,MAKEUP,6203.2,SEWER,6203.2,7.6,68,127.5,,
C3,INITIAL_FLUSH,FS-1200-A,1,721.1,2023,6.065,4.004,187378,56.72,176.29,0.143,6203.2,8553.2,1,MAKEUP,6203.2,SEWER,6203.2,7.6,68,,temporary bypass on HX-201B; 40 mesh strainer,
C3,CHEM_CLEAN,FS-1200-A,1,721.1,2023,6.065,4.004,187378,56.72,176.29,24.000,6203.2,8553.2,,MAKEUP,6203.2,SEWER,6203.2,8.4,68,,temporary bypass on HX-201B; 40 mesh strainer,
C3,DRAIN_RINSE,FS-1200-A,1,721.1,2023,6.065,4.004,187378,56.72,176.29,0.860,6203.2,8553.2,6,MAKEUP,37219.2,SEWER,31016.0,8.4,68,,,
C3,PASSIVATE,FS-1200-A,1,664.4,2023,6.065,3.689,172645,51.61,181.77,0.656,6203.2,8553.2,,NONE,0.0,NONE,0.0,,,,,
C3,HOLD,NONE,,,,,,,,,336.000,6203.2,8553.2,,NONE,0.0,NONE,0.0,,,,,
C3,FINAL_FILL,FS-600-A,1,500.0,2023,6.065,2.776,129925,123.49,129.33,0.559,6203.2,8553.2,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,2350.0,NONE,0.0,,,,5 micron fine filtration through the temporary filter skid at ,RETAIN_AND_TOP_UP
C4,INITIAL_FLUSH,FS-600-A,1,360.6,4021,6.065,4.005,187404,99.04,154.11,0.275,5956.6,8536.6,1,MAKEUP,5956.6,SEWER,5956.6,7.6,68,,"temporary bypass on CDU-301, CP-311; 40 mesh strainer",
C4,CHEM_CLEAN,FS-600-A,1,360.6,4021,6.065,4.005,187404,99.04,154.11,24.000,5956.6,8536.6,,MAKEUP,5956.6,HAUL,5956.6,8.4,68,,"temporary bypass on CDU-301, CP-311; 40 mesh strainer",
C4,DRAIN_RINSE,FS-600-A,1,360.6,4021,6.065,4.005,187404,99.04,154.11,2.202,5956.6,8536.6,8,CASCADE_FROM:C5,47652.8,HAUL,41696.2,8.4,68,,,
C4,PASSIVATE,FS-600-A,1,360.6,4021,6.065,4.005,187404,99.04,154.11,0.775,5956.6,8536.6,,NONE,0.0,NONE,0.0,,,,,
C4,HOLD,NONE,,,,,,,,,336.000,5956.6,8536.6,,NONE,0.0,NONE,0.0,,,,,
C4,FINAL_FILL,FS-600-A,1,323.9,4021,6.065,3.597,168331,154.82,159.49,0.633,5956.6,8536.6,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,2580.0,NONE,0.0,,,,"5 micron fine filtration through the temporary filter skid at CDU-301, CP-311",RETAIN_AND_TOP_UP
C5,HYDRO,NONE,,,,,,,,,2.000,6405.8,8985.8,,MAKEUP,6405.8,SEWER,6405.8,7.6,68,127.5,,
C5,INITIAL_FLUSH,FS-600-A,1,476.6,4033,4.026,4.004,124378,130.03,134.79,0.224,6405.8,8985.8,1,MAKEUP,6405.8,SEWER,6405.8,7.6,68,,"temporary bypass on CDU-302, CP-321; 40 mesh strainer",
C5,CHEM_CLEAN,FS-600-A,1,476.6,4033,4.026,4.004,124378,130.03,134.79,24.000,6405.8,8985.8,,MAKEUP,6405.8,HAUL,6405.8,8.4,68,,"temporary bypass on CDU-302, CP-321; 40 mesh strainer",
C5,DRAIN_RINSE,FS-600-A,1,476.6,4033,4.026,4.004,124378,130.03,134.79,1.792,6405.8,8985.8,8,CASCADE_FROM:C1,51246.4,HAUL,44840.6,8.4,68,,,
C5,PASSIVATE,FS-600-A,1,390.4,4033,4.026,3.280,101882,102.29,149.74,0.773,6405.8,8985.8,,NONE,0.0,NONE,0.0,,,,,
C5,HOLD,NONE,,,,,,,,,336.000,6405.8,8985.8,,NONE,0.0,NONE,0.0,,,,,
C5,FINAL_FILL,FS-600-A,1,320.4,4033,4.026,2.692,83615,154.97,160.01,0.634,6405.8,8985.8,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,2580.0,NONE,0.0,,,,"5 micron fine filtration through the temporary filter skid at CDU-302, CP-321",RETAIN_AND_TOP_UP
C6,HYDRO,NONE,,,,,,,,,2.000,4227.1,5677.1,,MAKEUP,4227.1,SEWER,4227.1,7.6,68,127.5,,
C6,INITIAL_FLUSH,FS-600-A,1,360.6,4041,6.065,4.005,187404,102.12,154.11,0.195,4227.1,5677.1,1,MAKEUP,4227.1,SEWER,4227.1,7.6,68,,temporary bypass on CP-331; 40 mesh strainer,
C6,CHEM_CLEAN,FS-600-A,1,360.6,4041,6.065,4.005,187404,102.12,154.11,24.000,4227.1,5677.1,,MAKEUP,4227.1,HAUL,4227.1,8.1,68,,temporary bypass on CP-331; 40 mesh strainer,
C6,DRAIN_RINSE,FS-600-A,1,360.6,4041,6.065,4.005,187404,102.12,154.11,1.563,4227.1,5677.1,8,CASCADE_FROM:C8,33816.8,HAUL,29589.7,8.1,68,,,
C6,PASSIVATE,FS-600-A,1,324.2,4041,6.065,3.600,168487,92.03,159.45,0.717,4227.1,5677.1,,NONE,0.0,NONE,0.0,,,,,
C6,HOLD,NONE,,,,,,,,,336.000,4227.1,5677.1,,NONE,0.0,NONE,0.0,,,,,
C6,FINAL_FILL,FS-600-A,1,317.9,4041,6.065,3.530,165213,155.51,160.37,0.576,4227.1,5677.1,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,1450.0,NONE,0.0,,,,5 micron fine filtration through the temporary filter skid at CP-331,RETAIN_AND_TOP_UP
C7,HYDRO,NONE,,,,,,,,,2.000,4086.0,6036.0,,MAKEUP,4086.0,SEWER,4086.0,7.6,68,127.5,,
C7,INITIAL_FLUSH,FS-600-A,1,295.7,5011,6.065,3.284,153676,107.21,163.37,0.230,4086.0,6036.0,1,MAKEUP,4086.0,SEWER,4086.0,7.6,68,,temporary bypass on CL-401; 40 mesh strainer,
C7,CHEM_CLEAN,FS-600-A,1,295.7,5011,6.065,3.284,153676,107.21,163.37,24.000,4086.0,6036.0,,MAKEUP,4086.0,HAUL,4086.0,8.4,68,,temporary bypass on CL-401; 40 mesh strainer,
C7,DRAIN_RINSE,FS-600-A,1,295.7,5011,6.065,3.284,153676,107.21,163.37,1.842,4086.0,6036.0,8,CASCADE_FROM:C6,32688.0,HAUL,28602.0,8.4,68,,,
C7,PASSIVATE,FS-600-A,1,273.1,5011,6.065,3.033,141930,102.19,165.33,0.749,4086.0,6036.0,,NONE,0.0,NONE,0.0,,,,,
C7,HOLD,NONE,,,,,,,,,336.000,4086.0,6036.0,,NONE,0.0,NONE,0.0,,,,,
C7,FINAL_FILL,FS-600-A,1,273.1,5011,6.065,3.033,141930,151.12,165.33,0.619,4086.0,6036.0,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,1950.0,NONE,0.0,,,,5 micron fine filtration through the temporary filter skid at ,RETAIN_AND_TOP_UP
C8,INITIAL_FLUSH,FS-600-A,1,151.3,6013,3.068,3.283,77721,66.93,175.89,0.638,5790.8,5790.8,1,MAKEUP,5790.8,SEWER,5790.8,7.6,68,,temporary bypass on ; 40 mesh strainer,
C8,CHEM_CLEAN,FS-600-A,1,151.3,6013,3.068,3.283,77721,66.93,175.89,24.000,5790.8,5790.8,,MAKEUP,5790.8,HAUL,5790.8,8.1,68,,temporary bypass on ; 40 mesh strainer,
C8,DRAIN_RINSE,FS-600-A,1,151.3,6013,3.068,3.283,77721,66.93,175.89,3.827,5790.8,5790.8,6,CASCADE_FROM:C4,34744.8,HAUL,28954.0,8.1,68,,,
C8,PASSIVATE,FS-600-A,1,151.3,6013,3.068,3.283,77721,66.93,175.89,1.138,5790.8,5790.8,,NONE,0.0,NONE,0.0,,,,,
C8,HOLD,NONE,,,,,,,,,336.000,5790.8,5790.8,,NONE,0.0,NONE,0.0,,,,,
C8,FINAL_FILL,FS-600-A,1,151.3,6013,3.068,3.283,77721,66.93,175.89,0.500,5790.8,5790.8,,MAKEUP 25% softened through SV-2 - partially softened split-stream blend with unsoftened site supply,0.0,NONE,0.0,,,,5 micron fine filtration through the temporary filter skid at ,RETAIN_AND_TOP_UP

```


## deliverables/resources.json

```text
{
 "skid_assignment": {
  "C1": "FS-1200-A",
  "C2": "FS-1200-A",
  "C3": "FS-1200-A",
  "C4": "FS-600-A",
  "C5": "FS-600-A",
  "C6": "FS-600-A",
  "C7": "FS-600-A",
  "C8": "FS-600-A"
 },
 "fill_assignment": {
  "C1": "FS-600-A",
  "C2": "FS-600-A",
  "C3": "FS-600-A",
  "C4": "FS-600-A",
  "C5": "FS-600-A",
  "C6": "FS-600-A",
  "C7": "FS-600-A",
  "C8": "FS-600-A"
 },
 "water_profile": {
  "total_gal": 613462.0,
  "metered_gal": 570151.1,
  "trucked_gal": 43310.9,
  "blended_usd_per_kgal": 7.1602,
  "by_day_gal": {
   "4": 65327.7,
   "5": 81956.3,
   "7": 82026.8,
   "8": 52236.7,
   "9": 48113.9,
   "10": 46013.7,
   "11": 56217.7,
   "12": 52487.3,
   "13": 41383.3,
   "14": 29602.9,
   "15": 15695.9,
   "16": 29139.6,
   "31": 13260.0
  }
 },
 "discharge_profile": {
  "sewer_gal": 348864.8,
  "haul_gal": 180088.3,
  "authorised_total_gal": 500000.0,
  "by_day_gal": {
   "4": 65327.7,
   "5": 65771.3,
   "7": 65700.6,
   "8": 45358.8,
   "9": 36314.7,
   "10": 42018.1,
   "11": 5956.6,
   "12": 5790.8,
   "13": 8454.2,
   "15": 8172.0
  }
 },
 "power_check": {
  "C1": {
   "location": "MER-1",
   "available_kw": 150.0,
   "skid_kw": 75.0
  },
  "C2": {
   "location": "MER-1",
   "available_kw": 150.0,
   "skid_kw": 75.0
  },
  "C3": {
   "location": "MER-2",
   "available_kw": 95.0,
   "skid_kw": 75.0
  },
  "C4": {
   "location": "L3-N",
   "available_kw": 90.0,
   "skid_kw": 30.0
  },
  "C5": {
   "location": "L3-S",
   "available_kw": 90.0,
   "skid_kw": 30.0
  },
  "C6": {
   "location": "L3-W",
   "available_kw": 90.0,
   "skid_kw": 30.0
  },
  "C7": {
   "location": "ROOF",
   "available_kw": 45.0,
   "skid_kw": 30.0
  },
  "C8": {
   "location": "YARD",
   "available_kw": 120.0,
   "skid_kw": 30.0
  }
 },
 "softener": {
  "sf1_capacity_gal_per_day": 5621.5,
  "softened_draw_by_day_gal": {
   "31": 3315.0
  }
 },
 "chemical_orders": [
  {
   "product_id": "CL-100",
   "lead_days": 10,
   "first_use_day": 4.02,
   "order_by_day": -7
  },
  {
   "product_id": "CL-210",
   "lead_days": 14,
   "first_use_day": 12.65,
   "order_by_day": -3
  },
  {
   "product_id": "MO-500",
   "lead_days": 21,
   "first_use_day": 8.02,
   "order_by_day": -14
  },
  {
   "product_id": "PD-45",
   "lead_days": 7,
   "first_use_day": 4.02,
   "order_by_day": -4
  },
  {
   "product_id": "PS-330",
   "lead_days": 18,
   "first_use_day": 9.19,
   "order_by_day": -10
  },
  {
   "product_id": "TA-90",
   "lead_days": 12,
   "first_use_day": 11.45,
   "order_by_day": -2
  }
 ],
 "cost": {
  "water_usd": 4392.49,
  "sewer_usd": 3279.33,
  "haul_usd": 30254.83,
  "chemicals_usd": 7408.0,
  "skid_usd": 8180.0,
  "crew_usd": 18040.0,
  "total_usd": 71554.66,
  "unit_days": 14,
  "crew_shifts": 22,
  "by_circuit_usd": {
   "C1": 12306.12,
   "C2": 4715.56,
   "C3": 4443.18,
   "C4": 10764.74,
   "C5": 11735.09,
   "C6": 8767.38,
   "C7": 9921.01,
   "C8": 8901.59
  }
 }
}
```


## deliverables/schedule.csv

```text
activity_id,circuit_id,stage,predecessor_ids,start_h,finish_h,skid_id,crew_id,water_draw_gal
C1-INIT-1,C1,INITIAL_FLUSH,,96.000,96.465,FS-1200-A,CR-1,32983.4
C1-CHEM-2,C1,CHEM_CLEAN,,96.465,108.000,FS-1200-A,CR-1,15852.9
C1-CHEM-3,C1,CHEM_CLEAN,,108.000,120.000,FS-1200-A,CR-2,16491.7
C1-CHEM-4,C1,CHEM_CLEAN,,120.000,120.465,FS-1200-A,CR-1,638.8
C1-DRAI-5,C1,DRAIN_RINSE,,120.465,121.611,FS-1200-A,CR-1,81361.2
C1-DRAI-6,C1,DRAIN_RINSE,,168.000,169.156,FS-1200-A,CR-1,82000.0
C1-DRAI-7,C1,DRAIN_RINSE,,192.000,192.487,FS-1200-A,CR-1,34539.2
C1-PASS-8,C1,PASSIVATE,,192.487,193.452,FS-1200-A,CR-1,0.0
C1-HOLD-9,C1,HOLD,,193.452,529.452,NONE,CR-1,0.0
C3-HYDR-10,C3,HYDRO,,193.452,195.452,FS-1200-A,CR-1,6203.2
C3-INIT-11,C3,INITIAL_FLUSH,,195.452,195.595,FS-1200-A,CR-1,6203.2
C3-CHEM-12,C3,CHEM_CLEAN,,195.595,204.000,FS-1200-A,CR-1,2172.4
C3-CHEM-13,C3,CHEM_CLEAN,,204.000,216.000,FS-1200-A,CR-2,3101.6
C3-CHEM-14,C3,CHEM_CLEAN,,216.000,219.595,FS-1200-A,CR-1,929.2
C3-DRAI-15,C3,DRAIN_RINSE,,219.595,220.455,FS-1200-A,CR-1,37219.2
C3-PASS-16,C3,PASSIVATE,,220.455,221.111,FS-1200-A,CR-1,0.0
C3-HOLD-17,C3,HOLD,,221.111,557.111,NONE,CR-1,0.0
C2-INIT-18,C2,INITIAL_FLUSH,,221.111,221.260,FS-1200-A,CR-1,5596.0
C2-CHEM-19,C2,CHEM_CLEAN,,221.260,228.000,FS-1200-A,CR-1,1571.5
C2-CHEM-20,C2,CHEM_CLEAN,,228.000,240.000,FS-1200-A,CR-2,2798.0
C2-CHEM-21,C2,CHEM_CLEAN,,240.000,245.260,FS-1200-A,CR-1,1226.5
C2-DRAI-22,C2,DRAIN_RINSE,,245.260,246.157,FS-1200-A,CR-1,27980.0
C2-PASS-23,C2,PASSIVATE,,246.157,246.806,FS-1200-A,CR-1,0.0
C2-HOLD-24,C2,HOLD,,246.806,582.806,NONE,CR-1,0.0
C5-HYDR-25,C5,HYDRO,,246.806,248.806,FS-600-A,CR-1,6405.8
C5-INIT-26,C5,INITIAL_FLUSH,,248.806,249.030,FS-600-A,CR-1,6405.8
C5-CHEM-27,C5,CHEM_CLEAN,,249.030,252.000,FS-600-A,CR-1,792.7
C5-CHEM-28,C5,CHEM_CLEAN,,252.000,264.000,FS-600-A,CR-2,3202.9
C5-CHEM-29,C5,CHEM_CLEAN,,264.000,273.030,FS-600-A,CR-1,2410.2
C5-DRAI-30,C5,DRAIN_RINSE,,273.030,274.822,FS-600-A,CR-1,44840.6
C5-PASS-31,C5,PASSIVATE,,274.822,275.596,FS-600-A,CR-1,0.0
C5-HOLD-32,C5,HOLD,,275.596,611.596,NONE,CR-1,0.0
C4-INIT-33,C4,INITIAL_FLUSH,,275.596,275.871,FS-600-A,CR-1,5956.6
C4-CHEM-34,C4,CHEM_CLEAN,,275.871,276.000,FS-600-A,CR-1,32.1
C4-CHEM-35,C4,CHEM_CLEAN,,276.000,288.000,FS-600-A,CR-2,2978.3
C4-CHEM-36,C4,CHEM_CLEAN,,288.000,299.871,FS-600-A,CR-1,2946.2
C4-DRAI-37,C4,DRAIN_RINSE,,299.871,300.000,FS-600-A,CR-1,2444.8
C4-DRAI-38,C4,DRAIN_RINSE,,300.000,302.073,FS-600-A,CR-2,39251.4
C4-PASS-39,C4,PASSIVATE,,302.073,302.849,FS-600-A,CR-2,0.0
C4-HOLD-40,C4,HOLD,,302.849,638.849,NONE,CR-1,0.0
C8-INIT-41,C8,INITIAL_FLUSH,,302.849,303.487,FS-600-A,CR-2,5790.8
C8-CHEM-42,C8,CHEM_CLEAN,,303.487,312.000,FS-600-A,CR-2,2054.2
C8-CHEM-43,C8,CHEM_CLEAN,,312.000,324.000,FS-600-A,CR-1,2895.4
C8-CHEM-44,C8,CHEM_CLEAN,,324.000,327.487,FS-600-A,CR-2,841.2
C8-DRAI-45,C8,DRAIN_RINSE,,327.487,331.314,FS-600-A,CR-2,28954.0
C8-PASS-46,C8,PASSIVATE,,331.314,332.452,FS-600-A,CR-2,0.0
C8-HOLD-47,C8,HOLD,,332.452,668.452,NONE,CR-1,0.0
C6-HYDR-48,C6,HYDRO,,332.452,334.452,FS-600-A,CR-2,4227.1
C6-INIT-49,C6,INITIAL_FLUSH,,334.452,334.647,FS-600-A,CR-2,4227.1
C6-CHEM-50,C6,CHEM_CLEAN,,334.647,336.000,FS-600-A,CR-2,238.3
C6-CHEM-51,C6,CHEM_CLEAN,,336.000,348.000,FS-600-A,CR-1,2113.6
C6-CHEM-52,C6,CHEM_CLEAN,,348.000,358.647,FS-600-A,CR-2,1875.3
C6-DRAI-53,C6,DRAIN_RINSE,,358.647,360.000,FS-600-A,CR-2,25611.0
C6-DRAI-54,C6,DRAIN_RINSE,,360.000,360.210,FS-600-A,CR-1,3978.7
C6-PASS-55,C6,PASSIVATE,,360.210,360.927,FS-600-A,CR-1,0.0
C6-HOLD-56,C6,HOLD,,360.927,696.927,NONE,CR-1,0.0
C7-HYDR-57,C7,HYDRO,,360.927,362.927,FS-600-A,CR-1,4086.0
C7-INIT-58,C7,INITIAL_FLUSH,,362.927,363.158,FS-600-A,CR-1,4086.0
C7-CHEM-59,C7,CHEM_CLEAN,,363.158,372.000,FS-600-A,CR-1,1505.4
C7-CHEM-60,C7,CHEM_CLEAN,,372.000,384.000,FS-600-A,CR-2,2043.0
C7-CHEM-61,C7,CHEM_CLEAN,,384.000,387.158,FS-600-A,CR-1,537.6
C7-DRAI-62,C7,DRAIN_RINSE,,387.158,389.000,FS-600-A,CR-1,28602.0
C7-PASS-63,C7,PASSIVATE,,389.000,389.750,FS-600-A,CR-1,0.0
C7-HOLD-64,C7,HOLD,,389.750,725.750,NONE,CR-1,0.0
C1-FINA-65,C1,FINAL_FILL,,744.000,744.500,FS-600-A,CR-1,0.0
C2-FINA-66,C2,FINAL_FILL,,744.500,745.078,FS-600-A,CR-1,2350.0
C3-FINA-67,C3,FINAL_FILL,,745.078,745.657,FS-600-A,CR-1,2350.0
C4-FINA-68,C4,FINAL_FILL,,745.657,746.289,FS-600-A,CR-1,2580.0
C5-FINA-69,C5,FINAL_FILL,,746.289,746.924,FS-600-A,CR-1,2580.0
C6-FINA-70,C6,FINAL_FILL,,746.924,747.500,FS-600-A,CR-1,1450.0
C7-FINA-71,C7,FINAL_FILL,,747.500,748.119,FS-600-A,CR-1,1950.0
C8-FINA-72,C8,FINAL_FILL,,748.119,748.619,FS-600-A,CR-1,0.0

```
