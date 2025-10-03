# Application Documentation

**Application Owner**: Gruppo 2 (owner team) – TODO: add official distribution email
<br>**Document Version**: 0.1.0
<br>**Reviewers**: TODO - Assign technical and legal reviewers for EU AI Act compliance validation.

## Key Links

* Code Repository: local path `guide_creator_flow/` (remote URL: TODO add Git origin)
* Deployment Pipeline: Not yet defined (local execution only) – TODO design CI (lint/tests/security) + CD (container/publish)
* API: Local CLI / flow execution only (no network API exposed)
* Cloud Account: Azure OpenAI Service (subscription / resource group identifiers: TODO)
* Project Management Board: TODO (e.g. GitHub Projects / Azure Boards link)
* Application Architecture: `src/guide_creator_flow/main.py`, `src/guide_creator_flow/crews/**`, `utilis/document.py`, `utilis/checker.py`, `utilis/vectore_store.py`, `utilis/search.py`, `utilis/models.py`, `src/guide_creator_flow/tools/custom_tool.py`

## General Information 

<div style="color: gray">
EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 1, 2, 3
<!-- info: this section covers the AI Act requirement of a description of the intended purpose, version and provider, relevant versions and updates. In Article 11, 2(d) a datasheet is required which describes all training methodologies and techniques as well as the characteristics of the training dataset, general description of the dataset, information about their provenance, scope and main characteristics, how the data was obtained and selected, labelling procedures conducted, and data cleaning methodologies deployed. -->
<p></p>
</div>


**Purpose and Intended Use**:
    
* The application orchestrates a multi-agent AI flow (crewAI Flow) to: (1) evaluate topical relevance of user questions related to tourism/travel, (2) retrieve relevant information from local PDF brochures and trusted web sources (whitelisted domains), (3) generate evidence‑grounded answers and structured Markdown reports, and (4) provide comprehensive travel guidance.
* Problem addressed: enabling intelligent information retrieval and synthesis for tourism-related queries using both local knowledge bases and curated web sources from trusted travel platforms.
* Target users and stakeholders: travel consultants, tourism professionals, researchers, and developers working on travel recommendation systems; primarily for professional/commercial use in tourism industry.
* KPIs: Answer relevance and accuracy, user satisfaction with travel recommendations, system availability, response time, other metrics related to the llm output content (conciseness, argument strength, document redundancy...).
* Ethical/regulatory considerations: transparency about AI-generated content, source attribution for travel information, data privacy for user queries, avoiding biased travel recommendations. TODO: Complete DPIA determination (likely low-risk justification).
* Prohibited uses: Critical safety decisions (emergency services, medical advice during travel), legal advice regarding travel regulations, financial advice for travel investments, autonomous booking without human oversight.
* **Operational environment:** Local Python execution; Azure OpenAI APIs; whitelisted travel domains (lonelyplanet.com, tripadvisor.com, booking.com, etc.). No persistent server yet.


## Risk classification

<div style="color: gray">
Prohibited Risk: EU AI Act Chapter II <a href="https://artificialintelligenceact.eu/article/5/" style="color:blue; text-decoration:underline">Article 5</a>
<br>High-Risk: EU AI Act Chapter III, Section 1 <a href="https://artificialintelligenceact.eu/article/6/" style="color:blue; text-decoration:underline">Article 6</a>, <a href="https://artificialintelligenceact.eu/article/7/" style="color:blue; text-decoration:underline">Article 7</a>  
<br>Limited Risk: Chapter IV <a href="https://artificialintelligenceact.eu/article/50/" style="color:blue; text-decoration:underline">Article 50</a>
<p></p>
</div>

<!--info: The AI Act classifies AI systems into four different risk categories. The EU AI Act categorizes AI systems into four risk levels: unacceptable, high, limited, and minimal risk, each with corresponding regulatory requirements.  
Unacceptable risk (Chapter II, Article 5) includes systems that pose a clear threat to safety or fundamental rights (e.g. social scoring, recidivism scoring) and are banned.  
High-risk systems are delineated in Chapter III, Section 1, Articles 6 and 7, including AI used in sensitive domains like healthcare, law enforcement, education, employment, and critical infrastructure. These must meet strict requirements and conduct conformity assessment practices, including risk management, transparency, and human oversight.  
Limited-risk systems, delineated in Chapter IV Article 50, such as chatbots, must meet transparency obligations (e.g. disclosing AI use).  
Minimal-risk systems, like spam filters or AI in video games, face no specific requirements. -->

