# STATE — Current Status

Last updated: 2026-06-22
Current phase: **Phase 2 — Rule-based stance on video — COMPLETE.** Next: Phase 3 (strike recognition).

## Done
- Defined goal, 5-stage architecture, success criteria (SPEC.md).
- Dataset chosen and verified: Mendeley `c456bnk8bm` V2 (3,635 train / 980 val / 491 test, COCO-17 YOLO-pose).
- **Phase 1 — Detection + Pose:** trained YOLO11n-pose on Colab (T4) on V2. Val metrics strong (Box mAP50 ~0.995, Pose mAP50 ~0.993). Weights in Google Drive. Brought weights local for inference.
- **Phase 2 — Stance on video, end-to-end:**
  - Tracking via Ultralytics `model.track()` + BoT-SORT → per-frame `(track_id, bbox, kpts[17,3])`.
  - Hybrid fighter-pairing: lock the 2 most-seen track IDs once, filter every frame to those IDs.
  - Per-frame stance rule + temporal smoothing (`smooth_win`) + switch confirmation (`min_switch`).
  - Aggregation: per-fighter stance distribution, confirmed switch count, switch timeline.
  - Video overlay (skeleton + live stance label) for visual verification.
  - **Validated** on a clean ~28 s stand-up segment (ankle-based rule, post `min_vis` fix): Fighter 1 (Whittaker) 92% orthodox / 0 switches; Fighter 2 (Adesanya) 70/30 with 5 confirmed switches (at `smooth_win=11, min_switch=60`). Matches known real-world stances — free sanity check passed.

## Resolved this session
- **Margin-driven orientation rule (shoulders + hips + ankles) FAILED and was reverted.** It was built to address camera-angle errors but **overfit to the validation clip** — tuning it to pass the two sample frames did not generalize to real footage. On broadcast video the shoulder/hip signals collapse under side-on angles into pseudo-label noise, producing confident-wrong calls and excessive `ambiguous` output. **Shipped method = the ankle-based lead-foot rule** (the one that produced the validated 28 s result). Do not re-add multi-joint orientation voting.
- Phase 2 is complete on the ankle rule. Current working params: `smooth_win=9, min_switch=30`.

## Next (Phase 3 — strike recognition)
- Source the 20 named fights as continuous video; set up annotation tooling for temporal strike spans.
- Hand-label MVP strike classes (jab, cross, hook, leg_kick, head_kick, idle) on a small set.
- Build skeleton-based temporal model (ST-GCN / PoseConv3D). This is where training returns to Colab.

## Known issues / risks
- **Tracker fragmentation on long clips.** Full/long clips produce heavily fragmented track IDs (one fighter split across several IDs after occlusion/cuts). Clean short stand-up segments work reliably; full-fight robustness is deferred (see DECISIONS — re-ID/shot-detection stack).
- **ID persistence through occlusion** (overlapping exchanges, ref crossings, camera cuts) — the #1 reliability risk for whole-fight processing. Deferred to the video/aggregation stage.
- **Pose pseudo-labels** are approximate; per-frame stance jitter is expected and is why smoothing exists. Depth-ambiguity (toward/away from camera) cannot be resolved by a 2D rule. The ankle rule handles it by abstaining on square/low-separation frames; adding torso joints to "fix" camera angles was tried and made it worse (see Resolved). Residual camera-angle errors are an accepted MVP limitation, revisited only if Phase 3 forces it.

## Open questions
- Annotation tool: CVAT vs Label Studio for temporal strike spans.
- Fighter identity resolution (A vs B): trunk/shorts color is a strong cue (broadcast graphic shows it) — anchor re-ID on color when building full-fight tracking.
- Skill-tree representation + visualization lib (NetworkX / D3 / React).

## Notes
- At end of each working session: output updated SPEC / STATE / DECISIONS and re-upload to the Project.
- Trim to a single continuous stand-up shot (no cuts, both fighters apart) before running the pipeline. Use `run_and_pair` diagnostics (`unique IDs total`, `top tracks by frame count`) to confirm a clip is clean: want ~2 unique IDs and near-100% both-fighter frames.
