# 🎓 Student Academic Performance Prediction — A+ Task Checklist (FINAL)

**Course:** IT7103 Advanced AI Applications · **Weight:** 40% (CILO 1 + CILO 2) · **Due:** 15-Dec-2026, 11:55 PM

> ⚠️ **IMPORTANT RULES**
> * **Only the team leader submits** (on Moodle)
> * Built in **Google Colab**, submitted as the downloaded **.ipynb**
> * Report must be **.doc / .docx** (not PDF)
> * **ONE ZIP folder only (RAR = zero marks)** containing the .ipynb file(s) + .docx report
> * ZIP named **StreamNo_LeaderID** (e.g. `08_202316152`)
> * Late = capped at 60%; **2 calendar days late = 0%**
> * Every step in Colab must have **comments / markdown explanations**
> * **Justify every choice** (the brief says "explain and justify methodological choices at each stage")
> * Any references consulted must be **cited**

---

## 📊 0. Dataset Facts (Project1-EducationDataset.csv) — PLAN FOR THESE
* **154,314 rows × 20 columns**, 6,000 students, 8 semesters (2020Spring → 2023Fall)
* **Several courses per student per semester** (avg ~4.3) → lag/rolling features need a **student-semester level** (e.g. average final_exam per semester) or a clearly justified alternative
* **Semester is text like `2021Spring`** → alphabetical sort is WRONG (`2020Fall` comes before `2020Spring`). Convert to a **year + term order number** before sorting
* **Missing values (~7% each):** study_hours, attendance_rate, homework_avg, **final_exam (target)**
* **Invalid / inconsistent values to clean:**
  * grade_letter has **`ERR` label (780 rows)**
  * numeric_grade has **-10 and 105–110** (outside 0–100)
  * prior_gpa has **-1 and 5.0** (outside 0–4)
  * age up to **65**, study_hours up to **119.9** per week → outlier candidates
  * **1,911 repeated student + semester + course rows** with different values → decide: duplicates, retakes, or errors?
* **Severe class imbalance:** B = 27,775 rows but **F = only 544** → balancing is required
* **numeric_grade correlates 0.66 with final_exam** → likely calculated from it, so treat it as **leakage for regression too** (or justify keeping it)
* **3,873 students have 6+ semesters** → plenty of choice for the time-series student

## 🟢 1. Project Setup
* [ ] Create shared Google Colab notebook (share with all members as editors)
* [ ] Create shared Google Drive folder (dataset, notebook, report, figures)
* [ ] Load dataset into Colab (Drive mount or upload)
* [ ] Import libraries (pandas, numpy, matplotlib/seaborn, scikit-learn, statsmodels, imbalanced-learn)
* [ ] Set a fixed **random_state** for reproducibility
* [ ] Title cell with group members' names + student IDs + short project intro
* [ ] Split tasks between members (see table at bottom)

## 🟡 2. Data Cleaning & Outlier Handling (Objective 1)
* [ ] Initial inspection: `shape`, `info()`, `describe()`, `head()`, `value_counts()` on categoricals
* [ ] **Missing values:** count + % per column, then handle (impute / drop) and **justify**
  * [ ] **Do not impute the target** (final_exam) for training — drop those rows for regression
* [ ] **Inconsistent records:**
  * [ ] Remove/fix **`ERR`** grade_letter rows
  * [ ] Fix numeric_grade outside **0–100**, prior_gpa outside **0–4**
  * [ ] Investigate the **repeated student+semester+course** rows and justify handling
  * [ ] Check static fields are consistent per student (gender, scholarship)
* [ ] **Detect outliers** (boxplots + IQR or Z-score) on numerical columns
* [ ] **Handle outliers** (cap / remove / keep) and **explain impact** on model readiness
* [ ] Show **before vs after** (row counts, describe(), plots)

## 🟠 3. Pre-processing & Feature Engineering (Objective 2)
* [ ] Encode categoricals (one-hot for subject_area etc.; **ordinal** for year_of_study)
* [ ] Decide on high-cardinality columns (course_name / course_code / instructor) and justify
* [ ] Binary columns (scholarship, internship) as 0/1; note **extracurricular is a count 0–10**, not binary
* [ ] Scale numerical features where the model needs it (fit scaler on **train only**)
* [ ] Derived indicators (e.g. study_hours × attendance_rate, homework_avg − prior_gpa scaled, credit load per semester)
* [ ] **Time-series structure:**
  * [ ] **Create semester order number** and **sort each student by it**
  * [ ] **Lag features** (e.g. **final_exam_prev_semester**, prev homework_avg)
  * [ ] **Rolling statistics** (e.g. rolling mean of final_exam) — **shift first** so it never includes the current semester
  * [ ] **Continuous semester sequence per student** — fill missing semesters with NaN (reindex)
* [ ] **Check class imbalance** in grade_letter (bar chart)
* [ ] Prepare **two feature sets**: regression (target = final_exam) and classification (target = grade_letter)
* [ ] Train/test split (**stratified** for classification); fit all preprocessing on train only (use a **Pipeline**)

## 🔵 4. Correlation & Time Series Analysis (Objective 3)
* [ ] Correlation matrix + **heatmap** (features vs final_exam)
* [ ] Rank top features correlated with the target
* [ ] Scatter/box plots of key relationships (study_hours, attendance_rate, prior_gpa, homework_avg vs final_exam)
* [ ] **Time-series analysis:** average final_exam per semester (trend line), lag feature vs current score correlation
* [ ] Short written interpretation under every plot

