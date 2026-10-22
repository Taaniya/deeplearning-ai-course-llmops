# LLMOps – Automation and Orchestration with Pipelines

**Course:** LLMOps (Coursera)
**Lecture:** Automation and Orchestration with Pipelines (with Code Example)

---

## 1. Why Automation and Orchestration Matter in LLMOps

- Tuning an LLM is rarely a one-time activity; you may need to tune a model **multiple times** to arrive at one that works for your use case.
- Experimentation is especially critical with LLMs because the field is still learning what works.
- Automating the experiment-running process makes the iteration cycle faster and less error-prone.

---

## 2. MLOps Workflow for Large Language Models

A typical LLM training/tuning workflow follows these stages:

1. **Gather training data**
2. **Training / Tuning** – Fine-tune the model on your data.
3. **Evaluation** – Evaluate during and after training.
4. **Trained model output** – e.g., a TensorFlow SavedModel format.
5. **Deploy to production** – Expose the model as a REST API or batch process.
6. **Generate predictions** on production data.
7. **Ongoing evaluation** – Based on results, update training data and re-run the workflow.

### Three Key Concepts in the Workflow

| Concept | Definition |
|---|---|
| **Orchestration** | Specifying *which step runs first*, then what comes next (ordering of steps). |
| **Automation** | Ensuring you, as a person, do *not* need to manually run each script. The execution of code is automated end-to-end. |
| **Deployment** | Taking your trained model and putting it into a production environment. |

**Distinction between orchestration and automation:**
- *Orchestration* = defining the sequence (e.g., data preparation → model training).
- *Automation* = removing the need to manually execute one file after another.

---

## 3. Frameworks for Orchestration and Automation

Popular frameworks for building ML workflows:
- **Apache Airflow**
- **Kubeflow Pipelines** *(used in this lecture)*

### Kubeflow Pipelines
- An **open-source framework** for constructing machine learning pipelines.
- Provides a **DSL (Domain Specific Language)** library to define pipeline steps and execution logic.
- Key imports used:
  - `kfp.dsl` – the "drawing board" to design/define the pipeline.
  - `kfp.compiler` – compiles the pipeline into an executable YAML file.

#### What is a DSL?
- DSL = **Domain Specific Language** — a set of instructions tailored specifically for building MLOps pipelines.
- With the KFP DSL, you focus on the **what** (the tasks to perform); Kubeflow handles the **how**.

---

## 4. Core Concepts: Components and Pipelines

### Components
- A **component** is a single step in your ML workflow.
- It maps to one self-contained unit of code (e.g., a Python function).
- Examples of steps as components:
  - Step 1: Data processing / data engineering
  - Step 2: Model training
- Each component is decorated with `@dsl.component` to tell the system it is a Kubeflow pipeline component.
- Components **run in containerized environments**.

#### What are Containers?
- Containers are self-contained environments that bundle your dependencies and software code.
- Advantages:
  - No need to manage a server, OS, or install dependencies manually each time.
  - Portable – can execute on environment A, B, or C without depending on the underlying hardware.

### Pipelines
- A **pipeline** is an orchestrated set of components — it defines:
  - The **order of execution** of steps.
  - How **data flows** from one step to another (data can go downstream).
- Decorated with `@dsl.pipeline` to tell the system this is a pipeline.

---

## 5. Hello World Pipeline – Code Walkthrough

### Step 1: Define Component 1 – `say_hello`
```python
@dsl.component
def say_hello(name: str) -> str:
    hello_text = f"Hello, {name}"
    return hello_text
```

### Step 2: Define Component 2 – `how_are_you`
```python
@dsl.component
def how_are_you(hello_text: str) -> str:
    # Takes the output of the first component as input
    ...
```
- The second component is **dependent on the output of the first**.
- The first component must run before the second.

### Step 3: Define the Pipeline – `hello_pipeline`
```python
@dsl.pipeline
def hello_pipeline(recipient: str) -> str:
    hello_task = say_hello(name=recipient)
    how_task = how_are_you(hello_text=hello_task.output)
    return how_task.output
```
- Defines the sequence: `say_hello` runs first, then `how_are_you`.
- The output of `hello_task` is passed as input to `how_are_you`.

### Step 4: Compile the Pipeline
- Use the **compiler** to generate a **YAML file**.
- YAML files are human-readable and store:
  - Pipeline configuration
  - Sequence of steps
  - Component information
  - Dependencies
  - Order of execution

### Step 5: Define Pipeline Arguments
- Arguments (e.g., `recipient = "world"`) are **injected into the pipeline at execution time**.

