# RAG System  
### The Key Technologies and Architecture

This article describes how to build a **RAG (Retrieval-Augmented Generation) system**, rather than a simple "chat with PDF/HTML" demo.

Here the overall architecture is:

**Documents → Parsing → Chunking → Embeddings → Vector DB → Hybrid
Retrieval → Re-ranking → LLM → Citation/Validation → Evaluation →
CI/CD**

## Key Technologies and Concepts


|Area                  | Technology / Concept | Purpose              |
|----------------------|----------------------|----------------------|
| AI Architecture      |  **RAG**             |   Ground LLM answers in your own documents |
| Input                |  PDF, Markdown, Web  |   Knowledge sources pages  |                 
| Processing           |  **Chunking**        |   Break documents into ~500--800 token sections |
| Chunking             |  **~100-token overlap** | Preserve context across chunk boundaries |
| AI                   |  **Embedding model** |   Convert text into vectors representing semantic meaning |
| Vector DB            |  **ChromaDB**        |   Store and search embeddings |
| Vector DB            |  **Weaviate**        |   Alternative production-oriented vector database |
| Retrieval            |  **Vector/Semantic Search**   |   Find documents based on meaning |                                
| Retrieval            |  **BM25**            |   Traditional keyword-based retrieval |
| Retrieval            |  **Hybrid Search**   |   Combine BM25 + vector search |
| Ranking              |  **Cross-encoder re-ranker**  |   Re-score retrieved chunks for better relevance |
| Re-ranking           |  **Cohere Rerank**   |   Managed re-ranking model/API |
| Re-ranking           |  **Sentence Transformers**    |   Open-source cross-encoder/reranking models |
| Orchestration        |  **LangChain**       |   Build RAG pipelines |
| Orchestration        |  **LangGraph**       |   Build more complex/stateful AI workflows |
| Reliability          |  **Citation enforcement**    |   Require answers to be supported by retrieved evidence |
| Reliability          |  **Abstention / refusal**    |   Don't answer when evidence is insufficient |
| Prompt Engineering   |  **Versioned prompt configuration**  |   Treat prompts as controlled application artifacts |
| Evaluation           |  **Golden dataset**  |   50--200 manually verified Q&A examples |
| Evaluation           |  **RAGAS**           |   Evaluate RAG quality |
| Evaluation metric    |  **Faithfulness**    |   Check whether generated claims are supported by retrieved context |
| DevOps               |  **CI pipeline**     |   Automatically run AI evaluations |
| Quality Gate         |  **Evaluation threshold**   |   Fail PR/build when RAG quality regresses |


There are **three maturity levels** described in the video.

## Phase 1 --- Basic RAG

The first implementation is:

``` text
PDF / Markdown / Web
        │
        ▼
 Document Loader
        │
        ▼
 Chunking
 500–800 tokens
 ~100 overlap
        │
        ▼
 Embedding Model
        │
        ▼
    ChromaDB
        │
        ▼
 Semantic Search
     Top-K
        │
        ▼
       LLM
        │
        ▼
 Answer + Citations
```

For example, the user asks:

> "What are the conditions for cancelling the contract?"

The query is embedded, Chroma finds perhaps the **top 5 relevant
chunks**, those chunks are supplied to the LLM, and the LLM produces an
answer based on them.

The important part is that you don't just return:

> The contract can be terminated with 30 days' notice.

You return something more like:

> The contract can be terminated by providing 30 days' written notice.\
> Source: Contract.pdf, Section 8.2, page 14.

That makes the RAG output traceable.

## Phase 2 --- Production-Quality Retrieval

This is one of the most technically important parts of the approach.

Instead of relying purely on vector search:

``` text
                    User Question
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
             BM25 Search       Vector Search
             keywords          semantic
                 │                 │
                 └────────┬────────┘
                          ▼
                  Hybrid Retrieval
                          │
                          ▼
                    Top Candidates
                          │
                          ▼
                  Cross Encoder
                     Re-ranker
                          │
                          ▼
                  Best Documents
                          │
                          ▼
                         LLM
                          │
                          ▼
                Evidence Validation
                          │
                 ┌────────┴────────┐
                 ▼                 ▼
             Supported        Unsupported
                 │                 │
                 ▼                 ▼
          Answer + citation   Decline answer
```

### Why BM25 + Vector Search?

Suppose your document contains:

``` text
VX-VEH-0012398
```

and the user searches for exactly that ID.

Semantic vector search may not be particularly good at matching
identifiers. **BM25** can be excellent at it.

But if the user asks:

