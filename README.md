# Skill Gap Predictor

A Flask-based career-readiness application that compares a learner's self-assessed skills with a structured target-career profile, explains the biggest gaps, and generates a personalized learning roadmap with career-specific projects.

> **Important:** the readiness score is an interpretable skill-alignment measure. It is **not** a probability of getting hired and has not been scientifically validated as an employment predictor.

## What it does

1. **Choose a target career** from 21 supported roles.
2. **Automatically load career requirements** including required skill level and importance.
3. **Self-assess each skill** on a 1–10 scale.
4. **Calculate readiness** using weighted skill alignment, giving Critical skills more influence.
5. **Explain the result** with overall readiness and Critical-skill alignment.
6. **Identify strengths and skill gaps**, ranked by priority.
7. **Generate a personalized roadmap** based on the learner's current levels and gaps.
8. **Recommend projects** that target the learner's actual gaps rather than showing a generic project list.

## Supported careers

The current career database covers 21 roles across data/AI, software development, cloud/infrastructure, cybersecurity, design, product/business, marketing, databases, QA, and networking.

Examples:

- Data Scientist
- Data Analyst
- Machine Learning Engineer
- AI Engineer
- Software Engineer
- Frontend Developer
- Backend Developer
- Full Stack Developer
- Mobile App Developer
- Data Engineer
- Cloud Engineer
- DevOps Engineer
- Cybersecurity Analyst
- Security Engineer
- UI/UX Designer
- Product Manager
- Business Analyst
- Digital Marketing Specialist
- Database Administrator
- QA Engineer
- Network Engineer

## Architecture

```text
Career Selection
      |
      v
Career Intelligence
  - skills
  - required levels
  - importance
      |
      v
Skill Self-Assessment
      |
      v
Reality Check
  - weighted readiness
  - critical-skill alignment
  - strengths
  - prioritized gaps
      |
      v
Personalized Roadmap
  - learning stage
  - practice guidance
  - career-specific projects
      |
      v
Career Readiness Report
```

### Core modules

| Module | Responsibility |
|---|---|
| `src/career_intelligence.py` | Loads and validates the authoritative career database |
| `src/career_data.py` | Compatibility facade for career data access |
| `src/skill_normalizer.py` | Maps aliases to canonical skill names |
| `src/reality_check.py` | Calculates readiness, strengths, gaps, and Critical-skill alignment |
| `src/skill_intelligence.py` | Stores skill descriptions, learning stages, and practice guidance |
| `src/project_intelligence.py` | Ranks projects against a learner's actual skill gaps |
| `src/roadmap.py` | Builds the five-phase personalized roadmap |
| `src/career_recommender.py` | Ranks careers by transparent skill alignment |
| `app.py` | Flask web application and request handling |

## Readiness calculation

For each skill:

```text
alignment = min(current_level / required_level, 1) × 100
```

The overall readiness score is a weighted average:

```text
Critical skill weight  = 1.5
Important skill weight = 1.0
```

Skill gaps are:

```text
gap = max(required_level - current_level, 0)
```

The application also calculates Critical-skill alignment separately so a strong overall score cannot hide a serious weakness in a high-priority skill.

### Readiness labels

| Score / condition | Label |
|---|---|
| < 40 | Major Gap |
| 40–59 | Early Foundation |
| 60–74 | Developing |
| 75–89 | Near Ready |
| 90+ | Strong Alignment |
| Severe Critical gaps | Critical Gaps |

## Project intelligence

Project recommendations are driven by:

```text
Target career
+ Current skill levels
+ Skill gaps
+ Skill importance
```

Projects are ranked by how strongly they address the learner's gaps. This means two learners targeting the same career can receive different project priorities.

## Data

### Raw datasets

The original datasets are stored under:

```text
data/raw/job_skill_set/all_job_post.csv
data/raw/vocational_skill_job/Skill_Job_Matching_Dataset.csv
```

### Authoritative career database

The web application uses:

```text
data/processed/career_intelligence.csv
```

This is the **single source of truth** for the career profiles used by the application.

Other derived career CSVs from earlier project iterations were removed to avoid duplicate sources of truth.

### Dataset notes

See [`DATASET_NOTES.md`](DATASET_NOTES.md) for the dataset context and processing notes.

## ML component

The repository also contains an **optional dataset-based ML experiment** based on the vocational skill/job-matching dataset.

The experiment:

- cleans the vocational dataset,
- engineers skill-gap features,
- one-hot encodes categorical features,
- trains a Random Forest classifier,
- reports accuracy, ROC-AUC, classification report, and confusion matrix.

