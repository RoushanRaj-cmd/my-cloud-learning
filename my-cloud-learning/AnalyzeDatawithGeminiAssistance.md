# Analyze Data with Gemini in Google Cloud

This guide provides step-by-step instructions for setting up and using Google Cloud's Gemini AI for data analysis in BigQuery.

## Prerequisites

- Google Cloud Console account
- Access to Cloud Shell
- Basic understanding of SQL and BigQuery

## Setup Instructions

### 1. Environment Configuration

1. Sign in to Google Cloud Console
2. Open Cloud Shell terminal
3. Set up environment variables:
   ```bash
   PROJECT_ID=$(gcloud config get-value project)
   REGION=set_at_lab_start
   USER=$(gcloud config get-value account 2> /dev/null)
   ```
4. Enable Cloud AI Companion API:
   ```bash
   gcloud services enable cloudaicompanion.googleapis.com --project ${PROJECT_ID}
   ```
5. Grant necessary IAM roles:
   ```bash
   gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/cloudaicompanion.user
   gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/serviceusage.serviceUsageViewer
   ```

### 2. BigQuery Setup

1. Create a new dataset:
   - Navigate to BigQuery in Google Cloud Console
   - Create dataset with ID: `bqml_tutorial`
   - Set location type to Multi-region
2. Enable Gemini features in BigQuery:
   - Click on Gemini in the toolbar
   - Enable the following features:
     - Auto completion
     - Auto generation
     - Explanation

## Using Gemini for Data Analysis

### 1. Basic Data Discovery

Use natural language prompts to discover available data:
- "How do I view which datasets and tables are available to me in BigQuery?"
- "How do I get started with BigQuery?"
- "What are the benefits of using BigQuery for data analysis?"

### 2. SQL Query Explanation

Gemini can explain complex SQL queries. Example:
```sql
SELECT u.id as user_id, u.first_name, u.last_name, avg(oi.sale_price) as avg_sale_price   
FROM `bigquery-public-data.thelook_ecommerce.users` as u   
JOIN `bigquery-public-data.thelook_ecommerce.order_items` as oi   
ON u.id = oi.user_id   
GROUP BY 1,2,3   
ORDER BY avg_sale_price DESC   
LIMIT 10
```

To get an explanation:
1. Select the query
2. Right-click and choose "Explain current selection"

### 3. SQL Query Generation

Gemini can generate SQL queries based on natural language prompts. Example:
```sql
SELECT
    DATE(order_items.created_at) AS order_date,
    order_items.product_id,
    products.name AS product_name,
    ROUND(SUM(order_items.sale_price), 2) AS total_sales
FROM
    `bigquery-public-data.thelook_ecommerce.order_items` AS order_items
LEFT JOIN
    `bigquery-public-data.thelook_ecommerce.products` AS products
ON
    order_items.product_id = products.id
GROUP BY
    order_date,
    order_items.product_id,
    product_name
ORDER BY
    total_sales DESC;
```

## Tips for Best Practices

1. Review table schemas before running queries
2. Use clear and specific prompts for better results
3. Verify generated queries before execution
4. Refresh the page if Gemini features are not visible

## Additional Resources

- [Google Cloud Console](https://console.cloud.google.com)
- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Gemini Documentation](https://cloud.google.com/ai-platform/docs) 