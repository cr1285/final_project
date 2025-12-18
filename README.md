
# Economic Conditions and Online Hostility Toward Immigrants

**Final Project – Data Science**

## 1. Title and abstract

### Title

**Economic Conditions and Online Hostility Toward Immigrants: Evidence from Congressional YouTube Comments**

### Abstract

This project studies whether state-level economic conditions help explain online hostility toward immigrants. We combine two types of data for the period 2021–2024. First, we use about 600,000 YouTube comments posted on official channels of members of the U.S. House of Representatives. Second, we assemble a monthly economic panel for New York State, including the non-citizen share from the CPS, employment, unemployment, wages, and the New York Fed Coincident Index (NYPHCI).

On the text side, we fit a Latent Dirichlet Allocation (LDA) topic model to the full comment stream to identify immigration-related discussion and other major topics. For the subset of comments classified as immigration-related, we apply two pre-trained transformer models from Hugging Face: an English sentiment classifier and a multilingual toxicity / hate-speech classifier. We aggregate these outputs by month to construct several “immigration hostility” indices that combine (i) how salient immigration is in the overall conversation and (ii) how negative or hateful immigration comments are.

We then merge these indices with the New York economic panel and estimate a series of monthly regression models. The non-citizen share is not significantly associated with unemployment or wages and has, at best, a weak positive association with total employment. By contrast, the immigration hostility indices are systematically higher when New York’s coincident index is lower: worse overall economic conditions coincide with more negative and hateful rhetoric about immigrants on congressional YouTube channels, even though the immigrant share itself is relatively stable. This pattern is consistent with theories in which broader economic anxiety, rather than the direct labor-market impact of immigration, fuels online hostility.

---

## 2. Repository structure

Top-level folders:

* `code/`

  * `LDA/` – notebooks for topic modelling

    * `LDA relevant.ipynb`
  * `regression/` – R Markdown for regressions

    * `regression.Rmd`, `regression.html`
    * `regression2.Rmd`, `regression2.html`
  * `sentiment/` – (if needed) helper functions for sentiment / hate analysis
  * `analyzing_comments_new_env.ipynb` – scraping and cleaning YouTube comments
  * `analyzing_immigraiton.ipynb` – immigration sentiment & hate analysis
  * `final_project_creating_dataset.ipynb` – building congressional metadata

* `data/`

  * `ECON/`

    * `CPS data/` – economic inputs

      * `Unemployment.xlsx`, `all_employees.xlsx`, `monthly_wage.xlsx`,
        `replace_GDP.xlsx`, `yearly_populationNY.xlsx`
  * `regression/`

    * `data.xlsx` – final merged monthly panel for regressions
  * `sentiment/`

    * `raw_data/` – raw comment-level files used in sentiment analysis
    * `clean_data/` – cleaned / filtered sentiment datasets (e.g., immigration-only comments)

* `figures/`

  * `ECON/` – economic time-series figures

    * `economic_graphs.docx`
  * `LDA/` – topic-model figures

    * `LDA graphs.docx`
  * `sentiment/` – PNGs for sentiment & hate results

    * `n_hate_over_time.png`, `n_negative_imm_com_over_time.png`,
      `n_negative_over_time.png`, `party_share_anti_imm_comm.png`,
      `volume_vs_hate_share.png`, etc.

* `report/`

  * `DS_report.pdf` – final project report
  * `pre_slides.pdf` – presentation slides
  * `proposal.docx` – original project proposal

* `reference_materials/` – background papers and notes

---

## 3. Inputs: data and raw files

### 3.1 Congressional and social-media metadata

**`data/sentiment/raw_data/df_reps_with_youtube.csv`**
Representative-level dataset used to identify which House members have YouTube channels and to attach political characteristics to each channel. One row per member of Congress.

Key variables:

* `first`, `last` – first and last name of the member of Congress
* `id.bioguide` – unique Bioguide ID, used to merge across official congressional datasets
* `current_type` – type of office (here: Representative)
* `current_party` – party affiliation (Democrat, Republican, etc.)
* `current_state` – U.S. state represented
* `term_start`, `term_end` – start and end dates of the congressional term
* `social.youtube`, `social.youtube_id` – whether the member has a YouTube presence and the corresponding channel identifier / username
* `social.twitter`, `social.twitter_id` – Twitter/X presence and handle
* `social.instagram`, `social.instagram_id` – Instagram presence and handle
* `social.facebook` – Facebook page or identifier
* `youtube_url` – fully constructed URL to the member’s official YouTube channel (standardized from different kinds of YouTube identifiers)

