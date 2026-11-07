# ioBroker Adapter Checklist V3.5

This file is the authoritative check specification. Work through every item.
Record each result for the report.

**Reference documents** (always fetch with `curl` — see warnings below):
- State roles: `https://raw.githubusercontent.com/ioBroker/ioBroker.docs/master/docs/en/dev/stateroles.md`
- io-package.json schema: `https://github.com/ioBroker/ioBroker.js-controller/blob/master/schemas/io-package.json`

> ⚠️ **TOOL WARNING — stateroles.md:** The WebFetch tool returns a summarised/cached version that
> omits roles and produces false findings. **Always fetch it with `curl`:**
> ```bash
> curl -sS "https://raw.githubusercontent.com/ioBroker/ioBroker.docs/master/docs/en/dev/stateroles.md"
> ```
> To search for a specific role, pipe through `grep`:
> ```bash
> curl -sS "https://raw.githubusercontent.com/ioBroker/ioBroker.docs/master/docs/en/dev/stateroles.md" | grep "value\.rssi"
> ```
> Never use WebFetch for this URL.

---

## Documentation & README

- README must be in pure English. A `README.de.md` / `README_de.md` for German is acceptable as an
  addition, but the primary README must be English only.
- If the adapter targets a specific device or commercial service, README must contain a link to that
  product or manufacturer. (Not required for general-purpose adapters such as widget-only, mail
  sending, or generic AI interfaces.)
- README must not contain instructions to install the adapter directly from a GitHub repository or
  via raw `npm install` commands.
- README must include the full license text or a link to the `LICENSE` file.
- Check all documentation files (all languages) for any hint that a paid license is required for
  full functionality.

---

## Icon

- Adapter must have a unique icon. It must not use the create-adapter creator icon at
  `https://github.com/ioBroker/create-adapter/blob/master/adapter-creator.png`.
- Verify that the actual file format of the icon matches its file extension (e.g., a file named
  `.png` that is actually JPEG must be flagged).

---

## Metadata — `io-package.json` and `package.json`

- `common.type` must match the adapter's actual functionality. Flag for manual review if uncertain.
- `common.connectionType` must match the actual connection implementation. Flag for manual review
  if uncertain.
- `package.json` must specify `"engines": { "node": ">=22.0.0" }`. Do not suggest lowering this
  requirement.
- `package.json` must not contain scripts executed during `npm install` (e.g., `preinstall`,
  `postinstall`, `install` hooks that do real work). Log any such scripts and request an explanation.
- Authors' email addresses in `package.json`, `io-package.json`, copyright lines in `README.md` and
  `LICENSE` must be present and valid. Reject one-time-use addresses and `noreply` addresses such
  as `xxxx@users.noreply.github.com`.
- Warn if the author/maintainer name is not identical across `package.json`, `io-package.json`,
  `README.md` copyright line, and `LICENSE`.
- License should be MIT. If not, flag it for manual review.

### licenseInformation Rules

- `"type": "free"` — the `link` field is **not required**. Do not flag its absence.
- `"type": <anything else>` — the `link` field should point to the license text; flag if missing.

### Field Casing Rules

Always verify field names against the official io-package.json schema. Known case-sensitive field:
- `"nogit"` — **all lowercase**. This is the correct spelling per the schema (`common.nogit`).
  Do NOT flag it as wrong or suggest `"noGit"`.

---

## Object IDs & Object Structure

- Object IDs constructed from external data or user input must be filtered. At minimum, characters
  defined by ioBroker constant `FORBIDDEN_CHARS` must be removed. Spaces should be removed (or
  converted to `_`).
  - If filtering uses only `FORBIDDEN_CHARS` (not restricted to `a-zA-Z0-9_-`), add a
    recommendation in **Observations** suggesting stricter sanitization. This is non-blocking.
- All object IDs must use English wording unless the value is received verbatim from an external
  source. Check both source code and the object dump (if attached). This covers state IDs, folder
  IDs, config parameter names, and all other object paths.
