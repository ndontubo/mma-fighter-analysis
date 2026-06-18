# DECISIONS — Running Log

Append-only log of choices and their rationale. Newest at top. Mark each as **[Decided]** or **[Proposed]** (not yet committed). When a Proposed item is acted on, update it to Decided with the date.

---

## 2026-06-18 (later)

**[Decided] "Clinch/grappling excluded" is a scope decision, NOT an occlusion-handling strategy.**
Two separate things that must not be conflated:
- *Scope:* clinch and ground grappling are dropped as analysis targets — no strike classes, no pose analysis there. Justified because the dataset omits them entirely (stand-up stills only). Settled, not revisited.
- *Tracking reliability:* the inference videos (the 20 fights) WILL still contain clinch, grappling, and other occlusion moments even though we ignore them as content. The tracker must still survive these or cleanly re-acquire IDs afterward, or fighters A/B get swapped when stand-up resumes.
Occlusion risk is broader than clinch — it also covers overlapping exchanges, one fighter crossing in front of the other, the referee passing between them, and camera cuts/replays. Dropping clinch as a *class* does not remove the ID-persistence problem.

**[Decided] Dataset = Mendeley `c456bnk8bm`, use V2.**
V2 is a strict superset of V1 (bounding boxes + COCO-17 pose). No reason to use V1 alone.

**[Decided] Build as a Claude Project with persistent knowledge files.**
SPEC / DECISIONS / STATE files act as living memory across sessions.

**[Decided] MVP scope = Stages 1–2 only (detection + pose).**
This is the only part the dataset directly supports. Get it finishable and clean before touching temporal work.

**[Decided] Stages 3–5 require external video + self-labeling.**
The dataset has no temporal, strike, stance, or identity labels. Plan for sourcing the 20 named source fights as continuous video and annotating strike spans ourselves.

**[Decided] Stance = geometric rule over pose, not a learned classifier.**
Lead-foot logic on ankle/hip keypoints, smoothed over a window. Don't over-engineer.

**[Proposed] Detector + pose model = YOLOv11 pose (Ultralytics).**
Matches the dataset's native YOLO-pose format; least friction. Confirm after first training run.

**[Proposed] Tracker = ByteTrack or BoT-SORT, + OSNet re-ID.**
ID persistence through occlusion (clinch, overlapping exchanges, ref crossing, camera cuts) is the #1 reliability risk. Decide once we move to video.

**[Proposed] Strike recognition = skeleton-based temporal model (ST-GCN / PoseConv3D).**
Operates on pose sequences. Not started; revisit at Phase 3.

**[Proposed] MVP strike classes = jab, cross, hook, leg_kick, head_kick, idle.**
Small enough to label by hand, broad enough to be meaningful.

---

## Open questions (not yet decided)
- Training/compute environment and GPU limits — needs to be specified.
- Annotation tool: CVAT vs Label Studio for temporal strike spans.
- How to resolve fighter identity (A vs B): shorts color, corner assignment, or one manual label per round.
- Skill-tree representation: weighted tree vs graph; visualization lib (NetworkX / D3 / React).
