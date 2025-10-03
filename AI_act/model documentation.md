# Guide Creator Flow (RAG) - Model Documentation
<!-- info: Derived from the project code under src/guide_creator_flow -->

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>, paragraph 1
    <!-- info:  
    The AI Act requires a description of  
    (a) the intended purpose, version, and provider,  
    (b) a description of how the system interacts with software and hardware,  
    (c) relevant versions and updates,  
    (d) all the forms in which the AI system is put into service
    The overview part should also include:  
    (e) the hardware on which the system is intended to run,  
    (f) whether the system is part of the safety component of a product,  
    (g) a basic description of the user interface, and  
    (h) the instructions for use for the deployers. 
    -->
    <p></p>
</div>

**Model Owner**: Gruppo 2 (TODO: add legal entity / contact email)  
**Document Version**: 0.1.0 (from `pyproject.toml`)  
**Reviewers**: Information not available. (TODO: Add names & review dates)

---

## EU AI Act Compliance Summary

### Risk Categorisation

| Category Considered | Assessment | Rationale |
|---------------------|------------|-----------|
| Prohibited Practices (Art. 5) | Not Applicable | System does not engage in subliminal techniques, exploit vulnerabilities of specific groups, perform social scoring, biometric identification, or other prohibited uses. |
| High-Risk (Annex III) | Not Applicable | Intended use limited to internal knowledge assistance / report generation; not in domains listed in Annex III (e.g., employment, education, critical infrastructure, law enforcement). |
| Limited Risk (Transparency Obligations, Art. 52) | Applicable | Conversational / generative answering system producing AI-generated textual outputs; requires user disclosure and review. |
| Minimal Risk | Partially Applicable | Retrieval and formatting functions largely low-risk; generative component elevates to Limited Risk due to potential for misleading content. |

**Declared Risk Level:** Limited Risk. Primary obligations: transparency of AI-generated content, user awareness, basic logging/traceability, mitigation of hallucinations.

### Core Compliance Mapping (Current vs Gaps)

| Requirement Area | Applicability | Existing Elements (Evidence) | Gaps / TODO |
|------------------|--------------|--------------------------------|-------------|
| Transparency (Arts. 13, 52) | Yes | RAG approach enables source-grounding; Markdown report structure; retrieval context available. | TODO: Explicit user-facing AI disclosure banner; cite sources inline or in appendix; document usage limitations. |
| Human Oversight (Art. 14) | Light | Manual invocation & potential human review prior to external sharing. | TODO: Define reviewer role & approval workflow; risk-based escalation for low-confidence outputs. |
| Data Governance (Art. 10) | Yes | Local PDFs only; deterministic chunking; skip-on-error read strategy. | TODO: Data inventory & provenance log; PII detection/redaction; update & removal policy; lawful basis statement. |
| Technical Robustness & Accuracy (Art. 15) | Yes | RAGAS metrics (context_precision, context_recall, faithfulness, answer_relevancy). | TODO: Thresholds & alerting; retry/fallback strategy on LLM failures; robustness testing plan. |
| Cybersecurity / Resilience (Art. 15) | Yes | Secrets via environment variables (no hard-coded keys). | TODO: Threat model; dependency vulnerability scanning; prompt injection defenses; rate limiting & anomaly detection. |
| Logging & Traceability (Arts. 12, 20) | Partial | Potential to log queries/retrieved chunks (not described as implemented). | TODO: Structured logging (query hash, retrieved doc IDs, LLM parameters, timestamps); retention & access policy. |
| Performance Monitoring | Partial | RAGAS CSV output (implied). | TODO: Versioned metric history; regression tests; periodic evaluation schedule. |
| Fairness / Non-Discrimination | Low | Domain: travel/knowledge docs; no demographic targeting. | TODO: Statement confirming absence of protected attribute processing; check for sensitive personal data in corpus. |
| User Instructions (Art. 11(1)(h)) | Yes | Intended use described. | TODO: Operational runbook (inputs, failure modes, disclaimers). |
| Versioning & Change Management | Yes | Version 0.1.0 declared. | TODO: Formal change log for retrieval parameter or pipeline modifications. |
| Risk Management System (Art. 9) | Partial | Implicit quality evaluation via metrics. | TODO: Formal iterative risk assessment & mitigation register. |

