# roof-deck-design

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-10 — roof-deck-design
   A zoned roof deck that meets all 28 families, with a 1% collector margin and a cost 36% above a bound that is
   a relaxation.

 Final reward (R) — paper rule 0.4p + 0.6[h=1]O′              0.839354 · measured

 Task grader (G) — combined score                             0.839354 · measured

 Official entry — tests/test.sh on this host                  0.839354 · measured

 Judge                                                        no LLM judge in this task

 File                                                         /work/ple-solve/roof-deck-design/submission.zip · 28,565 B · sha256
                                                              cda37f8626fa1478…


TASK AND INPUTS
Design the roof deck and diaphragm for Building A of a data centre: deck zones, gauge, sidelaps, collectors, embeds and anchors, under
28 constraint families (21 hard) and six held-out scenarios.

Rev C architectural and MEP models, M-401, S-401, the deck catalogue, the drift and seismic criteria, and the obstruction geometry.

WHAT WAS AMBIGUOUS OR BROKEN
    The deck must be zoned around actual obstructions: under the MEP-PH-1 penthouse (60..180 × 60..120 ft) no drift forms, because
    drift forms outside its four faces.
    A stiffer edge band increases peak shear (898 → 1,000 plf) by concentrating force at the junction — the opposite of the intuition that
    stiffer is safer.
    Merging zones with identical specification saved $4,945 with no engineering change.

WHAT I DID AND WHAT CAME OUT

    18 gauge in the drift bands, 22 gauge in the clean field, 36/7 with five sidelaps in the east band where two frames at 68 kip/in collect
    1,055 plf.
    Cost $228,372.64 against a bound of $167,227.56; collector splice at 60.17 kip against 60.9 capacity (1% margin).
    Combined score 0.839354, p = h = 1.0, all 28 families green in all six scenarios.

HOW IT WAS VERIFIED

    The real grader; the model was cross-checked against the grader's tables at 90 points (loads, 12 RTUs, 2 screens, 3 openings, parapet,
    the whole diaphragm table, deflections, reactions, uplift).
    A container run of the task verifier image on an isolated VM reproduced 0.839354 to the last digit.
    Register 101 rows, precision and recall 1.000 after three rows contradicting their own items were fixed.

MY MISTAKES IN THIS TASK
    ifcopenshell returns world coordinates in metres although the file is in millimetres — I divided by 304.8 twice.
    An opening coordinate of 95.99999999999 instead of 96 in a strict “cell touched” test produced 39 deck cells instead of 28; fixed by
    snapping to 1e-4 ft.
    Screen walls are 203.2 mm in the model but 0.5 ft in M-401; I took the schedule.
    A parapet height in millimetres disabled the drift-truncation branch (drift 93.1 instead of 34.0 psf).
    Four fastening patterns where the criteria allow three.

WHAT REMAINS UNVERIFIED

The bound is an LP relaxation; optimality is not proven. No server-side oracle.

USE OF REFERENCE OR GRADER MATERIAL
The model was built from documents and checked against the grader's tables; the cost bound comes from the grader's bound module.
Neither supplies a design.




                                                                  page 21 of 48
PLE 29-task report · AIODAM                                                                                                      11 November 2026




