# Symfony AI - Store Component Documentation

## Overview

The Store component provides a low-level abstraction for storing and retrieving documents in a vector store. It's designed for use cases like **Retrieval Augmented Generation (RAG)** in agentic applications.

## Installation

```bash
composer require symfony/ai-store
```

## Purpose

The Store component implements low-level interfaces that can be implemented by different vendor-specific bridges. It provides higher-level features to:
- Populate stores with documents
- Query stores for similar documents using vector similarity search

## Key Features

### 1. Indexing

The **Indexer** service populates a store with documents by converting them into embeddings:

```php
use Symfony\AI\Store\Document\TextDocument;
use Symfony\AI\Store\Indexer;

$indexer = new Indexer($platform, $model, $store);
$document = new TextDocument('This is a sample document.');
$indexer->index($document);
```

### 2. Retrieving

The **Retriever** service searches for documents based on a query string by vectorizing the query and retrieving similar documents:

```php
use Symfony\AI\Store\Retriever;

$retriever = new Retriever($vectorizer, $store);
$documents = $retriever->retrieve('What is the capital of France?');

foreach ($documents as $document) {
    echo $document->metadata->get('source');
}
```

Optional parameters can customize retrieval (limit, filters, etc.).

## Supported Vector Stores

- Azure AI Search
- Chroma
- Cloudflare Vectorize
- InMemory
- Manticore Search
- MariaDB Vector
- Meilisearch
- Milvus
- MongoDB Atlas
- Neo4j
- OpenSearch
- Pinecone
- PostgreSQL (pgvector)
- Qdrant
- Supabase
- SurrealDB
- Symfony Cache
- Typesense
- Weaviate

**Note:** InMemory and PSR-6 cache stores load all data into PHP memory and are suitable only for testing.

## Console Commands

When using the Store component with AiBundle in Symfony:

```bash
# Initialize the store
php bin/console ai:store:setup <store_name>

# Clean up the store
php bin/console ai:store:drop <store_name>
```

Configuration example:
```yaml
# config/packages/ai.yaml
ai:
  store:
    chromadb:
      symfonycon:
        collection: 'symfony_blog'
```

## Implementing a Custom Bridge

Implement the **StoreInterface** to create a custom vector store:

```php
use Symfony\AI\Platform\Vector\Vector;
use Symfony\AI\Store\Document\VectorDocument;
use Symfony\AI\Store\StoreInterface;

class MyStore implements StoreInterface
{
    public function add(VectorDocument ...$documents): void
    {
        // Add vectorized documents to store
    }

    public function query(Vector $vector, array $options = []): iterable
    {
        // Query store for similar documents
        return $documents;
    }
}
```

### Managing Stores

For stores requiring setup/teardown (table creation, indexes, etc.), implement **ManagedStoreInterface**:

```php
use Symfony\AI\Store\ManagedStoreInterface;
use Symfony\AI\Store\StoreInterface;

class MyCustomStore implements ManagedStoreInterface, StoreInterface
{
    public function setup(array $options = []): void
    {
        // Create store structure
    }

    public function drop(array $options = []): void
    {
        // Drop store and related vectors
    }
}
```

## Examples

Full working examples are available in the Symfony AI repository:
- Basic Retriever example
- RAG examples for: Cloudflare, Manticore, MariaDB, Meilisearch, In-Memory, Milvus, MongoDB, Neo4j, OpenSearch, Pinecone, Qdrant, SurrealDB, Symfony Cache, Typesense, Weaviate, and Supabase

---

**License**: This work is licensed under a [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) license.
