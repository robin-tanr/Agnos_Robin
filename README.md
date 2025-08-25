# Agnos Task 2 — Symptom Recommender (TH/EN)

Recommends the next **K** symptoms to ask/confirm from a patient’s **sex, age,** and **selected symptoms**. Short by design (**K=3**) with a brief reason per suggestion. *Triage assist only; not a diagnostic tool.*

## What’s here
- **Colab.ipynb** — end-to-end: load → normalize (TH/EN) → rules & item-CF → blend → evaluate → serve FastAPI.
- **REPORT.md** — concise write-up (problem, data, methods, results, takeaways).

## Run (Colab)
1. Open **Colab.ipynb** and **Run all**.
2. Place **AgnosDataset.csv** at the path printed by the notebook (or change `PROJ`).
3. When you see “Uvicorn running on …:8000”, open **http://127.0.0.1:8000/docs**.

## Test (Swagger)
- **GET `/health`** should return: `{"status":"ok","symptoms":150,"rules":36}`.
- **POST `/recommend`** → click **Try it out** and use:
  ```json
  {"age":28,"sex":"F","selected":["ไอ","น้ำมูกไหล"],"top_k":3}
