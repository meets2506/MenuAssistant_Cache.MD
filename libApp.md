# GenerateNano SDK

## Overview

The `GenerateNano.jar` SDK powers both **POS** and **Restaurant** applications by offering an all-in-one solution for:
- Vector-based and traditional search (Graph, Semantic, BM25)
- Portal initialization and communication
- LLM (Large Language Model) integration with streaming and prompt-response support

This README explains the architecture, core modules, and how to use the SDK in a simple and practical way.

---

## Applications Supported

- **POS App**: Customer checkout, billing, item scanning, etc.
- **Restaurant App**: Menu handling, order processing, table tracking, etc.

---

## What’s Inside `GenerateNano.jar`

| Module/File               | Description                                         |
|--------------------------|-----------------------------------------------------|
| `graph.jar`              | Enables graph-based search                         |
| `semantic_search.jar`    | Adds context-aware, semantic search                |
| `bm25.jar`               | Traditional keyword-based search using BM25        |
| `portal.jar`             | Handles communication with external APIs or portals|
| `interplay_interface.so` | Connects to LLM models (also referred to as `libllama.so`) |

---

## Core Functions

### 1. LLM Functions

- `generateStartLLM(llm, streaming_function)`  
  Starts LLM in streaming mode.

- `generateInitLLM(context, mlock, config)`  
  Initializes LLM with memory locking and context setup.

- `generateLLMResponse(prompt)`  
  Gets a response from the LLM for the given prompt.

---

### 2. Vector Database Functions

- `generateInitVDB(path, search_type)`  
  Initializes the VDB with Graph / BM25 / Semantic type.

- `generateSearchVDB(vdb, query)`  
  Executes a search on the initialized VDB.

---

### 3. Portal Communication Functions

- `generatePortalInit(portal_ip_settings)`  
  Initializes communication with the remote portal using IP/config.

- `generatePortalSend(config, path)`  
  Sends data or instructions to the portal.

---

## Architecture Diagram

```
POS App       Restaurant App
     \             //
    [ Uses GenerateNano.jar SDK ]
               |
  ---------------------------------------
 | Search Modules       |  Portal Module |  LLM Interface |
 | - graph.jar          |  - portal.jar  |  - libllama.so |
 | - bm25.jar           |                |                |
 | - semantic_search.jar|                |                |
  ---------------------------------------
               |
        [ Exposed SDK Functions ]
        - generateInitLLM
        - generateStartLLM
        - generateLLMResponse
        - generateInitVDB
        - generateSearchVDB
        - generatePortalInit
        - generatePortalSend
```

---

## How to Use

1. Import `GenerateNano.jar` and required `.jar` and `.so` files in your app.
2. Initialize components:
   - Call `generateInitLLM` to setup LLM
   - Call `generateInitVDB` to load the vector database
   - Call `generatePortalInit` to connect the app to your portal
3. Use runtime methods:
   - `generateLLMResponse()` for dynamic responses
   - `generateSearchVDB()` for search results
   - `generatePortalSend()` to send data to external systems

---

## Notes

- All search types (Graph, Semantic, BM25) are pluggable.
- LLM uses a native `.so` file (`interplay_interface.so`) to interface directly with system-level models like LLaMA.
- Keep configuration paths consistent for portability.

---

## Support

For integration help or bug reports, contact your internal SDK team.
