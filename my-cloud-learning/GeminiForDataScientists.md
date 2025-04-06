# Gemini for Data Scientists: Customer Segmentation Analysis

This guide demonstrates how to use Google Cloud's Gemini AI to perform customer segmentation analysis using BigQuery and Python. The tutorial covers data preparation, model creation, visualization, and generating actionable insights.

## Overview

This project showcases how to:
- Create and manage BigQuery datasets
- Build Python notebooks in BigQuery
- Implement K-means clustering for customer segmentation
- Generate visualizations and insights
- Create marketing campaigns using Gemini AI

## Prerequisites

- Google Cloud Console account
- Access to BigQuery
- Basic understanding of:
  - Python programming
  - SQL
  - Machine Learning concepts
  - Data analysis

## Project Structure

### 1. Environment Setup
- Create BigQuery dataset
- Set up Python notebook
- Configure Colab Enterprise runtime

### 2. Data Analysis Pipeline
- Data preparation and import
- Model development
- Visualization
- Insight generation

## Step-by-Step Guide

### 1. Dataset Creation
1. Navigate to BigQuery in Google Cloud Console
2. Create a new dataset:
   - Dataset ID: `ecommerce`
   - Location type: Multi-Region
   - Multi-region: US (multiple regions in United States)

### 2. Python Notebook Setup
1. Create a new Python notebook in BigQuery
2. Connect to Colab Enterprise runtime
3. Import required libraries:
   ```python
   from google.cloud import bigquery
   from google.cloud import aiplatform
   import bigframes.pandas as bpd
   import pandas as pd
   from vertexai.language_models._language_models import TextGenerationModel
   from vertexai.generative_models import GenerativeModel
   from bigframes.ml.cluster import KMeans
   from bigframes.ml.model_selection import train_test_split
   ```

4. Set up project variables:
   ```python
   project_id = '<your-project-id>'
   dataset_name = "ecommerce"
   model_name = "customer_segmentation_model"
   table_name = "customer_stats"
   location = "<your-region>"
   client = bigquery.Client(project=project_id)
   aiplatform.init(project=project_id, location=location)
   ```

### 3. Data Processing
1. Create customer statistics table:
   ```sql
   CREATE OR REPLACE TABLE ecommerce.customer_stats AS
   SELECT
     user_id,
     DATE_DIFF(CURRENT_DATE(), CAST(MAX(order_created_date) AS DATE), day) AS days_since_last_order,
     COUNT(order_id) AS count_orders,
     AVG(sale_price) AS average_spend
   FROM (
       SELECT
         user_id,
         order_id,
         sale_price,
         created_at AS order_created_date
         FROM `bigquery-public-data.thelook_ecommerce.order_items`
         WHERE created_at BETWEEN '2022-01-01' AND '2023-01-01'
   )
   GROUP BY user_id;
   ```

2. Load data into BigQuery DataFrame:
   ```python
   bqdf = bpd.read_gbq(f"{project_id}.{dataset_name}.{table_name}")
   bqdf.head(10)
   ```

### 4. Model Development
1. Split data and create K-means model:
   ```python
   # Split data
   df_train, df_test = train_test_split(bq_df, test_size=0.2, random_state=42)
   
   # Create and fit model
   kmeans = KMeans(n_clusters=5)
   kmeans.fit(df_train)
   
   # Save model
   kmeans.to_gbq(f"{project_id}.{dataset_name}.{model_name}")
   ```

2. Generate predictions:
   ```python
   predictions_df = kmeans.predict(df_test)
   predictions_df.head(10)
   ```

3. Visualize results:
   ```python
   import matplotlib.pyplot as plt
   
   plt.figure(figsize=(10, 6))
   plt.scatter(predictions_df['days_since_last_order'], 
              predictions_df['average_spend'], 
              c=predictions_df['CENTROID_ID'], 
              cmap='viridis')
   
   plt.title('Attribute grouped by K-means cluster')
   plt.xlabel('Days Since Last Order')
   plt.ylabel('Average Spend')
   plt.colorbar(label='Cluster ID')
   plt.show()
   ```

### 5. Insight Generation
1. Analyze cluster characteristics:
   ```python
   query = """
   SELECT
     CONCAT('cluster ', CAST(centroid_id as STRING)) as centroid,
     average_spend,
     count_orders,
     days_since_last_order
   FROM (
     SELECT centroid_id, feature, ROUND(numerical_value, 2) as value
     FROM ML.CENTROIDS(MODEL `{0}.{1}`)
   )
   PIVOT (
     SUM(value)
     FOR feature IN ('average_spend', 'count_orders', 'days_since_last_order')
   )
   ORDER BY centroid_id
   """.format(dataset_name, model_name)
   
   df_centroid = client.query(query).to_dataframe()
   df_centroid.head()
   ```

2. Generate marketing insights using Gemini:
   ```python
   model = GenerativeModel("gemini-1.0-pro")
   
   prompt = f"""
   You're a creative brand strategist, given the following clusters, come up with \
   creative brand persona, a catchy title, and next marketing action, \
   explained step by step. Identify the cluster number, the title of the person, a persona for them and the next marketing step.
   
   Clusters:
   {cluster_info}
   
   For each Cluster:
   * Title:
   * Persona:
   * Next marketing step:
   """
   
   responses = model.generate_content(
      prompt,
      generation_config={
         "temperature": 0.1,
         "max_output_tokens": 800,
         "top_p": 1.0,
         "top_k": 40,
      }
   )
   
   print(responses.text)
   ```

## Key Features

- **Data Processing**: Efficient handling of large-scale ecommerce data
- **Machine Learning**: Implementation of K-means clustering for customer segmentation
- **Visualization**: Interactive scatter plots for cluster analysis
- **AI Integration**: Use of Gemini AI for generating marketing insights

## Results

The analysis produces:
- Customer segments based on:
  - Recency of purchase
  - Purchase frequency
  - Average spend
- Visual representations of clusters
- Marketing personas and recommendations

## Cleanup

To avoid incurring charges, you can clean up resources by:
1. Deleting the project (if it was created specifically for this tutorial)
2. Removing individual resources:
   ```python
   # Delete customer_stats table
   client.delete_table(f"{project_id}.{dataset_name}.{table_name}", not_found_ok=True)
   print(f"Deleted table: {project_id}.{dataset_name}.{table_name}")
   
   # Delete K-means model
   client.delete_model(f"{project_id}.{dataset_name}.{model_name}", not_found_ok=True)
   print(f"Deleted model: {project_id}.{dataset_name}.{model_name}")
   ```

## Additional Resources

- [Google Cloud Console](https://console.cloud.google.com)
- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs)
- [Gemini AI Documentation](https://cloud.google.com/ai-platform/docs)

## Note

This tutorial is designed for data scientists and analysts working with customer data. Always validate AI-generated code and insights before implementation in production environments.