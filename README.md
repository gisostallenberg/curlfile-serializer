# CURLFileSerializer

Serialize and unserialize a CURLFile

> ## ⚠️ Avoid in new code
>
> This package implements the `Serializable` interface, which is **deprecated as
> of PHP 8.1** and slated for removal: it emits a deprecation notice today and
> will eventually stop working. Its `unserialize()` also does
> `extract(unserialize($data))` — injecting variables into local scope, and
> running a nested `unserialize()` with no `allowed_classes` guard (an
> object-injection footgun if the blob is ever untrusted).
>
> There is no alternative package, and you cannot serialize a `CURLFile`
> directly either — PHP forbids that since 7.4. The job is only round-tripping
> its three strings, so store those and rebuild the object. On a wrapper class,
> use the modern `__serialize()`/`__unserialize()` magic methods (PHP 7.4+) —
> no package needed:
>
> ```php
> public function __serialize(): array
> {
>     return [
>         $this->curlfile->getFileName(),
>         $this->curlfile->getMimeType(),
>         $this->curlfile->getPostFilename(),
>     ];
> }
>
> public function __unserialize(array $data): void
> {
>     [$name, $mime, $postname] = $data;
>     $this->curlfile = new CURLFile($name, $mime, $postname);
> }
> ```
>
> Or skip the wrapper entirely and persist the three strings yourself. Note that
> moving to the magic methods changes the stored format, so treat it as a
> breaking change for any already-persisted data.

## Installation
```bash
composer require gisostallenberg/curlfile-serializer
```

## Usage example
```php
<?php

use GisoStallenberg\CURLFileSerializer\CURLFileSerializer;

$curlFileSerializer = CURLFileSerializer::create(new CURLFile('/path/to/file.txt'));
$serialized = serialize($curlFileSerializer);

$curlFileSerializer = unserialize($serialized);
$curlFile = $curlFileSerializer->getCURLFile(); // Is an instanceof CURLFile
```