> "Why did this vehicle fail to become available?"

and the document says:

> "Stock configuration validation failed."

Semantic/vector search can understand that those concepts may be
related.

Therefore:

-   **BM25 → exact terminology**
-   **Vector search → semantic meaning**
-   **Hybrid retrieval → both**

### Cross-Encoder Re-ranking

The **cross-encoder reranker** improves the final ranking.

Instead of trusting the initial similarity scores, it evaluates:

``` text
Query + Document 1 → relevance score
Query + Document 2 → relevance score
Query + Document 3 → relevance score
...
```

and reorders the candidate chunks according to relevance.

## Phase 3 --- Production AI Engineering

This is the part that differentiates the project from many portfolio RAG
applications.

Create a **Golden Evaluation Dataset**.

For example:

  Question                           Expected Answer   Expected Source
  ---------------------------------- ----------------- -----------------
  What is the cancellation period?   30 days           Contract §8.2
  Who approves refunds?              Finance Manager   Policy §12
  What is the maximum claim?         \$10,000          Policy §14

You might maintain **50--200 verified questions**.

Then every change to the system runs automated RAG evaluation.

``` text
Developer
    │
    ▼
Pull Request
    │
    ▼
CI Pipeline
    │
    ▼
Build RAG
    │
    ▼
Run Golden Dataset
    │
    ▼
RAGAS Evaluation
    │
    ├── Faithfulness
    ├── Answer Relevance
    ├── Context Precision
    └── Context Recall
    │
    ▼
Quality Threshold
    │
 ┌──┴───┐
 ▼      ▼
PASS    FAIL
 │       │
Merge   Block PR
```

For example, you could define:

``` text
Faithfulness >= 0.90
Context Precision >= 0.85
Answer Relevance >= 0.85
```

If somebody changes the chunking strategy, embedding model, system
prompt, retrieval parameters, or reranker and the score drops:

``` text
Faithfulness

Previous: 0.92
New:      0.81

Threshold: 0.90

❌ BUILD FAILED
```

This represents a much more mature **AI engineering lifecycle** than
simply demonstrating that an LLM can answer questions.

## Suggested Technology Stack

A practical implementation of the project described in the video could
look like:

``` text
Frontend
   │
   ▼
React / Angular
   │
   ▼
ASP.NET Core / FastAPI
   │
   ▼
LangGraph
   │
   ├── Query Processing
   ├── BM25 Retrieval
   ├── Vector Retrieval
   ├── Hybrid Ranking
   ├── Cross Encoder
   └── Citation Validation
   │
   ▼
LLM
Azure OpenAI / OpenAI
   │
   ▼
Answer + Sources
```

### Data Layer

``` text
Documents
   │
   ▼
Parser
   │
   ▼
Chunker
   │
   ▼
Embedding Model
   │
   ▼
ChromaDB / Weaviate
```

### Engineering / DevOps Layer

``` text
Git
 │
 ▼
Pull Request
 │
 ▼
Azure DevOps / GitHub Actions
 │
 ▼
RAGAS Evaluation
 │
 ▼
Quality Gate
 │
 ▼
Docker
 │
 ▼
AKS / Kubernetes
```

## Azure/.NET-Oriented Implementation

For an engineer with strong **.NET, Azure, AKS, Azure DevOps and AI
development** experience, the generic architecture can be adapted to:

``` text
Angular / React
       │
       ▼
ASP.NET Core Web API
       │
       ▼
LangGraph Python Service
       │
       ▼
Azure OpenAI
       │
       ▼
Azure AI Search
       │
       ▼
Cohere / SentenceTransformer Re-ranker
       │
       ▼
RAGAS Evaluation
       │
       ▼
Docker
       │
       ▼
AKS
       │
       ▼
Azure DevOps CI/CD
       │
       ▼
OpenTelemetry / Datadog
```

This makes the project demonstrate more than RAG or prompt engineering.
It demonstrates:

-   **AI engineering**
-   **Retrieval architecture**
-   **LLM orchestration**
-   **Software architecture**
-   **Cloud engineering**
-   **DevOps and CI/CD**
-   **Automated AI evaluation**
-   **Observability**
-   **Production reliability**

## Core Takeaway

A production-grade RAG system is not simply:

**Documents → Vector DB → LLM**

A mature implementation adds:

**Hybrid Retrieval + Re-ranking + Evidence/Citation Enforcement + Golden
Dataset + Automated Evaluation + CI/CD Quality Gates + Observability**

Those engineering practices are what move a RAG application from a
**demo** toward a **production-ready AI system**.
