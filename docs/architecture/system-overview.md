# SearchForge — System Overview

## 1. Problem

### What is SearchForge?

SearchForge is a hybrid search engine that combines traditional lexical search with semantic vector search to retrieve relevant content for a user's query.

Traditional lexical search primarily relies on matching terms between the query and indexed content. SearchForge combines lexical matching with semantic similarity so that relevant content can still be retrieved when the wording of the query differs from the wording of the indexed content.

### What problem does it solve?

SearchForge addresses poor search relevance caused by relying solely on exact or lexical keyword matching.

For example, for a query such as:

> "How to feed the dog?"

A traditional keyword-based system may primarily retrieve content containing terms such as "feed" and "dog".

SearchForge can additionally retrieve content related to concepts such as dog nutrition, pet feeding, or appropriate food for dogs, even when the exact query terms are not present.

The system therefore follows the general flow:

**User query → retrieve using multiple search strategies → combine evidence → rank results → return relevant results**

### What does the user provide?

The user provides a natural-language search query, for example:

* "How can I earn money?"
* "How do I become a backend developer?"
* "How does garbage collection work in Java?"

### What does SearchForge return?

SearchForge returns a ranked list of relevant search results based on the relationship between the user's query and the indexed content.

## 2. Goals
    {SearchForge Goals

1. Natural-language search
   ↓
   Users submit natural-language queries.

2. Lexical retrieval
   ↓
   BM25 / term-based retrieval.

3. Semantic retrieval
   ↓
   Retrieve conceptually related content despite wording differences.

4. Hybrid retrieval
   ↓
   Combine lexical + semantic signals and produce ranked results.

5. Evaluation
   ↓
   Establish measurable benchmarks and compare retrieval approaches.

6. Performance
   ↓
   Measure latency, throughput, indexing performance, and resource usage.
    Natural-language retrieval
    Users should be able to submit natural-language queries and retrieve relevant indexed content.
    Lexical retrieval
    The system should support traditional term-based retrieval, eventually using algorithms such as BM25.
    Semantic retrieval
    The system should retrieve semantically related content even when the query and document use different wording.
    Hybrid retrieval and ranking
    The system should combine lexical and semantic retrieval signals to produce a ranked result set.
    Search relevance evaluation
    Search quality should be measurable using retrieval metrics rather than judged only by manually looking at results.
    Performance measurement
    Search latency and system throughput should be measured so that performance improvements can be demonstrated quantitatively.
    Scalable indexing foundation
    The architecture should eventually support indexing a large and continuously growing collection of 
    content rather than relying on a small hardcoded dataset.
    Engineering depth
    The project should demonstrate the underlying search-engine concepts—indexing, retrieval, ranking, concurrency, persistence, and performance engineering—instead of being merely a CRUD API around a search library.
## 3. Non-Goals

SearchForge will intentionally avoid the following in its initial versions:

    1. **Internet-scale crawling**
      SearchForge will not attempt to crawl and index the entire public web. Initial development will use a controlled collection of websites and documents.
    2. **Personalized search**
    Initial ranking will not depend on individual users, browsing history, location, or personal profiles.
    3. **Multimedia search**
    Image, video, audio, and multimodal retrieval are outside the initial scope. The primary focus is text-based search.
    4. **Recommendation systems**
       SearchForge is designed to retrieve content based on an explicit user query rather than provide personalized recommendations.
    5. **Training custom foundation models**
       SearchForge will use existing embedding models and, if required later, an existing language model rather than training its own large language model.
    6. **Large-scale distributed microservices architecture**
       The initial system will use a modular-monolith architecture. Components will only be separated when there is a demonstrated engineering requirement.
    7. **Full-scale commercial search features**
       Advertising, subscriptions, payments, enterprise account management, and other commercial features are outside the project's scope.
    8. **Answer generation as the core retrieval mechanism**
     Natural-language explanations may be added as a later RAG capability, but generated answers will not replace the underlying retrieval and ranking system.


## 4. High-Level Architecture

SearchForge will initially be implemented as a modular monolith with two major processing flows: an indexing pipeline and a query/retrieval pipeline.

### 4.1 Indexing Pipeline

The initial system will operate on a controlled collection of documents and web content. External web crawling will be introduced later as an additional data source.

```text
Controlled Documents
        │
        ▼
Document Ingestion
        │
        ▼
Document Parsing
        │
        ▼
Text Processing / Normalization
        │
        ├──────────────────┐
        ▼                  ▼
Lexical Representation   Semantic Representation
        │                  │
        ▼                  ▼
Inverted Index           Vector Index
```

Documents will retain metadata such as document ID, title, source URL, and relationships to their processed content units.

Semantic retrieval will use smaller content units or chunks rather than relying exclusively on a single representation for an entire document. Chunking strategy will be evaluated and refined during implementation.

### 4.2 Query and Retrieval Pipeline

```text
User Query
    │
    ▼
Query Processing
    │
    ├──────────────────────┐
    ▼                      ▼
Lexical Retrieval      Semantic Retrieval
    │                      │
    ▼                      ▼
BM25 Scoring          Vector Similarity
    │                      │
    └──────────┬───────────┘
               ▼
        Hybrid Ranking
               │
               ▼
             Top-K
               │
               ▼
       Result Processing
               │
               ▼
        Search Interface
```

The lexical and semantic retrieval paths will operate as complementary retrieval strategies. Their results will be combined by a hybrid ranking component rather than treating either retrieval method as universally sufficient.

### 4.3 Future Extensions

The architecture will allow additional components to be introduced without redesigning the core retrieval system:

* Concurrent web crawler as an additional ingestion source
* Persistent indexing infrastructure
* Retrieval evaluation and benchmarking
* Performance monitoring and optimization
* Grounded natural-language explanation using retrieved content

## 5. Major Components

### 5.1 Document Ingestion

Responsible for accepting documents or web content into the indexing pipeline and assigning stable document identifiers and metadata.

### 5.2 Document Storage

Stores the original document content and associated metadata such as document ID, title, source URL, and content relationships.

### 5.3 Document Parser

Extracts usable text and metadata from supported document formats.

### 5.4 Text Processing / Lexical Analyzer

Processes textual content for lexical retrieval through operations such as tokenization, normalization, stop-word handling, and optional stemming or lemmatization.

### 5.5 Chunk Manager

Divides content into smaller semantic units when required for semantic indexing and retrieval while maintaining the relationship between chunks and their parent documents.

### 5.6 Embedding Model Inference

Converts documents, chunks, and search queries into vector representations for semantic retrieval.

### 5.7 Inverted Index

Stores term-to-document or term-to-content relationships and associated statistics required for lexical retrieval and BM25 scoring.

### 5.8 Vector Index

Stores vector representations and supports efficient similarity search for semantic retrieval.

### 5.9 Query Processor

Normalizes and prepares user queries for the retrieval pipelines.

### 5.10 Lexical Retrieval

Retrieves and scores candidate results using the inverted index and BM25.

### 5.11 Semantic Retrieval

Retrieves semantically similar candidate content using vector similarity search.

### 5.12 Hybrid Ranking

Combines lexical and semantic retrieval signals and produces the final ranked result set.

### 5.13 Result Processing

Transforms ranked internal results into a response containing information such as title, snippet, source URL, and relevance information.

### 5.14 Search API

Provides an interface through which clients can submit queries and receive search results.

### 5.15 Search Interface

Provides the user-facing interface for entering queries and viewing ranked results.

### 5.16 Evaluation and Observability

Provides mechanisms for measuring retrieval quality, latency, throughput, indexing performance, and resource utilization.

                

            SEARCHFORGE
│
├── 1. Document Ingestion
│
├── 2. Document Parser
│
├── 3. Text Processing / Normalization
│
├── 4. Lexical Index
│      └── Inverted Index
│
├── 5. Semantic Index
│      └── Vector Index
│
├── 6. Query Processing
│
├── 7. Lexical Retrieval
│      └── BM25
│
├── 8. Semantic Retrieval
│      └── Vector Similarity
│
├── 9. Hybrid Ranker
│
├── 10. Result Processing
│
├── 11. Search API
│
├── 12. Evaluation
│
└── 13. Performance / Observability



## 6. Data Flow

SearchForge consists of two primary data flows: document indexing and query processing.

### 6.1 Document Indexing Flow

```text
Document
    ↓
Document Parser
    ↓
Text Processing / Normalization
    │
    ├──────────────────────────┐
    ▼                          ▼
Lexical Representation       Chunking
    │                          │
    ▼                          ▼
Inverted Index            Embedding Model
                               │
                               ▼
                          Vector Index
```

The original document and its metadata are retained in document storage. The indexing pipeline produces the structures required for lexical and semantic retrieval.

### 6.2 Query Processing Flow

```text
User Query
    ↓
Query Processing
    │
    ├───────────────────────┐
    ▼                       ▼
Lexical Retrieval      Semantic Retrieval
    │                       │
    ▼                       ▼
BM25 Scoring          Vector Similarity
    │                       │
    └───────────┬───────────┘
                ▼
          Hybrid Ranking
                │
                ▼
              Top-K
                │
                ▼
        Result Processing
                │
                ▼
             API / UI
```

The lexical and semantic retrieval paths independently generate candidate results. The hybrid ranking stage combines their retrieval signals and produces the final ranked result set.

### 6.3 Result Flow

The final results will contain user-facing information such as document title, relevant content or snippet, source URL, and ranking information where appropriate.

The system may later use the retrieved content as grounded context for a natural-language explanation layer.


## 7. Technology Choices

### Backend and API

SearchForge will use Java with Spring Boot for the backend and API layer. Spring Boot provides a mature backend ecosystem and fits the project's requirements for REST APIs, dependency management, testing, and production-oriented application development.

Authentication and authorization mechanisms such as Spring Security and JWT will only be introduced if they become necessary for a defined product requirement.

### Document Storage

A relational database accessed through Spring Data JPA will initially be used for structured document metadata and persistent application data.

The search indexes will remain conceptually separate from relational application storage because inverted indexes and vector indexes have different access patterns and requirements.

### Inverted Index

The initial lexical retrieval engine will implement the core inverted-index data structure directly rather than hiding the implementation behind a search-engine framework.

This allows the project to demonstrate and measure fundamental information-retrieval concepts such as postings lists, term statistics, document frequency, and BM25 scoring.

### Embedding Model

Semantic retrieval will use a pretrained embedding model rather than training a custom foundation model.

The exact embedding model will be selected later based on factors such as retrieval quality, dimensionality, inference cost, language support, and deployment constraints.

### Vector Retrieval

The initial semantic retrieval implementation will begin with a simple similarity-search baseline. For a controlled corpus, brute-force vector comparison can provide a straightforward reference implementation.

If corpus size or query latency makes exhaustive comparison impractical, an approximate nearest-neighbor approach such as HNSW can be evaluated as a later optimization.

### Frontend

The user interface will initially use React with TypeScript and Tailwind CSS.

The frontend will remain intentionally lightweight and focused on demonstrating the search workflow: query submission, ranked results, snippets, and source URLs.

### General Technology Principle

Technology choices will be driven by measurable system requirements rather than by adding frameworks or infrastructure solely for resume value. More advanced technologies will be introduced when they solve a demonstrated engineering problem.


## 8. Storage and Indexing Strategy

SearchForge will separate application data storage from search-specific indexing structures.

### 8.1 Initial Controlled Corpus

The first version will use a local, controlled collection of documents and web-content snapshots. This keeps the corpus deterministic and makes indexing and retrieval experiments reproducible.

### 8.2 Document and Metadata Storage

Original documents and structured metadata will be maintained separately from the search indexes.

A relational database such as PostgreSQL will be used for persistent metadata as the system evolves. Metadata may include document identifiers, titles, source URLs, content hashes, timestamps, and relationships between documents and chunks.

### 8.3 Lexical Index

Lexical retrieval will use a dedicated inverted-index structure containing terms, postings, and statistics required for BM25 scoring.

The initial implementation will prioritize understanding and implementing the underlying data structure rather than immediately depending on an external search engine.

### 8.4 Semantic Index

Semantic representations will associate content chunks with embedding vectors.

The initial implementation will use a simple vector-search baseline suitable for the controlled corpus. As the corpus grows, specialized approximate-nearest-neighbor indexing or vector storage technologies can be evaluated based on measured performance requirements.

### 8.5 Separation of Responsibilities

Different storage structures will serve different access patterns:

```text
Document metadata → Relational storage
Original content  → Document storage
Lexical retrieval → Inverted index
Semantic retrieval → Vector index
```

The architecture will avoid assuming that one storage technology should handle every workload.

### 8.6 Evolution

The storage architecture will evolve as corpus size, query volume, persistence requirements, and performance constraints increase. More specialized storage technologies will be introduced only when their benefits can be demonstrated through measurable requirements or benchmarks.


## 9. Evolution Strategy

SearchForge will be developed incrementally rather than as a single large implementation. Each version will introduce a distinct engineering capability while keeping the previous functionality operational.

### Version 0 — Foundation

Establish the Spring Boot project, repository structure, build system, testing setup, configuration, and architecture documentation.

### Version 1 — Lexical Search

Implement the foundational text-search engine:

* Controlled document corpus
* Document parsing and normalization
* Tokenization
* Inverted index
* Term statistics
* BM25 scoring
* Query processing
* Top-K lexical retrieval
* Search API

This version establishes the first functional search engine baseline.

### Version 2 — Semantic Retrieval

Add semantic search capabilities:

* Content chunking
* Embedding generation
* Vector representations
* Vector similarity retrieval
* Semantic Top-K retrieval

Lexical retrieval will remain operational so that the two approaches can be compared.

### Version 3 — Hybrid Search

Combine lexical and semantic retrieval:

* Candidate generation from both retrieval systems
* Score normalization or combination
* Hybrid ranking
* Final Top-K results

This version becomes the core hybrid-search implementation.

### Version 4 — Evaluation

Introduce a reproducible retrieval evaluation framework:

* Query/relevance dataset
* Precision@K
* Recall@K
* MRR
* NDCG
* Comparison of lexical, semantic, and hybrid retrieval

Search quality improvements will be supported by measurable evidence.

### Version 5 — Persistent and Concurrent Indexing

Improve the indexing infrastructure:

* Persistent index structures
* Batch indexing
* Concurrent processing
* Worker-based ingestion
* Failure handling
* Indexing performance measurements

### Version 6 — Controlled Web Crawler

Introduce web acquisition as an additional ingestion source:

* Seed URLs
* URL queue
* Concurrent crawling
* HTML extraction
* Duplicate detection
* Crawl failure handling
* Controlled crawl limits

The crawler will feed the existing ingestion and indexing pipeline rather than becoming tightly coupled to the search engine.

### Version 7 — Performance Engineering

Measure and optimize the system under increasing workload:

* Search latency
* P50/P95/P99 latency
* Queries per second
* Indexing throughput
* Memory consumption
* Corpus-size scaling

Optimization decisions will be based on measured bottlenecks.

### Version 8 — Grounded Natural-Language Results

As a later extension, introduce a retrieval-grounded explanation layer that uses highly ranked retrieved content to generate natural-language explanations while preserving the underlying source URLs.

The retrieval system will remain the foundation of the product rather than being replaced by the language model.

### Evolution Principle

SearchForge will evolve from a small deterministic retrieval system into a more capable search platform through measurable incremental improvements. New technologies and architectural complexity will be introduced only when a demonstrated requirement justifies them.



## 10. Current Scope

The current development scope is limited to establishing the project foundation and preparing the architecture for the first lexical-search implementation.

### Currently Included

* Java 21 and Spring Boot project foundation
* Maven Wrapper
* Git repository and project structure
* Basic application configuration
* Automated test foundation
* Architecture documentation
* Controlled local document corpus design
* Planned document ingestion and parsing pipeline
* Planned lexical processing pipeline
* Planned inverted-index architecture

### Not Yet Implemented

The following capabilities are intentionally deferred:

* BM25 implementation
* Semantic embeddings
* Vector retrieval
* Hybrid ranking
* Web crawling
* Concurrent crawling
* Persistent search indexes
* Retrieval evaluation framework
* Performance optimization
* Natural-language answer generation
* Advanced frontend functionality

These capabilities will be introduced incrementally according to the evolution strategy.

### Immediate Next Milestone

The next implementation milestone is the construction of the foundational lexical retrieval engine.

The immediate objective is to process a controlled collection of documents, construct an inverted index, process a user query, retrieve candidate documents, score them using a lexical ranking method, and return the Top-K results.

The implementation will begin with the underlying search concepts and data structures before introducing more advanced retrieval technologies.