* **Limited risk** (AI system that interacts with natural persons through content generation and information provision).
* **Reasoning**: The system generates travel-related content and recommendations in response to user queries, which constitutes interaction with natural persons as defined in Article 50. The system does not fall under prohibited uses (Article 5) or high-risk categories (Annex III) as it does not involve: biometric identification, critical infrastructure management, education/vocational training assessment, employment decisions, essential private/public services access, law enforcement, migration/asylum/border control, or administration of justice/democratic processes.
* **EU AI Act compliance requirements**: As a limited-risk system, must comply with transparency obligations under Article 50 - users must be informed they are interacting with an AI system.
   
## Application Functionality 

<div style="color: gray">
EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>, paragraph 1, 2, 3
<!-- Info: this section covers the delineation of the general purpose of the system required in Article 1, with a focus on defining what the system should do and how it should work.-->
<p></p>
</div>


* **Instructions for use for deployers**: <div style="color: gray">(EU AI Act <a href="https://artificialintelligenceact.eu/article/13/" style="color:blue; text-decoration:underline">Article 13</a>)</div>
* **Environment Setup**: Export `AZURE_API_BASE`, `AZURE_API_KEY`, `AZURE_API_VERSION`.
* **Installation**: Python >=3.10 <3.14. `pip install uv` then `crewai install` at project root.
* **Data Preparation**: Place travel-related PDF documents in `input_directory/` folder for local knowledge base.
* **Execution**: Run `crewai run` (or entrypoints `kickoff` / `run_crew`).
* **Transparency Compliance**: System displays "AI-generated content" disclaimer to users as required by Article 50.
* **User Interaction**: System prompts for tourism-related questions and routes irrelevant queries appropriately.
* **Model Capabilities**:
  * Can: relevance evaluation; retrieval (hybrid Qdrant semantic ANN + full‑text + optional MMR diversification); evidence-based recommendations; structured Markdown report generation.
  * Cannot: make autonomous travel bookings, provide real-time pricing, access non-whitelisted websites, provide medical or legal travel advice, operate without human oversight.
  * Supported languages: English and Italian for user queries; multilingual content processing via Azure OpenAI models.
* **Input Data Requirements**:
  * Local documents: Travel-related PDFs in `input_directory/` (current corpus: 9 PDFs).
  * User queries: Natural language questions about tourism, travel, holidays, or trips.
  * Invalid inputs: Non-tourism questions (routed to "Not Relevant"), missing Azure credentials, corrupted documents.
* **Output Explanation**:
  * Primary output: Structured Markdown report (`output/report.md`) with recommendations (citations planned – TODO implement inline numbering).
  * Content includes: direct answers, expanded explanations, key points, supporting evidence with source attribution.
  * Transparency: All AI-generated content clearly marked; sources explicitly cited; limitations stated when information is insufficient.
* **System Architecture Overview**:
  * Multi-agent flow: Relevance evaluation → Parallel retrieval (local PDF via Qdrant + web search) → Content generation → Report compilation.
  * AI Agents: Topic relevance evaluator, RAG specialist, web research agent, Markdown documentation writer.
  * Data sources: Qdrant vector store for local documents, whitelisted travel websites, Azure OpenAI for embeddings and text generation.

## Models and Datasets

<div style="color: gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 2 (d)
<p></p>
</div>

<!--All information about models and datasets that are used in the application should be found in their respective dataset or model documentation.  The purpose here is mainly to provide links to those documentation. --> 
<!--In Article 11, 2 (d) a datasheet is required which describes all training methodologies and techniques as well as the charatcteristics of the training dataset, general description of the dataset, information about their provenance, scope and main characteristics, how the data was obtained and selected labelling procedures conducted and data cleaning methodologies deployed -->

