# Database

## Overview

Tempest provides data persistence through a query builder and decoupled model architecture supporting SQLite, MySQL, and PostgreSQL databases.

> **Note:** Tempest's database component is currently experimental and is not covered by the backwards compatibility promise.

## Database Configuration

### Default Setup

By default, Tempest connects to a local SQLite database at `.tempest/database.sqlite`. Override this by creating `app/Config/database.config.php`:

```php
use Tempest\Database\Config\SQLiteConfig;
use function Tempest\root_path;

return new SQLiteConfig(
    path: root_path('database.sqlite'),
);
```

### Alternative Databases

Available configuration classes include `SQLiteConfig`, `MysqlConfig`, and `PostgresConfig`:

```php
use Tempest\Database\Config\PostgresConfig;
use function Tempest\env;

return new PostgresConfig(
    host: env('DATABASE_HOST', default: '127.0.0.1'),
    port: env('DATABASE_PORT', default: '5432'),
    username: env('DATABASE_USERNAME', default: 'postgres'),
    password: env('DATABASE_PASSWORD', default: 'postgres'),
    database: env('DATABASE_DATABASE', default: 'postgres'),
);
```

## Querying the Database

### Direct Database Injection

```php
use Tempest\Database\Database;
use Tempest\Database\Query;

final class BookRepository
{
    public function __construct(
        private readonly Database $database,
    ) {}

    public function findById(int $id): array
    {
        return $this->database->fetchFirst(new Query(
            sql: 'SELECT id, title FROM books WHERE id = ?',
            bindings: [$id],
        ));
    }
}
```

### Query Builder Approach

```php
use function Tempest\Database\query;

final class BookRepository
{
    public function findById(int $id): array
    {
        return query('books')
            ->select('id', 'title')
            ->where('id', $id)
            ->first();
    }
}
```

### Combined Method

```php
use Tempest\Database\Database;
use function Tempest\Database\query;

final class BookRepository
{
    public function __construct(
        private readonly Database $database,
    ) {}

    public function findById(int $id): array
    {
        return $this->database->fetchFirst(
            query('books')
                ->select('id', 'title')
                ->where('id = ?', $id),
        );
    }
}
```

## Models

Models represent persisted data as objects. They require no special interface—any PHP object with public typed properties works:

```php
use Tempest\Validation\Rules\HasLength;
use App\Author;

final class Book
{
    #[HasLength(min: 1, max: 120)]
    public string $title;

    public ?Author $author = null;

    /** @var \App\Chapter[] */
    public array $chapters = [];
}
```

### Models with Query Builders

```php
use App\Models\Book;
use function Tempest\Database\query;

final class BookRepository
{
    public function findById(int $id): Book
    {
        return query(Book::class)
            ->select()
            ->with('chapters', 'author')
            ->where('id', $id)
            ->first();
    }
}
```

Tempest infers relation types from property type hints and docblocks.

## Model Relations

### Automatic Inference

Relations are inferred from type information:

```php
final class Book
{
    public ?Author $author = null;
    // ^ BelongsTo relation

    /** @var \App\Books\Chapter[] */
    public array $chapters = [];
    // ^ HasMany relation
}
```

> **Note:** Due to a restriction with reflection, relation types in docblocks must always be fully qualified.

### Explicit Relation Attributes

Use dedicated attributes when property names don't map one-to-one to the database schema:

```php
use Tempest\Database\BelongsTo;
use Tempest\Database\HasMany;
use Tempest\Database\HasOne;

final class Book
{
    #[BelongsTo(ownerJoin: 'author_uuid', relationJoin: 'uuid')]
    public ?Author $author = null;

    /** @var \App\Chapter[] */
    #[HasMany(ownerJoin: 'chapter_uuid', relationJoin: 'uuid')]
    public array $chapters = [];

    #[HasOne(ownerJoin: 'book_uuid', relationJoin: 'uuid')]
    public ?Isbn $isbn = null;
}
```

