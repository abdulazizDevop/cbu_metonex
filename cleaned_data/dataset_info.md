# Dataset Documentation

## CRITICAL: Default Column Meaning

**default** ustuni - bu bizning target variable (maqsadli o'zgaruvchi):

- **0 = XAVFLI** - Mijoz qarzni to'lay olmaydi (default bo'ladi)
- **1 = XAVFSIZ** - Mijoz qarzni to'lay oladi (xavfsiz)

**Eslatma:** Bu odatdagi binary classification muammosi. Model 0 va 1 ni bashorat qilishi kerak.

---

## 1. application_metadata.csv (14 ustun)

**Asosiy kalit:** `customer_ref` (10000-99999)

| Ustun | Ma'lumot turi | Tavsif | Model uchun ahamiyati | Tozalash kerakmi |
|-------|---------------|--------|----------------------|------------------|
| customer_ref | int64 | Mijoz referens raqami (asosiy kalit) | ⭐⭐⭐ Kalit ustun | ❌ Yo'q |
| application_id | int64 | Arizaning unikal ID raqami | ⭐ Past (faqat identifikator) | ❌ Yo'q |
| application_hour | int64 | Arizaning soati (0-23) | ⭐⭐ O'rta (vaqt tahlili) | ❌ Yo'q |
| application_day_of_week | int64 | Hafta kuni (0-6) | ⭐⭐ O'rta (vaqt tahlili) | ❌ Yo'q |
| account_open_year | int64 | Hisob ochilgan yil | ⭐⭐ O'rta (mijoz sadoqati) | ❌ Yo'q |
| preferred_contact | object | Afzal aloqa usuli (Mail/Phone/Email) | ⭐⭐ O'rta | ✅ Kategoriyal kodlash |
| referral_code | object | Referal kodi | ⭐ Past (ko'p noyob qiymat) | ⚠️ Ehtimol o'chirish |
| account_status_code | object | Hisob holati kodi | ⭐⭐ O'rta | ✅ Kategoriyal kodlash |
| random_noise_1 | float64 | Tasodifiy shovqin (model uchun foydasiz) | ❌ Foydasiz | ⚠️ O'chirish tavsiya etiladi |
| num_login_sessions | int64 | Login sessiyalar soni | ⭐⭐ O'rta (faollik ko'rsatkichi) | ❌ Yo'q |
| num_customer_service_calls | int64 | Mijozlar xizmati qo'ng'iroqlari | ⭐⭐ O'rta (muammo ko'rsatkichi) | ❌ Yo'q |
| has_mobile_app | int64 | Mobil ilovasi bor (0/1) | ⭐⭐ O'rta (texnologiya qabul qilish) | ❌ Yo'q |
| paperless_billing | int64 | Qog'ozsiz hisob (0/1) | ⭐⭐ O'rta | ❌ Yo'q |
| **default** | int64 | **TARGET: 0=xavfli, 1=xavfsiz** | ⭐⭐⭐ **TARGET VARIABLE** | ❌ Yo'q |

---

## 2. demographics.csv (8 ustun)

**Kalit:** `cust_id` → `customer_ref` ga join qilinadi

| Ustun | Ma'lumot turi | Tavsif | Model uchun ahamiyati | Tozalash kerakmi |
|-------|---------------|--------|----------------------|------------------|
| cust_id | int64 | Mijoz ID (customer_ref bilan bir xil) | ⭐⭐⭐ Kalit | ❌ Yo'q |
| age | int64 | Yoshi | ⭐⭐⭐ Yuqori (demografik omil) | ❌ Yo'q |
| annual_income | object | Yillik daromad | ⭐⭐⭐ **YUQORI** (moliyaviy qobiliyat) | ✅ **$ va vergulni olib tashlash** |
| employment_length | float64 | Ish staji (yillar) | ⭐⭐⭐ Yuqori | ✅ NaN to'ldirish (median yoki 0) |
| employment_type | object | Ish turi | ⭐⭐⭐ Yuqori | ✅ **Standartlashtirish** (Full-time, FULL_TIME → "Full-time") |
| education | object | Ta'lim darajasi | ⭐⭐ O'rta | ✅ **Standartlashtirish** (case) |
| marital_status | object | Oilaviy holati | ⭐⭐ O'rta | ✅ **Standartlashtirish** (case) |
| num_dependents | int64 | Qaramog'idagilar soni | ⭐⭐ O'rta | ❌ Yo'q |

