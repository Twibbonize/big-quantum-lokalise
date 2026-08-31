# AGENTS.md

Translation-content repo. No build, no tests, no app code — only `lokalise/*.json`.
Push to `staging` or `main` syncs `lokalise/` to S3 (`.github/workflows/github-actions.yml`).

## Layout

`lokalise/` holds three independent file families, one file per locale:

| Family | Files | Contents |
|---|---|---|
| main | `<locale>.json` | all web UI strings |
| notification | `notification-<locale>.json` | push/email notification copy |
| time | `time-<locale>.json` | relative-time strings |

54 locales: `af ar bg bn cs da de el en en-au en-gb es fa fi fr gu he hi hr hu id it ja km kn ko lo ml mr ms my nb nl or pa pl pt ro ru sk sq sr sv sw ta te th tl-ph tr uk ur vi zh-cn zh-tw`

`en.json` is source of truth. Every other file mirrors its key tree.

## Adding keys

1. Add to `lokalise/en.json` first, inside the existing section (`page_*`, `general.*`, …). Nesting is `page_x.section.key`, snake_case leaves.
2. Add the **same key path** to all other 53 locales with a real translation — never copy the English string as a placeholder, never leave the key out.
3. Formatting: 4-space indent, trailing newline, no BOM. Preserve existing key order; append new sections at the end of the object.
4. Do not touch the top-level `"default": "EN"` — it is `EN` in every file, including non-English ones.
5. Only edit the family the key belongs to. A UI string does not go into `notification-*` or `time-*`.

Verify before commit:

```sh
# every locale parses and has the new key
for f in lokalise/*.json; do python3 -c "import json,sys;json.load(open(sys.argv[1]))" "$f" || echo "BAD $f"; done
grep -L '"my_new_key"' lokalise/[a-z]*.json   # lists files missing it
```

## Placeholders

Two systems coexist. Match what the surrounding file already uses — do not convert one to the other.

### `{name}` — ICU / interpolation (main files)

`{plan}`, `{count}`, `{username}`, `{price}`, `{date}`, `{link}`, `{br}`, …

- Keep the placeholder token **identical** in every locale. `{plan}` stays `{plan}` in Arabic.
- Translate only the words around it. Reorder freely to fit target grammar.
- `{br}` and `{link}`/`{icon}` are markup slots — never wrap them, never duplicate them.

### `[%x]` — server-side tokens (notification files)

Full reference: [`lokalise/NOTIFICATION_TOKENS.md`](lokalise/NOTIFICATION_TOKENS.md). Catalogue of every notification string: [`lokalise/NOTIFICATION_KEYS.md`](lokalise/NOTIFICATION_KEYS.md).

| Token | Means |
|---|---|
| `[%p]` | plan / package name |
| `[%n]` | number (supporter count etc.) |
| `[%g]` | date (grace/expiry) |
| `[%c]` | creator / user display name |
| `[%s]` | campaign name |
| `[%r]` | reason (education grant revocation) |

Verbatim, case-sensitive, brackets included. Never translate, never pluralize the word inside, never change to `{}`.

These replaced the positional `{0}` that `notification-*` used previously — a single string carried two different `{0}` values, so which was which depended on argument order. Do not reintroduce `{0}` in these files.

Word order is free: `[%c]` and `[%s]` may appear in whichever order the target grammar needs (`ja`/`ko` put `[%s]` first in `panel.approval.body`). Only the token spelling is fixed.

### `%s` / `%d` — printf (time files only)

`"past": "%s ago"`, `"minutes": "%d minutes"`. Keep the specifier; translate the rest.

## ICU plurals

Plural keys use ICU MessageFormat, not Symfony pipes (`one|other` was converted in `daee84c` — do not reintroduce it).

```json
"supporters": "{count, plural, one{supporter} other{supporters}}"
```

Rules:

- Variable name (`count`) is identical across all locales.
- Include exactly the CLDR plural categories the target locale has — no more, no less. Extra categories are ignored; missing ones fall back wrongly.
- Category order: `zero one two few many other`.
- `other` is mandatory in every locale.

CLDR categories per locale in this repo (taken from the converted keys, e.g. `general.campaign.supporters`):

| Categories | Locales |
|---|---|
| `other` | id ja km ko lo ms my th vi zh-cn zh-tw |
| `one other` | af bg bn da de el en en-au en-gb es fa fi fr gu hi hu it kn ml mr nb nl or pa pt sq sv sw ta te tl-ph tr ur |
| `one few other` | hr ro sr |
| `one few many other` | cs pl ru sk uk |
| `one two many other` | he |
| `zero one two few many other` | ar |

- Locales with no grammatical plural still need the wrapper: `{count, plural, other{…}}`.
- Copy the category set from an already-converted plural key in the same file rather than guessing.

## Context for translators

Nothing in the JSON carries context — key path is the only signal. So:

- Name keys by **where** and **what**: `page_video_campaign.errors.too_large`, not `msg_17`.
- Before translating, read the sibling keys in the same section to get the surface (button label vs. body copy vs. toast) and pick register accordingly — short imperative for buttons, full sentence with punctuation for descriptions.
- Match the tone `id.json` and `en.json` already use for that section: informal second person (`kamu`), sentence case, no trailing period on labels, period on multi-sentence body copy.
- UI-length matters: a button label that triples in length in German breaks layout. Keep labels near source length.
- Do not translate brand terms: Twibbon, Twibbonize, Chroma Key, Beta, Watermark (unless the section already localizes it).
- Record notable additions in `lokalise/DIFF_staging_vs_main.md` style when doing a bulk import (key | en | id table).

## Commits

One logical change per commit; touching all 54 files in one commit is normal and expected for a key addition.