- All `common.name` and `common.desc` values must use English or implement i18n multilanguage.
  Exception: `common.doc` — `de` and `en` keys are sufficient.
- `common.titleLang` — identical text across all language keys is **acceptable** (brand/product
  names are the same in every language). If only a few languages differ from the rest, do not flag
  it as an error; report it as an **Observation** only:
  > Either translate all entries or align all to the same brand-name value.
- Object IDs must be unique case-insensitively. Two IDs that differ only in casing (e.g. `myRoom`
  and `myroom`) must be flagged as an error — this causes unpredictable behavior across platforms.
- Adapters must not create states or objects outside their own namespace. Flag as error.
- Adapters must not modify `system.adapter.*` states belonging to another adapter or global
  `system.*` states. Reading is acceptable.
- Warn (do not error) if the adapter modifies its own `system.adapter.<own-name>.*` states and ask
  the developer to explain the purpose.

### Object Hierarchy Rules

Two violations must be flagged as 🔴 errors:
1. `device` nested under `channel` ❌
2. Any object nested under `state` ❌ (state must never have children)

| Parent   | Allowed children                                       |
|----------|--------------------------------------------------------|
| `folder` | `folder` ✅  `device` ✅  `channel` ✅  `state` ✅   |
| `device` | `folder` ✅  `channel` ✅  `state` ✅                 |
| `channel`| `folder` ✅  `channel` ✅  `state` ✅                 |
| `state`  | **none** — any child object is always a 🔴 error       |

```python
if otype == 'device' and parent_type == 'channel':
    flag_violation()   # error: device nested under channel
if parent_type == 'state':
    flag_violation()   # error: state must never have children
if otype == 'folder':
    pass               # always valid under any parent
```

- If configuration or external data may cause additional dynamic objects to be created at runtime
  that are absent from the dump, evaluate those objects' parents, types, attributes, and roles from
  the code.

---

## State Roles & Attributes

Validate all state roles against the official reference — always fetch with `curl` (see warning at
the top). Only roles explicitly listed there are valid.

### Role Validity Rules

Sub-roles (e.g. `value.xyz`) are only valid if:
1. They are explicitly listed in the reference document, OR
2. The reference explicitly states that `*` wildcards are permitted for that role family.

**Role families with documented wildcard sub-roles:**
- `level.mode.*` — documented wildcard
- `switch.mode.*` — documented wildcard
- `indicator.*` — documented wildcard

**Role families that do NOT have wildcards (`value.*`, `sensor.*`):**
Each sub-role must be explicitly listed. Do not accept invented sub-roles such as `value.foo` or
`value.co` unless they appear in the reference document.
- `value.rssi` IS listed in the reference — do NOT flag it.

If an adapter uses an unlisted sub-role, flag it as 🔴 and suggest the nearest valid listed role
(usually the parent, e.g. `value`). Always verify against the live document via `curl` before
flagging.

**`state` role:** Explicitly valid as a generic fallback. The reference states: "If you don't know
which role the state has, use this one." Do NOT flag `state` as 🔴. It may be flagged as a 🟡
recommendation to use a more specific role when one clearly applies (e.g. numeric+writable →
`level`), but never as an error.

### Role / Attribute Constraints

| Role(s)                        | write         | read          | type                        |
|-------------------------------|---------------|---------------|-----------------------------|
| `indicator`, `indicator.*`    | MUST be false | MUST be true  | `boolean`                   |
| `value`, `value.*`            | MUST be false | —             | `number`                    |
| `sensor`, `sensor.*`          | MUST be false | —             | `boolean`                   |
| `switch`, `switch.*`          | MUST be true  | MUST be true  | `boolean`                   |
| `level`, `level.*`            | MUST be true  | MUST be true  | `number` (see exceptions ↓) |
| `button`                      | MUST be true  | MUST be false | `boolean`                   |
| `text`, `text.url`, `html`, `json`, `list` | either | either | —              |
| `date`                        | either        | —             | `string` (ISO) or `number` (epoch ms) |
| `info.*`                      | should be false | —           | `string`                    |

