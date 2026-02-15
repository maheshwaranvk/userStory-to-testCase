# RAG MongoDB Demo v7 - Complete Project Documentation

## Project Overview

**RAG-Mongo-Demo** is a **Retrieval-Augmented Generation (RAG)** system built with MongoDB Atlas Vector Search that enables intelligent search, retrieval, and analysis of test cases and user stories. It demonstrates modern AI/ML techniques including vector embeddings, keyword search, hybrid search, reranking, and query preprocessing.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        FRONTEND (React)                         │
│  - Query Interface                                              │
│  - Data Management (Upload, Convert)                            │
│  - Multiple Search Modes (BM25, Vector, Hybrid, Reranking)     │
│  - Query Preprocessing & Results Analysis                       │
└──────────────────────┬──────────────────────────────────────────┘
                       │
                       │ HTTP/REST API
                       ↓
┌─────────────────────────────────────────────────────────────────┐
│                   BACKEND (Express.js)                          │
│  - File Upload & Processing (Excel to JSON)                     │
│  - Job Tracking System                                          │
│  - Search Orchestration (BM25, Vector, Hybrid)                  │
│  - Reranking & Summarization                                    │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
   ┌─────────┐  ┌───────────────┐  ┌──────────┐
   │ Mistral │  │   MongoDB     │  │   Groq   │
   │   AI    │  │   Atlas       │  │   LLM    │
   │(Embed)  │  │(Search Index) │  │(Rerank)  │
   └─────────┘  └───────────────┘  └──────────┘
```

---

## 📁 Directory Structure & File Breakdown

### **Root Level Files**
```
package.json
├─ name: "rag-mongo-demo"
├─ scripts:
│  ├ dev: Run both client & server concurrently
│  ├ server: Start Express backend on port 3001
│  ├ client: Start React frontend on port 3000
│  └ build: Build React production bundle
└─ dependencies:
   ├ Express, CORS, Multer (Backend)
   ├ MongoDB driver
   ├ Mistral AI SDK (embeddings)
   ├ Groq SDK (reranking/summarization)
   ├ OpenAI SDK (alternative)
   ├ React, Material-UI (Frontend)
   └ Concurrently (dev runner)
```

---

## 🖥️ BACKEND: `server/index.js` (2134 lines)

### Purpose
Central Express server handling all API endpoints, file uploads, database operations, and coordination between external AI services.

### Key Components

#### 1. **Initialization & Setup**
```javascript
- PORT: 3001 (configurable via .env)
- CORS enabled for frontend communication
- Multer storage configured for file uploads
- DNS fix for macOS (Google DNS: 8.8.8.8)
```

#### 2. **Job Tracking System**
```javascript
createJob(files)        // Create unique job ID
updateJob(jobId, updates) // Update progress
getJob(jobId)          // Retrieve job status
```
- Tracks embedding generation progress
- Auto-cleanup of jobs older than 1 hour
- Enables resuming interrupted jobs

#### 3. **Database Validation**
```javascript
validateDbCollectionIndex()
- Checks if database exists
- Verifies collection exists
- Validates Atlas Search indexes
- Confirms documents exist (optional)
```

#### 4. **Main API Endpoints**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/health` | GET | Server health check |
| `/api/jobs/active` | GET | List in-progress jobs |
| `/api/jobs/:jobId` | GET | Get specific job status |
| `/api/metadata/distinct` | GET | Fetch distinct values for filters (modules, priorities, risks, types) |
| `/api/upload-excel` | POST | Convert Excel to JSON & store |
| `/api/embeddings/generate` | POST | Create embeddings for test cases |
| `/api/search` | POST | Vector similarity search |
| `/api/search/bm25` | POST | Keyword search (BM25) |
| `/api/search/hybrid` | POST | Combined BM25 + Vector search |
| `/api/search/rerank` | POST | Rerank search results using Groq LLM |

---

## 🎨 FRONTEND: `client/src/App.js` (491 lines)