---

## 6. Passing Data Between Components – Best Practices

- Data from one component can be passed downstream to the next.
- **Best practice:** Pass the **path/URI** to the location of files (e.g., a cloud bucket path), rather than the files themselves.
- **Why?** With large data, placing all data in one container's memory is not scalable. Passing a URI avoids memory bottlenecks.

---

## 7. Executing the Pipeline – Vertex AI Pipelines

### Running on Kubernetes vs. Managed Environment
- You can run a Kubeflow pipeline on a **Kubernetes cluster** – but this requires managing and operating the cluster yourself, which is a lot of work.
- Alternatively, use **Vertex AI Pipelines** (Google Cloud) – a **serverless** managed environment that executes the YAML file for you.

### Setting Up a Pipeline Job (Vertex AI)
Key parameters to configure:
- **Pipeline YAML file** – the compiled pipeline to execute.
- **Job name** – an identifier for the run.
- **Pipeline arguments** – injected at runtime.
- **Region** – e.g., `us-central1`.
- **Pipeline root** – where temporary/artifact files are stored during execution.

```python
job = aiplatform.PipelineJob(
    display_name="hello-pipeline",
    template_path="pipeline.yaml",
    pipeline_root=PIPELINE_ROOT,
    parameter_values={"recipient": "world"},
    location="us-central1"
)
job.submit()
```

### Vertex AI Pipelines UI
- You can visually see the execution of the pipeline (each step/component).
- You can inspect parameters (type and value) and the output of each component.

---

## 8. Reusability of Pipelines

- A key advantage of pipelines is **reusability**.
- Once built, a pipeline can be shared with colleagues or reused for similar use cases.
- Example: A Q&A language model pipeline built for one use case can be reused by a colleague with a similar Q&A requirement.
- Reusing a pipeline means you **only need to update the arguments**, not rebuild from scratch.

---

## 9. Real-World Example – Fine-Tuning PaLM 2 with an Open-Source Pipeline

### Overview
- Uses a **reusable open-source Kubeflow pipeline** for supervised fine-tuning of **PaLM 2** (Google's foundation model).
- Inputs: training data (JSONL) and evaluation data (JSONL) from the previous lab.

### Model Used
- **Text-Bison 001** – a PaLM family model optimized for question answering.
- Fine-tuning method: **Parameter-Efficient Fine-Tuning (PEFT)**.

### Key Pipeline Parameters

| Parameter | Description |
|---|---|
| `model_name` | Name with a date timestamp for versioning (e.g., `qa-model-20241022`). |
| `train_steps` | Number of training steps (recommended: 100–500 for extractive Q&A; lecture uses 200). |
| `evaluation_interval` | Frequency (in steps) at which the model is evaluated against the eval set (default: 20). |
| `dataset_uri` | Path/URI to training data in cloud storage. |
| `evaluation_data_uri` | Path/URI to evaluation data in cloud storage. |
| `location` | GCP region (e.g., `us-central1`). |
| `project_id` | Google Cloud project ID. |

### Training Steps vs. Epochs
- **Training step** = feeding one batch of data into the model, updating internal parameters, and calculating loss/error.
- **Epoch** = a full pass over the entire dataset.
- These are **not** the same concept.

### Caching
- Setting `enable_caching=True` means: if a step was already executed previously with the same code and arguments, it will **not re-execute** that step.
- A step will only re-run if its code or arguments have been updated.
- This saves time and compute cost when iterating on pipelines.

### Execution Note
- Running this pipeline requires **accelerators (GPUs or TPUs)** and can take up to a full day and is computationally expensive.

---

## 10. Summary of Key Takeaways

1. Automation and orchestration are essential in LLMOps to manage the repeated experimentation required when tuning LLMs.
2. **Kubeflow Pipelines** provides a DSL for defining ML pipelines as code using Python functions decorated as components and pipelines.
3. A **component** = one step; a **pipeline** = the orchestrated sequence of steps.
4. Components run in **containers**, making them portable and environment-independent.
5. Best practice: pass **URIs/paths** between components, not the data files themselves, for scalability.
6. Pipelines are compiled into **YAML files**, which are then submitted for execution.
7. **Vertex AI Pipelines** provides a serverless managed layer for executing Kubeflow YAML pipelines on Google Cloud.
8. **Pipeline reusability** is a major advantage – update arguments and rerun instead of rebuilding.
9. **Caching** prevents redundant re-execution of unchanged pipeline steps.
10. Model versioning (e.g., using date timestamps in model names) is important for tracking experiments.
