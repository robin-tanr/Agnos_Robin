# Agnos Task 2 — Symptom Recommender

**Goal**  
Recommend the **next K symptoms** to ask/confirm from a patient’s **sex, age**, and current **selected symptoms**.  
**Use:** Triage assist only. **Not** a diagnostic tool.  
**Defaults:** **K = 3**, **α = 0.7** (rules-heavy blend).

---

## 1) Data

- ~1,000 rows: `gender`, `age`, `summary (JSON)`, `search_term`.
- From `summary` we read: `yes_symptoms`, `no_symptoms`, `idk_symptoms`.
- After cleaning: **991** baskets (rows kept), ~**150** unique symptoms.
- **Cold-start segments:** `sex ∈ {M, F, U}` × `age_band ∈ {0–12, 13–19, 20–40, 41–60, 60+}`.

---

## 2) Preprocess (TH–EN normalize + clean)

Make all terms consistent in Thai and remove non-symptom tokens.

- **EN→TH synonyms (examples):**  
  `cough→ไอ`, `fever→ไข้`, `sputum/phlegm→เสมหะ`, `rhinorrhea/postnasal drip→น้ำมูกไหล`,  
  `stuffy nose→คัดจมูก`, `wheezing→หายใจมีเสียงหวีด`, `shortness of breath→หายใจลำบาก`,  
  `night cough→ไอกลางคืน`, `nosebleed→เลือดกำเดาไหล`, `skin rash→ผื่น`.
- **Thai aliases** for common phrases.
- **Split compounds:** e.g., `"น้ำมูกไหลไอ" → [น้ำมูกไหล, ไอ]`, `"ปวดหัวปวดท้ายทอย" → [ปวดหัว, ปวดท้ายทอย]`.
- **Strip prefixes/connectors:** remove `มี…`; split by `, + | และ กับ ร่วมกับ`.
- **Remove admin tokens:** e.g., `การรักษาก่อนหน้า / previous treatment`, `แพ้ยา`, `โรคประจำตัว`.

Artifacts: `symptom_map.json`, `stop_terms.txt`.

---

## 3) Models

1) **Popularity (Baseline-0)**  
   - Global Top-K.  
   - Segment Top-K (sex × age_band) for cold-start.

2) **Association Rules (Baseline-1, FP-Growth, 1⇒1)**  
   - Gives reason: **“A ⇒ B (lift x.xx)”**.  
   - After cleaning: about **36** useful rules.

3) **Item-based CF (v1)**  
   - One-hot baskets; **cosine** similarity.  
   - Score by similarity to selected symptoms.

4) **Blend**  
   - Final score = **α·RuleScore + (1−α)·CFScore**, with **α = 0.7**.  
   - **Why α=0.7:** rules give clear reasons; CF improves coverage.  
   - **Why K=3:** average positives ~1.9 → short list is easier to use.

---

## 4) Evaluation

Two protocols; metrics **Precision, Recall, MAP, Coverage** at K ∈ {3,5,7,10}.

- **(A) Leave-one-out (LOO):** rows with ≥2 yes; hide 1.  
- **(B) Beyond-search:** input = `search_term`; ground truth = `yes − search`.

**Key numbers (this run):**

| Protocol | MAP@5 | Coverage@5 |
|---|---:|---:|
| **LOO** | **0.674** | **0.760** |
| **Beyond** | **0.436** | — |

**Reading the result**  
LOO is strong (good when some symptoms are already selected).  
Beyond improved after normalization, but still limited by **vocabulary/alias coverage**; changing α (0.5/0.6/0.7) had little effect.

---

## 5) Example

**Input:** `sex=F, age=28, selected=["ไอ","น้ำมูกไหล"], K=3`  
**Typical output:**
- `เสมหะ` — reason: **ไอ ⇒ เสมหะ** (lift ~2.x)  
- `คัดจมูก` — reason: **น้ำมูกไหล ⇒ คัดจมูก** (lift ~2.x)  
- `เจ็บคอ` — reason: item-CF similar to **ไอ**

Cold-start (no selections): use segment/global popularity.

---

## 6) Serving (FastAPI)

- `GET /health` → `{"status":"ok","symptoms":150,"rules":36}` (typed response).  
- `POST /recommend` → TH/EN accepted; returns normalized `input`, Top-K with `{symptom, score, why}`, plus disclaimer.  
- `GET /app` mini UI; `/` redirects to `/docs`.  
- Colab-friendly; can expose with ngrok (token via ENV) or Cloudflare Tunnel.

---

## 7) What works / What to improve

**Works well**
- Explainable results (rules) + coverage (CF).  
- TH/EN input; output in Thai canonical.  
- Cold-start via segment popularity.  
- Short list (K=3) fits triage flow.

**Next steps**
1) Add more high-impact aliases/phrases (close the **Beyond** gap).  
2) Keep stop-term list updated.  
3) Optional: light re-ranker over rule/CF candidates; per-segment α; adaptive K.

---

## 8) Repro

- Run **Colab.ipynb** end-to-end (data → preprocess → models → eval → API).  
- Defaults: **α=0.7**, **K=3**.  
- API returns a disclaimer in responses.

---

*Disclaimer: Not for diagnosis; triage assist only.*