This model is kept separate from the current career-readiness engine because the original dataset's target represents a different problem. The web application's readiness score should therefore not be described as the ML model's employment prediction.

### Run the optional ML experiment

```bash
python -m src.preprocess
python -m src.train_model
```

The generated model is written to `models/` and is intentionally ignored by Git because serialized scikit-learn models can be version-sensitive.

## Installation

Use Python **3.11 or 3.12** for the most predictable environment.

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd Skill_Gap_Predictor
```

### 2. Create a virtual environment

#### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocks activation:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Run the application

```bash
python app.py
```

Open the local Flask address shown in the terminal.

For development-only debug mode:

```bash
# Windows PowerShell
$env:FLASK_DEBUG="1"
python app.py
```

```bash
# macOS / Linux
FLASK_DEBUG=1 python app.py
```

Debug mode is disabled by default.

## Testing

Run the complete automated test suite:

```bash
python -m pytest -q
```

The test suite covers:

- career database integrity,
- career metadata and required levels,
- skill normalization and aliases,
- career recommendation ranking,
- skill-gap calculations,
- roadmap generation,
- project recommendations across all supported careers.

The final project should pass all tests before publication.

## Project structure

```text
Skill_Gap_Predictor/
├── app.py
├── README.md
├── DATASET_NOTES.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/
│   │   ├── job_skill_set/
│   │   │   └── all_job_post.csv
│   │   └── vocational_skill_job/
│   │       └── Skill_Job_Matching_Dataset.csv
│   └── processed/
│       ├── career_intelligence.csv
│       └── skill_job_cleaned.csv  # generated locally; ignored by Git
│
├── notebooks/
│   ├── 01_dataset_inspection.ipynb
│   └── 01_exploratory_data_analysis.ipynb
│
├── src/
│   ├── __init__.py
│   ├── career_data.py
│   ├── career_intelligence.py
│   ├── career_recommender.py
│   ├── create_career_database.py
│   ├── inspect_datasets.py
│   ├── preprocess.py
│   ├── project_intelligence.py
│   ├── reality_check.py
│   ├── roadmap.py
│   ├── skill_gap.py
│   ├── skill_intelligence.py
│   ├── skill_normalizer.py
│   └── train_model.py
│
├── static/
│   ├── css/style.css
│   └── js/app.js
│
├── templates/
│   ├── index.html
│   ├── assessment.html
│   ├── results.html
│   └── error.html
│
└── tests/
    ├── test_career_intelligence.py
    ├── test_career_recommender.py
    ├── test_end_to_end.py
    ├── test_skill_gap.py
    └── test_skill_normalizer.py
```

## Rebuilding generated data

The authoritative career database can be regenerated from the career definitions and skill intelligence:

```bash
python -m src.create_career_database
```

The command writes:

```text
data/processed/career_intelligence.csv
```

Do not edit the generated CSV manually when changing the career definitions; update `CAREER_PROFILES` / `CAREER_METADATA` and regenerate it.

## Limitations

- Skill levels are self-assessed, so results depend on honest and reasonably calibrated input.
- Career requirements are curated project data, not a live labor-market feed.
- The readiness score measures alignment with this project's defined skill profiles; it does not predict hiring outcomes.
- The optional Random Forest experiment solves the dataset's original job-match classification problem and is intentionally separate from the current readiness engine.
- Project recommendations are rule/data driven rather than learned from historical hiring outcomes.
- Serialized ML artifacts are not committed because scikit-learn versions can affect compatibility.

## Future improvements

- Add validated assessment questions instead of relying only on self-ratings.
- Version and evaluate career profiles against real job-posting data.
- Add explainable recommendation evidence and confidence indicators.
- Add user accounts and saved assessments.
- Add a deployment configuration and production WSGI server.
- Add CI with automated tests on every GitHub push.
- Add accessibility and browser-level end-to-end tests.
- Expand career coverage and make profiles easier to maintain as structured data.

## Portfolio talking points

This project demonstrates:

- Python and Flask application development
- Data preprocessing and exploratory analysis
- Machine learning model training and evaluation
- Feature engineering
- Rule-based / weighted recommendation logic
- Skill normalization
- Personalized roadmap generation
- Career-specific project ranking
- Error handling and automated testing
- Clean separation between experimental ML and the production application

## License

Add the license required by your course, organization, or dataset terms before publishing. Also verify the redistribution terms of the included source datasets before making the repository public.
