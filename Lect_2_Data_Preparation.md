# LLMOps – Data Preparation (with Code Example)
**Course:** LLMOps (Coursera)

---

## 1. Overview

- A key step in LLMOps is dealing with large amounts of text data.
- This lecture covers:
  - Retrieving Stack Overflow text data from a data warehouse
  - Handling data too large to fit in memory using SQL
  - Modifying data to tune a model for task-specific use

---

## 2. Setup & Authentication

### 2.1 Authentication
- A **credential variable** is used to authenticate identity and grant permissions to access cloud services and data.

### 2.2 Project Setup
- A **project** organizes all cloud resources; all data lives inside the project.

### 2.3 Region
- A **region** specifies where cloud resources will run (e.g., a data center).
- Example used in the lecture: **US Central 1**

### 2.4 Vertex AI SDK
- Import and initialize the **Vertex AI library** (SDK) using the project ID, region, and credentials.
- The SDK enables interaction with Vertex AI services in the cloud.

---

## 3. Data Warehouse & BigQuery

### 3.1 What is a Data Warehouse?
- A **central repository** for storing and analyzing large amounts of data from various sources.
- Used to gain insights and make informed decisions.
- Application database data can be brought into the warehouse and made available to teams.

### 3.2 BigQuery
- The data warehouse used in this lecture.
- **Serverless** — no managed service required.
- Supports **SQL** queries.

### 3.3 SQL vs. Pandas

| SQL | Pandas |
|-----|--------|
| Best when data sits in a data warehouse | Best when data fits in memory on your computer |
| Handles large-scale processing efficiently | Great for in-memory data exploration |
| Good for cleaning and data preparation | Good for smaller datasets |

> **Key insight:** SQL + a data warehouse is a powerful, scalable combination when building LLMOps systems.

---

## 4. Exploring the Stack Overflow Dataset

### 4.1 Dataset Structure
- **Dataset:** Stack Overflow public dataset
- Contains **tables** with different types of data (e.g., answers, comments, questions)
- Tables in a data warehouse are organized collections of data.

### 4.2 Querying Table Names
- Use SQL to list all available tables within the dataset:
  ```sql
  SELECT table_name
  FROM `bigquery-public-data.stackoverflow.INFORMATION_SCHEMA.TABLES`
  ```
- The BigQuery client sends the SQL query to the data warehouse API and returns results to the notebook.

### 4.3 Goal
- For **parameter-efficient fine-tuning (supervised fine-tuning)**, a dataset with both a **question** and an **answer** is needed.
- Relevant tables: `post_questions` and `answers`

---

## 5. Retrieving Data into a Pandas DataFrame

- Query a table and load results into a **Pandas DataFrame** for exploration.
- Use a **LIMIT** clause to avoid fetching all data at once.
- Workflow:
  1. Write the SQL query
  2. Send to the data warehouse via the BigQuery client
  3. Load result into an **Apache Arrow table** first (for easier conversion)
  4. Convert to a Pandas DataFrame

> **Why Arrow?** It puts data in a format that is easier for Pandas to read.

---

## 6. Dealing with Large Datasets

### 6.1 The Problem
- Querying all rows and columns without a LIMIT may return a **403 error** — data is too large to return.
- Large datasets often do **not fit into memory** of a local or virtual machine.

### 6.2 Best Practices for Large Data

1. **Process in the data warehouse using SQL**
   - Perform joins, filtering, and transformations via SQL rather than exporting all data and processing in Pandas.

2. **Export to fast storage after selection**
   - Once training data is selected, export to an **SSD** or **cloud storage bucket** for fast access during training/tuning.
   - This prevents GPU/accelerator underutilization caused by slow data reads.

3. **Track data lineage**
   - Keep track of where data comes from and what transformations it has gone through.

---

## 7. Optimized Query for Tuning Dataset

### 7.1 Query Design
- Combines two tables: `post_questions` and `answers`
- Uses a **JOIN** on `accepted_answer_id` (unique identifier shared between both tables)
- Uses a **WHERE clause** to:
  - Ensure a valid answer exists in the row
  - Filter only Python-related questions
  - Filter by a date range (time variable)
