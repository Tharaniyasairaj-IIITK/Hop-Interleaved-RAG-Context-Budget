# Hop-Interleaved-RAG-Context-Budget

# Hop-Interleaved RAG under a Context Budget

**Mitigating Evidence Loss: Order-Invariant and Distractor-Robust Multihop Retrieval by Hop-Interleaved Selection under a Context Budget**

Code, tests, result files and manuscript sources for a lightweight multihop retrieval algorithm with provable
budget–coverage guarantees. The algorithm builds the query of each hop from newly surfaced bridge entities and fills a
fixed context of $B$ passages with a round-robin (hop-interleaved) selector. It needs **no LLM and no neural index**.

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Tests](https://img.shields.io/badge/tests-20%20passed-brightgreen)

---

## Highlights

* **Coverage guarantee (Theorem 1).** If every gold passage lies in the top-*t* of some hop list and *B ≥ H·t*, the selected context contains all of them. The budget condition is tight (Proposition 2) and gives a design rule (Corollary 1: *B* slots guarantee a rank depth of ⌊*B/H*⌋ per hop).
* **Separation (Proposition 3).** For every *k* there is an instance on which single-step top-*k* BM25 misses a gold passage and the algorithm achieves full coverage.
* **Order invariance (Proposition 5).** Answers are invariant to the order of the routed evidence for equivariant routing, with a total-variation bound for approximate invariance.
* **Cost.** Running time *O(H(S + K log K + L))* per question; 0.06–0.23 s per question on two CPU cores.
* **Real-data gains.** On 3,000 real questions (HotpotQA, 2WikiMultihopQA, MuSiQue), average Recall@5 rises from **61.2 to 68.7**; the coverage theorem is confirmed on all 9,000 question–budget pairs (0 violations).
* **Distractor robustness.** Enlarging each corpus up to 4.5× with natural passages lowers Recall@5 by at most 2.4 points; the advantage over BM25 is retained (+6.8 vs. +7.5).

## Key results

Recall@5 (%) on the 1,000-question development subsets. Rows marked *reported* are taken from the HippoRAG evaluation of the same subsets (dense or LLM-based components; **not re-run here and not protocol-matched**). Rows marked *measured* use the own BM25 index, with no LLM and no neural component.

| Method | MuSiQue | 2Wiki | HotpotQA | Average |
|---|---:|---:|---:|---:|
| *reported* BM25 | 41.2 | 61.9 | 72.2 | 58.4 |
| *reported* Contriever | 46.6 | 57.5 | 75.5 | 59.9 |
| *reported* GTR | 49.1 | 67.9 | 73.3 | 63.4 |
| *reported* ColBERTv2 | 49.2 | 68.2 | 79.3 | 65.6 |
| *reported* IRCoT + BM25 | 44.7 | 75.6 | 79.0 | 66.4 |
| *reported* HippoRAG (ColBERTv2) | 51.9 | 89.1 | 77.7 | 72.9 |
| *reported* IRCoT + HippoRAG (ColBERTv2) | 57.6 | 93.9 | 83.0 | 78.2 |
| *measured* single-step BM25 (top-*k*) | 44.5 | 66.1 | 73.1 | 61.2 |
| *measured* iterative, RRF-merged | 48.6 | 73.7 | 76.8 | 66.4 |
| ***measured* iterative, hop-interleaved (this repo)** | **49.9** | **77.7** | **78.4** | **68.7** |

The within-system gain of +7.5 points (61.2 → 68.7) equals the gain reported for IRCoT over BM25 (58.4 → 66.4) and is obtained without an LLM. The per-cell values, including Recall@2, are in `prl/tables/retrieval.tex` (generated from `results/real/bench_*.json`).

Full-coverage check of Theorem 1 (B = 3t, H = 3, K = 10): see `results/real/lemma_check.json` and Table 3 of the paper. Interleaving raises full coverage over single-step top-*B* retrieval in all nine dataset–budget settings.

## Repository layout

```
src/gmrncm/            library: BM25 retriever, iterative multihop retrieval, hop-interleaved grounding,
                       metrics, reference reader, NCM wrappers
scripts/               experiment, table, figure and document-build scripts
tests/                 20 unit/property tests (incl. random test of Theorem 1, separation instance)
results/real/          saved result files for the three real datasets (bench_*, distractor_*, lemma_check)
notebooks/             executed experiment notebook (synthetic world + real-data cells)
prl/   ipl/            LaTeX sources, tables, figure and DOCX of the Pattern Recognition Letters / Information Processing Letters versions
short/ paper/          earlier manuscripts (ACM WWW short paper, long paper)
data/fixtures/         tiny format fixtures used by converter tests
docs/README_development_notes.md   earlier development notes of the full GMR-NCM pipeline
```

## Installation

```bash
git clone https://github.com/Tharaniyasairaj-IIITK/Hop-Interleaved-RAG-Context-Budget.git
cd Hop-Interleaved-RAG-Context-Budget
python -m venv .venv && source .venv/bin/activate
pip install -e . && pip install -r requirements.txt
pytest -q tests          # 20 passed
```
CPU only; no GPU is required for any result in the paper.

## Reproducing the real-data results

```bash
bash scripts/download_real_data.sh data/real               # 1,000-question dev subsets + corpora (HippoRAG repo, MIT)
for d in hotpotqa 2wiki musique; do
  python scripts/run_real_benchmark.py --datasets $d --seeds 0,1,2 --epochs 8 --out results/real/bench_$d.json
  python scripts/run_distractor_study.py --dataset $d      # corpus enlargement (≈10 min each)
done
python scripts/lemma_check.py                              # per-question check of Theorem 1 -> results/real/lemma_check.json
python scripts/make_short_figure.py                        # Figure 1 -> short/figures/
python scripts/make_ipl_tables.py                          # tables -> ipl/tables/ (copy to prl/tables/)
```
The first run of `run_real_benchmark.py` caches retrieval traces in `results/real/traces_*.json` (not tracked in git);
the saved result JSON files are included, so tables and figures can be rebuilt without re-running the benchmark.

### Build the manuscripts

```bash
cd prl && pdflatex main && bibtex main && pdflatex main && pdflatex main      # Elsevier two-column layout, 7 pages
pdflatex '\def\REVIEW{1}\input{main.tex}'                                      # line-numbered review layout
cd .. && python scripts/build_ipl_docx.py prl                                  # DOCX (requires pandoc)
```

## Minimal usage

```python
from gmrncm.data.hippo import load_hippo_subset
from gmrncm.pipeline import build_traces
from gmrncm.grounding import ground_context

examples, passages = load_hippo_subset("hotpotqa", "data/real", limit=20)
traces, by_id = build_traces(examples, passages)             # dict: question id -> retrieval trace (H=3, K=10)
qid = next(iter(traces))
context = ground_context(traces[qid], by_id, budget=5, mode="multihop")   # list of 5 passages, hop-interleaved
print([p.passage_id for p in context])
```

## Data

Benchmark data are the public development subsets of HotpotQA, 2WikiMultihopQA and MuSiQue as redistributed with
[HippoRAG](https://github.com/OSU-NLP-Group/HippoRAG) (MIT licence; the licences of the original datasets apply).
`scripts/download_real_data.sh` fetches them and writes SHA-256 checksums to `data/real/SHA256SUMS`. No dataset files
are committed to this repository.

## Scope of the evidence

* Theorems hold for any scorer; the measurements use BM25 and a rule-based English entity extractor on Wikipedia-based corpora.
* Baseline scores cited in the paper are reported values and are given as context; conclusions are drawn from within-system differences.
* Part (b) of Proposition 3 is verified by exact BM25 computation for *k* ∈ {1, 5, 20, 50} (`tests/test_separation.py`); a closed-form proof for all *k* is left open.
* The small extractive reference reader does not improve answer accuracy through grounding on the real subsets; the claims of the paper concern retrieval and invariance.
* Topically related distractors are not evaluated.

## Citation

```bibtex (temporary)
@article{hopinterleaved2026,
  title   = {Mitigating Evidence Loss: Order-Invariant and Distractor-Robust Multihop Retrieval by Hop-Interleaved Selection under a Context Budget},
  author  = {Author 1 and Author 2},
  journal = {Manuscript under review},
  year    = {2026},
  url     = {https://github.com/Tharaniyasairaj-IIITK/Hop-Interleaved-RAG-Context-Budget}
}
```

## License

MIT (see [LICENSE](LICENSE)). Original dataset licences apply to the benchmark data.
