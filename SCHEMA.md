# Pearls of Persian: Repository Schema Specification

---

## Part 1: Content Directory Metadata Specification (`meta.yml`)

### 1. Overview
Every directory under the `content/` path is required to contain a `meta.yml` file. This file serves as the single source of truth for:
1. Describing the directory's display metadata (such as `type` and `title`).
2. Providing data for the presentation layer (such as `description`).
3. Defining the explicit, canonical ordering and visibility of its immediate children.

This specification enables deterministic navigation, selective content rendering, and structured display in the presentation layer.

---

### 2. File Location & Directory Consistency
Every directory under `content/` must contain a `meta.yml` file. Additionally, a directory must satisfy a strict structural exclusivity rule. It must contain:
* **Either** only subdirectories,
* **Or** only Markdown documents (`*.md`),
* **Never both** within the same directory level (excluding the `meta.yml` file itself).

```text
content/
├── meta.yml                           # Root metadata
├── عطار-نیشابوری/
│   ├── meta.yml                       # Poet metadata
│   └── منطق-الطیر/
│       ├── meta.yml                   # Book metadata
│       ├── مقدمه/
│       │   ├── meta.yml               # Section metadata
│       │   ├── مجمع-مرغان.md
│       │   └── خطاب-هدهد.md
│       └── هفت-وادی/
│           ├── meta.yml               # Section metadata
│           ├── وادی-طلب.md
│           └── وادی-عشق.md
```

---

### 3. Schema Structure

#### Required Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `schema` | String | Schema version number in SemVer format. | `"1.0.0"` |
| `type` | String | Semantic category of this directory level | `"poet"` |
| `title` | String | Human-readable title for display | `"عطار نیشابوری"` |
| `children` | Array of Strings | Ordered list of visible child item names | `["منطق-الطیر", "الهی-نامه"]` |

#### Optional Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `description` | String | Short description shown on directory pages | `"مجموعه‌ای از آثار عرفانی"` |
| `tags` | Array of Strings | Directory-level metadata tags | `["عرفان", "قرن-ششم"]` |
| `notes` | String | Internal editorial notes (not displayed publicly) | `"ترتیب فصول نیاز به بازبینی دارد"` |

*Note: Automated Continuous Integration (CI) systems must ignore any unknown keys to ensure forward compatibility.*

---

### 4. Field Specifications

#### 4.1 `schema`
* **Type:** String  
* **Required:** Yes  
* **Purpose:** Identifies the metadata schema version in SemVer format.
* **Value:** `"1.0.0"`

#### 4.2 `type`
* **Type:** String  
* **Required:** Yes  
* **Purpose:** Categorizes the directory structure to guide rendering and layout templates.

The following standardized, controlled vocabulary is defined for the `type` field:

| Type | Description |
|------|-------------|
| `root` | The repository root (only applicable to `content/`) |
| `poet` | A poet's complete works and profile |
| `book` | A distinct literary work or publication |
| `office` | A major volume or daftar (دفتر) |
| `chapter` | A chapter subdivision |
| `section` | A section division |
| `genre` | A collection organized by poetic style (e.g., غزلیات، رباعیات، قطعات) |
| `collection` | A generic container utilized when no canonical classification is suitable |

#### 4.3 `title`
* **Type:** String  
* **Required:** Yes  
* **Purpose:** Provides the human-readable title for breadcrumbs, navigation menus, and index headings. This value may differ from the physical directory name.

#### 4.4 `children`
* **Type:** Array of Strings  
* **Required:** Yes  
* **Purpose:** Establishes the explicit, ordered sequence of immediate child items. It also serves as a visibility filter.

**Format:**
```yaml
children:
  - <child-directory-name>
  - <child-directory-name>
```
Or for document-level directories:
```yaml
children:
  - <child-file-name.md>
  - <child-file-name.md>
```

**Rules:**
1. **No Mixing:** The array must list exclusively directory names or exclusively Markdown filenames (ending in `.md`). A combination of both in a single `children` list is invalid.
2. **Exact Matching:** Each entry must match the filesystem name exactly.
3. **Selective Visibility (Filtering):** Only the filesystem items explicitly listed in the `children` array are exposed to the presentation layer. If a subdirectory or Markdown file physically exists in the directory but is omitted from `children`, it is excluded from rendering and navigation.
4. **No Self-Reference:** The `meta.yml` file itself must not be included.
5. **No Duplicates:** Duplicate entries are prohibited. In the event of a duplication, the first occurrence takes precedence, and a compilation warning should be logged.

#### 4.5 `description`
* **Type:** String (multi-line supported)  
* **Required:** No  
* **Purpose:** Provides a short introduction or summary displayed on section index pages.

#### 4.6 `tags`
* **Type:** Array of Strings  
* **Required:** No  
* **Purpose:** Categorizes the directory using labels or keywords.

#### 4.7 `notes`
* **Type:** String (multi-line supported)  
* **Required:** No  
* **Purpose:** Holds internal editorial metadata. This field is ignored by the public presentation layer.

