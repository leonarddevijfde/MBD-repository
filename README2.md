`markdown
# Integrated Flood-Risk Management Plan for the IJssel River

A reproducible workflow for exploring, designing and stress-testing flood-risk policies with **EMA Workbench**.

---

## 1  Setup  

* **Python** 3.9 or newer.  
* Clone this repository  
  bash
  git clone https://github.com/your-org/IJssel-flood-plan.git
  cd IJssel-flood-plan
`

* Install requirements

  bash
  pip install -r requirements.txt
  

When the environment is ready, open the notebooks in Jupyter Lab or VS Code and run them in order.

---

## 2  Notebook workflow

### **Step 1 – Exploratory modelling** `Exploratory_Modelling.ipynb`

* **Baseline run**: 40 000 scenarios with every lever off → *data/exploratory\_results\_40000\_zero\_policy.tar.gz*
* **Max-intervention run**: 40 000 scenarios with every lever on → *data/exploratory\_results\_40000\_one\_policy.tar.gz*
  Comparing the two extremes reveals the full outcome range and the basic risk–investment trade-off.

### **Step 2 – Scenario discovery** `Scenario_Discovery.ipynb`

* Load the zero-policy results, split uncertainties and levers, and tag “high-risk” rows (top 15 % on deaths).
* Run PRIM (density 0.8) to carve out boxes with a shared risk signature, pick the best box (e.g. Box 38) and inspect its constraints (`A1_pfail`, `A3_pfail`).
* From that box pull four representative scenarios and save them to *data/Selected\_Scenarios2.csv*.
  These scenarios become the stress-tests for later optimisation.

### **Step 3 – Global sensitivity analysis** `GSA.ipynb`

* Freeze all levers at zero and fire 1 000 Sobol samples through the model (saved to *results\_sequential.tar.gz*).
* Compute first- and total-order indices for **Expected Annual Damage** and **Expected Number of Deaths**.

  * Damage: `A3_pfail` dominates, `A1_pfail` a clear second.
  * Deaths: `A1_pfail` tops the list, then `A3_pfail`; `discount rate 1`, `A2_Bmax`, `A3_Brate` show smaller effects.
    Bar plots make the ranking obvious and guide where reliability upgrades matter most.

### **Step 4 – MOEA analysis** `MOEA.ipynb`

* Read the four stress-test scenarios, set ε = 1 000 000 € for each cost metric and ε = 10 for deaths, allow 40 000 function evaluations, run three seeds.
* ε-NSGA-II mutates 40 levers, logs every non-dominated solution to *archives/optimization.tar.gz* plus per-seed CSVs.
* A parallel-coordinates plot shows the emerging five-objective Pareto front.
* Keep only true non-dominated points (≈1 500 of 12 000) and drop policies with deaths > 0.01.
* Export the surviving lever sets to *Policies\_filtered\_on\_deaths.csv* for robustness testing.

### **Step 5 – Policy evaluation & robustness** `Policy_Evaluation.ipynb`

* Load the filtered policies and run each through 1 000 Latin Hypercube scenarios.
* **Signal-to-Noise Ratio**: spot policies that perform well **and** consistently.
* Drop the 200 worst on deaths, and any plan with more than two dike-height or RfR interventions (political realism check).
* **Regret analysis**: calculate per-scenario regret across all metrics, visualise with a heat-map and a parallel-coords plot.
* Roll up mean investment, damage and deaths; scatter-plot the cost–risk cloud.
* **Select finalists**: four policies that meet the lever-count rule and score best on a composite of normalised metrics, plus one global best.

  * Lever settings → *Final\_policies.csv*
  * Outcome summaries → *Final\_policies\_outcomes.csv*

---

### Outputs at a glance

| Notebook              | Key artefacts                                                                                  |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| Exploratory Modelling | `exploratory_results_40000_zero_policy.tar.gz`   `exploratory_results_40000_one_policy.tar.gz` |
| Scenario Discovery    | `Selected_Scenarios2.csv`                                                                      |
| GSA                   | `results_sequential.tar.gz`   Sobol bar plots                                                  |
| MOEA                  | `archives/optimization.tar.gz`   `result__*.csv`   `Policies_filtered_on_deaths.csv`           |
| Policy Evaluation     | `Final_policies.csv`   `Final_policies_outcomes.csv`   SNR plot   Regret heat-map              |

---

Run the notebooks in sequence to reproduce the full analysis, from broad uncertainty exploration to a shortlist of robust, politically viable flood-risk policies.


```