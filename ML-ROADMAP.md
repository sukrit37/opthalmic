# SurgiData MVP: ML feasibility and action plan

## Feasibility verdict

The annotation and dataset platform is feasible as a focused MVP. A 50–100 surgery pilot can validate the workflow, annotation protocol, reviewer agreement, and whether AI proposals reduce annotation time. It is not enough to establish reliable generalization across surgeons, sites, microscopes, and acquisition settings, and it is not a credible basis for training detailed anatomical segmentation from scratch.

Keep V1 centered on case management, timestamped surgical-phase annotation, simple instrument-presence intervals, human correction/review, and provenance-preserving JSON/CSV export. Do not make anatomy segmentation, complication detection, skill scoring, outcome prediction, or intraoperative guidance launch requirements.

## What the supplied papers indicate

- **Cataract-1K** (Scientific Data, 2024; [doi:10.1038/s41597-024-03193-4](https://doi.org/10.1038/s41597-024-03193-4)) reports 1,000 videos overall, phase annotations on 56 regular videos across 13 phases, and pixel-level annotations on selected frames from 30 videos.
- **Cataract-LMM** (Scientific Data, 2026; [doi:10.1038/s41597-026-07464-0](https://doi.org/10.1038/s41597-026-07464-0)) reports 3,000 videos acquired at two centers, temporal phase labels on 150 videos across 13 phases, instance masks on selected frames, and interaction annotations on a subset.

The labeled subsets are much smaller than the total video counts. Treat access rights, license, consent, ethics approvals, and commercial-use permissions as prerequisites; publication of a dataset does not automatically grant the rights your product needs.

## Recommended modeling path

### 1. Build a dependable label foundation

Before training, agree on a phase taxonomy and written boundary rules with ophthalmologists. Track uncertain boundaries, out-of-order or skipped phases, visibility, video quality, procedure variant, surgeon, microscope/camera, frame rate, and source site. Double-annotate a stratified subset and adjudicate disagreements. Keep original annotator labels and reviewer changes, rather than overwriting history.

### 2. Phase proposals first

Start with an offline, frame-wise classifier and temporal post-processing:

1. Sample at approximately 1–2 frames per second for a first baseline; test higher sampling for short phases and phase transitions.
2. Extract features using a pretrained image/video encoder (for example, an ImageNet backbone or a pretrained video transformer such as VideoMAE). Compare options on held-out cases instead of assuming one architecture is best.
3. Train a lightweight temporal convolutional network such as MS-TCN as the first temporal head. Benchmark a small Transformer temporal head if data and compute support it.
4. Predict phase probabilities per sampled time point. Smooth short label flicker and merge adjacent predictions into intervals, while preserving uncertainty and allowing annotators to edit boundaries.
5. Calibrate confidence and abstain / request human review on low-confidence or out-of-distribution clips. Do not make a prediction look clinically authoritative merely because it has a score.

This is a proposal engine for retrospective annotation, not real-time operative advice.

### 3. Instrument proposals, then segmentation only if justified

For V1, prefer instrument-presence labels or time intervals (e.g., phaco probe, I/A handpiece, cannula, IOL injector) over pixel masks. Start with a pretrained detector or multi-label image classifier, sample frames, aggregate predictions into intervals, and retain human correction.

Only add instance segmentation after establishing a meaningful mask-labeling budget and use case. Fine-tune a modern segmentation model (for example, Mask2Former or a YOLO segmentation model) from pretrained weights and evaluate on independently annotated, surgeon/site-held-out cases. Dense pixel annotation is substantially more costly than temporal labels; a handful of frames per video is not enough to claim robust continuous segmentation.

### 4. Evaluation that answers the product question

- Split at **patient and surgeon** level; reserve a center/site for external evaluation when possible. Never randomly split frames from the same video across train and test.
- Report per-phase recall, macro-F1, segmental F1 at declared temporal-IoU thresholds, and boundary timing error. Include class imbalance and confidence intervals.
- For instruments, report per-class precision/recall and interval overlap; for masks, report per-class Dice/IoU and performance by site/device.
- Measure annotator time per case, AI suggestion acceptance/edit/rejection, reviewer agreement, adjudication rate, and quality defects.
- The primary product gate is **reduced annotation time without worse clinical label quality**, not a single headline accuracy.

With only 50–100 local surgeries, use patient/surgeon-grouped cross-validation for internal iteration, but describe results as pilot evidence. Keep a final set untouched where possible; add cases and independent sites before making generalization claims.

## Suggested 16-week plan

| Phase | Timing | Deliverable / exit criterion |
|---|---:|---|
| Governance and protocol | Weeks 0–2 | Approved access/consent path, privacy plan, phase taxonomy, instrument vocabulary, and annotation manual |
| Platform and baseline | Weeks 2–6 | Case intake, video review, timestamped labels, reviewer workflow, QC, versioned JSON/CSV export; measure fully manual time and agreement |
| Label collection | Weeks 3–8 | 50–100 cases with metadata; double-annotated, adjudicated subset; dataset inventory and split manifest |
| Phase model baseline | Weeks 5–9 | Pretrained encoder + MS-TCN proposal model, patient/surgeon-grouped evaluation, confidence and abstention behavior |
| Instrument baseline | Weeks 8–12 | Human-correctable presence/interval suggestions for a small, well-defined instrument set |
| Shadow pilot | Weeks 12–16 | Prospective retrospective-annotation trial; report time saved, acceptance/edit rates, agreement, subgroup/site limitations, and go/no-go decision |

Timing assumes access to videos and ophthalmologist annotation/review time. If approval or data access is not ready, that becomes the critical path.

## Product and data safeguards

- Keep identifiable video out of the prototype and public demo. Production needs encrypted storage and transfer, least-privilege access, audit logs, retention/deletion controls, and a documented de-identification process.
- Verify informed consent, ethics / IRB requirements, data ownership, licenses, and intended commercial use before uploading or training on any video.
- Store source video separately from annotation exports; export stable case/event IDs, schema version, timestamps, label taxonomy version, annotator/reviewer IDs, model/version, confidence, review state, and edit history.
- Treat AI labels as recommendations requiring human acceptance. Version the model and log which suggestions were accepted, changed, or rejected.
- Obtain clinical, privacy, regulatory, and security review before any clinical deployment or intraoperative use.

## Main risks

1. **Data access and rights:** may block the whole modeling schedule.
2. **Domain shift:** acquisition device, compression, surgeon technique, and site can change appearance and timing.
3. **Annotation ambiguity:** short phases, transitions, skipped steps, and disagreement can cap apparent model performance.
4. **Small labeled sample:** total videos do not equal phase-labeled videos; segmentation requires much denser labels.
5. **Overclaiming:** a small internal pilot cannot establish clinical validity, generalization, or autonomous decision capability.
