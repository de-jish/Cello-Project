# Cello Fingering Generator

> An ML system that generates playable cello fingerings for sheet music, paired with a web tool for viewing and editing them on the score.

Coming up with fingerings is one of the most tedious parts of learning a new cello piece. Most of that know-how lives in private lessons, marked-up scores, and personal annotations where it is hard for others to access. This project tackles that with two tools: a model that suggests a solid starting set of fingerings for a piece, and a web platform where cellists can see those fingerings on the score and adjust them.

## Status

In active development. See the Roadmap for what is built and what is planned.

## Why this approach

A few design decisions shape the whole project:

- **The optimizer generalizes, the model specializes.** A rule-based optimizer grounded in cello ergonomics produces reasonable fingerings for any monophonic piece with no training data, so the tool is useful from day one. The learned model improves on that for repertoire it has seen, and the optimizer covers the long tail.
- **Pretraining solves the small-data problem.** Annotated cello fingerings barely exist in machine-readable form, so a model trained only on them would stay shallow. Pretraining a transformer on abundant unlabeled music first, then fine-tuning on the small labeled set, follows the same recipe that makes modern language models work.
- **Annotations are separated from notation.** The platform stores fingerings as a layer keyed to note positions, not the copyrighted score itself, which keeps multiple interpretations coexisting over one piece and sidesteps most copyright issues.

## How it works

### 1. Rule-based optimizer (baseline and fallback)

For each note, the system enumerates the physically possible (string, position, finger) combinations. A cost function penalizes shift distance, string crossings, awkward stretches, and unwanted position changes, and the Viterbi algorithm finds the minimum-cost path through the whole piece. This is a pure combinatorial optimization problem and needs no training data.

### 2. Transformer (pretrained, then fine-tuned)

- **Pretraining:** a transformer is trained on a large corpus of unlabeled MusicXML using a self-supervised objective (masked or next-note prediction), learning general musical structure with zero fingering labels.
- **Fine-tuning:** the pretrained model is adapted on the small labeled fingering dataset to predict a fingering sequence for a given piece.
- **Comparison:** the fine-tuned model is evaluated against the optimizer and against an identical transformer trained from scratch, isolating how much pretraining actually helps.

### 3. Web platform

The score renders in the browser with fingerings overlaid, and users can click a note to change its fingering and save the result. The same interface doubles as the labeling tool for building the dataset. A deployed inference endpoint serves model fingerings inside the platform, with the optimizer as the fallback for out-of-distribution pieces.

## Tech stack

- **Language:** Python
- **ML:** PyTorch
- **Music parsing:** music21
- **Score rendering:** Verovio
- **Serving:** FastAPI
- **Data format:** MusicXML

## Project structure

```
.
├── data/
│   ├── unlabeled/        # Public-domain MusicXML corpus for pretraining
│   └── labeled/          # Hand-annotated fingerings for fine-tuning
├── src/
│   ├── representation/   # music21 parsing, candidate enumeration, schema
│   ├── optimizer/        # Cost function and Viterbi decoder
│   ├── model/            # Transformer, pretraining, fine-tuning, eval
│   └── serving/          # Inference endpoint
├── web/                  # Score display and annotation editor
└── README.md
```

## Roadmap

- [ ] **Phase 0** Representation, corpus, and tooling. music21 parsing, fingering schema, per-note candidate enumeration.
- [ ] **Phase 1** Rule-based optimizer. Cost function plus Viterbi decoding for a complete fingering.
- [ ] **Phase 2** Minimal platform. Score display, edit, and save, which also serves as the labeling interface.
- [ ] **Phase 3** Datasets. Build the labeled fingering set and assemble the large unlabeled pretraining corpus.
- [ ] **Phase 4** Pretraining. Self-supervised transformer trained on the unlabeled corpus.
- [ ] **Phase 5** Fine-tuning and evaluation. Adapt the model for fingering, then compare against the optimizer and a from-scratch baseline.
- [ ] **Phase 6** Deployment. Serve the model end to end inside the platform with the optimizer as fallback.
- [ ] **Phase 7** Stretch goals. Active learning, preference learning from user votes, and a community layer.

## Dataset and copyright

The pretraining corpus is drawn from public-domain MusicXML (the music21 corpus, IMSLP, and MuseScore). The labeled fingering set is built by hand using the platform's editor, with the optimizer pre-filling a draft that is then corrected. Where possible, multiple acceptable fingerings are recorded per passage so evaluation does not penalize the model for choosing a valid alternative. Fingering annotations are stored separately from any copyrighted notation.

## Evaluation

Cello fingering rarely has a single correct answer, so naive per-note accuracy against one reference is misleading. Evaluation accounts for multiple valid fingerings and is supplemented with human judgment of playability, so a model that picks a different but reasonable fingering is not unfairly marked wrong.

## Author

Joshua Jang