### Models

Link to all model integrated in the AI/ML System

| Model   | Link to Single Source of Truth | Description of Application Usage |
|---------|--------------------------------|----------------------------------|
| AzureChatOpenAI (deployment "gpt-4o") | Defined in `utilis/models.py` | Relevance evaluation, answer & report generation.
| AzureOpenAIEmbeddings | Defined in `utilis/models.py` | Embeddings for Qdrant vector store retrieval.

### Datasets

Link to all dataset documentation and information used to evaluate the AI/ML System.  
(Note, Model Documentation should also contain dataset information and links for all datasets used to train and test each respective model) 

| Dataset / Source Folder | Link to Single Source of Truth | Description of Application Usage |
|-------------------------|--------------------------------|----------------------------------|
| Local PDFs under `guide_creator_flow/input_directory/` | Repository directory | Source corpus for ingestion & embedding (9 files).

**Current PDF Corpus:** `document_pii_to_hide.pdf`, `Dubai Brochure.pdf`, `dubai_hotels_untrusted.pdf`, `Las Vegas Brochure.pdf`, `London Brochure.pdf`, `Margies Travel Company Info.pdf`, `New York Brochure.pdf`, `prova.pdf`, `San Francisco Brochure.pdf`

## Deployment
    
Current state: local-only execution; production deployment plan pending.

### Infrastructure and Environment Details

* **Cloud Setup**: Local runtime + Azure OpenAI (chat & embeddings). No containerization yet (TODO: create Dockerfile).
* **APIs**: Azure OpenAI (version from `AZURE_API_VERSION`), Azure Text Analytics (PII), Azure Content Safety (prompt shield). Auth: `AZURE_API_KEY`. Latency benchmarks: TODO.

## Integration with External Systems

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a> ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 1 (b, c, d, g, h), 2 (a)
  <p></p>
</div>

* **Systems**:
  * Dependencies: crewAI, LangChain, Qdrant client, PyMuPDF, Azure Text Analytics, Azure Content Safety, Azure OpenAI, (planned) RAGAS.
  * Data flow: User input → Relevance (`PoemCrew`) → Parallel local retrieval (`RagCrew`) + whitelisted web search (`ResearchCrew`) → Synthesis (`OutputCrew`).
  * Error-handling: Basic try/except; ingestion skips flagged PDFs (prompt shield True). TODO: structured logging & error codes.

## Deployment Plan

* **Infrastructure**:
  * Environments: Development (local). Staging/Production: TODO define.
  * Resource scaling/backup: N/A single user. TODO: snapshot Qdrant & docs for backup.
* **Integration Steps**:
  * Install deps; set Azure env vars; place PDFs under `docs/`; run the flow.
  * Dependencies: see `pyproject.toml`.
  * Rollback: Revert Git commit; rebuild Qdrant index (procedure: TODO document runbook).
* **User Information**: Local developer machine.


## Lifecycle Management

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 6
  <p></p>
</div>
    
* Monitoring procedures: TODO implement lightweight JSONL logging + periodic metrics review.
* Versioning: `pyproject.toml` version 0.1.0. Changelog file: TODO (`CHANGELOG.md`).
* **Metrics**:
  * Application performance: TODO collect latency & retrieval time stats.
  * Model performance: RAGAS metrics planned (faithfulness, context_precision/recall, answer_relevancy, correctness).
  * Infrastructure: Not instrumented (TODO add simple profiler).
* **Key Activities**:
  * Evaluate outputs with RAGAS (`ragas_results.csv`) and iterate.
  * Update PDFs and rebuild FAISS index when documents change.
  * Dependency/model updates: Information not available.
* **Documentation Needs**:
  * **Monitoring Logs**: TODO define format (`logs/events.jsonl`).
  * **Incident Reports**: TODO create template.
  * **Retraining Logs**: Not applicable (no training) – embed rebuild events should be logged.
  * **Audit Trails**: TODO structured query + retrieval logging.
  **Maintenance of change logs** categories: new, updated, deprecated, removed, bug fixes, security.
