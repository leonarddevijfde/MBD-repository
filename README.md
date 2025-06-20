Integrated Flood-Risk Management Plan for the IJssel River

Setup

1. Have an up to date version of python installed
2. Clone the repository to your python editor
3. Install the required python packages. These are in the requirements.txt file and can be installed using pip install


Once everything is installed, the notebooks can be runned.

Step 1 – Exploratory modelling

Inside Exploratory Modelling.ipynb two large experiment batches are created.

Baseline run. Forty-thousand scenarios are generated with every policy lever switched off, providing a reference case with no interventions. The output is stored in data/exploratory_results_40000_zero_policy.tar.gz.
Max-intervention run. Another set of forty-thousand scenarios is executed with every lever fully activated, representing the most intensive policy package. These results are written to data/exploratory_results_40000_one_policy.tar.gz.
Running the extreme ends of the decision space side-by-side allows us to visualise the full range of possible outcomes and the trade-offs between risk reduction and intervention intensity.

Step 2: Scenario Discovery

The steps for running the Scenario_Discovery.ipynb notebook are as follows:

1. Load and Prepare Data

    Load results from exploratory modeling (exploratory_results_40000_zero_policy.tar.gz).

    Convert experiments and outcomes to DataFrames and merge them.

    Split input data into uncertainties and levers.

2. Apply PRIM for Scenario Discovery

    Use Expected Number of Deaths as the key outcome.

    Define high-risk scenarios as those above the 85th percentile.

    Run PRIM with a density threshold of 0.8.

    Select boxes with high density and acceptable coverage (0.8–0.9).

3. Analyze Selected Box

    Choose the best-performing box (e.g. Box 38).

    Inspect constraints on key parameters (A.1_pfail, A.3_pfail).

    Visualize tradeoffs and inspect box structure.

4. Extract Representative Scenarios

    Filter full results to match box constraints.

    Identify best/worst outcomes for fatalities and damages.

    Extract a diverse subset of four scenarios.

5. Export Scenarios

    Save the selected high-risk scenarios to ./data/Selected_Scenarios2.csv for use in policy robustness testing.

Output:

A CSV file containing four critical scenarios for evaluating flood policy performance under deep uncertainty.\


Step 3 – Global Sensitivity Analysis (GSA)

GSA.ipynb answers one question: which uncertainties drive the results the most?

* *Setup*

  * Fix every lever at zero → “no-action” baseline.
  * Draw 1 000 Sobol samples from the uncertainty ranges (Samplers.SOBOL).
  * Run the samples with SequentialEvaluator; save inputs + outputs to results_sequential.tar.gz.

* *Sobol processing*

  * Build the SALib problem with get_SALib_problem.
  * Compute first-order (S1) and total-order (ST) indices for

    * Expected Annual Damage
    * Expected Number of Deaths.

* *Quick findings*

  * *Damage*: A3_pfail is by far the strongest driver, A1_pfail comes next.
  * *Deaths*: ranking reverses – A1_pfail leads, A3_pfail follows, with discount-rate-1, A2_Bmax, and A3_Brate as lesser but visible contributors.
  * Bar plots with error bars (plot_sobol) make the ranking easy to read.

Step 4 – MOEA analysis

MOEA.ipynb uses ε-NSGA-II to search the lever space for policies that balance five objectives: expected annual damage, dike costs, Room-for-the-River costs, evacuation costs and expected deaths.

* *Set-up*

  * Read five “stress-test” scenarios that came out of the earlier PRIM screening (Selected_Scenarios2.csv).
  * Choose ε-boxes of one million euro on the four cost metrics and ten casualties on the life-safety metric.
  * Allow 40 000 function evaluations per run and repeat each run three times with different random seeds.
  * Log every non-dominated solution to archives/optimization.tar.gz; save the full results for each seed in separate CSV files.

* *Optimisation loop*

  * For every scenario and seed, ε-NSGA-II mutates the 40 lever settings, evaluating each candidate in the dike model.
  * Convergence is tracked with hyper-volume, generational distance and ε-progress measures (code included, though column-name quirks in EMA archives prevented the plots from rendering during the demo).

* *Inspect the Pareto front*

  * A helper pulls all result__*.csv files and combines them into one DataFrame.
  * A parallel-coordinates plot gives a quick sense of the trade-offs among the five objectives for the chosen ε setting.

* *Filter non-dominated solutions*

  * A vectorised dominance check keeps only the true Pareto points (≈ 1 500 remain out of \~12 000).
  * To keep the follow-up robustness work manageable—and politically realistic—policies with expected deaths above 0.01 are dropped.

* *Export for robustness testing*

  * The surviving lever sets write to Policies_filtered_on_deaths.csv; these are the candidates that feed directly into the robustness notebook.


Step 5 – Policy evaluation and robustness check

Policy_Evaluation.ipynb takes the Pareto‐optimal policies we obtained from the MOEA and stress-tests them under uncertainty.

* *Load policies and model*

  * Read the CSV file Policies_filtered_on_deaths.csv, which contains the non-dominated lever settings.
  * Pull in the IJssel dike model (problem formulation 2).
  * Turn each policy row into an EMA Workbench Policy object with its own name.

* *Run 1 000 scenarios per policy*

  * A MultiprocessingEvaluator sends each policy through 1 000 Latin Hypercube scenarios, capturing every outcome of interest.
  * The raw experiments and outputs are merged into one tidy DataFrame for later plots.

* *Signal-to-noise ratio (SNR)*

  * For each outcome we compute SNR:

    * maximise objectives → mean ÷ std,
    * minimise objectives → mean × std.
  * A parallel-coordinates plot shows SNR patterns across all policies, making it easy to spot which options deliver stable performance, not just good averages.

* *Filter the policy pool*

  * Drop the 200 policies with the highest expected deaths, leaving a set that is politically palatable.
  * Keep lever settings where no more than two dike-height or RfR interventions occur – a nod to stakeholder tolerance.

* *Regret analysis*

  * For each scenario we compute regret: the gap between a policy’s outcome and the best performer in that same scenario.
  * Regret matrices feed a heat-map and a parallel-coordinates plot, highlighting which policies avoid worst-case losses across multiple metrics.

* *Roll-up metrics*

  * Average investment, damage and death figures are calculated for every remaining policy.
  * A scatter plot (investment on the x-axis, damage on the y-axis, colour by deaths) visualises the cost–risk trade-off.

* *Pick five finalists*

  * Four policies meet the lever-count rule and score best on a composite of normalised deaths, damage and cost.
  * One extra “global best” policy is added, even if its lever count is higher, so decision-makers can weigh a pure performance benchmark.
  * Lever settings go to Final_policies.csv; their outcome summaries to Final_policies_outcomes.csv.
