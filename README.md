# 🏨 Hotel Booking Analytics: Snowflake + AWS S3 + Power BI

An end-to-end data engineering and analytics project that loads raw hotel booking data from **AWS S3** into **Snowflake**, cleans it using a **Medallion architecture (Bronze → Silver → Gold)**, and visualizes insights in a **Power BI** dashboard.

---

## 📌 Problem Statement

The hotel's booking data is raw and inconsistent (invalid emails, negative amounts, wrong date order, misspelled statuses, duplicate bookings). Management has no clear view of monthly revenue, booking trends, or city-level performance.

**Goal:** clean and standardize the data, then deliver quick, accurate insights through a dashboard.

---

## 🏗️ Architecture

```
CSV file in AWS S3
      │   (Storage Integration + IAM Role)
      ▼
Snowflake External Stage ──► COPY INTO
      ▼
🥉 BRONZE_TBL   raw data, all columns as strings
      ▼   clean, validate, type-cast, de-duplicate
🥈 SILVER_TBL   cleaned and typed data
      ▼   aggregate
🥇 GOLD tables  GOLDEN_DAILY_BOOKING
                GOLDEN_HOTEL_CITY_SALES
                GOLDEN_CLEAN_BOOKING
      ▼
📊 Power BI Dashboard
```

| Layer | Purpose |
|---|---|
| **Bronze** | Raw copy of the source file, nothing changed |
| **Silver** | Validated, standardized, correctly typed data |
| **Gold** | Business-ready tables for reporting |

---

## 🛠️ Tech Stack

- **Cloud storage:** AWS S3, AWS IAM
- **Data warehouse:** Snowflake (SQL)
- **BI tool:** Power BI
- **Version control:** Git & GitHub

---

## 📂 Repository Structure

```
hotel-booking-snowflake-pipeline/
├── README.md
├── .gitignore
├── sql/
│   └── hotel_booking_pipeline.sql
├── data/
│   └── sample_hotel_bookings.csv
├── powerbi/
│   └── Hotel_Booking_Analytics_report.pbix
├── docs/
│   ├── Hotel_Analytics_BRD.docx
│   └── dashboard.png
```

> Adjust the file names above to match what you actually commit.

---

## 🧹 Data Cleaning Rules (Silver Layer)

| Issue | Handling |
|---|---|
| Invalid or missing emails | Set to `NULL`; valid emails trimmed and lowercased |
| Invalid dates | Rows removed |
| Check-out earlier than check-in | Rows removed |
| Negative amounts | Converted to positive with `ABS()` |
| Amounts with decimals | Stored as `NUMBER(12,2)` to avoid losing precision |
| Misspelled statuses (e.g., "Confirmd", "Confirmeeed") | Standardized to "Confirmed" |
| City and customer names | Trimmed and converted to proper case |
| Duplicate bookings | One row kept per `booking_id` |
| Guest count | Converted to integer |

---

## 🚀 How to Run

### 1. Prerequisites
- Snowflake account (role with permission to create a storage integration, e.g. `ACCOUNTADMIN`)
- AWS account with an S3 bucket containing the booking CSV
- Power BI Desktop

### 2. Set up AWS
1. Upload the CSV to your S3 bucket.
2. Create an IAM role that Snowflake can assume, with read access to the bucket.

### 3. Run the SQL script
Open `sql/hotel_booking_pipeline.sql` in a Snowflake worksheet and run it **top to bottom**:

1. Create database and schema
2. Create the CSV file format
3. Create the storage integration (replace the `<ACCOUNT_ID>` and `<ROLE_NAME>` placeholders)
4. Run `DESC STORAGE INTEGRATION`, then copy `STORAGE_AWS_IAM_USER_ARN` and `STORAGE_AWS_EXTERNAL_ID` into your IAM role's **trust policy**
5. Create the stage and check it with `LIST @HOTEL_EX`
6. Create the Bronze table and load it with `COPY INTO`
7. Review the data quality checks
8. Create and load the Silver table
9. Create the Gold tables
10. Verify the results

### 4. Connect Power BI
1. Open the `.pbix` file in Power BI Desktop.
2. Connect to your Snowflake account and warehouse.
3. Point the report to the Gold tables in `HOTEL_DB.PUBLIC` and refresh.

---

## 📊 Dashboard

![Dashboard](docs/dashboard.png)

**Visuals included**
- **KPI cards:** Total Revenue, Average Booking Value, Total Guests, Total Bookings
- **Area charts:** Revenue and bookings by year and month
- **Bar charts:** Revenue by hotel city, bookings by room type, bookings by booking status

---

## 🔒 Security Notes

- No AWS keys, passwords or real account IDs are stored in this repo (placeholders only).
- Access to S3 uses an **IAM role through a Snowflake storage integration**.
- The sample data is **fake**. Do not commit real customer names or emails.

---

## 🔮 Future Improvements

- Automate ingestion with **Snowpipe** (auto-ingest from S3)
- Schedule Bronze → Silver → Gold with **Streams and Tasks**
- Use `MERGE` for incremental loads
- Quarantine table for rejected rows
- Add a date dimension and filters (city, date, status) to the dashboard
- Add dbt for transformations and testing

---

## 👤 Author

**Ayush Nagre**
- GitHub: [@AYUSH-053](https://github.com/AYUSH-053)