**`level.color.*` type exceptions** — these are `string`, not `number`:
- `level.color.rgb` → `common.type = string` (hex color like `#rrggbb`)
- `level.color.rgbw` → `common.type = string`
- `level.color.cie` → `common.type = string`

Do NOT flag `text`-role states for having `write: true` — it is explicitly permitted.

```python
WRITE_MUST_BE_FALSE = {'indicator', 'value', 'sensor'}   # and subtypes
WRITE_MUST_BE_TRUE  = {'switch', 'level'}                 # and subtypes
# text / html / json / list / date — no write restriction
# button: write=true AND read=false
# level.color.rgb / level.color.rgbw / level.color.cie — type=string (exception to level.* type=number)
```

- Verify that roles retrieved from external sources are properly validated before use.
- Check object attributes and roles as far as possible by inspecting code — do not rely on the
  object dump as the only source; it might be manipulated.

---

## Language & Translations

- Log messages must be in English and must not use any translation mechanism.
- All labels, constants, and text in web pages and admin UI must use pure English or support
  translation into all supported ioBroker languages:
  `en`, `de`, `ru`, `pt`, `nl`, `fr`, `it`, `es`, `pl`, `uk`, `zh-cn`
- All text output to users or stored in states must be English or support i18n with at least English
  and German. Even text that is intentionally non-English must be flagged as an error.
- In general, no non-English text is allowed except inside i18n/translation objects and pure source
  code comments. German comments in source code are acceptable and must NOT be flagged.
- Check for unused translation keys within i18n files.

### Translation Completeness — Two-Step Check

**Checking that all language keys are present is NOT sufficient.** A translation is only valid if
its value is actually translated — not a copy of the English text.

For every language-keyed object (e.g. `{ "en": "...", "de": "...", "ru": "...", ... }`):

**Step 1 — Keys check:** All 11 required keys must be present:
`en`, `de`, `ru`, `pt`, `nl`, `fr`, `it`, `es`, `pl`, `uk`, `zh-cn`. Flag any that are missing.

**Step 2 — Untranslated copy detection:** For every non-English language key, check that its value
is NOT identical to the `en` value. If a non-English language has the same string as `en`, that
entry is **untranslated** and must be flagged as an error — even if the key exists.

Example of a failing entry (must be flagged):
```json
{
  "en": "Use at your own risk",
  "de": "Nutzung auf eigene Gefahr",
  "ru": "Use at your own risk",    ← ERROR: untranslated copy of English
  "pt": "Use at your own risk",    ← ERROR: untranslated copy of English
  "nl": "Use at your own risk",    ← ERROR: untranslated copy of English
  "pl": "Use at your own risk",    ← ERROR: untranslated copy of English
  "uk": "Use at your own risk",    ← ERROR: untranslated copy of English
  "zh-cn": "Use at your own risk"  ← ERROR: untranslated copy of English
}
```

**Apply to all of:**
- `admin/jsonConfig.json` — every label, text, header, staticText, and other translatable fields
- `io-package.json` — `titleLang`, `desc`, `news` entries, and any other translated field
- i18n JSON files (e.g. `admin/i18n/*.json`)
- Inline translation objects in source code (`.js`, `.ts`)

**Reporting:** Group all untranslated-copy findings into one report entry per source file. List
the affected field paths and languages. Up to 5 examples per finding.

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

## Admin UI

- If the adapter does not use `jsonConfig` for `adminUI`, `custom`, or `adminTab`, verify that it
  supports all ioBroker languages and implements translation properly.
- The admin UI must use the system-wide language configured in admin — no private adapter-specific
  language switching.
- If `jsonConfig` is not used, note this fact. If no feature requiring a custom UI is present,
  suggest migrating to `jsonConfig`.
- If `adminUI` uses a `customComponent`, `globalDependencies` in `io-package.json` must require
  `admin >= 8.0.0`.

### adminTab Reporting Rule

