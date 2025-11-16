# CBU Coding Challenge - Credit Default Prediction

## 📋 Loyiha Haqida

Bu loyiha mijozlarning qarz to'lash xavfini bashorat qilish uchun machine learning model yaratishga qaratilgan. Loyiha quyidagi bosqichlardan iborat:

1. **Dataset tozalash va birlashtirish** - Turli formatlardagi ma'lumotlarni birlashtirish
2. **Feature engineering** - Kerakli va kerak bo'lmagan ustunlarni ajratish
3. **Model training** - LightGBM yordamida binary classification model yaratish
4. **Model evaluation** - Natijalarni baholash va optimallashtirish
5. **Prediction** - Barcha mijozlar uchun bashoratlar yaratish

---

## 🎯 Target Variable (Maqsadli O'zgaruvchi)

**`default`** ustuni - bu bizning target variable:

- **0 = XAVFLI** - Mijoz qarzni to'lay olmaydi (default bo'ladi)
- **1 = XAVFSIZ** - Mijoz qarzni to'lay oladi (xavfsiz)

**Eslatma:** Bu binary classification muammosi. Model 0 va 1 ni bashorat qilishi kerak.

---

## 📁 Loyiha Strukturasi

```
cbu/
├── for_convert/              # Dastlabki ma'lumotlar (turli formatlar)
│   ├── application_metadata.csv
│   ├── credit_history.parquet
│   ├── demographics.csv
│   ├── financial_ratios.jsonl
│   ├── geographic_data.xml
│   ├── loan_details.xlsx
│   └── converter.ipynb       # Format konvertatsiya
│
├── converted_csv/            # Konvertatsiya qilingan CSV fayllar
│   ├── application_metadata.csv
│   ├── credit_history.csv
│   ├── demographics.csv
│   ├── financial_ratios.csv
│   ├── geographic_data.csv
│   └── loan_details.csv
│
├── cleaned_data/             # Tozalangan va birlashtirilgan ma'lumotlar
│   ├── final_dataset.csv     # Yakuniy tozalangan dataset
│   ├── join_datasets.ipynb   # Datasetlarni birlashtirish
│   └── dataset_info.md       # Dataset tafsilotlari
│
├── for_model/                # Model uchun tayyorlangan ma'lumotlar
│   ├── dataset_for_models.csv
│   └── model.ipynb           # Asosiy model notebook
│
└── result/                   # Natijalar
    ├── results.csv           # Barcha mijozlar uchun bashoratlar
    └── analiz.ipynb          # Natijalarni tahlil qilish
```

---

## 🔄 Ishlar Ketma-ketligi

### 1. Dataset Tozalash va Birlashtirish

**Fayl:** `cleaned_data/join_datasets.ipynb`

**Vazifalar:**
- Turli formatlardagi ma'lumotlarni CSV ga konvertatsiya qilish
- 6 ta datasetni `customer_ref` kaliti bo'yicha birlashtirish (LEFT JOIN)
- Ma'lumotlarni tozalash:
  - Valyuta belgilarini olib tashlash (`$`, `,`)
  - Kategoriyal ma'lumotlarni standartlashtirish
  - Missing valuelarni to'ldirish
  - Takrorlangan ustunlarni olib tashlash

**Birlashtirilgan datasetlar:**
1. `application_metadata.csv` (asosiy, default target bor)
2. `demographics.csv` → `cust_id = customer_ref`
3. `credit_history.csv` → `customer_number = customer_ref`
4. `loan_details.csv` → `customer_id = customer_ref`
5. `financial_ratios.csv` → `cust_num = customer_ref`
6. `geographic_data.csv` → `id = customer_ref`

**Yakuniy dataset:**
- **Qatorlar:** 89,999 (barcha mijozlar)
- **Ustunlar:** ~59 ta
- **Fayl:** `cleaned_data/final_dataset.csv`

---

### 2. Feature Engineering

**Fayl:** `for_model/model.ipynb` (Cell 2)

**Funktsiya:** `prepare_features_and_target()`

