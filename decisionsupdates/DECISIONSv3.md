# DECISIONS — Running Log

Append-only log of choices and their rationale. Newest at top. Mark each as **[Decided]** or **[Proposed]** (not yet committed). When a Proposed item is acted on, update it to Decided with the date.

---

## 2026-06-18 (Phase 1 trained)

**[Decided] Compute environment = Google Colab free tier (NVIDIA T4, 15 GB VRAM); M2/MPS for light inference only.**
Resolves the long-standing open blocker. M2 trains via MPS but is slow and hits CPU-fallback ops; T4 is ~5–10× faster for this small job at zero cost. Premium Colab GPUs (L4/A100/H100) rejected as overkill — bottleneck is dataset size, not GPU. Config implications now fixed: `device=0` on Colab (`'mps'` locally), copy dataset to Colab local disk before training (Drive read ~0.1 MB/s was the real bottleneck — not the GPU), write outputs to Drive to survive idle-disconnect.

**[Decided] Baseline model = YOLO11n-pose (nano).** (Promoted from Proposed.)
Trained 50 epochs on V2, Colab T4, ~1.0 h, batch 32, imgsz 640. Results validate the choice: nano is plenty for a single-class detector. Step up to `s`/`m` only if later quality demands it.

**Phase 1 baseline results (val split, 971 imgs / 1938 instances):**
- Box: P 0.998, R 0.999, mAP50 **0.995**, mAP50-95 **0.969** — near-perfect, detection is solved.
- Pose: P 0.989, R 0.989, mAP50 **0.993**, mAP50-95 **0.847**.
- The Box-vs-Pose strict-threshold gap is expected: V2 keypoints are pseudo-labels, so pose accuracy is bounded by label accuracy, not model capacity. Also note pose metrics are measured *against the pseudo-labels*, so 0.847 ≈ "agrees with pseudo-labels," not "matches true human joints." Trust boxes fully; treat exact keypoint positions as approximate, especially under occlusion. Consistent with SPEC §3.
- Weights: `runs/baseline-2/weights/best.pt` (on Drive).

**[Decided] Dataset I/O pattern on Colab = read from local disk, write to Drive.**
Copy V2 from mounted Drive to `/content/` once per session (few min, one-time), train from there. Local disk wipes on disconnect so the copy + `data_fixed.yaml` are re-run each session; that's an accepted cost. Outputs go to Drive so trained models persist.

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

**[Decided] Detector + pose model = YOLOv11 pose (Ultralytics).** (Confirmed after first training run — see 2026-06-18 Phase 1 trained.)
Matches the dataset's native YOLO-pose format; least friction. Baseline metrics confirm it works well.

**[Proposed] Tracker = ByteTrack or BoT-SORT, + OSNet re-ID.**
ID persistence through occlusion (clinch, overlapping exchanges, ref crossing, camera cuts) is the #1 reliability risk. Decide once we move to video.

**[Proposed] Strike recognition = skeleton-based temporal model (ST-GCN / PoseConv3D).**
Operates on pose sequences. Not started; revisit at Phase 3.

**[Proposed] MVP strike classes = jab, cross, hook, leg_kick, head_kick, idle.**
Small enough to label by hand, broad enough to be meaningful.

---

## Open questions (not yet decided)
- Annotation tool: CVAT vs Label Studio for temporal strike spans.
- How to resolve fighter identity (A vs B): shorts color, corner assignment, or one manual label per round.
- Skill-tree representation: weighted tree vs graph; visualization lib (NetworkX / D3 / React).