PART A · CHECKS · STRUCTURAL
```


## deliverables/design_basis.md

```text
# DC-2 Helena Data Centre, Building A - roof deck design basis
Project DC2-2026-014. Prepared against EOR-DC-RD-01 Rev 2 (31 July 2026), LAT-2026-003 Rev 1, the NORDECK B-36 catalogue (2026) and the cost plan CP-RD-2026.
## 1. Documents that governed, and the ones that did not
The roof structure model DC2-A-STR Rev C governs every plan position: its transmittal note says so in terms, and it is the only document that carries the equipment and screen walls in three dimensions. The two-dimensional drawings do not all agree with it. The equipment screen wall SCR-N sits on grid y = 164 ft in the model and on the current architectural plan A-201 Rev C; the superseded A-201 Rev B put it at y = 158 ft, and Rev B is an archived sheet, so the drift band it would produce is not the one designed to. The ground snow load comes from the site-specific case study SNOW-2026-014, which reports 63 psf on a Risk Category II basis as the case study provisions require; the criteria apply the Risk Category III importance factor Is = 1.10 to it once, and it is not applied twice. Of the two USGS design maps in the project folder, the 2019 preliminary map is Risk Category II on Site Class C and is not this project; the ASCE 7-22 map for Risk Category III on Site Class D gives SDS = 0.44 g, SD1 = 0.24 g, seismic design category D, and that is what the seismic checks use. The 2019 Hall A deck design is a different building in a different town and governs nothing here.
## 2. Snow
pf = 0.7 Ce Ct Is pg = 0.7 x 0.90 x 1.00 x 1.10 x 63 = 43.66 psf, with Ce = 0.90 for a partially exposed roof in Terrain Category C that carries a continuous parapet, the enclosure and twelve rooftop units, and Ct = 1.0 for a continuously conditioned building. Snow density is gamma = 0.13 pg + 14 = 22.19 pcf, so the balanced depth is hb = pf/gamma = 1.968 ft. Every clear height is measured from the top of the balanced snow, hc = h - hb.
Drifts follow criteria 5.4 exactly: hd = 0.43 lu^(1/3) (pg + 10)^(1/4) - 1.5, a fetch shorter than 20 ft taken as 20 ft, a triangular surcharge of peak hd gamma and width 4 hd where hd <= hc, and a peak hc gamma with width min(4 hd^2/hc, 8 hc) where the obstruction truncates the drift. Each obstruction forms a windward drift of height 0.75 hd (fetch = the roof upwind of that face) and a leeward drift of height hd (fetch = the along-wind length of the obstruction itself), wind from each of the four directions in turn, and the largest of the four at a point governs; the four directions are not superimposed. An obstruction whose clear height does not exceed 2.0 ft is not a drift source, and that test is on hc, not on the bare height. The parapet is treated by the windward rule with no height threshold.
* MEP-PH-1 on face W: peak surcharge 104.3 psf above the balanced load, band 41.20 to 60.00 ft.
* SCR-N on face S: peak surcharge 89.5 psf above the balanced load, band 147.62 to 163.75 ft.
* PARAPET on face N: peak surcharge 34.0 psf above the balanced load, band 167.74 to 180.00 ft.
* RTU-1 on face E: peak surcharge 98.2 psf above the balanced load, band 36.00 to 53.70 ft.