* new features added
* updates to existing functionality
* deprecated features
* removed features
* bug fixes
* security and vulnerability fixes

### Risk Management System

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/9/" style="color:blue; text-decoration:underline">Article 9</a>
  <br>EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>
  ; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a>
  <p></p>
</div>
<!--**Instructions:**  A thorough risk management system is mandated by the AI Act, especially for high-risk AI systems. This section documents the  proactive efforts to ensure the AI system operates safely and ethically. In general in this section you should document all the measures undertaken to make sure that a system operates safely on the market. Example: Consider a facial recognition system used for real-time law enforcement in public spaces. This is categorized as high-risk under the EU AI Act. If developers document the risk that the system might misidentify individuals—particularly among minority groups due to biased training data—they can plan for rigorous dataset audits, independent bias testing, and establish human oversight in decision-making. Without documenting this risk, the system might be deployed without safeguards, leading to wrongful detentions and legal liabilities. Systematic documentation ensures these issues are not only identified but addressed before harm occurs.-->


**Risk Assessment Methodology:** Categorization under EU AI Act + operational risk review; update per release (TODO risk register file).

**Identified Risks:** 

1. **Content Accuracy Risks**: AI-generated travel information may contain inaccuracies or outdated recommendations
2. **Bias and Discrimination**: Potential bias in travel recommendations based on training data or source selection
3. **Privacy Risks**: User queries may contain personal travel preferences or sensitive location information
4. **Dependency Risks**: System relies on external Azure OpenAI services and whitelisted web sources
5. **Misinformation Propagation**: Risk of amplifying incorrect travel information from source documents

**Potential Harmful Outcomes:** Inaccurate or outdated travel info; destination bias; PII exposure if redaction insufficient; Azure outage.

**Likelihood and Severity:** Medium likelihood minor inaccuracies; low severity (informational scope + human review) – monitor once citations added.

#### Risk Mitigation Measures

**Preventive Measures:** 
 - Topic relevance filtering (PoemCrew)
 - Source whitelisting (10 domains list in code)
 - Evidence grounding (planned explicit citation extraction – TODO)
 - Transparency disclaimer (planned – TODO)
 - Human oversight (manual review required)

**Protective Measures:** 
 - Information gap disclosure (TODO add low retrieval count message)
 - Source attribution (planned; not enforced yet)
 - Local data processing (no external storage)
 - Error handling (basic; improve with structured errors TODO)
 - Content validation (multi-agent; add metrics gating TODO)

## Testing and Validation (Accuracy, Robustness, Cybersecurity)

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/15/" style="color:blue; text-decoration:underline">Article 15</a>
  <p></p>
</div>

**Testing and Validation Procedures (Accuracy):**
- **Automated Quality Assessment**: System generates evaluation metrics using RAGAS framework for each response
- **Multi-Agent Validation**: Cross-verification between RAG specialist and documentation writer agents
- **Source Verification**: All generated content traced back to original sources for accuracy validation
- **Human Review Process**: Generated reports require human review before deployment or distribution

**Performance Metrics:** 
- **RAGAS Metrics**: context_precision, context_recall, faithfulness, answer_relevancy, answer_correctness
- **Response Quality**: Source attribution completeness, information gap identification accuracy
- **System Reliability**: Uptime, error rates, response times for user queries
- **Compliance Metrics**: Transparency disclosure compliance, human oversight verification

**Validation Results:** Planned RAGAS metrics to `ragas_results.csv` (implementation pending – TODO thresholds & schedule).

**Measures for Accuracy:** 
- **Document Quality Control**: Ensure travel documents are current, accurate, and from reliable sources
- **Parameter Optimization**: Tune retrieval parameters (chunk_size: 700, chunk_overlap: 120, final_k: 6) based on performance metrics
- **Hybrid Search Configuration**: Balance semantic and keyword search (text_boost: 0.20, mmr_lambda: 0.6) for optimal retrieval
- **Continuous Monitoring**: Track accuracy metrics over time and adjust system parameters accordingly

  
### Accuracy throughout the lifecycle

