# CLAUDE Processing Notes — ioBroker Adapter Reviews

Corrections and refinements to review logic from `CLAUDE-AdapterReview.txt`. Read before every review. This file takes precedence where it conflicts with the instructions file.

Role reference: https://github.com/ioBroker/ioBroker.docs/blob/master/docs/en/dev/stateroles.md

io-package.json schema reference: https://github.com/ioBroker/ioBroker.js-controller/blob/master/schemas/io-package.json

> ⚠️ **TOOL WARNING — stateroles.md:** The WebFetch tool returns a summarised/cached version of stateroles.md that omits roles and produces false findings. **Always fetch stateroles.md using `curl`:**
> ```bash
> curl -sS "https://raw.githubusercontent.com/ioBroker/ioBroker.docs/master/docs/en/dev/stateroles.md"
> ```
> Never use WebFetch for this URL. If you need to search for a specific role, pipe through `grep`:
> ```bash
> curl -sS "https://raw.githubusercontent.com/ioBroker/ioBroker.docs/master/docs/en/dev/stateroles.md" | grep "value\.rssi"
> ```

---

## Object Hierarchy Rules

Two violations to flag as 🔴 errors:
1. `channel` → `device` ❌ (device nested under channel)
2. Any object nested under `state` ❌ (state must never have children)

| Parent | Allowed children |
|---|---|
| `folder` | `folder` ✅ `device` ✅ `channel` ✅ `state` ✅ |
| `device` | `folder` ✅ `channel` ✅ `state` ✅ |
| `channel` | `folder` ✅ `channel` ✅ `state` ✅ |
| `state` | **none** — having any child object is always a 🔴 error |

Python validation pattern:
```python
if otype == 'device' and parent_type == 'channel':
    flag_violation()   # error: device nested under channel
if parent_type == 'state':
    flag_violation()   # error: state must never have children
if otype == 'folder':
    pass               # always valid under any parent
```

---

## Role / Attribute Rules

| Role(s) | write | read | type |
|---|---|---|---|
| `indicator`, `indicator.*` | MUST be `false` | MUST be `true` | `boolean` |
| `value`, `value.*` | MUST be `false` | — | `number` |
| `sensor`, `sensor.*` | MUST be `false` | — | `boolean` |
| `switch`, `switch.*` | MUST be `true` | MUST be `true` | `boolean` |
| `level`, `level.*` | MUST be `true` | MUST be `true` | `number` |
| `button` | MUST be `true` | MUST be `false` | `boolean` |
| `text`, `text.url`, `html`, `json`, `list` | either | either | — |
| `date` | either | — | `string` (ISO) or `number` (epoch ms) |
| `info.*` | should be `false` | — | `string` |

**IMPORTANT exceptions to the `level.*` type rule:**
- `level.color.rgb` — `common.type=string` (hex color like `#rrggbb`). Explicitly listed in the roles reference as string. Do NOT flag as type mismatch.
- `level.color.rgbw` — `common.type=string`. Same as above.
- `level.color.cie` — `common.type=string`. Same as above.

Do NOT flag `text`-role states for having `write: true` — it is explicitly permitted.

**`state` role:** Explicitly valid as a generic fallback. The reference states: "If you don't know which role the state has, use this one." Do NOT flag `state` as an error. It may be flagged as 🟡 recommendation to use a more specific role when one clearly applies (e.g. numeric+writable → `level`), but never 🔴.

---

## Role Validity Rules

**Only roles explicitly listed in the state roles reference are valid.** Sub-roles (e.g. `value.xyz`) are only valid if:
1. They are explicitly listed in the reference document, OR
2. The reference document explicitly states that `*` wildcards are permitted for that role family AND the parent family documents wildcard support.

**Known role families with documented wildcard sub-roles:**
- `level.mode.*` — documented wildcard
- `switch.mode.*` — documented wildcard
- `indicator.*` — documented wildcard (all share the same constraints)

**Role families that do NOT have a wildcard:**
- `value.*` — Each sub-role must be explicitly listed. Do NOT invent sub-roles like `value.foo`, `value.co`, etc. unless they appear in the reference document. If no listed `value.*` sub-role fits, use `value`. Note: `value.rssi` IS listed in the reference (line 139) — do NOT flag it.
- `sensor.*` — Each sub-role must be explicitly listed.

**Consequence for reviews:** If an adapter uses a role like `value.foo` etc. that does not appear in the reference document, flag it as 🔴 (invalid role) and suggest the nearest valid listed role (usually the parent, e.g. `value`). Always verify against the live document using `curl` before flagging — the WebFetch tool is unreliable for this URL.