### Purpose
Main React component providing navigation and theming for the entire application.

### Key Features

#### 1. **Theme System**
```javascript
- Enterprise color palette
- Light/Dark mode toggle
- Material-UI (MUI) theme provider
- Responsive design
```

#### 2. **Navigation Structure**
```
Sidebar Navigation:
├─ Dashboard
├─ Data Management
│  ├─ Convert Excel to JSON
│  └─ Store Embeddings
├─ Search
│  ├─ Query Search (Vector)
│  ├─ BM25 Search (Keyword)
│  ├─ Hybrid Search (Combined)
│  └─ Reranking Search
├─ Query Processing
│  ├─ Query Preprocessing
│  └─ Summarization & Dedup
└─ Settings
   └─ Configuration Management
```

#### 3. **State Management**
```javascript
- Drawer toggle (mobile responsiveness)
- Theme mode (light/dark)
- Breadcrumb navigation
- Component rendering based on selected menu item
```

---

## 📊 DATA MANAGEMENT COMPONENTS

### **1. `client/src/components/data/ConvertToJson.js` (356 lines)**

**Purpose**: Convert Excel files to JSON format for database ingestion

**Workflow**:
1. User uploads .xlsx/.xls file
2. User specifies sheet name (default: "testcases")
3. File sent to backend `/api/upload-excel`
4. Backend parses Excel using `xlsx` library
5. Convert rows to JSON objects with fields:
   ```javascript
   {
     id, title, module, description, steps, 
     expectedResults, preRequisites, priority, 
     risk, automationManual, ...metadata
   }
   ```

**State Management**:
```javascript
- selectedFile: Current file being processed
- sheetName: Target Excel sheet
- uploading: Loading state
- result: Conversion result with row count
- activeStep: Progress through conversion wizard
```

**Key Functions**:
- `handleFileSelect()`: Validate and select file
- `handleUpload()`: Send to server for conversion
- `displayResults()`: Show converted records

---

### **2. `client/src/components/data/EmbeddingsStore.js` (674 lines)**

**Purpose**: Manage embedding generation for test cases and storage in MongoDB

**Workflow**:
1. Display list of converted JSON files
2. Select files for embedding generation
3. Configure batch processing settings
4. Submit job to backend
5. Monitor progress in real-time
6. Embeddings stored in MongoDB with metadata

**State Management**:
```javascript
- files: List of available JSON files
- selectedFiles: User-selected files for processing
- currentJobId: Active job ID for tracking
- jobProgress: Real-time progress updates
- processing: Job running state
```

**Key Functions**:
- `loadFiles()`: Fetch available files from server
- `handleStartEmbedding()`: Initiate embedding job
- `pollJobProgress()`: Check job status every 500ms
- `displayJobMetrics()`: Show cost, tokens, progress

**Backend Job Flow**:
```javascript
1. Create job ID (job-${timestamp}-${random})
2. Read JSON files from server
3. Call generateBatchEmbeddings()
   - Batch 50 test cases at a time
   - Generate embeddings via Mistral AI
   - Track cost & token usage
4. Insert embeddings + metadata to MongoDB
5. Update job progress
6. Return results with statistics
```

---

## 🔍 SEARCH COMPONENTS

### **3. `client/src/components/search/QuerySearch.js` (809 lines)**

**Purpose**: Vector similarity search using semantic embeddings

**Features**:
```javascript
- Semantic understanding of queries
- Metadata-based filtering (module, priority, risk, automation)
- Results displayed in DataGrid
- Score breakdown and highlighting
- Cost tracking
```

**Search Flow**:
1. User enters query (e.g., "Share Diagnostic Reports with Patients via WhatsApp")
2. Optional: Apply metadata filters
3. Backend processes:
   ```
   a) Generate embedding for query via Mistral AI
   b) Search MongoDB vector index for similar embeddings
   c) Return top N most similar test cases
   d) Apply metadata filters if specified
   ```
