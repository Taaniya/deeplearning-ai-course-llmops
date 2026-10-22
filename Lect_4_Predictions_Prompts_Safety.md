# LLMOps – Predictions, Prompts, Safety
**Course:** LLMOps (Coursera)
**Lecture:** Predictions, Prompts, Safety (with code example)

---

## 1. Setup & Prerequisites

- Run setup code to obtain **credentials** and a **project ID**.
- Use a fixed region: **US Central 1**.
- Use the **Vertex AI SDK** for all interactions with the deployed model.
- Initialize the Vertex AI SDK with: project ID, region, and credentials.

---

## 2. Model Deployment Options

Two primary deployment strategies exist:

### 2.1 Batch Deployment
- Use when predictions do **not** need to be real-time.
- Example use case: scoring weekly customer product reviews as positive or negative.
- Process: collect all examples → send them to the trained model → run `.predict()` on the batch.
- This is an **offline process**.

### 2.2 REST API Deployment
- Use when predictions need to be **online and low-latency**.
- Example use case: a Q&A chat application for Stack Overflow-style questions.
- Flow: user sends a question → request hits the API → model predicts → response returns to UI.
- Models can be packaged as REST APIs using libraries like **Flask** or **FastAPI**, then containerized.

---

## 3. Getting Predictions (Code Walkthrough)

### 3.1 Retrieving Endpoints
- Use `model.list_tuned_model_names()` (via the SDK) to retrieve a list of deployed model endpoints.
- Multiple endpoints can exist (e.g., the pipeline was run multiple times → multiple deployments).

### 3.2 Load Balancing
- Use the **`random` library** to randomly select one endpoint from the list.
- This is a basic form of **load balancing** to distribute traffic across deployed models.

### 3.3 Sending a Prompt and Getting a Response
- Load the randomly selected model endpoint.
- Write a prompt (e.g., *"How can I load a CSV file using Pandas?"*).
- Call `.predict(prompt)` on the loaded model to get a response.
- Use **`pprint` (PrettyPrint)** to format the raw response for readability.

### 3.4 Parsing the Response
- The response object contains multiple keys, including:
  - **`content`** – the actual answer/response to the prompt.
  - **`safety_attributes`** – safety scores and metadata.
  - **`citation_metadata`** – citation information, if any.
- Extract the answer using the `content` key.

---

## 4. Latency Considerations

- **Model size**: larger models take longer to respond.
- **Prompt size**: longer prompts increase prediction latency.
- Discuss **permissible latency** with stakeholders before deployment (e.g., is 2 seconds acceptable?).

### Options to Reduce Latency:
- Choose a **smaller language model**.
- Deploy the model on **GPUs**.
- Deploy the model in a **region geographically close** to the application (e.g., deploy in Singapore if the app runs in Singapore).

---

## 5. Training–Production Data Consistency (Skew)

- The model was **trained** on data combining an **instruction + question**.
- If only a question (without the instruction) is sent in production, there is a **mismatch (skew)** between training data and production data.
- This skew **can negatively impact model performance**.
- **Fix:** Prepend the same instruction used during training to every production prompt before sending it to the model.
  - Format: `prompt = instruction + question`
- Ensuring production data matches training data format is critical for consistent model behavior.

---

## 6. Model Monitoring in Production

Key areas to monitor:

| Monitoring Area | Description |
|---|---|
| **Operational Metrics** | How often predictions are requested (request volume, frequency) |
| **Model Performance** | Evaluation metrics of the model in production |
| **Safety** | Detecting bias or harmful language in model output |
| **Scalability** | Load testing the API (sending many simultaneous predictions) |
| **Controlled Rollout** | Start with a small percentage of traffic (e.g., 5%) before full deployment |

---

## 7. Safety Attributes

### 7.1 Why Safety Matters
- LLMs can generate **unexpected, offensive, insensitive, or factually incorrect** outputs.
- The versatility of LLMs makes it hard to predict all possible unintended outputs.
- Developers and practitioners must **test and deploy models safely and responsibly**.

### 7.2 Two Levels of Safety Checking

**Level 1 – Provider-level blocking:**
- Before returning a response, the responsible AI service checks for safety issues.
- If an issue is detected, the response may be **blocked**.
- Check via the `blocked` key in the `safety_attributes` object (value: `True` or `False`).

**Level 2 – Developer-defined thresholds:**
- The response includes **probability scores** for each safety category.
- Developers can use these scores to define **custom thresholds**.
- Example categories: finance, politics, and other safety rating categories.

### 7.3 Safety Score Details
- Each safety attribute has a **confidence (probability) score** between **0 and 1**, rounded to one decimal place.
- The score reflects the **likelihood** of content being unsafe.
- A score of `0.1` is very low (safe); scores closer to `1` indicate higher risk.
- There is also a **severity score**, which is distinct from the probability score.

### 7.4 Probability vs. Severity – Key Distinction

| | Example |
|---|---|
| **Higher probability, lower severity** | *"The panda pinned me down"* – more likely to be flagged as unsafe, but context is ambiguous |
| **Lower probability, higher severity** | *"The panda beat me"* – ambiguous (could be a race), but if taken literally, the harm could be more severe |

- **Safety filter confidence scores are based on probability of being unsafe, NOT severity.**
- Important: content with a **low probability** of being unsafe can still have a **high severity** of harm.

### 7.5 Setting Thresholds
- There is **no universal threshold** that works for all use cases.
- Thresholds depend on:
  - The specific use case.
  - The type of users the application serves.
  - Other project requirements.
- Practitioners must **examine their own data and model responses** to decide acceptable thresholds.
- Confidence scores are **predictions only** and should not be the sole basis for reliability decisions.

---

## 8. Citation Metadata

- When dealing with generative AI, it is preferable to produce **original content** rather than replicate existing content.
- Systems should be designed to **limit the chance of replicating existing content at length**.
- The model response includes **`citation_metadata`**, which can be checked for citations.
- If a response is based on existing content, the model returns the **source URL or page** it was cited from.
- If no citation exists, the metadata will be empty (the response is original).
- Suggested experiment: try prompts from well-known works (e.g., Shakespeare's *"To be or not to be"*) to observe if citations are triggered.

---

## Summary of Key Takeaways

1. Use **Vertex AI SDK** to interact with deployed tuned models.
2. Choose **batch** for offline use cases; choose **REST API** for real-time use cases.
3. Use **random load balancing** to distribute requests across multiple deployed endpoints.
4. Always ensure **production data format matches training data format** to avoid skew.
5. Monitor models in production across: operational metrics, performance, safety, scalability, and rollout strategy.
6. Leverage **safety attributes** at two levels: provider-side blocking and developer-defined thresholds.
7. Understand the difference between **probability scores** (likelihood of harm) and **severity scores** (degree of harm).
8. Use **citation metadata** to detect if model responses replicate existing content.