**Tozalash misollari:**
- `annual_income`: "$61800" → 61800, "28,600" → 28600
- `employment_type`: "Full-time", "FULL_TIME", "Full Time", "Fulltime" → "Full-time"

---

## 3. credit_history.csv (12 ustun)

**Kalit:** `customer_number` → `customer_ref` ga join qilinadi

| Ustun | Ma'lumot turi | Tavsif | Model uchun ahamiyati | Tozalash kerakmi |
|-------|---------------|--------|----------------------|------------------|
| customer_number | int64 | Mijoz raqami (customer_ref bilan bir xil) | ⭐⭐⭐ Kalit | ❌ Yo'q |
| credit_score | int64 | Kredit balli (300-850) | ⭐⭐⭐ **YUQORI** (eng muhim) | ❌ Yo'q |
| num_credit_accounts | int64 | Kredit hisoblar soni | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| oldest_credit_line_age | float64 | Eng qadimgi kredit yoshi (yillar) | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| oldest_account_age_months | float64 | Eng qadimgi hisob yoshi (oylar) | ⭐⭐ O'rta | ❌ Yo'q |
| total_credit_limit | float64 | Jami kredit limiti | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| num_delinquencies_2yrs | float64 | 2 yil ichida to'lov buzilishlari | ⭐⭐⭐ **YUQORI** (xavf ko'rsatkichi) | ✅ NaN to'ldirish (0) |
| num_inquiries_6mo | int64 | 6 oy ichida so'rovlar soni | ⭐⭐ O'rta | ❌ Yo'q |
| recent_inquiry_count | int64 | So'nggi so'rovlar soni | ⭐⭐ O'rta (num_inquiries_6mo bilan bir xil) | ⚠️ Takrorlangan, o'chirish mumkin |
| num_public_records | int64 | Ochiq yozuvlar soni | ⭐⭐⭐ Yuqori (xavf ko'rsatkichi) | ❌ Yo'q |
| num_collections | int64 | To'plangan qarzlar soni | ⭐⭐⭐ Yuqori (xavf ko'rsatkichi) | ❌ Yo'q |
| account_diversity_index | float64 | Hisob xilma-xillik indeksi | ⭐⭐ O'rta | ❌ Yo'q |

**Eslatma:** `num_inquiries_6mo` va `recent_inquiry_count` bir xil, bittasini o'chirish mumkin.

---

## 4. loan_details.csv (10 ustun)

**Kalit:** `customer_id` → `customer_ref` ga join qilinadi

| Ustun | Ma'lumot turi | Tavsif | Model uchun ahamiyati | Tozalash kerakmi |
|-------|---------------|--------|----------------------|------------------|
| customer_id | int64 | Mijoz ID (customer_ref bilan bir xil) | ⭐⭐⭐ Kalit | ❌ Yo'q |
| loan_type | object | Qarz turi | ⭐⭐⭐ Yuqori | ✅ **Standartlashtirish** (Personal, PERSONAL → "Personal") |
| **loan_amount** | object | **Qarz miqdori (Model 2 target)** | ⭐⭐⭐ **TARGET MODEL 2** | ✅ **$ va vergulni olib tashlash** |
| loan_term | int64 | Qarz muddati (oylar) | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| interest_rate | float64 | Foiz stavkasi (%) | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| loan_purpose | object | Qarz maqsadi | ⭐⭐ O'rta | ✅ **Standartlashtirish** |
| loan_to_value_ratio | float64 | Qarz/qiymat nisbati | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| origination_channel | object | Qarz berish kanali | ⭐⭐ O'rta | ✅ Kategoriyal kodlash |
| loan_officer_id | int64 | Qarz ofitseri ID | ⭐ Past (identifikator) | ⚠️ Ehtimol o'chirish |
| marketing_campaign | object | Marketing kampaniyasi | ⭐ Past | ⚠️ Ehtimol o'chirish |

**Tozalash misollari:**
- `loan_amount`: "$17,700" → 17700, "9,300" → 9300, "$8700" → 8700
- `loan_type`: "Personal", "PERSONAL", "Personal Loan" → "Personal"

---

## 5. financial_ratios.csv (16 ustun)

**Kalit:** `cust_num` → `customer_ref` ga join qilinadi

