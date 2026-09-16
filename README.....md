# Zepto E-commerce SQL Data Analysis

End-to-end SQL analysis of a real e-commerce inventory dataset scraped from [Zepto](https://www.zeptonow.com/), one of India's largest quick-commerce platforms. The project covers the full analyst workflow — schema design, data import, exploration, cleaning, and business-driven querying in PostgreSQL.

---

## Project Overview

The objective was to work a messy, real-world product catalogue the way an analyst would in a retail or e-commerce setting:

- Design a schema and load raw scraped inventory data into PostgreSQL
- Run exploratory analysis on categories, stock availability, and pricing consistency
- Clean the dataset — handle nulls, drop invalid records, and correct price encoding
- Answer business questions around pricing, discounting, inventory, and revenue

---

## Dataset

Sourced from [Kaggle](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset/data?select=zepto_v2.csv), originally scraped from Zepto's public product listings.

Each row is a single SKU. The same product name appears more than once because a product can be listed in multiple package sizes, weights, or categories — which is exactly how live catalogue data behaves.

| Column | Description |
|---|---|
| `sku_id` | Unique identifier per product entry (synthetic primary key) |
| `name` | Product name as listed on the app |
| `category` | Product category — Fruits, Snacks, Beverages, etc. |
| `mrp` | Maximum retail price (stored in paise, converted to ₹) |
| `discountPercent` | Discount applied against MRP |
| `discountedSellingPrice` | Final price after discount (also converted to ₹) |
| `availableQuantity` | Units held in inventory |
| `weightInGms` | Product weight in grams |
| `outOfStock` | Boolean stock availability flag |
| `quantity` | Units per package (mixed with grams for loose produce) |

---

## Workflow

### 1. Schema design

The raw file has no unique key, so a synthetic primary key was added to make rows individually addressable.

```sql
CREATE TABLE zepto (
  sku_id                 SERIAL PRIMARY KEY,
  category               VARCHAR(120),
  name                   VARCHAR(150) NOT NULL,
  mrp                    NUMERIC(8,2),
  discountPercent        NUMERIC(5,2),
  availableQuantity      INTEGER,
  discountedSellingPrice NUMERIC(8,2),
  weightInGms            INTEGER,
  outOfStock             BOOLEAN,
  quantity               INTEGER
);
```

### 2. Data import

Loaded via pgAdmin's import utility. The command-line equivalent, which gives clearer error output:

```sql
\copy zepto(category, name, mrp, discountPercent, availableQuantity,
            discountedSellingPrice, weightInGms, outOfStock, quantity)
FROM 'data/zepto_v2.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', QUOTE '"', ENCODING 'UTF8');
```

The initial import failed on a UTF-8 encoding error caused by special characters in product names. Resolved by re-saving the source file in CSV UTF-8 format.

### 3. Data exploration

- Counted total records and previewed the dataset structure
- Checked for null values across every column
- Listed distinct product categories
- Compared in-stock against out-of-stock product counts
- Identified product names appearing multiple times as separate SKUs

### 4. Data cleaning

- Removed records where MRP or discounted selling price was zero — these are not sellable and distort any price aggregation
- Converted `mrp` and `discountedSellingPrice` from paise to rupees, making every downstream revenue figure meaningful

### 5. Business analysis

- Ranked the top 10 best-value products by discount percentage
- Flagged high-MRP products currently out of stock — the largest revenue leaks
- Estimated potential revenue per product category
- Isolated premium products (MRP above ₹500) carrying minimal discount
- Identified the 5 categories offering the deepest average discounts
- Calculated price per gram to surface genuine value-for-money SKUs
- Segmented products into Low, Medium, and Bulk weight bands
- Measured total inventory weight held per category

---

## Running this project

1. **Clone the repository**

   ```bash
   git clone https://github.com/sr165208-hue/zepto-SQL-data-analysis-project.git
   cd zepto-SQL-data-analysis-project
   ```

2. **Open `zepto_SQL_data_analysis.sql`** — contains table creation, exploration, cleaning, and all analysis queries.

3. **Load the dataset** into pgAdmin or any PostgreSQL client. Create a database, import the CSV (converting to UTF-8 if needed), then run the SQL file.

---

## Tools

PostgreSQL · pgAdmin · SQL

---

## License

MIT — free to fork and use.

---

## Author

**Sanjay Rawat** — Data Analyst

[LinkedIn](https://www.linkedin.com/in/sanjay-rawat-a0b157290/) · [GitHub](https://github.com/sr165208-hue)