## UUID Primary Keys

Use the `#[Uuid]` attribute to employ UUID v7 as primary keys:

```php
use Tempest\Database\PrimaryKey;
use Tempest\Database\Uuid;

final class Book
{
    #[Uuid]
    public PrimaryKey $uuid;

    public function __construct(
        public string $title,
        public string $author_name,
    ) {}
}
```

In migrations, use `uuid: true`:

```php
use Tempest\Database\MigratesUp;
use Tempest\Database\QueryStatement;
use Tempest\Database\QueryStatements\CreateTableStatement;

final class CreateBooksTable implements MigratesUp
{
    public string $name = '2024-08-12_create_books_table';

    public function up(): QueryStatement
    {
        return new CreateTableStatement('books')
            ->primary('uuid', uuid: true)
            ->text('title')
            ->text('author_name');
    }
}
```

## Table Names

By default, table names are the pluralized, snake_cased version of the class name. Override with the `#[Table]` attribute:

```php
use Tempest\Database\Table;

#[Table('table_books')]
final class Book
{
    // …
}
```

### Custom Naming Strategies

Implement a custom strategy without applying attributes to each model:

```php
use Tempest\Database\Tables\NamingStrategy;
use function Tempest\Support\str;

final class PrefixedPascalCaseStrategy implements NamingStrategy
{
    public function getName(string $model): string
    {
        return 'table_' . str($model)
            ->classBasename()
            ->pascal()
            ->toString();
    }
}
```

Register in your database config:

```php
use Tempest\Database\Config\SQLiteConfig;

return new SQLiteConfig(
    path: __DIR__ . '/../database.sqlite',
    namingStrategy: new PrefixedPascalCaseStrategy(),
);
```

## Data Transfer Object Properties

Store arbitrary objects in JSON columns using `#[SerializeAs]`:

```php
use Tempest\Mapper\SerializeAs;

final class User
{
    public PrimaryKey $id;

    public function __construct(
        public string $email,
        public Settings $settings,
    ) {}
}

#[SerializeAs('user_settings')]
final class Settings
{
    public function __construct(
        public readonly Theme $theme,
        public readonly bool $hide_sidebar_by_default,
    ) {}
}

enum Theme: string
{
    case DARK = 'dark';
    case LIGHT = 'light';
    case AUTO = 'auto';
}
```

## Property Attributes

### Hashed Properties

The `#[Hashed]` attribute hashes values during serialization, avoiding re-hashing:

```php
final class User
{
    public PrimaryKey $id;

    public function __construct(
        public string $email,
        #[Hashed, SensitiveParameter]
        public ?string $password,
    ) {}
}
```

> **Note:** Hashing requires the `SIGNING_KEY` environment variable to be set.

### Encrypted Properties

The `#[Encrypted]` attribute encrypts during serialization and decrypts during deserialization:

```php
final class User
{
    #[Encrypted]
    public ?string $accessToken;
}
```

### Virtual Properties

Exclude properties from the database mapper using `#[Virtual]`:

```php
use Tempest\Database\Virtual;
use Tempest\DateTime\DateTime;
use Tempest\DateTime\Duration;

final class Book
{
    public DateTime $publishedAt;

    #[Virtual]
    public DateTime $saleExpiresAt {
        get => $this->publishedAt->add(Duration::days(5));
    }
}
```

## IsDatabaseModel Trait

The `IsDatabaseModel` trait provides active record pattern support:

```php
use Tempest\Database\IsDatabaseModel;
use Tempest\Validation\Rules\HasLength;
use App\Author;

final class Book
{
    use IsDatabaseModel;

    #[HasLength(min: 1, max: 120)]
    public string $title;

    public ?Author $author = null;

    /** @var \App\Chapter[] */
    public array $chapters = [];
}
```

Query examples:

```php
$book = Book::create(
    title: 'Timeline Taxi',
    author: $author,
    chapters: [
        new Chapter(index: 1, contents: '…'),
        new Chapter(index: 2, contents: '…'),
        new Chapter(index: 3, contents: '…'),
    ],
);

$books = Book::select()
    ->whereAfter('publishedAt', DateTime::now())
    ->orderBy('title', Direction::DESC)
    ->limit(10)
    ->with('author')
    ->all();

$books[0]->chapters[2]->delete();
```

## Migrations

### Writing Migrations

Classes implementing `MigratesUp` or `MigratesDown` and `.sql` files are auto-discovered:

```php
use Tempest\Database\MigratesUp;
use Tempest\Database\QueryStatement;
use Tempest\Database\QueryStatements\CreateTableStatement;

final class CreateBooksTable implements MigratesUp
{
    public string $name = '2024-08-12_create_books_table';

    public function up(): QueryStatement
    {
        return new CreateTableStatement('books')
            ->primary()
            ->text('title')
            ->datetime('created_at')
            ->datetime('published_at', nullable: true)
            ->belongsTo('books.author_id', 'authors.id');
    }
}
```

Or use raw SQL:

```sql
CREATE TABLE Publisher
(
    `id`   INTEGER,
    `name` TEXT NOT NULL
);
```

> **Note:** The file name of `.sql` migrations and the `$name` property of DatabaseMigration classes determine the order in which migrations are applied.

### Up and Down Migrations

Down migrations require explicitly implementing `MigratesDown`:

```php
use Tempest\Database\MigratesDown;
use Tempest\Database\QueryStatement;
use Tempest\Database\QueryStatements\DropTableStatement;

final class CreateBookTable implements MigratesDown
{
    public string $name = '2024-08-12_drop_book_table';

    public function down(): QueryStatement
    {
        return new DropTableStatement('books');
    }
}
```

### Migration Commands

```bash
# Apply unapplied migrations
./tempest migrate:up

# Drop all tables and rerun migrations
./tempest migrate:fresh

# Validate migration integrity
./tempest migrate:validate
```

### Migration Validation

An integrity check compares current migration hashes with stored values:

```bash
./tempest migrate:validate
```

> **Note:** Only the actual SQL query of a migration, minified and stripped of comments, is hashed during validation.

### Rehashing Migrations

Update migration hashes in the database:

```bash
./tempest migrate:rehash
```

## Database Seeders

Populate databases with data by implementing `DatabaseSeeder`:

```php
use Tempest\Database\DatabaseSeeder;
use UnitEnum;

final class BookSeeder implements DatabaseSeeder
{
    public function run(null|string|UnitEnum $database): void
    {
        query(Book::class)
            ->insert(title: 'Timeline Taxi')
            ->onDatabase($database)
            ->execute();
    }
}
```

Run seeders via:

```bash
./tempest database:seed
./tempest migrate:fresh --seed
```

### Multiple Seeders

Create multiple seeder classes:

```bash
./tempest database:seed --all
./tempest database:seed --seeder="Tests\Tempest\Fixtures\MailingSeeder"
./tempest migrate:fresh --seed --all
```

### Seeding Multiple Databases

Use the `--database` option:

```bash
./tempest database:seed --database="backup"
./tempest migrate:fresh --database="main"
```

## Multiple Databases

### Connecting Multiple Databases

Create config files with tags:

```php
// app/database.config.php
use Tempest\Database\Config\SQLiteConfig;

return new SQLiteConfig(
    path: __DIR__ . '/../database.sqlite',
    tag: 'main',
);
```

```php
// app/database-backup.config.php
use Tempest\Database\Config\SQLiteConfig;

return new SQLiteConfig(
    path: __DIR__ . '/../database-backup.sqlite',
    tag: 'backup',
);
```

Or use enums for better refactorability:

```php
use Tempest\Database\Config\SQLiteConfig;
use App\Database\DatabaseType;

return new SQLiteConfig(
    path: __DIR__ . '/../database-backup.sqlite',
    tag: DatabaseType::BACKUP,
);
```