| Ustun | Ma'lumot turi | Tavsif | Model uchun ahamiyati | Tozalash kerakmi |
|-------|---------------|--------|----------------------|------------------|
| cust_num | int64 | Mijoz raqami (customer_ref bilan bir xil) | ⭐⭐⭐ Kalit | ❌ Yo'q |
| monthly_income | object | Oylik daromad | ⭐⭐⭐ **YUQORI** | ✅ **$ va vergulni olib tashlash** |
| existing_monthly_debt | object | Mavjud oylik qarz | ⭐⭐⭐ **YUQORI** | ✅ **$ va vergulni olib tashlash** |
| monthly_payment | object | Oylik to'lov | ⭐⭐⭐ Yuqori | ✅ **$ va vergulni olib tashlash** |
| debt_to_income_ratio | float64 | Qarz/daromad nisbati | ⭐⭐⭐ **YUQORI** (eng muhim) | ❌ Yo'q |
| debt_service_ratio | float64 | Qarz xizmati nisbati | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| payment_to_income_ratio | float64 | To'lov/daromad nisbati | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| credit_utilization | float64 | Kreditdan foydalanish (%) | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| revolving_balance | object | Aylanma balans | ⭐⭐⭐ Yuqori | ✅ **$ va vergulni olib tashlash, NaN to'ldirish (0)** |
| credit_usage_amount | object | Kreditdan foydalanish miqdori | ⭐⭐ O'rta | ✅ **$ va vergulni olib tashlash** |
| available_credit | object | Mavjud kredit | ⭐⭐ O'rta | ✅ **$ va vergulni olib tashlash** |
| total_monthly_debt_payment | object | Jami oylik qarz to'lovi | ⭐⭐⭐ Yuqori | ✅ **$ va vergulni olib tashlash** |
| annual_debt_payment | float64 | Yillik qarz to'lovi | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| loan_to_annual_income | float64 | Qarz/yillik daromad nisbati | ⭐⭐⭐ Yuqori | ❌ Yo'q |
| total_debt_amount | object | Jami qarz miqdori | ⭐⭐⭐ Yuqori | ✅ **$ va vergulni olib tashlash** |
| monthly_free_cash_flow | object | Oylik erkin pul oqimi | ⭐⭐⭐ Yuqori | ✅ **$ va vergulni olib tashlash** |

**Tozalash misollari:**
- `monthly_income`: "5,150.00" → 5150.0, "$2,383.33" → 2383.33
- `revolving_balance`: "$142,213.10" → 142213.10, NaN → 0

---

## 6. geographic_data.csv (8 ustun)

**Kalit:** `id` → `customer_ref` ga join qilinadi

| Ustun | Ma'lumot turi | Tavsif | Model uchun ahamiyati | Tozalash kerakmi |
|-------|---------------|--------|----------------------|------------------|
| id | int64 | Mijoz ID (customer_ref bilan bir xil) | ⭐⭐⭐ Kalit | ❌ Yo'q |
| state | object | Shtat (2 harf kodi) | ⭐⭐ O'rta (geografik omil) | ✅ Kategoriyal kodlash |
| regional_unemployment_rate | float64 | Mintaqaviy ishsizlik darajasi (%) | ⭐⭐ O'rta | ❌ Yo'q |
| regional_median_income | int64 | Mintaqaviy o'rtacha daromad | ⭐⭐ O'rta | ❌ Yo'q |
| regional_median_rent | float64 | Mintaqaviy o'rtacha ijarasi | ⭐⭐ O'rta | ❌ Yo'q |
| housing_price_index | float64 | Uy-joy narxi indeksi | ⭐⭐ O'rta | ❌ Yo'q |
| cost_of_living_index | float64 | Yashash xarajatlari indeksi | ⭐⭐ O'rta | ❌ Yo'q |
| previous_zip_code | int64 | Oldingi pochta indeksi | ⭐ Past | ⚠️ Ehtimol o'chirish |

---

## Model Feature Importance Summary

### Model 1: Default Prediction (Binary Classification)
**Target:** `default` (0=xavfli, 1=xavfsiz)

**Eng muhim ustunlar (High Priority):**
- credit_score ⭐⭐⭐
- debt_to_income_ratio ⭐⭐⭐
- annual_income ⭐⭐⭐
- employment_length ⭐⭐⭐
- num_delinquencies_2yrs ⭐⭐⭐
- num_collections ⭐⭐⭐
- num_public_records ⭐⭐⭐
- total_credit_limit ⭐⭐⭐
- credit_utilization ⭐⭐⭐
- monthly_income ⭐⭐⭐