### Planned Mitigations (Outline)
1. Transparency: Add AI-generated notice; append source citation table; include limitations section.  
2. Quality Control: Define RAGAS metric thresholds (e.g., minimum faithfulness) and flag low scores.  
3. Data Governance: Implement PII scanning + manifest of documents (filename, source, ingestion date).  
4. Security: Add prompt injection heuristic filter; integrate dependency vulnerability scan.  
5. Oversight: Require human approval for external distribution; log reviewer decisions.  
6. Traceability: Implement structured JSONL logs with unique request IDs.  

### Outstanding TODO Tags
`TODO-TRANSPARENCY-DISCLOSURE`, `TODO-CITATIONS`, `TODO-LIMITATIONS-SECTION`, `TODO-HUMAN-OVERSIGHT-WORKFLOW`, `TODO-DATA-INVENTORY`, `TODO-PII-REDACTION`, `TODO-METRIC-THRESHOLDS`, `TODO-LLM-RETRY`, `TODO-THREAT-MODEL`, `TODO-PROMPT-INJECTION-DEFENSE`, `TODO-LOGGING-SCHEMA`, `TODO-VERSION-CHANGELOG`, `TODO-RISK-REGISTER`

---

## Overview 

<div style="color:gray">
EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>, paragraph 1

<!-- info: This section enables all stakeholders to have a glimpse into the model selection, design, and development processes.  
You can use this section to provide transparency to users and high-level information to all relevant stakeholders.-->
<p></p>
</div>

### Model Type

**Model Type:** Retrieval-Augmented Generation (RAG) pipeline via crewAI Flow with Azure OpenAI Chat LLM and Qdrant vector search (hybrid semantic + full‑text + optional MMR diversification). (Corrected: FAISS references in earlier drafts are obsolete.)

### Model Description 

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>  paragraph 1(a)
    <p></p>
</div>

* Description
  A multi-agent RAG system that: (1) evaluates query relevance, (2) retrieves relevant document chunks from a Qdrant collection built from local PDFs, (3) performs whitelisted travel domain web search, (4) generates an evidence-grounded answer with Azure OpenAI Chat (deployment "gpt-4o"), and (5) produces a Markdown report; RAGAS evaluation is planned (not yet integrated). Orchestration uses crewAI Flow (`PoemFlow`) coordinating crews: `PoemCrew` (relevance), `RagCrew` (hybrid retrieval via `Search.hybrid_search`), `ResearchCrew` (whitelisted web search), and `OutputCrew` (answer + report). Embeddings & LLM use Azure OpenAI via LangChain wrappers.

### Status 
<!-- scope: telescope -->
<!-- info: Select **one:** -->
**Status Date:** 2025-10-02

**Status:** Under Preparation

### Relevant Links

Main code: `main.py`, `utilis/document.py`, `utilis/checker.py`, `utilis/vectore_store.py`, `utilis/search.py`, crew modules under `src/guide_creator_flow/crews/`.

### Developers

Development Team: Jacopo Bonanno, Pietro Montresori, Roberto Parodo, Monica Salvati, Luca Sangiovanni, 
Roles to define (TODO): model risk reviewer, compliance reviewer, security liaison.

### Owner
<!-- info: Remember to reference developers and owners emails. -->
* Team: Gruppo 2 (owner) – contact email distribution list: TODO add

## Version Details and Artifacts 

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 1(c)
    <p></p>
</div>

<!-- scope: periscope -->
<!-- info: Provide details about the current model version
and which model version the current model card corresponds to.

For models without version number, use "Not currently tracked"
but be sure to track the release date of the model.
-->


**Current Model Version:**
0.1.0 (from `pyproject.toml`)

**Model Version Release Date:**
Information not available.

**Model Version at last Model Documentation Update:**
0.1.0

**Artifacts:**

* Qdrant collection (cosine distance, HNSW m=32/ef_construct=256, int8 scalar quantization).
* Source PDFs under `input_directory/` (9 current files – see data documentation).
* PII pre-processing (Azure Text Analytics – current single category) and prompt attack screening (Azure Content Safety) prior to chunk acceptance.
* No custom trainable weights (LLM & embeddings accessed remotely; retrieval deterministic locally).

## Intended and Known Usage

