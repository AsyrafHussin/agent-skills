---
title: Controller Middleware
impact: MEDIUM
impactDescription: Reusable request filtering and modification
tags: controllers, middleware, authentication, authorization
---

## Controller Middleware

**Impact: MEDIUM (Reusable request filtering and modification)**

Use middleware to handle cross-cutting concerns like authentication, rate limiting, and request modification.

## Bad Example

```php
// Authentication and authorization checks in every method
class AdminController extends Controller
{
    public function index()
    {
        if (!auth()->check()) {
            return redirect()->route('login');
        }

        if (!auth()->user()->isAdmin()) {
            abort(403);
        }

        return view('admin.dashboard');
    }

    public function users()
    {
        if (!auth()->check()) {
            return redirect()->route('login');
        }

        if (!auth()->user()->isAdmin()) {
            abort(403);
        }

        return view('admin.users');
    }

    public function settings()
    {
        if (!auth()->check()) {
            return redirect()->route('login');
        }

        if (!auth()->user()->isAdmin()) {
            abort(403);
        }

        return view('admin.settings');
    }
}
```

## Good Example

```php
// Middleware in controller constructor
class AdminController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('admin');

        // Apply to specific methods
        $this->middleware('verified')->only(['store', 'update']);
        $this->middleware('throttle:10,1')->only('store');
    }

    public function index()
    {
        return view('admin.dashboard');
    }

    public function users()
    {
        return view('admin.users');
    }
}

// Using middleware in routes
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin', [AdminController::class, 'index']);
    Route::resource('admin/users', AdminUserController::class);
});

// Using controller middleware attribute (Laravel 10+)
#[Middleware(['auth', 'admin'])]
class AdminController extends Controller
{
    public function index()
    {
        return view('admin.dashboard');
    }
}

// Custom middleware
namespace App\Http\Middleware;

class EnsureUserIsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        if (!$request->user()?->isAdmin()) {
            abort(403, 'Unauthorized. Admin access required.');
        }

        return $next($request);
    }
}

// Register middleware alias in bootstrap/app.php (Laravel 11+)
return Application::configure(basePath: dirname(__DIR__))
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'admin' => EnsureUserIsAdmin::class,
            'subscribed' => EnsureUserIsSubscribed::class,
        ]);
    })
    ->create();

// Or in Kernel.php (Laravel 10 and earlier)
protected $middlewareAliases = [
    'admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
];

// Middleware with parameters
namespace App\Http\Middleware;

class CheckRole
{
    public function handle(Request $request, Closure $next, string ...$roles): Response
    {
        if (!$request->user()?->hasAnyRole($roles)) {
            abort(403);
        }

        return $next($request);
    }
}

// Usage with parameters
Route::get('/reports', [ReportController::class, 'index'])
    ->middleware('role:admin,manager');

// Middleware for API rate limiting
Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('posts', PostController::class);
});

// Conditional middleware
class ApiController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth:sanctum');

        $this->middleware(function ($request, $next) {
            if ($request->user()->needsTwoFactor()) {
                return response()->json([
                    'message' => '2FA required',
                    'requires_2fa' => true,
                ], 403);
            }

            return $next($request);
        });
    }
}

// Middleware groups for common patterns
// bootstrap/app.php
$middleware->group('api', [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
]);
```

## Why

- **DRY**: Authentication/authorization logic in one place
- **Reusability**: Same middleware used across multiple controllers
- **Separation of concerns**: Controllers focus on business logic
- **Composability**: Stack multiple middleware for complex requirements
- **Testability**: Middleware tested independently
- **Declarative**: Clear what protection is applied just by looking at the route/controller
