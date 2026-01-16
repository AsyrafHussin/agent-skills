---
name: laravel-best-practices
description: Laravel 12 conventions and best practices. Use when creating controllers, models, migrations, validation, services, or structuring Laravel applications. Triggers on tasks involving Laravel architecture, Eloquent, database, API development, or PHP patterns.
---

# Laravel 12 Best Practices

Comprehensive best practices guide for Laravel 12 applications. Contains 45+ rules across 8 categories for building scalable, maintainable Laravel applications.

## When to Apply

Reference these guidelines when:
- Creating controllers, models, and services
- Writing migrations and database queries
- Implementing validation and form requests
- Building APIs with Laravel
- Structuring Laravel applications

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Architecture & Structure | CRITICAL | `arch-` |
| 2 | Eloquent & Database | CRITICAL | `db-` |
| 3 | Controllers & Routing | HIGH | `ctrl-` |
| 4 | Validation & Requests | HIGH | `valid-` |
| 5 | Security | HIGH | `sec-` |
| 6 | Performance | MEDIUM | `perf-` |
| 7 | API Design | MEDIUM | `api-` |
| 8 | Testing | LOW-MEDIUM | `test-` |

## Quick Reference

### 1. Architecture & Structure (CRITICAL)

- `arch-single-responsibility` - One job per class
- `arch-service-classes` - Extract business logic to services
- `arch-action-classes` - Single-purpose action classes
- `arch-repository-pattern` - When to use repositories
- `arch-dto-pattern` - Data transfer objects
- `arch-folder-structure` - Organize by domain/feature

### 2. Eloquent & Database (CRITICAL)

- `db-eager-loading` - Prevent N+1 queries
- `db-chunking` - Process large datasets
- `db-query-scopes` - Reusable query logic
- `db-model-events` - Use observers for side effects
- `db-migrations` - Migration best practices
- `db-indexes` - Proper indexing strategy

### 3. Controllers & Routing (HIGH)

- `ctrl-resource-controllers` - Use resource controllers
- `ctrl-thin-controllers` - Keep controllers thin
- `ctrl-route-model-binding` - Implicit binding
- `ctrl-api-resources` - Transform responses
- `ctrl-invokable` - Single action controllers

### 4. Validation & Requests (HIGH)

- `valid-form-requests` - Use form request classes
- `valid-custom-rules` - Create custom rules
- `valid-authorization` - Authorize in form requests
- `valid-messages` - Custom error messages
- `valid-conditional` - Conditional validation

### 5. Security (HIGH)

- `sec-mass-assignment` - Protect against mass assignment
- `sec-sql-injection` - Prevent SQL injection
- `sec-xss` - Prevent XSS attacks
- `sec-csrf` - CSRF protection
- `sec-authentication` - Auth best practices
- `sec-authorization` - Policies and gates

### 6. Performance (MEDIUM)

- `perf-caching` - Cache strategies
- `perf-queues` - Queue heavy operations
- `perf-lazy-collections` - Memory efficient processing
- `perf-database-optimization` - Query optimization
- `perf-config-cache` - Cache configuration

### 7. API Design (MEDIUM)

- `api-versioning` - API versioning strategy
- `api-resources` - API resource transformers
- `api-pagination` - Paginate responses
- `api-error-handling` - Consistent error responses
- `api-rate-limiting` - Rate limiting

### 8. Testing (LOW-MEDIUM)

- `test-feature-tests` - Feature/integration tests
- `test-unit-tests` - Unit test patterns
- `test-factories` - Model factories
- `test-mocking` - Mock external services
- `test-database` - Database testing

## Essential Patterns

### Controller with Form Request

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Http\Requests\UpdatePostRequest;
use App\Models\Post;
use Illuminate\Http\RedirectResponse;

class PostController extends Controller
{
    public function store(StorePostRequest $request): RedirectResponse
    {
        // Validation happens automatically
        $validated = $request->validated();

        $post = Post::create($validated);

        return redirect()
            ->route('posts.show', $post)
            ->with('success', 'Post created successfully.');
    }

    public function update(UpdatePostRequest $request, Post $post): RedirectResponse
    {
        $post->update($request->validated());

        return redirect()
            ->route('posts.show', $post)
            ->with('success', 'Post updated successfully.');
    }
}
```

### Form Request Class

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Post::class);
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'body' => ['required', 'string', 'min:100'],
            'category_id' => ['required', 'exists:categories,id'],
            'tags' => ['nullable', 'array'],
            'tags.*' => ['exists:tags,id'],
            'published_at' => ['nullable', 'date', 'after:now'],
        ];
    }

    public function messages(): array
    {
        return [
            'body.min' => 'The post body must be at least 100 characters.',
        ];
    }
}
```

### Service Class Pattern

```php
<?php

namespace App\Services;

use App\Models\User;
use App\Models\Post;
use App\Events\PostPublished;
use Illuminate\Support\Facades\DB;

class PostService
{
    public function __construct(
        private readonly NotificationService $notifications,
    ) {}

    public function publish(Post $post): Post
    {
        return DB::transaction(function () use ($post) {
            $post->update([
                'published_at' => now(),
                'status' => 'published',
            ]);

            event(new PostPublished($post));

            $this->notifications->notifyFollowers($post->author, $post);

            return $post->fresh();
        });
    }
}
```

### Eloquent Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Builder;

class Post extends Model
{
    use HasFactory;

    protected $fillable = [
        'title',
        'slug',
        'body',
        'category_id',
        'published_at',
    ];

    protected $casts = [
        'published_at' => 'datetime',
    ];

    // Relationships
    public function author(): BelongsTo
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function tags(): BelongsToMany
    {
        return $this->belongsToMany(Tag::class)->withTimestamps();
    }

    // Scopes
    public function scopePublished(Builder $query): Builder
    {
        return $query->whereNotNull('published_at')
            ->where('published_at', '<=', now());
    }

    public function scopeByCategory(Builder $query, int $categoryId): Builder
    {
        return $query->where('category_id', $categoryId);
    }

    // Accessors & Mutators
    protected function title(): Attribute
    {
        return Attribute::make(
            set: fn (string $value) => ucfirst($value),
        );
    }
}
```

### Migration Best Practices

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->foreignId('category_id')->constrained()->cascadeOnDelete();
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('body');
            $table->timestamp('published_at')->nullable();
            $table->timestamps();

            // Indexes for common queries
            $table->index(['user_id', 'published_at']);
            $table->index('category_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

### Eager Loading

```php
// ❌ N+1 Problem
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name;  // Query per post
}

// ✅ Eager loading
$posts = Post::with(['author', 'category', 'tags'])->get();
foreach ($posts as $post) {
    echo $post->author->name;  // No additional queries
}

// ✅ Nested eager loading
$posts = Post::with([
    'author.profile',
    'comments.user',
    'tags',
])->get();

// ✅ Constrained eager loading
$posts = Post::with([
    'comments' => fn ($query) => $query->latest()->limit(5),
])->get();
```

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/arch-service-classes.md
rules/db-eager-loading.md
rules/valid-form-requests.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Laravel-specific context and references
