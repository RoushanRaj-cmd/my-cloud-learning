# Form Parser with Google Cloud Document AI

This guide demonstrates how to use Google Cloud's Document AI to parse forms and extract structured data from documents. The tutorial covers setting up Document AI, creating a Form Parser processor, and extracting both key-value pairs and table data from documents.

## Prerequisites

- Google Cloud Console account
- Access to Cloud Shell
- Basic understanding of Python programming

## Setup Instructions

### 1. Enable Document AI API

1. Open Cloud Shell in Google Cloud Console
2. Enable the Document AI API:
   ```bash
   gcloud services enable documentai.googleapis.com
   ```
3. Install required Python packages:
   ```bash
   pip3 install --upgrade pandas
   pip3 install --upgrade google-cloud-documentai
   ```

### 2. Create Form Parser Processor

1. Navigate to Document AI in Google Cloud Console:
   - Open Navigation menu
   - Go to View All Products > Artificial Intelligence > Document AI
2. Create a new processor:
   - Click "Explore Processors"
   - Select "Form Parser"
   - Click "Create Processor"
   - Name: `lab-form-parser`
   - Select your region
   - Click "Create"
3. Save your Processor ID for later use

## Usage Guide

### 1. Basic Form Parsing

1. Download sample form:
   ```bash
   gcloud storage cp gs://cloud-samples-data/documentai/codelabs/form-parser/intake-form.pdf .
   ```

2. Create `form_parser.py`:
   ```python
   import pandas as pd
   from google.cloud import documentai_v1 as documentai

   def online_process(
       project_id: str,
       location: str,
       processor_id: str,
       file_path: str,
       mime_type: str,
   ) -> documentai.Document:
       opts = {"api_endpoint": f"{location}-documentai.googleapis.com"}
       documentai_client = documentai.DocumentProcessorServiceClient(client_options=opts)
       resource_name = documentai_client.processor_path(project_id, location, processor_id)

       with open(file_path, "rb") as image:
           image_content = image.read()
           raw_document = documentai.RawDocument(
               content=image_content, mime_type=mime_type
           )
           request = documentai.ProcessRequest(
               name=resource_name, raw_document=raw_document
           )
           result = documentai_client.process_document(request=request)
           return result.document

   def trim_text(text: str):
       return text.strip().replace("\n", " ")

   # Configuration
   PROJECT_ID = "YOUR_PROJECT_ID"
   LOCATION = "YOUR_PROJECT_LOCATION"  # Format is 'us' or 'eu'
   PROCESSOR_ID = "FORM_PARSER_ID"
   FILE_PATH = "form.pdf"
   MIME_TYPE = "application/pdf"

   # Process document
   document = online_process(
       project_id=PROJECT_ID,
       location=LOCATION,
       processor_id=PROCESSOR_ID,
       file_path=FILE_PATH,
       mime_type=MIME_TYPE,
   )

   # Extract data
   names = []
   name_confidence = []
   values = []
   value_confidence = []

   for page in document.pages:
       for field in page.form_fields:
           names.append(trim_text(field.field_name.text_anchor.content))
           name_confidence.append(field.field_name.confidence)
           values.append(trim_text(field.field_value.text_anchor.content))
           value_confidence.append(field.field_value.confidence)

   # Create DataFrame
   df = pd.DataFrame({
       "Field Name": names,
       "Field Name Confidence": name_confidence,
       "Field Value": values,
       "Field Value Confidence": value_confidence,
   })

   print(df)
   ```

3. Run the script:
   ```bash
   python3 form_parser.py
   ```

### 2. Table Parsing

1. Download sample form with tables:
   ```bash
   gcloud storage cp gs://cloud-samples-data/documentai/codelabs/form-parser/form_with_tables.pdf .
   ```

2. Create `table_parsing.py`:
   ```python
   from os.path import splitext
   from typing import List, Sequence
   import pandas as pd
   from google.cloud import documentai

   def online_process(
       project_id: str,
       location: str,
       processor_id: str,
       file_path: str,
       mime_type: str,
   ) -> documentai.Document:
       opts = {"api_endpoint": f"{location}-documentai.googleapis.com"}
       documentai_client = documentai.DocumentProcessorServiceClient(client_options=opts)
       resource_name = documentai_client.processor_path(project_id, location, processor_id)

       with open(file_path, "rb") as image:
           image_content = image.read()
           raw_document = documentai.RawDocument(
               content=image_content, mime_type=mime_type
           )
           request = documentai.ProcessRequest(
               name=resource_name, raw_document=raw_document
           )
           result = documentai_client.process_document(request=request)
           return result.document

   def get_table_data(
       rows: Sequence[documentai.Document.Page.Table.TableRow], text: str
   ) -> List[List[str]]:
       all_values: List[List[str]] = []
       for row in rows:
           current_row_values: List[str] = []
           for cell in row.cells:
               current_row_values.append(
                   text_anchor_to_text(cell.layout.text_anchor, text)
               )
           all_values.append(current_row_values)
       return all_values

   def text_anchor_to_text(text_anchor: documentai.Document.TextAnchor, text: str) -> str:
       response = ""
       for segment in text_anchor.text_segments:
           start_index = int(segment.start_index)
           end_index = int(segment.end_index)
           response += text[start_index:end_index]
       return response.strip().replace("\n", " ")

   # Configuration
   PROJECT_ID = "YOUR_PROJECT_ID"
   LOCATION = "YOUR_PROJECT_LOCATION"
   PROCESSOR_ID = "FORM_PARSER_ID"
   FILE_PATH = "form_with_tables.pdf"
   MIME_TYPE = "application/pdf"

   # Process document
   document = online_process(
       project_id=PROJECT_ID,
       location=LOCATION,
       processor_id=PROCESSOR_ID,
       file_path=FILE_PATH,
       mime_type=MIME_TYPE,
   )

   # Extract tables
   output_file_prefix = splitext(FILE_PATH)[0]

   for page in document.pages:
       for index, table in enumerate(page.tables):
           header_row_values = get_table_data(table.header_rows, document.text)
           body_row_values = get_table_data(table.body_rows, document.text)

           # Create DataFrame
           df = pd.DataFrame(
               data=body_row_values,
               columns=pd.MultiIndex.from_arrays(header_row_values),
           )

           print(f"Page {page.page_number} - Table {index}")
           print(df)

           # Save to CSV
           output_filename = f"{output_file_prefix}_pg{page.page_number}_tb{index}.csv"
           df.to_csv(output_filename, index=False)
   ```

3. Run the script:
   ```bash
   python3 table_parsing.py
   ```

## Expected Output

### Form Parser Output
The script will output a DataFrame containing:
- Field names
- Field name confidence scores
- Field values
- Field value confidence scores

### Table Parser Output
The script will:
1. Print each table found in the document
2. Save each table as a separate CSV file
3. Display table contents in the console

## Additional Resources

- [Google Cloud Console](https://console.cloud.google.com)
- [Document AI Documentation](https://cloud.google.com/document-ai/docs)
- [Form Parser Documentation](https://cloud.google.com/document-ai/docs/processors-list)

## Note

Remember to replace the following placeholders in the code:
- `YOUR_PROJECT_ID`
- `YOUR_PROJECT_LOCATION`
- `FORM_PARSER_ID`

Always validate the extracted data before using it in production applications. 