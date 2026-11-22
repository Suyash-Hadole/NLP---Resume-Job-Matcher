# Demo Guide

A practical walkthrough for presenting the project and answering viva questions. The demo works with **no Gemini key**; the AI card simply explains that cloud features are disabled while every local feature continues to work.

## Assessment criterion mapping

| Criterion | Where to demonstrate it | Evidence to show |
|---|---|---|
| **1. Information extraction and vectorisation** | The single matching dashboard | NER/profile table, extracted-text preview, semantic + TF-IDF scores, skill gap; extraction benchmark API |
| **2. Sentiment analysis and information extraction** | `notebooks/sentiment_analysis.ipynb` + dashboard NER | VADER scores, charts, 36-case evaluation, accuracy 0.917, macro-F1 0.916, confusion matrix; structured extraction in Criterion 1 |
| **3A. Topic modelling with LDA** | `notebooks/topic_modelling.ipynb` | 15-document corpus, 4 LDA topics, keyword weights, document-topic heatmap, perplexity 1063.19 |
| **3B. Gemini integration** | Gemini card inside results + header chips | Explicit consent, redaction counts, schema-validated response, classified errors; local features work with no key |
| **4. Gemini career assistant** | Four task cards in the results card | Match explanation, resume improvement, bullet suggestions, interview preparation; scores unchanged |

---

## 1. Pre-demo checklist

```powershell
cd D:\resume-job-matcher
.\.venv\Scripts\Activate.ps1
python -m pytest -q
python app.py
```

Open http://127.0.0.1:5000.

- [ ] Status pill says backend ready and three chips are visible.
- [ ] Chips read spaCy NER ready, Embeddings loaded/lazy/fallback, Gemini optional or configured.
- [ ] There is **one** dashboard - no navigation tabs.
- [ ] `/api/health` returns JSON with `status: "ok"`.
- [ ] Both notebooks are open with saved outputs.

If the port is busy: `$env:PORT=5055; python app.py`.

---

## 2. Recommended 10-minute run sheet

| Time | Area | What to show | What to say |
|---|---|---|---|
| 1 min | Header | Status pill, three chips, blue/white layout | "One focused matching app; local NLP is default, Gemini is optional." |
| 3 min | Role + upload + report | Load sample resume, choose Data Scientist and Pune, click Analyze and rank | "NER, MiniLM embeddings, TF-IDF baseline and skill gap all produce this report." |
| 2 min | Report details | NER table, extracted-text preview, matched/missing skills, raw cosines | "Every number comes from the uploaded resume and the entered job." |
| 1 min | Job links | Show five Explore Job Opportunities links; change location to Remote | "These are real external searches, not invented vacancies." |
| 1 min | Notebooks | Show sentiment confusion matrix and LDA topic charts | "Sentiment and topics are separate executed subject notebooks." |
| 1 min | Gemini card | Without a key show 'not configured'; with a key run one consented task | "Consent is mandatory, PII is redacted, output is advisory only." |
| 1 min | Wrap-up | `pytest` result and RUBRIC_MAPPING.md | "157 tests; no live API needed." |

---

## 3. Copy-paste demo inputs

### Resume (save as `priya.txt` or use Load sample resume)

```text
Priya Sharma
priya.sharma@example.com | +91 98765 43210 | github.com/priyasharma
Data Scientist with 5 years of experience building machine learning models.
Skills: Python, SQL, pandas, NumPy, scikit-learn, XGBoost, Tableau, Apache Spark, statistics, A/B testing
Experience
- Built predictive models with scikit-learn and XGBoost and ran A/B testing experiments.
- Created Tableau dashboards that tracked model performance for business teams.
Education
B.Tech in Computer Science, Pune University
```

### Job description

```text
We are hiring a Data Scientist to use Python, pandas, NumPy, scikit-learn, SQL and Tableau for data analysis, statistics, machine learning and A/B testing. Experience with Apache Spark and AWS is a plus.
```

### Role and location demo

- Type job title: `AI Engineer`; location: `Pune` - show the five links.
- Change location to `Remote` or `India` - links refresh with the new encoded query.

---

## 4. Step-by-step script

### A. Matching dashboard

