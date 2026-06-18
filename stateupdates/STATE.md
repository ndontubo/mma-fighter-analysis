# STATE — Current Status

Last updated: 2026-06-18
Current phase: **Phase 1 — Detection + Pose MVP (not yet started)**

## Done
- Defined goal, 5-stage architecture, and success criteria (see SPEC.md).
- Identified and reviewed dataset (Mendeley `c456bnk8bm`); chose V2.
- Confirmed dataset covers Stages 1–2 only; documented the gaps.

## In progress
- Nothing actively building yet.

## Next (immediate)
1. Download V2 and verify actual file structure + YOLO-pose label format on a few samples.
2. Confirm `data.yaml` paths and class config.
3. Train a baseline YOLOv11-pose model on the V2 train split.
4. Run it on a sample clip with overlaid skeletons; eyeball quality.

## Blockers
- Training/compute environment not yet specified (GPU type/limits unknown).

## Open questions
- Tracker + re-ID choice (deferred to video stage).
- Annotation tooling for strike labeling.
- Fighter identity resolution method.
- Source video acquisition for the 20 fights (for Stages 3–5).

## Notes
- Keypoints in V2 are pseudo-labels (YOLOv11x-pose generated) — treat pose accuracy as approximate, especially under occlusion.
- At end of each working session: ask Claude to output updated STATE.md and DECISIONS.md, then re-upload to the Project.