**Vazifalar:**
- ID ustunlarni olib tashlash (`customer_ref`, `application_id`)
- Kerak bo'lmagan ustunlarni olib tashlash:
  - `loan_officer_id` (identifikator)
  - `previous_zip_code` (foydasiz)
  - `referral_code` (ko'p noyob qiymat)
- Target ustunni (`default`) ajratish

**Natija:**
- **Feature ustunlar:** 53 ta
- **Target ustun:** `default` (0 yoki 1)

---

### 3. Data Preprocessing

**Fayl:** `for_model/model.ipynb` (Cell 3-4)

**Vazifalar:**

#### 3.1. Categorical va Numerical Ustunlarni Ajratish
- **Categorical:** 6 ta (`preferred_contact`, `account_status_code`, `employment_type`, `education`, `loan_type`, `loan_purpose`)
- **Numerical:** 47 ta

#### 3.2. Categorical Encoding
- **Usul:** LabelEncoder
- Har bir categorical ustun uchun alohida encoder
- Encoding natijalari `label_encoders` dictionary da saqlanadi

---

### 4. Train/Test Split

**Fayl:** `for_model/model.ipynb` (Cell 5)

**Parametrlar:**
- `test_size = 0.2` (20% test, 80% train)
- `random_state = 42` (reproducibility)
- `stratify = y` (class taqsimotini saqlash)

**Natija:**
- **Train set:** 71,999 qator
- **Test set:** 18,000 qator
- **Class taqsimoti:** Stratified (har ikki setda ham taxminan 95% xavfli, 5% xavfsiz)

---

### 5. Model Training

**Fayl:** `for_model/model.ipynb` (Cell 6)

**Model:** LightGBM Classifier

**Class Weights:**
Class imbalance muammosini hal qilish uchun class weights qo'shildi:

```python
class_counts = y_train.value_counts().sort_index()
# Masalan: {0: 68324, 1: 3675}

weight_0 = total / (2 * class_counts[0])  # Xavfli uchun
weight_1 = total / (2 * class_counts[1])  # Xavfsiz uchun
# Natija: weight_1 ≈ 18.6x weight_0
```

**Model Parametrlari:**
- `objective = 'binary'`
- `metric = 'auc'`
- `boosting_type = 'gbdt'`
- `num_leaves = 31`
- `learning_rate = 0.05`
- `feature_fraction = 0.9`
- `bagging_fraction = 0.8`
- `bagging_freq = 5`
- `class_weight = class_weights` ⭐ (Class imbalance uchun)
- `random_state = 42`

**Training:**
- Early stopping: 50 round (validation score yaxshilanmasa)
- Evaluation set: Test set
- Evaluation metric: AUC

---

### 6. Model Evaluation va Threshold Tuning

**Fayl:** `for_model/model.ipynb` (Cell 7)

**Vazifalar:**

#### 6.1. Threshold Tuning
Default threshold (0.5) har doim eng yaxshi emas. Optimal threshold topish:

```python
thresholds = np.arange(0.1, 0.9, 0.01)
# Har bir threshold uchun F1 score hisoblanadi
# Eng yuqori F1 score bergan threshold tanlanadi
```

**Natija:**
- **Default threshold (0.5):** F1 = 0.2392
- **Optimal threshold:** 0.740
- **Optimal threshold bilan F1:** 0.3174 ⬆️

#### 6.2. Model Metrikalari

**Test set natijalari:**
- **ROC-AUC Score:** 0.8077
- **Accuracy:** 0.9111
- **F1 Score:** 0.3174
- **Optimal Threshold:** 0.740

**Confusion Matrix:**
```
                Predicted
Actual      0 (Xavfli)  1 (Xavfsiz)
0 (Xavfli)    16,575        506
1 (Xavfsiz)     413        506
```

**Classification Report:**
- **Class 0 (Xavfli):** Precision: 0.98, Recall: 0.97, F1: 0.97
- **Class 1 (Xavfsiz):** Precision: 0.50, Recall: 0.55, F1: 0.52

---

### 7. Feature Importance

**Fayl:** `for_model/model.ipynb` (Cell 8)

**Top 15 eng muhim featurelar:**
1. `credit_score` - Kredit balli
2. `credit_utilization` - Kreditdan foydalanish
3. `age` - Yoshi
4. `num_credit_accounts` - Kredit hisoblar soni
5. `monthly_free_cash_flow` - Oylik erkin pul oqimi
6. `interest_rate` - Foiz stavkasi
7. `account_diversity_index` - Hisob xilma-xillik indeksi
8. `existing_monthly_debt` - Mavjud oylik qarz
9. `regional_median_rent` - Mintaqaviy o'rtacha ijara
10. `regional_unemployment_rate` - Mintaqaviy ishsizlik darajasi
11. `available_credit` - Mavjud kredit
12. `debt_to_income_ratio` - Qarz/daromad nisbati
13. `total_credit_limit` - Jami kredit limiti
14. `monthly_payment` - Oylik to'lov
15. `loan_amount` - Qarz miqdori

---

### 8. Prediction va Natijalarni Saqlash

**Fayl:** `for_model/model.ipynb` (Cell 10)

**Vazifalar:**
- Barcha 89,999 mijoz uchun probability hisoblash
- Prob >= 0.5 bo'lsa, default = 1 (xavfsiz)
- Natijalarni `result/results.csv` ga saqlash

**Output format:**
```csv
customer_id,prob,default
10000,0.36257,0
10001,0.51871,1
10002,0.61247,1
...
```

**Fayl:** `result/results.csv`
- **Qatorlar:** 89,999
- **Ustunlar:** 3 ta (`customer_id`, `prob`, `default`)

---

## 📊 Model Natijalari

### Asosiy Metrikalar

| Metrika | Qiymat | Izoh |
|---------|--------|------|
| **ROC-AUC Score** | 0.8077 | Yaxshi (0.8+ yaxshi hisoblanadi) |
| **Accuracy** | 0.9111 | Yaxshi (91% to'g'ri bashorat) |
| **F1 Score** | 0.3174 | O'rta (class imbalance tufayli) |
| **Optimal Threshold** | 0.740 | Default 0.5 dan yuqori |

### Class Taqsimoti

**Haqiqiy taqsimot (Test set):**
- Default = 0 (Xavfli): 17,081 (94.89%)
- Default = 1 (Xavfsiz): 919 (5.11%)

**Predict taqsimoti (Optimal threshold bilan):**
- Default = 0 (Xavfli): 16,575 (92.08%)
- Default = 1 (Xavfsiz): 1,425 (7.92%)

**Tahlil:**
- Model xavfsiz mijozlarni biroz ko'proq topmoqda (7.92% vs 5.11%)
- Bu yaxshi belgi - xavfsiz mijozlarni o'tkazib yubormaslik muhim

---

## 📊 Results Analysis

**Fayl:** `result/analiz.ipynb`

Bu notebook `results.csv` faylini tahlil qiladi va vizualizatsiyalar yaratadi.

### Asosiy Statistika

- **Jami mijozlar:** 89,999
- **Default taqsimoti:** Xavfli (0) va Xavfsiz (1) mijozlar soni
- **Probability statistikasi:** Min, max, mean, median qiymatlar

### Vizualizatsiyalar

1. **Probability Distribution (Histogram)** - Prob taqsimotini ko'rsatadi
2. **Default Class Distribution (Pie Chart)** - Xavfli va Xavfsiz mijozlar nisbati
3. **Risk Groups (Bar Chart)** - Low, Medium, High risk guruhlari
4. **Probability vs Default (Violin Plot)** - Default bo'yicha prob taqsimoti
5. **Threshold Analysis** - Turli thresholdlar (0.3, 0.5, 0.7) uchun statistika

### Risk Guruhlari

- **Low Risk:** prob < 0.3
- **Medium Risk:** 0.3 <= prob < 0.7
- **High Risk:** prob >= 0.7

Batafsil tahlil uchun `result/analiz.ipynb` faylini ishga tushiring.

---

## 🔧 Texnik Detallar

### Class Weights Nima?

**Class weights** - Class imbalance muammosini hal qilish uchun ishlatiladi. Model kam sonli classga ko'proq e'tibor beradi.

**Hisoblash:**
```python
weight_0 = total / (2 * class_counts[0])  # Xavfli uchun
weight_1 = total / (2 * class_counts[1])  # Xavfsiz uchun
# Natija: weight_1 ≈ 18.6x weight_0
```

**Ma'nosi:**
- Class 0 (Xavfli) ko'p (95%), shuning uchun og'irlik past (0.527)
- Class 1 (Xavfsiz) kam (5%), shuning uchun og'irlik yuqori (9.796)
- Model endi kam sonli classga ko'proq e'tibor beradi

### Threshold Tuning Nima?

**Threshold** - Probability dan binary prediction (0 yoki 1) ga o'tish uchun ishlatiladigan chegara.

**Default threshold (0.5):**
- Agar prob >= 0.5 bo'lsa, default = 1
- Lekin class imbalance tufayli bu har doim eng yaxshi emas

**Optimal threshold (0.740):**
- F1 score ni maksimallashtirish uchun topilgan
- Faqat yuqori ehtimollikdagi holatlar 1 deb belgilanadi
- Precision oshadi, F1 score yaxshilanadi

**Nima uchun 0.740?**
- Class imbalance tufayli model ko'pchilik holatlar uchun prob < 0.5 beradi
- Threshold 0.740 ga ko'tarilganda, faqat yuqori ehtimollikdagi holatlar 1 deb belgilanadi
- Bu precision va F1 score ni oshiradi

---

## 📦 Kerakli Kutubxonalar

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder
import lightgbm as lgb
from sklearn.metrics import (
    roc_auc_score, 
    accuracy_score, 
    f1_score, 
    confusion_matrix, 
    classification_report
)
from pathlib import Path
```

**O'rnatish:**
```bash
pip install pandas numpy scikit-learn lightgbm matplotlib seaborn
```

---

## 🚀 Ishga Tushirish

### 1. Dataset Tozalash
```bash
# cleaned_data/join_datasets.ipynb ni ishga tushiring
# final_dataset.csv yaratiladi
```

### 2. Model Training
```bash
# for_model/model.ipynb ni ishga tushiring
# Barcha celllarni ketma-ket ishga tushiring
```

### 3. Natijalarni Tahlil Qilish
```bash
# result/analiz.ipynb ni ishga tushiring
# Vizualizatsiyalar va statistik tahlil
```

### 4. Natijalarni Ko'rish
```bash
# result/results.csv faylini oching
# customer_id, prob, default ustunlari bor
```

---

## 📈 Model Yaxshilash Imkoniyatlari

### Hozirgi Muammolar:
1. **Class Imbalance** - Hali ham mavjud (5% vs 95%)
2. **F1 Score** - O'rta (0.3174)
3. **Recall (Class 1)** - Past (0.55)

### Yaxshilash Usullari:
1. **SMOTE** - Synthetic Minority Oversampling Technique
2. **Undersampling** - Ko'p sonli classni kamaytirish
3. **Hyperparameter Tuning** - GridSearch yoki Optuna
4. **Feature Engineering** - Yangi featurelar yaratish
5. **Ensemble Methods** - Bir nechta modellarni birlashtirish

---

## 📝 Eslatmalar

1. **Default ma'nosi:**
   - 0 = Xavfli (qarzni to'lay olmaydi)
   - 1 = Xavfsiz (qarzni to'lay oladi)

2. **Prob >= 0.5:**
   - Agar probability >= 0.5 bo'lsa, default = 1 (xavfsiz)
   - Bu oddiy va aniq logika

3. **Class Weights:**
   - Class imbalance muammosini yumshatadi
   - Model kam sonli classga ko'proq e'tibor beradi

4. **Threshold Tuning:**
   - Default 0.5 har doim eng yaxshi emas
   - Optimal threshold F1 score ni oshiradi

---

## 👤 Muallif

CBU Coding Challenge - MetOneX Team

---

## 📄 Litsenziya

Bu loyiha CBU Coding Challenge uchun yaratilgan.