---

### 5. Directory Metadata Examples

#### Example 1: Root Level (`content/meta.yml`)
```yaml
schema: "1.0.0"
type: "root"
title: "گوهرهای پارسی با محسن"
description: >
  مجموعه‌ای از شعر کلاسیک فارسی با تحقیق و تفسیر
tags:
  - "شعر-کلاسیک"
  - "ادبیات-فارسی"
children:
  - عطار-نیشابوری
  - مولوی
  - سعدی
```

**Note:** The `meta.yml:title` is actually the site title.

#### Example 2: Poet Level (`content/عطار-نیشابوری/meta.yml`)
```yaml
schema: "1.0.0"
type: "poet"
title: "عطار نیشابوری"
description: >
  فریدالدین ابوحامد محمد عطار نیشابوری (متوفی حدود 618 ه.ق.) از بزرگترین
  شاعران و عارفان ایران است.
tags:
  - "قرن-ششم"
  - "عرفان"
children:
  - منطق-الطیر
  - الهی-نامه
```

#### Example 3: Book Level (`content/عطار-نیشابوری/منطق-الطیر/meta.yml`)
```yaml
schema: "1.0.0"
type: "book"
title: "منطق‌الطیر"
description: >
  داستان سفر پرندگان به سوی سیمرغ به زبان شعر.
tags:
  - "مثنوی"
children:
  - مقدمه
  - هفت-وادی
```

#### Example 4: Section Level (`content/عطار-نیشابوری/منطق-الطیر/مقدمه/meta.yml`)
```yaml
schema: "1.0.0"
type: "section"
title: "مقدمه"
description: "در آغاز کتاب و مجمع مرغان"
children:
  - مجمع-مرغان.md
  - خطاب-هدهد.md
  - سخن-هدهد.md
```

#### Example 5: Genre Level (`content/مولوی/دیوان-شمس/غزلیات/meta.yml`)
```yaml
schema: "1.0.0"
type: "genre"
title: "غزلیات"
description: "غزلیات دیوان شمس تبریزی"
tags:
  - "غزل"
  - "شور-و-وجد"
children:
  - غزل-10.md
  - غزل-256.md
```

#### Example 6: Office Level (`content/مولوی/مثنوی-معنوی/دفتر-اول/meta.yml`)
```yaml
schema: "1.0.0"
type: "office"
title: "دفتر اول"
description: "دفتر نخست مثنوی معنوی"
children:
  - نی-نامه.md
  - پادشاه-و-کنیزک.md
```

---

### 6. Directory Validation Rules (CI/CD Checklist)

A directory and its corresponding `meta.yml` file are considered valid if they satisfy the following automated criteria:

#### Required Field Validation
* The `meta.yml` file must exist and parse as valid YAML.
* The `schema` value must be a valid SemVer string.
* The `type` and `title` fields must be present and contain non-empty strings.
* The `children` field must be present and formatted as an array of strings.

#### Directory Consistency Validation
* The target directory must contain either exclusively subdirectories or exclusively Markdown documents (`*.md`), with the sole exception of the `meta.yml` file itself.
* The `children` array must not contain a mix of directories and `.md` file references.

#### Integrity & Visibility Validation
* Every name declared in the `children` array must exist in the filesystem as an immediate child of the directory.
* Filesystem items that exist on disk but are omitted from the `children` array are permitted; they are quietly ignored by the presentation layer and should not trigger validation errors or warnings.
* The `children` list must contain no duplicates.
* The `meta.yml` file itself must not be included in the `children` list.

#### Extensibility & Optional Fields
* If present, `description` must be a string, `tags` must be an array of strings, and `notes` must be a string.
* **Forward Compatibility:** Continuous Integration (CI) and build scripts must ignore any unrecognized or custom keys in `meta.yml` without failing the validation process.

---

## Part 2: Contributor Profile Specification

### 1. Overview
Every contributor participating in the repository must be represented by a single YAML file located in the `contributors/` directory.

```text
contributors/
├── mm91mm92.yml
├── alice-johnson.yml
└── bob-smith.yml
```

#### File Location and Identity
* **Path:** `contributors/<id>.yml`
* The filename `<id>` (excluding the `.yml` extension) serves as the unique identifier for the contributor. This `<id>` is the value referenced in the `contributors` metadata array within literary documents.
* The file serves as the single source of truth for the contributor's identity, contact info, and professional profile.

---

### 2. Schema Structure

#### Required Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `schema` | String | Identifies the metadata schema version in SemVer format. | `"1.0.0"` |
| `name` | String | Full name in Latin script (used for international search and indexing). | `"Mohsen Malekmohammadi"` |
| `name_fa` | String | Full name in Persian script (used for the primary presentation layer). | `"محسن ملک‌محمدی"` |
| `email` | String | Contact email address. | `"mm91mm92@gmail.com"` |

#### Optional Fields

