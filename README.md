# RACE: distribution-free control of over-correction in Arabic GEC

## Pipeline

All steps run on CPU (tested on 2 cores, 7 GB RAM). `pip install -r code/requirements.txt`.

1. `python3 code/prep_data.py` – extracts sentences from the OpenITI works and the
   Tanzil Qur'an text, mines silver rhetorical / isti'ara labels from the two tafsir works,
   builds splits. `code/canonical_istiara.py` holds the 30 canonical exemplar tokens.
2. `python3 code/gec_data.py` – seeded synthetic error injection (six families) and
   data-derived edit tag set (|T| = 913).
3. `python3 code/pretrain.py` – MLM pretraining of the shared encoder (d=256, 4 layers).
4. `bash code/run_queue.sh code/queue_main.txt` then `code/queue_rest.txt` – trains
   RACE, baselines (BiLSTM, scratch transformer, pretrained tagger, stacked hybrid,
   seq2seq) and the seven ablations; each line is `model seed`. Predictions go to
   `results/preds/`.
5. `python3 code/analysis.py` – calibration, conformal risk control, rhetoric-gate sweeps,
   metrics, Welch/Holm statistics, ablation table data.
6. `python3 code/serve_bench.py` – measured latency, batching, worker scaling.
   `python3 code/routing_extra.py` – routing sweep, theta chosen on the development split, mixture risk,
   end-to-end conformal threshold. `python3 code/analysis_extra.py` – 1,000 calibration resplits,
   Qur'an/training-prose overlap, non-inferiority test, AUC intervals, duplicate check.
7. `python3 code/simulate.py` – discrete-event cluster model; `python3 code/load_test.py` – measured
   open-loop load test on the local node, compared with the model (run on an idle machine).
   `python3 code/multidomain.py build eval` – four extra held-out OpenITI works (`data/openiti_extra/`), recalibration.
   `python3 code/analysis_extra.py rcbase` – conformal risk control vs naive threshold vs Learn-then-Test.
   `python3 code/bootstrap_sig.py` – paired sentence-level bootstrap of F0.5 differences (classical bootstrap table).
   `python3 code/mem_profile.py` – resident memory of light and heavy serving workers.
   `python3 code/design_space.py` – risk-invariant design-space exploration of routing level and cores (core-count table).
   `python3 code/monitor.py synthetic` – recalibration-budget study and label-free shift monitor (classical prose).
   `python3 code/cost_all.py` – equal-footing latency/throughput of every model (idle machine).
   `code/cs/` – hyperparameter search (`make_queues.py`, `hp_select.py`), leak-free data (`leakfree_data.py`),
   evaluation of any tag (`eval_tags.py`), and GPU / multi-node scripts (`hf_tagger.py`, `hf_seq2seq.py`,
   `llm_correct.py`, `gen_eval.py`, `cluster_bench.py`; see `code/cs/README_CS.md`).
8. `python3 code/gen_tables.py && python3 code/make_figs.py && python3 code/figs_arch.py`
   – regenerates tables, `numbers.tex` and figures.
9. `pdflatex main && bibtex main && pdflatex main && pdflatex main`.



3. In `Q/`: `python3 code/train.py --model ours --seed S` and `--model pt_tagger --seed S` for S = 1, 2, 3.
4. In `Q/`: `python3 code/real_eval.py` (Table 4, resplits, bootstrap) and `python3 code/monitor.py real`.
   Copy `Q/results/real_*.csv`, `monitor_real*` and `Q/data/processed/qalb_stats.json` to `results/real/`.