## 🟣 5. Regression — Predict final_exam (Objective 4)
* [ ] **Exclude numeric_grade / grade_letter** (or explicitly test their effect and say so)
* [ ] **Baseline** (predict the mean) for reference
* [ ] **One single model** (e.g. Linear Regression / Decision Tree)
* [ ] **One ensemble model** (e.g. Random Forest / Gradient Boosting)
* [ ] Evaluate on **test set**: **RMSE, MAE, R²** + extra (e.g. MAPE / adjusted R²)
* [ ] Plot **actual vs predicted** + residuals
* [ ] Comparison table: baseline vs single vs ensemble

## 🔎 6. Classification — Predict grade_letter (Objective 5)
* [ ] **Multi-class** target (A, A-, B+, B, B-, C+, C, C-, D, F) after removing `ERR`
* [ ] **Exclude numeric_grade** (direct proxy → leakage); optional extra experiment that **explicitly** shows its effect
* [ ] Check for other leakage and justify features kept
* [ ] **Handle imbalance** (SMOTE / class_weight / resampling) — **training data only**, never the test set
* [ ] Show class distribution **before vs after** balancing
* [ ] Evaluate with **accuracy, precision, recall, F1-macro**
* [ ] **Confusion matrix** (plotted) + full classification report
* [ ] **Discuss** how imbalance affected results (esp. the rare F class)

## 🟤 7. Time Series Forecasting — One Student (Objective 6)
* [ ] Pick a student with **at least 6 semesters** (explain why you chose them)
* [ ] Filter + **sort chronologically** (using the semester order number, not text)
* [ ] Aggregate to **one final_exam value per semester** (they take several courses)
* [ ] **EDA** of the series (line plot, trend, variability)
* [ ] **Justify** the forecasting method based on short series length (e.g. naive, moving average, exponential smoothing, simple ARIMA)
* [ ] **Chronological train/test split** (no shuffling)
* [ ] Fit on train, forecast the test period
* [ ] Evaluate with **RMSE, MAE, MAPE** (+ any other)
* [ ] **Plot actual vs predicted**
* [ ] (Optional ⭐) **Time series cross-validation** (expanding window) for forecast stability

## 🟢 8. Ensemble Modeling — Both Tasks (Objective 7)
* [ ] Ensemble for **classification** (e.g. Random Forest, Gradient Boosting, Voting/Stacking)
* [ ] Ensemble for **regression**
* [ ] **Compare ensembles vs single models** in one results table per task
* [ ] Discuss **strengths and limitations** (accuracy, training time, interpretability, overfitting)

## 🟠 9. Hyperparameter Optimization (Objective 8)
* [ ] Choose a search strategy (GridSearchCV / RandomizedSearchCV / Optuna) and **explain why**
* [ ] Use **cross-validation** inside the search (tune on train only)
* [ ] Tune the **regression** model
* [ ] Tune the **classification** model
* [ ] Report search space + **best parameters**
* [ ] **Before vs after tuning** table on the test set
* [ ] **Interpret carefully** — is the gain real or tiny/noise?

## 🧠 10. Interpretation & Critical Analysis
* [ ] **Feature importance** (tree importances / permutation importance / coefficients)
* [ ] Explain which factors drive academic success and why that makes sense
* [ ] Error analysis: which grades/students the models get wrong
* [ ] **Critical analysis:** limitations, suitability of the methods for this problem
* [ ] Practical use: how advisors could use it to flag at-risk students early

## 🔴 11. Notebook Quality (Codebase Deliverable)
* [ ] Section headings that match Objectives 1–8
* [ ] **Every step has comments / markdown explanation**
* [ ] **Modular** code (reusable functions for cleaning, evaluation, plots)
* [ ] Workflow visible end-to-end: preprocessing → training → evaluation → **result visualisation**
* [ ] **Runtime → Restart and run all** with no errors
* [ ] All outputs and plots saved in the notebook
* [ ] Download the final **.ipynb** from Colab

## 📝 12. Project Report (.docx)
* [ ] **Cover page with full names + student IDs of all members**
* [ ] Completed **Assessment Cover Sheet** attached
* [ ] Explains the **steps taken to complete the project**
* [ ] a. Problem explanation + real-world relevance
* [ ] b. Objectives + AI approach adopted
* [ ] c. Data description + preparation steps
* [ ] d. ML methods + **rationale for choices**
* [ ] e. Performance results + **comparison across approaches**
* [ ] f. Model interpretation (**influential features**, decision patterns)
* [ ] g. Practical implications, challenges, limitations, improvements
* [ ] h. Summary of findings + final reflections
* [ ] Key figures/tables (heatmap, confusion matrix, forecast plot, comparison + tuning tables)
* [ ] References cited properly
* [ ] Proofread + consistent professional formatting

## 📦 13. Final Submission
* [ ] Final **.ipynb** (downloaded from Colab)
* [ ] Final **.docx** report
* [ ] Both in **one ZIP folder** (**NOT RAR**)
* [ ] Named **StreamNo_LeaderID** (e.g. **08_202316152**)
* [ ] **Team leader uploads to Moodle** before **15-Dec-2026, 11:55 PM**
* [ ] Re-download and check the ZIP opens correctly

## 👥 Team Task Split
| Member | Sections Owned | Status |
|--------|----------------|--------|
| Member 1 (Leader) | | |
| Member 2 | | |
| Member 3 | | |
| Member 4 | | |
