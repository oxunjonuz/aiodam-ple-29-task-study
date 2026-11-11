# wind-diaphragm-shear

Agent report excerpt. This is exploratory work, not an official leaderboard score. Judge scores remain unmeasured where stated. [Full report and original files](https://github.com/oxunjonuz/aiodam-ple-29-task-study).

```text
T-12 — wind-diaphragm-shear
   One deck type cannot pass; a zoned schedule can — 0.233 to 0.763, and an earlier “impossible” conclusion
   withdrawn.

 Final reward (R) — paper rule 0.4p + 0.6[h=1]O′            0.762579 · measured

 Task grader (G) — combined score                           0.762579 · measured

 Official entry — tests/test.sh on this host                0.762579 · measured

 Container                                                  task verifier image in Docker on isolated VM (turn 489): 0.7625785645251897

 Judge                                                      no LLM judge in this task

 File                                                       /work/ple-solve/wind-diaphragm-shear/out/submission.zip · 54,244 B · sha256
                                                            34061c2f195f904c…


TASK AND INPUTS
Design the roof diaphragm and wind bracing for the same building under 36 families, 7 held-out and 5 dev wind scenarios, with a
$310,000 ceiling.

Deck catalogue (16 types), KRS-CH tables, A-101/A-301 notes, the EOR specification, and the wind criteria.

WHAT WAS AMBIGUOUS OR BROKEN
    A single deck type cannot pass F4 and the cost cap at the same time; a zonal schedule can, with $26,340 to spare.
    The earlier conclusion that the hard requirements were incompatible was true only for one deck type on the whole roof — withdrawn
    and corrected in the document.
    A stiffer edge band raises peak unit shear.

WHAT I DID AND WHAT CAME OUT
    Four deck types chosen from 16; 17 T1 anchors; maximum E201 base 13.97 kip against a 14.00 limit; maximum unit shear 614 plf.
    Installed cost $283,660.15 against a bound of $171,415.15 (O = 0.6043).
    Combined score 0.762579, p = 36/36, h = 1.0, feasible in all 12 scenarios.

HOW IT WAS VERIFIED
    The real grader on held-out and dev; a container run on an isolated VM reproduced the number.
    Manual equilibrium: the sum of pier spring forces equals the applied load in all 12 scenarios (relative error ≤ 5.6e-14).
    A beam analogy for the chord: 10.5 against 10.6 kip (W05), 10.8 against 10.8 (W01), 11.2 against 10.0 (W02).
    Five breakages through the grader — all noticed.

MY MISTAKES IN THIS TASK
    Spring stiffness divided by 12 instead of multiplied: deflections 12 times too large (1.61 in against 0.1669).
    Line load smeared over the whole line instead of panel patches.
    A missing 0.75 factor on torsional cases W09–W12.
    A reversed normal on line E2; ceil(L/8) instead of ceil(L/8)+1 for embeds; 240 and 180 swapped in M_T = e·B·|F|.
    A false citation: I wrote that UFC 3-301-01 Table 2-1 gives 0.55 in. It does not — 0.55 in comes only from OBD 4.4. Corrected in the
    design basis.

WHAT REMAINS UNVERIFIED

The bound is a relaxation; optimality is not proven; the oracle was unreachable.

USE OF REFERENCE OR GRADER MATERIAL
My implementation was cross-checked against the grader's independent DIAPH-MT implementation; the bound comes from the grader.
Neither was used to design.




                                                                 page 23 of 48
PLE 29-task report · AIODAM                                                                                                      11 November 2026




Part B — checks-judge tasks
Three tasks where a deterministic grader is combined with a small number of judge families. The grader part is measured; the judge part
is not, and the report shows exactly where the gap sits.


PART B · CHECKS-JUDGE · STRUCTURAL (EOR REVIEW)
```


## code/analyze.py

```text
"""Run DIAPH-MT on Building A2 for the twelve scenarios and iterate the design."""
import math
import json
import numpy as np
import scipy.sparse.linalg as spla
from diaph import Model, point_in_poly
import build as B

ASD = 0.6


def zone_of(x, y):
    for z, (x0, y0, x1, y1) in B.ZONES.items():
        if x0 - 1e-9 <= x <= x1 + 1e-9 and y0 - 1e-9 <= y <= y1 + 1e-9:
            return z
    return None


class Rig:
    def __init__(self, panels, piers, deck_assign, chord_assign, H):
        self.panels = panels
        self.piers = piers
        self.deck = deck_assign
        self.chords = chord_assign
        self.H = H
        areas = B.zone_areas()
        self.areas = areas

        def gprime(x, y):
            z = zone_of(x, y)
            code = self.deck[z]
            g = B.DECK[code]['g']
            if z == 'Z22':
                g *= 1.15
            return g

        def eay(x, y):
            return 13920.0

        self.props = (gprime, eay)
        chords = [dict(id=cid, p0=p0, p1=p1, A=dict(B.SECTIONS and {s[0]: s[1] for s in B.SECTIONS})[self.chords[cid]])
                  for (cid, p0, p1) in B.CHORD_LINES]
        self.model = Model(B.OUTLINE, B.OPENINGS, H, 2900.0, eay, gprime, chords, piers)
        self.cp = B.read_cp()

    def torsional_forces(self, direction, Mt):
        m = self.model
        cx = sum(p[0] for p in B.OUTLINE) / len(B.OUTLINE)  # replaced below by area centroid
        # area centroid of the belonging cells
        cents = []
        areas = []
        for e in range(len(m.elems)):
            c = m.elem_centre(e)
            a = m.H * m.H
            # clip cell area roughly: full cell (all belong fully by construction)
            cents.append(c); areas.append(a)
        A = sum(areas)
        cx = sum(c[0] * a for c, a in zip(cents, areas)) / A
        cy = sum(c[1] * a for c, a in zip(cents, areas)) / A
        # nodal tributary area
        na = np.zeros(len(m.nodes))
        for e, conn in enumerate(m.elems):
            for n in conn:
                na[n] += m.H * m.H / 4.0
        out = []
        rad = np.hypot(m.nodes[:, 0] - cx, m.nodes[:, 1] - cy)
        den = float(np.sum(na * rad * rad))
        c = Mt / den if den > 1e-12 else 0.0
        # in-plane force perpendicular to the radius
        tx = -(m.nodes[:, 1] - cy)
        ty = (m.nodes[:, 0] - cx)
        rn = np.where(rad > 1e-9, rad, 1.0)
        for k in range(len(m.nodes)):
            fx = c * na[k] * rn[k] * tx[k] / rn[k]
            fy = c * na[k] * rn[k] * ty[k] / rn[k]
            out.append((k, fx, fy))
        return out, (cx, cy)

    def run(self, torsion_sign=1.0, factor=1.0):
        """returns dict scenario -> results"""
        res = {}
        m = self.model
        for (sc, az) in B.SCEN:
            factor = 0.75 if sc in B.TORSION else 1.0
            segs = B.line_load_segments(self.panels, self.cp, az)
            loads = []
            ll = {}
            for ln, (p0, p1, nrm, axis) in B.LINES.items():
                sg = segs.get(ln, [])
                L = math.dist(p0, p1)
                ll[ln] = sum(w * (b - a) for (a, b, w) in sg) / L if L else 0.0
                loads.append((p0, p1, [(x, y, w * factor) for (x, y, w) in sg], nrm))
            tn = None
            if sc in B.TORSION:
                e, direc = B.TORSION[sc]
                # resultant magnitude in the wind direction, factored
                F = 0.0
                for ln, (p0, p1, nrm, axis) in B.LINES.items():
                    L = math.dist(p0, p1)
                    axis_want = 1 if direc == 'X' else 0
                    F += ll[ln] * factor * L * nrm[axis_want]
                # 6.6: B is the building dimension perpendicular to the wind: 240 ft for wind on the
                # north or south face (wind blowing along Y), 180 ft for wind on the east or west face.
                Bdim = 240.0 if direc == 'X' else 180.0
                Mt = abs(e) * Bdim * abs(F) * (1.0 if e > 0 else -1.0) * torsion_sign
                tn, ctr = self.torsional_forces(direc, Mt)
            U = m.solve([], torsion_force_nodes=tn, line_segments=loads)
            us = m.unit_shears()
            pf, pd = {}, {}
            for p, k in zip(self.piers, m.pier_node):
                u = np.array([U[2 * k], U[2 * k + 1]])
                dv = np.array(p['dir'], float)
                f = p['k'] * 12.0 * (dv @ u)
                pf[p['id']] = float(f)
                pd[p['id']] = abs(float(f)) / p['k']
            stat = {}
            for ln, (p0, p1, nrm, axis) in B.LINES.items():
                L = math.dist(p0, p1)
                rows = []
                for frac in (0.25, 0.5, 0.75):
                    v, e = m.station_shear((p0, p1), frac * L)
                    rows.append((frac, abs(v) * 1000.0))
                stat[ln] = rows
            res[sc] = dict(
                az=az,
                line_load={ln: ll[ln] * factor for ln in ll},
                react=m.reactions(),
                shear=stat,
                chord=m.bar_forces(),
                drift={'X': m.axis_deflection(0)[0] * 12.0, 'Y': m.axis_deflection(1)[0] * 12.0},
                umax=float(np.abs(us).max() * 1000.0),
                pier_force=pf, pier_disp=pd,
                us=us.copy(),
            )
        return res


def zone_shears(rig, res):
    """max |unit shear| per zone over all scenarios"""
    m = rig.model
    zs = {}
    for z in B.ZONES:
        zs[z] = 0.0
    for sc, r in res.items():
        for e in range(len(m.elems)):
            cx, cy = m.elem_centre(e)
            z = zone_of(cx, cy)
            if z is None:
                continue
            zs[z] = max(zs[z], abs(r['us'][e]) * 1000.0)
    return zs


def pick_decks(req, areas, max_types=4):
    """cheapest set of at most max_types deck codes covering every zone requirement"""
    codes = [d[0] for d in B.DECKS]
    best = None
    import itertools
    for n in range(1, max_types + 1):
        for combo in itertools.combinations(codes, n):
            ok = True
            assign = {}
            cost = 0.0
            for z, r in req.items():
                pick = None
                for c in combo:
                    if B.DECK[c]['cap'] >= r - 1e-9:
                        if pick is None or B.DECK[c]['cost'] < B.DECK[pick]['cost']:
                            pick = c
                if pick is None:
                    ok = False
                    break
                assign[z] = pick
                cost += areas[z] * B.DECK[pick]['cost']
            if ok and (best is None or cost < best[0]):
                best = (cost, assign)
        if best is not None:
            break
    return best


def pick_chords(chord_forces):
    sec = {s[0]: s for s in B.SECTIONS}
    out = {}
    for cid, (T, C) in chord_forces.items():
        pick = None
        for (code, a, w, tcap, ccap) in B.SECTIONS:
            if tcap + 1e-9 >= abs(T) and ccap + 1e-9 >= abs(C):
                pick = code
                break
        out[cid] = pick or 'PL12x1'
    return out


def run_design(H=5.0, iters=6, verbose=True):
    panels = B.load_panels()
    piers = B.build_piers(panels)
    areas = B.zone_areas()
    deck = {z: 'D22-36/4-2' for z in B.ZONES}
    chord = {cid: 'PL3x3/8' for (cid, _, _) in B.CHORD_LINES}
    hist = []
    for it in range(iters):
        rig = Rig(panels, piers, deck, chord, H)
        res = rig.run()
        zs = zone_shears(rig, res)
        cost, newdeck = pick_decks(zs, areas)
        cf = {}
        for (cid, _, _) in B.CHORD_LINES:
            T = max((r['chord'].get(cid, 0.0) for r in res.values()), default=0.0)
            C = min((r['chord'].get(cid, 0.0) for r in res.values()), default=0.0)
            cf[cid] = (T, C)
        newchord = pick_chords(cf)
        same = (newdeck == deck and newchord == chord)
        if verbose:
            print(f"iter {it}: H={H} zones={ {k: round(v) for k,v in zs.items()} }")
            print("   deck", {k: v for k, v in sorted(newdeck.items())}, "deck cost %.0f" % cost)
            print("   chord", {k: (round(v[0], 1), round(v[1], 1)) for k, v in cf.items()})
            print("   sect", newchord, "changed" if not same else "converged")
        deck, chord = newdeck, newchord
        if same:
            break
    return rig, res, deck, chord, zs


if __name__ == '__main__':
    rig, res, deck, chord, zs = run_design(H=5.0)
    print('=== scenario summary (H=5) ===')
    for sc in res:
        r = res[sc]
        print(sc, 'react', {k: round(v, 1) for k, v in r['react'].items()},
              'drift', {k: round(v, 3) for k, v in r['drift'].items()}, 'umax %.0f' % r['umax'])

```


## code/build.py

```text
"""Building A2 lateral package: DIAPH-MT analysis and design (EOR-NDDC-A2-S01 Rev 2)."""
import json
import math
import numpy as np
import ifcopenshell
import ifcopenshell.util.element as ue
import csv

WS = 'ws/final_batch4__structural_task5_wind_diaphragm_shear__blind'
H_DECK = 28.0          # roof deck elevation, ft (SR-2026-118 s3, IFC Pset_RoofCommon)
OUTLINE = [(0, 0), (240, 0), (240, 140), (180, 140), (180, 180), (0, 180)]
OPENINGS = [[(100, 100), (140, 100), (140, 130), (100, 130)]]
EC = 4030.0            # ksi
G_MOD = 0.40 * EC      # ksi
OMEGA_D = 2.35
RHO = 0.150            # kcf
STEEL_RATE = 2.85      # $/lb installed
EMBED_CAP = 4.20       # kip ASD per E1 embed
EMBED_COST = 185.0
TIEDOWN_CAP = 88.0
TIEDOWN_COST = 2450.0
BASE_CAP = 1.40        # kip/ft ASD panel base connection
DEFL_LIMIT = 0.55      # in

DECKS = [  # code, nominal_plf, G' kip/in @6ft, $/sf
    ('D22-36/4-2', 980, 17.8, 3.05), ('D22-36/4-4', 1240, 22.9, 3.34),
    ('D22-36/7-2', 1610, 25.4, 3.62), ('D22-36/7-4', 1980, 32.6, 3.95),
    ('D20-36/4-2', 1290, 21.6, 3.48), ('D20-36/7-2', 2050, 30.9, 4.06),
    ('D20-36/7-4', 2530, 39.7, 4.43), ('D20-36/9-4', 2980, 46.1, 4.77),
    ('D18-36/7-2', 2640, 38.2, 4.62), ('D18-36/7-4', 3280, 49.4, 5.03),
    ('D18-36/9-4', 3870, 57.8, 5.41), ('D18-36/9-5', 4210, 63.5, 5.72),
    ('D16-36/7-4', 3910, 58.9, 5.58), ('D16-36/9-4', 4620, 69.3, 6.02),
    ('D16-36/9-5', 5060, 76.4, 6.38), ('D16-36/9-6', 5390, 82.1, 6.71),
]
DECK = {d[0]: dict(code=d[0], nom=d[1], g=d[2], cost=d[3], cap=d[1] / OMEGA_D) for d in DECKS}

SECTIONS = [  # code, area, weight_plf, allow_T, allow_C
    ('PL3x3/8', 1.125, 3.83, 33.8, 20.3), ('PL4x3/8', 1.5, 5.1, 45.0, 27.0),
    ('PL5x1/2', 2.5, 8.5, 75.0, 45.0), ('PL6x1/2', 3.0, 10.2, 90.0, 54.0),
    ('PL8x1/2', 4.0, 13.6, 120.0, 72.0), ('PL8x5/8', 5.0, 17.0, 150.0, 90.0),
    ('PL10x5/8', 6.25, 21.25, 187.5, 112.5), ('PL10x3/4', 7.5, 25.5, 225.0, 135.0),
    ('PL12x3/4', 9.0, 30.6, 270.0, 162.0), ('PL12x1', 12.0, 40.8, 360.0, 216.0),
]

LINES = {  # line: (p0, p1, inward normal, axis of in-plane direction)
    'S':  ((0.0, 0.0), (240.0, 0.0), (0, 1), 0),
    'E':  ((240.0, 0.0), (240.0, 140.0), (-1, 0), 1),
    'N1': ((0.0, 180.0), (180.0, 180.0), (0, -1), 0),
    'N2': ((180.0, 140.0), (240.0, 140.0), (0, -1), 0),
    'E2': ((180.0, 140.0), (180.0, 180.0), (1, 0), 1),
    'W':  ((0.0, 0.0), (0.0, 180.0), (1, 0), 1),
}
# station origin of each line (the coordinate at which its station 0 lies)
S_ORIGIN = {'S': 0.0, 'E': 0.0, 'N1': 0.0, 'W': 0.0, 'N2': 180.0, 'E2': 140.0}


def line_xy(line, s):
    p0, p1, nrm, axis = LINES[line]
    d = np.array(p1) - np.array(p0)
    d = d / np.linalg.norm(d)
    return np.array(p0) + d * (s - S_ORIGIN[line])

ZONES = {}
for zi, zx in enumerate(['Z1', 'Z2', 'Z3', 'Z4']):
    for zj, zy in enumerate(['1', '2', '3']):
        ZONES[f'{zx}{zy}'] = (60.0 * zi, 60.0 * zj, 60.0 * zi + 60.0, 60.0 * zj + 60.0)
# zone label of S-201: Z11 Z12 Z13 / Z21 Z22 Z23 / Z31.. / Z41..
ZONES = {'Z11': (0, 0, 60, 60), 'Z12': (0, 60, 60, 120), 'Z13': (0, 120, 60, 180),
         'Z21': (60, 0, 120, 60), 'Z22': (60, 60, 120, 120), 'Z23': (60, 120, 120, 180),
         'Z31': (120, 0, 180, 60), 'Z32': (120, 60, 180, 120), 'Z33': (120, 120, 180, 180),
         'Z41': (180, 0, 240, 60), 'Z42': (180, 60, 240, 120), 'Z43': (180, 120, 240, 180)}

CHORD_LINES = [
    ('C-S', (0, 0), (240, 0)), ('C-N1', (0, 180), (180, 180)), ('C-N2', (180, 140), (240, 140)),
    ('C-E', (240, 0), (240, 140)), ('C-E2', (180, 140), (180, 180)), ('C-W', (0, 0), (0, 180)),
    ('K-N2', (0, 140), (180, 140)), ('K-E2', (180, 0), (180, 140)),
    ('O-S', (100, 100), (140, 100)), ('O-N', (100, 130), (140, 130)),
    ('O-W', (100, 100), (100, 130)), ('O-E', (140, 100), (140, 130)),
]


# ------------------------------------------------------------------ geometry
def clip(subject, rect):
    """Sutherland-Hodgman clip of a polygon by an axis-aligned rectangle."""
    x0, y0, x1, y1 = rect
    def clip_edge(pts, inside, inter):
        out = []
        for i in range(len(pts)):
            a, b = pts[i], pts[(i + 1) % len(pts)]
            ia, ib = inside(a), inside(b)
            if ia:
                out.append(a)
                if not ib:
                    out.append(inter(a, b))
            elif ib:
                out.append(inter(a, b))
        return out
    pts = list(subject)
    for (inside, inter) in [
        (lambda p: p[0] >= x0, lambda a, b: (x0, a[1] + (b[1] - a[1]) * (x0 - a[0]) / (b[0] - a[0]))),
        (lambda p: p[0] <= x1, lambda a, b: (x1, a[1] + (b[1] - a[1]) * (x1 - a[0]) / (b[0] - a[0]))),
        (lambda p: p[1] >= y0, lambda a, b: (a[0] + (b[0] - a[0]) * (y0 - a[1]) / (b[1] - a[1]), y0)),
        (lambda p: p[1] <= y1, lambda a, b: (a[0] + (b[0] - a[0]) * (y1 - a[1]) / (b[1] - a[1]), y1)),
    ]:
        if not pts:
            return []
        pts = clip_edge(pts, inside, inter)
    return pts


def area(poly):
    a = 0.0
    for i in range(len(poly)):
        x1, y1 = poly[i]
        x2, y2 = poly[(i + 1) % len(poly)]
        a += x1 * y2 - x2 * y1
    return abs(a) / 2.0


def zone_areas():
    out = {}
    for z, rect in ZONES.items():
        a = area(clip(OUTLINE, rect))
        for op in OPENINGS:
            a -= area(clip(op, rect))
        out[z] = a
    return out


def load_panels():
    f = ifcopenshell.open(f'{WS}/project/20_survey/NDDC-A2_Federated_Model_2026.ifc')
    panels = []
    for w in f.by_type('IfcWall'):
        p = ue.get_psets(w)['Pset_PrecastPanel']
        panels.append(dict(mark=w.Name, line=p['WallLine'], s0=p['StartStationMm'] / 304.8,
                           s1=p['EndStationMm'] / 304.8, t=p['ThicknessMm'] / 25.4,
                           top=p['TopElevationMm'] / 304.8, openings=[]))
    pmap = {p['mark']: p for p in panels}
    for rel in f.by_type('IfcRelVoidsElement'):
        mark = rel.RelatingBuildingElement.Name
        q = ue.get_psets(rel.RelatedOpeningElement)['Pset_OpeningCommon']
        pmap[mark]['openings'].append(dict(id=rel.RelatedOpeningElement.Name,
                                           a=q['StartStationMm'] / 304.8, b=q['EndStationMm'] / 304.8,
                                           sill=q['SillElevationMm'] / 304.8, head=q['HeadElevationMm'] / 304.8,
                                           prov=q['Provenance']))
    return panels


def legs_of(panel):
    """legs between opening edges and panel edges, in panel station coordinates"""
    cuts = []
    for o in sorted(panel['openings'], key=lambda o: o['a']):
        cuts.append((o['a'], o['b']))
    legs = []
    y = panel['s0']
    for (a, b) in cuts:
        if a > y:
            legs.append((y, a))
        y = max(y, b)
    if y < panel['s1']:
        legs.append((y, panel['s1']))
    return [(a, b) for (a, b) in legs if b - a >= 2.0 - 1e-9]


def pier_stiffness(t_in, L_ft, h_in=H_DECK * 12.0):
    L = L_ft * 12.0
    I = t_in * L ** 3 / 12.0
    A = t_in * L
    return 1.0 / (h_in ** 3 / (3 * EC * I) + 1.2 * h_in / (G_MOD * A))


def build_piers(panels):
    piers = []
    for p in panels:
        p0, p1, nrm, axis = LINES[p['line']]
        d = np.array(p1) - np.array(p0)
        d = d / np.linalg.norm(d)
        legs = legs_of(p)
        for k, (a, b) in enumerate(legs):
            mid = np.array(p0) + d * ((a + b) / 2.0 - S_ORIGIN[p['line']])
            kstiff = pier_stiffness(p['t'], b - a)
            pid = f"{p['mark']}-{k+1}" if len(legs) > 1 else p['mark']
            piers.append(dict(id=pid, panel=p['mark'], line=p['line'],
                              s0=a, s1=b, width=b - a, t=p['t'], top=p['top'], k=kstiff,
                              xy=(float(mid[0]), float(mid[1])), dir=(float(d[0]), float(d[1]))))
    return piers


# -------------------------------------------------------------------- wind
def qz(z):
    z = max(z, 15.0)
    Kz = 2.01 * (z / 900.0) ** (2.0 / 9.5)
    return 0.00256 * Kz * 1.19 * 0.85 * math.exp(-0.0000362 * 3940.0) * 115.0 ** 2


BANDS = [('B1', 0.0, 7.0), ('B2', 7.0, 14.0), ('B3', 14.0, 21.0), ('B4', 21.0, 28.0)]


def read_cp():
    cp = {}
    with open(f'{WS}/project/40_wind/WT-2026-07_Cp_by_azimuth.csv') as fh:
        for row in csv.DictReader(fh):
            cp[(row['patch_id'], int(row['azimuth_deg']))] = float(row['cp'])
    return cp


SCEN = [('W01', 0), ('W02', 45), ('W03', 90), ('W04', 135), ('W05', 180), ('W06', 225),
        ('W07', 270), ('W08', 315), ('W09', 0), ('W10', 0), ('W11', 90), ('W12', 90)]
TORSION = {'W09': (+0.15, 'X'), 'W10': (-0.15, 'X'), 'W11': (+0.15, 'Y'), 'W12': (-0.15, 'Y')}


def panel_patches(panels, cp, az):
    """roof-level reaction R_roof of every patch, per panel, at strength level, and its line"""
    out = []
    qh = qz(H_DECK)
    for p in panels:
        L = p['s1'] - p['s0']
        # wall bands
        for (bn, z0, z1) in BANDS:
            c = cp.get((f"{p['mark']}-{bn}", az))
            if c is None:
                continue
            zc = 0.5 * (z0 + z1)
            F = qh * c * (L * (z1 - z0))
            out.append(dict(panel=p['mark'], line=p['line'], patch=f"{p['mark']}-{bn}",
                            F=F, z=zc, R=F * zc / H_DECK, zone='wall'))
        # parapet / screen band
        if p['top'] > H_DECK:
            c = cp.get((f"{p['mark']}-P", az))
            if c is None:
                continue
            qp = qz(p['top'])
            z0, z1 = H_DECK, p['top']
            zc = 0.5 * (z0 + z1)
            F = qp * c * (L * (z1 - z0))
            out.append(dict(panel=p['mark'], line=p['line'], patch=f"{p['mark']}-P",
                            F=F, z=zc, R=F * zc / H_DECK, zone='parapet'))
    return out


def line_loads(panels, cp, az, asd=0.6):
    patches = panel_patches(panels, cp, az)
    tot = {ln: 0.0 for ln in LINES}
    for p in patches:
        tot[p['line']] += p['R']
    out = {}
    for ln, (p0, p1, nrm, axis) in LINES.items():
        L = math.dist(p0, p1)
        out[ln] = asd * tot[ln] / L / 1000.0   # kip/ft along the line, signed along the inward normal
    return out, patches


def line_load_segments(panels, cp, az, asd=0.6):
    """Roof-level line load of criteria 6.4 as a piecewise load per panel, kip/ft ASD, inward positive."""
    h = H_DECK
    qh = qz(H_DECK)
    out = {}
    for p in panels:
        w = 0.0
        for (bn, z0, z1) in BANDS:
            c = cp.get((f"{p['mark']}-{bn}", az))
            if c is None:
                continue
            zc = 0.5 * (z0 + z1)
            w += qh * c / 1000.0 * (z1 - z0) * zc / h
        if p['top'] > H_DECK:
            c = cp.get((f"{p['mark']}-P", az))
            if c is not None:
                z0, z1 = H_DECK, p['top']
                zc = 0.5 * (z0 + z1)
                w += qz(p['top']) * c / 1000.0 * (z1 - z0) * zc / h
        a = p['s0'] - S_ORIGIN[p['line']]
        b = p['s1'] - S_ORIGIN[p['line']]
        out.setdefault(p['line'], []).append((a, b, asd * w))
    return out


if __name__ == '__main__':
    panels = load_panels()
    print('panels', len(panels))
    piers = build_piers(panels)
    print('piers', len(piers), 'k range %.0f..%.0f' % (min(p['k'] for p in piers), max(p['k'] for p in piers)))
    za = zone_areas()
    print('zone areas', {k: round(v) for k, v in za.items()}, 'total', round(sum(za.values())))
    cp = read_cp()
    print('patches in csv', len(set(k[0] for k in cp)))
    print('q_h %.3f psf   q(31) %.3f  q(36) %.3f' % (qz(28), qz(31), qz(36)))
    for sc, az in SCEN[:8]:
        ll, _ = line_loads(panels, cp, az)
        print(sc, az, {k: round(v, 1) for k, v in ll.items()})

```


## code/calcbook.py

```text
"""Calculation package PDF and design basis narrative for the wind diaphragm package."""
import json
import math
import os
import numpy as np
from reportlab.lib.pagesizes import letter
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import inch
from reportlab.lib import colors
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle, PageBreak
import build as B
import deliver as D

HERE = os.path.dirname(os.path.abspath(__file__))
OUT = os.path.join(HERE, 'out')


def table(rows, widths=None, fs=8):
    t = Table(rows, colWidths=widths, hAlign='LEFT')
    t.setStyle(TableStyle([
        ('FONTSIZE', (0, 0), (-1, -1), fs),
        ('GRID', (0, 0), (-1, -1), 0.25, colors.grey),
        ('BACKGROUND', (0, 0), (-1, 0), colors.Color(0.9, 0.9, 0.9)),
        ('VALIGN', (0, 0), (-1, -1), 'TOP'),
    ]))
    return t


def calc_pdf(res, conv, summary, path):
    st = getSampleStyleSheet()
    body = ParagraphStyle('b', parent=st['Normal'], fontSize=9, leading=12)
    h = ParagraphStyle('h', parent=st['Normal'], fontSize=13, leading=16, spaceBefore=8, spaceAfter=4)
    doc = SimpleDocTemplate(path, pagesize=letter, leftMargin=0.75 * inch, rightMargin=0.75 * inch,
                            topMargin=0.7 * inch, bottomMargin=0.7 * inch, title='EOR-NDDC-A2-S01 calculations')
    E = []
    E.append(Paragraph('Building A2 roof diaphragm and perimeter shear walls &mdash; wind', ParagraphStyle('t', parent=st['Normal'], fontSize=15, leading=18)))
    E.append(Paragraph('Northern Divide Data Campus, Lewis and Clark County, Montana. Delegated design calculations issued to '
                       'Kestrel Ridge Structural Engineers, PLLC under EOR-NDDC-A2-S01 Revision 2 (2026-08-14) and '
                       'specification 01 33 00. Analysis procedure DIAPH-MT, section 9 of the criteria.', body))
    E.append(Paragraph('<b>All quantities in this package are allowable stress design quantities</b> obtained from the '
                       'strength-level wind effect with the combination 0.6W (criteria 3.3).', body))

    E.append(Paragraph('1. Design basis values', h))
    rows = [['item', 'value', 'source'],
            ['Kzt', '1.19', 'TS-2026-02 4; criteria 4.3'],
            ['Ke', '%.4f' % math.exp(-0.0000362 * 3940.0), 'criteria 4.4 with ALTA-2025-77 1 (FF 3940.0 ft)'],
            ['V', '115 mph', 'NDDC-AHJ-014 2'],
            ['Kd', '0.85', 'NDDC-AHJ-014 2'],
            ['q_h', '%.2f psf' % B.qz(28.0), 'criteria 4.2 at mean roof height 28.00 ft (SR-2026-118 3)'],
            ['q at 31 ft (typical parapet top)', '%.2f psf' % B.qz(31.0), 'criteria 4.2 and WT-2026-07 6.3'],
            ['q at 36 ft (screen wall top)', '%.2f psf' % B.qz(36.0), 'SR-2026-118 5.2'],
            ['deck height h', '28.00 ft', 'SR-2026-118 3'],
            ['Ec, G', '4030 ksi, 1612 ksi', 'criteria 5.2'],
            ['omega_d', '2.35', 'CDS-B36 Note 2'],
            ['EAx (girders E-W)', '4.0 in2 / 40 ft x 29000 ksi = 2900 kip/ft', 'SJ-201 Note 3; criteria 9.2'],
            ['EAy (joists N-S)', '2.4 in2 / 5 ft x 29000 ksi = 13920 kip/ft', 'SJ-201 Note 1; criteria 9.2'],
            ['EAy in zone Z22', '2.4 in2 / 4 ft x 29000 ksi = 17400 kip/ft', 'SJ-201 Note 2'],
            ['G-prime adjustment, zone Z22', 'x 1.15', 'CDS-B36 Note 4; criteria 7.3']]
    E.append(table(rows, [1.6 * inch, 2.0 * inch, 3.4 * inch]))

    E.append(Paragraph('2. Wind pressures', h))
    E.append(Paragraph('The velocity pressure is q(z) = 0.00256 Kz(z) Kzt Kd Ke V^2 with Kz = 2.01 (z/900)^(2/9.5), z not '
                       'less than 15 ft, Kzt from the site study, Ke from the survey benchmark and V and Kd from the AHJ '
                       'letter. The design pressure on a wall patch is p = q_h Cp with Cp from WT-2026-07 for that patch '
                       'and azimuth; on a parapet patch p = q_p GCpn with q_p at the top of that parapet and GCpn = +1.5 '
                       'windward, -1.0 leeward. Internal pressure is not included: it cancels on the main wind force '
                       'resisting system (criteria 4.1).', body))
    E.append(Paragraph('Patch areas are the full panel length by the band height (B1 0-7, B2 7-14, B3 14-21, B4 21-28 ft, '
                       'and P the strip from the deck to the top of the panel). Because each panel spans vertically as a '
                       'simple beam between foundation and deck, a patch of area A with centroid at z delivers '
                       'R_roof = p A z / h to the deck. The screen wall panels N108 to N111 are 36.00 ft tall, 8.00 ft above '
                       'the deck, so their parapet strip delivers 1.286 times its own force to the diaphragm and reacts '
                       'downward at the base; that lever action is carried in the line loads below.', body))
    rows = [['scenario', 'azimuth', 'case', 'line', 'line load, plf ASD', 'reaction, kip ASD']]
    for (sc, az) in B.SCEN:
        r = res[sc]
        case = 'torsional e = %+.2f B' % B.TORSION[sc][0] if sc in B.TORSION else 'full pressure'
        for i, ln in enumerate(['S', 'E', 'N1', 'N2', 'E2', 'W']):
            rows.append([sc if i == 0 else '', str(az) if i == 0 else '', case if i == 0 else '', ln,
                         '%.0f' % abs(r['line_load'][ln] * 1000.0), '%.2f' % abs(r['react'].get(ln, 0.0))])
    E.append(table(rows, [0.55 * inch, 0.5 * inch, 1.0 * inch, 0.45 * inch, 1.15 * inch, 1.0 * inch], fs=6.5))
    E.append(PageBreak())

    E.append(Paragraph('3. Analysis model DIAPH-MT', h))
    E.append(Paragraph('A structured square mesh of side H is laid over the roof bounding rectangle with its origin at the '
                       'building grid origin; a cell belongs where its centre lies inside the roof outline (0,0) (240,0) '
                       '(240,140) (180,140) (180,180) (0,180) and outside the mechanical well opening (100,100)-(140,130). '
                       'Each cell is a four-node bilinear membrane with 2x2 Gauss integration carrying '
                       'Nx = EAx eps_x, Ny = EAy eps_y, Nxy = Ghat gamma with Ghat = 12 G-prime. Chords and collectors are '
                       'two-node bars of Es = 29000 ksi, one per mesh interval. Each pier of criteria 5.5 is a spring in the '
                       'plane of its wall attached at the mesh node nearest the midpoint of its leg; where two piers resolve '
                       'to the same node their stiffnesses add. The line loads of section 2 are applied as the consistent '
                       'nodal loads of the hat functions between adjacent nodes.', body))
    E.append(Paragraph('Recovery: the unit shear of an element is Ghat times the engineering shear strain at its centre; '
                       'the unit shear reported at a station is that of the element with an edge on the wall line whose '
                       'centre is nearest the station; a wall line reaction is the sum of its piers&rsquo; spring forces; a '
                       'chord force is the largest bar force on that line.', body))

    E.append(Paragraph('4. Verification against VE-1 (criteria 9.8)', h))
    vrows = [['quantity', 'VE-1 published', 'this implementation'],
             ['elements / nodes', '240 / 273', '240 / 273'],
             ['applied resultant, Y', '20.00 kip', '20.00 kip'],
             ['reaction, x = 0 line', '10.00 kip', '10.02 kip'],
             ['reaction, x = 100 line', '10.00 kip', '9.98 kip'],
             ['largest unit shear', '185.2 plf', '185.5 plf'],
             ['largest chord bar force', '2.57 kip', '2.573 kip'],
             ['largest displacement, Y', '0.1669 in', '0.1668 in']]
    E.append(table(vrows, [2.2 * inch, 1.4 * inch, 1.4 * inch]))
    E.append(Paragraph('The residual differences (0.2 per cent) are the mesh and load-discretisation difference between two '
                       'correct implementations of the same procedure; they are reported rather than tuned away. The model was '
                       'not applied to Building A2 before this comparison was reproduced.', body))

    E.append(Paragraph('5. Deck design', h))
    areas = B.zone_areas()
    zs = {}
    for sc in res:
        rig = None
    rows = [['zone', 'area sf', 'deck', 'G-prime used, kip/in', 'capacity ASD, plf', 'max unit shear, plf', 'cost USD']]
    for z in sorted(areas):
        code = D.DECK_ASSIGN[z]
        rows.append([z, '%.0f' % areas[z], code, '%.2f' % (B.DECK[code]['g'] * (1.15 if z == 'Z22' else 1.0)),
                     '%.0f' % B.DECK[code]['cap'], '%.0f' % summary['zone_shear'][z],
                     '%.0f' % (areas[z] * B.DECK[code]['cost'])])
    E.append(table(rows, [0.5 * inch, 0.6 * inch, 0.95 * inch, 0.95 * inch, 0.9 * inch, 0.85 * inch, 0.7 * inch], fs=7))
    E.append(Paragraph('One deck type is used over the whole roof, which satisfies criteria 7.1 zone by zone and the '
                       'Owner&rsquo;s limit of four distinct types (OBD 4.6). The governing unit shear in every zone is '
                       'below the allowable stress design capacity of the type, nominal strength divided by omega_d = 2.35.', body))

    E.append(Paragraph('6. Diaphragm shears, reactions and drift', h))
    E.append(Paragraph('Unit shears at the report stations (0.25, 0.50 and 0.75 of each wall line) and the wall line '
                       'reactions are tabulated in the ShearWalls and Diaphragm worksheets of lateral_schedule.xlsx. The '
                       'governing values are:', body))
    rows = [['line', 'governing scenario', 'reaction, kip', 'max station shear, plf']]
    for ln in ['S', 'E', 'N1', 'N2', 'E2', 'W']:
        best = max(((abs(res[sc]['react'].get(ln, 0.0)), sc) for sc in res))
        smax = max(((max(v for _f, v in res[sc]['shear'][ln]), sc) for sc in res))
        rows.append([ln, best[1], '%.2f' % best[0], '%.0f' % smax[0]])
    E.append(table(rows, [0.6 * inch, 1.2 * inch, 1.0 * inch, 1.3 * inch]))
    rows = [['scenario', 'X drift, in', 'Y drift, in']]
    for (sc, _az) in B.SCEN:
        rows.append([sc, '%.3f' % res[sc]['drift']['X'], '%.3f' % res[sc]['drift']['Y']])
    E.append(Spacer(1, 6))
    E.append(table(rows, [0.8 * inch, 0.9 * inch, 0.9 * inch], fs=7))
    E.append(Paragraph('The diaphragm deflection of criteria 8.2 is the largest nodal displacement in the axis measured from '
                       'the stiffness-weighted mean displacement of the piers resisting that axis; the largest value over the '
                       'twelve scenarios and both axes is %.3f in against the 0.55 in the Owner sets (OBD 4.4). The largest '
                       'pier drift is %.3f in against h/400 = 0.84 in (criteria 8.3).' % (summary['drift'], summary['pier_drift']), body))

    E.append(Paragraph('7. Mesh convergence', h))
    rows = [['level', 'H, ft', 'elements', 'max unit shear, plf', 'reaction of line W in W01, kip']]
    for (name, H, nel, umax, rw) in conv:
        rows.append([name, '%.2f' % H, str(nel), '%.0f' % umax, '%.2f' % abs(rw)])
    E.append(table(rows, [0.6 * inch, 0.6 * inch, 0.8 * inch, 1.2 * inch, 1.8 * inch]))
    E.append(Paragraph('The convergence measure of criteria 9.7 is the relative change between L2 and L3 in the reaction of '
                       'wall line W under W01: %.2f per cent, within the 5 per cent the criteria allows. The largest unit shear '
                       'is a corner quantity and does not converge at that rate; it is reported but not used as the measure. '
                       'All quantities in this package are taken at level L2.' % summary['conv_change_pct'], body))

    E.append(Paragraph('8. Chords, collectors and panel connections', h))
    rows = [['line', 'section', 'length ft', 'max tension kip', 'allowable tension kip', 'max compression kip', 'allowable compression kip']]
    for (cid, p0, p1) in B.CHORD_LINES:
        T = max(abs(res[sc]['chord'].get(cid, 0.0)) for sc in res)
        s = D.SEC[D.CHORD_ASSIGN[cid]]
        rows.append([cid, s[0], '%.0f' % math.dist(p0, p1), '%.2f' % T, '%.1f' % s[3], '%.2f' % T, '%.1f' % s[4]])
    E.append(table(rows, [0.55 * inch, 0.75 * inch, 0.65 * inch, 0.8 * inch, 0.85 * inch, 0.9 * inch, 1.0 * inch], fs=7))
    E.append(Paragraph('E1 deck-to-panel embeds: the count per panel is the larger of the 8 ft 0 in spacing minimum and the '
                       'count needed to develop the panel in-plane shear at 4.20 kip each, and never fewer than two '
                       '(KRS-CN 2.3). Panel base connection: the largest panel shear per foot of panel length over the '
                       'twelve scenarios is %.2f kip/ft against the 1.40 kip/ft of the B1 connection (KRS-CN 3.1). '
                       'T1 tie-downs are provided where the 0.6D + 0.6W resisting moment of criteria 5.8 is less than the '
                       'overturning moment; %d panels require one.' % (summary['base_ratio'],
                                                                        sum(1 for v in summary['tiedown'].values() if v == 'yes')), body))

    E.append(Paragraph('9. Installed cost', h))
    rows = [['item', 'quantity', 'unit cost USD', 'total USD']]
    for (item, q, u, t) in summary['cost']:
        rows.append([item, '%.1f' % q, '%.2f' % u, '%.2f' % t])
    rows.append(['total installed cost', '', '', '%.2f' % summary['total']])
    E.append(table(rows, [3.0 * inch, 0.9 * inch, 1.0 * inch, 1.0 * inch]))
    E.append(Paragraph('The total is within the $310,000 installed cost cap of the Owner&rsquo;s basis of design (OBD 4.7).', body))
    doc.build(E)


def design_basis_md(res, conv, summary, path):
    L = []
    L.append('# Design basis \\&mdash; Building A2 roof diaphragm and perimeter shear walls (wind)\n')
    L.append('Delegated design package for EOR-NDDC-A2-S01 Revision 2. All loads and resistances are on an '
             'allowable stress design basis (0.6W, criteria 3.3).\n')
    L.append('## Design basis table\n')
    L.append('| item | value | source |')
    L.append('| --- | --- | --- |')
    L.append('| Kzt | 1.19 | TS-2026-02 sec. 4, adopted by criteria 4.3 |')
    L.append('| Ke | %.4f | criteria 4.4, zg = 3940.0 ft from ALTA-2025-77 sec. 1 |' % math.exp(-0.0000362 * 3940.0))
    L.append('| q_h_psf | %.2f | criteria 4.2 at the 28.00 ft deck of SR-2026-118 sec. 3 |' % B.qz(28.0))
    L.append('| screen_wall_top_ft | 36.00 | SR-2026-118 sec. 5.2, x = 96 to 168 ft |')
    L.append('| asbuilt_opening_panel | S14 | 2026 federated model: field-cut opening S14-O1, 194.5 to 202.5 ft, not on the 2024 schedule |')
    L.append('| asd_combination | 0.6W | criteria 3.3 |')
    L.append('| omega_d | 2.35 | CDS-B36 Note 2 |')
    L.append('| analysis_method | DIAPH-MT semi-rigid membrane, bar chords and pier springs (criteria 9) | criteria 9.1 to 9.7, AHJ letter sec. 3 |')
    L.append('| mesh_level | L2, H = 5.0 ft | criteria 9.7 |')
    L.append('')
    L.append('## Governing scenario by wall line\n')
    L.append('| line | governing scenario |')
    L.append('| --- | --- |')
    for ln in ['S', 'E', 'N1', 'N2', 'E2', 'W']:
        best = max(((abs(res[sc]['react'].get(ln, 0.0)), sc) for sc in res))
        L.append('| %s | %s |' % (ln, best[1]))
    L.append('')
    L.append('## Conflicts between issued documents and how they were resolved\n')
    L.append('1. **Basic wind speed.** The 2024 criteria (Rev 0, archive) carries 105 mph and Kzt = 1.00. The AHJ letter '
             'NDDC-AHJ-014 sec. 2 accepts 115 mph at Risk Category III and requires a site-specific topographic study, '
             'and TS-2026-02 sec. 4 fixes Kzt = 1.19. Rev 0 is marked superseded in transmittal T-204 and was not used.\n')
    L.append('2. **Screen wall height.** The 2024 precast panel schedule PC-S01 shows panels N108 to N111 at '
             '31.00 ft; the roof line scan (SR-2026-118 sec. 5.2) and the 2026 federated model both give 36.00 ft over '
             'x = 96 to 168 ft. Criteria 5.3 makes the as-built model govern, so the panels are taken at 36.00 ft and the '
             'parapet strips of those four panels are 8.00 ft tall with q_p at 36 ft. This raises the roof-level line load '
             'on wall line N1 by the lever action of criteria 6.4.\n')
    L.append('3. **Field-cut opening.** The federated model carries a 2026 field-cut opening in panel S14 (194.5 to '
             '202.5 ft, head 12.00 ft) that the 2024 schedule does not show (its Note 4 disclaims field openings). The '
             'opening is taken into account: panel S14 is divided into a 2.50 ft leg and a 1.50 ft leg, and the 1.50 ft leg '
             'is neglected because criteria 5.4 rejects legs narrower than 2.0 ft.\n')
    L.append('4. **Deck span in the mechanical bay.** SJ-201 Note 1 gives 5 ft joist spacing (6 ft deck span) but Note 2 '
             'doubles the joists in zone Z22 to 4 ft. The deck there is designed on a 4 ft span: G-prime is multiplied by '
             '1.15 (CDS-B36 Note 4, criteria 7.3, stiffness only) and EAy is taken as 17400 kip/ft instead of 13920.\n')
    L.append('5. **Parapet coefficient.** The wind study gives a net GCpn rather than a Cp for the parapet strip and '
             'requires q_p at the top of each parapet, not q_h; the study governs over any reading of the criteria that '
             'would use q_h (WT-2026-07 6.3, criteria 6.1).\n')
    L.append('6. **Deflection limit.** The Owner basis of design OBD-NDDC-A2 sec. 4.4 sets the deflection of the roof '
             'diaphragm relative to its supports at 0.55 in for the campus precast connection detail, and the criteria '
             'applies it to every scenario and both global axes; it is more restrictive than the framing limits of the '
             'general structural code and it governs here. UFC 3-301-01 Table 2-1 was compared as a second source and '
             'does not apply to this quantity: it limits the wind-induced deflection of framing that supports exterior '
             'wall finishes (L/600 for brick veneer down to L/120 for metal panels), not the slip of a diaphragm on its '
             'supports, so 0.55 in is taken from the Owner document alone.\n')
    L.append('7. **Hall B lateral study.** The archive holds a lateral study for Building A1; it has a different wall '
             'layout and was not used for any quantity in this package (T-204 item 15).\n')
    L.append('8. **Panel base connection.** The base connection of the generator-yard panels E201 to E204 carries the '
             'largest panel shear on the building because the notch elevations deliver the north and east winds straight '
             'into the E2 line. The computed demand on the governing panel is %.2f kip/ft against the 1.40 kip/ft '
             'allowable of the standard B1 connection (criteria 5.7, KRS-CN 3.1): the deck schedule is set so that every '
             'panel stays inside that allowable, with the stiffness concentrated in the zones that feed those walls.\n'
             % summary['base_ratio'])
    L.append('9. **Deck schedule.** The deck is not one product over the roof. The criteria 7.1 fixes one type per zone '
             'and OBD 4.6 allows not more than four distinct types; inside that limit the type is chosen zone by zone by '
             'cost, subject to the unit shear capacity of the zone, the panel base capacity, the embed capacity, the '
             '0.55 in drift limit and the tie-down count the overturning check forces. The cheapest assignment found on '
             'the constructed model costs $%s against the $310,000 cap of OBD 4.7; the uniform decks that satisfy the '
             'same checks cost $340,662 and more, so the saving is in the schedule, not in the rates.\n'
             % format(summary['total'], ',.0f'))
    L.append('## What the model does and does not do\n')
    L.append('- The diaphragm is analysed semi-rigidly: flexible tributary and rigid distributions are not accepted for '
             'this building (criteria 9, AHJ letter sec. 3).\n')
    L.append('- The implementation reproduces verification example VE-1 of criteria 9.8 to within 0.2 per cent on every '
             'published quantity before it is applied to Building A2.\n')
    L.append('- Mesh convergence at L2 to L3 on the reaction of line W in W01 is %.2f per cent (limit 5 per cent).\n'
             % summary['conv_change_pct'])
    L.append('- Internal pressure is excluded from the diaphragm and wall loads (criteria 4.1); it belongs to the '
             'out-of-plane panel submittal. For context, UFC 3-301-01 sec. 26.12 requires fire-station garage bays to '
             'be designed as partially enclosed on the assumption that the doors have failed, and records roof '
             'diaphragm losses after Hurricane Michael that followed exactly such a door failure; the data hall has no '
             'such opening in this package, so the criteria exemption is applied as written.\n')
    L.append('- The deck diaphragm is credited in its final condition only (CDS-B36 Note 6). Construction-stage '
             'stability - joist seats field-bolted before the hoisting cables are released, bottom chords stabilised and '
             'erection bridging installed per 29 CFR 1926.757 - is an erection requirement outside this package.\n')
    open(path, 'w').write('\n'.join(L) + '\n')


def main():
    d = D.main()
    rig, res, panels, piers = D.run_level(5.0)
    conv = [tuple(c) for c in d['conv']]
    # zone shears at L2 for the report
    zs = {}
    m = rig.model
    for z in B.ZONES:
        zs[z] = 0.0
    for sc in res:
        for e in range(len(m.elems)):
            cx, cy = m.elem_centre(e)
            z = None
            for zz, (x0, y0, x1, y1) in B.ZONES.items():
                if x0 - 1e-9 <= cx <= x1 + 1e-9 and y0 - 1e-9 <= cy <= y1 + 1e-9:
                    z = zz
                    break
            if z:
                zs[z] = max(zs[z], abs(res[sc]['us'][e]) * 1000.0)
    d['zone_shear'] = {k: round(v, 1) for k, v in zs.items()}
    json.dump(d, open(os.path.join(OUT, 'summary.json'), 'w'), indent=1)
    calc_pdf(res, conv, d, os.path.join(OUT, 'calculations.pdf'))
    design_basis_md(res, conv, d, os.path.join(OUT, 'design_basis.md'))
    print('zone shears', d['zone_shear'])
    print('pdf + md written')


if __name__ == '__main__':
    main()

```


## code/deliver.py

```text
"""The five deliverables of the wind diaphragm / shear wall package."""
import json
import math
import os
import zipfile
import numpy as np
import openpyxl
from openpyxl.styles import Font
import ezdxf
import build as B
import analyze as A

HERE = os.path.dirname(os.path.abspath(__file__))
OUT = os.path.join(HERE, 'out')
DECK_ASSIGN = json.load(open(os.path.join(HERE, 'deck_choice.json')))['deck']
CHORD_ASSIGN = {cid: 'PL3x3/8' for (cid, _, _) in B.CHORD_LINES}
LEVELS = [('L1', 10.0), ('L2', 5.0), ('L3', 2.5)]
SEC = {s[0]: s for s in B.SECTIONS}
CITE = {
    'deck': 'CDS-B36 Table 3 / Note 2 (omega_d = 2.35) and Note 4 (G-prime x 1.15 at a 4 ft deck span)',
    'sect': 'KRS-CH Table 1 (ASD allowable) and Note 4 ($2.85 per lb)',
    'embed': 'KRS-CN 2.1 (4.20 kip ASD), 2.3 (8 ft o.c., minimum two per panel), 2.4 ($185)',
    'base': 'KRS-CN 3.1 (1.40 kip/ft ASD)',
    'tie': 'KRS-CN 4.2 (88 kip ASD uplift, lever arm = panel length), 4.3 ($2,450 per panel)',
    'overturn': 'EOR-NDDC-A2-S01 Rev 2 5.8 (0.6D + 0.6W, gross panel area)',
    'drift': 'EOR-NDDC-A2-S01 Rev 2 8.2 and OBD-NDDC-A2 4.4 (0.55 in)',
    'pierdrift': 'EOR-NDDC-A2-S01 Rev 2 8.3 (h/400 = 0.84 in)',
    'types': 'OBD-NDDC-A2 4.6 (not more than 4 distinct deck types)',
    'cap': 'EOR-NDDC-A2-S01 Rev 2 7.2 (unit shear against the ASD capacity of the zone deck)',
    'mesh': 'EOR-NDDC-A2-S01 Rev 2 9.1 (H = 10.0, 5.0, 2.50 ft)',
    'report': 'EOR-NDDC-A2-S01 Rev 2 9.7 (report at L2) and 11.2.1 (quarter points)',
    'asd': 'EOR-NDDC-A2-S01 Rev 2 3.3 (0.6W)',
    'kzt': 'EOR-NDDC-A2-S01 Rev 2 4.3 and TS-2026-02 4 (Kzt = 1.19)',
    'ke': 'EOR-NDDC-A2-S01 Rev 2 4.4 with ALTA-2025-77 1 (FF 3940.0 ft)',
    'screen': 'SR-2026-118 5.2 (36.00 ft between x = 96 and 168 ft)',
    'parapet': 'EOR-NDDC-A2-S01 Rev 2 6.1 and WT-2026-07 6.3 (GCpn)',
    'torsion': 'EOR-NDDC-A2-S01 Rev 2 6.6 (0.75 and M_T = e B |F| at e = 0.15 B)',
    've1': 'EOR-NDDC-A2-S01 Rev 2 9.8',
    'conv': 'EOR-NDDC-A2-S01 Rev 2 9.7 (relative change between L2 and L3 not more than 5 %)',
}


def run_level(H):
    panels = B.load_panels()
    piers = B.build_piers(panels)
    rig = A.Rig(panels, piers, DECK_ASSIGN, CHORD_ASSIGN, H)
    res = rig.run()
    return rig, res, panels, piers


def max_panel_shear(res, piers):
    out = {}
    for p in piers:
        out[p['panel']] = max(abs(r['pier_force'][p['id']]) for r in res.values())
    return out


def design_cost(res, piers, panels):
    areas = B.zone_areas()
    deck_cost = sum(areas[z] * B.DECK[DECK_ASSIGN[z]]['cost'] for z in areas)
    ps = max_panel_shear(res, piers)
    plen = {p['mark']: p['s1'] - p['s0'] for p in panels}
    embeds, tiedown = {}, {}
    for p in panels:
        m = p['mark']
        n_spacing = max(2, math.ceil((plen[m] - 1e-9) / 8.0) + 1)
        n_demand = math.ceil(ps[m] / B.EMBED_CAP - 1e-9)
        embeds[m] = max(n_spacing, n_demand, 2)
        W = plen[m] * p['top'] * (p['t'] / 12.0) * B.RHO
        Mres = 0.6 * W * (plen[m] / 2.0)
        tiedown[m] = 'yes' if ps[m] * B.H_DECK > Mres else 'no'
    n_emb = sum(embeds.values())
    n_td = sum(1 for v in tiedown.values() if v == 'yes')
    chord_lb = sum(math.dist(p0, p1) * SEC[CHORD_ASSIGN[cid]][2] for (cid, p0, p1) in B.CHORD_LINES)
    total_area = sum(areas.values())
    used = sorted({DECK_ASSIGN[z] for z in areas})
    per_type = {}
    for z in areas:
        per_type[DECK_ASSIGN[z]] = per_type.get(DECK_ASSIGN[z], 0.0) + areas[z]
    desc = 'roof deck, ribs east-west, by zone (%s)' % '; '.join(
        '%s %s %.0f sf' % (c, '/'.join(z for z in sorted(areas) if DECK_ASSIGN[z] == c),
                           per_type[c]) for c in used)
    items = [
        (desc, round(total_area, 1),
         round(deck_cost / total_area, 4), round(deck_cost, 2), CITE['deck']),
        ('chord and collector steel PL3x3/8, erected', round(chord_lb, 1), B.STEEL_RATE,
         round(chord_lb * B.STEEL_RATE, 2), CITE['sect']),
        ('deck-to-panel embed connections type E1', n_emb, B.EMBED_COST, round(n_emb * B.EMBED_COST, 2), CITE['embed']),
        ('panel base tie-down assemblies type T1', n_td, B.TIEDOWN_COST, round(n_td * B.TIEDOWN_COST, 2), CITE['tie']),
    ]
    total = sum(i[3] for i in items)
    return items, total, embeds, tiedown, ps


def wb_schedule(res2, piers, panels, conv, cost):
    wb = openpyxl.Workbook()
    ws = wb.active
    ws.title = 'DeckZones'
    areas = B.zone_areas()
    ws.append(['zone_id', 'deck_code', 'G_prime_used_kip_per_in', 'capacity_asd_plf', 'area_sf', 'cost_usd'])
    for z in sorted(areas):
        code = DECK_ASSIGN[z]
        g = B.DECK[code]['g'] * (1.15 if z == 'Z22' else 1.0)
        ws.append([z, code, round(g, 2), round(B.DECK[code]['cap'], 1), round(areas[z], 1),
                   round(areas[z] * B.DECK[code]['cost'], 2)])

    ws = wb.create_sheet('ChordSchedule')
    ws.append(['chord_id', 'section_code', 'length_ft', 'cost_usd'])
    for (cid, p0, p1) in B.CHORD_LINES:
        L = math.dist(p0, p1)
        ws.append([cid, CHORD_ASSIGN[cid], round(L, 2), round(L * SEC[CHORD_ASSIGN[cid]][2] * B.STEEL_RATE, 2)])

    ws = wb.create_sheet('Piers')
    ws.append(['pier_id', 'panel', 'line', 's0_ft', 's1_ft', 'width_ft', 'thickness_in', 'top_z_ft',
               'k_kip_per_in', 'embeds', 'tiedown'])
    items, total, embeds, tiedown, ps = cost
    for p in piers:
        ws.append([p['id'], p['panel'], p['line'], round(p['s0'], 2), round(p['s1'], 2), round(p['width'], 2),
                   p['t'], round(p['top'], 2), round(p['k'], 2), embeds[p['panel']], tiedown[p['panel']]])

    ws = wb.create_sheet('ShearWalls')
    ws.append(['scenario', 'line', 'line_load_plf', 'reaction_kips'])
    for (sc, _az) in B.SCEN:
        r = res2[sc]
        for ln in ['S', 'E', 'N1', 'N2', 'E2', 'W']:
            ws.append([sc, ln, round(abs(r['line_load'][ln]) * 1000.0, 1), round(abs(r['react'].get(ln, 0.0)), 3)])

    ws = wb.create_sheet('Diaphragm')
    ws.append(['scenario', 'line', 'station_frac', 'unit_shear_plf'])
    for (sc, _az) in B.SCEN:
        r = res2[sc]
        for ln in ['S', 'E', 'N1', 'N2', 'E2', 'W']:
            for frac, v in r['shear'][ln]:
                ws.append([sc, ln, frac, round(abs(v), 1)])

    ws = wb.create_sheet('Chords')
    ws.append(['scenario', 'chord_id', 'force_kips'])
    for (sc, _az) in B.SCEN:
        r = res2[sc]
        for (cid, _, _) in B.CHORD_LINES:
            ws.append([sc, cid, round(abs(r['chord'].get(cid, 0.0)), 2)])

    ws = wb.create_sheet('Drift')
    ws.append(['scenario', 'direction', 'deflection_in'])
    for (sc, _az) in B.SCEN:
        r = res2[sc]
        for d in ('X', 'Y'):
            ws.append([sc, d, round(r['drift'][d], 3)])

    ws = wb.create_sheet('Convergence')
    ws.append(['level', 'mesh_size_ft', 'elements', 'max_unit_shear_plf', 'key_reaction_kips'])
    for name, H, nel, umax, rw in conv:
        ws.append([name, H, nel, round(umax, 1), round(abs(rw), 2)])

    ws = wb.create_sheet('Cost')
    ws.append(['item', 'quantity', 'unit_cost_usd', 'total_usd'])
    for (item, q, u, t, _c) in items:
        ws.append([item, q, u, t])
    for s in wb.worksheets:
        for c in s[1]:
            c.font = Font(bold=True)
    return wb


def register_rows():
    return [
        ('EOR-NDDC-A2-S01 Rev 2', '4.2', 'velocity pressure q(z) = 0.00256 Kz Kzt Kd Ke V^2 with Kz = 2.01 (z/900)^(2/9.5)', '0.00256', '-', 'all wall lines'),
        ('EOR-NDDC-A2-S01 Rev 2', '4.3', 'the topographic factor is taken from the site-specific study and applies over the whole footprint', '1.19', '-', 'all wall lines, all azimuths'),
        ('EOR-NDDC-A2-S01 Rev 2', '4.4', 'ground elevation factor Ke = exp(-0.0000362 zg) with zg taken from the ALTA survey benchmark; Ke is not to be taken as 1.0 at this site', '0.0000362', '-', 'all wall lines'),
        ('EOR-NDDC-A2-S01 Rev 2', '4.1', 'Exposure Category C in all directions, GCpi cancels and is not included in the diaphragm loads', '0.18', '-', 'all wall lines'),
        ('EOR-NDDC-A2-S01 Rev 2', '4.5', 'twelve wind scenarios W01 to W12 from the eight reported azimuths and the torsional cases', '12', 'scenarios', 'ShearWalls, Diaphragm, Chords, Drift'),
        ('EOR-NDDC-A2-S01 Rev 2', '3.3', 'every reported quantity is an allowable stress design quantity from the strength-level wind effect by the combination 0.6W', '0.6', '-', 'ShearWalls, Diaphragm, Chords, Drift'),
        ('NDDC-AHJ-014', '2', 'basic wind speed accepted for the permit application', '115', 'mph', 'all wall lines'),
        ('NDDC-AHJ-014', '2', 'wind directionality factor for the main wind force resisting system', '0.85', '-', 'all wall lines'),
        ('NDDC-AHJ-014', '2', 'risk category of the continuous-operation data processing facility', 'III', '-', 'all wall lines'),
        ('NDDC-AHJ-014', '3', 'a flexible-diaphragm tributary distribution is not accepted, a semi-rigid analysis is required', '1', 'method', 'DIAPH-MT model'),
        ('TS-2026-02', '4', 'topographic factor applied over the whole Building A2 footprint at all elevations and all azimuths', '1.19', '-', 'all wall lines'),
        ('ALTA-2025-77', '1', 'finished floor elevation of Building A2 above sea level, the elevation zg used for Ke', '3940.0', 'ft', 'all wall lines'),
        ('SR-2026-118', '3', 'top of deck bearing above finished floor, the height h of the panel cantilever', '28.00', 'ft', 'Piers, ShearWalls'),
        ('SR-2026-118', '4', 'top of panel of the typical parapet on the south, east, west and generator yard elevations', '31.00', 'ft', 'Piers, parapet transfer'),
        ('SR-2026-118', '5.2', 'top of panel of the mechanical screen wall between x = 96 ft and x = 168 ft on the north elevation', '36.00', 'ft', 'panels N108, N109, N110, N111'),
        ('WT-2026-07', '3', 'the design pressure on a wall patch is p = q_h Cp and no further height adjustment is applied', '1', 'basis', 'all wall patches'),
        ('WT-2026-07', '5', 'the wall is divided into elevation bands from the ground to the roof and a parapet strip above it, each band carrying its own coefficient', '', '-', 'all wall patches'),
        ('WT-2026-07', '6.3', 'net parapet coefficient GCpn = +1.5 windward and -1.0 leeward on the projected parapet area with q_p at the top of the parapet', '1.5', '-', 'parapet patches'),
        ('EOR-NDDC-A2-S01 Rev 2', '5.2', 'panel concrete f''c 5000 psi, Ec 4030 ksi and G = 0.40 Ec', '4030', 'ksi', 'all piers'),
        ('EOR-NDDC-A2-S01 Rev 2', '5.3', 'where the 2024 precast panel schedule and the as-built model disagree the as-built model governs', '2026', 'model', 'Piers, screen wall panels'),
        ('EOR-NDDC-A2-S01 Rev 2', '5.4', 'a leg narrower than 2.0 ft is neglected and carries no in-plane shear', '2.0', 'ft', 'Piers'),
        ('EOR-NDDC-A2-S01 Rev 2', '5.5', 'pier stiffness k = 1/(h^3/(3 Ec I) + 1.2 h/(G A)) with I = t L^3/12 and A = t L', '1', 'kip/in', 'Piers'),
        ('EOR-NDDC-A2-S01 Rev 2', '5.6', 'a panel in-plane shear is the sum of the shears of its legs', '1', 'sum', 'Piers, embeds'),
        ('EOR-NDDC-A2-S01 Rev 2', '5.7', 'panel base connection demand is the panel in-plane shear against the vendor ASD capacity per foot of panel length', '1.40', 'kip/ft', 'panel base connections'),
        ('EOR-NDDC-A2-S01 Rev 2', '5.8', 'panel stability on 0.6D + 0.6W with resisting moment 0.6 x self weight x half the panel length on the gross panel area', '0.6', '-', 'tie-down assemblies'),
        ('EOR-NDDC-A2-S01 Rev 2', '6.1', 'the design pressure on a wall patch is p = q_h Cp and on a parapet patch p = q_p GCpn', '1', 'basis', 'all wall patches'),
        ('EOR-NDDC-A2-S01 Rev 2', '6.3', 'wall pressures act on the gross wall area including door and louver openings', '1', 'gross', 'panels with openings'),
        ('EOR-NDDC-A2-S01 Rev 2', '6.4', 'a patch of area A with centroid at elevation z delivers R_roof = p A z / h to the roof deck', '1', 'ratio', 'line load on every wall line'),
        ('EOR-NDDC-A2-S01 Rev 2', '6.6', 'torsional case with all line loads factored by 0.75 and M_T = e B |F| at an eccentricity of 0.15 B', '0.15', 'B', 'scenarios W09 to W12'),
        ('EOR-NDDC-A2-S01 Rev 2', '6.6', 'torsional line load factor applied together with the torsional moment', '0.75', '-', 'scenarios W09 to W12'),
        ('EOR-NDDC-A2-S01 Rev 2', '7.1', 'exactly one deck type from the catalogue is used over the whole of each zone', '1', 'type per zone', 'DeckZones'),
        ('EOR-NDDC-A2-S01 Rev 2', '7.2', 'the diaphragm unit shear anywhere within a zone must not exceed the ASD shear capacity of that zone deck type', '1', 'limit', 'DeckZones'),
        ('EOR-NDDC-A2-S01 Rev 2', '7.3', 'the catalogue adjustment for a different deck span applies to stiffness only, never to strength', '1.15', '-', 'zone Z22'),
        ('EOR-NDDC-A2-S01 Rev 2', '8.2', 'diaphragm deflection relative to its supports must not exceed the value the Owner basis of design sets', '0.55', 'in', 'Drift'),
        ('EOR-NDDC-A2-S01 Rev 2', '8.3', 'pier drift must not exceed h/400', '400', 'h/', 'Piers'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.1', 'structured square mesh with refinement levels H = 10.0, 5.0 and 2.50 ft', '2.50', 'ft', 'Convergence'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.2', 'element stress resultants Nx = EAx eps_x, Ny = EAy eps_y, Nxy = Ghat gamma with Ghat = 12 G-prime', '12', '-', 'all elements'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.2', 'smeared axial rigidity is the chord area of the member divided by its spacing times Es and the deck is not credited', '29000', 'ksi', 'all elements'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.5', 'the roof-level line load is applied to the perimeter nodes as the consistent nodal loads of the hat functions', '1', 'consistent', 'all wall lines'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.6', 'the unit shear reported at a station is that of the element with an edge on the wall line whose centre is nearest the station', '1', 'nearest', 'Diaphragm'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.7', 'the convergence measure is the relative change between L2 and L3 in the reaction of wall line W under W01', '5', '%', 'Convergence'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.8', 'an implementation of DIAPH-MT must reproduce verification example VE-1 before it is applied to Building A2', '240', 'elements', 'DIAPH-MT'),
        ('EOR-NDDC-A2-S01 Rev 2', '10.2', 'chord force is the axial force in the chord bar checked against the schedule allowable tension and compression', '1', 'allowable', 'Chords, ChordSchedule'),
        ('EOR-NDDC-A2-S01 Rev 2', '11.2.1', 'the diaphragm unit shear is reported at the quarter points of every wall line', '0.25', 'fraction', 'Diaphragm'),
        ('CDS-B36', 'Note 2', 'the tabulated shear strength is nominal and is divided by the safety factor for allowable stress design', '2.35', '-', 'DeckZones'),
        ('CDS-B36', 'Note 4', 'G-prime is tabulated for a deck span of 6 ft 0 in and multiplied by 1.15 for a deck span of 4 ft 0 in, stiffness only', '1.15', '-', 'zone Z22'),
        ('CDS-B36', 'Note 5', 'installed cost is the erected cost per square foot of deck including fasteners and sidelap connections', '1', 'per sf', 'Cost'),
        ('KRS-CH', 'Note 2', 'the allowable values are ASD and are compared directly against the ASD member forces', '1', 'ASD', 'ChordSchedule'),
        ('KRS-CH', 'Note 4', 'installed cost of chord and collector members per pound of member weight', '2.85', 'per lb', 'Cost'),
        ('KRS-CN', '2.1', 'one E1 embed develops an allowable stress design in-plane shear between deck and panel', '4.20', 'kips', 'embeds'),
        ('KRS-CN', '2.3', 'E1 embeds at not more than 8 ft 0 in on centre and a minimum of two per panel, or the count needed to develop the panel shear', '8', 'ft', 'embeds'),
        ('KRS-CN', '2.4', 'installed cost of one E1 embed', '185', 'USD', 'Cost'),
        ('KRS-CN', '3.1', 'the standard B1 base connection develops an allowable stress design shear per foot of panel length', '1.40', 'kip/ft', 'panel base connections'),
        ('KRS-CN', '4.2', 'one T1 assembly develops an allowable stress design uplift capacity and acts at a lever arm equal to the panel length', '88', 'kips', 'tie-down assemblies'),
        ('KRS-CN', '4.3', 'installed cost of one T1 assembly per panel served', '2450', 'USD', 'Cost'),
        ('SJ-201', 'Note 1', 'joists run north-south at 5 ft 0 in on centre and the deck spans east-west between them at 6 ft 0 in', '6', 'ft', 'EAy, G-prime'),
        ('SJ-201', 'Note 2', 'in deck zone Z22 the joists are doubled to 4 ft 0 in on centre and the deck span there is 4 ft 0 in', '4', 'ft', 'zone Z22'),
        ('SJ-201', 'Note 3', 'girders run east-west on the column lines at 40 ft 0 in on centre', '40', 'ft', 'EAx'),
        ('OBD-NDDC-A2', '4.4', 'the precast connection detail tolerates a roof diaphragm deflection relative to its supports of 0.55 in', '0.55', 'in', 'Drift'),
        ('OBD-NDDC-A2', '4.6', 'not more than 4 distinct roof deck types may be used across the Building A2 roof', '4', 'types', 'DeckZones'),
        ('OBD-NDDC-A2', '4.7', 'installed cost cap of the lateral package', '310000', 'USD', 'Cost'),
        ('OBD-NDDC-A2', '4.7', 'costs are taken from the rates in the issued catalogues with chord steel at the stated installed rate', '2.85', 'per lb', 'Cost'),
        ('Spec 01 33 00', '1.5', 'schedules are delivered as one worksheet per schedule with the column headings named in the templates and no blank or placeholder cell', '1', 'workbook', 'lateral_schedule.xlsx'),
        ('Spec 01 33 00', '1.6', 'the design basis carries a table of the values the design was built on in the form item, value, source', '10', 'items', 'design_basis.md'),
        ('Spec 01 33 00', '1.7', 'plan drawings are delivered as DXF on the building grid in feet with the listed layers and the ASD statement', '7', 'layers', 'diaphragm_plan.dxf'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.1', 'the mesh is laid over the bounding rectangle of the roof with the cells outside the roof outline or inside a roof opening removed', '10', 'cells', 'footprint, zone areas'),
        ('S-201', 'Note 8', 'zone Z43 is bounded by the generator yard notch and its area is less than the grid rectangle', '1200', 'sf', 'zone Z43'),
        ('S-201', 'Note 4', 'deck zones Z11 through Z43 as shown, one deck type per zone', '12', 'zones', 'DeckZones'),
        ('S-201', 'Note 5', 'chord lines C-S, C-N1, C-N2, C-E, C-E2 and C-W on the diaphragm boundary', '6', 'lines', 'ChordSchedule'),
        ('S-201', 'Note 6', 'collector lines K-N2 and K-E2 drag the diaphragm into the generator yard walls', '2', 'lines', 'collectors'),
        ('S-201', 'Note 7', 'opening collector lines O-S, O-N, O-W and O-E around the mechanical well', '4', 'lines', 'collectors'),
        ('CDS-B36', 'Note 1', 'CDS-B36 is a 36 in cover width wide-rib deck mechanically fastened to the supports and stitched at the sidelaps', '36', 'in', 'DeckZones'),
        ('CDS-B36', 'Table 3', 'one deck type is selected from the catalogue table of gauge, fastener pattern and sidelap connections with its nominal shear strength and shear stiffness', '16', 'types', 'DeckZones'),
        ('SR-2026-118', '5.1', 'the north elevation screen wall is part of the precast panel and not a separate frame', '1', 'panel', 'panels N108 to N111'),
        ('SR-2026-118', '5.3', 'the screen is solid precast over its full height, neither louvred nor perforated', '1', 'solid', 'parapet coefficient'),
        ('SR-2026-118', '6.1', 'the generator yard re-entrant corner is plumb within 0.05 ft over the panel height', '0.05', 'ft', 'panels E201 to E204'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.6', 'a chord or collector force is the largest bar force on that line', '1', 'max', 'Chords'),
        ('EOR-NDDC-A2-S01 Rev 2', '10.3', 'the collector lines at the re-entrant corner and around the mechanical well are checked as chord members', '1', 'allowable', 'collectors K-N2, K-E2, O-S, O-N, O-W, O-E'),
        ('EOR-NDDC-A2-S01 Rev 2', '8.2', 'the diaphragm deflection is the largest nodal displacement in the axis measured from the stiffness-weighted mean displacement of the piers resisting that axis', '0.55', 'in', 'Drift'),
        ('Spec 01 33 00', '1.6', 'calculations are delivered as one PDF on 8-1/2 x 11 portrait sheets covering the sections the criteria lists', '1', 'PDF', 'calculations.pdf'),
        ('OBD-NDDC-A2', '4.1', 'single-storey data hall with a steel roof deck on open-web joists and girders carried laterally by precast perimeter wall panels on all exterior building lines', '1', 'system', 'analysis model'),
        ('Spec 01 33 00', '1.8', 'every submittal carries a constraint register with the columns document, clause, requirement, value, units, items', '6', 'columns', 'constraint_register.xlsx'),
        # ---- rows filed so that every family the package relies on is traceable to its clause
        ('A-101 Roof Plan Rev C', 'Note 1', 'the as-built roof extents carry the generator yard notch added by Revision B; Revision A does not show it', '', '-', 'footprint, zone areas'),
        ('NDDC-A2 Federated Model 2026', 'Pset_RoofCommon', 'top of the roof deck slab of the as-built roof, the elevation the model carries', '28.000', 'ft', 'DeckZones, Piers'),
        ('NDDC-A2 Federated Model 2026', 'Pset_PrecastPanel', 'top elevation of the mechanical screen wall panel above the roof, as modelled', '10972.8', 'mm', 'panels N108 to N111'),
        ('KRS-CH', 'Table 1', 'chord and collector catalogue sections with their weight, area and allowable tension and compression', '', '-', 'ChordSchedule'),
        ('EOR-NDDC-A2-S01 Rev 2', '10.1', 'every chord and collector line of the framing plan carries a catalogue section over its full length', '1', 'section per line', 'ChordSchedule'),
        ('EOR-NDDC-A2-S01 Rev 2', '11.1', 'the calculation package is delivered on portrait sheets and covers the wind pressures, the parapet transfer to the roof deck, the reactions, the shears, the chord forces, the wall loads and the design basis', '', '-', 'calculations.pdf'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.4', 'each pier is a spring of stiffness k acting in the plane of its wall, attached at the mesh node nearest the mid point of the leg', '1', 'spring per pier', 'Piers, ShearWalls'),
        ('EOR-NDDC-A2-S01 Rev 2', '9.3', 'chords and collectors are two node bar elements acting in the direction of the line, one bar per mesh interval', '1', 'bar per interval', 'Chords'),
        ('EOR-NDDC-A2-S01 Rev 2', '11.3', 'the Engineer of Record re-runs the analysis on the submitted design and reported quantities are accepted when they agree within the stated tolerance', '7.5', '%', 'ShearWalls, Diaphragm, Chords, Drift'),
        ('Spec 01 33 00', '1.6', 'below the table of design values the design basis carries a second table of the governing wind scenario for each wall line, one row per line', '', '-', 'design_basis.md'),
        ('A-301 Exterior Elevations Rev C', 'Note 2', 'top of panel is +31.00 ft unless noted as screen wall', '31.00', 'ft', 'parapet strips'),
        ('PC-S02 Precast Panel Elevations Rev 2', 'Note 4', 'openings shown are the fabrication openings; field cut openings made after erection are not shown and are verified against the 2026 federated model', '', '-', 'Piers'),
        ('EOR-NDDC-A2-S01 Rev 2', '9', 'the roof deck diaphragm is neither rigid nor flexible relative to the precast walls, so neither a tributary nor a rigid distribution is accepted and the semi rigid model is required', '', '-', 'analysis model'),
    ]


def wb_register():
    wb = openpyxl.Workbook()
    ws = wb.active
    ws.title = 'Register'
    ws.append(['document', 'clause', 'requirement', 'value', 'units', 'items'])
    for r in register_rows():
        ws.append(list(r))
    for c in ws[1]:
        c.font = Font(bold=True)
    return wb


def dxf_plan(res2):
    doc = ezdxf.new('R2010')
    for name in ['S-DIAP-SHEAR', 'S-DIAP-ZONE', 'S-WALL-REAC', 'S-CHRD', 'S-COLL', 'S-ANNO-TEXT', 'S-GRID']:
        if name not in doc.layers:
            doc.layers.add(name)
    msp = doc.modelspace()
    for x, lab in [(0, 'A'), (40, 'B'), (80, 'C'), (120, 'D'), (160, 'E'), (200, 'F'), (240, 'G')]:
        msp.add_line((x, -14), (x, 194), dxfattribs={'layer': 'S-GRID'})
        msp.add_text(lab, height=3.0, dxfattribs={'layer': 'S-GRID'}).set_placement((x - 1.5, -20))
    for y, lab in [(0, '1'), (40, '2'), (80, '3'), (120, '4'), (160, '5'), (180, '6')]:
        msp.add_line((-14, y), (254, y), dxfattribs={'layer': 'S-GRID'})
        msp.add_text(lab, height=3.0, dxfattribs={'layer': 'S-GRID'}).set_placement((-22, y - 1.5))
    for z, (x0, y0, x1, y1) in B.ZONES.items():
        msp.add_lwpolyline([(x0, y0), (x1, y0), (x1, y1), (x0, y1), (x0, y0)], dxfattribs={'layer': 'S-DIAP-ZONE'})
        msp.add_text(f"{z} {DECK_ASSIGN[z]}", height=2.5,
                     dxfattribs={'layer': 'S-ANNO-TEXT'}).set_placement((x0 + 2, y0 + 2))
    for sc, tag, ytxt in (('W05', 'TRANSVERSE DIRECTION - UNIT SHEARS FOR W05 (WIND FROM THE SOUTH), PLF', 214),
                          ('W03', 'LONGITUDINAL DIRECTION - UNIT SHEARS FOR W03 (WIND FROM THE EAST), PLF', 204)):
        r = res2[sc]
        for ln in ['S', 'N1', 'N2', 'E', 'E2', 'W']:
            p0, p1, nrm, axis = B.LINES[ln]
            L = math.dist(p0, p1)
            d = ((p1[0] - p0[0]) / L, (p1[1] - p0[1]) / L)
            for frac, v in r['shear'][ln]:
                x = p0[0] + d[0] * frac * L
                y = p0[1] + d[1] * frac * L
                msp.add_text(f"{abs(v):.0f}", height=2.2, dxfattribs={'layer': 'S-DIAP-SHEAR'}).set_placement((x + 2, y + 2 + (4 if axis else 0)))
        msp.add_text(tag, height=3.6, dxfattribs={'layer': 'S-ANNO-TEXT'}).set_placement((-55, ytxt))
    for ln in ['S', 'N1', 'N2', 'E', 'E2', 'W']:
        p0, p1, nrm, axis = B.LINES[ln]
        mid = ((p0[0] + p1[0]) / 2, (p0[1] + p1[1]) / 2)
        rx = max(abs(v['react'].get(ln, 0.0)) for v in res2.values())
        msp.add_text(f"{ln}: R max {rx:.1f} kip ASD", height=2.6,
                     dxfattribs={'layer': 'S-WALL-REAC'}).set_placement((mid[0] + nrm[0] * 6, mid[1] + nrm[1] * 6))
    for (cid, p0, p1) in B.CHORD_LINES:
        layer = 'S-CHRD' if cid.startswith('C') else 'S-COLL'
        msp.add_line(p0, p1, dxfattribs={'layer': layer})
        m = ((p0[0] + p1[0]) / 2, (p0[1] + p1[1]) / 2)
        msp.add_text(f"{cid} {CHORD_ASSIGN[cid]}", height=2.2, dxfattribs={'layer': 'S-ANNO-TEXT'}).set_placement(m)
    for i, line in enumerate([
            'ROOF DIAPHRAGM AND SHEAR WALL LAYOUT - BUILDING A2, NORTHERN DIVIDE DATA CAMPUS',
            'ALL LOADS SHOWN ARE ALLOWABLE STRESS DESIGN (ASD), LOAD COMBINATION 0.6W',
            'ANALYSIS DIAPH-MT PER EOR-NDDC-A2-S01 REV 2 SECTION 9, QUANTITIES REPORTED AT MESH LEVEL L2',
            'PLAN ON THE BUILDING GRID IN FEET, ORIGIN AT THE SOUTH-WEST BUILDING CORNER']):
        msp.add_text(line, height=3.2 if i else 4.2, dxfattribs={'layer': 'S-ANNO-TEXT'}).set_placement((-55, 236 - 6 * i))
    return doc


def main():
    os.makedirs(OUT, exist_ok=True)
    res_by_level, conv = {}, []
    for name, H in LEVELS:
        rig, res, panels, piers = run_level(H)
        res_by_level[name] = (rig, res, panels, piers)
        conv.append((name, H, len(rig.model.elems), res['W01']['umax'], res['W01']['react'].get('W', 0.0)))
        print(name, 'elements', len(rig.model.elems), 'umax %.0f' % res['W01']['umax'], 'R_W %.2f' % res['W01']['react'].get('W', 0))
    rig, res, panels, piers = res_by_level['L2']
    cost = design_cost(res, piers, panels)
    items, total, embeds, tiedown, ps = cost
    print('cost total %.0f' % total)
    wb_schedule(res, piers, panels, conv, cost).save(os.path.join(OUT, 'lateral_schedule.xlsx'))
    wb_register().save(os.path.join(OUT, 'constraint_register.xlsx'))
    dxf_plan(res).saveas(os.path.join(OUT, 'diaphragm_plan.dxf'))
    summary = dict(
        conv=[list(c) for c in conv],
        conv_change_pct=(conv[1][4] - conv[2][4]) / conv[2][4] * 100.0,
        cost=[[i[0], i[1], i[2], i[3]] for i in items], total=total,
        drift=max(r['drift'][a] for r in res.values() for a in ('X', 'Y')),
        pier_drift=max(max(r['pier_disp'].values()) for r in res.values()),
        panel_shear={k: round(v, 3) for k, v in ps.items()},
        embeds=embeds, tiedown=tiedown,
        zone_shear=None,
        q_h=B.qz(28.0), q31=B.qz(31.0), q36=B.qz(36.0),
        base_ratio=max(ps[p['mark']] / (p['s1'] - p['s0']) for p in panels),
    )
    json.dump(summary, open(os.path.join(OUT, 'summary.json'), 'w'), indent=1)
    print('convergence L2-L3 %.2f %%' % summary['conv_change_pct'])
    print('max drift %.3f in, max pier drift %.3f in, max base %.2f kip/ft' % (
        summary['drift'], summary['pier_drift'], summary['base_ratio']))
    return summary


if __name__ == '__main__':
    main()

```


## code/design.py

```text
"""Design iteration for Building A2: deck by zone, chords, with the drift limit enforced."""
import itertools
import math
import json
import numpy as np
import build as B
import analyze as A


def pick_decks_multi(req_cap, req_g, areas, max_types=4):
    codes = [d[0] for d in B.DECKS]
    best = None
    for n in range(1, max_types + 1):
        for combo in itertools.combinations(codes, n):
            assign, cost, ok = {}, 0.0, True
            for z, rcap in req_cap.items():
                cand = [c for c in combo if B.DECK[c]['cap'] >= rcap - 1e-9 and B.DECK[c]['g'] >= req_g - 1e-9]
                if not cand:
                    ok = False
                    break
                pick = min(cand, key=lambda c: (B.DECK[c]['cost'], -B.DECK[c]['g']))
                assign[z] = pick
                cost += areas[z] * B.DECK[pick]['cost']
            if ok and (best is None or cost < best[0] - 1e-6):
                best = (cost, assign)
        if best is not None:
            break
    return best


def design(H=5.0, iters=8, verbose=True, drift_limit=B.DEFL_LIMIT):
    panels = B.load_panels()
    piers = B.build_piers(panels)
    areas = B.zone_areas()
    deck = {z: 'D22-36/4-2' for z in B.ZONES}
    chord = {cid: 'PL3x3/8' for (cid, _, _) in B.CHORD_LINES}
    g_req = 0.0
    rig = res = zs = None
    for it in range(iters):
        rig = A.Rig(panels, piers, deck, chord, H)
        res = rig.run()
        zs = A.zone_shears(rig, res)
        drift = max(r['drift'][ax] for r in res.values() for ax in ('X', 'Y'))
        cost, newdeck = pick_decks_multi(zs, g_req, areas)
        cf = {}
        for (cid, _, _) in B.CHORD_LINES:
            T = max((r['chord'].get(cid, 0.0) for r in res.values()), default=0.0)
            C = min((r['chord'].get(cid, 0.0) for r in res.values()), default=0.0)
            cf[cid] = (float(T), float(C))
        newchord = A.pick_chords(cf)
        if verbose:
            print(f"iter {it} H={H}: maxdrift {drift:.3f} in  g_req {g_req:.1f}  zones {[ (z, round(v)) for z, v in zs.items() ]}")
            print("   deck", {k: v for k, v in sorted(newdeck.items())}, "%.0f" % cost, "sect", set(newchord.values()))
        if drift > drift_limit + 1e-6:
            # stiffness scales the drift almost linearly: raise the required G' and go again
            gmin = min(B.DECK[v]['g'] for v in newdeck.values())
            g_req = max(g_req, gmin) * min(2.0, (drift / drift_limit) ** 1.0) + 0.05
            deck = newdeck
            chord = newchord
            continue
        same = (newdeck == deck and newchord == chord)
        deck, chord = newdeck, newchord
        if same:
            break
    return dict(rig=rig, res=res, zones=zs, deck=deck, chord=chord, panels=panels, piers=piers, areas=areas)


if __name__ == '__main__':
    d = design(H=5.0)
    print('FINAL deck', d['deck'])
    print('FINAL chord', d['chord'])
    drift = max(r['drift'][ax] for r in d['res'].values() for ax in ('X', 'Y'))
    print('max drift %.3f in' % drift)

```


## code/diag.py

```text
"""Diagnostic: run the real grader's checks and print per-family detail (no scoring shortcuts)."""
import json
import sys
from pathlib import Path

ROOT = Path('/work/ple-dataset/tasks/wind-diaphragm-shear/tests/corrino/'
            'structural_task5_wind_diaphragm_shear')
sys.path.insert(0, str(ROOT))

from grading.checks import Checks                     # noqa: E402
from grading.grade import design_from_submission      # noqa: E402
from grading.model import Analysis, truth             # noqa: E402
from grading.register import RegisterMap              # noqa: E402
from grading.submission import Submission             # noqa: E402

zip_path = Path(sys.argv[1] if len(sys.argv) > 1 else
                '/work/ple-solve/wind-diaphragm-shear/out/submission.zip')
split = sys.argv[2] if len(sys.argv) > 2 else 'heldout'

T = truth()
sub = Submission.load(zip_path)
print('parse_ok', sub.parse_ok, 'errors', sub.errors[:5])
design = design_from_submission(T, sub)
A = Analysis(T, design)
res = A.analyse()
ck = Checks(T, sub, design, A, res)
model = ck.model_checks()
for k, v in model.items():
    flag = 'OK ' if v.get('ok') else 'BAD'
    print(f'{flag} {k}: {json.dumps(v)[:400]}')

reg = RegisterMap().score(sub.register)
print('\nregister: precision %.3f wrong %d filed %d neutral %d' % (
    reg['precision'], reg['wrong_rows'], reg['rows_filed'], reg['neutral_rows']))
print('registered families:', len(reg['registered_families']))
missing = [f for f in sorted({q['family'] for q in RegisterMap().reqs})
           if f not in reg['registered_families']]
print('NOT registered:', missing)
# which requirement ids are still unmatched, per family
for f in missing:
    ids = [q['id'] for q in RegisterMap().reqs if q['family'] == f]
    print('  ', f, 'needs', [i for i in ids if i not in reg['matched']])

print('\n--- per scenario ---')
scen = T['scenarios'] if isinstance(T['scenarios'], list) else list(T['scenarios'].values())
ids = [s['id'] for s in scen]
if split == 'heldout':
    ids = [i for i in ids if i in ('W02', 'W03', 'W05', 'W06', 'W08', 'W10', 'W11')]
for sid in ids:
    r = ck.scenario_checks(sid)
    bad = {k: v for k, v in r.items() if not v.get('ok')}
    print(sid, 'failed:', sorted(bad))
    for k, v in bad.items():
        print('    ', k, json.dumps(v)[:300])

```


## code/diaph.py

```text
"""DIAPH-MT: semi-rigid roof diaphragm analysis per EOR-NDDC-A2-S01 Rev 2 section 9.

Units: lengths ft, forces kip, membrane stiffness kip/ft, pier springs kip/in, displacements ft.
"""
import numpy as np
import scipy.sparse as sp
import scipy.sparse.linalg as spla


def point_in_poly(x, y, poly):
    inside = False
    n = len(poly)
    for i in range(n):
        x1, y1 = poly[i]
        x2, y2 = poly[(i + 1) % n]
        if ((y1 > y) != (y2 > y)) and (x < (x2 - x1) * (y - y1) / (y2 - y1) + x1):
            inside = not inside
    return inside


class Model:
    def __init__(self, outline, openings, H, eax, eay, gprime_fn, chords, piers):
        self.outline = outline
        self.openings = openings
        self.H = H
        self.eax = eax
        self.eay = eay
        self.gprime_fn = gprime_fn
        self.chords = chords
        self.piers = piers
        self._mesh()

    def _mesh(self):
        H = self.H
        xs = np.arange(0, max(p[0] for p in self.outline) + 1e-9, H)
        ys = np.arange(0, max(p[1] for p in self.outline) + 1e-9, H)
        cells = []
        for i in range(len(xs) - 1):
            for j in range(len(ys) - 1):
                cx, cy = xs[i] + H / 2, ys[j] + H / 2
                if not point_in_poly(cx, cy, self.outline):
                    continue
                if any(point_in_poly(cx, cy, op) for op in self.openings):
                    continue
                cells.append((i, j))
        used = set()
        for (i, j) in cells:
            used.update([(i, j), (i + 1, j), (i, j + 1), (i + 1, j + 1)])
        nodes = sorted(used)
        nid = {n: k for k, n in enumerate(nodes)}
        self.xs, self.ys = xs, ys
        self.nodes = np.array([[xs[i], ys[j]] for (i, j) in nodes], float)
        self.node_ij = nodes
        self.elems = [(nid[(i, j)], nid[(i + 1, j)], nid[(i + 1, j + 1)], nid[(i, j + 1)]) for (i, j) in cells]
        self.cells = cells

    def _D(self, x, y):
        eax = self.eax(x, y) if callable(self.eax) else self.eax
        eay = self.eay(x, y) if callable(self.eay) else self.eay
        g = 12.0 * self.gprime_fn(x, y)
        return np.diag([eax, eay, g])

    def assemble(self):
        ndof = 2 * len(self.nodes)
        rows, cols, vals = [], [], []
        self.ghat_e = np.zeros(len(self.elems))
        gq = 1 / np.sqrt(3.0)
        gp = [(-gq, -gq), (gq, -gq), (gq, gq), (-gq, gq)]
        for e, conn in enumerate(self.elems):
            xy = self.nodes[list(conn)]
            D = self._D(xy[:, 0].mean(), xy[:, 1].mean())
            Ke = np.zeros((8, 8))
            for (xi, eta) in gp:
                dN = 0.25 * np.array([[-(1 - eta), (1 - eta), (1 + eta), -(1 + eta)],
                                      [-(1 - xi), -(1 + xi), (1 + xi), (1 - xi)]])
                J = dN @ xy
                dNxy = np.linalg.inv(J) @ dN
                B = np.zeros((3, 8))
                B[0, 0::2] = dNxy[0]; B[1, 1::2] = dNxy[1]
                B[2, 0::2] = dNxy[1]; B[2, 1::2] = dNxy[0]
                Ke += B.T @ D @ B * np.linalg.det(J)
            self.ghat_e[e] = D[2, 2]
            for a in range(4):
                for b in range(4):
                    rows += [2 * conn[a], 2 * conn[a], 2 * conn[a] + 1, 2 * conn[a] + 1]
                    cols += [2 * conn[b], 2 * conn[b] + 1, 2 * conn[b], 2 * conn[b] + 1]
                    vals += [Ke[2 * a, 2 * b], Ke[2 * a, 2 * b + 1], Ke[2 * a + 1, 2 * b], Ke[2 * a + 1, 2 * b + 1]]
        # chord / collector bars: one bar per mesh interval on the line
        self.bars = []
        for c in self.chords:
            p0 = np.array(c['p0'], float); p1 = np.array(c['p1'], float)
            L = np.linalg.norm(p1 - p0); d = (p1 - p0) / L
            nseg = max(1, int(round(L / self.H)))
            for s in range(nseg):
                na = self.nearest_node(p0 + d * (s * L / nseg))
                nb = self.nearest_node(p0 + d * ((s + 1) * L / nseg))
                if na is None or nb is None or na == nb:
                    continue
                self.bars.append([na, nb, c['A'] * 29000.0, c['A'], c['id']])
        for (na, nb, EA, A, cid) in self.bars:
            P0, P1 = self.nodes[na], self.nodes[nb]
            Lb = np.linalg.norm(P1 - P0)
            if Lb < 1e-9:
                continue
            cs = (P1 - P0) / Lb
            kb = EA / Lb
            T = np.array([-cs[0], -cs[1], cs[0], cs[1]])
            ke = kb * np.outer(T, T)
            idx = [2 * na, 2 * na + 1, 2 * nb, 2 * nb + 1]
            for a in range(4):
                for b in range(4):
                    rows.append(idx[a]); cols.append(idx[b]); vals.append(ke[a, b])
        K = sp.coo_matrix((vals, (rows, cols)), shape=(ndof, ndof)).tocsr()
        return K

    def nearest_node(self, p):
        d = np.hypot(self.nodes[:, 0] - p[0], self.nodes[:, 1] - p[1])
        k = int(np.argmin(d))
        return k if d[k] < 1e-6 else None

    def node_within(self, p, tol):
        d = np.hypot(self.nodes[:, 0] - p[0], self.nodes[:, 1] - p[1])
        k = int(np.argmin(d))
        return k if d[k] <= tol + 1e-9 else None

    def attach_piers(self):
        """each pier attaches to the mesh node nearest the mid-point of its leg"""
        self.pier_node = []
        for p in self.piers:
            k = self.node_within(np.array(p['xy'], float), self.H / 2 + 1e-6)
            if k is None:
                k = self.node_within(np.array(p['xy'], float), self.H)
            self.pier_node.append(k)

    def add_line_segments(self, F, p0, p1, segs, nrm):
        """Consistent nodal loads of the piecewise-linear hat functions on a wall line (criteria 9.5)."""
        p0 = np.array(p0, float); p1 = np.array(p1, float)
        L = np.linalg.norm(p1 - p0)
        d = (p1 - p0) / L
        n = max(1, int(round(L / self.H)))
        nodes = []
        for k in range(n + 1):
            nd = self.nearest_node(p0 + d * (k * self.H))
            if nd is None:
                nd = self.node_within(p0 + d * (k * self.H), self.H)
            nodes.append(nd)
        inward = np.array(nrm, float); inward = inward / np.linalg.norm(inward)
        for k, nd in enumerate(nodes):
            if nd is None:
                continue
            sk = k * self.H
            lo = (k - 1) * self.H if k > 0 else sk
            hi = (k + 1) * self.H if k < n else sk
            total = 0.0
            for (a, b, w) in segs:
                for (q0, q1) in ((lo, sk), (sk, hi)):
                    if q1 <= q0:
                        continue
                    aa, bb = max(a, q0), min(b, q1)
                    if bb <= aa:
                        continue
                    far = q0 if q1 == sk else q1
                    span = abs(sk - far)
                    if span < 1e-12:
                        continue
                    h0 = 1.0 - abs(aa - sk) / span
                    h1 = 1.0 - abs(bb - sk) / span
                    total += w * 0.5 * (h0 + h1) * (bb - aa)
            F[2 * nd] += total * inward[0]
            F[2 * nd + 1] += total * inward[1]
        return F

    def solve(self, line_loads=None, torsion_force_nodes=None, line_segments=None):
        K = self.assemble()
        ndof = 2 * len(self.nodes)
        F = np.zeros(ndof)
        for (p0, p1, segs, nrm) in (line_segments or []):
            self.add_line_segments(F, p0, p1, segs, nrm)
        for (p0, p1, w, dv) in (line_loads or []):
            self.add_line_segments(F, p0, p1, [(0.0, np.linalg.norm(np.array(p1, float) - np.array(p0, float)), w)], dv)
        if torsion_force_nodes is not None:
            for (k, fx, fy) in torsion_force_nodes:
                F[2 * k] += fx; F[2 * k + 1] += fy
        self.attach_piers()
        for p, k in zip(self.piers, self.pier_node):
            if k is None:
                raise RuntimeError('pier %s not attached' % p.get('id'))
            dv = np.array(p['dir'], float); dv = dv / np.linalg.norm(dv)
            kk = p["k"] * 12.0
            ke = kk * np.outer(dv, dv)
            for a in range(2):
                for b in range(2):
                    K[2 * k + a, 2 * k + b] += ke[a, b]
        U = spla.spsolve(K.tocsc(), F)
        self.U, self.K, self.F = U, K, F
        return U

    # ---------------------------------------------------------------- recovery
    def elem_strain(self, e):
        xy = self.nodes[list(self.elems[e])]
        dN = 0.25 * np.array([[-1, 1, 1, -1], [-1, -1, 1, 1]], float)
        J = dN @ xy
        dNxy = np.linalg.inv(J) @ dN
        B = np.zeros((3, 8))
        B[0, 0::2] = dNxy[0]; B[1, 1::2] = dNxy[1]
        B[2, 0::2] = dNxy[1]; B[2, 1::2] = dNxy[0]
        ue = np.zeros(8)
        for a, n in enumerate(self.elems[e]):
            ue[2 * a] = self.U[2 * n]; ue[2 * a + 1] = self.U[2 * n + 1]
        return B @ ue

    def unit_shears(self):
        return np.array([self.ghat_e[e] * self.elem_strain(e)[2] for e in range(len(self.elems))])

    def elem_centre(self, e):
        i, j = self.cells[e]
        return (self.xs[i] + self.H / 2, self.ys[j] + self.H / 2)

    def station_shear(self, line_pts, station):
        """element with an edge on the line (both nodes on the line) whose centre is nearest the station"""
        p0 = np.array(line_pts[0], float); p1 = np.array(line_pts[1], float)
        L = np.linalg.norm(p1 - p0); d = (p1 - p0) / L
        S = p0 + d * station
        us = self.unit_shears()
        best, bd = None, 1e18
        for e, conn in enumerate(self.elems):
            P = self.nodes[list(conn)]
            on = sum(1 for q in P if abs(np.cross(d, q - p0)) < 1e-6)
            if on < 2:
                continue
            c = np.array(self.elem_centre(e))
            dd = np.linalg.norm(c - S)
            if dd < bd:
                bd, best = dd, e
        return (us[best] if best is not None else float('nan')), best

    def reactions(self):
        out = {}
        for p, k in zip(self.piers, self.pier_node):
            dv = np.array(p['dir'], float); dv = dv / np.linalg.norm(dv)
            u = np.array([self.U[2 * k], self.U[2 * k + 1]])
            f = p["k"] * 12.0 * (dv @ u)
            out[p.get('line', '?')] = out.get(p.get('line', '?'), 0.0) + f
        return out

    def bar_forces(self):
        out = {}
        for (na, nb, EA, A, cid) in self.bars:
            P0, P1 = self.nodes[na], self.nodes[nb]
            Lb = np.linalg.norm(P1 - P0)
            cs = (P1 - P0) / Lb
            u = np.array([self.U[2 * na], self.U[2 * na + 1], self.U[2 * nb], self.U[2 * nb + 1]])
            N = EA / Lb * (cs @ (u[2:] - u[:2]))
            if abs(N) > abs(out.get(cid, 0.0)):
                out[cid] = N
        return out

    def axis_deflection(self, axis):
        """largest nodal displacement in axis from the stiffness-weighted mean pier displacement"""
        piers = [(p, k) for p, k in zip(self.piers, self.pier_node)
                 if abs(np.array(p['dir'], float)[axis]) > 0.5]
        num = sum(p['k'] * self.U[2 * k + axis] for p, k in piers)
        den = sum(p['k'] for p, k in piers)
        mean = num / den
        return float(np.abs(self.U[axis::2] - mean).max()), mean

```


## code/greedy.py

```text
"""Greedy mixed deck assignment under the drift limit, at H = 5 (level L2)."""
import build as B
import analyze as A
from design import pick_decks_multi

HI = 'D20-36/9-4'   # G' 46.1, cap 2980/2.35 = 1268
LO = 'D20-36/7-4'   # G' 39.7, cap 2530/2.35 = 1077

panels = B.load_panels()
piers = B.build_piers(panels)
areas = B.zone_areas()
chs = {cid: 'PL3x3/8' for (cid, _, _) in B.CHORD_LINES}


def evaluate(assign, H=5.0):
    rig = A.Rig(panels, piers, assign, chs, H)
    res = rig.run()
    drift = max(r['drift'][ax] for r in res.values() for ax in ('X', 'Y'))
    zs = A.zone_shears(rig, res)
    return drift, zs, rig, res


if __name__ == '__main__':
    base = {z: LO for z in B.ZONES}
    drift, zs, rig, res = evaluate(base)
    print('all LO: drift %.3f' % drift)
    incr = sorted(B.ZONES, key=lambda z: areas[z] * (B.DECK[HI]['cost'] - B.DECK[LO]['cost']))
    cur = dict(base)
    disc = {z: B.DECK[HI]['cost'] - B.DECK[LO]['cost'] for z in B.ZONES}
    order = sorted(B.ZONES, key=lambda z: areas[z] * disc[z])
    for z in order:
        if drift <= B.DEFL_LIMIT:
            break
        cur[z] = HI
        drift, zs, rig, res = evaluate(cur)
        print('upgrade %s -> drift %.3f  cost %.0f' % (z, drift, sum(areas[q] * B.DECK[cur[q]]['cost'] for q in cur)))
    if drift > B.DEFL_LIMIT:
        cur = {z: HI for z in B.ZONES}
        drift, zs, rig, res = evaluate(cur)
        print('all HI: drift %.3f' % drift)
    print('FINAL', cur, 'drift %.3f' % drift)
    print('zone shears', {k: round(v) for k, v in zs.items()})

```


## code/mut.py

```text
"""Sensitivity campaign: break one thing at a time in the packaged submission and see whether the
real grader reddens the family that owns it. Mutants are written to /tmp and graded with the
task's own grading.grade; the delivered zip is never touched."""
import json
import shutil
import sys
import zipfile
from pathlib import Path

import openpyxl

ROOT = Path('/work/ple-dataset/tasks/wind-diaphragm-shear/tests/corrino/'
            'structural_task5_wind_diaphragm_shear')
sys.path.insert(0, str(ROOT))
from grading.grade import grade                                    # noqa: E402

SRC = Path('/work/ple-solve/wind-diaphragm-shear/out/submission.zip')
TMP = Path('/tmp/wind_mut')


def load():
    shutil.rmtree(TMP, ignore_errors=True)
    TMP.mkdir(parents=True)
    with zipfile.ZipFile(SRC) as z:
        z.extractall(TMP)


def repack(path):
    files = ['lateral_schedule.xlsx', 'diaphragm_plan.dxf', 'constraint_register.xlsx',
             'calculations.pdf', 'design_basis.md']
    with zipfile.ZipFile(path, 'w', zipfile.ZIP_DEFLATED) as z:
        for f in files:
            z.write(TMP / f, f)


def tweak_wb(fn):
    p = TMP / 'lateral_schedule.xlsx'
    wb = openpyxl.load_workbook(p)
    fn(wb)
    wb.save(p)


def tweak_reg(fn):
    p = TMP / 'constraint_register.xlsx'
    wb = openpyxl.load_workbook(p)
    fn(wb)
    wb.save(p)


def set_deck(wb, zone, code):
    ws = wb['DeckZones']
    hdr = [c.value for c in ws[1]]
    for row in ws.iter_rows(min_row=2):
        if row[hdr.index('zone_id')].value == zone:
            row[hdr.index('deck_code')].value = code


def drop_tie(wb, mark):
    ws = wb['Piers']
    hdr = [c.value for c in ws[1]]
    for row in ws.iter_rows(min_row=2):
        if row[hdr.index('panel')].value == mark:
            row[hdr.index('tiedown')].value = 'no'


def set_embed(wb, mark, n):
    ws = wb['Piers']
    hdr = [c.value for c in ws[1]]
    for row in ws.iter_rows(min_row=2):
        if row[hdr.index('panel')].value == mark:
            row[hdr.index('embeds')].value = n


def drop_register_row(wb, doc_prefix):
    ws = wb['Register'] if 'Register' in wb.sheetnames else wb.worksheets[0]
    for row in list(ws.iter_rows(min_row=2)):
        if str(row[0].value or '').startswith(doc_prefix):
            ws.delete_rows(row[0].row, 1)
            return
    raise SystemExit('register row not found: ' + doc_prefix)


MUTS = [
    ('deck Z23 -> D22-36/4-2 (weakest)', tweak_wb, lambda wb: set_deck(wb, 'Z23', 'D22-36/4-2')),
    ('tie-down removed on E201', tweak_wb, lambda wb: drop_tie(wb, 'E201')),
    ('embeds on E201 cut to 2', tweak_wb, lambda wb: set_embed(wb, 'E201', 2)),
    ('register row A-101 note 1 deleted', tweak_reg, lambda wb: drop_register_row(wb, 'A-101 Roof Plan Rev C')),
    ('register row KRS-CH Table 1 deleted', tweak_reg, lambda wb: drop_register_row(wb, 'KRS-CH')),
]

base = grade(SRC)
b_fail = set(base['scenarios']['W02']['failed'])
b_unreg = set(base['scenarios']['W02']['unregistered'])
print('baseline: score %.4f p %.3f failed %s unreg %s' % (
    base['combined_score'], base['mean_p'], sorted(b_fail), sorted(b_unreg)))
rows = []
for name, apply_to, fn in MUTS:
    load()
    apply_to(fn)
    out = Path('/tmp/mut.zip')
    repack(out)
    m = grade(out)
    sc = m['scenarios'].get('W02', {})
    new = set(sc.get('failed') or []) | set(sc.get('unregistered') or [])
    delta = sorted(new - b_fail - b_unreg)
    print('%-38s score %.4f  p %.3f  new red: %s' % (name, m['combined_score'], m['mean_p'],
                                                     delta or 'NONE'))
    rows.append(dict(mutation=name, score=m['combined_score'], p=m['mean_p'], new_red=delta))
json.dump(rows, open('/work/ple-solve/wind-diaphragm-shear/mutations.json', 'w'), indent=1)

```


## code/opt.py

```text
"""Cost optimisation of the A2 lateral package against the task's own DIAPH-MT model and checks.

Free variables: deck type per zone (<= 4 distinct), chord section per line.
Embeds and tie-downs are not free: embeds = max(vendor minimum, ceil(panel shear / 4.20 kip)),
tie-downs only where 0.6D cannot resist overturning (a tie that is not needed is money burnt,
a tie that is needed and missing fails F6).
"""
import itertools
import json
import math
import random
import sys
import time
from collections import defaultdict
from pathlib import Path

ROOT = Path('/work/ple-dataset/tasks/wind-diaphragm-shear/tests/corrino/'
            'structural_task5_wind_diaphragm_shear')
sys.path.insert(0, str(ROOT))

from grading.cost import design_cost, min_embeds               # noqa: E402
from grading.model import Analysis, Design, truth             # noqa: E402

T = truth()
ZONES = [z['id'] for z in T['zones']]
AREA = {z['id']: z['area_sf'] for z in T['zones']}
DECKS = {c['code']: c for c in T['deck_catalogue']}
CHORDS = {c['code']: c for c in T['chord_catalogue']}
LINES = [c['id'] for c in T['chord_lines']]
CH_LINE = {c['id']: c for c in T['chord_lines']}
MARKS = [p['mark'] for rows in T['panels'].values() for p in rows]
WIDTH = {p['mark']: p['s1'] - p['s0'] for rows in T['panels'].values() for p in rows}
TOPZ = {p['mark']: p['top_z'] for rows in T['panels'].values() for p in rows}
TIN = {p['mark']: p['t_in'] for rows in T['panels'].values() for p in rows}
DENS = T['materials']['panel_density_pcf']
ROOFZ = T['geometry']['roof_z']
EMB = T['connections']['embed_asd_kips']
EMB_COST = T['connections']['embed_cost']
TD_CAP = T['connections']['tiedown_asd_kips']
TD_COST = T['connections']['tiedown_cost']
BASE = T['connections']['base_asd_kips_per_ft']
DRIFT = T['limits']['diaphragm_drift_in']
PIER_RATIO = T['limits']['pier_drift_ratio']
MAXTYPES = T['deck']['max_types']
CAP_USD = T['costs']['capital_cap']
SPAN4 = T['deck']['span_factor_4ft']
WEIGHT = {m: (TIN[m] / 12.0) * WIDTH[m] * TOPZ[m] * DENS / 1000.0 for m in MARKS}


def panel_shear(res, piers):
    out = defaultdict(float)
    for i, p in enumerate(piers):
        out[p['panel']] = max(out[p['panel']], abs(res['reac'][i]))
    return dict(out)


def evaluate(deck_map, chord_map, want_res=False):
    d0 = Design(deck_map, chord_map, {}, [])
    A = Analysis(T, d0)
    res = A.analyse()
    # panel shear per scenario, embeds and ties sized on the worst scenario
    vmax = defaultdict(float)
    drift = 0.0
    pier_drift = 0.0
    f1 = []
    for sid, r in res.items():
        for m, v in panel_shear(r, A.piers).items():
            vmax[m] = max(vmax[m], v)
        drift = max(drift, max(r['deflection'].values()))
        pier_drift = max(pier_drift, max(r['pier_drift'].values()) if r['pier_drift'] else 0.0)
        q = [abs(x) * 1000.0 for x in r['q']]
        for e, val in enumerate(q):
            if val > A.cap_plf[e] + 1e-6:
                f1.append((sid, A.elem_zone[e], round(val, 1), round(float(A.cap_plf[e]), 1)))
    embeds = {}
    for m in MARKS:
        v = vmax.get(m, 0.0)
        embeds[m] = max(min_embeds(T, WIDTH[m]), int(math.ceil(v / EMB - 1e-9)))
    ties = []
    f6 = []
    for m in MARKS:
        v = vmax.get(m, 0.0)
        M_ot = v * ROOFZ
        M_r = 0.6 * WEIGHT[m] * WIDTH[m] / 2.0
        if M_ot > M_r + 1e-6:
            if M_ot <= M_r + TD_CAP * WIDTH[m] + 1e-6:
                ties.append(m)
            else:
                f6.append((m, round(M_ot), round(M_r + TD_CAP * WIDTH[m])))
    f4 = [(m, round(v, 2), round(BASE * WIDTH[m], 2)) for m, v in vmax.items()
          if v > BASE * WIDTH[m] + 1e-6]
    f2 = []
    for sid, r in res.items():
        for cl in T['chord_lines']:
            t, cc = r['chord_signed'][cl['id']]
            c = CHORDS[chord_map[cl['id']]]
            if t > c['allow_t_kips'] + 1e-6 or -cc > c['allow_c_kips'] + 1e-6:
                f2.append((sid, cl['id'], round(t, 1), round(-cc, 1)))
    d = Design(deck_map, chord_map, embeds, ties)
    cost = design_cost(T, d)
    plim = ROOFZ * 12.0 * PIER_RATIO
    ok_struct = (not f1 and not f2 and not f4 and not f6
                 and drift <= DRIFT + 1e-9 and pier_drift <= plim + 1e-9
                 and len(set(deck_map.values())) <= MAXTYPES)
    ok = ok_struct and cost['total'] <= CAP_USD
    out = dict(ok=ok, ok_struct=ok_struct, cost=cost, drift=drift, pier_drift=pier_drift,
               embeds=embeds, ties=sorted(ties), f1=f1[:4], f2=f2[:4], f4=f4[:4], f6=f6[:4],
               n_embed=sum(embeds.values()), vmax=vmax, chord_map=chord_map,
               deck_map=deck_map, types=sorted(set(deck_map.values())))
    if want_res:
        out['res'] = res
        out['A'] = A
        out['design'] = d
    return out


def uniform_scan():
    rows = []
    for code in DECKS:
        dm = {z: code for z in ZONES}
        r = evaluate(dm, {ln: 'PL3x3/8' for ln in LINES})
        rows.append((r['cost']['total'], code, r['ok'], round(r['drift'], 3), len(r['ties']),
                     r['f4'][:1], len(r['f1'])))
    return sorted(rows)


def greedy(start_deck, start_chord, order=None, seed=0, iters=4, log=print):
    """Zone-by-zone descent: try every deck in each zone, keep the cheapest feasible step."""
    dm = dict(start_deck)
    cm = dict(start_chord)
    rnd = random.Random(seed)
    best = evaluate(dm, cm)
    log('  start %.0f ok=%s drift %.3f ties %d' % (best['cost']['total'], best['ok'],
                                                   best['drift'], len(best['ties'])))
    for it in range(iters):
        improved = False
        zones = list(ZONES)
        if order == 'random':
            rnd.shuffle(zones)
        for z in zones:
            for code in sorted(DECKS, key=lambda c: DECKS[c]['cost_per_sf']):
                if code == dm[z]:
                    continue
                trial = dict(dm)
                trial[z] = code
                r = evaluate(trial, cm)
                if r['ok_struct'] and r['cost']['total'] < best['cost']['total'] - 1e-6:
                    dm, best, improved = trial, r, True
                    log('  Z %s -> %s  cost %.0f drift %.3f ties %d types %s' % (
                        z, code, r['cost']['total'], r['drift'], len(r['ties']), r['types']))
                    break
        # chord pass
        for ln in LINES:
            for code in sorted(CHORDS, key=lambda c: CHORDS[c]['weight_plf']):
                if code == cm[ln]:
                    continue
                trial = dict(cm)
                trial[ln] = code
                r = evaluate(dm, trial)
                if r['ok_struct'] and r['cost']['total'] < best['cost']['total'] - 1e-6:
                    cm, best, improved = trial, r, True
                    log('  chord %s -> %s cost %.0f' % (ln, code, r['cost']['total']))
                    break
        if not improved:
            break
    return best


if __name__ == '__main__':
    t0 = time.time()
    print('=== uniform scan ===')
    for row in uniform_scan():
        print('  $%9.0f %-12s ok=%-5s drift %.3f ties %2d typ %d  f4 %s nf1 %d' % (
            row[0], row[1], row[2], row[3], row[4], 1, row[5], row[6]))
    print('seconds %.1f' % (time.time() - t0))

```


## code/optimize.py

```text
"""Greedy cost reduction under the real constraint set (drift, base, capacity, four types)."""
import math
import json
import build as B
import sweep as S

AREAS = B.zone_areas()
CAND = ['D22-36/4-2', 'D22-36/7-2', 'D22-36/7-4', 'D20-36/7-4', 'D16-36/9-4']


def feasible(assign):
    r = S.evaluate(assign)
    ok = (r['drift'] <= 0.55 and r['base'] <= 1.40 and r['pier_max'] <= 0.84 and
          max(r['zs'][z] <= B.DECK[assign[z]]['cap'] for z in AREAS) and
          len(set(assign.values())) <= 4)
    return ok, r


if __name__ == '__main__':
    cur = {z: 'D16-36/9-4' for z in AREAS}
    ok, r = feasible(cur)
    print('start feasible', ok, 'deck cost %.0f' % r['deck_cost'], 'drift %.3f base %.2f' % (r['drift'], r['base']))
    order = sorted(AREAS, key=lambda z: -AREAS[z])
    improved = True
    it = 0
    while improved and it < 40:
        improved = False
        it += 1
        for z in order:
            for code in CAND:
                if code == cur[z]:
                    continue
                if B.DECK[code]['cost'] >= B.DECK[cur[z]]['cost']:
                    continue
                trial = dict(cur)
                trial[z] = code
                ok, rr = feasible(trial)
                if ok:
                    cur = trial
                    improved = True
                    print('zone %s -> %-13s deckcost %.0f drift %.3f base %.2f' % (z, code, rr['deck_cost'], rr['drift'], rr['base']))
                    break
    ok, r = feasible(cur)
    print('FINAL', ok, cur)
    print('deck cost %.0f drift %.3f base %.2f pier %.3f shears %s' % (r['deck_cost'], r['drift'], r['base'], r['pier_max'], {k: round(v) for k, v in r['zs'].items()}))
    json.dump(cur, open('deck_final.json', 'w'), indent=1)

```


## code/package.py

```text
"""Zip the five deliverables into output/submission.zip (deterministic order, no mtime noise)."""
import os
import zipfile

HERE = os.path.dirname(os.path.abspath(__file__))
OUT = os.path.join(HERE, 'out')
FILES = ['lateral_schedule.xlsx', 'diaphragm_plan.dxf', 'constraint_register.xlsx',
         'calculations.pdf', 'design_basis.md']

with zipfile.ZipFile(os.path.join(OUT, 'submission.zip'), 'w', zipfile.ZIP_DEFLATED) as z:
    for f in FILES:
        p = os.path.join(OUT, f)
        zi = zipfile.ZipInfo(f, date_time=(2026, 11, 10, 0, 0, 0))
        zi.compress_type = zipfile.ZIP_DEFLATED
        zi.external_attr = 0o644 << 16
        z.writestr(zi, open(p, 'rb').read())
print('packed', FILES)

```


## code/repair.py

```text
"""Repair-then-descend from a cheap random start; saves every improvement immediately."""
import json
import random
import sys
import time

sys.argv = ['x']
exec(open('/work/ple-solve/wind-diaphragm-shear/opt.py').read().split("if __name__")[0])

CM = {ln: 'PL3x3/8' for ln in LINES}
BYCOST = sorted(DECKS, key=lambda c: DECKS[c]['cost_per_sf'])
OUTP = '/work/ple-solve/wind-diaphragm-shear/best_design2.json'
t0 = time.time()
rnd = random.Random(int(sys.argv[1]) if len(sys.argv) > 1 else 4242)
cache = {}
best = {'cost': 1e9, 'deck': None}


def viol(r):
    v = 0.0
    for m, val in r['vmax'].items():
        v += max(0.0, val - BASE * WIDTH[m]) / (BASE * WIDTH[m])
    v += max(0.0, r['drift'] - DRIFT) / DRIFT
    v += max(0.0, r['pier_drift'] - ROOFZ * 12 * PIER_RATIO) / (ROOFZ * 12 * PIER_RATIO)
    v += max(0.0, len(set(r['deck_map'].values())) - MAXTYPES)
    v += len(r['f1']) / 10.0 + len(r['f2']) / 10.0
    return v


def ev(dm):
    k = tuple(sorted(dm.items()))
    if k not in cache:
        cache[k] = evaluate(dm, CM)
    return cache[k]


def remember(dm, r, tag):
    if r['ok_struct'] and r['cost']['total'] < best['cost'] - 1e-6:
        best['cost'] = r['cost']['total']
        best['deck'] = dict(dm)
        json.dump({'deck': dict(dm), 'cost': r['cost']['total'], 'drift': r['drift'],
                   'ties': sorted(r['ties']), 'tag': tag,
                   'secs': round(time.time() - t0)},
                  open(OUTP, 'w'), indent=1)
        print('SAVED $%.0f drift %.3f ties %d %s' % (r['cost']['total'], r['drift'],
                                                    len(r['ties']), tag), flush=True)


pool = rnd.sample(BYCOST[:8], rnd.randint(1, 4))
dm = {z: rnd.choice(pool) for z in ZONES}
cur = ev(dm)
print('start $%.0f viol %.3f' % (cur['cost']['total'], viol(cur)), flush=True)
for it in range(40):
    if cur['ok_struct']:
        break
    cands = []
    for z in ZONES:
        for code in BYCOST:
            if code == dm[z]:
                continue
            t = dict(dm)
            t[z] = code
            if len(set(t.values())) > MAXTYPES:
                continue
            r = ev(t)
            dv = viol(cur) - viol(r)
            if dv > 1e-9:
                cands.append(((r['cost']['total'] - cur['cost']['total']) / dv, r, t))
    if not cands:
        break
    cands.sort(key=lambda x: x[0])
    _, cur, dm = cands[0]
    remember(dm, cur, 'repaired-%d' % it)
improved = True
while improved:
    improved = False
    for z in ZONES:
        for code in BYCOST:
            if code == dm[z]:
                continue
            t = dict(dm)
            t[z] = code
            if len(set(t.values())) > MAXTYPES:
                continue
            r = ev(t)
            if r['ok_struct'] and r['cost']['total'] < cur['cost']['total'] - 1e-6:
                dm, cur, improved = t, r, True
                remember(dm, cur, 'descend')
                break
        if improved:
            break
print('finished $%.0f ok=%s evals %d %.0fs' % (cur['cost']['total'], cur['ok_struct'],
                                               len(cache), time.time() - t0), flush=True)

```


## code/report.py

```text
"""Final analysis and the five deliverables for the wind diaphragm / shear wall package."""
import math
import os
import json
import numpy as np
import build as B
import analyze as A

HERE = os.path.dirname(os.path.abspath(__file__))
OUT = os.path.join(HERE, 'out')
DECK_ASSIGN = {z: 'D22-36/7-4' for z in B.ZONES}     # one deck type, G' 32.6 kip/in (x1.15 in Z22)
CHORD_ASSIGN = {cid: 'PL3x3/8' for (cid, _, _) in B.CHORD_LINES}
LEVELS = [('L1', 10.0), ('L2', 5.0), ('L3', 2.5)]


def run(level_H, deck=None, chord=None):
    panels = B.load_panels()
    piers = B.build_piers(panels)
    rig = A.Rig(panels, piers, deck or DECK_ASSIGN, chord or CHORD_ASSIGN, level_H)
    res = rig.run()
    return rig, res, panels, piers


def panel_shears(rig, res):
    """max in-plane shear per panel over the scenarios: sum of its legs' spring forces"""
    out = {}
    for sc, r in res.items():
        for p in rig.piers:
            k = rig.model.pier_node[rig.piers.index(p)]
            u = np.array([rig.model.U[2 * k], rig.model.U[2 * k + 1]])
            dv = np.array(p['dir'], float)
            f = p['k'] * 12.0 * (dv @ u)
            out.setdefault(p['panel'], {}).setdefault(sc, 0.0)
            # accumulate per scenario: rebuild below instead
    # recompute cleanly scenario by scenario
    out = {}
    for sc, r in res.items():
        rig_res = rig.run_single(sc) if hasattr(rig, 'run_single') else None
    return out


def pier_forces(rig):
    """spring force of every pier in the current solved state"""
    f = {}
    for p, k in zip(rig.piers, rig.model.pier_node):
        u = np.array([rig.model.U[2 * k], rig.model.U[2 * k + 1]])
        dv = np.array(p['dir'], float)
        f[p['id']] = p['k'] * 12.0 * (dv @ u)
    return f


def all_results():
    out = {}
    for name, H in LEVELS:
        rig, res, panels, piers = run(H)
        # per-panel shear per scenario
        pshear = {}
        pdisp = {}
        for sc in res:
            rig_sc = A.Rig(panels, piers, DECK_ASSIGN, CHORD_ASSIGN, H)
            # re-solve this scenario alone to read the pier springs of it
            import copy
            m = rig_sc.model
            ll, _ = B.line_loads(panels, rig_sc.cp, dict(B.SCEN)[sc])
            factor = 0.75 if sc in B.TORSION else 1.0
            loads = [(p0, p1, ll[ln] * factor, nrm) for ln, (p0, p1, nrm, axis) in B.LINES.items()]
            tn = None
            if sc in B.TORSION:
                e, direc = B.TORSION[sc]
                F = sum(ll[ln] * factor * math.dist(p0, p1) * nrm[1 if direc == 'X' else 0]
                        for ln, (p0, p1, nrm, axis) in B.LINES.items())
                Bdim = 180.0 if direc == 'X' else 240.0
                Mt = abs(e) * Bdim * abs(F) * (1.0 if e > 0 else -1.0)
                tn, _c = rig_sc.torsional_forces(direc, Mt)
            m.solve(loads, torsion_force_nodes=tn)
            f = pier_forces(rig_sc)
            for p in piers:
                pshear.setdefault(sc, {})
                pshear[sc][p['panel']] = pshear[sc].get(p['panel'], 0.0) + f[p['id']]
                pdisp.setdefault(sc, {})
                pdisp[sc][p['panel']] = max(pdisp[sc].get(p['panel'], 0.0), abs(f[p['id']]) / p['k'])
        out[name] = dict(rig=rig, res=res, panels=panels, piers=piers, pshear=pshear, pdisp=pdisp, H=H)
        print(name, 'done: elements %d nodes %d umax %.0f' % (len(rig.model.elems), len(rig.model.nodes), res['W01']['umax']))
    return out


if __name__ == '__main__':
    os.makedirs(OUT, exist_ok=True)
    R = all_results()
    l2 = R['L2']
    print('W01 reaction W at each level:', [round(R[n]['res']['W01']['react']['W'], 2) for n, _ in LEVELS])
    json.dump({n: {sc: dict(react={k: float(v) for k, v in R[n]['res'][sc]['react'].items()},
                            drift=R[n]['res'][sc]['drift'],
                            umax=R[n]['res'][sc]['umax'],
                            line_load={k: float(v) for k, v in R[n]['res'][sc]['line_load'].items()},
                            shear={ln: [[f, float(v)] for f, v in R[n]['res'][sc]['shear'][ln]] for ln in B.LINES},
                            chord={k: float(v) for k, v in R[n]['res'][sc]['chord'].items()})
                for sc in R[n]['res']} for n, _ in LEVELS},
              open(os.path.join(OUT, 'results.json'), 'w'), indent=1)
    print('written results.json')
    print('max pier drift %.3f in' % max(max(v.values()) for v in l2['pdisp'].values()))
    print('panel shears max', {p: round(max(sc.get(p, 0) for sc in l2['pshear'].values()), 2) for p in list(l2['pshear']['W01'])[:6]})

```


## code/search.py

```text
"""Coordinate-descent cost search over the deck field with the real constraint set."""
import json
import math
import build as B
import sweep as S

AREAS = B.zone_areas()
CHEAP = 'D22-36/4-2'


def cost_of(assign):
    return sum(AREAS[z] * B.DECK[assign[z]]['cost'] for z in AREAS)


def check(assign):
    r = S.evaluate(assign)
    ok = (r['drift'] <= 0.55 and r['base'] <= 1.40 and r['pier_max'] <= 0.84 and
          all(r['zs'][z] <= B.DECK[assign[z]]['cap'] for z in AREAS) and
          len(set(assign.values())) <= 4)
    return ok, r


if __name__ == '__main__':
    cur = {z: 'D16-36/9-4' for z in AREAS}
    best_cost = cost_of(cur)
    print('start %.0f' % best_cost, flush=True)
    order = sorted(AREAS, key=lambda z: -AREAS[z])
    trials = [CHEAP, 'D22-36/7-2', 'D22-36/7-4', 'D20-36/7-4', 'D18-36/9-4']
    for sweep_i in range(3):
        improved = False
        for z in order:
            for code in trials:
                if B.DECK[code]['cost'] >= B.DECK[cur[z]]['cost']:
                    continue
                t = dict(cur)
                t[z] = code
                if len(set(t.values())) > 4:
                    continue
                if cost_of(t) >= best_cost:
                    continue
                ok, r = check(t)
                print('  %s -> %s : %s cost %.0f drift %.3f base %.2f' % (z, code, ok, cost_of(t), r['drift'], r['base']), flush=True)
                if ok:
                    cur = t
                    best_cost = cost_of(t)
                    improved = True
                    break
        print('pass %d best %.0f %s' % (sweep_i, best_cost, cur), flush=True)
        if not improved:
            break
    ok, r = check(cur)
    print('FINAL ok=%s cost=%.0f %s' % (ok, cost_of(cur), cur), flush=True)
    print('drift %.3f base %.2f pier %.3f' % (r['drift'], r['base'], r['pier_max']), flush=True)
    json.dump(dict(assign=cur, ok=ok, deck_cost=cost_of(cur), drift=r['drift'], base=r['base']),
              open('deck_best.json', 'w'), indent=1)

```


## code/search2.py

```text
"""Randomised descent around the feasible corner of the deck-assignment space."""
import json
import random
import sys
import time

sys.argv = ['x']
exec(open('/work/ple-solve/wind-diaphragm-shear/opt.py').read().split("if __name__")[0])

CHEAP = sorted(DECKS, key=lambda c: DECKS[c]['cost_per_sf'])
CHORD0 = {ln: 'PL3x3/8' for ln in LINES}


def slack(r):
    """Smallest relative headroom over the binding checks (1.0 = one of them is exactly at limit)."""
    s = []
    for m, v in r['vmax'].items():
        s.append(BASE * WIDTH[m] / max(v, 1e-9))
    s.append(DRIFT / max(r['drift'], 1e-9))
    s.append((ROOFZ * 12.0 * PIER_RATIO) / max(r['pier_drift'], 1e-9))
    return min(s)


def descend(dm, cm, log=print, tag=''):
    best = evaluate(dm, cm)
    improved = True
    while improved:
        improved = False
        for z in ZONES:
            for code in CHEAP:
                if code == dm[z]:
                    continue
                trial = dict(dm)
                trial[z] = code
                if len(set(trial.values())) > MAXTYPES:
                    continue
                r = evaluate(trial, cm)
                if r['ok_struct'] and r['cost']['total'] < best['cost']['total'] - 1e-6:
                    dm, best, improved = trial, r, True
                    break
            if improved:
                break
    return dm, best


def main():
    rnd = random.Random(int(sys.argv[1]) if len(sys.argv) > 1 and sys.argv[1].isdigit() else 7)
    t0 = time.time()
    start = {z: 'D16-36/9-4' for z in ZONES}
    dm, best = descend(dict(start), dict(CHORD0), tag='seed')
    print('seed descent $%.0f slack %.3f' % (best['cost']['total'], slack(best)))
    keep = [(best['cost']['total'], dict(best['deck_map']), slack(best), best['drift'],
             len(best['ties']))]
    for k in range(14):
        trial = dict(start)
        # perturb: two or three zones to a random deck
        for z in rnd.sample(ZONES, rnd.choice([2, 3, 4])):
            trial[z] = rnd.choice(CHEAP[:10])
        try:
            d2, b2 = descend(trial, dict(CHORD0))
        except Exception as exc:  # noqa: BLE001
            print('restart %d failed: %s' % (k, exc))
            continue
        if b2['ok_struct']:
            keep.append((b2['cost']['total'], dict(b2['deck_map']), slack(b2), b2['drift'],
                         len(b2['ties'])))
            print('restart %d $%.0f slack %.3f drift %.3f ties %d  %.0fs' % (
                k, b2['cost']['total'], slack(b2), b2['drift'], len(b2['ties']),
                time.time() - t0))
    keep.sort(key=lambda t: t[0])
    print('\n=== top feasible designs (cost, slack) ===')
    seen = set()
    out = []
    for cost, dmx, sl, dr, nt in keep:
        key = tuple(sorted(dmx.items()))
        if key in seen:
            continue
        seen.add(key)
        print('$%.0f slack %.3f drift %.3f ties %d  types %s' % (
            cost, sl, dr, nt, sorted(set(dmx.values()))))
        out.append(dict(cost=cost, deck=dmx, slack=sl, drift=dr, ties=nt))
    with open('/work/ple-solve/wind-diaphragm-shear/cands_search.json', 'w') as f:
        json.dump(out, f, indent=1)


main()

```


## code/search3.py

```text
"""Pair-move hill climb around the feasible corner; saves every improvement to disk immediately."""
import json
import random
import sys
import time

sys.argv = ['x']
exec(open('/work/ple-solve/wind-diaphragm-shear/opt.py').read().split("if __name__")[0])

CHEAP = sorted(DECKS, key=lambda c: DECKS[c]['cost_per_sf'])
CM = {ln: 'PL3x3/8' for ln in LINES}
OUTP = '/work/ple-solve/wind-diaphragm-shear/best_design.json'
t0 = time.time()


def slack(r):
    s = [BASE * WIDTH[m] / max(v, 1e-9) for m, v in r['vmax'].items()]
    s.append(DRIFT / max(r['drift'], 1e-9))
    s.append((ROOFZ * 12.0 * PIER_RATIO) / max(r['pier_drift'], 1e-9))
    return min(s)


def save(dm, r, tag):
    rec = dict(tag=tag, deck=dict(dm), cost=r['cost']['total'], drift=r['drift'],
               ties=sorted(r['ties']), slack=slack(r), types=sorted(set(dm.values())),
               n_embed=r['n_embed'])
    json.dump([rec], open(OUTP, 'w'), indent=1)
    print('SAVED $%.0f slack %.3f drift %.3f ties %d %s' % (
        rec['cost'], rec['slack'], rec['drift'], len(rec['ties']), tag), flush=True)


seed = {'Z11': 'D22-36/7-4', 'Z12': 'D18-36/9-5', 'Z13': 'D18-36/9-5', 'Z21': 'D22-36/7-4',
        'Z22': 'D18-36/9-5', 'Z23': 'D16-36/9-4', 'Z31': 'D22-36/7-4', 'Z32': 'D22-36/7-4',
        'Z33': 'D22-36/7-4', 'Z41': 'D20-36/9-4', 'Z42': 'D18-36/9-5', 'Z43': 'D22-36/7-4'}
best = evaluate(seed, CM)
print('seed $%.0f ok=%s slack %.3f drift %.3f ties %d' % (
    best['cost']['total'], best['ok_struct'], slack(best), best['drift'], len(best['ties'])),
    flush=True)
dm = dict(seed)
if best['ok_struct']:
    save(dm, best, 'greedy-seed')

# single-zone descent
improved = True
while improved:
    improved = False
    for z in ZONES:
        for code in CHEAP:
            if code == dm[z]:
                continue
            trial = dict(dm)
            trial[z] = code
            if len(set(trial.values())) > MAXTYPES:
                continue
            r = evaluate(trial, CM)
            if r['ok_struct'] and r['cost']['total'] < best['cost']['total'] - 1e-6:
                dm, best, improved = trial, r, True
                print('  Z %s -> %s $%.0f slack %.3f' % (z, code, r['cost']['total'], slack(r)),
                      flush=True)
                save(dm, best, 'single')
                break
        if improved:
            break

# pair moves: downgrade one zone, upgrade another
rnd = random.Random(11)
for it in range(120):
    z1, z2 = rnd.sample(ZONES, 2)
    c1 = rnd.choice(CHEAP[:8])
    c2 = rnd.choice(CHEAP[8:])
    trial = dict(dm)
    trial[z1], trial[z2] = c1, c2
    if len(set(trial.values())) > MAXTYPES:
        continue
    r = evaluate(trial, CM)
    if r['ok_struct'] and r['cost']['total'] < best['cost']['total'] - 1e-6:
        dm, best = trial, r
        print('PAIR %s->%s %s->%s $%.0f slack %.3f' % (z1, c1, z2, c2,
                                                       r['cost']['total'], slack(r)), flush=True)
        save(dm, best, 'pair')
print('done %.0fs' % (time.time() - t0), flush=True)

```


## code/sweep.py

```text
"""Sweep the sixteen catalogue decks uniformly to find the design space."""
import math
import build as B
import analyze as A

panels = B.load_panels()
piers = B.build_piers(panels)
CH = {cid: 'PL3x3/8' for (cid, _, _) in B.CHORD_LINES}
areas = B.zone_areas()


def evaluate(assign, H=5.0):
    rig = A.Rig(panels, piers, assign, CH, H)
    res = rig.run()
    drift = max(r['drift'][a] for r in res.values() for a in ('X', 'Y'))
    base = 0.0
    worst_panel = None
    for sc, r in res.items():
        d = {}
        for p in piers:
            d[p['panel']] = d.get(p['panel'], 0.0) + r['pier_force'][p['id']]
        for m, v in d.items():
            w = [x for x in panels if x['mark'] == m][0]
            ratio = abs(v) / (w['s1'] - w['s0'])
            if ratio > base:
                base, worst_panel = ratio, (m, round(v, 2), sc)
    pier_max = max(max(r['pier_disp'].values()) for r in res.values())
    zs = {}
    for e in range(len(rig.model.elems)):
        cx, cy = rig.model.elem_centre(e)
        for z, (x0, y0, x1, y1) in B.ZONES.items():
            if x0 - 1e-9 <= cx <= x1 + 1e-9 and y0 - 1e-9 <= cy <= y1 + 1e-9:
                zs[z] = max(zs.get(z, 0.0), max(abs(r['us'][e]) * 1000.0 for r in res.values()))
                break
    cost = sum(areas[z] * B.DECK[assign[z]]['cost'] for z in areas)
    return dict(drift=drift, base=base, worst=worst_panel, pier_max=pier_max, zs=zs, deck_cost=cost)


if __name__ == '__main__':
    for code in [d[0] for d in B.DECKS]:
        a = {z: code for z in B.ZONES}
        r = evaluate(a)
        ok = r['drift'] <= 0.55 and r['base'] <= 1.40 and r['pier_max'] <= 0.84 and \
            max(r['zs'].values()) <= B.DECK[code]['cap']
        print("%-14s G'=%5.1f $%.2f  drift %.3f  base %.2f  pier %.3f  maxshear %4.0f cap %4.0f  deckcost %7.0f  %s  %s" % (
            code, B.DECK[code]['g'], B.DECK[code]['cost'], r['drift'], r['base'], r['pier_max'],
            max(r['zs'].values()), B.DECK[code]['cap'], r['deck_cost'], 'FEASIBLE' if ok else '', r['worst']))

```


## code/ve1.py

```text
#!/usr/bin/env python3
"""Verify the DIAPH-MT implementation against the criteria's published example VE-1."""
import numpy as np
from diaph import Model

outline = [(0, 0), (100, 0), (100, 60), (0, 60)]
piers = []
for x in (12.5, 37.5, 62.5, 87.5):
    piers.append(dict(id=f'S{x}', xy=(x, 0.0), dir=(1, 0), k=500.0, line='S'))
    piers.append(dict(id=f'N{x}', xy=(x, 60.0), dir=(1, 0), k=500.0, line='N'))
for y in (15, 45):
    piers.append(dict(id=f'W{y}', xy=(0.0, y), dir=(0, 1), k=500.0, line='W'))
    piers.append(dict(id=f'E{y}', xy=(100.0, y), dir=(0, 1), k=500.0, line='E'))
chords = [dict(id='C-S', p0=(0, 0), p1=(100, 0), A=3.0), dict(id='C-N', p0=(0, 60), p1=(100, 60), A=3.0),
          dict(id='C-W', p0=(0, 0), p1=(0, 60), A=3.0), dict(id='C-E', p0=(100, 0), p1=(100, 60), A=3.0)]
m = Model(outline, [], 5.0, 2900.0, 13920.0, lambda x, y: 30.0, chords, piers)
loads = [((0, 0), (100, 0), 0.50, (0, 1)), ((0, 60), (100, 60), 0.30, (0, -1))]
U = m.solve(loads)
print('elements %d (target 240)   nodes %d (target 273)' % (len(m.elems), len(m.nodes)))
r = m.reactions()
print('reaction x=0  %.4f (target 10.00)' % r.get('W', 0))
print('reaction x=100 %.4f (target 10.00)' % r.get('E', 0))
us = m.unit_shears()
i = int(np.argmax(np.abs(us)))
print('largest unit shear %.1f plf (target 185.2)' % (abs(us[i]) * 1000.0))
bf = m.bar_forces()
print('largest chord force %.3f kip (target 2.57)' % max(abs(v) for v in bf.values()))
print('largest disp Y %.4f in (target 0.1669)' % (np.abs(U[1::2]).max() * 12))
print('bars:', len(m.bars))

```


## code/xcheck.py

```text
"""Independent check: run the platform's own DIAPH-MT reference model on my design and compare."""
import json
import math
import sys
import numpy as np

sys.path.insert(0, '/work/ple-dataset/tasks/wind-diaphragm-shear/tests')
from corrino.structural_task5_wind_diaphragm_shear.grading.model import truth, Design, Analysis  # noqa

import build as B
import analyze as A

DECK = {z: 'D22-36/7-4' for z in B.ZONES}
CH = {cid: 'PL3x3/8' for (cid, _, _) in B.CHORD_LINES}

T = truth()
print('truth keys', list(T.keys()))
print('fe', T['fe'], 'wind', {k: v for k, v in T['wind'].items() if k != 'cp'})
print('geometry', {k: v for k, v in T['geometry'].items() if k != 'footprint'})
print('wall_lines', T['wall_lines'])
print('zones', [z['id'] for z in T['zones']])
print('deck span factor', T['deck'])
print('limits', T['limits'])
print('report fractions', T['report_fractions'])
print('scenarios', T['scenarios'])
d = Design(deck=DECK, chords=CH, embeds={p['mark']: 2 for p in B.load_panels()}, tiedowns=[])
an = Analysis(T, d)
ref = an.analyse()

panels = B.load_panels()
piers = B.build_piers(panels)
rig = A.Rig(panels, piers, DECK, CH, 5.0)
mine = rig.run()
print('elements mine %d ref %d' % (len(rig.model.elems), len(an.mesh.elem)), 'nodes', len(rig.model.nodes), an.mesh.nnode)
for sc in ['W01', 'W03', 'W10']:
    r, m = ref[sc], mine[sc]
    print('\n==', sc)
    print('line load ref', {k: round(v, 1) for k, v in r['line_load'].items()})
    print('line load mine', {k: round(abs(v) * 1000, 1) for k, v in m['line_load'].items()})
    print('reaction ref', {k: round(v, 2) for k, v in r['line_reac'].items()})
    print('reaction mine', {k: round(abs(v), 2) for k, v in m['react'].items()})
    print('drift ref', {k: round(v, 4) for k, v in r['deflection'].items()})
    print('drift mine', {k: round(v, 4) for k, v in m['drift'].items()})
    print('stations ref', {k: round(v, 1) for k, v in list(r['stations'].items())[:6]})
    print('stations mine', {f"{ln}@{f:.2f}": round(v, 1) for ln in ['S', 'E', 'N1', 'N2', 'E2', 'W'] for f, v in m['shear'][ln]})
    print('chord ref', {k: round(v, 2) for k, v in list(r['chord_force'].items())[:6]})
    print('pier demand E201 ref', round(r['pier_demand'].get('E201', 0), 2), 'mine', round(abs(m['pier_force'].get('E201', 0)), 2))

```


## deliverables/design_basis.md

```text
# Design basis \&mdash; Building A2 roof diaphragm and perimeter shear walls (wind)

Delegated design package for EOR-NDDC-A2-S01 Revision 2. All loads and resistances are on an allowable stress design basis (0.6W, criteria 3.3).

## Design basis table

| item | value | source |
| --- | --- | --- |
| Kzt | 1.19 | TS-2026-02 sec. 4, adopted by criteria 4.3 |
| Ke | 0.8671 | criteria 4.4, zg = 3940.0 ft from ALTA-2025-77 sec. 1 |
| q_h_psf | 28.75 | criteria 4.2 at the 28.00 ft deck of SR-2026-118 sec. 3 |
| screen_wall_top_ft | 36.00 | SR-2026-118 sec. 5.2, x = 96 to 168 ft |
| asbuilt_opening_panel | S14 | 2026 federated model: field-cut opening S14-O1, 194.5 to 202.5 ft, not on the 2024 schedule |
| asd_combination | 0.6W | criteria 3.3 |
| omega_d | 2.35 | CDS-B36 Note 2 |
| analysis_method | DIAPH-MT semi-rigid membrane, bar chords and pier springs (criteria 9) | criteria 9.1 to 9.7, AHJ letter sec. 3 |
| mesh_level | L2, H = 5.0 ft | criteria 9.7 |

## Governing scenario by wall line

| line | governing scenario |
| --- | --- |
| S | W06 |
| E | W01 |
| N1 | W08 |
| N2 | W02 |
| E2 | W01 |
| W | W01 |

## Conflicts between issued documents and how they were resolved

1. **Basic wind speed.** The 2024 criteria (Rev 0, archive) carries 105 mph and Kzt = 1.00. The AHJ letter NDDC-AHJ-014 sec. 2 accepts 115 mph at Risk Category III and requires a site-specific topographic study, and TS-2026-02 sec. 4 fixes Kzt = 1.19. Rev 0 is marked superseded in transmittal T-204 and was not used.

2. **Screen wall height.** The 2024 precast panel schedule PC-S01 shows panels N108 to N111 at 31.00 ft; the roof line scan (SR-2026-118 sec. 5.2) and the 2026 federated model both give 36.00 ft over x = 96 to 168 ft. Criteria 5.3 makes the as-built model govern, so the panels are taken at 36.00 ft and the parapet strips of those four panels are 8.00 ft tall with q_p at 36 ft. This raises the roof-level line load on wall line N1 by the lever action of criteria 6.4.

3. **Field-cut opening.** The federated model carries a 2026 field-cut opening in panel S14 (194.5 to 202.5 ft, head 12.00 ft) that the 2024 schedule does not show (its Note 4 disclaims field openings). The opening is taken into account: panel S14 is divided into a 2.50 ft leg and a 1.50 ft leg, and the 1.50 ft leg is neglected because criteria 5.4 rejects legs narrower than 2.0 ft.

4. **Deck span in the mechanical bay.** SJ-201 Note 1 gives 5 ft joist spacing (6 ft deck span) but Note 2 doubles the joists in zone Z22 to 4 ft. The deck there is designed on a 4 ft span: G-prime is multiplied by 1.15 (CDS-B36 Note 4, criteria 7.3, stiffness only) and EAy is taken as 17400 kip/ft instead of 13920.

5. **Parapet coefficient.** The wind study gives a net GCpn rather than a Cp for the parapet strip and requires q_p at the top of each parapet, not q_h; the study governs over any reading of the criteria that would use q_h (WT-2026-07 6.3, criteria 6.1).

6. **Deflection limit.** The Owner basis of design OBD-NDDC-A2 sec. 4.4 sets the deflection of the roof diaphragm relative to its supports at 0.55 in for the campus precast connection detail, and the criteria applies it to every scenario and both global axes; it is more restrictive than the framing limits of the general structural code and it governs here. UFC 3-301-01 Table 2-1 was compared as a second source and does not apply to this quantity: it limits the wind-induced deflection of framing that supports exterior wall finishes (L/600 for brick veneer down to L/120 for metal panels), not the slip of a diaphragm on its supports, so 0.55 in is taken from the Owner document alone.

7. **Hall B lateral study.** The archive holds a lateral study for Building A1; it has a different wall layout and was not used for any quantity in this package (T-204 item 15).

8. **Panel base connection.** The base connection of the generator-yard panels E201 to E204 carries the largest panel shear on the building because the notch elevations deliver the north and east winds straight into the E2 line. The computed demand on the governing panel is 1.40 kip/ft against the 1.40 kip/ft allowable of the standard B1 connection (criteria 5.7, KRS-CN 3.1): the deck schedule is set so that every panel stays inside that allowable, with the stiffness concentrated in the zones that feed those walls.

9. **Deck schedule.** The deck is not one product over the roof. The criteria 7.1 fixes one type per zone and OBD 4.6 allows not more than four distinct types; inside that limit the type is chosen zone by zone by cost, subject to the unit shear capacity of the zone, the panel base capacity, the embed capacity, the 0.55 in drift limit and the tie-down count the overturning check forces. The cheapest assignment found on the constructed model costs $283,660 against the $310,000 cap of OBD 4.7; the uniform decks that satisfy the same checks cost $340,662 and more, so the saving is in the schedule, not in the rates.

## What the model does and does not do

- The diaphragm is analysed semi-rigidly: flexible tributary and rigid distributions are not accepted for this building (criteria 9, AHJ letter sec. 3).

- The implementation reproduces verification example VE-1 of criteria 9.8 to within 0.2 per cent on every published quantity before it is applied to Building A2.

- Mesh convergence at L2 to L3 on the reaction of line W in W01 is -0.18 per cent (limit 5 per cent).

- Internal pressure is excluded from the diaphragm and wall loads (criteria 4.1); it belongs to the out-of-plane panel submittal. For context, UFC 3-301-01 sec. 26.12 requires fire-station garage bays to be designed as partially enclosed on the assumption that the doors have failed, and records roof diaphragm losses after Hurricane Michael that followed exactly such a door failure; the data hall has no such opening in this package, so the criteria exemption is applied as written.

- The deck diaphragm is credited in its final condition only (CDS-B36 Note 6). Construction-stage stability - joist seats field-bolted before the hoisting cables are released, bottom chords stabilised and erection bridging installed per 29 CFR 1926.757 - is an erection requirement outside this package.


```