1. Select the **Data Scientist** sample role (or type your own title and paste a description).
2. Set location to **Pune**.
3. Click **Load sample resume** (or upload 1-3 PDF/DOCX/TXT files).
4. Click **Analyze and rank**.
5. Point out:
   - ranking with overall, semantic and TF-IDF scores plus raw cosines;
   - matched and missing skills with high-priority flags;
   - NER and extracted candidate information inside the result card;
   - extracted resume text preview;
   - fallback warning if MiniLM or spaCy NER is unavailable.

**Talking point:** sentiment and Gemini are absent from this calculation by design. Ranking is deterministic.

### B. Explore Job Opportunities

1. Show the section beneath the report.
2. Explain each link is an external search URL containing the entered title and location.
3. Click one link (optional) to show it opens a real job-search page in a new tab.
4. Emphasise the disclaimer: this app does not claim any specific vacancy exists.

### C. Notebooks

1. Open `notebooks/sentiment_analysis.ipynb` and show the saved confusion matrix and metrics.
2. Open `notebooks/topic_modelling.ipynb` and show the four topics, keyword bars and document-topic heatmap.
3. Mention both datasets are embedded so the notebooks run independently.

### D. Gemini card (inside results)

**Without a key:** show the status card and confirm local matching still works.

**With a key:**

1. Select a resume (defaults to top-ranked).
2. Tick the consent checkbox.
3. Run **Match explanation**; show summary, strengths, gaps and redaction counts.
4. Run **Bullet suggestions** on a resume with existing bullets; show that only existing bullets are rewritten.
5. Show `advisory_only` and `does_not_change_scores` in the response.

**Verified live:** a real key in the local `.env` file (ignored by Git) was used; `/api/ai/test` and all four consented tasks returned HTTP 200 with schema-valid output, with 1 email, 1 phone, 1 URL and 1 name redacted before sending. Requests use a 45-second per-model timeout. If `gemini-3.8-flash` is overloaded, unavailable, rate-limited or times out, the app automatically falls back to `gemini-3.6-flash`, then `gemini-3.5-flash`, `gemini-3.1-flash-lite`, `gemini-2.5-flash` or `gemini-2.0-flash`.

**Talking point:** consent is checked before any text is prepared or sent; without it the API returns 400 and Gemini is never called.

---

## 5. Viva questions and model answers

**Q: Why VADER instead of a trained classifier?** A: It is transparent, needs no training data and gives interpretable scores. The notebook still evaluates it on a balanced labelled set, so limitations are measured.

**Q: Why report macro-F1 as well as accuracy?** A: Macro-F1 averages per-class F1 equally, so a majority class cannot hide weak classes. The confusion matrix shows exact errors.

**Q: Why LDA with CountVectorizer?** A: LDA is a generative model for word counts; CountVectorizer supplies counts. TF-IDF is used for similarity, not as LDA input.

**Q: How do job links avoid fake vacancies?** A: The API only builds encoded search URLs for five known platforms. It never scrapes or stores listings, and the UI labels them external searches.

**Q: How do you stop Gemini inventing experience?** A: Prompts forbid new facts; context is limited to local analysis and a redacted excerpt; schemas validate output; bullet rewrites must match existing bullets and cannot add numbers.

**Q: How do you defend against prompt injection?** A: Untrusted text is wrapped and bracket-neutralised, and system prompts explicitly treat it as data. Validation is a second defence.

**Q: Does Gemini change ranking?** A: No. Scores are computed first and `does_not_change_scores` is true.

**Q: What leaves the machine?** A: Nothing unless consent is given. Even then, emails, URLs, phones and names are redacted and only a length-limited excerpt is sent.

---

## 6. Troubleshooting

| Symptom | Quick fix |
|---|---|
| Status pill red | Check terminal traceback; confirm server running |
| spaCy chip blank pipeline | `python -m spacy download en_core_web_sm`, restart |
| Embeddings chip fallback | No MiniLM download; check internet/proxy. Matching still works |
| Gemini card not configured | Expected without key; local features work |
| Job links look wrong | Verify title/location fields; links reflect exact input |
| Tests fail | Run from `D:\resume-job-matcher` with the venv active |

---

## 7. Closing summary (30 seconds)

"The system keeps the original deterministic matching pipeline and presents it in one focused dashboard: NER, MiniLM embeddings, TF-IDF, skill gap, ranking, extracted information and external job-search links. Sentiment and LDA are separate executed notebooks with real metrics and charts. Gemini is an optional, consent-gated card whose advisory output never changes local scores. The full suite passes with 157 tests and needs no API key."
