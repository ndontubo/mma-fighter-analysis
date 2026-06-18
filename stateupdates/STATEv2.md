# STATE — Current Status

Last updated: 2026-06-18
Current phase: **Phase 1 — Detection + Pose MVP (trained; visual check pending → then Phase 2)**

## Done
- Defined goal, 5-stage architecture, and success criteria (see SPEC.md).
- Identified and reviewed dataset (Mendeley `c456bnk8bm`); chose V2.
- Confirmed dataset covers Stages 1–2 only; documented the gaps.
- Downloaded V2, verified structure + YOLO-pose label format. Counts: train 3635 / valid 980 / test 491 (images:labels 1:1 in every split). ~10 corrupt labels in train auto-skipped (~0.3%, negligible — pseudo-label artifacts).
- Fixed Roboflow `data.yaml` relative-path quirk (`../`) → wrote `data_fixed.yaml` with absolute `path:` root.
- Settled compute environment: Colab free T4 for training, M2/MPS for light inference (see SPEC §6 / DECISIONS).
- **Trained baseline YOLO11n-pose on V2** — 50 epochs, Colab T4, ~1.0 h, batch 32, imgsz 640. Dataset copied to Colab local disk first (Drive read was the bottleneck). Weights + plots saved to Drive (`runs/baseline-2/`).
- **Val metrics:** Box mAP50 **0.995** / mAP50-95 **0.969** (P 0.998, R 0.999); Pose mAP50 **0.993** / mAP50-95 **0.847**.

## In progress
- Final Phase 1 step: visual skeleton check — running `best.pt` on test frames, eyeballing box+pose quality and overlap/occlusion behavior.

## Next (immediate)
1. Eyeball annotated test-frame predictions; confirm skeletons land on correct joints and both fighters are boxed.
2. If visual check passes → mark Phase 1 fully done, begin Phase 2 (rule-based stance from pose keypoints).
3. (Optional) quick cleanup pass on the ~10 corrupt train labels if anything looks off — low priority.

## Blockers
- None currently. (Compute environment — previously the open blocker — now resolved: Colab free T4.)

## Open questions
- Tracker + re-ID choice (deferred to video stage).
- Annotation tooling for strike labeling.
- Fighter identity resolution method.
- Source video acquisition for the 20 fights (for Stages 3–5).

## Notes
- Keypoints in V2 are pseudo-labels (YOLOv11x-pose generated) — treat pose accuracy as approximate, especially under occlusion.
- At end of each working session: ask Claude to output updated STATE.md and DECISIONS.md, then re-upload to the Project.
