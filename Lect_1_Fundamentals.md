# LLMOps – The Fundamentals
**Course:** LLMOps (Coursera)
**Lecture:** The Fundamentals

---

## 1. What is MLOps?

- **MLOps** stands for Machine Learning Operations — an ML engineering culture and practice that aims to **unify ML development (dev) and ML operations (ops)**.
- **Automation and monitoring** at every step of the ML system is crucial.
- Key activities MLOps addresses:
  - Automating data engineering, model training/tuning, and API deployment
  - Monitoring how an LLM performs in production
  - Integrating pipeline steps (data engineering → tuning → deployment)
  - Testing both individual steps and the end-to-end workflow
  - Managing model releases and code deployments
  - Handling specialized hardware (GPUs, TPUs) for training/tuning

---

## 2. The MLOps Framework / Workflow

A standard MLOps workflow consists of the following sequential steps:

| Step | Description |
|---|---|
| **Data Ingestion** | Pulling data from application or other databases |
| **Data Validation** | Checking for missing data or errors |
| **Data Transformation** | Converting data (e.g., text) into a format suitable for the LLM; handling missing values |
| **Model Training / Tuning** | Training or fine-tuning the model on prepared data |
| **Model Analysis** | Evaluating the model and understanding its performance |
| **Model Serving** | Deciding on and implementing a serving strategy for the use case |
| **Logging** | Logging metrics related to the model in production |

> **Note:** Running each of these steps manually is time-consuming and inefficient. This is where **automation and orchestration** come in.

---

## 3. Orchestration and Automation

- **Orchestration** = telling the system which step to execute first, which one next, and so on (e.g., data ingestion → validation → serving).
- **Automation** = automating this ordered process so it runs without manual intervention.
- This workflow is **one-dimensional per use case** (e.g., one pipeline for a summarization use case).

---

## 4. LLMOps vs. MLOps — Key Differences

### LLMOps (MLOps for LLMs)
Focused **more narrowly** on everything related to LLM development and getting it running smoothly in production. Key concerns include:

- **Experimentation** with multiple foundation models (e.g., PaLM, LLaMA) to find the best fit for a use case
- **Prompt design and prompt management** — both during experimentation and in production
- **Supervised tuning** — fine-tuning a model on domain-specific data (e.g., summaries) to improve performance
- **Monitoring LLMs in production**
- **Evaluating LLMs** — still an active research area; evaluation methods may evolve over time

### LLM System Design
Looks at the **entire application**, which may include work from front-end, back-end, data engineers, and mobile app developers. Key concerns include:

- **Chaining multiple steps** together (e.g., summarizing large documents in batches)
- **Grounding** — providing the LLM with additional factual information to improve output quality
- **Tracking history** — recording past outputs and their performance

---

## 5. High-Level Architecture of an LLM-Driven Application

```
User Interface
      ↓
  User Input
      ↓
  Backend Processing
      ├── Pre-processing (e.g., chunking large input into smaller pieces)
      ├── Grounding (injecting facts into the prompt)
      ├── Prompt → Foundation Model (e.g., PaLM API / LLM)
      ├── Response ← LLM
      ├── Grounding (checking response against known facts)
      ├── Post-processing (cleaning and structuring the response)
      └── Responsible AI check (toxicity, bias, safety)
      ↓
  Final Output → User
```

- **Model Customization** (optional, iterative):
  1. Data preparation
  2. Model tuning
  3. Evaluation
  4. Deploy fine-tuned model into production environment
  5. Use fine-tuned model in the LLM-driven application

---

## 6. LLMOps Pipeline (Course Focus)

The course focuses on building and operating a **model customization workflow**. The pipeline steps are:

1. **Data Preparation** — Preparing and **versioning datasets** to keep track of what was used
2. **Supervised Tuning Pipeline** — A pipeline that performs supervised fine-tuning of the LLM (automated)
3. **Artifact Generation** — Generating a file containing:
   - **Workflow**: the ordered steps of the pipeline (Step A → Step B → Step C...)
   - **Configuration**: parameters used to execute the workflow (e.g., which dataset to use)
4. **Pipeline Execution** — Running the pipeline, which also **deploys the model**
5. **Get Predictions** — Using prompts to get responses from the deployed LLM
6. **Responsible AI Check** — Using **safety scores** to check the safety of LLM responses

---

## 7. Two Essential LLMOps Topics (Course Focus)

### Orchestration
- Defining the **order of steps** in your pipeline
- Example: data preparation → supervised tuning → model deployment

### Automation
- **Automating the pipeline** to reduce manual work and improve developer efficiency

---

## 8. LLM Topics Beyond the Scope of This Course

The following topics are mentioned but **not covered in depth** in this course:

- **Prompt design and prompt management** (during experimentation and in production)
- **LLM evaluation** (only briefly discussed; extensive detail not provided)
- **Monitoring LLMs in production**
- **Testing of LLM systems/applications**

> These are suggested as areas to explore after completing the course.

---

## 9. What's Coming Next in the Course

- The next lesson covers **data preparation** — specifically preparing and versioning datasets for the LLMOps pipeline.
