# Bayesian networks for heart disease

Can a network built from clinical knowledge predict heart disease as well as one learned from the data?

## Data

The Cleveland heart disease data (UCI): 303 patients, with 6 rows dropped because of missing values in vessels affected and the thalassemia result. The 0 to 4 severity field became a yes or no target, with 137 of the remaining 297 patients (46%) diseased. A stratified 75/25 split gave 222 training and 75 test patients.

## Method

- **Preparation:** five continuous measures grouped into categories, using clinical cut-offs for age, blood pressure and cholesterol.
- **Expert network:** 18 edges in three layers (background risk factors, disease, clinical findings), with every edge tested by a chi-square test.
- **Learned network:** hill climbing with the BIC score. The first run drew arrows from disease into age and from thalassemia into sex, which cannot be causal, so every edge into age and sex was blocked and the search rerun.
- **Fitting and inference:** both networks fitted with the K2 prior using `bnlearn` and `pgmpy`, then used for 192 evidence combinations, one-variable-at-a-time changes, reverse queries and classification of the 75 test patients.

## Results

| Network | Edges | ROC-AUC | Accuracy | Precision |
|---|---|---|---|---|
| Expert | 18 | 0.920 | 81.3% | 0.839 |
| Learned | 8 | 0.922 | 85.3% | 0.900 |

- The two networks perform almost the same on unseen patients; the learned network is slightly more accurate and precise at the 0.50 threshold.
- 14 of the 18 expert edges passed the chi-square test. Three of the four that failed involve cholesterol, which fits a sample of patients already referred for angiography. They were kept, because pruning them would have removed cholesterol from the network.
- The learned network kept the diagnostic core (thalassemia, vessels affected, exercise angina and maximum heart rate all linked to disease) but left five variables unconnected, including age.
- A normal thallium scan is the strongest single result: it cuts the odds of disease to about a third. A reversible defect multiplies them by 3.5, and asymptomatic chest pain by about 2.9.
- The Markov blanket behaves as the graph says: with all ten blanket variables known, adding blood pressure, fasting blood sugar or ST slope left the probability of disease unchanged at 0.0051.

## Limitations

With 222 training patients, BIC drops weak links such as age even though its chi-square test gave p < 0.001. Because sex reaches disease only through the thallium result in the learned network, switching a high-risk patient from male to female raises the estimate from 0.957 to 0.989, the opposite of what clinicians expect. A better test score does not make the learned network the right one to use. The K2 prior was chosen over BDeu, which squeezed every probability towards 0.5.

## Files

| File | Contents |
|---|---|
| `bayesian_networks_heart_disease.ipynb` | Full notebook with outputs |
| `expert_dag_model.pkl` | Saved expert network |
| `bnlearn_dag_model.pkl` | Saved learned network |
| `bn_context.pkl` | Saved context used for inference |
| `reports/` | Written report and presentation slides (PDF) |

## How to run

Open the notebook in Google Colab and upload the Cleveland heart disease data file named in the first cell. The saved `.pkl` models can be loaded with `pgmpy`.

## Team

EDAB6808 Assignment 3, BCom Honours in Business Analytics, University of the Free State, 2026. Group project by Fumane Malangabe and Buhle Mbokodo.