**O'rta ahamiyatli:**
- age, education, employment_type, marital_status
- loan_type, loan_term, interest_rate
- num_credit_accounts, oldest_credit_line_age
- state, regional_unemployment_rate

**Past ahamiyatli (ehtimol o'chirish):**
- random_noise_1 ❌
- referral_code ⚠️
- loan_officer_id ⚠️
- marketing_campaign ⚠️
- previous_zip_code ⚠️

### Model 2: Loan Amount Prediction (Regression)
**Target:** `loan_amount`

**Eng muhim ustunlar:**
- annual_income ⭐⭐⭐
- monthly_income ⭐⭐⭐
- debt_to_income_ratio ⭐⭐⭐
- credit_score ⭐⭐⭐
- total_credit_limit ⭐⭐⭐
- monthly_free_cash_flow ⭐⭐⭐
- employment_length ⭐⭐⭐

---

## Join Strategy

**Asosiy kalit:** `customer_ref` (application_metadata.csv dan)

**Join ketma-ketligi:**
1. **application_metadata.csv** (asosiy, default target bor)
2. **demographics.csv** → `cust_id = customer_ref`
3. **credit_history.csv** → `customer_number = customer_ref`
4. **loan_details.csv** → `customer_id = customer_ref`
5. **financial_ratios.csv** → `cust_num = customer_ref`
6. **geographic_data.csv** → `id = customer_ref`

**Join turi:** LEFT JOIN (barcha 89,999 qator saqlanadi)

**Join keylari:** Barcha kalit ustunlar (`customer_ref`, `cust_id`, `customer_number`, `customer_id`, `cust_num`, `id`) bir xil qiymatlarga ega (100% bir xil).

---

## Data Cleaning Checklist

### 1. Numeric Cleaning (Currency & Commas)
- [ ] `annual_income`: "$61800" → 61800, "28,600" → 28600
- [ ] `loan_amount`: "$17,700" → 17700, "9,300" → 9300
- [ ] `monthly_income`: "5,150.00" → 5150.0, "$2,383.33" → 2383.33
- [ ] `existing_monthly_debt`: "$288.71" → 288.71
- [ ] `monthly_payment`: "$592.13" → 592.13
- [ ] `revolving_balance`: "$142,213.10" → 142213.10
- [ ] `credit_usage_amount`: "$142,213.10" → 142213.10
- [ ] `available_credit`: "$26,886.90" → 26886.90
- [ ] `total_monthly_debt_payment`: "1330.77" → 1330.77 (agar string bo'lsa)
- [ ] `total_debt_amount`: "159,913.10" → 159913.10
- [ ] `monthly_free_cash_flow`: "3,819.23" → 3819.23

### 2. Categorical Standardization
- [ ] `employment_type`: "Full-time", "FULL_TIME", "Full Time", "Fulltime" → "Full-time"
- [ ] `education`: Case standartlashtirish
- [ ] `loan_type`: "Personal", "PERSONAL", "Personal Loan" → "Personal"
- [ ] `loan_purpose`: Standartlashtirish
- [ ] `marital_status`: Case standartlashtirish

### 3. Missing Values
- [ ] `employment_length`: NaN → median yoki 0
- [ ] `revolving_balance`: NaN → 0
- [ ] `num_delinquencies_2yrs`: NaN → 0

### 4. Data Type Conversion
- [ ] Barcha tozalangan string raqamlarni float/int ga o'tkazish
- [ ] Boolean ustunlarni int (0/1) ga ta'minlash

### 5. Drop Columns (takrorlangan yoki foydasiz)
- [ ] `random_noise_1` ❌
- [ ] `recent_inquiry_count` (num_inquiries_6mo bilan bir xil)
- [ ] Join keylari (join qilgandan keyin faqat `customer_ref` saqlanadi)

---

## Expected Final Dataset

**O'lchami:** (89999, ~60-62 ustun)
**Qatorlar:** 89,999 (barcha mijozlar)
**Ustunlar:** ~60-62 (takrorlangan va foydasiz ustunlar o'chirilgandan keyin)

**Asosiy ustunlar:**
- `customer_ref` (asosiy kalit)
- `default` (Model 1 target)
- `loan_amount` (Model 2 target)
- Barcha tozalangan va standartlashtirilgan feature ustunlar

