# 🧹 Paid Premium Leads — Data Cleaning Project

> A real-world data wrangling exercise: taking a raw Google Maps scrape and transforming it into a clean, analysis-ready leads dataset.

---

## 📌 Project Overview

This project demonstrates end-to-end **data cleaning and enrichment** on a raw CSV export of local business leads scraped from Google Maps. The source data contained several common real-world issues — encoding corruption, redundant columns, missing values, and inconsistent formatting — all resolved programmatically using Python.

---

## 🗂️ Files

| File | Description |
|---|---|
| `paid_premium_leadsform.csv` | Raw source data (Google Maps scrape) |
| `paid_premium_leads_cleaned.xlsx` | Cleaned, formatted output ready for analysis or outreach |
| `clean_leads.py` | Python cleaning script |

---

## 🔍 Issues Found in Raw Data

| Issue | Detail |
|---|---|
| **BOM encoding corruption** | Title column header contained `Ã¯Â»Â¿""title"""` due to improper UTF-8 handling |
| **Exploded category columns** | 10 separate `categories/0`–`categories/9` columns instead of one field |
| **Redundant column** | `categoryName` duplicated `categories/0` |
| **UTM tracking noise** | One website URL contained full UTM query parameters |
| **Street typo** | `9109 S S Chicago Ave` — doubled directional prefix |
| **Inconsistent casing** | `team web development` not title-cased |
| **Missing ratings** | 7 of 13 rows had no `totalScore` or `reviewsCount` |

---

## ✅ Cleaning Steps Applied

```python
# 1. Read with UTF-8 BOM handling
df = pd.read_csv('paid_premium_leadsform.csv', encoding='utf-8-sig')

# 2. Normalize column names
df.columns = [c.strip().strip('"') for c in df.columns]

# 3. Collapse 10 category columns into one deduplicated field
cat_cols = [c for c in df.columns if c.startswith('categories/')]
df['categories'] = df.apply(lambda r: ', '.join(dict.fromkeys(
    str(r[c]).strip() for c in cat_cols
    if pd.notna(r[c]) and str(r[c]).strip() not in ('', 'nan')
)), axis=1)

# 4. Strip UTM parameters from website URLs
df['website'] = df['website'].apply(lambda u: re.sub(r'\?.*', '', str(u)).rstrip('/'))

# 5. Fix street typo
df['street'] = df['street'].str.replace(r'\bS S\b', 'S', regex=True)

# 6. Title-case lowercased business names
df['business_name'] = df['business_name'].apply(
    lambda n: n.title() if n == n.lower() else n
)
```

---

## 📊 Output Format

The cleaned file is exported as a styled `.xlsx` with:

- **Frozen header row** for easy scrolling
- **Alternating row bands** for readability
- **Yellow-highlighted rows** flagging records with missing ratings
- **Dark navy header** with white bold labels
- **Clean column set**: Business Name, Phone, Website, Street, City, State, Country, Rating, Review Count, Categories, Google Maps URL

---

## 🛠️ Tech Stack

- **Python 3**
- **pandas** — data loading, transformation, and export
- **openpyxl** — Excel formatting and styling
- **re** — URL cleaning via regex

---

## 💡 Key Takeaways

- Real-world scraped data almost always requires encoding fixes — always specify `utf-8-sig` when reading CSVs from web tools
- Wide categorical columns (one column per tag) are a common scraper anti-pattern; collapsing them makes the data far more usable
- Small issues like URL noise and inconsistent casing matter when the output is used for sales outreach or CRM import
- Flagging (not dropping) incomplete rows preserves data while making gaps visible

---

## 🚀 How to Run

```bash
pip install pandas openpyxl
python clean_leads.py
```

Output will be saved to `paid_premium_leads_cleaned.xlsx`.