- **`adminTab` is NOT present:** Report `- [x] adminTab: Not present — nothing to check ✓`. Do
  NOT add any warning or ⚠️.
- **`adminTab` IS present:** Flag as `- [ ] ⚠️ adminTab present` and ask the developer to confirm
  its purpose and that it provides clear benefit over the standard jsonConfig UI. Also check:
  - The tab's visible name must match or contain the adapter name.
  - The tab must handle multiple adapter instances if the adapter allows multiple instances.
  - If the tab only contains a standard `devicemanager` page, suggest removing it or using the
    `jsonConfig` `devicemanager` component.

---

## Source Code Structure

- **Plain JavaScript adapters**: Sources must be in the repository root (`main.js`) and under
  `/lib`. Sources within `/build` are not allowed.
- **TypeScript adapters**:
  - Sources in `/src`, generated files in `/build`.
  - `package.json` `main` field must point to the generated file in `/build`, not a stub that
    delegates to the real module.
- Unused event handlers (`onStateChange`, `onObjectChange`) must be removed.

---

## State Change Handling

- All writable states must be subscribed and handled in the `onStateChange` routine. Writable states
  that are never processed are an error.
- `onStateChange` must honor the `ack` flag:
  - For states owned by this adapter instance: ignore the event if `ack === true`.
  - For foreign states (output of another adapter): process only if `ack === true`.
  - The role `text` allows both `read` and `write` to be `true` — this is not an error.

---

## Networking & HTTP

- All `axios` (or other HTTP/HTTPS library) calls must specify a `timeout`.
- Scheduled adapters that access external systems (outside the local LAN, typically cloud services)
  should include a small random delay to avoid synchronized stampede. A fixed delay calculated at
  startup is acceptable.
  - Applies to all schedule types (`setInterval`/`cron` at specific timestamps).
  - Daemon adapters using `setInterval`/`setTimeout` loops do not need a random delay.
  - Do not suggest a random delay if the adapter starts processing immediately after startup.
- Cloud service polling intervals must not be more frequent than every 10 seconds.
- Access to cloud services must not occur at fixed timestamps (e.g., every full hour, 5 minutes
  after midnight).

---

## Timers & Intervals

- Configurable timeouts/intervals must be validated in code: they must be positive, non-zero, and
  no greater than `2,147,483,647` ms (Node.js max for `setTimeout`/`setInterval`). UI-level
  validation alone is insufficient.
- If a processing loop is driven by `setInterval`, the loop's execution time (sum of all timeouts
  within it) must be less than the minimum interval. Suggest using `setTimeout` at the end of the
  loop instead of `setInterval`.

---

## Sensitive Data

- Passwords, tokens, and other sensitive data (fields listed in `protectedNative` or
  `encryptedNative` in `io-package.json`, plus any data obviously used for authentication) must
  never be written to log files.

---

## Security & Integrity

- Code must not modify files within npm-installed package directories.
- Code must not shell out to execute scripts in a subprocess. Log a warning if it does.
- Code must not use any method to modify the adapter's own code tree or run `npm` commands during
  runtime. Flag as error.
- Code must not download executable code at runtime. Flag as error.
- Scan all scripts/code for attempts to install packages directly from GitHub, manipulate ioBroker
  adapter directories (outside `iobroker-data`), or use `sudo`. Flag as errors.
- Check documentation for any instructions suggesting additional scripts, manual `npm` commands, or
  `sudo` to install, modify, or repair the adapter. Flag as errors.

---

## Code Quality

- Check for dead code and unused config parameters.
- If `json2iob` is used, verify the adapter meets all requirements from
  `https://github.com/TA2k/json2iob` and does not bypass them via the external package. Skip this
  check if `json2iob` is not used.

---

## Object Dump Validation (only if a dump is attached to the session)

- Verify that the dump's object structure belongs to this adapter and still matches the current
  code.
- Log the approximate creation date by finding the newest `ts` timestamp in the dump.
- If the dump appears to be from a previous adapter version, log an error requesting a fresh dump.

