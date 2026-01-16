---
title: Form Request Classes for Validation
impact: HIGH
impactDescription: Separation of concerns and reusable validation
tags: validation, form-requests, authorization, clean-code
---

## Form Request Classes for Validation

**Impact: HIGH (Separation of concerns and reusable validation)**

## Why It Matters

Form Request classes separate validation from controllers, making code cleaner, reusable, and easier to test. They also provide a dedicated place for authorization logic.

## Incorrect

```php
// ❌ Validation in controller - cluttered and not reusable
class PostController extends Controller
{
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body' => 'required|string|min:100',
            'category_id' => 'required|exists:categories,id',
            'tags' => 'nullable|array',
            'tags.*' => 'exists:tags,id',
            'published_at' => 'nullable|date|after:now',
        ]);

        // Check authorization manually
        if (!auth()->user()->can('create', Post::class)) {
            abort(403);
        }

        // ... create post
    }

    public function update(Request $request, Post $post)
    {
        // Same validation repeated
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'body' => 'required|string|min:100',
            // ...
        ]);

        // ...
    }
}
```

## Correct

### Create Form Request

```bash
php artisan make:request StorePostRequest
php artisan make:request UpdatePostRequest
```

### Form Request Class

```php
<?php

namespace App\Http\Requests;

use App\Models\Post;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class StorePostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return $this->user()->can('create', Post::class);
    }

    /**
     * Get the validation rules that apply to the request.
     */
    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'slug' => ['nullable', 'string', 'max:255', 'unique:posts,slug'],
            'body' => ['required', 'string', 'min:100'],
            'excerpt' => ['nullable', 'string', 'max:500'],
            'category_id' => ['required', 'exists:categories,id'],
            'tags' => ['nullable', 'array'],
            'tags.*' => ['exists:tags,id'],
            'published_at' => ['nullable', 'date', 'after:now'],
            'featured_image' => ['nullable', 'image', 'max:2048'],
        ];
    }

    /**
     * Get custom messages for validator errors.
     */
    public function messages(): array
    {
        return [
            'body.min' => 'The post content must be at least 100 characters.',
            'category_id.exists' => 'The selected category does not exist.',
            'featured_image.max' => 'The image must not exceed 2MB.',
        ];
    }

    /**
     * Get custom attributes for validator errors.
     */
    public function attributes(): array
    {
        return [
            'category_id' => 'category',
            'published_at' => 'publication date',
        ];
    }

    /**
     * Prepare the data for validation.
     */
    protected function prepareForValidation(): void
    {
        $this->merge([
            'slug' => $this->slug ?? Str::slug($this->title),
        ]);
    }
}
```

### Update Request with Different Rules

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class UpdatePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('update', $this->route('post'));
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'slug' => [
                'nullable',
                'string',
                'max:255',
                // Ignore current post when checking uniqueness
                Rule::unique('posts', 'slug')->ignore($this->route('post')),
            ],
            'body' => ['required', 'string', 'min:100'],
            'category_id' => ['required', 'exists:categories,id'],
            'tags' => ['nullable', 'array'],
            'tags.*' => ['exists:tags,id'],
        ];
    }
}
```

### Clean Controller

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Http\Requests\UpdatePostRequest;
use App\Models\Post;

class PostController extends Controller
{
    public function store(StorePostRequest $request)
    {
        // Validation and authorization happen automatically

        $post = Post::create($request->validated());

        if ($request->hasFile('featured_image')) {
            $post->addMediaFromRequest('featured_image')
                 ->toMediaCollection('featured');
        }

        return redirect()
            ->route('posts.show', $post)
            ->with('success', 'Post created successfully.');
    }

    public function update(UpdatePostRequest $request, Post $post)
    {
        $post->update($request->validated());

        return redirect()
            ->route('posts.show', $post)
            ->with('success', 'Post updated successfully.');
    }
}
```

### Conditional Validation

```php
public function rules(): array
{
    return [
        'payment_method' => ['required', 'in:credit_card,bank_transfer'],

        // Only required if payment method is credit card
        'card_number' => [
            Rule::requiredIf($this->payment_method === 'credit_card'),
            'nullable',
            'string',
            'size:16',
        ],

        // Required sometimes
        'coupon_code' => [
            'sometimes',
            'nullable',
            'exists:coupons,code',
        ],
    ];
}
```

### After Validation Hook

```php
public function after(): array
{
    return [
        function (Validator $validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add(
                    'field',
                    'Something is wrong with this field!'
                );
            }
        }
    ];
}
```

## Benefits

- Separation of concerns
- Reusable validation rules
- Authorization in one place
- Testable in isolation
- Cleaner controllers