Supporting raw files (only used inside scripts, not in the analysis directly):

* `reference_materials/df_legs_media.csv`, `reference_materials/clean-df_legs_media.csv` – larger legislators–media datasets used to build the representative file
* `reference_materials/legislators-current-demographics.yaml`, `reference_materials/legislators-social-media.yaml`, `reference_materials/reps__comittees.yaml` – public YAML files with congressional demographics and social-media handles

---

### 3.2 YouTube comments

**`data/sentiment/raw_data/reps_yt_comments_2021_2025.csv`**
Comment-level dataset. One row = one YouTube comment on a video from an official House member’s channel (2021–2025).

Main variables:

* `rep.first_name`, `rep.last_name`, `rep.id_bioguide` – representative associated with the channel
* `rep_chamber` – chamber of Congress (here: House)
* `rep_party` – political party
* `rep_state` – represented state
* `channel_id` – YouTube channel ID
* `video_id` – YouTube video ID where the comment appears
* `comment_id` – unique ID for the comment
* `author` – username of the commenter
* `text` – **raw text** content of the comment
* `published_at` – timestamp of the comment (ISO 8601)
* `is_reply` – whether the comment is a reply (`TRUE`/`FALSE`)
* `parent_id` – parent comment ID if it is a reply

Additional variables created in the text-analysis pipeline (stored in intermediate files or notebooks, not necessarily written back to this CSV):

* `month` – year–month period of `published_at` (used for monthly aggregation)
* `tokens` – cleaned token list per comment (lowercased, alphabetic, stopwords removed, stemmed)
* `main_topic` – main LDA topic for the comment (integer 0–19)
* `main_topic_prob` – probability of the main LDA topic
* `topic_immigration_prob` – LDA probability mass assigned to the immigration topic
* `is_immigration_main` – indicator that `main_topic` is the immigration topic
* `sent_label`, `sent_score_raw`, `sent_score` – outputs from the sentiment model
* `hate_label`, `hate_score_raw`, `is_hate`, `hate_score` – outputs from the hate / toxicity model

Immigration-only and cleaned versions of these datasets are stored under `data/sentiment/clean_data/`.

---

### 3.3 Economic data (New York)

Economic inputs are stored in:

**`data/ECON/CPS data/`**

* `Unemployment.xlsx` – monthly New York unemployment rate
* `all_employees.xlsx` – monthly employment in New York
* `monthly_wage.xlsx` – average monthly wage
* `replace_GDP.xlsx` – macro indicator used as a GDP proxy (New York Fed Coincident Index, NYPHCI)
* `yearly_populationNY.xlsx` – annual New York population

**`data/ECON/CPS data/NY_CPS_noncitizen_share_monthly_2020_2025.xlsx`**
Monthly non-citizen share (main immigrant-share variable) computed from CPS.

> **Important:** the underlying CPS micro data file (`cps.dta`) is **not included** in the repository because it is too large and must be downloaded separately from the CPS / IPUMS source. The R code in `code/regression/cps.Rmd` assumes that a local copy of the CPS micro data is available but does not push it to GitHub.

Merged analysis dataset:

* `data/regression/data.xlsx` – monthly panel for New York containing immigrant share, employment, unemployment, wages, NYPHCI, COVID indicator, and the final hostility indices used in the regressions

---

### 3.4 Other files

* `report/pre_slides.pdf`, `report/proposal.docx` – project proposal and early slides
* `figures/ECON/economic_graphs.docx`, `figures/LDA/LDA graphs.docx` – collections of intermediate figures (economic time series, topic salience, hostility indices)
* `reference_materials/*` – background readings and notes, not used directly in the code
* System / housekeeping files: `.gitignore`, `.DS_Store`, and `*-checkpoint.ipynb` / `*-checkpoint.csv` can be ignored

---

## 4. Code: what each script / notebook does

### 4.1 Data construction and scraping

**`code/final_project_creating_dataset.ipynb`**

* Reads congressional YAML files and `df_legs_media.csv` from `reference_materials/`
* Cleans and merges metadata to create `data/sentiment/raw_data/df_reps_with_youtube.csv`
* Standardizes YouTube channel IDs and constructs `youtube_url`

**`code/analyzing_comments_new_env.ipynb`**

