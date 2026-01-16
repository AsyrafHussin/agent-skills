# PHP Best Practices

Modern PHP 8.x patterns, PSR standards, and SOLID principles for clean, maintainable code.

## Overview

This skill provides guidance for:
- PHP 8.x modern features
- Type system best practices
- PSR standards compliance
- SOLID principles
- Error handling patterns
- Performance optimization

## Categories

### 1. Type System (Critical)
Strict types, return types, union/intersection types, nullable handling.

### 2. Modern Features (Critical)
Constructor promotion, readonly, enums, attributes, match expressions, named arguments.

### 3. PSR Standards (High)
PSR-4 autoloading, PSR-12 coding style, naming conventions.

### 4. SOLID Principles (High)
Single responsibility, open/closed, Liskov substitution, interface segregation, dependency inversion.

### 5. Error Handling (High)
Custom exceptions, specific catches, proper exception hierarchy.

### 6. Performance (Medium)
Generators, lazy loading, native functions, avoiding globals.

### 7. Security (Critical)
Input validation, output escaping, password hashing, prepared statements.

## Usage

Ask Claude to:
- "Review my PHP code"
- "Check PHP types"
- "Audit PHP for SOLID"
- "Check PHP best practices"

## Key Guidelines

### Always Use
- `declare(strict_types=1)` at file start
- Constructor property promotion
- Readonly properties for immutable data
- Enums instead of class constants
- Match expressions over switch
- Named arguments for clarity
- Type declarations everywhere

### Avoid
- Mixed type when specific type possible
- Hard-coded dependencies
- Fat interfaces
- Suppressing errors with @
- Global variables
- God classes

## References

- [PHP Manual](https://www.php.net/manual/)
- [PHP-FIG PSR Standards](https://www.php-fig.org/psr/)
- [PHP The Right Way](https://phptherightway.com/)
