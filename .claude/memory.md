# CBU - Qarz to'lash xavfini bashorat qilish (ML)

## Maqsad
Machine Learning bilan mijozlarning qarz to'lash qobiliyatini bashorat qilish (Credit Default Prediction). LightGBM model.

## Tech Stack
- **Language:** Python
- **ML:** LightGBM, scikit-learn
- **Data:** pandas, numpy
- **Visualization:** matplotlib, seaborn
- **Environment:** Jupyter Notebook

## Arxitektura
```
cleaned_data/
  join_datasets.ipynb   — 6 ta dataset birlashtirish
for_model/
  model.ipynb           — LightGBM training
result/
  results.csv           — Bashoratlar (customer_id, prob, default)
  analiz.ipynb          — Natijalar tahlili
```

## Muhim logika
- 6 ta dataset ni customer_ref bo'yicha LEFT JOIN
- 53 feature, 1 target (default/no-default)
- Class imbalance: class weights bilan hal
- Optimal threshold: 0.740, F1: 0.3174
- ROC-AUC: 0.8077, Accuracy: 0.9111
- Data formatlar: CSV, PARQUET, JSON, XML, XLSX
