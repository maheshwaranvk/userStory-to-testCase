# RAG Test Case Generation with MongoDB Atlas Vector Search

A full-stack application for generating, searching, and managing test cases using Retrieval-Augmented Generation (RAG) with MongoDB Atlas Vector Search, BM25 indexing, and hybrid search capabilities.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Running the Application](#running-the-application)
- [API Documentation](#api-documentation)
- [Scripts & Utilities](#scripts--utilities)
- [Search Mechanisms](#search-mechanisms)
- [Embeddings](#embeddings)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project implements a sophisticated test case generation and management system that combines:

- **RAG (Retrieval-Augmented Generation)**: Uses MongoDB Atlas Vector Search to retrieve relevant documents and context
- **Hybrid Search**: Combines BM25 keyword search with vector similarity search for optimal retrieval
- **Vector Embeddings**: Generates embeddings using Mistral AI's embedding models
- **LLM Integration**: Uses Groq, OpenAI, and Mistral APIs for test case generation and summarization
- **Web UI**: React-based frontend with Material-UI for easy interaction

## ✨ Features

### Core Features
- **Vector Search**: MongoDB Atlas Vector Search for semantic similarity
- **BM25 Search**: Keyword-based search with field-level weighting
- **Hybrid Search**: Combines vector and keyword search for better results
- **Re-ranking**: Reranks search results using Groq's inference API
- **Query Preprocessing**: Normalizes queries, expands synonyms, maps abbreviations
- **Batch Embeddings**: Efficiently generates embeddings for large datasets
- **File Upload**: Support for Excel files (XLSX) for bulk data import

### Search Features
- Full-text search with relevance scoring
- Semantic search with vector embeddings
- Advanced filtering and field-specific weighting
- Result deduplication and summarization

## 🛠️ Tech Stack

### Frontend
- **React 19**: Modern UI framework
- **Material-UI (MUI)**: Component library
- **Axios**: HTTP client
- **React Router**: Client-side routing

### Backend
- **Node.js / Express**: REST API server
- **MongoDB Atlas**: Cloud database with vector search
- **Multer**: File upload handling
- **dotenv**: Environment configuration

### AI/ML Services
- **Mistral AI**: Embedding generation
- **Groq**: LLM for re-ranking and summarization
- **OpenAI**: Optional LLM integration
- **Hugging Face**: Alternative embedding models

### Data Processing
- **XLSX**: Excel file parsing
- **Sequelize**: ORM for MySQL (optional)
- **p-limit**: Concurrency control for batch operations

## 📦 Prerequisites

- Node.js (v16+)
- npm or yarn
- MongoDB Atlas account with Vector Search enabled
- API keys for:
  - Mistral AI (for embeddings)
  - Groq (for re-ranking and summarization)
  - OpenAI (optional)

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/ddilesh/testcasegenai.git
cd testcasegenai
```

### 2. Install Root Dependencies

```bash
npm install
```

This will automatically install client dependencies as well (via postinstall script).

### 3. Install Individual Folder Dependencies (if needed)

```bash
# Client
cd client && npm install

# Server (if separate installation needed)
cd ../server && npm install

# Scripts
cd ../src && npm install
```

## ⚙️ Configuration

### 1. Environment Variables

Create a `.env` file in the root directory:

```env
# MongoDB Atlas
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/?retryWrites=true&w=majority
DB_NAME=testcases_db
COLLECTION_NAME=testcases
JIRA_COLLECTION_NAME=jira_stories

# AI/LLM Services
MISTRAL_API_KEY=your_mistral_api_key
GROQ_API_KEY=your_groq_api_key
OPENAI_API_KEY=your_openai_api_key

# Server
PORT=3001
NODE_ENV=development
```

### 2. Create MongoDB Collections

The application expects the following collections in MongoDB:

- **testcases**: Main collection for test cases
- **jira_stories**: Collection for user stories from Jira
- **embeddings_index**: Vector index for embeddings

Ensure your MongoDB Atlas cluster has Vector Search enabled.

### 3. Configure Vector Index

Create a vector search index in MongoDB Atlas:

```json
{
  "fields": [
    {
      "type": "vector",
      "path": "embedding",
      "similarity": "cosine",
      "dimensions": 1024
    },
    {
      "type": "filter",
      "path": "id"
    },
    {
      "type": "filter",
      "path": "title"
    }
  ]
}
```

## 📂 Project Structure

```
rag-testcasegen/
├── client/                          # React frontend
│   ├── public/                      # Static files
│   ├── src/
│   │   ├── components/
│   │   │   ├── data/               # Data conversion components
│   │   │   ├── processing/         # Query preprocessing components
│   │   │   └── search/             # Search interface components
│   │   ├── App.js                  # Main app component
│   │   └── index.js                # Entry point
│   └── package.json
│
├── server/                          # Express backend
│   └── index.js                    # Main server file with API endpoints
│
├── src/                             # Scripts and utilities
│   ├── config/                      # Index configurations
│   │   ├── testcases-bm25-index.json
│   │   ├── testcases-vector-index.json
│   │   ├── user-stories-bm25-index.json
│   │   └── user-stories-vector-index.json
│   ├── data/                        # Data files
│   │   ├── stories.json
│   │   ├── testcases.json
│   │   ├── testcases.xlsx
│   │   └── userstories.xlsx
│   └── scripts/
│       ├── data-conversion/         # Excel to JSON conversion
│       │   ├── excel-to-json.js
│       │   ├── excel-to-userstories.js
│       │   └── fetch-jira-stories.js
│       ├── embeddings/              # Embedding generation
│       │   ├── create-embeddings-batch-mistral.js
│       │   └── create-userstories-embeddings-batch-mistral.js
│       ├── query-preprocessing/     # Query enhancement
│       │   ├── abbreviationMapper.js
│       │   ├── dictionaries.js
│       │   ├── normalizer.js
│       │   ├── queryPreprocessor.js
│       │   ├── synonymExpander.js
│       │   └── test-preprocessing.js
│       ├── search/                  # Search implementations
│       │   ├── bm25-search.js
│       │   ├── compare-field-weights.js
│       │   ├── rerank-search.js
│       │   ├── score-fusion-search.js
│       │   ├── search-combined-stores.js
│       │   ├── search-jira-stories.js
│       │   └── search-vector-db.js
│       └── utilities/               # Helper functions
│           ├── groqClient.js
│           ├── mistralEmbedding.js
│           └── delete-all-documents.js
│
├── releases/                        # Release notes and documentation
├── package.json                     # Root package configuration
├── .gitignore                       # Git ignore rules
├── .env.example                     # Environment variables template
└── README.md                        # This file
```

## 🏃 Running the Application

### Development Mode (Client + Server)

```bash
npm run dev
```

This runs both the React client and Node.js server concurrently.

### Client Only

```bash
npm run client
```

Starts React development server on `http://localhost:3000`

### Server Only

```bash
npm run server
```

Starts Express server on `http://localhost:3001`

### Production Build

```bash
npm run build
```

Creates optimized React build in `client/build/`

## 📡 API Documentation

### Test Case Endpoints

#### Upload and Process Excel File
```
POST /api/upload
Content-Type: multipart/form-data

Body: { file: <excel_file> }

Response: { 
  jobId: "job_123",
  message: "Processing started"
}
```

#### Search Test Cases
```
POST /api/search
Content-Type: application/json

Body: {
  query: "login validation",
  method: "hybrid",  // "vector", "bm25", or "hybrid"
  limit: 10,
  filters: {}
}

Response: {
  results: [...],
  metadata: {...}
}
```

#### Get Job Status
```
GET /api/job/:jobId

Response: {
  jobId: "job_123",
  status: "completed",  // "processing", "completed", or "failed"
  progress: 100,
  results: {...}
}
```

#### Generate Test Cases
```
POST /api/generate-testcases
Content-Type: application/json

Body: {
  requirement: "User should be able to login",
  count: 5,
  template: "agile"
}

Response: {
  testcases: [...]
}
```

## 🔍 Scripts & Utilities

### Data Conversion
```bash
# Convert Excel to JSON
node src/scripts/data-conversion/excel-to-json.js <excel_file>

# Convert Excel to User Stories
node src/scripts/data-conversion/excel-to-userstories.js <excel_file>

# Fetch Jira Stories
node src/scripts/data-conversion/fetch-jira-stories.js
```

### Embeddings
```bash
# Generate embeddings for testcases
node src/scripts/embeddings/create-embeddings-batch-mistral.js

# Generate embeddings for user stories
node src/scripts/embeddings/create-userstories-embeddings-batch-mistral.js
```

### Search Operations
```bash
# BM25 Search
node src/scripts/search/bm25-search.js "search query"

# Vector Search
node src/scripts/search/search-vector-db.js "search query"

# Hybrid Search with Re-ranking
node src/scripts/search/rerank-search.js "search query"

# Score Fusion Search
node src/scripts/search/score-fusion-search.js "search query"
```

### Query Preprocessing
```bash
# Test query preprocessing
node src/scripts/query-preprocessing/test-preprocessing.js "your query"
```

### Utilities
```bash
# Delete all documents from collection
node src/scripts/utilities/delete-all-documents.js
```

## 🔎 Search Mechanisms

### 1. BM25 Search
- Traditional keyword-based full-text search
- Field-level weighting for prioritization
- Fast and efficient for exact/partial matches
- Configuration in `config/testcases-bm25-index.json`

### 2. Vector Search
- Semantic similarity using embeddings
- Powered by MongoDB Atlas Vector Search
- Finds contextually related documents
- Uses Mistral AI embeddings (1024 dimensions)

### 3. Hybrid Search
- Combines BM25 and vector search
- Score fusion using configurable weights
- Re-ranking with Groq API
- Best for comprehensive retrieval

### 4. Query Preprocessing
- Normalization: lowercase, special character removal
- Abbreviation expansion (e.g., "pwd" → "password")
- Synonym expansion (e.g., "bug" → "issue, defect")
- Typo tolerance using similarity matching

## 🧠 Embeddings

### Generation
- Uses Mistral AI's embedding model
- 1024-dimensional vectors
- Batch processing with concurrency control
- Stores embeddings in MongoDB

### Configuration
- Model: `mistral-embed`
- Batch size: Configurable
- Dimensions: 1024
- Similarity metric: Cosine

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For issues, questions, or suggestions:
- Open an issue on [GitHub Issues](https://github.com/ddilesh/testcasegenai/issues)
- Check existing documentation in `releases/` folder

## 📚 Additional Resources

- [MongoDB Atlas Vector Search Documentation](https://www.mongodb.com/docs/atlas/atlas-search/vector-search/)
- [Mistral AI API Documentation](https://docs.mistral.ai/)
- [Groq API Documentation](https://console.groq.com/docs)
- [Express.js Documentation](https://expressjs.com/)
- [React Documentation](https://react.dev/)

---

**Last Updated**: January 24, 2026
**Version**: 1.0.0