4. Display results sorted by similarity score

**Query Preprocessing** (via `/api/search` endpoint):
```javascript
- Normalize query text
- Expand abbreviations (API → Application Programming Interface)
- Expand synonyms (bug → defect, error)
- Preserve test case IDs if mentioned
```

---

### **4. `client/src/components/search/BM25Search.js` (485 lines)**

**Purpose**: Keyword-based search using MongoDB text indexes

**Features**:
```javascript
- Fast exact/fuzzy keyword matching
- Field-weighted search (title > description > steps)
- Metadata filtering
- Performance metrics (response time)
```

**Field Weights**:
```javascript
id: 10.0              // Exact ID match highest priority
title: 5.0            // Title very important
module: 3.0           // Categorization important
description: 2.0      // Moderate importance
expectedResults: 1.5  // Lower importance
steps: 1.0            // Base importance
preRequisites: 0.8    // Lowest importance
```

**Search Pipeline**:
1. Build compound search query with field boosting
2. Apply fuzzy matching (maxEdits=1, prefixLength=2)
3. Execute aggregation pipeline
4. Apply metadata filters
5. Return ranked results

---

### **5. `client/src/components/search/HybridSearch.js` (613 lines)**

**Purpose**: Combine BM25 + Vector search with configurable weights

**Key Innovation**:
```javascript
- Slider controls for weight distribution
- Preset buttons: Balanced, Keyword-Heavy, Semantic-Heavy
- Combines strengths of both search methods
- User controls the fusion strategy
```

**Score Fusion**:
```javascript
finalScore = (bm25Weight × bm25_score) + (vectorWeight × vector_score)
// Example: 50% + 50% = balanced
// Example: 70% BM25 + 30% Vector = keyword-focused
```

**Use Cases**:
- Product names + semantic understanding
- Exact terms + conceptual relevance
- Adapts to user preference

---

### **6. `client/src/components/search/RerankingSearch.js`**

**Purpose**: Use LLM (Groq) to intelligently rerank search results

**Workflow**:
1. Get initial results from vector search
2. Send query + results to Groq API
3. LLM scores each result for relevance (0-100)
4. Return top K results by LLM score
5. Display with LLM reasoning

**Benefits**:
- More sophisticated relevance judgment
- Understands context better than algorithms
- Catches edge cases traditional ranking misses

---

## 🧠 QUERY PROCESSING COMPONENTS

### **7. `client/src/components/processing/QueryPreprocessing.js`**

**Purpose**: Demonstrate query normalization, abbreviation expansion, synonym expansion

**Pipeline Stages**:
```
1. NORMALIZATION
   Input:  "API Testing & BUG TRACKING!!"
   Output: "api testing bug tracking"

2. ABBREVIATION EXPANSION
   API → Application Programming Interface
   BM25 → Okapi Best Matching 25
   UI → User Interface

3. SYNONYM EXPANSION
   bug → defect, error, issue
   testing → qa, validation, verification
   Creates multiple query variations

4. RESULT
   Multiple expanded query options for searching
```

---

### **8. `client/src/components/processing/SummarizationDedup.js`**

**Purpose**: Summarize results and remove duplicates

**Features**:
- Deduplication: Remove near-duplicate results
- Summarization: Use Groq LLM to summarize long descriptions
- Cost optimization: Batch processing for efficiency
- Result consolidation: Combine similar results

---

### **9. `client/src/components/processing/PromptSchemaManager.js`**

**Purpose**: Define and manage LLM prompts for reranking/summarization

---

## 🔧 BACKEND UTILITIES & SCRIPTS

### **10. `src/scripts/utilities/mistralEmbedding.js` (270 lines)**

**Purpose**: Generate embeddings via Mistral AI API

**Functions**:

#### `generateEmbedding(text)`
```javascript
- Generates embedding for single text
- Retry logic (3 attempts with exponential backoff)
- Returns: { embedding, model, usage, metadata }
- Cost: ~$0.10 per 1M tokens
```

