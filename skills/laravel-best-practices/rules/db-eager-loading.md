---
title: Eager Loading to Prevent N+1 Queries
impact: CRITICAL
impactDescription: 10-100× performance improvement
tags: eloquent, n+1, performance, database, eager-loading
---

## Eager Loading to Prevent N+1 Queries

**Impact: CRITICAL (10-100× performance improvement)**

## Why It Matters

The N+1 query problem is one of the most common performance issues in Laravel. Without eager loading, accessing a relationship in a loop triggers a separate query for each item.

## Incorrect

```php
// ❌ N+1 Problem - 1 query for posts + N queries for authors
$posts = Post::all();

foreach ($posts as $post) {
    echo $post->author->name;  // Query executed for EACH post
}

// With 100 posts, this executes 101 queries!
```

```php
// ❌ N+1 in Blade template
@foreach($posts as $post)
    <p>{{ $post->author->name }}</p>        // Query per post
    <p>{{ $post->category->name }}</p>      // Another query per post
    @foreach($post->tags as $tag)           // Yet another query per post
        <span>{{ $tag->name }}</span>
    @endforeach
@endforeach
```

## Correct

```php
// ✅ Eager loading - 3 queries total regardless of post count
$posts = Post::with(['author', 'category', 'tags'])->get();

foreach ($posts as $post) {
    echo $post->author->name;     // No additional query
    echo $post->category->name;   // No additional query
    foreach ($post->tags as $tag) {
        echo $tag->name;          // No additional query
    }
}
```

### Nested Eager Loading

```php
// ✅ Load nested relationships
$posts = Post::with([
    'author.profile',           // author and their profile
    'author.posts',             // author and their other posts
    'comments.user',            // comments and comment authors
    'comments.replies.user',    // nested replies
])->get();
```

### Constrained Eager Loading

```php
// ✅ Add constraints to eager loaded relationships
$posts = Post::with([
    // Only load approved comments
    'comments' => function ($query) {
        $query->where('approved', true)
              ->orderBy('created_at', 'desc');
    },

    // Only load recent comments
    'comments' => fn ($query) => $query->latest()->limit(5),

    // Select specific columns
    'author' => fn ($query) => $query->select('id', 'name', 'avatar'),
])->get();
```

### Lazy Eager Loading

```php
// ✅ Load after initial query
$posts = Post::all();

// Later, load relationships
$posts->load('author', 'category');

// With constraints
$posts->load([
    'comments' => fn ($query) => $query->latest(),
]);
```

### Load Missing

```php
// ✅ Only load if not already loaded
$posts = Post::with('author')->get();

// Won't reload author, will load comments
$posts->loadMissing(['author', 'comments']);
```

### Eager Load Counts

```php
// ✅ Load count without loading all records
$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;  // No additional query
}

// With constraints
$posts = Post::withCount([
    'comments',
    'comments as approved_comments_count' => fn ($query) =>
        $query->where('approved', true),
])->get();
```

### Prevent Lazy Loading (Development)

```php
// app/Providers/AppServiceProvider.php
use Illuminate\Database\Eloquent\Model;

public function boot(): void
{
    // Throw exception on lazy loading (dev only)
    Model::preventLazyLoading(! app()->isProduction());

    // Or just log instead of throwing
    Model::handleLazyLoadingViolationUsing(function ($model, $relation) {
        Log::warning("Lazy loading {$relation} on {$model}");
    });
}
```

### Common Patterns

```php
// ✅ In controller
class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with(['author', 'category'])
            ->latest()
            ->paginate(15);

        return view('posts.index', compact('posts'));
    }

    public function show(Post $post)
    {
        // Load for single model
        $post->load(['author', 'comments.user', 'tags']);

        return view('posts.show', compact('post'));
    }
}

// ✅ Default eager loading in model
class Post extends Model
{
    // Always load these relationships
    protected $with = ['author'];
}
```

## Detection

Use Laravel Debugbar or Telescope to detect N+1 queries:
- Look for repeated queries
- Check query count per request
- Use `preventLazyLoading()` in development

## Impact

- 10x-100x performance improvement
- Reduced database load
- Faster page load times
