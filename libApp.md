# Graph-Based Search Engine for LLM API Server

## Overview

This document explains the graph-based search functionality integrated into the Java LLM API server. The graph search offers an alternative to the traditional BM25 search method, providing more semantically relevant search results for complex queries.

## Architecture

```
┌──────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                  │     │                 │     │                 │
│  Client Request  │────▶│  LlamaServer    │────▶│  Search Engine  │
│  (HTTP API)      │     │  (API Handler)  │     │  (BM25/Graph)   │
│                  │     │                 │     │                 │
└──────────────────┘     └─────────────────┘     └────────┬────────┘
                                                          │
                                                          ▼
┌──────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                  │     │                 │     │                 │
│  Response        │◀────│  Response       │◀────│  Document       │
│  to Client       │     │  Processing     │     │  Retrieval      │
│                  │     │                 │     │                 │
└──────────────────┘     └─────────────────┘     └─────────────────┘
```

### Core Components

1. **GraphSearchEngine Interface**: Defines the contract for graph-based search operations.
2. **MacGraphSearchEngine**: Mac-specific implementation of the graph search engine.
3. **Response Generator**: Processes search results to generate appropriate responses.
4. **Fallback Mechanism**: Provides summarized information when direct Q&A matches aren't found.

## How It Works

### 1. Initialization

The graph search engine is initialized at server startup:

```java
// Initialize the graph search engine
graphSearchEngine = MacGraphSearchEngine.getInstance();
int graphInitResult = graphSearchEngine.initialize("data/source", GRAPH_INDEX_PATH, MAX_SEARCH_RESULTS);
```

### 2. Index Building

The graph index is built from source documents:

```java
// Build the graph search index
int buildResult = graphSearchEngine.buildIndex();
```

### 3. Query Processing

When a client submits a query with `search_method=graph`:

1. The query is validated to ensure it's related to payment terminals
2. The graph search engine retrieves relevant document snippets
3. The system checks for direct Question & Answer matches in the results
4. If a direct match is found, it returns the answer
5. If no direct match is found, it provides a summarized response of the top snippets

### 4. Fallback Mechanism

The fallback mechanism activates when no direct Q&A match is found:

```java
// If using graph search and no direct Q&A match, provide a summary of the top snippets
if (searchMethod.equalsIgnoreCase("graph")) {
    StringBuilder summary = new StringBuilder();
    summary.append("Based on the information available:\n");
    int maxSnippets = Math.min(3, snippets.size());
    for (int i = 0; i < maxSnippets; i++) {
        Snippet snippet = snippets.get(i);
        summary.append("- ").append(snippet.content.trim())
               .append(" (Source: ").append(snippet.docName).append(")\n");
    }
    return summary.toString();
}
```

## How to Use

### API Usage

To use the graph search functionality, send a POST request to the API endpoint with the `search_method` parameter set to "graph":

```bash
curl -X POST http://localhost:8002/api/query \
     -H "Content-Type: application/json" \
     -d '{
           "query": "How does payment processing work?",
           "search_method": "graph"
         }'
```

### Response Format

For graph search with direct Q&A matches:
```json
{
  "text": "Answer from matched Q&A pair"
}
```

For graph search with no direct Q&A matches (fallback):
```json
{
  "text": "Based on the information available:
- Snippet 1 content (Source: document1.txt)
- Snippet 2 content (Source: document2.txt)
- Snippet 3 content (Source: document3.txt)"
}
```

## Benefits Over BM25

1. **Semantic Understanding**: Graph search understands query meaning, not just keywords
2. **Better for Complex Queries**: Handles nuanced questions more effectively
3. **Direct Q&A Matching**: Can find specific answers in Q&A formatted documents
4. **Fallback Mechanism**: Provides useful information even when exact answers aren't found

## Configuration

The graph search is configured in `LlamaServer.java`:

```java
private static final String GRAPH_INDEX_PATH = "data/graph_index";
private static final int MAX_SEARCH_RESULTS = 10;
```

## Testing

Test the graph search functionality using the provided test script:

```bash
python3 run_test_queries.py
```

This script tests both BM25 and graph search methods with various queries to compare their responses.

## Troubleshooting

Common issues and solutions:

1. **No results returned**: Ensure the graph index has been properly built
2. **Empty responses**: Check that the query is related to payment terminals
3. **Performance issues**: The graph search may be more resource-intensive than BM25

## Further Development

Future enhancements planned for the graph search:

1. Improved semantic matching algorithm
2. Support for multi-language queries
3. Integration with vector embeddings for even better semantic understanding
4. Query expansion to handle synonyms and related terms