#### `generateBatchEmbeddings(textArray)`
```javascript
- Processes multiple texts efficiently
- Max 100 texts per batch
- Returns: { embeddings, usage, model }
- Handles timeout (30s) and errors
```

**Configuration**:
```javascript
MISTRAL_API_KEY: From .env
MISTRAL_EMBEDDING_MODEL: "mistral-embed"
MAX_RETRIES: 3
RETRY_DELAY_MS: 1000 (with exponential backoff)
```

---

### **11. `src/scripts/utilities/groqClient.js` (406 lines)**

**Purpose**: Reranking and summarization via Groq LLM

**Functions**:

#### `rerankDocuments(query, documents, topK)`
```javascript
- Takes initial search results
- Prompts Groq LLM to rank by relevance
- Returns top K reranked results
- Model: llama-3.2-3b-preview (fast, cost-effective)
```

#### `summarizeResults(results)`
```javascript
- Condenses long test case descriptions
- Extracts key information
- Generates concise summaries
- Model: llama-3.3-70b-versatile (more powerful)
```

**Groq Model Choices**:
```javascript
RERANK_MODEL: "openai/gpt-oss-120b"         // Fast reranking
SUMMARIZATION_MODEL: "openai/gpt-oss-120b"  // More capable
```

---

### **12. `src/scripts/search/bm25-search.js` (352 lines)**

**Purpose**: Standalone keyword search utility (backend)

**Features**:
- Compound search with field boosting
- Fuzzy matching support
- Score breakdown
- Aggregation pipeline construction
- Metadata filtering

**Pipeline**:
```javascript
[$search] → [$addFields for scores] → [$match filters] 
→ [$limit] → [$project for results]
```

---

### **13. `src/scripts/embeddings/create-embeddings-batch-mistral.js` (322 lines)**

**Purpose**: Batch process test cases and generate embeddings

**Configuration**:
```javascript
BATCH_SIZE: 50              // Test cases per batch
CONCURRENT_LIMIT: 3         // Parallel API calls
DELAY_BETWEEN_BATCHES: 1000ms
MONGODB_BATCH_SIZE: 100     // Docs inserted per call
```

**Process**:
```
1. Read test cases from JSON files
2. Split into batches of 50
3. For each batch:
   a) Format test case text (combine fields)
   b) Call Mistral AI embedding API
   c) Map embeddings back to test cases
   d) Track cost & tokens
4. Batch insert to MongoDB with embeddings
5. Report statistics
```

**Cost Tracking**:
```javascript
- Mistral pricing: ~$0.10 per 1M tokens
- Calculates per batch and total cost
- Provides token usage metrics
```

---

### **14. `src/scripts/query-preprocessing/queryPreprocessor.js` (443 lines)**

**Purpose**: Main query preprocessing orchestrator

**Pipeline**:
```javascript
1. extractTestCaseIds()      // Extract IDs if present
2. normalizeComplete()        // Lowercase, remove punctuation
3. expandAbbreviations()      // API → Application Programming Interface
4. expandSynonyms()           // bug → defect, error, issue
5. Re-attach test case IDs
```

**Options**:
```javascript
enableAbbreviations: true
enableSynonyms: true
maxSynonymVariations: 5
customAbbreviations: {}
customSynonyms: {}
smartExpansion: false
preserveTestCaseIds: true
```

**Returns**:
```javascript
{
  original: "original query",
  normalized: "normalized query",
  abbreviationExpanded: "expanded abbreviations",
  synonymExpanded: ["query1", "query2", "query3"],
  metadata: {
    processingTime: 45,
    expandedCount: 3,
    ...
  }
}
```

---

### **15. `src/scripts/query-preprocessing/dictionaries.js`**

**Purpose**: Define abbreviation and synonym mappings

