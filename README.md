# meraki/schema-language-english

English messages for [`meraki/schema`](https://github.com/merakiframework/schema).

**Data, not code.** This package contains two files — `en.mfr` and `en_AU.mfr` — and no PHP at all.
That is deliberate: they are [ICU MessageFormat 2](https://unicode.org/reports/tr35/tr35-messageFormat.html)
resources, so a Rust or JavaScript implementation of `meraki/schema` reads exactly the same wording.
A sentence is written once and rendered identically wherever the schema is used.

## Using it

```
composer require meraki/schema-language-english
```

```php
use Meraki\Schema\Facade;
use Meraki\Schema\Message\Mf2\Mf2Provider;

$schema = new Facade('signup', messages: Mf2Provider::fromPackage('meraki/schema-language-english'));
$schema->add($schema->createAddressField('billing', ['AU']));

$result = $schema->validate($data, locale: 'en-AU');

$result->forField('billing')->messages->forPart('postal_code')->first;
// "That is not a valid postcode for the country you chose."
```

To change a sentence without forking this package, lay your own directory over it — a later pack
overrides an earlier one entry by entry:

```php
Mf2Provider::fromPackage('meraki/schema-language-english')
    ->withPack(__DIR__ . '/../resources/lang');
```

## What is in here

| | |
| --- | --- |
| `en.mfr` | the base: a sentence for every failure the library reports |
| `en_AU.mfr` | Australian English — five words that differ, and nothing else |

`en_AU.mfr` is short on purpose. It redefines `part.postal_code` as *postcode*, and every message
that mentions one follows, because those messages say `{$part}` rather than spelling it out. A
variant that stays this short is a variant working properly.

Note what does **not** belong in `en_AU.mfr`: this is the language a reader speaks, not the country
an address is in. An Australian reader can be entering a New Zealand address.

## Checking it

```
composer install
composer test
```

which is `schema-lang validate .` — the linter that ships with `meraki/schema`. Since this package
has no code, that is what stands in for a compiler. It reads the vocabulary off the library's own
classes, so it cannot go stale, and it catches a message using an unimplemented MessageFormat
feature, a key the library never asks for, and — the one that matters most — a message naming a
variable nothing will fill, such as `{$minimum}` where the library supplies `{$bound}`.

`schema-lang keys` lists every key a pack may define and the variables each may name.

## Writing another language

Copy the structure: a repository with `.mfr` files at the root, one per locale, `@locale` at the top
of each, and a `composer.json` with no autoload section. Then wire the same check into CI.

The four variables are `{$field}`, `{$kind}`, `{$part}` and `{$bound}`, and they arrive already in
your language — `{$kind}` and `{$part}` resolve through the `kind.*` and `part.*` entries you
define, and a list-valued `{$bound}` is joined using your own `list.separator` and
`list.lastSeparator`.

Full instructions are in [docs/MESSAGES.md](https://github.com/merakiframework/schema/blob/main/docs/MESSAGES.md).

## Development

The dev dependency currently points at a sibling checkout:

```json
"repositories": [{ "type": "path", "url": "../schema" }]
```

Swap that for a version constraint from Packagist once `meraki/schema` 2.0 is tagged.

## License

MIT.