> **Note:** The default connection is the connection without a tag.

### Querying Multiple Databases

Inject separate database instances:

```php
use Tempest\Database\Database;
use Tempest\Container\Tag;
use App\Database\DatabaseType;
use function Tempest\Database\query;

final class DatabaseBackupCommand
{
    public function __construct(
        private Database $main,
        #[Tag(DatabaseType::BACKUP)] private Database $backup,
    ) {}

    public function __invoke(): void
    {
        $books = $this->main->fetch(
            query(Book::class)
                ->select()
                ->where('published_at < ?', '2025-01-01')
        );

        $this->backup->execute(
            query(Book::class)->insert(...$books)
        );
    }
}
```

Or use the shorthand `onDatabase()` method:

```php
use App\Database\DatabaseType;
use function Tempest\Database\query;

final class DatabaseBackupCommand
{
    public function __invoke(): void
    {
        $books = query(Book::class)
            ->select()
            ->where('published_at < ?', '2025-01-01')
            ->onDatabase(DatabaseType::MAIN)
            ->all();

        query(Book::class)
            ->insert(...$books)
            ->onDatabase(DatabaseType::BACKUP)
            ->execute();
    }
}
```

Works with active-record style models too:

```php
use App\Database\DatabaseType;

final class DatabaseBackupCommand
{
    public function __invoke(): void
    {
        $books = Book::select()
            ->where('published_at < ?', '2025-01-01')
            ->onDatabase(DatabaseType::MAIN)
            ->all();

        Book::insert(...$books)
            ->onDatabase(DatabaseType::BACKUP)
            ->execute();
    }
}
```

### Migrating Multiple Databases

Specify the database with the `--database` flag:

```bash
./tempest migrate:up --database=main
./tempest migrate:down --database=backup
./tempest migrate:fresh --database=main
./tempest migrate:validate --database=backup
```

When unspecified, the default database is used.

### Database-Specific Migrations

Implement `ShouldMigrate` to conditionally run migrations:

```php
use Tempest\Database\Database;
use Tempest\Database\MigratesUp;
use Tempest\Database\ShouldMigrate;

final class MigrationForBackup implements MigratesUp, ShouldMigrate
{
    public string $name = '…';

    public function shouldMigrate(Database $database): bool
    {
        return $database->tag === DatabaseType::BACKUP;
    }

    public function up(): QueryStatement
    { /* … */ }
}
```

### Dynamic Databases

For multi-tenant systems, add databases dynamically:

```php
final class ConnectTenant
{
    public function __invoke(string $tenantId): void
    {
        $this->container->config(new SQLiteConfig(
            path: __DIR__ . "/tenant-{$tenantId}.sqlite",
            tag: $tenantId,
        ));
    }
}
```

Run migrations programmatically using `MigrationManager`:

```php
use Tempest\Database\Migrations\MigrationManager;

final class OnboardTenant
{
    public function __construct(
        private MigrationManager $migrationManager,
    ) {}

    public function __invoke(string $tenantId): void
    {
        $setupMigrations = [
            new CreateMigrationsTable(),
            // Additional migrations
        ];

        foreach ($setupMigrations as $migration) {
            $this->migrationManager->onDatabase($tenantId)->executeUp($migration);
        }
    }
}
```

Register dynamic connections within application entry points via middleware or kernel events:

```php
use Tempest\Container\Container;
use Tempest\Router\HttpMiddleware;
use Tempest\Core\Priority;

#[Priority(Priority::HIGHEST)]
final class ConnectTenantMiddleware implements HttpMiddleware
{
    public function __construct(
        private Container $container,
    ) {}

    public function __invoke(Request $request, HttpMiddlewareCallable $next): Response
    {
        $tenantId = // Tenant ID resolution from request

        (new ConnectTenant)($tenantId);

        return $next($request);
    }
}
```