**Abbreviation Dictionary**:
```javascript
{
  api: "application programming interface",
  ui: "user interface",
  bm25: "okapi best matching 25",
  qa: "quality assurance",
  ...
}
```

**Synonym Dictionary**:
```javascript
{
  bug: ["defect", "error", "issue", "problem"],
  testing: ["qa", "validation", "verification"],
  dashboard: ["homepage", "main page", "landing"],
  ...
}
```

---

### **16. `src/scripts/query-preprocessing/normalizer.js`**

**Purpose**: Clean and standardize query text

**Operations**:
```javascript
- Convert to lowercase
- Remove special characters & punctuation
- Normalize whitespace
- Handle contractions
- Extract test case IDs (TC-001, etc.)
```

---

### **17. `src/scripts/query-preprocessing/abbreviationMapper.js`**

**Purpose**: Expand abbreviations intelligently

**Smart Expansion**:
- Context-aware expansion
- Preserve acronyms if appropriate
- Track mapping history

---

### **18. `src/scripts/query-preprocessing/synonymExpander.js`**

**Purpose**: Generate synonym variations of queries

**Features**:
- Multiple synonym suggestions per word
- Limit variations to avoid explosion
- Include original query
- Minimum synonym length filter

---

## 📦 DATA STORAGE

### **MongoDB Collections**

#### **test_cases Collection**
```javascript
{
  _id: ObjectId,
  id: "TC-001",
  title: "Share Diagnostic Reports",
  module: "Reports",
  description: "Share diagnostic reports with patients via WhatsApp",
  steps: "1. Login\n2. Select patient...",
  expectedResults: "Reports sent successfully",
  preRequisites: "Valid patient record",
  priority: "High",
  risk: "Medium",
  automationManual: "Manual",
  
  // Embedding fields
  embedding: [0.123, -0.456, 0.789, ...], // 1024-dim vector
  embeddingMetadata: {
    model: "mistral-embed",
    tokens: 150,
    cost: 0.000015,
    apiSource: "mistral-batch",
    batchNumber: 5,
    createdAt: ISODate("2026-01-07T09:00:00Z")
  },
  
  createdAt: ISODate("2026-01-07T08:30:00Z")
}
```

### **Search Indexes**

#### **vector_index_test_cases** (Vector Search)
```javascript
{
  "type": "vector",
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "similarity": "cosine"
    }
  ]
}
```

#### **bm25_search** (Text Search)
```javascript
{
  "type": "search",
  "fields": {
    "id": {},
    "title": { "boost": 5 },
    "module": { "boost": 3 },
    "description": { "boost": 2 },
    "expectedResults": { "boost": 1.5 },
    "steps": { "boost": 1 },
    "preRequisites": { "boost": 0.8 }
  }
}
```

---

## 🔄 DATA FLOW EXAMPLES

### **Example 1: Search Flow**

```
User enters: "share diagnostic reports"
            ↓
      QueryPreprocessing
      - Normalize: "share diagnostic reports"
      - Expand synonyms: ["share diagnostic reports", 
                          "distribute diagnostic reports",
                          "send diagnostic reports"]
      ↓
  Multiple searches with expanded queries
      ↓
  For each query variation:
    - Generate embedding (Mistral)
    - Vector search in MongoDB
    - BM25 search in MongoDB
    ↓
  Combine results
      ↓
  Optional: Rerank with Groq LLM
      ↓
  Optional: Dedup & Summarize
      ↓
  Return results with scores
```

---

### **Example 2: Embedding Generation Flow**

```
User selects: ["testcases.json"]
            ↓
  Backend reads JSON
  Parse 500 test cases
            ↓
  Split into batches:
    Batch 1: items 1-50
    Batch 2: items 51-100
    Batch 3: items 101-150
    ... (10 batches total)
            ↓
  For each batch (max 3 concurrent):
    - Format text for each test case
    - Call Mistral API
    - Get 1024-dimensional embedding
    - Track cost & tokens
    - Map embedding back to test case
            ↓
  Batch insert to MongoDB
    (100 test cases per insert operation)
            ↓
  Update job progress
            ↓
  Return statistics:
    - Total cost: $0.045
    - Total tokens: 450,000
    - Processing time: 2m 30s
    - Success rate: 100%
```