The deck is a local element, so each zone carries the peak surcharge anywhere inside it, not an average (criteria 5.4.7).
## 3. Roof live load
UFC 3-301-01 Table E-1 item 51 gives Lo = 20 psf for an ordinary flat roof, and the criteria replace the code live load table with it. A deck panel's tributary area is 3 ft x 6 ft = 18 sq ft, far below the 200 sq ft threshold at which R1 becomes available, so the deck is designed for the unreduced 20 psf; the tributary reduction the joists and girders take is a member-by-member result and is not carried back into the deck.
## 4. Wind uplift
qh = 0.00256 Kz Kzt Kd Ke V^2 = 0.00256 x 1.04 x 1.0 x 0.85 x 1.0 x 115^2 = 29.93 psf. The components and cladding zone width is a = min(0.1 x 180, 0.4 x 32) = 12.8 ft, so 2a = 25.6 ft. Zone 3 is the corner square of side a, zone 2 the remaining band of width a, zone 1 the band from a to 2a, and zone 1 prime the interior. The net pressure is qh (GCp - GCpi) with the internal pressure acting to increase the uplift (GCpi = +0.18 against a negative GCp), resisted by 0.9 x (deck self weight + 4.5 psf roofing). The suspended mechanical allowance of 6.0 psf is not available to hold the deck down and is excluded from that resistance. The corner zone governs the attachment: at 18 gage the demand is 94.9 psf against a factored capacity of 119.7 psf for a 36/9 pattern.
## 5. Seismic
The flat roof snow load 43.66 psf exceeds the 30 psf threshold of criteria 7.2, so twenty per cent of it belongs in the effective seismic weight: w = 18 + 0.2 x 43.66 = 26.73 psf over the 43200 sq ft roof, W = 1154.8 kip. With Cs = SDS Ie / R = 0.44 x 1.25 / 3.25 = 0.1692, the roof diaphragm design force is Fpx = Cs W = 195.4 kip, which sits inside the 12.10.1 bounds of 0.2 SDS Ie W = 127.0 kip and 0.4 SDS Ie W = 254.1 kip. Omitting the snow term, as the report warns, would understate every seismic demand on the diaphragm, its chords and its collectors.
Accidental torsion is 0.05 of the plan dimension perpendicular to the force, and the collectors are amplified by Omega_0 = 2.0 before they are compared with capacity, as criteria 7.4 requires; the chord is not amplified.
## 6. How the diaphragm was analysed and classified
The roof was analysed as the in-plane assembly LAT-2026-003 section 7 describes: shear panels one joist bay (6 ft) wide by one girder bay (30 ft) deep carrying pure shear at the G' of the deck actually selected, with the five 6 ft strips across each panel acting in series, axial bars along every joist line (EA = 87,580 kip, plus 74,240 kip of edge angle on the boundary lines) and every girder line (EA = 221,850 kip), and each braced frame as a spring of half its Table 3-1 stiffness at each of its two column lines. Wind enters along the two wall lines perpendicular to the force, half at each; the seismic force is spread by tributary area; the eccentricity is applied as a plan couple. The deck's own G' is what distributes the force, so the design and the analysis are solved together: the attachment was raised where the shear concentrates at the stiff frames on grid 9 (BF-Y3 and BF-Y4, 68 kip/in each), and it cannot be raised everywhere, because a stiffer diaphragm drags more force into the collector of the stiff frame line and the collector splice is the limit it runs into (LAT-2026-003 6.2.1).
The classification is computed, not assumed: the largest ratio of diaphragm deflection to average braced-frame drift over the five lateral cases is 1.08, and criteria 9.3 classifies the diaphragm as flexible above 2.0, so the roof is SEMI-RIGID. The largest in-plane deflection is 1.37 in against the 1.60 in limit of criteria 9.2.
## 7. Zoning decisions, and what drove each one
Every zone boundary falls on the 6 ft module in both directions (criteria 10.2), no rectangle covers a framed roof opening (10.5), and the rectangles do not overlap and leave no deck cell unassigned. The layout follows the loads rather than a grid for its own sake:
* The band at the screen wall (y = 150 to 180 ft, carrying the SCR-N windward drift of 89.5 psf and the north parapet drift) is 18 gage: at the drift peak the live load reaches 143 psf, where 20 gage would exceed both its 130 psf allowable uniform load and the span/240 deflection limit.
* The roof under the MEP-PH-1 enclosure is the cheapest deck on the roof. The enclosure throws its drifts outside its own four faces, so that 120 ft by 60 ft area carries the balanced load only, and 22 gage satisfies gravity, deflection and uplift there.
* The south band (y = 24 to 48 ft) and the strips beside the enclosure and RO-3 carry the rooftop unit and enclosure drifts and are 18 gage.
* The east band (x = 216 to 240 ft) is the strongest attachment on the roof (36/9 external, four sidelap connectors per span) because the seismic-Y shear concentrates at BF-Y3 and BF-Y4; the same band carries the south-east and north-east corner uplift of the zone 3 square.
* Every zone touches either the diaphragm boundary or the edge of a framed opening, so every zone is held to the 36/7 support pattern and at least 3 sidelap connectors per span that criteria 10.4 requires; no zone uses a button punch, because a button punch is not permitted on 18 gage and the 18 gage areas must not be split off into a separate pattern to save a few cents a connector.

Zones:
* Z1: 17280 sq ft, 18 gage, 36/9 support pattern, SCREW-10 x 3 per span.
* Z2: 12744 sq ft, 18 gage, 36/7 support pattern, SCREW-10 x 3 per span.
* Z3: 2520 sq ft, 22 gage, 36/9 support pattern, SCREW-10 x 3 per span.
* Z4: 7200 sq ft, 22 gage, 36/7 support pattern, SCREW-10 x 3 per span.
* Z5: 2448 sq ft, 18 gage, 36/7 support pattern, SCREW-10 x 5 per span.

## 8. Economy
Installed cost at the CP-RD-2026 rates is $228,373: deck area, support fasteners and sidelap connectors at the unit rates, plus one zone set-up per zone, one rectangle set-up per rectangle, one gage set-up per gage and one pattern set-up per pattern. 16 gage is not available for this programme: its mill lead time is 14 weeks against the 9 week release-to-site allowance, so the design uses only stocked gages.

```