### Intended Use
<!-- info: This section focuses on the initial purpose and/or reasoning
for creating the model. It is important to define this section as the intended use directly affects the AI Act classification. For example:
A face recognition model for personal photo apps → Limited risk
The same model used for law enforcement → High or unacceptable risk


Example Use Case: A university research team develops a machine learning model to predict the likelihood of hospital readmission among diabetic patients over the age of 65, using data from a regional healthcare network. The model is trained and validated specifically on this elderly population and is intended to support hospital planning and academic research. However, the team does not document the model’s intended use or demographic limitations. A health-tech company later integrates the model into a mobile app aimed at helping diabetes patients of all ages manage their care. The model performs poorly for younger users, frequently overestimating their risk of readmission. This leads to unnecessary anxiety, inappropriate self-care decisions, and false alerts to care providers. The misapplication draws criticism for lacking transparency, and regulators question the ethics of deploying a model outside its original context.   -->

* Description
  Assist users in generating evidence-grounded answers and a Markdown report about a travel/tourism query (default topic: "Tourism/Holidays/Trips"). Retrieves local PDFs + curated web results, synthesizes with Azure OpenAI Chat. RAGAS evaluation planned. Execute via `crewai run` or entrypoints `kickoff` / `run_crew`.

### Domain(s) of use

* Description
  Internal knowledge assistance and documentation/report creation on the configured topic domain using local PDF sources.


**Specific tasks performed:**
* Relevance evaluation of user query (`PoemCrew`).
* Document retrieval via Qdrant hybrid retriever (`RagCrew` using `Search.hybrid_search`).
* RAG answer generation and Markdown report authoring (`OutputCrew`).
* RAGAS evaluation (context precision/recall, faithfulness, answer relevancy; optional correctness).

  **Instructions for use for deployers**:
<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 13</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> 
    <p></p>
</div>

### Out Of Scope Uses

Out of scope: Legal, medical, financial, safety-critical guidance; autonomous booking; real-time pricing guarantees.

### Known Applications 

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>  paragraph 1(f)
    <p></p>
</div>

<!-- info: Fill out the following section if the model has any
current known usages.
-->

CLI interactive application generating `output/report.md` (see application documentation for operational flow).

Note, this table may not be exhaustive.  Model users and documentation consumers at large
are highly encouraged to contribute known usages.

## Model Architecture 

<div style="color:gray">
EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>  paragraph 2(b), 2(c)

Info – AI Act requirements:  
This section should contain a description of the elements of the model and the processes of its training and development.  

Article 11(2)(b) requires the design specifications of a system, model selection, and what the system is designed to optimize for, as well as potential trade-offs.  

Article 11(2)(c) requires a description of the system’s architecture, how software components are built on or feed into each other, and the computational resources needed to develop, train, test, and validate the system.
</div>


<!-- Info: Describing the architecture is fundamental for reproducibility, transparency, and effective maintenance. Without clear records of the model’s layers, activation functions, input/output shapes, and training configurations, it becomes difficult to reproduce results, debug issues, or update the model reliably.  -->


* Architecture Description
  The system is a modular RAG pipeline with: (1) data ingestion & sanitisation, (2) chunking & embedding, (3) vector indexing in Qdrant, (4) hybrid retrieval (semantic + full‑text + scoring fusion + optional MMR), (5) multi-agent orchestration for query validation, retrieval, and answer/report generation, and (6) quality / compliance related evaluation (planned RAGAS + logging). There is no model training loop; all intelligence derives from Azure-hosted foundation models and deterministic retrieval logic.

* Key components
  - `utilis/document.py (DocumentLoader)`: Loads PDFs, extracts text, runs PII redaction + prompt shield, sets `trusted` flag (filename heuristic).
  - `utilis/checker.py (Checker)`: Extraction (PyMuPDF) with font size >6 & background filter, Azure Text Analytics PII redaction, Content Safety prompt shield, UTF‑8 cleaning.
  - `utilis/models.py`: Factory for AzureChatOpenAI (deployment `gpt-4o`), AzureOpenAIEmbeddings, Qdrant client.
  - `utilis/vectore_store.py (VectoreStore)`: Recreates collection; upserts points with metadata (`doc_id`, `source`, `title`, `lang`, `text`, `chunk_id`, `trusted`).
  - `utilis/search.py (Search)`: Executes hybrid retrieval: semantic ANN query, full‑text MatchText pre-filter, score normalization and fusion with additive text boost, optional MMR diversification for final selection.  
  - Crews (`src/guide_creator_flow/crews/*`): Orchestrated agents handling query relevance (`poem_crew`), retrieval (`rag_crew`), synthesis/reporting (`output_crew`), plus auxiliary crews for merging or research (where defined).  
  - Tooling (`tools/custom_tool.py`): Custom integration points (details in code if extended).  
  - Environment: Config via `.env` (Azure keys, endpoints) ensuring secrets are not hard-coded.

