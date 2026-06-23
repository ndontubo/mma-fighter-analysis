# DECISIONS — Running Log

Append-only log of choices and their rationale. Newest at top. Mark each as **[Decided]** or **[Proposed]** (not yet committed). When a Proposed item is acted on, update it to Decided with the date.

---

## 2026-06-22

**[Decided] Phase 2 complete — rule-based stance working end-to-end on video.**
Pipeline runs: track → pair → per-frame stance → smooth → aggregate → overlay. Validated on a clean ~28 s segment; output matches known stances (Whittaker orthodox & static, Adesanya switching). Success criterion met.

**[Decided] Tracker = BoT-SORT via Ultralytics `model.track()`.** (Promoted from Proposed.)
Built-in, zero-friction, good enough for clean stand-up segments. `persist=True, stream=True`. Known to fragment IDs through occlusion/cuts — accepted for Phase 2, addressed later by the re-ID stack.

**[Decided] Fighter pairing = hybrid (lock-then-filter).** (Promoted from open question.)
Lock the 2 most-seen track IDs across the clip once, then filter every frame to those IDs. Stable identity (needed for per-fighter aggregation) + clean per-frame filtering + auto-rejects strays (ref, crowd, short blips). Chosen over per-frame selection (no stable identity — wrecks aggregation) and pure whole-clip (no per-frame stray rejection).

**[Decided] Stance rule = ankle-based lead-foot rule.** (Geometric rule, not learned.)
The lead foot (ankle 15/16 projecting further toward the opponent) determines stance: left leads → orthodox, right → southpaw. Visibility-gated with knee/hip fallback; abstains on square stances. This is the method that produced the validated 28 s result and is the **shipped Phase 2 rule.**

**[REJECTED — does not work] Margin-driven multi-joint orientation rule.**
Attempted to improve camera robustness by voting over shoulders + hips + ankles, weighted by geometric separation along a torso→torso forward axis. Evolution and failure:
1. Ankle-only lead-foot rule — worked on clean frames, showed some camera-angle errors. (This is what we reverted to.)
2. Torso-orientation with fixed weights (shoulders/hips > ankles) — failed: side-on fighters have near-zero shoulder/hip width, so those votes are pseudo-label noise; weighting them above ankles amplified the noise.
3. Margin-driven voting (each pair votes ∝ its separation) — looked principled and passed the two sample frames, but **overfit to the clip and did not work on real footage.** Tuning to two frames did not generalize; broadcast side-on angles collapsed the torso signals into noise that margin-normalization inflated into confident-wrong calls, plus excessive `ambiguous`.
**Conclusion:** adding shoulders/hips hurt more than it helped. The extra joints are unreliable exactly when you need them (side-on). Do not re-attempt multi-joint orientation voting for stance. If camera-angle errors must be fixed, the path is depth (3D, deferred to Phase 3+), not more 2D joints.

**[Decided] Visibility threshold = `min_vis=0.5` (confidence), not `2` (flag).**
Root-caused an "everything reads absent" bug: inference returns float confidences (~0.98), label files use integer flags {0,1,2}. `min_vis=2` is never satisfiable by inference. Inference also returns pixel coords, not normalized — never mix the two formats.

**[Decided] Two separate smoothing knobs.**
`smooth_win` (small, ~9 frames) kills single-frame flicker → governs per-frame labels. `min_switch` (~30 frames ≈ 1 s) is how long a new stance must hold to count as a real switch → governs the switch-event count. They optimize against different ground truth and must stay separate. Lower `min_switch` catches brief deliberate switches (fits Adesanya); higher counts only settled stances.

**[Decided] Phase 1 complete — YOLO11n-pose trained.**
Trained on Colab T4 on V2. Val: Box mAP50 ~0.995, Pose mAP50 ~0.993. Nano model is enough; inference runs locally on M2 (MPS/CPU). Copy dataset Drive→Colab-local before training (never read dataset directly from Drive — I/O bottleneck).