* Uses `df_reps_with_youtube.csv` and a local YouTube API key (not stored in the repo)
* Scrapes top-level comments from each representative’s channel for 2021–2025
* Parses timestamps, constructs `month`, flags replies vs. top-level comments
* Saves `data/sentiment/raw_data/reps_yt_comments_2021_2025.csv`

---

### 4.2 Topic modelling (LDA)

**`code/LDA/LDA relevant.ipynb`**

* Loads `data/sentiment/raw_data/reps_yt_comments_2021_2025.csv`

* Cleans text using NLTK:

  * lowercasing and tokenization
  * removing non-alphabetic tokens (numbers, URLs, emojis, punctuation)
  * removing English stopwords
  * stemming with Porter stemmer

* Builds a Gensim `Dictionary` from all token lists and filters extremes:

  * `no_below = 50`, `no_above = 0.5`
  * resulting vocabulary ≈ 7,800 distinct stemmed tokens

* Converts each comment to a bag-of-words vector (`doc2bow`)

* Fits a 20-topic LDA model with `LdaMulticore`:

  * `num_topics = 20`, multi-core inference
  * inspects top words for each topic and assigns human-readable labels
  * topic 0 is identified as the **Immigration** topic (`border`, `illegal`, `immigr`, etc.)

* For each comment, records:

  * full topic probability vector
  * `main_topic` = topic with highest probability
  * `main_topic_prob` = probability of that topic
  * `topic_immigration_prob` = probability assigned to Immigration topic (topic 0)
  * `is_immigration_main` = indicator that `main_topic` is Immigration

* Aggregates by `month` to obtain:

  * **overall distribution of main topics**
  * **ImmigrationMainShare** – share of comments whose main topic is immigration
  * **ImmigrationProbMean** – mean immigration topic probability across all comments

* Writes intermediate CSVs to `data/sentiment/clean_data/` and exports topic-related figures to `figures/LDA/`.

---

### 4.3 Sentiment and hate models (Hugging Face)

**`code/analyzing_immigraiton.ipynb`**

* Loads LDA outputs and identifies **immigration-focused comments**:

  * primarily comments with `main_topic == Immigration topic (0)`
  * after filtering, sample size ≈ 17,000 immigration comments

* Applies two pre-trained transformer models from Hugging Face:

  * an English RoBERTa-based sentiment model from **CardiffNLP** (positive / neutral / negative);
  * a multilingual toxicity / hate-speech model from **Unitary** (probability of toxic / hateful content).
    *(Exact model identifiers are specified at the top of the notebook.)*

* For each immigration comment, stores:

  * sentiment label and score: `sent_label`, `sent_score_raw`, `sent_score`
  * toxicity label and score: `hate_label`, `hate_score_raw`, `is_hate`, `hate_score`

* Aggregates to monthly measures:

  * number of immigration comments per month
  * share of negative immigration comments (`NegShare_t`)
  * share of hateful immigration comments (`HateShare_t`)

* Combines these with LDA-based salience measures:

  * conditional vs. unconditional shares (e.g., share of **all** comments that are both immigration-related and negative / hateful)
  * builds and visualizes monthly hostility time series (NegAll, HateAll, and composite indices)

* Saves exploratory figures to `figures/sentiment/`, including:

  * `n_negative_imm_com_over_time.png`
  * `n_hate_over_time.png`
  * `n_negative_over_time.png`
  * `party_share_anti_imm_comm.png`
  * `volume_vs_hate_share.png`

---

### 4.4 Hostility indices and regressions

**`code/regression/regression.Rmd` and `code/regression/regression2.Rmd`**

* Read:

  * monthly immigration topic measures from `data/sentiment/clean_data/` (ImmigrationMainShare, ImmigrationProbMean)
  * monthly sentiment / hate aggregates from `data/sentiment/clean_data/` (NegShare, HateShare)
  * New York economic panel from `data/regression/data.xlsx`

* Construct monthly hostility indices:

  * **NegAll(_t)** – share of all comments that are both immigration-related and negative
  * **HateAll(_t)** – share of all comments that are both immigration-related and hateful
  * standardize each series (z-scores)

* Build three composite **Immigration Hostility Indices**:

  * `Hostility_11 = z(NegAll) + 1 × z(HateAll)`
  * `Hostility_12 = z(NegAll) + 2 × z(HateAll)` (baseline)
  * `Hostility_13 = z(NegAll) + 3 × z(HateAll)`

* Merge hostility indices with economic variables to create the final monthly analysis dataset

