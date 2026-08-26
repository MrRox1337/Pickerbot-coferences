# Gap analysis: two reference papers vs. Picker-Bot

Working notes for the Introduction. Nothing here is applied to `main.tex` — these are candidate
arguments for you to pick from.

**Papers reviewed**

- **[P1]** Y. Luo, "Research on Sorting System of Industrial Robot Based on Machine Vision,"
  *2024 Int. Conf. on Power, Electrical Engineering, Electronics and Control (PEEEC)*,
  2024, pp. 270–274. DOI: 10.1109/PEEEC63877.2024.00055
- **[P2]** P. Chand and S. Lal, "Vision-Based Detection and Classification of Used Electronic
  Parts," *Sensors*, vol. 22, no. 23, p. 9079, Nov. 2022. DOI: 10.3390/s22239079

---

## The headline finding: you sit in a gap neither paper occupies

This is the single most useful framing I found. The two papers fail in *opposite* directions,
and the intersection they leave empty is exactly your contribution.

|                          | Recovery context (used, mixed, degraded parts) | Engineered supply (uniform, singulated) |
| ------------------------ | ---------------------------------------------- | --------------------------------------- |
| **Full robot integration** | ← *empty — this is Picker-Bot*                | **[P1]** Luo 2024                       |
| **Perception only**        | **[P2]** Chand & Lal 2022                      | —                                       |

- **[P1] has the robot but not the problem.** Complete hardware-to-controller integration
  (ABB IRB2400, IRC5 cabinet, TCP/IP socket to an Omron FH-550L), but it sorts *new, uniform
  metal workpieces by size* off a conveyor, under engineered backlighting.
- **[P2] has the problem but not the robot.** Explicitly motivated by educational e-waste and
  the circular economy — the same premise as yours — but the manipulator is never actually
  driven. From its Future Work: *"the pick and place of objects detected via the developed
  object detection algorithm **is being implemented**."* The Niryo Ned appears only in the
  conceptual framework figure.

You are the first of these three to close perception → world coordinates → executed manipulation
*on recovered educational components*. That is a defensible novelty sentence.

---

## [P2] Chand & Lal 2022 — your closest prior work

**Treat this as your primary anchor, not a competitor.** It independently validates your premise,
which makes your problem statement stronger, not weaker. Their intro:

> *"In educational environments where resourcing can be constrained, equipment and consumables
> used in projects can be recycled or reused... After project work, the constructed PCBs are left
> in storage or thrown away. Used components are often discarded instead of being reused."*

That is your Introduction's opening paragraph, written by someone else, in a Q1 journal. Cite it
to establish the problem is real and recognised — *then* show what it leaves open.

### Gap 2.1 — No orientation estimation at all (strongest gap)

I grepped the full text: **zero** occurrences of orientation, rotation, pose, yaw, angle, or
theta. Their detector returns axis-aligned boxes only:

> `BB_i = [ho_i, vo_i, hw_i, vh_i]`  — top-left corner, width, height

A box with no θ cannot tell a gripper how to align its jaws. For capacitors and pots (roughly
radially symmetric) they get away with it; for an Arduino, ESP32 or a 16×2 LCD it fails outright.
This is precisely what your OBB 5-tuple `(cx, cy, w, h, θ)` exists to solve, and your
§V-A argument that a high `mAP@50-95` under *rotated* IoU certifies θ is the direct evidence.