* Hyperparameter tuning methodology
  Not applicable (no training loop). Retrieval parameters (e.g., HNSW m=32, ef=256; MMR lambda, top_n_semantic, text boost, final_k) are static; no recorded systematic tuning. (TODO: Document rationale & change procedure.)

* Training Methodology
  Not applicable (no model training; builds Qdrant vector collection from ingested PDFs; relies on Azure OpenAI for LLM & embeddings).

* Training duration
  Not applicable.

* Compute resources used
  Local execution environment for preprocessing and retrieval; remote Azure APIs for inference (TODO: Specify minimal hardware baseline & performance characteristics).
     
 
### Data Collection and Preprocessing

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>  paragraph 2(d)
    <p></p>
</div>

<!--check data documentation to avoid duplicates of information and link it in this sectiion

In Article 11, 2 (d) a datasheet is required which describes all training methodologies and techniques as well as the characteristics of the training dataset, general description of the dataset, information about their provenance, scope and main characteristics, how the data was obtained and selected labelling procedures conducted and data cleaning methodologies deployed -->

* **Steps Involved**:
  * Data collection: Local PDF files in configured input directory (current repository `input_directory/`).
  * Text extraction: PyMuPDF (`Checker.extract_pdf_text`) with filtering by font size threshold and exclusion of near-background color text.
  * PII handling: Azure Text Analytics `recognize_pii_entities` invoked (currently filtered category: InternationalBankingAccountNumber). Redacted text used downstream. (TODO: Expand categories and maintain data governance register.)
  * Prompt attack screening: Azure Content Safety `text:shieldPrompt` endpoint; documents flagged with `attackDetected` are excluded from ingestion (rudimentary prompt injection defense). (TODO: Log decisions and broaden criteria.)
  * Data transformation: Recursive chunking (`RecursiveCharacterTextSplitter`) using configured `chunk_size` and `chunk_overlap` with multi-level separators for semantic boundary preservation.
  * Embedding: Azure OpenAI embeddings generated per chunk.
  * Indexing: Qdrant collection recreated per run (destructive) with HNSW + scalar quantization; points upserted with metadata (source, trusted flag, language, chunk ids).


       
### Data Splitting 

* **Subset Definitions**:
  * **Training set**: Not applicable.
  * **Validation set**: Not applicable.
  * **Test set**: Not applicable.
* **Splitting Methodology**:
  * Not applicable.
* **Proportions**:
  * Not applicable.
* **Reproducibility**:
  * Not applicable.
    
**Data Shuffling**:

* Shuffle applied: Not applicable. 
 
## Model Training Process 

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>  paragraph 2(c, g), paragraph 3

<!-- AI Act requirements info:  
In Article 11 paragraph 2(c), details about the computational resources needed to develop, train, test, and validate AI systems are required.  

Moreover, in accordance with Article 11 paragraph 2(g), this section must include the validation and testing procedures used, the data involved, and the main metrics adopted to measure accuracy, robustness, and compliance with the requirements laid out in Chapter III, Section 2.  

Paragraph 3 further requires detailed information about the monitoring, functioning, and control of the system, as well as logging of testing, with reports dated and signed by responsible stakeholders.-->
<p></p>
</div>


**Details of Processes**:

* **Initialisation**: Load environment variables; ingest & sanitize PDFs; (re)create Qdrant collection; embed and upsert chunk vectors; configure hybrid retrieval parameters (semantic top-N, text prefilter, additive boost, optional MMR); instantiate Azure OpenAI LLM & embeddings; set up multi-agent crews.
* **Loss Function**: Not applicable.
* **Optimiser**: Not applicable.
* **Hyperparameters**: Retrieval parameters (k=4, fetch_k=20, lambda_mult=0.3); chunk_size=2000; chunk_overlap=400.
        