**Data Quality and Management:** 
- **Document Lifecycle Management**: Regular review and update of travel documents in knowledge base
- **Content Validation**: Verify extracted text quality from PDFs; implement PII detection for sensitive information
- **Vector Store Maintenance**: Rebuild Qdrant index when documents are updated; monitor index performance
- **Source Trustworthiness**: Maintain whitelist of trusted travel websites; regular validation of source reliability

**Model Selection and Optimization:** 
- **Primary Models**: Azure OpenAI GPT-4o for text generation, Azure OpenAI embeddings for semantic search
- **Fallback Systems**: Sentence Transformers for local embedding generation when Azure unavailable
- **Configuration Management**: Centralized parameter control in utility modules (`utilis/vectore_store.py`, `utilis/document.py`)
- **Performance Tuning**: Regular optimization based on RAGAS metrics and user feedback

**Feedback Mechanisms:** 
- **Automated Assessment**: RAGAS metrics generated for each response and stored in CSV format
- **Human Review Loop**: Manual evaluation of generated reports for accuracy and completeness
- **Iterative Improvement**: Regular updates to document collection, prompt engineering, and system parameters
- **User Feedback Integration**: TODO - Implement user satisfaction scoring and feedback collection system

### Robustness 

**Robustness Measures:**

- **Input Validation**: Comprehensive validation of user queries, environment variables, and document formats
- **Error Handling**: Graceful handling of PDF parsing errors, network failures, and API timeouts
 - Fallback mechanisms: (Planned) local embedding model + simple cache (not implemented yet).
- **System Recovery**: Automatic retry logic for transient failures; clear error messages for permanent failures
- **Data Integrity**: Checksums and validation for document processing; consistent vector store operations

**Scenario-Based Testing:**

- **Off-Topic Queries**: System correctly routes non-tourism questions to "Not Relevant" response
- **Missing Resources**: Empty document collections handled gracefully with appropriate user messaging
- **Service Disruption**: System degrades gracefully when Azure OpenAI services unavailable
- **Malformed Input**: Robust handling of corrupted PDFs, invalid queries, and configuration errors
- **Load Testing**: TODO - Implement stress testing for concurrent user scenarios

**Redundancy and Fail-Safes:**
    
 - Multiple data sources: local + whitelisted web
 - Dual embedding models: planned fallback (TODO)
 - Validation layers: multi-agent + human review (add metric gate TODO)
 - Redundancy: TODO (vector store snapshot, failover endpoints)
    
**Uncertainty Estimation:**
    
 - Confidence indicators: TODO (derive from retrieval coverage & faithfulness)
 - Source quality metrics: metadata `trusted` flag (surface in output – TODO)
 - RAGAS scores: planned
 - Quantitative scoring: TODO
    

### Cybersecurity 

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 2 (h)
  <p></p>
</div>

**Data Security:**
 - Local processing only
 - Credentials via environment variables
 - PII protection: Azure Text Analytics category filter (needs expansion – TODO)
 - Prompt injection: ingestion-time Content Safety shield (extend to runtime – TODO)
 - Encryption at rest: TODO (if external storage introduced)

**Access Control:**
- **Local Execution Model**: Single-user local execution environment; no network-exposed endpoints
- **API Security**: Secure Azure OpenAI API key management with environment variable isolation
- **Document Access**: File system permissions control access to local document collections
- **TODO**: Implement role-based access control for multi-user deployment scenarios

**Incident Response:**
 - Logging: stdout only (TODO structured JSONL)
 - Monitoring: none (TODO lightweight dashboard)
 - Alerting: TODO (define triggers)
 - Recovery procedures: TODO
 - Security updates: manual dependency review (add `pip-audit` pipeline – TODO)

**Additional Security Measures:**
 - Input sanitization: basic relevance gating
 - Network security: domain whitelist enforced in query composition
 - Audit trail: TODO implement query + retrieval logging
 - Security assessment / pen-test: TODO pre-production

  

## Human Oversight 

