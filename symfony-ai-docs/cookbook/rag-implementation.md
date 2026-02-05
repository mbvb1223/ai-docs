# Implementing Retrieval Augmented Generation (RAG) - Symfony Docs

## What is RAG?

Retrieval Augmented Generation combines vector search with language models to provide agents access to external knowledge. RAG systems:

1. Convert documents into vector embeddings
2. Store embeddings in a vector database
3. Find similar documents based on user queries
4. Provide retrieved context to the language model
5. Generate responses based on retrieved information

**Ideal for:**
- Knowledge bases and documentation
- Product catalogs
- Customer support systems
- Research assistants
- Domain-specific chatbots

## Prerequisites

- Symfony AI Platform component
- Symfony AI Agent component
- Symfony AI Store component
- An embeddings model (e.g., OpenAI's text-embedding-3-small)
- A language model (e.g., gpt-4o-mini)
- Optional: A vector store (or use in-memory for testing)

## Step-by-Step Implementation

### Step 1: Initialize the Vector Store

```php
use Symfony\AI\Store\InMemory\Store;

$store = new Store();
```

For production, use persistent stores like ChromaDB, Pinecone, or MongoDB Atlas.

### Step 2: Prepare Documents

```php
use Symfony\AI\Store\Document\Metadata;
use Symfony\AI\Store\Document\TextDocument;
use Symfony\Component\Uid\Uuid;

$documents = [];
foreach ($movies as $movie) {
    $documents[] = new TextDocument(
        id: Uuid::v4(),
        content: 'Title: '.$movie['title'].PHP_EOL.
                'Director: '.$movie['director'].PHP_EOL.
                'Description: '.$movie['description'],
        metadata: new Metadata($movie),
    );
}
```

Each document needs:
- **ID**: Unique identifier (UUID v4 recommended)
- **Content**: Text to be embedded and searched
- **Metadata**: Additional information preserved with document

### Step 3: Create Embeddings and Index

```php
use Symfony\AI\Store\Document\Loader\InMemoryLoader;
use Symfony\AI\Store\Document\Vectorizer;
use Symfony\AI\Store\Indexer\DocumentIndexer;
use Symfony\AI\Store\Indexer\DocumentProcessor;

$platform = PlatformFactory::create(env('OPENAI_API_KEY'));
$vectorizer = new Vectorizer($platform, 'text-embedding-3-small');
$indexer = new DocumentIndexer(new DocumentProcessor($vectorizer, $store));
$indexer->index($documents);
```

The `DocumentIndexer` handles:
- Transforming/filtering documents
- Generating vector embeddings
- Storing vectors in the vector store

### Step 4: Configure Similarity Search Tool

```php
use Symfony\AI\Agent\Bridge\SimilaritySearch\SimilaritySearch;
use Symfony\AI\Agent\Toolbox\AgentProcessor;
use Symfony\AI\Agent\Toolbox\Toolbox;

$similaritySearch = new SimilaritySearch($vectorizer, $store);
$toolbox = new Toolbox([$similaritySearch]);
$processor = new AgentProcessor($toolbox);
```

The `SimilaritySearch` tool:
- Converts user queries into vectors
- Searches for similar vectors in store
- Returns most relevant documents

### Step 5: Create RAG-Enabled Agent

```php
use Symfony\AI\Agent\Agent;

$agent = new Agent(
    $platform,
    'gpt-4o-mini',
    [$processor],  // Input processors
    [$processor]   // Output processors
);
```

### Step 6: Query with Context

```php
use Symfony\AI\Platform\Message\Message;
use Symfony\AI\Platform\Message\MessageBag;

$messages = new MessageBag(
    Message::forSystem('Please answer all user questions only using SimilaritySearch function.'),
    Message::ofUser('Which movie fits the theme of the mafia?')
);
$result = $agent->call($messages);
```

The agent will:
1. Analyze the user's question
2. Call the similarity search tool
3. Retrieve relevant documents
4. Generate responses based on retrieved context

## Production-Ready RAG Systems

### Vector Store Selection - ChromaDB

```php
use Symfony\AI\Store\Bridge\ChromaDb\Store;

$store = new Store(
    $httpClient,
    'http://localhost:8000',
    'my_collection'
);
```

ChromaDB benefits:
- Local or self-hosted deployment
- Efficient vector similarity search
- Built-in persistence
- Easy Symfony AI integration

### Document Loading Strategies

**File-based loading:**
```php
use Symfony\AI\Store\Document\Loader\TextFileLoader;

$loader = new TextFileLoader('/path/to/documents');
```

**Database loading:**
```php
use Symfony\AI\Store\Document\Loader\InMemoryLoader;

$articles = $articleRepository->findAll();
$documents = array_map(
    fn($article) => new TextDocument(
        id: $article->getId(),
        content: $article->getTitle().PHP_EOL.$article->getContent(),
        metadata: new Metadata(['author' => $article->getAuthor()])
    ),
    $articles
);

$loader = new InMemoryLoader($documents);
```

## Advanced Configurations

### Chunking Large Documents

```php
use Symfony\AI\Store\Document\Transformer\TextSplitTransformer;

$transformer = new TextSplitTransformer(
    chunkSize: 1000,
    overlap: 200
);

$chunkedDocuments = $transformer->transform($documents);
```

Automatically:
- Splits documents into specified size chunks
- Adds overlap between chunks for context
- Preserves original metadata
- Tracks parent document IDs

### Custom Similarity Metrics

```yaml
# config/packages/ai.yaml
ai:
    store:
        memory:
            default:
                strategy: 'cosine'  # or 'euclidean', 'manhattan', 'chebyshev'
```

### Metadata Filtering

```php
$result = $store->query($vector, [
    'where' => [
        'category' => 'technical',
        'status' => 'published',
    ],
]);
```

Document content filtering:
```php
$result = $store->query($vector, [
    'where' => ['category' => 'technical'],
    'whereDocument' => ['$contains' => 'machine learning'],
]);
```

## Bundle Configuration

```yaml
# config/packages/ai.yaml
ai:
    platform:
        openai:
            api_key: '%env(OPENAI_API_KEY)%'

    vectorizer:
        default:
            platform: 'ai.platform.openai'
            model: 'text-embedding-3-small'

    store:
        chromadb:
            knowledge_base:
                collection: 'docs'

    indexer:
        docs:
            loader: 'App\Document\Loader\DocLoader'
            vectorizer: 'ai.vectorizer.default'
            store: 'ai.store.chromadb.knowledge_base'

    agent:
        rag_assistant:
            model: 'gpt-4o-mini'
            prompt:
                text: 'Answer questions using only the SimilaritySearch tool. If you cannot find relevant information, say so.'
            tools:
                - 'Symfony\AI\Agent\Bridge\SimilaritySearch\SimilaritySearch'
```

Then populate the store:
```bash
php bin/console ai:store:setup chromadb.knowledge_base
php bin/console ai:store:index docs
```

## Performance Optimization

### Batch Indexing

```php
$batchSize = 100;
foreach (array_chunk($documents, $batchSize) as $batch) {
    $indexer->index($batch);
}
```

### Caching Embeddings

```php
use Symfony\Contracts\Cache\CacheInterface;

class CachedVectorizer
{
    public function __construct(
        private Vectorizer $vectorizer,
        private CacheInterface $cache,
    ) {
    }

    public function vectorize(string $text): Vector
    {
        $key = 'embedding_'.md5($text);

        return $this->cache->get($key, function() use ($text) {
            return $this->vectorizer->vectorize($text);
        });
    }
}
```

## Best Practices

1. **Document Quality**: Ensure well-structured, relevant documents
2. **Chunk Size**: Experiment with 500-1500 token chunks
3. **Metadata**: Include useful metadata for filtering
4. **System Prompt**: Explicitly instruct agent to use similarity search
5. **Limit Results**: Balance relevance and context size
6. **Update Strategy**: Plan incremental updates vs. full reindexing
7. **Monitor Performance**: Track query latency and relevance metrics
8. **Test Queries**: Validate retrieval returns expected results

## Common Pitfalls

- **Too Large Chunks**: Reduces retrieval precision
- **Too Small Chunks**: Loses context
- **Missing Instructions**: Agent needs explicit similarity search instructions
- **Poor Document Quality**: Garbage in, garbage out
- **Incorrect Embeddings Model**: Use same model for indexing and querying
- **No Metadata**: Limits filtering capabilities

## Related Resources

- [Complete Example](https://github.com/symfony/ai/blob/main/examples/rag/in-memory.php)
- [RAG with ChromaDB](https://github.com/symfony/ai/blob/main/examples/rag/chromadb.php)
- [RAG with MongoDB](https://github.com/symfony/ai/blob/main/examples/rag/mongodb.php)
- [RAG with Pinecone](https://github.com/symfony/ai/blob/main/examples/rag/pinecone.php)
- [RAG with Meilisearch](https://github.com/symfony/ai/blob/main/examples/rag/meilisearch.php)

---

**License**: This work is licensed under a [Creative Commons BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) license.
