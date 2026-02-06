# Laravel 12.x Eloquent Relationships

## Introduction

Eloquent relationships make managing database relationships easy. Laravel supports several relationship types:

- One To One
- One To Many
- Many To Many
- Has One Through
- Has Many Through
- Polymorphic Relationships

## One to One (Has One)

A one-to-one relationship associates one model with exactly one other model.

### Defining the Relationship

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

### Accessing the Relationship

```php
$phone = User::find(1)->phone;
```

### Custom Foreign Keys

```php
return $this->hasOne(Phone::class, 'foreign_key');
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

### Defining the Inverse (BelongsTo)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

### Default Models

Return a default model when the relationship is null:

```php
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}

// With attributes
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}
```

## One to Many (Has Many)

A one-to-many relationship defines a relationship where a single model is the parent to one or more child models.

### Defining the Relationship

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

### Accessing the Relationship

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

### Adding Constraints

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

### Automatically Hydrating Parent Models (Chaperone)

Prevent N+1 queries by using the `chaperone()` method:

```php
public function comments(): HasMany
{
    return $this->hasMany(Comment::class)->chaperone();
}

// Or at query time:
$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

## Has One of Many

Retrieve a single related model from a one-to-many relationship:

```php
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}

public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}

public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

## Has One Through

A one-to-one relationship that proceeds through an intermediate model.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

### Using Defined Relationships

```php
return $this->through('cars')->has('owner');

// Dynamic
return $this->throughCars()->hasOwner();
```

## Has Many Through

Provides convenient access to distant relations via an intermediate relation.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```

## Many to Many Relationships

### Table Structure

```
users
    id - integer
    name - string

roles
    id - integer
    name - string

role_user
    user_id - integer
    role_id - integer
```

### Model Structure

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

### Accessing the Relationship

```php
$user = User::find(1);

foreach ($user->roles as $role) {
    // ...
}

$roles = User::find(1)->roles()->orderBy('name')->get();
```

### Custom Table and Keys

```php
return $this->belongsToMany(Role::class, 'role_user');
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

### Defining the Inverse

```php
class Role extends Model
{
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

### Retrieving Intermediate Table Columns

```php
$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

### Specifying Additional Pivot Columns

```php
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');
return $this->belongsToMany(Role::class)->withTimestamps();
```

### Customizing the Pivot Attribute Name

```php
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();

// Access via custom name
$podcast->subscription->created_at;
```

### Filtering via Pivot Table

```php
return $this->belongsToMany(Role::class)
    ->wherePivot('approved', 1);

return $this->belongsToMany(Role::class)
    ->wherePivotIn('priority', [1, 2]);

return $this->belongsToMany(Podcast::class)
    ->wherePivotNull('expired_at');

return $this->belongsToMany(Role::class)
    ->withPivotValue('approved', 1);
```

### Ordering via Pivot Table

```php
return $this->belongsToMany(Badge::class)
    ->orderByPivot('created_at', 'desc');
```

### Custom Pivot Models

```php
class Role extends Model
{
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class)->using(RoleUser::class);
    }
}
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    public $incrementing = true;
}
```

## Polymorphic Relationships

### One to One (Polymorphic)

```php
class Image extends Model
{
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}

class User extends Model
{
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

### One to Many (Polymorphic)

```php
class Comment extends Model
{
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

### Many to Many (Polymorphic)

```php
class Post extends Model
{
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}

class Tag extends Model
{
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

### Custom Polymorphic Types

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);
```

## Querying Relations

### Relationship Existence

```php
// Posts with at least one comment
$posts = Post::has('comments')->get();

// Posts with 3+ comments
$posts = Post::has('comments', '>=', 3)->get();

// Nested
$posts = Post::has('comments.images')->get();
```

### Querying with Constraints

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();
```

### Inline Relationship Queries

```php
$posts = Post::whereRelation('comments', 'is_approved', false)->get();
```

### Querying Relationship Absence

```php
$posts = Post::doesntHave('comments')->get();

$posts = Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();
```

## Aggregating Related Models

```php
$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}

// Multiple relations
$posts = Post::withCount(['comments', 'likes'])->get();

// Other aggregates
$posts = Post::withSum('comments', 'votes')->get();
$posts = Post::withAvg('comments', 'votes')->get();
$posts = Post::withMin('comments', 'votes')->get();
$posts = Post::withMax('comments', 'votes')->get();
```

## Eager Loading

Prevent N+1 queries:

```php
// Bad: N+1 queries
$books = Book::all();
foreach ($books as $book) {
    echo $book->author->name;
}

// Good: Eager load
$books = Book::with('author')->get();
foreach ($books as $book) {
    echo $book->author->name;
}
```

### Eager Loading Multiple Relationships

```php
$books = Book::with(['author', 'publisher'])->get();

// Nested
$books = Book::with('author.country')->get();
```

### Constraining Eager Loads

```php
use Illuminate\Database\Eloquent\Builder;

$users = User::with([
    'posts' => function (Builder $query) {
        $query->where('title', 'like', '%code%');
    },
])->get();
```

### Lazy Eager Loading

```php
$users = User::all();

if ($someCondition) {
    $users->load('posts', 'comments');
}

// Only load if not already loaded
$users->loadMissing('posts');
```

### Preventing Lazy Loading

```php
use Illuminate\Database\Eloquent\Model;

Model::preventLazyLoading(!app()->isProduction());
```

## Inserting & Updating Related Models

### The save Method

```php
$post = Post::find(1);

$comment = new Comment(['message' => 'A new comment.']);
$post->comments()->save($comment);

// Save multiple
$post->comments()->saveMany([
    new Comment(['message' => 'Comment 1']),
    new Comment(['message' => 'Comment 2']),
]);
```

### The create Method

```php
$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);

// Create multiple
$post->comments()->createMany([
    ['message' => 'Comment 1'],
    ['message' => 'Comment 2'],
]);
```

### Belongs To Relationships

```php
$comment = Comment::find(1);

$comment->post()->associate($post);
$comment->save();

// Dissociate
$comment->post()->dissociate();
$comment->save();
```

### Many to Many Relationships

```php
$user = User::find(1);

// Attach
$user->roles()->attach($roleId);
$user->roles()->attach([$roleId1, $roleId2]);

// With pivot data
$user->roles()->attach($roleId, ['expires_at' => now()->addMonths(3)]);

// Detach
$user->roles()->detach($roleId);
$user->roles()->detach();

// Sync
$user->roles()->sync([1, 2, 3]);
$user->roles()->sync([1 => ['expires_at' => now()], 2, 3]);

// Toggle
$user->roles()->toggle([1, 2, 3]);

// Update pivot
$user->roles()->updateExistingPivot($roleId, ['active' => false]);
```

## Touching Parent Timestamps

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    protected $touches = ['post'];

    public function post()
    {
        return $this->belongsTo(Post::class);
    }
}
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/eloquent-relationships)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