---

## licenseInformation Rules

- `"type": "free"` — `link` field is **NOT required**. Do not flag its absence.
- `"type": other` — `link` field should point to the license text; flag if missing.

---

## io-package.json Field Casing Rules

Always verify field names against the official schema at:
https://github.com/ioBroker/ioBroker.js-controller/blob/master/schemas/io-package.json

Known case-sensitive field names (verified against schema):
- `"nogit"` — **all lowercase**. This is the correct and only valid spelling per the schema (`common.nogit`). Do NOT flag it as wrong or suggest `"noGit"` as a correction.

---

## Validation Script Template

```python
WRITE_MUST_BE_FALSE = {'indicator', 'value', 'sensor'}   # and subtypes
WRITE_MUST_BE_TRUE  = {'switch', 'level'}                 # and subtypes
# text / html / json / list / date — no write restriction
# button: write=true AND read=false
# level.color.rgb / level.color.rgbw / level.color.cie — type=string (exception to level.* type=number)
```

---

## Translation Completeness Rules

**Checking that all language keys are present is NOT sufficient.** A translation is only valid if its value is actually translated into the target language — not a copy of the English text.

### Required check for every i18n object and every translated field in admin jsonConfig, io-package.json, and source files:

For every language-keyed object (e.g. `{ "en": "...", "de": "...", "ru": "...", ... }`):

1. **Keys check** — All 11 required keys must be present: `en`, `de`, `ru`, `pt`, `nl`, `fr`, `it`, `es`, `pl`, `uk`, `zh-cn`. Flag any that are missing.

2. **Value check (untranslated copy detection)** — For every non-English language key, check that its value is NOT identical to the `en` value. If a non-English language has the same string as `en`, that entry is **untranslated** and must be flagged as an error — even if the key exists. This is the most commonly missed check.

   Example of a FAILING entry (must be flagged):
   ```json
   {
     "en": "Use at your own risk",
     "de": "Nutzung auf eigene Gefahr",
     "ru": "Use at your own risk",   ← ERROR: untranslated copy of English
     "pt": "Use at your own risk",   ← ERROR: untranslated copy of English
     "nl": "Use at your own risk",   ← ERROR: untranslated copy of English
     "pl": "Use at your own risk",   ← ERROR: untranslated copy of English
     "uk": "Use at your own risk",   ← ERROR: untranslated copy of English
     "zh-cn": "Use at your own risk" ← ERROR: untranslated copy of English
   }
   ```

3. **Scope** — Apply this check to ALL of the following:
   - `admin/jsonConfig.json` — every label, text, header, staticText, and any other translatable field
   - `io-package.json` — `titleLang`, `desc`, `news` entries, and any other translated field
   - i18n JSON files (e.g. `admin/i18n/*.json`)
   - Inline translation objects in source code (`.js`, `.ts`)

4. **Reporting** — Group all untranslated-copy findings into one report entry per source file. List the affected field paths and languages. Up to 5 examples per finding.

### Python pattern for validation:

```python
def check_translations(obj, path=''):
    required_langs = {'en','de','ru','pt','nl','fr','it','es','pl','uk','zh-cn'}
    if isinstance(obj, dict) and set(obj.keys()) & required_langs:
        en_value = obj.get('en', '')
        missing = required_langs - set(obj.keys())
        if missing:
            flag(f'{path}: missing language keys: {sorted(missing)}')
        for lang, value in obj.items():
            if lang != 'en' and lang in required_langs and value == en_value:
                flag(f'{path}.{lang}: untranslated (identical to English)')
    elif isinstance(obj, dict):
        for k, v in obj.items():
            check_translations(v, f'{path}.{k}')
    elif isinstance(obj, list):
        for i, v in enumerate(obj):
            check_translations(v, f'{path}[{i}]')
```

---

## adminTab Reporting Rule

**`adminTab` is only a concern when it IS present in the adapter.** If no `adminTab` is implemented, there is nothing to check — mark it as passed (`- [x]`) and move on.

**When `adminTab` IS present:** Flag it as `- [ ] ⚠️` and ask the developer to confirm its purpose and that it provides clear benefit over the standard jsonConfig UI.

**When `adminTab` is NOT present:** Report `- [x] adminTab: Not present — nothing to check ✓`. Do NOT add any warning.

**Wrong behavior:** Reporting a warning or `⚠️` when no adminTab exists.
