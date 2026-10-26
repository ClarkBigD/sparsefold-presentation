# SparseFold paper presentation

A self-contained English HTML presentation and speaker script for a three-person, nine-minute paper talk, followed by one minute of questions.

| Presenter | Section | Slides | Duration |
|---|---|---|---|
| Speaker 1 | Background | 1–3 | 2:30 |
| Speaker 2 | Method | 4–7 | 3:30 |
| Speaker 3 | Results and conclusions | 8–11 | 3:00 |
| All | Discussion | 12 | 1:00 |

Slides 13–15 are optional backup material. Open `index.html` in a browser; no installation or network connection is required. Open `speaker-script.html` for the full printable script, or `speaker-script.md` for editable text.

Controls: arrow keys or Space to advance; Shift+Space to go back; F for fullscreen; N for notes; O for slide overview; T to start or pause the timer; R to reset it. The timer never advances slides automatically.

## Source and scope

Yunzhong Ren, Xin You, and Hailong Yang. **SparseFold: Index-Based Block Sparse Attention for Training-Free AlphaFold 3 Acceleration.** IEEE CLUSTER 2026, pp. 425–436. DOI: 10.1109/CLUSTER68337.2026.00046.

All reported measurements come from the supplied paper. Excerpts from Figures 1, 7, and 8b are credited on their slides; copyright remains with the original rights holders. The source PDF is not included. Educational diagrams are schematic, not measured attention maps. Complexity qualifications and proposed future tests are explicitly distinguished from reported experiments.

Icons: Lucide, ISC License. The bundled library retains its license header.

## Deployment

The GitHub Actions workflow `.github/workflows/pages.yml` publishes the static site to GitHub Pages on pushes to `main` and manual runs. The repository Pages source is GitHub Actions.