| Field             | Type             | Description                                                     | Example                               |
| ----------------- | ---------------- | --------------------------------------------------------------- | ------------------------------------- |
| `website`         | String (URL)     | Personal website or academic profile URL.                       | `"https://example.com"`               |
| `location`        | String           | Geographic location (City, Country).                            | `"Tehran, Iran"`                      |
| `affiliation`     | String           | Institutional or organizational affiliation in English.         | `"Payam-e Noor University"`           |
| `affiliation_fa`  | String           | Institutional or organizational affiliation in Persian.         | `"دانشگاه پیام‌نور"`                  |
| `bio`             | String           | Brief professional biography in English (multi-line supported). | Multi-line text block                 |
| `bio_fa`          | String           | Brief professional biography in Persian (multi-line supported). | Multi-line text block                 |
| `preferred_roles` | Array of Strings | List of common roles typically performed by this contributor.   | `["محقق", "نگه‌دارنده"]`              |
| `links`           | Object (Map)     | Flexible dictionary of external platform handles or URLs.       | See [[#2.1 The `links` Object]] below |

#### 2.1 The `links` Object
To maintain a highly extensible and clean structure, external identifiers, social profiles, and developer accounts are grouped inside the `links` object. 

The `links` field is formatted as a flat dictionary of key-value pairs:
```yaml
links:
  <platform>: <string>
```

##### Guidelines
* Common platform keys include: `github`, `orcid`, `linkedin`, `telegram`, `x` (formerly Twitter), etc.
* The value of each platform is flexible. To simplify authoring and accommodate diverse preferences, a value can be represented as anything from a bare username/ID to a full-fledged URL.

**Examples:**
```yaml
links:
  github: "https://github.com/mm91mm92"
  linkedin: "https://linkedin.com/in/mm91mm92"
  orcid: "0000-0002-1234-5678"
  telegram: "https://t.me/example"
```

---

### 3. Field Format & Validation Rules

#### Required Fields
* **`schema`:** Must be a valid SemVer string.
* **`name` / `name_fa`:** Must contain non-empty strings. Professional or legal names with proper capitalization should be used. The Persian name must be written in the Persian script.
* **`email`:** Must conform to standard email syntax rules (`username@domain.tld`) and represent an active, monitored address.

#### Optional Fields
* **`website`:** If present, must be a fully qualified URL beginning with `http://` or `https://`.
* **`bio` / `bio_fa`:** Multi-line strings are recommended to be written using the folded block scalar (`>`) for clean formatting.
* **`links` values:** The values within the `links` object are evaluated as free-form strings. No strict structural validation is enforced on these values, allowing either bare identifiers or complete URLs.

---

### 4. Contributor Profile Examples

#### Minimal Profile Example
```yaml
schema: "1.0.0"
name: Alice Johnson
name_fa: "آلیس جانسون"
email: "alice@example.com"
```

#### Complete (Robust) Profile Example
```yaml
schema: "1.0.0"
name: Mohsen Malekmohammadi
name_fa: "محسن ملک‌محمدی"
email: "mm91mm92@gmail.com"
website: "https://example.com"
location: "Tehran, Iran"
affiliation: "Payam-e Noor University"
affiliation_fa: "دانشگاه پیام‌نور"
bio: >
  Researcher and maintainer of the Pearls of Persian project,
  specializing in classical Persian poetry and mystical literature.
bio_fa: >
  پژوهشگر و نگه‌دارندهٔ پروژه «مروارید پارسی با محسن» با تخصص در
  شعر کلاسیک فارسی و ادبیات عرفانی.
preferred_roles:
  - "محقق"
  - "نگه‌دارنده"
links:
  github: "https://github.com/mm91mm92"
  orcid: "0000-0002-1234-5678"
  linkedin: "https://linkedin.com/in/mm91mm92"
  telegram: "example"
```

#### Directory Context Example
The representation of multiple contributor profiles under the `contributors/` path:

*File: `contributors/mm91mm92.yml`*
```yaml
schema: "1.0.0"
name: Mohsen Malekmohammadi
name_fa: "محسن ملک‌محمدی"
email: "mm91mm92@gmail.com"
website: "https://mohsen.example.com"
bio_fa: "پژوهشگر و نگه‌دارنده اصلی پروژه مروارید پارسی."
links:
  github: "https://github.com/mm91mm92"
```

*File: `contributors/alice-johnson.yml`*
```yaml
schema: "1.0.0"
name: Alice Johnson
name_fa: "آلیس جانسون"
email: "alice@example.com"
bio: "Persian literature editor and reviewer."
bio_fa: "ویراستار و بازبین ادبیات فارسی."
links:
  github: "https://github.com/alicej"
```

*File: `contributors/bob-smith.yml`*
```yaml
schema: "1.0.0"
name: Bob Smith
name_fa: "باب اسمیت"
email: "bob@example.com"
affiliation: "Stanford University"
affiliation_fa: "دانشگاه استنفورد"
```

---

### 5. Contributor Profile Validation Checklist

Automated validation engines should verify the following conditions:
* [ ] The contributor file exists directly under the `contributors/` path.
* [ ] The filename contains only lowercase alphanumeric characters, numbers, and single hyphens (no spaces or special characters, excluding the `.yml` extension).
* [ ] The file compiles successfully as standard YAML.
* [ ] All four required keys (`schema`, `name`, `name_fa`, `email`) are present and non-empty.
* [ ] Format checks pass for the `email` structure.
* [ ] The `links` field, if present, is configured as a flat map of strings (unrecognized keys inside `links` are allowed to maintain forward compatibility).
* [ ] Any unrecognized top-level keys are ignored by validation engines to preserve forward compatibility.

---

## Part 3: Literary Document Specification

### 1. Overview
A literary document is a UTF-8 Markdown file (`.md`) containing classical Persian poetry (verses) and/or prose (sentences) with scholarly annotations, located as a leaf node under `content/`. 

Each document consists of:
1. **YAML Front Matter:** Structured metadata.
2. **Primary Heading (H1):** The single document title rendered on the page.
3. **Content Body:** Verse or prose blocks, subtitles, commentary, and glossary tables.

The **canonical identifier** of a literary document is its **filename** and its path within the `content/` hierarchy. Metadata that is derivable from the repository structure or Git history is intentionally excluded from the front matter to maintain structural integrity. This includes:
* Hierarchy context (e.g., `poet`, `book`, breadcrumbs), which is derived from ancestor `meta.yml:title` entries.
* Chronology context (e.g., `last_modified`), which is derived from Git history.
* Order context, which is managed exclusively by the parent directory’s `meta.yml:children` array.

---

### 2. Front Matter Boundary & Metadata Schema

The document **must** begin directly with valid YAML front matter enclosed by triple-dash (`---`) delimiters. No content or empty space may precede the opening `---` delimiter.

Immediately after the closing `---` of the front matter, only blank lines and Markdown/HTML comments (`<!-- ... -->`) are allowed before the primary H1 heading.

#### Schema Format
```yaml
---
schema: <string>
title: <string>
status: <string>
prosody: <string>        # optional
prosody_name: <string>   # optional
contributors:
  - id: <string>
    role: <string>
tags: 
  - <string>             # optional
  - <string>             # optional
sources: 
  - <string>             # optional
  - <string>             # optional
notes: |                 # optional                  
  <string>   
  <string>                 
---
```

#### Required Front Matter Fields

##### `schema`
* **Type:** String (Semantic Versioning format: `"MAJOR.MINOR.PATCH"`) [6]
* **Required:** Yes
* **Value:** `"1.1.0"` (or subsequent minor/patch versions) [6]
* **Purpose:** Identifies the metadata schema version. Utilizing string format prevents float-point precision issues or numeric coercion (e.g., parsers treating `1.10` and `1.1` as numerically equivalent). [6]

###### Semantic Versioning Rules [6]

| Component | Increment When | Example |
|-----------|----------------|---------|
| **MAJOR** | Breaking changes requiring document rewrites | Renaming a required field<br>Removing support for H3 blocks<br>Changing `children` from array to object |
| **MINOR** | Backward-compatible additions | Adding optional `prosody` or `prosody_name` fields<br>Supporting enjambed verses<br>Adding a new optional H4 section |
| **PATCH** | Documentation fixes, no structural changes | Fixing typos in examples<br>Clarifying validation rule wording<br>Adding usage guidance |

###### Compatibility Promise [6]
* **Forward Compatibility:** Documents valid under schema `X.Y.Z` remain valid under schema `X.(Y+n).Z`. For example, a document with `schema: "1.0.0"` is still valid when validators support version `"1.1.0"`. [6]
* **Backward Compatibility:** Parsers supporting schema `X.Y.Z` must gracefully handle documents using `X.(Y-n).Z` by ignoring newer optional keys. [6]
* **Breaking Changes:** Major version increments (`1.x.x` → `2.0.0`) indicate breaking changes requiring document migration. [6]

##### `title`
* **Type:** String  
* **Required:** Yes  
* **Purpose:** Defines the canonical, concise display title of this document utilized for structural GUI elements, such as sidebars, breadcrumbs, hierarchy panels, and navigation trees.

##### `status`
* **Type:** String (Enumeration)  
* **Required:** Yes  
* **Allowed Values:** `"draft"`, `"final"`, `"archived"`
* **Purpose:** Tracks the editorial workflow stage of the document.

###### Status Definitions & Workflow Stages:

```
draft → final → archived
```

* `draft`: The document is in its initial stage. It might be actively written or annotated, contain errors, or need revision.
* `final`: The document is completed, ready for public display, and is under active monitoring for correctness and potential improvement.
* `archived`: The document was once marked as `final`, but is no longer actively monitored or maintained.

###### State Transitions:
* **Forward Progression (`draft` → `final`):** A new, erroneous, or incomplete document is completed.
* **Retirement Cycle (`final` → `archived`):** End of active monitoring and maintenance.
* **Backward Transition (`final` → `draft`):** Re-opened for major revision or corrective authoring.

###### Display Guidelines:
The `status` value does affect a literary document's visibility in the presentation layer. All documents remain publicly visible, accompanied by their corresponding workflow state status and matching visual indicator:

| Status | Public Display | Badge / Indicator |
|---------|----------------|-------------------|
| `draft` | Visible with warning | 🟡 "پیش‌نویس" (requires interface warning notice) |
| `final` | Visible | ✅ "تکمیل‌شده" |
| `archived` | Visible with warning | ⚫ "بایگانی‌شده" (requires interface warning notice) |

##### `contributors`
* **Type:** Array of Objects  
* **Required:** Yes (minimum of one entry is required)  
* **Purpose:** Credits individuals who participated in researching, editing, translating, or maintaining the document.

Each object within the array must contain the following fields:

| Field | Type | Required | Description |
|------|------|----------|-------------|
| `id` | String | Yes | Contributor identifier; must correspond to an existing record file: `contributors/<id>.yml` |
| `role` | String | Yes | The specific role performed by the contributor. |

**Recommended roles:**

| **Persian role** | **English role** |
| --------------: | : ----------------|
| `محقق` | Researcher |
| `ویراستار` | Editor |
| `بازبین‌گر` | Reviewer |
| `مترجم` | Translator |
| `نگه‌دارنده` | Maintainer |

---

#### Optional Front Matter Fields

##### `prosody` 
* **Type:** String  
* **Required:** No  
* **Purpose:** Documents the poetic metrical pattern (وزن شعر) of the literary text using the classical aruz (عروض) system. 
* **Example:** `"فعولن فعولن فعولن فعل"` 

##### `prosody_name` 
* **Type:** String  
* **Required:** No  
* **Purpose:** Documents the canonical name of the poetic meter (بحر شعر). 
* **Example:** `"رمل مثمن محذوف"` 

##### `tags`
* **Type:** Array of Strings  
* **Required:** No (defaults implicitly to `[]` if omitted)  
* **Purpose:** Provides thematic keywords for search indexing, filtering, and cross-referencing.

###### Syntax for Multi-Word Tags
Use **a single YAML string** containing spaces for multi-word tags. This is standard YAML syntax and fits Persian typography naturally.
```yaml
tags:
  - "عشق الهی"
  - "سیر و سلوک"
  - "وحدت وجود"
```

###### Persian Typography Guidelines
* Use **normal spaces** between separate words: `"عشق الهی"`.
* Use the **Zero-Width Non-Joiner (ZWNJ / نیم‌فاصله)** where Persian orthography expects compound word division: `"می‌خانه"`, `"گفت‌وگو"`, `"دل‌بر"`.

###### Normalization Rules
* Trim leading and trailing whitespace.
* Collapse repeated internal spaces to a single standard space.
* Standardize on a single canonical spelling (e.g., prefer `سلوک` consistently, avoiding alternate representations unless a distinction is explicitly intended).
* Do not encode multi-word tags with hyphens or underscores (such as `عشق-الهی`). Tags must remain human-readable labels; web presentation systems should generate URL slugs automatically.

##### `sources`
* **Type:** Array of Strings  
* **Required:** No (defaults implicitly to `[]` if omitted)  
* **Purpose:** Documents bibliographic and provenance information for the text, editions, commentaries, manuscripts, and online resources used.

###### Design Philosophy
* **Simplified Structure:** Each source is a plain string.
* **Contributor Responsibility:** Contributors format citations as appropriate.
* **No Validation Overhead:** Maintainers and automated validation scripts do not parse or enforce strict citation structure.
* **Maximum Flexibility:** Accommodates any citation style, methodology, or local standard (e.g., APA, MLA, Chicago, or custom Persian formats).

###### Guidelines for Contributors
While the format remains free-form, contributors are encouraged to provide sufficient information to identify and locate the cited source:
* **Online Sources:** Name/title, URL, access date.
  ```yaml
  sources:
    - "گنجور: https://ganjoor.net/attar/manteghotteyr/sh1/ (مشاهده: ۱۴۰۵/۰۴/۳۰)"
  ```
* **Published Editions:** Title, editor/author, publisher, place of publication, year of publication, page numbers.
  ```yaml
  sources:
    - "منطق‌الطیر، تصحیح تقی تفضلی، انتشارات سخن، تهران، ۱۳۶۲، ص ۱-۳"
  ```
* **Manuscripts:** Repository/library, manuscript identifier or catalog number, date, folio numbers.
  ```yaml
  sources:
    - "نسخه خطی کتابخانه ملی ایران، شماره ۱۲۳۴۵، ۷۲۳ ق، برگ ۳ب-۴الف"
  ```
* **Commentaries:** Title, author, publisher, year of publication, volume/pages cited.
  ```yaml
  sources:
    - "شرح منطق‌الطیر، بدیع‌الزمان فروزانفر، انتشارات دانشگاه تهران، ۱۳۵۳، جلد ۱، ص ۴۵-۵۲"
  ```

###### Style Guide Recommendations
System documentation should encourage consistent formatting standards:
* **Online Sources:** `نام منبع: URL (مشاهده: تاریخ)`
* **Books:** `عنوان، ویراستار/مؤلف، ناشر، مکان، سال، صفحات`
* **Manuscripts:** `مخزن، شماره نسخه، تاریخ، شماره برگ`

##### `notes`
* **Type:** String (multi-line supported)  
* **Required:** No  
* **Purpose:** For editorial notes, placeholders, or clarifications. This field is designated for internal workflow tracking and is not displayed publicly.

---

### 3. Document Structural Layout & Heading Hierarchy

#### 3.1 Document Title (H1)
* There **must** be **exactly one** H1 heading (`#`) in the entire document.
* The H1 heading **must** represent the display title rendered directly on the actual content page.
* **Semantic Differentiation from Front Matter `title`:** 
  * **`title` (Front Matter):** Displays strictly in GUI navigation spaces (breadcrumbs, hierarchy trees, sidebars, and structural panels). It represents a concise technical identifier. 
  * **H1 (Markdown Body):** Displays on the actual page itself, allowing for a more stylized, comprehensive, or layout-specific title representation. 
* There must be no other `H1` headings permitted anywhere else in the document.

#### 3.2 Subtitles & Major Sections (H2)
* All top-level divisions directly beneath the main document title **must** use H2 headings (`##`).
* H2 sections are optional and may be repeated as necessary to divide the document into major thematic, narrative, or structural divisions.
* If a document has no natural high-level divisions, H2 headings must be omitted entirely, with the content proceeding directly to H3 blocks.

#### 3.3 Verse/Prose Block Headings (H3)
* Each content block **must** begin with an H3 heading (`###`).
* The H3 heading must conform to a bare numeric format or a numeric range for enjambed content: [3]
  ```markdown
  ### <number>
  ```
  Or:
  ```markdown
  ### <number_range>
  ```
* The `<number>` or `<number_range>` variable must be represented as a Persian numeral (۱, ۲, ۳, ۱-۲, ۱۲۳-۱۲۴, etc.) to ensure typographical consistency. [3]
* By using plain numbers instead of "بیت", the H3 heading format accommodates both verses (poetry) and sentences (prose). 
* Using numeric ranges (e.g., `### ۱۲۳-۱۲۴`) allows the heading to accommodate multiple verses where enjambed couplets (موقوف‌المعانی) are annotated together as a cohesive semantic block. [3]
* There must be exactly one H3 heading per verse/prose block.

#### 3.4 Standard Sub-sections Within H3 Blocks (H4)
Within each H3 block, a maximum of five standardized sub-sections are permitted. These sub-sections must use H4 headings (`####`) and follow the canonical vocabulary and recommended order defined below. 

Any optional H4 heading may be omitted if not applicable, but must not be replaced with non-standard titles or alternate heading levels.

```
H3: <number> or <number_range>
├── H4: متن (Required)
├── H4: واژگان و ترکیبات (Optional)
├── H4: معنی روان (Optional)
├── H4: نکات ادبی (Optional)
└── H4: ارجاعات (Optional)
```

##### `متن` (Text) — Required
* **Purpose:** Contains the physical text of the content block. Only this section is strictly mandatory; all other H4 sections are optional. [2]
* **Formatting & Content Detection Rules:** [3]
  
  The refined content structure supports four distinct structural types: [3]

| Type | Line Count | Formatting | Use Case / Literary Application |
| :--- | :--- | :--- | :--- |
| **1. Prose (نثر)** | 1 | Plain text | Sentences (نثر) [3] |
| **2. Single Hemistich (مصرع)** | 1 | Bold (`**text**`) | Standalone hemistichs (مصرع مفرد) or epigraphs [3] |
| **3. Single Couplet (بیت)** | 2 | Bold (`**text**`), separated by exactly 1 blank line | Standard couplets [3] |
| **4. Enjambed Verse (موقوف‌المعانی)** | More than 2 (even count) | Bold (`**text**`), separated by exactly 1 blank line between consecutive hemistichs | Enjambed verses spanning multiple couplets (موقوف‌المعانی) or poetic forms (مخمس, مسبع) [3] |

##### `واژگان و ترکیبات` (Vocabulary & Phrases) — Optional [2]
* **Purpose:** Serves as an annotated glossary for difficult terms, phrases, or idiomatic expressions within the block.
* **Format:** Must use a standard Markdown table with the first column right-aligned (`|---:|---|`). The target word or phrase in the first column must be bolded, with the explanation provided in plain text.

##### `معنی روان` (Fluent Meaning) — Optional [2]
* **Purpose:** Provides a modern Persian paraphrase or prose translation of the block.
* **Format:** Written in contemporary Persian prose paragraphs. It must capture the core meaning of the verse or sentence without relying on literal word-for-word translation.

##### `نکات ادبی` (Literary Notes) — Optional [2]
* **Purpose:** Offers scholarly commentary on rhetorical figures, literary devices, stylistic nuances, meter, or rhyme.
* **Format:** Written in plain text, supporting multiple paragraphs if needed. It addresses aspects such as metaphors, similes, or thematic developments.

##### `ارجاعات` (References) — Optional [2]
* **Purpose:** Contains scholarly cross-references to related verses, chapters, external sources, or thematic connections.
* **Format:** No special formatting is mandatory. Bulleted lists are recommended.

#### 3.5 Contextual Typing and Composition [5]
* A literary document is structurally flexible and can contain a mix of both verse and prose blocks. [5]
* Each H3 content block is independently typed based on the physical structure of its `متن` section (e.g., line count, bolding, and blank-line separators). [5]
* No document-level layout or type declaration is needed. The presentation layer automatically detects and formats each block type on-the-fly. [5]

#### 3.6 Horizontal Rules Between Content Blocks
* A Markdown horizontal rule (`---`) is recommended **between content blocks** to serve as a distinct visual separator.
* Horizontal rules must not be positioned in a manner that conflicts or interferes with the YAML front matter delimiters at the top of the document.

#### 3.7 Prohibited Headings
1. **Setext Headings are Prohibited:** Headings underlined with equals signs or hyphens (e.g., `Title` followed by `====` on the next line) must not be used.
2. **Multiple H1 Headings are Prohibited:** Only one H1 heading is allowed per document.
3. **Improper Heading Levels within Blocks:** Block-level content must strictly map H3 to the block number and H4 to standard sub-sections. H2 or H3 must not be used inside a block.

---

### 4. Complete Document Templates & Examples

#### Complete Document Template
```markdown
---
schema: "1.1.0"
title: "<concise_gui_title>"
status: "<draft|final|archived>"
prosody: "<prosody_pattern>"      # optional
prosody_name: "<meter_name>"      # optional

contributors: 
  - id: "<contributor_id_1>"
    role: "<role_1>"
  - id: "<contributor_id_2>"
    role: "<role_2>"

tags: 
  - "<tag_1>"
  - "<tag_2>"

sources: 
  - "<source_citation_1>"
  - "<source_citation_2>"

notes: |
  <internal_editorial_notes>
---

<!--
قالب رسمی سند ادبی تحلیل‌شده.
- بخش‌های اختیاری (مانند واژگان و ترکیبات، معنی روان، نکات ادبی یا ارجاعات) در صورت عدم کاربرد حذف می‌شوند.
-->

# <stylized_page_title>

<!-- بخش‌های اختیاری H2 در صورت نیاز برای تقسیم‌بندی‌های کلان استفاده می‌شوند -->
## <subtitle_1>

<!-- The verse/sentence block -->
### <persian_numeral_or_range>

<!-- The text of the verse or the sentence, this heading is compulsory -->
#### متن

<text_content_block>

<!-- This H4 can be omitted if there is no difficult word or phrase to explain -->
#### واژگان و ترکیبات

| واژه/ترکیب | معنی/توضیح |
|---:|---|
| **<word_1>** | <explanation_1> |

<!-- This H4 heading is optional -->
#### معنی روان

<prose_translation>

<!-- This H4 heading is optional -->
#### نکات ادبی

<literary_commentary>

<!-- This H4 heading is optional -->
#### ارجاعات

- [<internal_reference_text>](relative/path/to/file.md)
- <external_citation>

---

<!-- Repeat block for the next verse or sentence -->

```

#### Complete Example Document (Mixed Poetry and Prose, with Enjambment and Prosody)
```markdown
---
schema: "1.1.0"
title: "مجمع مرغان"
status: "final"
prosody: "فعولن فعولن فعولن فعل"
prosody_name: "رمل مثمن محذوف"

contributors: 
  - id: "mm91mm92"
    role: "محقق"

tags: 
  - "هدهد"
  - "سیر و سلوک"
  - "عرفان"

sources: 
  - "گنجور: https://ganjoor.net/attar/manteghotteyr/sh1/ (مشاهده: ۱۴۰۵/۰۴/۳۰)"
  - "منطق‌الطیر، تصحیح تقی تفضلی، انتشارات سخن، تهران، ۱۳۶۲، ص ۱-۳"
---

# مجمع مرغان (داستان سفر پرندگان)

## آغاز داستان

### ۱

#### متن

**همه مرغان عالم طالب آن شده‌اند**

**که سیمرغ را یکدلانه ببینند**

#### واژگان و ترکیبات

| واژه/ترکیب | معنی/توضیح |
|---:|---|
| **سیمرغ** | پرنده‌ای افسانه‌ای که در ادبیات عرفانی نماد ذات حق است |
| **طالب** | جوینده، کسی که در جستجوی چیزی است |
| **یکدلانه** | به‌طور خالص و با تمام وجود |

#### معنی روان

تمام پرندگان جهان در جستجوی آن هستند که سیمرغ را با خلوص و یکدلی ببینند.

#### نکات ادبی

در این بیت، استعاره «سیمرغ» به‌کار رفته که نماد ذات الهی است. واژه «یکدلانه» بر اهمیت اخلاص در طلب تأکید می‌کند.

#### ارجاعات

- [خطاب هدهد](خطاب-هدهد.md)
- قرآن، سوره نمل، آیه ۲۰

---

### ۲

#### متن

**یکی هدهد آمد ز هر هفت کشور**

**بیامد به نزدیک مرغان به بشور**

#### واژگان و ترکیبات

| واژه/ترکیب | معنی/توضیح |
|---:|---|
| **هفت کشور** | اشاره به هفت اقلیم جهان |
| **بشور** | شور و نشاط |

---

### ۳

#### متن

این آغاز سخن است که هدهد خردمند بر زبان راند.

#### معنی روان

این نخستین گفتار هدهد دانا و خردمند بود که خطاب به پرندگان بیان نمود.

---

### ۴-۵

#### متن

**هـمانا کـه بـشنـیده‌ای نـامِ مـن**

**بـه هـر جـا کـه رفـتـست پـیـغامِ مـن**

**کـه مـن رُسـتـمِ گُـرد و شـیـراوژَنَـم**

**بـه جـنـگ انـدرون کـوهِ آهـن کـنـم**

#### معنی روان

مسلماً نام و آوازه من به هر کجا که سخنی از جنگاوری رفته، رسیده است؛ چرا که من رستم پهلوان و شیرگیر هستم و در جنگ‌ها می‌توانم کوهی از آهن را از جای برکنم.
```

---

### 5. Literary Document Validation Checklist

A literary document must comply with the following structural and field constraints to be considered valid:

#### Physical Structure & Syntax
* [ ] The file begins directly with YAML front matter enclosed by `---` delimiters.
* [ ] Front matter contains valid YAML syntax.
* [ ] The first content element following the front matter block is an H1 heading (`# ...`).
* [ ] The document contains exactly one H1 heading.
* [ ] No Setext-style headings are used.
* [ ] All division levels are properly nested (H2 for main sections, H3 for verse/prose blocks, H4 for standard components).

#### Front Matter Schema Checks
* [ ] `schema` is present and matches a valid semantic version string (e.g. `"1.1.0"`). [6]
* [ ] `title` is present and is a non-empty string. 
* [ ] `status` is present, is a quoted string, and is one of: `"draft"`, `"final"`, `"archived"`.
* [ ] `contributors` is present and contains at least one object.
* [ ] Each contributor item contains both `id` and `role` fields.
* [ ] Each contributor `id` references an existing record at `contributors/<id>.yml`.
* [ ] If `prosody` is present, it must be formatted as a string. [1]
* [ ] If `prosody_name` is present, it must be formatted as a string. [1]
* [ ] If `tags` is present, it must be formatted as an array of strings.
* [ ] Each tag must be a single string containing spaces for multi-word structures, rather than containing underscores or hyphens.
* [ ] If `sources` is present, it must be formatted as an array of strings.
* [ ] Each source item must be a non-empty string.
* [ ] If `notes` is present, it must be formatted as a string.

#### Verse/Prose Block Internal Checks
* [ ] Each content block begins with an H3 heading formatted exactly as: `### <number>` or `### <number_range>`, utilizing Persian numerals. [3]
* [ ] The required H4 sub-section `#### متن` is present in every block. [2]
* [ ] The `#### متن` section adheres to one of the four supported types: prose (1 plain-text line), single hemistich (1 bolded line), single couplet (2 bolded lines with exactly 1 blank line separator), or enjambed verse (even bolded line count separated by blank lines). [3]
* [ ] If the optional glossary `#### واژگان و ترکیبات` is present, it uses the correct right-aligned Markdown table format with bolded entry keys.
* [ ] Sequential content blocks are separated by a horizontal rule (`---`).

---

### 6. Presentation Layer & Git Integration Guidelines

#### Metadata Field Mapping

| Interface / Component | Associated Field |
|-----------------------|------------------|
| HTML `<title>` element | `title` |
| Primary Page Title | Document H1 (stylized title rendered on the page)  |
| Breadcrumb navigation | `title` (concise GUI title)  |
| Search index indexing | `title`, `tags` |
| Cross-referencing and links | `title` |
| Scholarly citation | `title`, `contributors`, `sources` |
| Filtering and grouping queries | `status`, `tags` |
| Academic attribution lists | `contributors` |
| Reference bibliography | `sources` |

#### Git Integration
Build systems can extract historical data from Git to supplement front matter metadata, avoiding manual entry:

**Extract Last Modified Timestamp:**
```bash
git log -1 --format="%ai" -- path/to/file.md
```

**Extract Full Revision History:**
```bash
git log --follow --format="%h|%ai|%an|%s" -- path/to/file.md
```