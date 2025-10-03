# Data Documentation Template 

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/10/" style="color:blue; text-decoration:underline">Article 10</a>
  <br>EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 1, 2 (d)
  <!-- info: The AI Act delineates the data governance practices required in Article 10 and requires a description of the intended purpose, version and provider, and relevant versions and updates.  
  In Article 11(2)(d), a datasheet is required which describes all training methodologies and techniques as well as the characteristics of the training dataset, a general description of the dataset, information about its provenance, scope and main characteristics, how the data was obtained and selected, labelling procedures conducted, and data cleaning methodologies deployed. -->
  <p></p>
</div>

**Dataset Owner**: Gruppo 2 (TODO: add accountable data steward name & contact email)
<br>**Document Version**: 0.1.0 (Initial EU AI Act–aligned draft)  
<br>**Reviewers**: TODO: Add legal / compliance / data governance reviewers

**Regulatory Scope Note**: This documentation is restricted to EU AI Act–relevant aspects (Articles 10, 11, 13, 14, 47 and Annex IV) for the dataset used by the travel guide generation / retrieval‑augmented content system contained in this repository.

<!-- info: Replace with dataset name -->

## Overview 
<!-- info: This section enables all stakeholders to have a glimpse into the data processes. You can use this session to provide transparency to users and high level information to all relevant stakeholders. -->

### Dataset Description 

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 1, 2(d)
  <p></p>
  <!-- info: The AI Act requires a description of  all training methodologies and techniques as well as the charatcteristics of the training dataset, general description of the dataset, information about their provenance, scope and main characteristics, how the data was obtained and selected labelling procedures conducted and data cleaning methodologies deployed.-->
</div>


This dataset consists of a small collection of 9 PDF documents (current snapshot) representing travel brochures (Dubai, London, New York, San Francisco, Las Vegas), company background material, and one synthetic/test file containing PII indicators (`document_pii_to_hide.pdf`) intended for redaction validation. The data is used exclusively to power a retrieval‑augmented generation (RAG) workflow that creates structured travel guides and derivative textual outputs. 

Primary purposes: (a) context retrieval for content synthesis, (b) semantic search across brochures, (c) generation of consolidated travel guides, (d) optional creative outputs (e.g., poems) based on the same corpus. 

The dataset is unbalanced across destinations (each brochure is a single file) and is static at present. Where PII is present, the design intent (per provided file name) is to test masking / exclusion prior to indexing. No special categories of personal data (Article 9 GDPR) are intentionally processed; confirmation pending audit (TODO). 

Risk exposure is limited due to domain (travel informational content) and absence of automated decision‑making affecting fundamental rights. Residual risks relate to: (i) inadvertent disclosure of PII if masking fails, (ii) hallucinated or outdated travel information in generated outputs, and (iii) potential copyright issues if verbatim reproduction occurs—mitigation measures outlined below. 

### Status
<!-- scope: telescope -->
<!-- info: Select **one:** -->
**Status Date:** 2025-10-02  
**Status:** Static Snapshot (Limited Maintenance)  
**Justification:** Current corpus is a fixed set of brochures and internal reference PDFs. No scheduled ingestion pipeline. Future additions would require a privacy/compliance re‑review (TODO: define change control workflow).

### Relevant Links
<!-- info: User studies show document users find quick access to relevant artefacts like papers, model demos, etc..
very useful. -->

* GitHub Repository: TODO add remote URL (local path: `guide_creator_flow/`)
* System Architecture / RAG Flow: See project `README.md` (contains operational context) – ensure alignment with data governance (TODO: cross‑reference once updated)
* API / Execution Entry Point: `guide_creator_flow/src/guide_creator_flow/main.py`


### Developers

* TODO: Add developer names and roles (e.g., ingestion maintainer, compliance reviewer, security contact)

### Owner
<!-- info: Remember to reference developers and owners emails. -->
* TODO: Add owning team and accountable data steward contact (email)