**[Proposed → Decided] Inference is local, training is Colab.**
Phase 2 (and all inference) runs on the M2 Mac; no GPU needed for short clips. Colab is only for training (Phase 1 done, Phase 3 strike model upcoming).

---

## Deferred / evaluated-and-parked (with reasoning)

**[Proposed] 3D pose estimation for stance — DEFERRED to Phase 3+.**
Evaluated and parked. Reasons: (a) monocular broadcast footage cannot yield *true* 3D — any 3D is inferred from the same noisy pseudo-label 2D, amplifying noise; (b) true 3D ground truth needs a multi-camera rig or depth sensor, which would be self-captured footage, not the actual fights (domain gap); (c) stance is a coarse binary — the ankle lead-foot rule with square-stance abstaining is a reasonable MVP, and residual camera-angle errors are an accepted limitation (note: adding 2D torso joints to fix them was tried and FAILED — see rejected entry); (d) 3D's real payoff is strike *trajectories* (Phase 3), so build a 3D stack once, there, if at all. If camera-angle stance errors ever must be fixed, depth is the only real lever, not more 2D joints.

**[Proposed] Neurosymbolic framing — NO separate work needed.**
The pipeline is *already* neurosymbolic: YOLO = neural perception, `frame_stance` = symbolic geometric rule. The productive upgrade, if pursued, is a richer *symbolic* layer (HMM/sequence constraints for stance persistence; a facing-validity ruleset), not a learned stance classifier (no stance labels exist, and a binary call doesn't warrant one). Not adding explicit neurosymbolic scaffolding.

**[Proposed] Grid-search tuning of `smooth_win` / `min_switch` — DEFERRED to Phase 4.**
Would optimize knobs against hand-labeled per-frame + switch-event ground truth. Deferred because tuning on a single clip overfits; the honest tuning set is multiple fights, which arrive at Phase 4 alongside the UFC-stats validation. Default knobs are sane for MVP.

**[Proposed] Full-fight tracking robustness (re-ID stack) — DEFERRED to video/aggregation stage.**
Long clips fragment IDs. The real fix is multi-part: shot-boundary detection (split at camera cuts, track within shots), OSNet-style appearance re-ID across occlusion, trunk/shorts-color identity anchoring, and skipping out-of-scope clinch/ground gaps with re-acquisition. This is Phase-4-sized infrastructure; not needed to prove stance on clean segments.

---

## Earlier decisions (pre-2026-06-22)

**[Decided] "Clinch/grappling excluded" is a scope decision, NOT an occlusion-handling strategy.**
Scope: clinch/ground dropped as analysis targets (dataset omits them). Tracking: the 20 fights still contain clinch/occlusion in footage — the tracker must survive or cleanly re-acquire IDs. Occlusion risk is broader than clinch (overlapping exchanges, ref crossings, camera cuts/replays). Dropping clinch as a class does not remove the ID-persistence problem.

**[Decided] Dataset = Mendeley `c456bnk8bm`, use V2.** Strict superset of V1 (bbox + COCO-17 pose).

**[Decided] Build as a Claude Project with persistent knowledge files.** SPEC / DECISIONS / STATE as living memory.

**[Decided] MVP scope = Stages 1–2 (detection + pose).** Only part the dataset directly supports.

**[Decided] Stages 3–5 require external video + self-labeling.** No temporal/strike/stance/identity labels in the dataset.

**[Decided] Stance = geometric rule over pose, not a learned classifier.** (See 2026-06-22 for the rule's final form.)

**[Decided] Detector + pose model = YOLO11 pose (Ultralytics).** (Confirmed by Phase 1 results.)

**[Proposed] Strike recognition = skeleton-based temporal model (ST-GCN / PoseConv3D).** Revisit at Phase 3.

**[Proposed] MVP strike classes = jab, cross, hook, leg_kick, head_kick, idle.**

---

## Open questions (not yet decided)
- Annotation tool: CVAT vs Label Studio for temporal strike spans.
- Fighter identity (A vs B): trunk/shorts color anchoring is the leading approach.
- Skill-tree representation: weighted tree vs graph; visualization lib (NetworkX / D3 / React).
