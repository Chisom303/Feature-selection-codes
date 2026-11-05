Noise-robust multi-objective feature selection with leakage testing for IoMT intrusion detection
Code and run-level results for the manuscript "Noise-Robust Multi-Objective Feature Selection with Leakage Testing for Intrusion Detection in Internet of Medical Things Networks".
Data
The raw data are not redistributed. Download the CICIoMT2024 dataset from the Canadian Institute for Cybersecurity (https://www.unb.ca/cic/datasets/iomt-dataset-2024.html), folder `WiFi_and_MQTT/attacks/csv`, and place these files next to the notebooks:
`Train/`: ARP_Spoofing, Benign, MQTT-DDoS-Connect_Flood, MQTT-Malformed_Data, Recon-Port_Scan, Recon-VulScan, TCP_IP-DDoS-SYN1 to 4, and TCP_IP-DoS-SYN1 to 4 (`*_train.pcap.csv`)
`Test/`: the same eight classes (`*_test.pcap.csv`)
The feature files use the column schema of the CICIoT2023 extractor; Appendix A of the manuscript lists the documented meaning of each feature. Note that the files spell Magnitude as `Magnitue`.
Notebooks (run in this order)
Notebook	Purpose	Output	Time
`0_Setup_Environment.ipynb`	Creates the `iomt` Python environment and Jupyter kernel	kernel "Python (iomt)"	minutes
`1_Data_Cleaning.ipynb`	Label coding and cleaning rules C1–C10 (train and test kept separate)	`cleaned_data/`	minutes
`2_Feature_Selection_Experiments_noIAT.ipynb`	Superseded exploratory version, kept for transparency; not used for any reported result (see below)	`results_no_IAT/` (not included)	20–40 minutes
`3_Revised_Experiments.ipynb`	Main experiment: 10 runs, 7 feature configurations, statistics	`results_revised/`	several hours
`4_Supplementary_Analyses.ipynb`	Reproducibility check, extra baselines, other perturbations, decision tree, full test set, threshold, stability (Nogueira), TOST, effect sizes, τ/θ sensitivity, hypervolume, IAT tests G1–G3	`results_supplementary/`	several hours
`5_Revision_Experiments.ipynb`	Size-controlled paired tests (A), IAT offset removal and part-file-disjoint tests (B), per-class recall after thresholding (C2), constraint checks (F1), search-budget sensitivity (D)	`results_revision/revision_results.xlsx`	2–3 hours
`6_Meaning_Coded_Search.ipynb`	Repeats the ten-run search with the meaning-based feature coding driving the noise objective	`results_revision/meaning_search_results.xlsx`	about 1 hour
`7_Correlated_Noise_v2.ipynb`	Group-correlated perturbation with each feature scaled by its documented definition	`results_revision/correlated_v2_results.xlsx`	about 30 minutes
Notebook 1 should report 1,713,169 training and 375,743 test records. Notebook 3 saves a checkpoint after every run. Notebook 4 reads `results_revised/checkpoint.pkl`, re-trains the saved subsets, and should report `proposed_subset_reproduced_runs 10` with differences of 0.0. Notebooks 5, 6, and 7 first run the code of Notebook 4, which skips the runs already saved in `results_supplementary/supp_checkpoint.pkl`; Notebooks 4 and 6 also save progress after every run, so they can be stopped and restarted.
Notebook 2 is the earlier protocol mentioned in Section 3.5 of the manuscript. It added noise scaled to each feature's standard deviation to every feature, including the 0/1 indicators, without clipping, which produced physically impossible values, and it used a smaller search, a single validation split, unweighted forests, and no consensus rule. It was replaced by Notebook 3, and none of its results are reported.
Two parts of Notebook 5 are not used in the manuscript: Part C1 (IAT jitter), because the units of IAT are undocumented and the jitter scale it derives is unrealistically large, and Part F2 (correlated noise), which scaled Magnitude by the length factor instead of its square root and is superseded by Notebook 7.
To reproduce Notebooks 4–7 without re-running Notebook 3, copy `results/results_revised/checkpoint.pkl` from this repository to `results_revised/checkpoint.pkl`, and `results/results_supplementary/supp_checkpoint.pkl` to `results_supplementary/supp_checkpoint.pkl`.
Included results
File	Contents	Manuscript
`results/cleaning/cleaning_report.xlsx`, `cleaning_summary.txt`	Cleaning log, class counts, duplicates, outliers	Table 2; Supplementary Table S1
`results/results_revised/experiment_results.xlsx`	Main experiment: run-level results (`All_runs`), means, CIs, statistical tests, selected subsets, feature frequency, per-class F1, Pareto front and confusion matrix of run 1	Tables 3, 4; Figures 2, 3; Supplementary Tables S3, S7, S9; Figures S3, S5
`results/results_revised/checkpoint.pkl`	Saved state of the main experiment	input to Notebooks 4–7
`results/results_supplementary/supplementary_results.xlsx`	Extra baselines, perturbations, decision tree, full test set, threshold, stability, effect sizes, TOST, bootstrap, τ/θ sensitivity, hypervolume, single-core latency, IAT tests G1–G3	Tables 5 (all but the last column), 6, 8, 9 (G1, G2); Supplementary Tables S4–S6, S8, S9; Figures S1, S2, S4
`results/results_supplementary/supp_checkpoint.pkl`	Saved state of the supplementary analysis, including per-run τ/θ results	input to Notebooks 5–7
`results/results_revision/revision_results.xlsx`	Size-controlled paired tests, IAT part-file tests, recall after thresholding, constraint checks, search budget	Tables 6 (paired column), 9 (centered IAT, part-file-disjoint rows); Supplementary Tables S10–S12
`results/results_revision/meaning_search_results.xlsx`	Meaning-coded search: means, paired tests, stability, overlap, feature frequency, subsets	Table 7
`results/results_revision/correlated_v2_results.xlsx`	Group-correlated perturbation: means, paired tests, run-level results, constraint check	Table 5, last column
`results/results_revision/archive_run*_G*.npz`, `archive_meaning_run*.npz`	NSGA-II archives (X = subsets, F = objectives, hv = population hypervolume) from the budget and meaning-coded searches	Supplementary Table S11; Table 7
Settings and seeds
Runs use seeds 0–9 for training samples, searches, and models; the test set is drawn once with seed 42; noisy test copies use fixed seeds shared by all configurations.
NSGA-II: population 24, 20 generations, two-point crossover, bit-flip mutation, duplicate elimination; inner evaluation with 800 records per class, 2-fold stratified cross-validation, and 10-tree forests.
Consensus: tolerance τ = 0.01, threshold θ = 0.5. Hypervolume is computed on the population in each generation with reference point (1, 1, 1).
Feature masks: range-based coding (main search) and meaning-based coding (evaluation and Notebook 6); see Appendix A of the manuscript.
Final models: class-balanced random forest with 100 trees (`balanced_subsample`), and a class-balanced decision tree.
Environment
Python 3.11.17, scikit-learn 1.9.1, pymoo 0.6.2, NumPy 2.4.6, pandas 3.0.6 (supplementary and revision analyses). Package requirements are in `requirements.txt`. Main experiment: Intel Core i5-6300U laptop; supplementary and revision analyses: Intel Core i7-8650U laptop.
License and citation
[Add a license, for example MIT, and the article citation once published.]