### Deployer instructions of Use
<!-- info: Important to determine if there are relevant use-cases or if the data is unsuitable for certair applications. -->
* **Instructions for deployers**:
  1. Run PII detection / masking on any new PDF prior to vectorization (Article 10 – data governance; GDPR alignment).  
  2. Maintain an exclusion list of sensitive terms; verify removal in embeddings store (`vectore_store.py`).  
  3. Provide transparency notice in any user-facing interface: “Generated travel guidance – verify details (prices, regulations) independently.” (Article 13 transparency for limited‑risk generative output).  
  4. Keep an audit log of document ingestion events (TODO: implement persistent logging).  
  5. Re-run quality / drift checks if brochures are replaced (TODO: define checklist).  
  6. Enforce role‑based access to raw documents vs. derived embeddings (Article 11 documentation & security).  

**Unsuitable Uses:** Automated risk scoring of individuals; decisions impacting eligibility, safety‑critical navigation, or legal compliance—out of scope (avoid function creep).  

**Human Oversight:** All generated guides must be manually reviewed before publication (Article 14 principles adapted proportionally for limited risk category).  

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/13/" style="color:blue; text-decoration:underline">Article 13</a>
  <p></p>
</div>

### Version Details

## Data Versioning 

(Article 11, paragraph 2(d))

**Data Version Control Tools:**  
Current: Ad hoc file placement under `input_directory/` (no formal version tags).  
Planned Controls (TODO to implement):  
* Introduce `Data_versioning.md` (changelog: file added/removed, PII status, checksum).  
* Optional: DVC for tracking brochure revisions if volume grows.  
* If large binary updates arise: Git‑LFS for PDFs.  
* Record SHA256 hash per file + ingestion timestamp (Annex IV traceability).  

Rollback Procedure (TODO): Maintain last known good snapshot; revert embeddings store upon detection of PII leakage or corrupted document.

### Maintenance of Metadata and Schema Versioning 

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 3
  <p></p>
</div>

#### Why

Data formats, schema, and other metadata changes can impact downstream processes. Tracking these ensures transparency.

#### How

Create a data dictionary (TODO):  
* Since documents are unstructured PDFs, capture: filename, title (if parsed), page count, language (detected), presence_of_pii (bool), last_scan_tool_version.  

Schema / Format Change Tracking:  
* Each ingestion event appends an entry to `Data_versioning.md` (TODO).  
* If text extraction changes (e.g., OCR added), note tool version & parameters.  

Metadata Co‑Storage:  
* Store JSON manifest (TODO) alongside PDFs: source, license (if any), ingestion timestamp, hash, PII scan result.  
* Quality metrics: token count per doc, OCR confidence (if applicable) (TODO).  
    
  <!-- What could help is to incorporate Data Lineage Tools
   as they provide end-to-end visibility of data transformations and dependencies.-->

## Known Usages 

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 3
  <p></p>
  <!--info: The AI Act requires delineating a system’s foreseeable unintended outcomes and sources of risks to health and safety, fundamental rights, and discrimination in view of the intended purpose of the AI system;  
  the human oversight measures needed in accordance with Article 14, including the technical measures put in place to facilitate the interpretation of the outputs of AI systems by the deployers;  
  and specifications on input data, as appropriate.-->
</div>

<!-- info: Fill out the following section if the dataset has any
current known usages. This is important to make sure that the dataset is used ethically and legally. A dataset created for classification may not be suitable for regression, or vice versa.
Moreover, labeling quality, data coverage, and structure vary with use case—assuming it can be used for anything is dangerous. For instance:A skin lesion dataset created for classification—labeling images as benign, malignant, or uncertain—is mistakenly used by an insurance company to train a regression model that predicts cancer risk scores. Because the dataset lacks continuous risk-related data such as treatment outcomes, progression timelines, or cost indicators, the model produces unreliable predictions. As a result, high-risk patients may be misclassified as low-risk, leading to denied or delayed insurance claims. This misuse not only puts patients at risk but also exposes the insurer to ethical and legal scrutiny. Hence it is important to define the safe extent of use of a dataset. 
-->
### Model(s)
<!-- scope: telescope -->
<!-- info: Provide a table of known models
that use this dataset.
-->

| **Model** | **Model Task** | **Purpose of Dataset Usage** | **Risk Category (EU AI Act)** |
|-----------|----------------|------------------------------|------------------------------|
| RAG Guide Generator (internal) | Retrieval + Text Generation | Provide consolidated travel guide output from brochure corpus | Limited / Minimal risk (informational content; human review) |
| Poem Generator (creative) | Creative Text Generation | Non-factual stylized outputs based on same corpus | Minimal risk |