- Uses a **LIMIT** of 10,000 examples

### 7.2 Output Columns
- `input_text` — Title of the question + the question body
- `output_text` — The accepted answer

---

## 8. Adding an Instruction Template

### 8.1 Why Instructions Matter
- Fine-tuning on instruction-formatted datasets improves model performance and generalization to unseen tasks.
- An **instruction** is a specific direction that conveys a task or action to the LLM.
- Without instructions, the model may not know what to do with a question-answer pair.
- Instructions can be:
  - A rule
  - Step-by-step directions
  - A procedure or an example

### 8.2 Instruction Template
- Example instruction used:
  > *"Please answer the following Stack Overflow question on Python and answer it like you are a developer answering Stack Overflow questions."*
- The template is combined with the `input_text` (question) to create a new column: **`input_text_instruct`**
- A new column is created to avoid overwriting the original `input_text`.

---

## 9. Train/Evaluation Split

- Use **scikit-learn's `train_test_split`** to divide the DataFrame.
- Default split: **80% training / 20% evaluation**
- Evaluation set = unseen data used to evaluate model performance during tuning.
- Use a **`random_state`** parameter to initialize the random number generator for fair, reproducible sampling.

### 9.1 Important: Consistent Parameters
- Keep parameters **as consistent as possible** across experiments to enable fair comparison.
- If too many parameters change at once (e.g., data split + hyperparameters), it becomes difficult to know what impacted model performance.
- Change **one or two parameters at a time** for better traceability.

### 9.2 Why Not Accuracy?
- Accuracy is **not calculated** for this use case because the output is text.
- Text is ambiguous — accuracy as a metric is not applicable here.

---

## 10. File Formats for Training Data

| Format | Type | Best For |
|--------|------|----------|
| **JSON Lines (JSONL)** | Text-based | Small to medium datasets; human-readable; each record on a separate line |
| **TF Record** | Binary | Large datasets; easier for computers to read; efficient for training |
| **Parquet** | Binary | Large and complex datasets; efficient reading during training/tuning |

> **Note:** The choice between TF Record and Parquet depends on preference and the model framework being used.

---

## 11. Versioning & Artifact Management

### 11.1 Why Version?
- LLMOps workflows involve many artifacts (tables, files, models).
- Versioning ensures:
  - **Traceability** — know which dataset generated which file
  - **Reproducibility** — reproduce results consistently
  - **Maintainability** — colleagues can understand and continue the work

### 11.2 Versioning Strategy Used
- File name includes:
  - Type of dataset (training or evaluation)
  - Name of the dataset (e.g., Stack Overflow Q&A)
  - **Timestamp** (date and time of generation) using Python's `datetime` library

---

## 12. Generating the JSONL File

- Only two columns are included in the output file:
  - `input_text_instruct` (instruction + question)
  - `output_text` (answer)
- Use `orient='records'` and `lines=True` when converting the DataFrame to JSONL — each question-answer pair is one line.
- File is written to the **local directory** in this example.
- **Best practice:** For large or many files, write to an SSD or cloud storage bucket.

---

## 13. Key Takeaways

1. Use **SQL + BigQuery** to handle data too large for memory.
2. Filter and join data **within the data warehouse** before exporting.
3. Store processed training data in **fast-access storage** (SSD or cloud bucket) to avoid GPU bottlenecks.
4. Add an **instruction template** to the dataset to guide LLM behavior during fine-tuning.
5. Use an **80/20 train/eval split** with a fixed random state for reproducibility.
6. Choose the right **file format** (JSONL for small/medium; TF Record or Parquet for large datasets).
7. **Version your artifacts** with descriptive names and timestamps.
8. Change **one parameter at a time** across experiments for meaningful comparisons.

---

## 14. What's Next

- The next lecture covers **automation, orchestration, and parameter-efficient fine-tuning**.