**Why this is a strong gap:** it is not a criticism of their work (their parts didn't need it) —
it is a genuine capability boundary that appears the moment you move to board-level modules.

### Gap 2.2 — They kept the deterministic detector you abandoned

Their pipeline is Canny (thresholds 0.1 / 0.04) → dilate → flood-fill → `regionprops`. They
replaced only the *classifier* (SNN → SVM+PCA → CNN), leaving the fragile stage untouched.

**You have data they don't.** Your §V-C five-scene baseline study measured exactly this stage and
found the usable threshold window collapsing from 130/255 values at one component to 8 at eleven.
Flood-fill also merges touching parts into one region — untested in [P2], whose figures show only
well-separated components. You can cite [P2] as the state of practice and your Table III as the
empirical reason to move the learning *down* into detection rather than bolting it on top.

### Gap 2.3 — Axis-wise scaling, not a projective homography

Their calibration (eqs. 1–2) derives two scalars from four corner markers:

> `x_cal = 194/(0.5(h_TR − h_TL + h_BR − h_BL))`, and similarly for `y_cal`

That is an **axis-wise linear scale factor in mm/pixel** — it corrects nothing for perspective,
lens projection, or a camera that is not perfectly normal to the bench. Your 77-point
`findHomography` (SLS, deliberately not RANSAC) is a strict generalisation of their 4-point model.
Their approach is also pinned to a fixed 0.37 m camera height; your two-point anisotropic
recalibration is a direct answer to a constraint they carry silently.

### Gap 2.4 — Bounded by classifier input resolution

Their own stated limitation: *"The size of the input image to the classifier is currently limited
by the resolution of the camera (960 × 720 pixels)"*, downsampling ROIs to 30 × 30 grayscale.
They note in §4.2 that YOLO-class models *"can detect and classify electronic components with a
single image of the entire workspace"* — i.e. they identify your architecture as the way forward
but do not take it. **You can say you took the step their Future Work points at.**

---

## [P1] Luo 2024 — the industrial-automation contrast

Useful for the "factory automation is the wrong tool" argument you already make in §II-A. It
supplies concrete, citable specifics where your current text is general.

### Gap 1.1 — Engineered lighting as a hard prerequisite

The paper states the dependency plainly:

> *"If the light source is not designed correctly, the image processing algorithm and the
> performance of the image system will be difficult to achieve the desired results."*

It requires a circular LED array plus backlight, chosen so segmentation is trivial. A returned-kit
bench in a teaching lab has none of this. This is the cleanest possible setup for your glare-and-
clutter robustness claim, and it is a *quoted requirement*, not your inference.

### Gap 1.2 — Depth and orientation explicitly discarded

> *"We can ignore the depth information of the object within a certain geometric size of the
> target object, and grasp and sort the target object."*

Legitimate for uniform workpieces on a conveyor. It also means the system never recovers pose —
reinforcing Gap 2.1 as a field-wide gap rather than one paper's omission. Note this connects to
your own §VI-B parallax limitation: you should be careful to claim you *recover θ*, not that you
solved depth, since you also use a planar model.

### Gap 1.3 — Identity comes from barcodes, not appearance

Their intelligent sorting module uses Omron FZ-Panda's *"visual processing processes corresponding
to bar code and two-dimensional code recognition"*, with an SQL lookup deciding the destination bin.

**This is a sharp point.** Their system doesn't recognise *what a component is* — it reads a tag
someone already applied and looks it up. Returned educational components carry no such tag. Recovery
demands recognition from appearance alone, which is a categorically harder problem.

### Gap 1.4 — Destination-driven, not requirement-driven ("push", in your terms)

Sorting is by *"different distribution destinations and package contents"* into
*"subsequent intelligent warehousing"*. There is no demand side: nothing describes what any
downstream consumer actually needs. Your BOQ engine — ignoring correctly-detected but unneeded
parts — has no analogue in either paper. **Neither paper has any concept of a demand specification.**
That makes your "logistics auditor" framing genuinely novel rather than merely a filter.

### Gap 1.5 — No perception metrics whatsoever

Grepped: no mAP, precision, recall, or localisation-error figure anywhere. Table II reports only
sorting *quantity* and *error rate* against manual labour (198/200 at 1.0% for workpiece A, down to
191/200 at 4.5% for C). The error rate more than quadruples across workpiece types with no
diagnosis of whether the fault is perception, grasping, or calibration.

**Handle with care.** This is a real weakness in their evaluation and it flatters your Table II.
But your §V-D concedes you report *functional verification in simulation, not a measured grasp
success rate* — so do not lean so hard on "they didn't measure" that a reviewer turns it around on
you. Best framing: their evaluation is *end-to-end throughput without perception attribution*,
yours is *perception characterised in isolation with manipulation verified functionally*. Different
scopes, both incomplete, honestly stated.

---

## Suggested Introduction structure

Your current Introduction is two paragraphs and goes straight from problem to solution, with all
comparison deferred to §II-A. Inserting a short third paragraph before "This project develops..."
would give the reviewer the gap explicitly.

A possible shape (yours to rewrite in your own voice):

1. **Keep ¶1** — kits returned incomplete, cannot be reissued, cost and e-waste.
2. **New ¶2 — what exists, and where it stops.** Industrial vision-guided sorting is mature but
   assumes an engineered supply: uniform parts, controlled backlighting, and identity supplied by
   barcode rather than appearance [P1]. Vision-based recovery of *used* educational components has
   been demonstrated [P2], but stops at classification — orientation is never recovered and the
   manipulator is left as future work.
3. **New ¶3 — the gap.** No existing system closes the loop from a demand specification, through
   appearance-based recognition of oriented used modules, to executed manipulation.
4. **Keep ¶2 as ¶4** — "This project develops a robotic system that..."

### Ready-to-paste bibliography entries

```latex
\bibitem{luo2024}
Y. Luo, ``Research on sorting system of industrial robot based on machine vision,'' in
\textit{Proc. 2024 Int. Conf. Power, Electrical Engineering, Electronics and Control (PEEEC)},
2024, pp. 270--274.

\bibitem{chand2022}
P. Chand and S. Lal, ``Vision-based detection and classification of used electronic parts,''
\textit{Sensors}, vol. 22, no. 23, p. 9079, Nov. 2022.
```

---

## Two risks worth pre-empting

1. **Platform justification.** [P2] makes a point of its US$3299 Niryo Ned being affordable for an
   education setting. You propose a VT6-A901S: 6-axis, 980 mm reach, industrial. A reviewer may
   ask why a workcell-scale industrial arm is needed for a benchtop task in a university. One
   sentence on institutional scale, existing lab hardware, or throughput would close this off.
   Your §IV-F already notes the system *"can cover any workspace size"* — worth surfacing.

2. **Class count.** [P2] handles 3 classes and flags that others reach 20–22. You also have 3
   (Arduino, ESP32, LCD). Since you are citing a paper that names its own class count as a
   limitation, expect the same question. Your §VI-B limitations paragraph is the natural place to
   own it, framed as dataset scale rather than architectural ceiling — YOLOv8-OBB scales to more
   classes without redesign, which is a genuine advantage over their per-class-retrained CNN.
