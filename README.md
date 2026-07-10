# Graph RAG - Knowledge Graph Based Retrieval

A comprehensive exploration of **Graph-based Retrieval-Augmented Generation (RAG)** using two complementary approaches to build intelligent document retrieval systems.

## 🎯 Overview

This repository demonstrates two advanced RAG techniques that leverage knowledge graphs for semantic understanding and intelligent document retrieval:

1. **Metadata-driven Graph Retrieval** (`graph_rag[1].ipynb`) — Uses manual metadata extraction and graph relationships
2. **LLM-powered Graph Transformation** (`graph_rag_with_llmgraph_transformer.ipynb`) — Automatically extracts entities and relationships using an LLM

Both approaches build knowledge graphs to improve retrieval accuracy beyond simple vector similarity search.

---

## 🔄 Approach 1: Metadata-Driven Graph Retrieval

### How It Works

**Step 1: Document Processing**
- Load PDF documents and split into chunks (1000 tokens with 200-token overlap)
- Each chunk retains its original structure and page information

**Step 2: Structured Metadata Extraction**
- For each chunk, use Gemini to extract four metadata categories:
  - **Topics**: Main concepts discussed
  - **People**: Names and entities mentioned
  - **Organizations**: Companies and institutions
  - **Technologies**: Tools, frameworks, and languages used
- Metadata is attached to each chunk as structured fields

**Step 3: Vector Embedding & Storage**
- Convert chunks to vector embeddings using Google Generative AI Embeddings
- Store in an in-memory vector store with embedded metadata

**Step 4: Graph-Based Retrieval**
- Build a knowledge graph with edges connecting chunks via shared metadata
- When a query arrives:
  - Find initial matching chunks via vector similarity
  - Traverse the graph using the **Eager** strategy to find related chunks
  - Respect a max depth limit (2 hops) to avoid over-retrieval
- Return the expanded set of relevant documents

**Step 5: Answer Generation**
- Concatenate retrieved documents as context
- Send to Gemini with a prompt asking to identify all relevant projects and technologies
- Output a detailed, structured answer

### Key Advantage
Metadata-driven retrieval explicitly links documents by concepts, people, and organizations—finding contextually related chunks that vector search alone might miss.

---

## 🔄 Approach 2: LLM-Powered Graph Transformation

### How It Works

**Step 1: Document Processing**
- Load PDF and split into larger chunks (1500 tokens with 100-token overlap)
- Larger chunks preserve more context for entity extraction

**Step 2: Automatic Entity & Relationship Extraction**
- Use `LLMGraphTransformer` with Gemini to parse each chunk and extract:
  - **Entities** (nodes): Person, EducationalInstitution, Degree, Skill, Technology, Project, etc.
  - **Relationships** (edges): STUDIED_AT, HAS_SKILL, WORKED_ON, USES, RESEARCHED, etc.
- The LLM intelligently identifies domain-specific entities and their connections

**Step 3: Knowledge Graph Construction**
- Aggregate all extracted nodes and relationships from all chunks
- Build a directed NetworkX graph where:
  - Nodes represent entities (e.g., "Python", "Machine Learning", "John Doe")
  - Edges represent typed relationships (e.g., "John Doe" --HAS_SKILL--> "Python")

**Step 4: Query & Answer**
- Use `GraphQAChain` to answer questions directly against the knowledge graph
- The chain traverses the graph to find connected entities and their properties
- Generates natural language answers grounded in the extracted graph structure

**Step 5: Graph Analysis (Optional)**
- Compute graph statistics: node count, edge count, density
- Identify most connected nodes (hubs in the knowledge)
- Analyze relationship type distribution to understand domain structure

### Key Advantage
Fully automatic extraction means no manual metadata definition needed. The LLM understands domain context and builds a rich, reusable knowledge graph without engineering effort.

---

## 📊 Comparison