Note: No high-risk Annex III use cases identified. No prohibited system characteristics present (e.g., subliminal techniques, social scoring).

Note, this table does not have to be exhaustive. Dataset users and documentation consumers at large
are highly encouraged to contribute known usages.

### Application(s)
<!-- scope: telescope -->
<!-- info: Provide a table of known AI/ML systems
that use this dataset.
-->

| **Application** | **Brief Description** | **Dataset Role** | **Human Oversight Mechanism** |
|-----------------|-----------------------|------------------|------------------------------|
| Travel Guide Creation CLI / Script | Generates curated destination guides | Source context retrieval | Manual review prior to distribution |
| Creative Poem Output Tool | Produces themed poems | Inspiration content base | Output labelled as non-factual |

## Dataset Characteristics

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 2(d)
  <p></p>
</div>
<!-- This section reflects the requirements of the AI Act of Article 11, paragraph 2 (d): where relevant, the data requirements in terms of datasheets describing the training methodologies and
techniques and the training data sets used, including a general description of these data sets, information about
their provenance, scope and main characteristics; how the data was obtained and selected; labelling procedures
(e.g. for supervised learning), data cleaning methodologies (e.g. outliers detection). Moreover, in order to comply with GDPR provisions, you need to disclose whether you are handling personal information. -->

**Data Types:** Unstructured text (PDF brochures), potential embedded images (ignored), possible PII (test file).  
**Size/Volume:** 9 PDFs (total size MB: TODO compute).  
**Number of Files:** 9.  
**Primary Use Case(s):** Retrieval‑augmented generation for travel content synthesis; creative derivative text.  
**Associated AI System(s):** RAG Guide Generator, Poem Generator (see model table).  
**Number of Features/Attributes:** Not applicable (no structured tabular features).  
**Label Information:** None (no supervised labels).  
**Geographical Scope:** Travel destinations reflected by brochure topics (multi‑regional).  
**Date of Collection:** Static ingestion date(s) unknown (TODO: determine initial commit / file system timestamps).  

## Data Origin and Source
<!-- importanto to define this step to understand also compliance with GDPR.  -->
**Source(s):** Presumed internal / publicly available marketing or tourism brochures (PDF). TODO: Confirm provenance and licensing.  
**Third-Party Data:** Likely includes third-party brochure content; licensing status unverified (TODO: perform copyright / license review).  
**Ethical Sourcing:** PII test document suggests intentional evaluation of masking pipeline; ensure no real customer sensitive data ingested without legal basis (TODO: confirm lawful basis / DPIA necessity).  

## Provenance

_Describe the history and origin of the data._

### Collection

#### Method(s) Used
<!-- scope: telescope -->
<!-- info: Select **all applicable** methods used to collect data.

