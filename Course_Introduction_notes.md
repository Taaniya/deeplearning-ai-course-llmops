# LLMOps – Introduction
**Course:** LLMOps (built in partnership with Google Cloud)
**Instructor:** Erwin Huizenga (Developer Advocate, Machine Learning, Google Cloud)

---

## 1. What is LLMOps?

- **LLMOps** is an extension of **MLOps (Machine Learning Operations)** applied specifically to applications built on **Large Language Models (LLMs)**.
- It describes the **processes and tools** used to automate the cycle of:
  - Data preparation
  - Model tuning
  - Deployment
  - Maintenance
  - Monitoring

---

## 2. Why LLMOps Matters

- Building a single LLM-based use case (e.g., summarizing customer emails) involves significant work:
  - Selecting the right LLM (often requires trying multiple models)
  - Tuning prompts
  - Setting up an evaluation framework
  - Deploying the application
  - Monitoring performance
- As the number of use cases grows (e.g., from 1 to 5+: summarizing, classifying, routing emails, etc.), the work multiplies proportionally.
- **Automated deployment and monitoring tools** reduce this burden significantly.
- When an LLM provider **deprecates a model**, tools to help migrate and test simplify the transition.

---

## 3. Key Complexities LLMOps Addresses

### Prompt Management
- Designing effective prompts requires an automated way to:
  - **Evaluate and compare** the performance of different prompts
  - **Update prompts** when the underlying LLM changes

### Multi-Step LLM Applications
- Applications that use **multiple LLM calls** (e.g., chaining processing steps) add additional complexity.
- Orchestration frameworks like **LangChain** are used for this.
- Managing **dependencies** across chained steps is a key challenge LLMOps helps address.

---

## 4. Context: Generative AI and Developer Community

- Rapid advancements in generative AI and LLMs have led to a surge in creativity and developer use cases.
- More use cases mean more applications to **build and manage**.
- Effective management and operational strategies are critical to **ship code to real users efficiently**.
- Building AI applications has become easier due to:
  - **Foundation models available as APIs**
  - **Open-source LLMs**
- This ease of building means developers may build **multiple use cases** simultaneously, increasing management complexity.
- Good LLMOps tooling and processes are essential to manage and scale multiple projects efficiently.

---

## 5. What You Will Learn in This Course

1. How to build an **end-to-end workflow** for an LLM-based application.
2. How to **prepare data for tuning** a large language model (e.g., using PEFTs – Parameter-Efficient Fine-Tuning methods).
3. How to **automate and orchestrate** the LLM tuning workflow for multiple use cases, including handling large text datasets that may not fit in memory.
4. How to **deploy a model as a REST API** and call the API.
5. How to build an **overall LLMOps workflow**.

### Tools and Technologies Covered
- **SQL** – for data preparation
- **Open-source tools** – for orchestration and automation
- **Vertex AI** – for deploying models to production

---

## 6. Prerequisites

- **No prior LLMOps knowledge required.**
- Familiarity with MLOps is helpful but not necessary.

---

## 7. LLMOps vs. MLOps

- LLMOps builds on MLOps concepts.
- LLMs introduce new challenges (prompt management, model deprecation, chaining, etc.) that push **beyond what traditional MLOps addressed**.
- These differences will be explored further in subsequent lectures.

---

*Notes based solely on the lecture transcript. No additional information has been added.*