## Model Training and Validation 
 
 <div style="color:gray">
 </div>
 
 Objective: Clarify what the model is supposed to achieve. 
 
- Problem statement: Generate evidence-grounded answers and a Markdown article to a user query using local documents.
- Business goals: Produce accurate, faithful, and relevant responses grounded in retrieved context.
- Metrics selected: RAGAS metrics — context_precision, context_recall, faithfulness, answer_relevancy (answer_correctness optionally when reference available).
- Rationale for each metric: Directly aligned with grounding, coverage, and relevance in RAG systems.
- Model predictions on the validation set evalutaion description: Not applicable (no supervised training/validation split). 

<!--
- Performance metrics (e.g., accuracy, F1 score, RMSE) are monitored.
- Documeting this step is important as it enables to detect errors and performance issues early on: Overfitting can be detected using validation loss trends.
-->

<!--### Performance Metrics

- Evaluation Metrics Used (e.g., Accuracy, Precision, Recall, AUC-ROC)
- Benchmarking Results
- Validation Process
- Real-World Performance
- Stress testing
- Performance across different environments and populations -->

**Hyperparameter Tuning**:
  Information not available (static configuration in code).
        
**Regularisation**:
  Not applicable.
    
**Early Stopping**:
  Not applicable.
 
## Model Testing and Evaluation

<!--
- Performance metrics (e.g., accuracy, F1 score, RMSE) are monitored.
- Documeting this step is important as it enables to detect errors and performance issues early on: Overfitting can be detected using validation loss trends.
-->

<!-- Example: In medical diagnosis, using accuracy alone can be misleading in imbalanced datasets, potentially missing critical cases like cancer. Metrics like recall (which measures the percentage of actual cancer cases the model correctly identifies. Critical for minimizing missed diagnoses), precision ( to ensure that when the model predicts cancer, it’s actually correct—important to reduce false alarms), F1 score, and AUC-ROC provide a more meaningful assessment by accounting for the real-world impact of false positives and false negatives. Choosing the right metrics ensures models are effective, trustworthy, and aligned with practical goals and consequences.

## Model Validation and Testing
- **Assess the metrics of model performance** 
   - accuracy:
   - precision: 
   - recall:
   - F1 score:

- **Advanced performance metrics**
  - ROC-AUC:
    - trade-off between true positive rate and false positive rate
  - PR- AUC
     - Evaluating precision and recall trade-off
  - Specificity
    - (True Negatives/(True Negatives+False Positives))
  - Log Loss (Cross-Entropy Loss):
    - Penalises incorrect probabilities assigned to classes.


- **Context dependant metrics**: 
  - Regression Metrics: For tasks predicting continuous values
  - Clustering Metrics: for tasks grouping similar data points
  - Ranking Metrics: for tasks predicting rankings (e.g., search engines recommendation systems)
  - NLP processing metrics (e.g., text classification, sequence-to-sequence tasks)


- **Fairness Metrics**:
    
    - Ensure the model treats different groups (e.g., based on gender, race) equitably.
    - Examples: Demographic parity, equal opportunity, and disparate impact.
- **Explainability Metrics**:
    
    - Measure how understandable and interpretable are the model’s decisions.
    - Examples: Feature importance, fidelity (how well explanations match the model), and sparsity (using fewer features for explanations).
    - 
- **Robustness Metrics**:
    
    - Assess how well the model performs under challenging or unexpected conditions.
    - Examples: Adversarial robustness, performance under data drift, and sensitivity to input changes.
 
- Limitations of the performance after the tests
- Simulate deployment scenarios to understand real-world implications.
- Define thresholds for acceptable performance levels.
- Justify the choice of metrics based on the application’s purpose.
   
--> 

**Performance Metrics**:
    
* Planned RAGAS evaluation outputs (context_precision, context_recall, faithfulness, answer_relevancy; optionally answer_correctness). (TODO: Implement automated execution + persistence, define acceptance thresholds & monitoring cadence.)
 
  **Confusion Matrix**:
     
* Not applicable.
 
  **ROC Curve and AUC**:
     
* Not applicable.
 
  **Feature Importance**:
     
