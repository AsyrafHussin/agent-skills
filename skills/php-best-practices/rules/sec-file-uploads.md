---
title: Validate File Uploads on Content, Not Client Claims
impact: CRITICAL
impactDescription: Security best practices and input safety
tags: php, security, validation
---

# Validate File Uploads on Content, Not Client Claims

## Why it matters
A file upload handler that trusts the client-supplied filename or MIME type can be trivially abused to upload a PHP shell and achieve remote code execution. The client controls every byte in the upload request — filename, stated type, and content. The server must independently verify all of them.

## Rule
Validate content type using `finfo` (not `$_FILES['type']`), restrict extensions to an explicit allowlist, generate a random filename, enforce a server-side size limit, store outside the web root, and serve through a controller with access checks.

## Bad
```php
<?php

declare(strict_types=1);

// No validation — uploads shell.php directly into the web root
move_uploaded_file(
    $_FILES['upload']['tmp_name'],
    'uploads/' . $_FILES['upload']['name']
);

// Extension-only check is bypassed with "shell.php.jpg" or null bytes
$ext = pathinfo($_FILES['upload']['name'], PATHINFO_EXTENSION);
if ($ext === 'jpg') {
    move_uploaded_file($_FILES['upload']['tmp_name'], 'uploads/' . $_FILES['upload']['name']);
}

// Client MIME type can be set to anything
if ($_FILES['upload']['type'] === 'image/jpeg') {
    move_uploaded_file($_FILES['upload']['tmp_name'], 'public/uploads/' . $_FILES['upload']['name']);
}
```

## Better
```php
<?php

declare(strict_types=1);

// Uses finfo (good), but still uses original filename (bad)
// and stores in web-accessible directory (bad)
$finfo    = new \finfo(FILEINFO_MIME_TYPE);
$mimeType = $finfo->file($_FILES['upload']['tmp_name']);
$allowed  = ['image/jpeg', 'image/png'];

if (in_array($mimeType, $allowed, true)) {
    move_uploaded_file(
        $_FILES['upload']['tmp_name'],
        'public/uploads/' . $_FILES['upload']['name'] // still dangerous
    );
}
```

## Best
```php
<?php

declare(strict_types=1);

final class FileUploadHandler
{
    private const MAX_BYTES = 5 * 1024 * 1024; // 5 MB

    /** @var array<string, string> mime => extension */
    private const ALLOWED = [
        'image/jpeg' => 'jpg',
        'image/png'  => 'png',
        'image/webp' => 'webp',
        'application/pdf' => 'pdf',
    ];

    public function __construct(private readonly string $storageRoot) {}

    /** @param array{error:int,size:int,tmp_name:string} $file */
    public function handle(array $file): string
    {
        if ($file['error'] !== UPLOAD_ERR_OK) {
            throw new UploadException("Upload error code: {$file['error']}");
        }

        if ($file['size'] > self::MAX_BYTES) {
            throw new UploadException('File exceeds the 5 MB limit');
        }

        // Detect content type from file bytes, not from the request
        $finfo    = new \finfo(FILEINFO_MIME_TYPE);
        $mimeType = $finfo->file($file['tmp_name']);

        if (!isset(self::ALLOWED[$mimeType])) {
            throw new UploadException("Unsupported file type: {$mimeType}");
        }

        // Random filename — never use the client-supplied name
        $ext      = self::ALLOWED[$mimeType];
        $filename = bin2hex(random_bytes(16)) . '.' . $ext;
        $dest     = $this->storageRoot . '/' . $filename;

        if (!move_uploaded_file($file['tmp_name'], $dest)) {
            throw new UploadException('Failed to persist uploaded file');
        }

        // For images: verify the file is a real image
        if (str_starts_with($mimeType, 'image/') && getimagesize($dest) === false) {
            unlink($dest);
            throw new UploadException('File is not a valid image');
        }

        return $filename;
    }
}

// Serve through a controller — never expose storage URLs directly
function serveFile(string $filename, string $storageRoot): void
{
    // Authorize the current user here before serving

    $path = $storageRoot . '/' . basename($filename); // basename strips traversal
    if (!is_file($path)) {
        throw new NotFoundException('File not found');
    }

    $finfo = new \finfo(FILEINFO_MIME_TYPE);
    header('Content-Type: '        . $finfo->file($path));
    header('Content-Length: '      . filesize($path));
    header('Content-Disposition: attachment; filename="' . basename($filename) . '"');
    readfile($path);
}
```

## Exceptions / trade-offs
Cloud storage (S3, GCS) with pre-signed upload URLs moves the storage concern off-server, but content-type and size validation must still happen before generating the pre-signed URL or via a server-side hook.

## Static-analysis notes
PHPStan and Psalm cannot detect unsafe `move_uploaded_file` calls without custom rules. The key invariant to enforce in review: no path derived from user input reaches the second argument of `move_uploaded_file`.

## Security notes
`$_FILES['name']` can contain `../` sequences and null bytes on older PHP versions. Always use `basename()` if any part of the original filename must be referenced, though the preferred approach is to discard it entirely and generate a random name.

`random_bytes(16)` provides 128 bits of entropy — sufficient to make filename enumeration infeasible.

## Related topics
- [sec-input-validation.md](sec-input-validation.md) — general boundary validation principles
- [sec-output-escaping.md](sec-output-escaping.md) — escaping filenames when reflected in HTML
