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


We can build this system under 3 stages.

## Phase 1 - Basic RAG

The first implementation is,

<img width="1199" height="1312" alt="image" src="https://github.com/user-attachments/assets/5be3d916-7d40-4fba-a020-2e31a39b7aff" />


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


## Phase 2 - Advance and Quality Retrieval

This is one of the most technically important parts of the approach.

Instead of relying purely on vector search:

<img width="1200" height="1310" alt="image" src="https://github.com/user-attachments/assets/57b4cb10-165f-405e-8c6b-11821f73e9f1" />

### What is BM25 search ?
BM25 (Best Matching 25) is a ranking algorithm used by search engines to score and order documents based on their relevance to a user's search query. It is the gold standard for traditional keyword (lexical) search and powers systems like Elasticsearch, Apache Lucene, and OpenSearch.

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

## Phase 3 - Production AI Engineering

This is the part that differentiates the project from many portfolio RAG
applications.

Create a **Golden Evaluation Dataset**.

What is a Golden Evaluation Dataset ?
A Golden Evaluation Dataset (often called a golden dataset, gold set, or evaluation set) is a collection of carefully verified questions and expected answers/evidence that you use to test whether your RAG system is still producing correct results.

Think of it as automated regression testing for your AI/RAG application.

For example:

  | Question                        |  Expected Answer |  Expected Source |
  ---------------------------------- ----------------- -----------------
  | What is the cancellation period? | 30 days          |  Contract §8.2 |
  | Who approves refunds?            | Finance Manager  |  Policy §12    |
  | What is the maximum claim?       | \$10,000         |  Policy §14    |

You might maintain **50--200 verified questions**.

Then every change to the system runs automated RAG evaluation.

<img width="1189" height="1323" alt="image" src="https://github.com/user-attachments/assets/78cdad03-3eb6-4f7b-9452-efadb486d7b1" />


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

<img width="1199" height="1312" alt="image" src="https://github.com/user-attachments/assets/f7b58ee3-56b3-4739-a5d6-f7dfda69f5c7" />


### Data Layer

<img width="1160" height="1355" alt="image" src="https://github.com/user-attachments/assets/f1bc3dd5-ba2b-4aad-9653-4fecdc9da641" />


### Engineering / DevOps Layer

<img width="1226" height="1283" alt="image" src="https://github.com/user-attachments/assets/81bf599c-7800-4c12-bd53-1807e23b164a" />


## Azure/.NET-Oriented Implementation

For an engineer with strong **.NET, Azure, AKS, Azure DevOps and AI
development** experience, the generic architecture can be adapted to:

<img width="1145" height="1374" alt="image" src="https://github.com/user-attachments/assets/da30f4ea-bab4-4112-80c8-e956e48eec91" />


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