* Not applicable.
 
  **Robustness Testing**:
 
* Information not available.
 
  **Comparison to Baselines**:
     
* Information not available.

### Model Bias and Fairness Analysis 

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>  paragraph 2 (f, g), paragraph 3, 4
    <p></p>
</div>

<!-- info: This section aims to cover the AI Act requirements layed out in Article 11 paragraph 2 g that requires the description of the potential discriminatory impacts. 
Paragraph 4 requires the assessment of the appropriateness of the performance metrics.-->  



![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXclauxwg1nWuPj2z0TcgUK9y69AqHzk_-jQ5BJwYeDkjPSOLVddFcHJ6-oOiuZ2p4Rk3VpqyKw9CvU7N1LOqYtpdjN6CV_hhTxTtpNj4auLmqhsaIQ5fRLIPnVpZOnhtR63YNELlg?key=Lv0_1kRp5_LSkJabUJ8gjQ)Implicit Bias, Measurement Bias, Temporal Bias, Selection Bias, Confounding Bias

#### Bias Detection Methods Used
    

**Pre-processing:** information not available
**In-processing:** information not available
    
**Post-processing:** information not available

**Results of Bias Testing:**
    Information not available.
    

#### Mitigation Measures
    

**Fairness adjustments:** Introduce fairness criteria (like demographic parity, equal opportunity, or equalized odds) into the model training process. 
    
**Adversarial Debiasing:** Use adversarial networks to remove biased information during training. The main model tries to make accurate predictions, while an adversary network tries to predict sensitive attributes from the model's predictions.
    

#### Retraining approaches

**Fairness Regularization:** Modify the model's objective function to penalize bias. This introduces regularization terms that discourage the model from making predictions that disproportionately affect certain groups.
    
**Fair Representation Learning:** Learn latent representations of the input data that remove sensitive attributes, ensuring that downstream models trained on these representations are fair.
    
### Post-Processing Techniques

**Fairness-Aware Recalibration:** After the model is trained, adjust decision thresholds separately for different demographic groups to reduce disparities in false positive/false negative rates.
    
**Output Perturbation:** Introduce randomness or noise to model predictions to make outcomes more equitable across groups.
    
**Fairness Impact Statement:** Explain trade-offs made to satisfy certain fairness criterias
    

## Model Interpretability and Explainability 

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>  paragraph 2(e)
    <p></p>
</div>

**Explainability Techniques Used:**
  <!-- for example: Shapley values, LIME, etc. Both SHAP and LIME are explainability techniques that help to understand why a machine learning model made a specific prediction — especially when the model is a complex "black box" like a random forest, gradient boosting, or deep neural net. Shap Uses game theory to assign each feature a value showing how much it contributed to a prediction. Lime builds a simple, interpretable model (like a linear model) near the point of interest to explain the prediction -->
    
  Examples: 
    SHAP (SHapley Additive exPlanations), 
    LIME (Local Interpretable Model-agnostic Explanations)

**Post-hoc Explanation Models**

* Feature Importance, Permutation Importance, SHAP (SHapley Additive exPlanations), LIME (Local Interpretable Model-agnostic Explanations):
* Partial Dependence Plots (PDP) 
* Counterfactual Explanations
* Surrogate Models
* Attention Mechanisms (for Deep Learning)
    

**Model-Specific Explanation Techniques**

<!-- info: this part is important to delineate why a model makes a decision or to debug and identify if the model is focusing on the right parts of the input. Especially fundamental for models deployed in critical domains such as medical, financial and legal or law enforcement. This section can be useful to draft the user-interface section of the documentation.) -->

* Grad-CAM (Gradient-weighted Class Activation Mapping) for CNNs and RNNs: especially for computer vision applications 
* Layer-wise Relevance Propagation (LRP): Works well for CNNs, fully connected nets, and some RNNs (classification focused)
* TreeSHAP (SHAP for Decision Trees)
    

How interpretable is the model’s decision-making process?  
The retrieval stage is deterministic and explainable: hybrid scoring (normalized semantic similarity + additive boost for text matches) and optional MMR diversification steps are auditable (`Search.hybrid_search`). Answer generation relies on an external foundation model (Azure GPT-4o); explainability currently limited to providing retrieved context (potential for citation list). (TODO: Add systematic citation output + rationale section to enhance transparency.)