* Estimate two sets of OLS models:

  1. **Stage 1 – Economic outcomes vs. immigration share**

     * unemployment, log wages, and log employment as dependent variables
     * regressors: non-citizen share, log NYPHCI, COVID dummy

  2. **Stage 2 – Hostility vs. economic conditions**

     * hostility indices as dependent variables
     * regressors: non-citizen share, log employment, log NYPHCI, COVID dummy
     * robustness checks with alternative hostility indices and salience measures

* Produce regression tables and HTML output (`code/regression/regression.html`, `code/regression/regression2.html`) used in the final report

---

### 4.5 CPS processing

**`code/regression/cps.Rmd`**  (if present)

* Processes CPS micro data from a **local file** (e.g., `cps.dta`) which is **not included in the repository** due to its size and licensing constraints.
* Restricts to New York State and relevant years
* Computes monthly non-citizen share for New York
* Exports `data/ECON/CPS data/NY_CPS_noncitizen_share_monthly_2020_2025.xlsx` and an HTML summary (`cps-data.html`)

To reproduce this part, each user must download CPS micro data separately (for example from IPUMS CPS), save it locally as `cps.dta` (or adjust the path in `cps.Rmd`), and then knit the R Markdown file.

---

## 5. Main outputs

### 5.1 Cleaned datasets

* `data/sentiment/raw_data/df_reps_with_youtube.csv` – cleaned representative-level metadata
* `data/sentiment/raw_data/reps_yt_comments_2021_2025.csv` – cleaned comment-level dataset
* `data/ECON/CPS data/NY_CPS_noncitizen_share_monthly_2020_2025.xlsx` – immigration share series for New York (derived from local CPS micro data, not pushed)
* `data/regression/data.xlsx` – final merged monthly panel used in regressions (economic variables + hostility indices)

Intermediate derived files (created inside notebooks, mostly under `data/sentiment/clean_data/`):

* topic-level CSVs (e.g., monthly topic shares, immigration salience series)
* monthly immigration sentiment / hate aggregates

---

### 5.2 Figures

Saved figures live in `figures/`:

* `figures/ECON/economic_graphs.docx` – economic time-series figures
* `figures/LDA/LDA graphs.docx` – topic-model figures
* `figures/sentiment/*.png` – sentiment & hate graphs, including:

  * `n_negative_imm_com_over_time.png`
  * `n_hate_over_time.png`
  * `n_negative_over_time.png`
  * `party_share_anti_imm_comm.png`
  * `volume_vs_hate_share.png`

---

### 5.3 Report and documentation

* `report/DS_report.pdf` – final project report
* `report/pre_slides.pdf` – presentation slides
* `code/regression/regression.html`, `code/regression/regression2.html` – detailed regression outputs

---

## 6. Reproducibility: recommended run order

To reproduce the main results (assuming access to a YouTube API key and CPS micro data):

1. **Build congressional metadata**

   * Run `code/final_project_creating_dataset.ipynb` to create `data/sentiment/raw_data/df_reps_with_youtube.csv`.

2. **Scrape YouTube comments**

   * Run `code/analyzing_comments_new_env.ipynb` (requires a valid YouTube API key) to create `data/sentiment/raw_data/reps_yt_comments_2021_2025.csv`.

3. **Topic modelling**

   * Run `code/LDA/LDA relevant.ipynb` to build the dictionary and corpus, fit the 20-topic LDA model, label topics, identify the immigration topic (topic 0), and export monthly topic-share and immigration-salience series to `data/sentiment/clean_data/` and `figures/LDA/`.

4. **Sentiment and hate analysis**

   * Run `code/analyzing_immigraiton.ipynb` on the immigration-topic comments to obtain monthly NegShare, HateShare, and other intermediate measures and save them under `data/sentiment/clean_data/` and `figures/sentiment/`.

5. **CPS processing (requires external CPS data)**

   * Download CPS micro data (e.g., from IPUMS CPS) and save it locally as `cps.dta` (or adjust the path in `code/regression/cps.Rmd`).
   * Run `code/regression/cps.Rmd` to compute the monthly New York non-citizen share and export `data/ECON/CPS data/NY_CPS_noncitizen_share_monthly_2020_2025.xlsx`.

6. **Merge data and run regressions**

   * Run `code/regression/regression.Rmd` / `code/regression/regression2.Rmd` to construct hostility indices, merge them with the New York economic panel (`data/regression/data.xlsx`), and estimate the Stage 1 and Stage 2 regression models.