<div style="color:gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/11/" style="color:blue; text-decoration:underline">Article 11</a>;; <a href="https://artificialintelligenceact.eu/annex/4/" style="color:blue; text-decoration:underline">Annex IV</a> paragraph 2(e)
  <br>EU AI Act <a href="https://artificialintelligenceact.eu/article/14/" style="color:blue; text-decoration:underline">Article 14</a>
  <p></p>
</div>

<!-- info: AI Act Article 11, paragraph 2(e) requirements: assessment of the human oversight measures needed in accordance with Article 14, including the assessment of the technical measures needed to facilitate the integration of the outputs of the AI systems by deployers. -->


**Human-in-the-Loop Mechanisms:**  
- **Interactive Query Process**: System requires human initiation for each query; users must provide explicit input
- **Relevance Validation**: If query deemed non-relevant to tourism, user is prompted to reformulate
- **Output Review**: All generated reports require human review before use or distribution
- **Quality Control**: Human validation of RAGAS metrics and content accuracy before deployment

**Override and Intervention Procedures:** 
- **Manual Control**: Users can interrupt execution at any point and modify inputs or system configuration
- **No Autonomous Operation**: System never operates without explicit human commands or supervision
- **Configuration Override**: Users can adjust retrieval parameters, source preferences, and output formats
- **Emergency Stop**: Immediate termination capabilities for all system processes

**User Instructions and Training:** 
- **Setup Documentation**: Comprehensive installation and configuration guide in repository README.md
- **Usage Guidelines**: Clear instructions for formulating effective tourism-related queries
- **EU AI Act Compliance Training**: TODO - Develop training materials on transparency obligations and human oversight requirements
- **Best Practices**: Guidelines for reviewing AI-generated content and validating source accuracy

**Limitations and Constraints of the System:** 
- **Domain Scope**: Limited to tourism/travel-related queries; other topics routed to "Not Relevant"
- **Data Dependencies**: Accuracy depends on quality of local documents and availability of whitelisted web sources
- **Technical Requirements**: Requires Azure OpenAI API access and proper environment configuration
- **Language Support**: Optimized for English and Italian; other languages may have reduced accuracy
- **Real-time Limitations**: No access to real-time pricing, availability, or booking systems
- **Geographic Scope**: Performance may vary based on regional coverage of source documents


## Incident Management
<!-- what happens when things go wrong. This part is particularly important to provide information on how incidents were dealth with and the processes put in place to minimize damage when things go wrong. -->
* **Common Issues**:
  * **Missing Azure Credentials**: `EnvironmentError` raised when `AZURE_API_BASE`, `AZURE_API_KEY`, or `AZURE_API_VERSION` not configured
  * **Document Processing Failures**: Invalid or corrupted PDFs skipped with logging; empty document collections handled gracefully
  * **Vector Store Issues**: Qdrant index automatically rebuilt from source documents; ensure `input_directory/` is accessible
  * **Network/API Failures**: Azure OpenAI service disruptions handled with fallback mechanisms and user notification
  * **Query Relevance Issues**: Off-topic queries routed to "Not Relevant" response; user prompted for tourism-related questions
* **Support Contact**:
  * **Primary Contact**: Gruppo 2 development team - TODO: Establish formal support channels
  * **Framework Support**: CrewAI documentation, GitHub repository, and Discord community
  * **Azure Support**: Microsoft Azure support for API-related issues
  * **Emergency Procedures**: TODO - Define escalation procedures for critical system failures


### Troubleshooting AI Application Deployment

This section outlines potential issues that can arise during the deployment of an AI application, along with their causes, resolutions, and best practices for mitigation.


#### Infrastructure-Level Issues

##### Insufficient Resources

* **Problem**: Inaccurate resource estimation for production workloads.
  * Unexpected spikes in user traffic can lead to insufficient resources such as compute, memory or storage that can lead to crashes and bad performance

* **Mitigation Strategy**:
  - **Resource Monitoring**: Implement monitoring of CPU, memory, and storage usage during system operation
  - **Scalability Planning**: Design system architecture to handle multiple concurrent users in production
  - **Performance Optimization**: Optimize vector search parameters and caching strategies to reduce resource consumption
  - **Load Testing**: Conduct stress testing to identify resource bottlenecks before production deployment


