# Agnos Task 2 — Symptom Recommender

Given a patient’s **sex, age** and **selected symptoms**, the model recommends the **next K symptoms** to ask/confirm.  
Short list by design (default **K=3**) and each suggestion includes a **reason** (rule/CF).  
**Scope:** triage assist only — **not** a diagnostic tool.

---

## What’s in this repo
- **Colab.ipynb** — end-to-end notebook: load → normalize (TH/EN) → build rules & item-CF → blend → evaluate → serve FastAPI (`/docs`, `/health`, `/recommend`).
- **REPORT.md** — 2–3 pages with Problem → Data → Methods → Results → So-what → Risks/Next.

> **Data** is not checked in. The notebook expects `AgnosDataset.csv`.

---

## Run

1. Open **Colab.ipynb** in Google Colab.
2. Put **`AgnosDataset.csv`** at the path printed by the notebook (default `/content/drive/MyDrive/AgnosHealth/`).  
   - The notebook also falls back to the repo folder; you can place the CSV next to the notebook.
3. **Run all cells** until you see: `Uvicorn running on http://0.0.0.0:8000`.
4. Open `http://127.0.0.1:8000/docs` (Swagger).  
   - To share publicly, use one of the safe cells in the notebook:
     - **ngrok** (set env var):
       ```python
       import os, getpass
       os.environ["NGROK_AUTHTOKEN"] = getpass.getpass("Paste ngrok token: ")
       ```.

---

## API (FastAPI)

- **GET `/health`** → `{"status":"ok","symptoms":150,"rules":36}` (typed response model).
- **POST `/recommend`**  
  Request:
  ```json
  {"age": int?, "sex": "M|F|U", "selected": ["symptom", "..."], "top_k": 3}