---

## 🛠️ CONFIGURATION (.env)

```bash
# MongoDB Atlas
MONGODB_URI="mongodb+srv://user:pass@cluster.mongodb.net/?appName=app"
DB_NAME="db_stories_tests"

# Collections & Indexes
COLLECTION_NAME="test_cases"
VECTOR_INDEX_NAME="vector_index_test_cases"
BM25_INDEX_NAME="bm25_search"

USER_STORIES_COLLECTION_NAME="user_stories"
USER_STORIES_VECTOR_INDEX_NAME="vector_index_user_story"

# Mistral AI (Embeddings)
MISTRAL_API_KEY="your_api_key"
MISTRAL_EMBEDDING_MODEL="mistral-embed"

# Groq AI (Reranking & Summarization)
GROQ_API_KEY="your_api_key"
GROQ_RERANK_MODEL="openai/gpt-oss-120b"
GROQ_SUMMARIZATION_MODEL="openai/gpt-oss-120b"

# Server
PORT=3001
```

---

## 🚀 Running the Project

### Setup
```bash
cd rag-mongo-demo-v7
npm install                    # Install all dependencies
```

### Development
```bash
npm run dev                    # Start both frontend & backend
# Frontend: http://localhost:3000
# Backend: http://localhost:3001
```

### Production Build
```bash
npm run build                  # Build React for production
```

---

## 🔑 Key Technologies

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Frontend | React 18 | UI framework |
| UI Library | Material-UI (MUI) | Component library |
| Backend | Express.js | REST API server |
| Database | MongoDB Atlas | Vector & text search |
| Embeddings | Mistral AI | Vector generation |
| Reranking | Groq LLM | Intelligent ranking |
| File Upload | Multer | Excel upload handling |
| Concurrency | p-limit | Rate limiting API calls |

---

## 📊 Performance Characteristics

### Search Performance
- **Vector Search**: 50-200ms (on MongoDB Atlas)
- **BM25 Search**: 30-100ms (keyword matching)
- **Hybrid Search**: 100-300ms (combined)
- **Reranking**: 500-2000ms (depends on LLM)

### Cost Estimates (Monthly)
- **Embeddings**: ~$5-15 (50,000 docs × 150 tokens × $0.10/1M)
- **Vector Search**: Included in MongoDB
- **BM25 Search**: Included in MongoDB
- **Reranking**: ~$2-5 (100 reranks × 100 tokens average)
- **Summarization**: ~$1-3 (occasional use)

---

## 🎯 Use Cases

1. **Test Case Discovery**: Find related test cases quickly
2. **Requirement Traceability**: Link requirements to test cases
3. **Test Coverage Analysis**: Identify uncovered areas
4. **Knowledge Management**: Search organizational knowledge base
5. **AI-Powered Recommendations**: Suggest related tests

---

## 🔐 Security Notes

⚠️ **IMPORTANT**: 
- API keys in `.env` file should be kept secret
- Don't commit `.env` to version control
- Use environment variables in production
- Implement authentication for API endpoints
- Validate all user inputs on backend

---

## 📝 Summary

This project demonstrates a complete **RAG (Retrieval-Augmented Generation)** system that:

1. **Ingests** test case data via Excel files
2. **Generates** semantic embeddings using Mistral AI
3. **Stores** embeddings in MongoDB Atlas Vector Search
4. **Searches** using multiple strategies (vector, keyword, hybrid)
5. **Reranks** results intelligently using Groq LLM
6. **Preprocesses** queries for better matching
7. **Summarizes** and deduplicates results

The architecture is modular, scalable, and demonstrates best practices for combining traditional search (BM25) with modern semantic search (vector embeddings).

