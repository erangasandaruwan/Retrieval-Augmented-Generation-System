# Retrieval-Augmented-Generation-System
This article and the application describe how to build a production-grade RAG (Retrieval-Augmented Generation) system, rather than a simple “chat with PDF” demo.  The architecture being described is roughly:  Documents → Parsing → Chunking → Embeddings → Vector DB → Hybrid Retrieval → Re-ranking → LLM → Citation/Validation → Evaluation → CI/CD