| Feature | Approach 1 | Approach 2 |
|---------|-----------|-----------|
| **Metadata Definition** | Manual (topics, people, orgs, tech) | Automatic LLM extraction |
| **Entity Types** | Fixed categories | Customizable (Person, Project, etc.) |
| **Relationship Model** | Similarity-based graph edges | Typed relationships (STUDIED_AT, etc.) |
| **Retrieval Strategy** | Vector → Graph traversal (Eager) | Direct graph query via GraphQAChain |
| **Setup Effort** | Medium (define metadata schema) | Low (configure entity types) |
| **Interpretability** | Clear, user-defined edges | Emergent from LLM extraction |
| **Best For** | Structured documents with clear topics | Documents with rich entity relationships |

---

## 🚀 Use Cases

- **Resume/CV Analysis**: Extract skills, education, projects; find gaps or highlight strengths
- **Research Papers**: Link citations, methodologies, and datasets; enable systematic literature review
- **Technical Documentation**: Connect concepts, tools, and use cases across documentation
- **Knowledge Management**: Build organizational knowledge graphs from unstructured documents

---

## 📦 Dependencies

Both notebooks require:
- `langchain` — LLM orchestration
- `langchain-google-genai` — Gemini LLM & embeddings
- `langchain-experimental` — LLMGraphTransformer
- `networkx` — Graph operations
- `pypdf` — PDF loading
- `json-repair` — Robust JSON parsing

Install all dependencies:
```bash
pip install langchain langchain-google-genai langchain-experimental \
            networkx pypdf json-repair langchain-graph-retriever
```

---

## 🔑 API Setup

Both approaches use Google Generative AI (Gemini). Get your free API key:
1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Create a new API key
3. Replace `"your gemini api key"` in the notebooks

**Free tier limits**: ~20 requests/day per key. For higher volume, use multiple keys or upgrade.

---

## 📝 Running the Notebooks

### Approach 1 (Metadata-Driven)
1. Upload a PDF document (tested with resumes, but works with any PDF)
2. Metadata is automatically extracted for each chunk
3. Enter a query to retrieve related documents via graph traversal
4. Answer is generated using Gemini

### Approach 2 (LLM-Powered)
1. Upload a PDF document
2. The LLM extracts entities and relationships automatically
3. A knowledge graph is built and visualized (if small enough)
4. Use the interactive query interface to ask questions
5. View graph statistics and relationship distributions

---

## 💡 Key Insights

**When to use Approach 1:**
- You have clear, predefined categories (metadata types)
- You want explicit control over what gets linked
- Metadata is relatively consistent across documents
- You need predictable, auditable retrieval logic

**When to use Approach 2:**
- Documents have rich entity relationships
- You want to automatically discover new connection types
- You're building a reusable knowledge graph
- Interpretability of relationships is less critical than coverage

**Hybrid Approach:**
Combine both! Use Approach 1 for fast retrieval with human-friendly metadata, then use Approach 2 to discover additional entity connections for deeper analysis.

---

## 📚 Graph RAG vs. Standard RAG

| Aspect | Standard RAG | Graph RAG |
|--------|-------------|-----------|
| **Retrieval Logic** | Vector similarity only | Similarity + graph traversal/reasoning |
| **Relationship Model** | Implicit (learned in embeddings) | Explicit (nodes, edges, types) |
| **Interpretability** | "Why this document?" harder to explain | Clear path: "Entity A connects to B via relationship R" |
| **Scalability** | Simple, fast | More complex but enables richer queries |
| **Use Cases** | General Q&A | Structured domain knowledge, entity-centric queries |

---

## 🔮 Future Enhancements

- **Multi-hop Reasoning**: Answer questions requiring chains of entity relationships
- **Temporal Graphs**: Track how entities and relationships evolve over time
- **Confidence Scoring**: Assign confidence to extracted entities and relationships
- **Dynamic Updates**: Add new documents and incrementally extend the graph
- **Web UI**: Interactive dashboard for graph exploration and querying
- **Hybrid Retrieval**: Combine vector search with graph traversal in a single pipeline

---

## 📄 License

Open source. Feel free to adapt for your use case.

---

## 🤝 Contributing

Ideas for improvement? Found a bug? PRs welcome!

---

**Built with ❤️ using LangChain, Gemini, and NetworkX**