---

## Pass 2: Dynamic Object Validation

This pass runs **after** Pass 1 and focuses specifically on objects the adapter creates at runtime
from configuration or external data — objects that may not appear in the object dump at all.

> ⚠️ Do not use WebFetch for any URL in this pass. Use `curl` for all remote reads (including
> stateroles.md). WebFetch returns invalid, truncated or outdated data.

### What to analyse

Scan all code paths where objects are created dynamically — typically driven by:
- Adapter configuration values (`this.config.*`)
- Data received from external systems (API responses, device discovery, etc.)
- User-defined channel/device lists

For every dynamically created object, verify:

**1. Object hierarchy — no violations allowed**
- The order `device → channel → state` must not be violated.
- `device` must never be nested under `channel`. Flag as 🔴 error.
- A `state` object must never have child objects — including dynamically created ones. Flag as 🔴.
- `folder` objects are valid under any parent.
- Verify that all parent objects in the path are explicitly created before the child. Missing parent
  creation is an error.

**2. Attributes and roles**
- All attributes (`common.type`, `common.role`, `common.read`, `common.write`, etc.) must be valid.
- Roles must be validated against the live stateroles.md reference (fetch with `curl` — see warning
  at top of file). The same role/attribute constraint table from the main checklist applies here.

**3. State objects must never have children**
- Check that no code path creates a child object under a `state`-type object, even as a side effect
  of dynamic object creation loops. Flag any such construct as 🔴 error.

**4. Case-collision in dynamically generated IDs**
- If object IDs are built from external data (device names, room names, API keys, etc.), evaluate
  whether the sanitization/normalization step prevents two IDs that differ only in casing from
  being created (e.g. `myRoom` and `myroom`). If the code does not normalize casing, flag as 🔴.

Pass 2 findings feed into the final consolidated report generated after Pass 3. Do not produce
any intermediate report output after Pass 2.

---

## Pass 3: Language & Naming Checks

This pass runs **after** Pass 2. It performs a focused audit of language usage across object IDs,
code logic, documentation, and UI components.

> ⚠️ Do not use WebFetch for any URL in this pass. Use `curl` for all remote reads.

### Object ID language

- All object IDs must use English naming. This includes:
  - States in the state tree
  - `config.*` objects and all native configuration fields
  - Channel and device IDs derived from configuration or external data
- If object IDs are built from user-supplied or external data, verify that the code does not
  preserve or embed non-English words in the resulting ID path.

### Object name attributes

- All `common.name` values must be in English or use an i18n translation object / i18n setup in
  code. This applies to all object types (state, channel, device, folder).

### Non-English string literals in code logic

This is a subtle but important check. Scan for non-English string literals used anywhere in code
logic that **determines or influences** object IDs, object names, roles, or types. This includes:

- String-matching heuristics (e.g. checking whether an MQTT topic contains the German word
  `"Schalter"` to assign a role)
- Keyword lists or pattern arrays containing non-English words whose result is used to derive an
  object ID, role name, or type
- Path-segment checks using non-English terms
- Any conditional logic that selects an object property based on a non-English string

Such literals must be flagged as 🔴 errors even if the non-English string is never stored directly
as an object ID. The rule is: **only English keywords are permitted in object-property heuristics.**

### File names

- Files written to the filesystem that are intended to be processed or read by users must use
  English filenames.

### Vis components

- Any widgets or components intended for use in ioBroker vis must use English text or be
  translated via the i18n system.

### Web pages

- Any web page provided or served by the adapter must use English or support translation via the
  i18n system.

### Template data

- Any template data provided to users must use English or support the i18n system.

### Secondary documentation

- All secondary documentation files (beyond `README.md`) must exist in English.
- Additional languages are acceptable and welcome.
- Full language coverage is not required, but an English version is mandatory.

### Reporting note

A check that passes must be recorded as a passed check (`- [x] …`) — not as an observation.
Pass 3 findings feed into the final consolidated report together with Pass 1 and Pass 2.