##### Network Failures

* **Problem**:  network bottlenecks  can lead to inaccessible or experiences latency of the application.

* **Mitigation Strategy**:
  - **Network Connectivity Testing**: Regular testing of Azure OpenAI API connectivity and whitelisted website accessibility
  - **Fallback Mechanisms**: Local embedding models and cached responses when network unavailable
  - **Timeout Management**: Implement appropriate timeouts and retry logic for network operations
  - **Service Monitoring**: Monitor Azure service status and implement alerts for service disruptions


##### Deployment Pipeline Failures

* **Problem**: pipeline fails to build, test, or deploy because of issues of compatibility between application code and infrastructure, environment variables or credentials misconfiguration.

* **Mitigation Strategy**: 
  - **Version Control**: Maintain stable versions and implement rollback procedures to last known good configuration
  - **Environment Consistency**: Use containerization (Docker) and virtual environments for consistent deployment
  - **Configuration Validation**: Implement automated validation of environment variables and dependencies
  - **Comprehensive Logging**: Enable detailed logging for troubleshooting deployment and configuration issues


#### Integration Problems

##### API Failures

* **Problem**: External APIs or internal services are unreachable due to network errors or authentication failures.

* **Mitigation Strategy**:
  - **Retry Logic**: Implement exponential backoff for transient API failures
  - **Credential Management**: Validate Azure API keys and implement secure credential rotation procedures
  - **API Monitoring**: Log and monitor all API responses for debugging and performance analysis
  - **Fallback Services**: Use local models when Azure OpenAI services are unavailable

##### Data Format Mismatches

* **Problem**: Crashes or errors due to unexpected data formats such as changes in the schema of external data sources or missing data validation steps.

* **Mitigation Strategy**: 
  - **Input Validation**: Implement comprehensive validation for all data inputs (PDFs, user queries, API responses)
  - **Schema Validation**: Use structured validation for API responses and document metadata
  - **Format Standardization**: Standardize internal data formats and implement conversion utilities
  - **Error Recovery**: Graceful handling of format mismatches with appropriate user feedback

#### Data Quality Problems

* **Problem**: Inaccurate or corrupt data leads to poor predictions.
* **Causes**:
  * No data validation or cleaning processes.
  * Inconsistent labelling in training datasets.

* **Mitigation Strategy**: 
  - **Data Quality Monitoring**: Implement automated checks for document completeness, readability, and content quality
  - **Regular Audits**: Establish schedule for reviewing and updating travel document collections
  - **Source Validation**: Verify reliability and currency of travel information sources
  - **Quality Metrics**: Track and monitor data quality indicators through RAGAS metrics and user feedback


#### Model-Level Issues

##### Performance or Deployment Issues

* **Problem**: Incorrect or inconsistent results due to data drift or inadequate training data for the real world deployment domain. 

* **Mitigation Strategy**:
  - **Performance Monitoring**: Continuous monitoring of model accuracy and response quality through RAGAS metrics
  - **Content Drift Detection**: Regular assessment of travel information currency and relevance
  - **Model Updates**: Stay current with Azure OpenAI model versions and capabilities
  - **Feedback Integration**: Incorporate user feedback and quality assessments into system improvements


#### Safety and Security Issues

##### Unauthorised Access

* **Problem**: Sensitive data or APIs are exposed due to misconfigured authentication and authorization.

##### Data Breaches

* **Problem**: User or model data is compromised due to insecure storage or lack of monitoring and logging of data access. 

* **Mitigation Strategy**: 
  - **Secure Storage**: Implement encryption for local document storage and vector databases
  - **Access Monitoring**: Log and monitor all data access patterns and system operations
  - **Anomaly Detection**: Implement alerts for unusual access patterns or system behavior
  - **Regular Security Audits**: Conduct periodic security assessments and vulnerability testing


#### Monitoring and Logging Failures

##### Missing or Incomplete Logs

* **Problem**: Lack of information to debug issues due to inefficient logging. Critical issues go unnoticed, or too many false positives occur by lack of implementation ofactionable information in alerts. 