Note on crowdsourcing, this covers the case where a crowd labels data
(make sure the reference the [Annotations and Labeling](#annotations-and-labeling)
section), or the case where a crowd is responsible for collecting and
submitting data independently to form a collective dataset.
-->
_Specify one or more of:_

* Taken from existing static PDF brochures (assumed)
* Internal synthetic / placeholder PII test file
* TODO: Confirm if any were scraped or vendor-provided

#### Methodology Detail(s) 

<div style="color: gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> 2 (a), (b), (d)
  <p></p>
</div>
<!-- scope: periscope -->
<!-- info: Provide a description of each collection method used. Use additional notes to capture any other relevant information or
considerations. (Usage Note: Duplicate and complete the following for collection method
type.) -->
**Collection Type:** Static brochure ingestion  
**Source:** Local PDFs in `input_directory/` (see repository).  
**Platform:** Local file system (no external API currently).  
**Is this source considered sensitive or high-risk?** No (marketing content) except PII test file (Yes – restricted).  
**Dates of Collection:** TODO: Derive from initial commit timestamps.  
**Update Frequency:** No changes (ad hoc future ingestion).  
**Additional Notes:** Formalize acceptance checklist before adding new PDFs (TODO).  

#### Source Description(s)
<!-- scope: microscope -->
<!-- info: Provide a description of each upstream source of data.

Use additional notes to capture any other relevant information or
considerations. -->
* **Source 1:** Destination brochure (e.g., Dubai) – marketing descriptive text.  
* **Source 2:** Company information PDF – internal descriptive material (verify confidentiality classification).  
* **Source 3:** PII test PDF – deliberately contains sample identifiers to validate redaction.  

**Additional Notes:** TODO: Provide license statements or internal authorization references.  

#### Collection Cadence
<!-- scope: telescope -->
<!-- info: Select **all applicable**: -->
**Static:** Yes (current state).  
**Streamed:** No.  
**Dynamic:** No scheduled refresh.  
**Others:** Future expansion requires change control + re‑assessment (TODO).  
    
## Data Pre-Processing 

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 2 (d, e)
  <p></p>
</div>

### Data Cleaning

* Handling missing data: Not applicable (unstructured documents).  
* Outlier treatment: Not currently implemented (TODO: consider file size anomaly detection).  
* Duplicates removal: Manual check only (TODO: hash-based duplicate detection).  
* Error correction: PDF text extraction relies on library defaults (TODO: document tool & version).  

### Data Transformation

* Normalization/Standardization: Not applicable.  
* Encoding categorical data: Not applicable.  
* Text/tokenization: Handled implicitly during embedding generation (TODO: specify embedding model + version for reproducibility).  

### Feature Engineering

* Feature selection: Not applicable.  
* Feature extraction: Embedding generation (semantic vectors) – underlying model unspecified (TODO).  
* Newly created features: Vector embeddings per chunk; chunk metadata (source file, page range) (TODO: formal spec).  

### Dimensionality Reduction

* Technique(s) used: None (direct embedding output used).  
* Number of dimensions after reduction: N/A (TODO: list embedding dimension once model confirmed).  

### Data Augmentation

* Augmentation technique(s): None (static text).  

## Data Annotation and Labeling 

<div style="color: gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> 2(d)
  <p></p>
</div>

* Annotation Process: None (no supervised labels).  
* Annotation platform: N/A.  
* Validation: Not applicable.  
* Annotator Demographics: Not applicable.  

## Validation Types

### Method(s) 

Example methods applicable (planned):  
* File integrity (hash) validation  
* PII scan validation (regex + ML-based optional)  
* Consistency check: each embedding vector linked to valid source file  

### Breakdown(s)

**(PII Scan Validation)**  
**Number of Data Points Validated:** TODO: Count of documents scanned.  

### Description(s)


## Sampling Methods


### Method(s) Used


### Characteristic(s)

### Sampling Criteria



### Description(s)

## Dataset Distribution and Licensing 

<div style="color: gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> 2(d)
  <p></p>
</div>

* Availability: Internal / controlled (not publicly redistributed).  
* Open/public or private dataset: Private (pending license confirmation).  
* Dataset Documentation Link: This file (update link if moved).  
* User Rights and Limitations: No redistribution until licenses verified; derived summaries must attribute original brochure source if published externally (TODO: define attribution template).  

## Access, Retention, and Deletion
<!-- info: Where applicable, collect input from privacy governance team -->
### Access

#### Relevant Links

* Storage Location: Local repository `input_directory/` (TODO: move to controlled storage with access logging).  
* Governance process: TODO: Add link to internal data access SOP.  
* Embeddings store: Details in `vectore_store.py` (TODO: confirm persistence backend + encryption).  

#### Data Security Classification in and out of scope delineation
<!-- scope: Is there a potentially harmful application iof this data, can you foresee this?  -->
<!-- info: Select **one**: Use your companies data access classification
standards (replace the classifications below accordingly) -->


#### Prerequisite(s)
<!-- scope: microscope -->
<!-- info: Please describe any required training or prerequisites to access
this dataset. -->
This dataset requires:  
* Completion of internal data handling / privacy training (TODO: reference module).  
* Acknowledgement of acceptable use + prohibition of sensitive re‑purpose.  
* Approval by data steward before adding new PDFs containing personal data.  

### Retention

#### Duration
<!-- scope: periscope -->
<!-- info: Specify the duration for which this dataset can be retained: -->
Retention: Keep only while actively needed for generating travel content; review annually (TODO: define maximum retention, e.g., 24 months).  

#### Reasons for Duration
<!-- scope: periscope -->
<!-- info: Specify the reason for duration for which this dataset can be retained: -->
Justification: Business utility for travel content generation; low sensitivity except PII test file (which should be synthetic and rotated regularly).  

#### Policy Summary
<!-- scope: microscope -->
<!-- info: Summarize the retention policy for this dataset. -->
**Policy:** TODO: Link to corporate data retention & deletion policy.  
  
## Data Risk Assessment

**Foreseeable Unintended Outcomes:** Hallucinated travel facts; outdated pricing or regulations; unredacted PII leakage if masking fails.  
**Potential Bias Sources:** Limited geographic diversity; promotional tone may bias generated summaries toward positive framing.  
**Harm Mitigation:** Mandatory disclaimers; PII scanning pipeline; manual editorial review; logging of generation prompts (TODO: implement logging).  
**Residual Risk Level:** Low (non-high-risk domain) – monitor for privacy incidents.  


## Cybersecurity Measures

<div style="color: gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 5
  <p></p>
</div>


### Data Security Measures

#### Data Storage

* **Encryption (TODO):** Encrypt storage at rest (AES-256) if moved to managed store; currently local dev environment (not compliant for production).  
* **Access Control:** Restrict repository & storage to authorized contributors; implement role-based groups.  
* **Backup:** TODO: Define backup cadence (weekly) + encrypted off-site copy.  
* **Integrity Monitoring:** Maintain SHA256 manifest (TODO).  
* **Security:** Local development environment—needs migration to hardened environment before production use (TODO).  

#### Data Transfer

* **Encryption in Transit:** Not applicable locally; TODO: enforce TLS 1.3 if remote retrieval API introduced.  
* **Endpoint Security:** Developer workstation security baseline (TODO: reference policy).  
* **API Security:** N/A currently (no exposed API).  
* **Data Masking:** PII test content must be excluded from embedding index or token replaced (`[REDACTED]`).  

#### Data Processing

* **Secure Environments:** TODO: Containerize processing with minimal privileges.  
* **Audit Logs:** Implement ingestion + generation event logging (hash, timestamp, user) (TODO).  
* **Data Minimisation:** Only ingest essential brochures; exclude raw PII unless strictly necessary for redaction testing.  



### Standards Applied
 <!-- info: provide information of the standards applied and certifications in this section-->

### Data post-market monitoring

**Data Drift Detection and Monitoring:** Given static corpus, primary drift concern is external factual staleness (information decay). TODO: Establish periodic (e.g., quarterly) manual spot check comparing generated outputs vs. authoritative current sources.  
**Drift Types:** Concept drift (outdated travel regulations), no active covariate drift internally unless new brochure distribution changes.  
**Audit Logs:** TODO: Implement structured log of (user, timestamp, doc added, PII scan result).  
* **Action Plans:** Remove or replace outdated brochures; regenerate embeddings; re‑issue disclaimer update.  



### EU Declaration of conformity

<div style="color: gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/47/" style="color:blue; text-decoration:underline">Article 47</a>
  <p></p>
</div>

 <!-- when applicable and certifications are available: it requires a systems name as well as the name and address of the provider; a statement that the EU declaration of conformity referred to in Article 47 is issued under the sole responsibility of the provider; a statement that the AI system is in conformity with this Regulation and, if applicable, with any other relevant Union law that provides for the issuing of the EU declaration of conformity referred to in Article 47, Where an AI system involves the processing of personal data;  a statement that that AI system complies with Regulations (EU) 2016/679 and (EU) 2018/1725 and Directive (EU) 2016/680, reference to the haharmonised standards used or any other common specification in relation to which
conformity is declared; the name and identification number of the notified body, a description of the conformity
assessment procedure performed, and identification of the certificate issued; the place and date of issue of the declaration, the name and function of the person who signed it, as well as an
indication for, or on behalf of whom, that person signed, a signature.-->

### Standards applied
* TODO: List adopted internal data handling policy ref codes.  
* OWASP ASVS (select controls applicable to data handling – planned).  
* ISO/IEC 27001 alignment (future consideration – not yet certified).  

## EU AI Act Risk Categorisation

| Aspect | Assessment |
|--------|------------|
| System Purpose | Informational content synthesis (travel guide) + creative outputs |
| Potential Annex III Category | None matched (no biometric ID, no employment/safety-critical decision) |
| Risk Level (EU AI Act) | Limited / Minimal Risk |
| Rationale | No automated decisions affecting rights; outputs advisory; human oversight enforced |
| Prohibited Practices Check | No subliminal manipulation, no exploitation of vulnerabilities, no social scoring, no real-time biometric categorisation |

## Compliance Alignment Summary

| Requirement (EU AI Act) | Applicability | Current Measures | Gaps / TODO |
|-------------------------|--------------|------------------|--------------|
| Article 10 (Data Governance) | Applicable (dataset curation, PII handling) | PII test file flagged; manual review before ingestion | Formal data manifest; automated PII scan pipeline |
| Article 11 (Technical Documentation) | Applicable | This document + code repository structure | Complete versioned data manifest; model embedding specification |
| Article 13 (Transparency) | Applicable (generative outputs) | Planned user disclaimer on generated guides | Implement standardized disclaimer injection |
| Article 14 (Human Oversight) | Proportionate oversight | Manual review mandatory pre-publication | Define checklist & sign-off log |
| Annex IV (Documentation Content) | Partially | High-level dataset description | Add hashes, embedding model version, ingestion dates |
| Accuracy / Performance | Limited context (no numerical KPIs) | Human qualitative validation | Define acceptance criteria (e.g., factual correctness sample rate) |
| Robustness | Basic (static corpus) | Deterministic retrieval + generation | Add regression tests against known prompts |
| Cybersecurity (Annex IV §5) | Partially | Local dev only | Encrypt at rest; access controls; audit logs |
| Data Minimisation | Applicable | Only brochures + test PII file | Remove any non-essential personal data |
| Logging & Traceability | Needed | None formal | Implement ingestion + generation logs |

## Human Oversight Framework (Adapted Article 14)
* Oversight Actors: Designated reviewer (TODO: assign role).  
* Review Scope: Factual accuracy, presence of PII, clear disclaimers.  
* Intervention Capability: Reviewer can block release; remove documents; trigger re‑embedding.  
* Tooling Needed (TODO): Checklist template; logging module output.  

## Transparency Measures (Article 13)
* User-facing Notice: “This travel guide is AI-generated from brochure material; verify critical information independently.”  
* Content Labelling: Tag outputs with generation timestamp + source doc list (TODO).  
* Limitation Disclosure: Data may be incomplete; not real-time.  

## Data Governance Controls (Article 10) – Planned Lifecycle
1. Intake Request -> Data Steward Approval (TODO workflow).  
2. PII Scan -> Pass/Fail recorded.  
3. Hash + Metadata Registration.  
4. Embedding Generation (record model + version).  
5. Availability in Retrieval Index.  
6. Periodic Review (staleness & licensing).  
7. Deletion / Revocation if obsolete or non-compliant.  

## Open TODO Consolidated List
* Add developer / owner identities.  
* Confirm provenance & licensing of all PDFs.  
* Implement `Data_versioning.md` + JSON manifest with hashes.  
* Define embedding model name + version; record dimension.  
* Automate PII scanning & redaction pipeline.  
* Introduce audit logging (ingestion + generation).  
* Define retention period & implement deletion workflow.  
* Create oversight checklist & sign-off log.  
* Add standardized user transparency disclaimer injection.  
* Licensing / attribution template for external publication.  
* Security hardening: encrypted storage, RBAC, backups.  
* Drift / factual staleness review schedule.  
* Add accuracy & qualitative evaluation criteria.  
* Containerization + minimal privilege runtime.  
* Reference internal policies / standards explicitly.  


### Documentation Metadata

### Version
0.1.0 (Draft) – 2025-10-02

### Template Version
Derived from internal EU AI Act–aligned data documentation template (customized) – TODO: Link canonical template.

### Documentation Authors
* TODO: Add Author Name, Team (Owner)
* TODO: Add Reviewer (Compliance)
* TODO: Add Reviewer (Security)
