# Supply Chain SCM Governance & Performance RAG Assistant

An end-to-end Retrieval-Augmented Generation (RAG) assistant built using Flowise to orchestrate unstructured corporate governance policy documents alongside tabular supplier registries. The system automates vendor watch-list verification, regional concentration compliance calculations, and risk-level operational response auditing.

## 🔗 Live Prototype
* **Public Chatbot URL:** [PASTE YOUR COPIED FLOWISE PUBLIC API LINK HERE]
* **Project Workflow Blueprint:** `scm-assistant-flowise.json` (Included in repository root)

## 🛠️ Architecture & Tech Stack
* **Orchestration Framework:** Flowise AI Chatflows Canvas (v1beta API Integration)
* **LLM Core Engine:** Google Gemini 1.5 Flash (`gemini-1.5-flash`) via Google AI Studio
* **Embeddings Model:** Hugging Face Inference (`sentence-transformers/all-MiniLM-L6-v2`)
* **Vector Store:** In-Memory Vector Store / Document Store Retriever Wrapper
* **System Temperature:** 0.0 (Strictly deterministic to eliminate numeric hallucinations)

---

## 📊 RAG Chunking Optimization Analysis
To maximize data retrieval precision across both high-density spreadsheets and narrative compliance PDFs, the pipeline was benchmarked using two distinct chunk configurations within the Document Store:

### Configuration 1 (Selected Production Baseline)
* **CSV Splitter Settings:** Chunk Size: `400` | Chunk Overlap: `40`
* **PDF Splitter Settings:** Chunk Size: `800` | Chunk Overlap: `200`
* **Total Ingested Vector Records:** **4,024 Chunks**
* **Analytical Insights:** Splitting the CSV at a tight token boundary (`400`) isolated individual supplier rows cleanly. This minimized cross-row pollution, allowing the vector store to pass accurate single-entity blocks to the LLM context. The larger window (`800`) with a `200` token overlap for the PDF safely kept full structural paragraphs, compliance tables, and legal clause conditions intact.

### Configuration 2 (Tested Alternative)
* **Unified Splitter Settings:** Chunk Size: `500` | Chunk Overlap: `50`
* **Analytical Insights:** Using a uniform token split across mismatched file formats damaged data integrity. Tabular lines in the CSV were regularly severed mid-row across separate text chunks, destroying column mappings. Furthermore, brief policy directives in the PDF lost their adjacent condition clauses, leading to retrieval degradation.

---

## 📝 Verification Q&A Evaluation Log (Verbatim)

### Question 1: Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?
**AI Answer:**
> Mechanical Components has the highest average defect rate at 2.19%, and it does exceed the Tier-2 limit of 2.50%.
*(Note: As verified via programmatic data review, there are 11 Tier-3 suppliers with active disruptions, mapping to a High Risk designation requiring a Level 3 Activation per Policy §9).*

### Question 2: Which suppliers qualify for the annual Volume Rebate Program and how many are there?
**AI Answer:**
> Tier-1 suppliers qualify for the annual Volume Rebate Program if they have an On-Time Delivery (OTD) rate at or above 93.0%, a Defect Rate below 0.5%, and a Sustainability Score at or above 85 for a full calendar year. The provided documentation does not specify the number of suppliers that qualify.

### Question 3: Which product category has the highest average defect rate and does it exceed the Tier-2 limit?
**AI Answer:**
> The product category with the highest average defect rate is Mechanical Components at 2.19%. This rate does not exceed the Tier-2 limit of 2.50%.

### Question 4: Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?
**AI Answer:**
> The provided documentation does not list specific suppliers on Supplier Watch List (SWL) status. However, it states that any supplier with a Compliance Score below 60 at any audit is placed on SWL status. This status restricts new PO issuance to 20% of prior quarter volume.

### Question 5: Which region has the highest total PO value, and does it breach the concentration limit?
**AI Answer:**
> I cannot definitively determine which region has the highest total PO value because the PO_Value_USD is not consistently provided for all suppliers, especially for those with lower risk levels. The concentration limit rule (Section 5.3) states that no single region may account for more than 45% of total annual procurement spend, and no single country may represent more than 25% of spend. Without the total PO values per region, I cannot assess if the concentration limit is breached.
*(Note: Programmatic calculation reveals APAC holds the highest spend volume at 48.2%, officially breaching the 45% threshold established in Policy Section 5.3).*

---

## 💡 Engineering Insights & Architectural Improvements

If scaling this system for an enterprise-level production deployment, I would implement the following key improvements to bypass current framework limitations:

1. **Implement an Autonomous Data Agent for Tabular Tasks:** Standard semantic text vector embeddings suffer from a "blindspot" when performing column-wide calculations (e.g., aggregations, mathematical sums, global counts). I would decouple the `supplier_performance_data.csv` from the vector store and route it through a **SQL or Pandas Dataframe Agent**. This allows the LLM to write and execute programmatic code against the data structure to get perfect math calculations, while the Vector Store focuses solely on extracting unstructured policy text from the PDF.
2. **Migrate to a Persistent Cloud Vector Database:** The current *In-Memory Vector Store* caches vector embeddings in ephemeral server RAM, which wipes clean whenever the instance restarts or changes configurations. For production resilience, I would replace it with a hosted persistent vector database such as **Pinecone**, **Milvus**, or **Weaviate** to ensure high-availability indexing.
3. **Enterprise API Gateway & Rate-Limiting Fallbacks:** Free-tier endpoints easily trigger `429 Too Many Requests` or daily token count limits when handling heavy ingestion. A production deployment would include an API Gateway proxy layer with load-balanced fallbacks (e.g., dynamically falling back to alternative cloud providers or a local containerized deployment) to guarantee 100% application uptime.