* **Mitigation Strategy**: 
  - **Comprehensive Logging**: Implement detailed logging for all system operations, errors, and user interactions
  - **Log Management**: Centralized log collection and analysis with appropriate retention policies
  - **Alert Optimization**: Fine-tune alerting thresholds to minimize false positives while capturing critical issues
  - **Monitoring Tools**: Implement monitoring dashboard for system health, performance, and compliance metrics


#### Recovery and Rollback

##### Rollback Mechanisms

* **Problem**: New deployment introduces critical errors.

* **Mitigation Strategy**: 
  - **Version Control**: Maintain version history of all system configurations and document collections
  - **Backup Procedures**: Regular backups of vector databases, configuration files, and document collections
  - **Rollback Testing**: Regular testing of rollback procedures to ensure rapid recovery capability
  - **Deployment Strategy**: Implement staged deployment with validation checkpoints

##### Disaster Recovery

* **Problem**: Complete system outage or data loss.

* **Mitigation Strategy**:
  - **Disaster Recovery Plan**: Comprehensive documented procedures for system recovery from various failure scenarios
  - **Automated Backups**: Regular automated backups of all critical system components and data
  - **Recovery Testing**: Periodic testing of disaster recovery procedures to ensure effectiveness
  - **Business Continuity**: Maintain alternative access methods and fallback procedures for critical operations

### EU Declaration of conformity 

<div style="color: gray">
  EU AI Act <a href="https://artificialintelligenceact.eu/article/47/" style="color:blue; text-decoration:underline">Article 47</a>
  <p></p>
</div>

<!-- when applicable and certifications are available: it requires a systems name as well as the name and address of the provider; a statement that the EU declaration of conformity referred to in Article 47 is issued under the sole responsibility of the provider; a statement that the AI system is in conformity with this Regulation and, if applicable, with any other relevant Union law that provides for the issuing of the EU declaration of conformity referred to in Article 47, Where an AI system involves the processing of personal data;  a statement that that AI system complies with Regulations (EU) 2016/679 and (EU) 2018/1725 and Directive (EU) 2016/680, reference to the harmonised standards used or any other common specification in relation to which
conformity is declared; the name and identification number of the notified body, a description of the conformity
assessment procedure performed, and identification of the certificate issued; the place and date of issue of the declaration, the name and function of the person who signed it, as well as an
indication for, or on behalf of whom, that person signed, a signature.-->

### Standards applied

**EU AI Act Compliance Standards:**
- **Article 50 Transparency Obligations**: Implementation of clear AI system disclosure to users
- **Article 11 Technical Documentation**: Comprehensive system documentation as per Annex IV requirements
- **Article 14 Human Oversight**: Implementation of human-in-the-loop mechanisms and oversight procedures

**Technical Standards:**
- **ISO/IEC 23053:2022**: Framework for AI risk management principles and processes
- **ISO/IEC 23894:2023**: AI risk management guidelines (TODO - Complete formal assessment)
- **GDPR Compliance**: Data protection measures for user query processing and document handling
- **Azure Security Standards**: Microsoft Azure security and compliance frameworks for cloud services

**Development Standards:**
- **Python PEP Standards**: Code quality and style guidelines
- **CrewAI Framework**: Multi-agent AI system development best practices
- **LangChain Standards**: Retrieval-augmented generation implementation guidelines

## Documentation Metadata

### Template Version
**EU AI Act Application Documentation Template v1.0** - Based on EU AI Act requirements for limited-risk AI systems
TODO - Establish version control and template repository link

### Documentation Authors

* Development Team: Jacopo Bonanno, Roberto Parodo, Monica Salvati, Luca Sangiovanni, Pietro Montresori
* EU AI Act Compliance Specialist: TODO - Assign legal/compliance reviewer
* Technical Documentation Lead: TODO - Assign technical writing specialist
* Quality Assurance Team: TODO - Assign QA validation team

**Document Review Status:** 
- Initial draft completed: ✓
- Technical review: TODO
- Legal compliance review: TODO  
- Final approval: TODO

**Last Updated:** October 2, 2025
**Next Review Date:** TODO - Establish regular review schedule