<!--
Some technical tools that can aid transparency include:
- Data Lineage Tools: Track the flow and transformation of data (e.g., Apache Atlas, Pachyderm).
- Explainability Libraries: SHAP, LIME, Captum, TensorFlow Explain.
- Version Control Systems: Git, DVC (Data Version Control) for datasets and models. -->

### EU Declaration of conformity 

<div style="color:gray">
    EU AI Act <a href="https://artificialintelligenceact.eu/article/47/" style="color:blue; text-decoration:underline">Article 47</a>(d)
    <p></p>
</div>

 <!-- when applicable and certifications are available: it requires a systems name as well as the name and address of the provider; a statement that the EU declaration of conformity referred to in Article 47 is issued under the sole responsibility of the provider; a statement that the AI system is in conformity with this Regulation and, if applicable, with any other relevant Union law that provides for the issuing of the EU declaration of conformity referred to in Article 47, Where an AI system involves the processing of personal data;  a statement that that AI system complies with Regulations (EU) 2016/679 and (EU) 2018/1725 and Directive (EU) 2016/680, reference to the harmonised standards used or any other common specification in relation to which
conformity is declared; the name and identification number of the notified body, a description of the conformity
assessment procedure performed, and identification of the certificate issued; the place and date of issue of the declaration, the name and function of the person who signed it, as well as an
indication for, or on behalf of whom, that person signed, a signature.-->

### Standards applied

Information not available.

---

## EU AI Act Compliance Gap Table (Consolidated)

| Requirement | Current Evidence | Gap / TODO | Risk if Unaddressed |
|-------------|------------------|------------|---------------------|
| Risk Level Justification | Declared Limited Risk with rationale | Periodic reassessment if scope expands | Misclassification under Annex III if domain changes |
| AI Disclosure | Not implemented | TODO-TRANSPARENCY-DISCLOSURE | Users may rely unaware of AI origin |
| Source Citations | Retrieval exists but no mandated output format | TODO-CITATIONS | Reduced verifiability & trust |
| Limitations Section | Absent | TODO-LIMITATIONS-SECTION | Over-reliance / misuse risk |
| Human Oversight Workflow | Not defined | TODO-HUMAN-OVERSIGHT-WORKFLOW | Lack of accountability for outputs |
| Data Inventory | Not present | TODO-DATA-INVENTORY | Loss of provenance control |
| PII Redaction | Not documented | TODO-PII-REDACTION | Potential personal data exposure |
| Metric Thresholds | Not defined | TODO-METRIC-THRESHOLDS | Undetected quality degradation |
| LLM Failure Handling | Not defined | TODO-LLM-RETRY | Silent failure / incomplete outputs |
| Threat Model | Not created | TODO-THREAT-MODEL | Unaddressed vulnerabilities |
| Prompt Injection Defense | Not implemented | TODO-PROMPT-INJECTION-DEFENSE | Manipulated responses / data leakage |
| Structured Logging | Not implemented | TODO-LOGGING-SCHEMA | Weak traceability & audit gaps |
| Version Change Log | Not implemented | TODO-VERSION-CHANGELOG | Unclear evolution of risk profile |
| Risk Register | Not implemented | TODO-RISK-REGISTER | Missing systematic mitigation tracking |

---

## Limitations & Disclaimers (EU AI Act Context)
* Outputs may contain hallucinations; verification required prior to external use (TODO-TRANSPARENCY-DISCLOSURE).  
* Coverage limited strictly to ingested local PDFs; absence of information does not imply non-existence.  
* No fairness / bias auditing implemented given domain; if domain expands, reassess (TODO-RISK-REGISTER).  
* Security controls (prompt injection filtering, dependency scanning) pending (TODO-PROMPT-INJECTION-DEFENSE).  
* No defined acceptance thresholds for RAGAS metrics yet (TODO-METRIC-THRESHOLDS).  

---

## Documentation Metadata

### Version
<!-- info: provide version of this document, if applicable (dates might also be useful) -->
0.1.0

### Template Version
<!-- info: link to model documentation template (i.e. could be a GitHub link) -->
Information not available.

### Documentation Authors
<!-- info: Give documentation authors credit

Select one or more roles per author and reference author's
emails to ease communication and add transparency. -->

* Information not available.
