# Pearls of Persian: Repository Schema Specification

**Schema Version: 1.0.0**

This specification defines the structure of the Pearls of Persian with Mohsen literary corpus. The project is a **personal reading-and-recording system** for Persian literary works. It is designed to support systematic reading, recording, revisiting, and sharing of literary texts and reading notes.

It is **not** intended to be:

-   a critical-edition system;
-   a manuscript-collation system;
-   a comprehensive bibliographic database;
-   a collaborative annotation platform;
-   a general-purpose CMS;
-   a database-backed dynamic application;
-   a multilingual parallel-text platform;
-   an exhaustive prosodic or linguistic encoding system.

**Design Principles:**

- **Reading-first:** The schema serves systematic literary reading and recording rather than formal scholarly editing.
- **Content-first authoring:** Markdown should remain pleasant for a human to write and maintain.
- **Structure from filesystem and metadata:** The filesystem and directory metadata define the literary hierarchy. Literary documents should not redundantly encode information that can be derived from their location or their parent directory metadata.
- **Presentation independence:** Authoring conventions should not unnecessarily encode presentation-layer decisions.
- **Persian-first, not Persian-only:** Persian typography and literary terminology are first-class concerns, while source files remain practical for general-purpose tooling.
- **Deterministic compilation:** The same valid repository state must produce the same TypeScript representation.
- **Git as history:** Information already available from Git history should not be duplicated in front matter merely for convenience.
- **Extensible without premature structure:** Not every potentially useful piece of information needs a dedicated schema field. The schema should remain deliberately small until a real use case justifies additional structure.

---

## Part 1: Corpus and Literary Hierarchy

### 2. The Corpus

The literary corpus is stored under the root-level `corpus/` directory.

``` text
corpus/
├── meta.yml
├── poet-a/
│   ├── meta.yml
│   ├── work-a/
│   │   ├── meta.yml
│   │   └── ...
│   └── work-b/
│       ├── meta.yml
│       └── ...
└── poet-b/
    ├── meta.yml
    └── ...
```

`corpus/` is the root of the project's **literary hierarchy**. The literary hierarchy has two fixed upper levels: 

``` text
corpus
└── poet
    └── work
```

Below the work level, the internal organization of a literary work is intentionally unrestricted. It is defined by the work's traditional or editorial structure through `meta.yml:children`.

### 3. Literary Hierarchy Concepts

The following concepts must be distinguished.

#### 3.1 Poet

A poet directory represents a poet and contains that poet's works.

#### 3.2 Work

A work directory represents a distinct literary work. `work` is preferred to `book` because not every literary work is appropriately described as a book. Examples include a divan, masnavi, anthology, or other recognized literary work.

#### 3.3 Structural Division

A **structural division** is an organizational subdivision of a literary work or of another structural division. Traditional Persian literary works may use different terms for such divisions, including:

-   دفتر
-   باب
-   فصل
-   بخش
-   قسم
-   جزء
-   گفتار
-   مجلس
-   روضه
-   مقاله

The schema does not prescribe a universal hierarchy among these terms. A directory of type `structural_division` preserves the actual structure of the particular work.

A structural division may contain other structural divisions, literary documents, or any combination of the two, in any order.

#### 3.4 Literary Unit / Narrative Unit / Literary Document

A **literary unit** is a recognizable literary or narrative unit within a work, such as a حکایت, داستان, غزل, قصیده, رباعی, دیباچه, or another unit appropriate to the work.

A **literary document** is the Markdown document that records such a unit.

The terms **literary unit** and **literary document** may be used almost interchangeably when the distinction between the conceptual unit and its Markdown representation is immaterial. **Narrative unit** may be used as a synonym for literary unit when the unit is specifically narrative.

Literary documents are always Markdown files and are leaf nodes of the literary hierarchy.

### 4. Structural Divisions vs. Literary Units

Structural divisions and literary units are different concepts. A structural division organizes a work; a literary unit is a literary or narrative component contained within that organization.

For example:

``` text
گلستان
├── دیباچه.md
├── باب اول
│   ├── حکایت-اول.md
│   ├── حکایت-دوم.md
│   └── ...
├── باب دوم
└── ...
```

Here:

-   `باب اول` and `باب دوم` are structural divisions.
-   `دیباچه.md` and the `حکایت` items are literary units.

The schema must not assume that every work follows:

``` text
work
└── structural division
    └── literary unit
```

A work may contain literary units directly, structural divisions directly, or any mixture of structural divisions and literary units appropriate to its actual structure.

### 5. Directory Inclusivity Rule

The filesystem provides the physical organization of the corpus, while each directory's `meta.yml:children` field defines the literary hierarchy among its immediate children.

A directory may contain Markdown documents, subdirectories, or a combination of the two **only as permitted by the directory's type**.

| Directory type | Permitted children |
| -------------- | ------------------ |
| `root` | `poet` directories only |
| `poet` | `work` directories only |
| `work` | Any combination and order of structural divisions and literary documents |
| `structural_division` | Any combination and order of structural divisions and literary documents |

There is therefore no general filesystem rule prohibiting a directory from containing both directories and Markdown documents.

For example, the following is valid:

``` text
corpus/
└── سعدی/
    └── گلستان/
        ├── meta.yml
        ├── دیباچه.md
        ├── باب-اول/
        ├── باب-دوم/
        └── خاتمه.md
```

Likewise, a structural division may contain both subordinate structural divisions and literary documents:

``` text
باب-اول/
├── meta.yml
├── حکایت-اول.md
├── بخش-فرعی/
└── حکایت-دوم.md
```

The `children` field is authoritative for membership and canonical order.

#### 5.1 Directory Types

The controlled vocabulary for `meta.yml:type` is:

| Type | Meaning |
|---|---|
| `root` | Root of the literary corpus |
| `poet` | Poet-level directory |
| `work` | Literary-work directory |
| `structural_division` | Any structural subdivision of a work or another structural division |

Traditional terminology such as `دفتر`, `باب`, `فصل`, `بخش`, or `روضه` belongs in the directory's `title`, not in a proliferation of schema types.

This keeps the schema generic while allowing each work to preserve its own traditional terminology.

---

## Part 2: Directory Metadata Specification (`meta.yml`)

### 6. Overview

Every directory in `corpus/` must contain a `meta.yml` file. The file is the single source of truth for:

1.  the directory's metadata;
2.  its semantic directory type;
3.  the membership of its immediate literary children;
4.  the canonical order of those children;
5.  the visibility of those children in the presentation layer.

### 7. Required Fields

| Field | Type | Description |
|---|---|---|
| `schema` | String | Unified repository schema version |
| `type` | String | Directory type |
| `title` | String | Human-readable title |
| `children` | Array of Strings | Ordered list of immediate literary children |

Example:

``` yaml
schema: "1.0.0"
type: "poet"
title: "عطار نیشابوری"
children:
  - منطق-الطیر
  - الهی-نامه
```

### 8. Optional Fields

| Field | Type | Description |
|---|---|---|
| `description` | String | Short public description |
| `tags` | Array of Strings | Directory-level keywords |
| `notes` | String | Internal editorial notes |

Unknown fields should be ignored by validation and compilation to preserve forward compatibility.

### 9. `schema`

The value is always:

``` yaml
schema: "1.0.0"
```

The schema is **unified**: directory metadata, contributor profiles, and literary documents all belong to the same repository schema and therefore use the same version number. The three components are not independently versioned.

### 10. `type`

`type` is required and must be one of:

- `root`
- `poet`
- `work`
- `structural_division`

The type determines which kinds of immediate children are permitted under the Directory Inclusivity Rule.

### 11. `title`

`title` is the human-readable title of the directory. It may differ from the physical directory name and is used by the presentation layer for navigation, breadcrumbs, headings, and other display purposes.

### 12. `children`

`children` is a required array of strings. It defines:

1.  which immediate filesystem children belong to the literary
    hierarchy;
2.  their canonical order.

Example:

``` yaml
children:
  - دیباچه.md
  - باب-اول
  - باب-دوم
  - خاتمه.md
```

#### Rules

1.  Every entry must exactly match the name of an immediate filesystem child.
2.  Every listed child must have a type permitted by the parent directory's `type`.
3.  A `children` array may contain both directory names and Markdown filenames when the parent type permits both.
4.  The order of entries is authoritative.
5.  Duplicate entries are prohibited.
6.  `meta.yml` itself must never appear in `children`.
7.  Filesystem children omitted from `children` are excluded from the public literary hierarchy and presentation layer.

### 13. Directory Examples

#### Root

``` yaml
schema: "1.0.0"
type: "root"
title: "گوهرهای پارسی با محسن"
children:
  - عطار-نیشابوری/
  - مولوی/
  - سعدی/
```

#### Poet

``` yaml
schema: "1.0.0"
type: "poet"
title: "عطار نیشابوری"
children:
  - منطق-الطیر/
  - الهی-نامه/
```

#### Work with Mixed Literary Hierarchy

``` yaml
schema: "1.0.0"
type: "work"
title: "گلستان"
children:
  - دیباچه.md
  - باب-اول/
  - باب-دوم/
  - خاتمه.md
```

#### Structural Division with Mixed Children

``` yaml
schema: "1.0.0"
type: "structural_division"
title: "باب اول"
children:
  - حکایت-اول.md
  - بخش-فرعی/
  - حکایت-دوم.md
```

------------------------------------------------------------------------

## Part 3: Contributor Profile Specification

### 14. Overview

Every contributor participating in the repository must be represented by a single YAML file located in the `contributors/` directory.

```text
contributors/
├── mm91mm92.yml
├── alice-johnson.yml
└── bob-smith.yml
```

##### File Location and Identity
* **Path:** `contributors/<id>.yml`
* The filename `<id>` (excluding the `.yml` extension) serves as the unique identifier for the contributor. This `<id>` is the value referenced in the `contributors` metadata array within literary documents.
* The file serves as the single source of truth for the contributor's identity, contact info, and professional profile.

### 15. Required Fields

``` yaml
schema: "1.0.0"
name: "..."
name_fa: "..."
email: "..."
```

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `schema` | String | Identifies the metadata schema version in SemVer format. | `"1.0.0"` |
| `name` | String | Full name in Latin script (used for international search and indexing). | `"Mohsen Malekmohammadi"` |
| `name_fa` | String | Full name in Persian script (used for the primary presentation layer). | `"محسن ملک‌محمدی"` |
| `email` | String | Contact email address. | `"mm91mm92@gmail.com"` |

### 16. Optional Fields

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

Unknown top-level fields and unknown keys inside `links` must not cause
validation failure.

#### The `links` Object
To maintain a highly extensible and clean structure, external identifiers, social profiles, and developer accounts are grouped inside the `links` object. 

The `links` field is formatted as a flat dictionary of key-value pairs:
```yaml
links:
  <platform>: <string>
```

**Guidelines:**
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


### 17. Contributor References & Profile Examples

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

#### Reference Contributors

A literary document references contributors by their IDs:

``` yaml
contributors:
  - id: "mm91mm92"
    role: "محقق"
```

The referenced file must exist at:

``` text
contributors/<id>.yml
```

---

## Part 4: Literary Document Specification

### 18. Overview

A literary document is a UTF-8 Markdown file representing a literary unit/narrative unit. It is a leaf node of the literary hierarchy.

A literary document consists of:

1.  the YAML front matter;
2.  exactly one H1 document title;
3.  optional H2 sections;
4.  one or more content blocks.

The document is intended for reading and recording literary text and associated notes, rather than for formal textual criticism.

### 19. Front Matter

The document must begin directly with YAML front matter:

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

#### Required fields

| Field | Type |
|---|---|
| `schema` | String |
| `title` | String |
| `status` | String |
| `contributors` | Array of objects |

##### `schema`
* **Type:** String (Semantic Versioning format: `"MAJOR.MINOR.PATCH"`) 
* **Required:** Yes
* **Value:** `"1.0.0"` for the time being and subsequent major, minor, or patch versions later
* **Purpose:** Identifies the metadata schema version. Utilizing string format prevents float-point precision issues or numeric coercion (e.g., parsers treating `1.10` and `1.1` as numerically equivalent). 

**Semantic Versioning Rules:**

| Component | Increment When | Example |
|-----------|----------------|---------|
| **MAJOR** | Breaking changes requiring document rewrites | Renaming a required field<br>Removing support for H3 blocks<br>Changing `children` from array to object |
| **MINOR** | Backward-compatible additions | Adding optional fields to the front matter<br>Supporting a new type of excerpt which would otherwise falls under one of the currently available types<br>Adding a new optional H4 section |
| **PATCH** | Documentation fixes, no structural changes | Fixing typos in examples<br>Clarifying validation rule wording<br>Adding usage guidance |

**Compatibility Promise:**
* **Forward Compatibility:** Documents valid under schema `X.Y.Z` remain valid under schema `X.(Y+n).Z`. For example, a document with `schema: "1.0.0"` is still valid when validators support version `"1.1.0"`. 
* **Backward Compatibility:** Parsers supporting schema `X.Y.Z` must gracefully handle documents using `X.(Y-n).Z` by ignoring newer optional keys. 
* **Breaking Changes:** Major version increments (`1.x.x` → `2.0.0`) indicate breaking changes requiring document migration. 

##### `title`
* **Type:** String  
* **Required:** Yes  
* **Purpose:** Defines the canonical, concise display title of this document utilized for structural GUI elements, such as sidebars, breadcrumbs, hierarchy panels, and navigation trees but not the page title.

##### `status`
* **Type:** String (Enumeration)  
* **Required:** Yes  
* **Allowed Values:** `"draft"`, `"final"`, `"archived"`
* **Purpose:** Tracks the editorial workflow stage of the document.

**Status Definitions & Workflow Stages:**

```
draft → final → archived
```

* `draft`: The document is in its initial stage. It might be actively written or annotated, contain errors, or need revision.
* `final`: The document is completed, ready for public display, and is under active monitoring for correctness and potential improvement.
* `archived`: The document was once marked as `final`, but is no longer actively monitored or maintained.

**State Transitions:**

* **Forward Progression (`draft` → `final`):** A new, erroneous, or incomplete document is completed.
* **Retirement Cycle (`final` → `archived`):** End of active monitoring and maintenance.
* **Backward Transition (`final` → `draft`):** Re-opened for major revision or corrective authoring.

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

#### Optional fields

| Field | Type | Purpose |
|---|---|---|
| `prosody` | String | Poetic meter pattern |
| `prosody_name` | String | Name of the meter |
| `tags` | Array of Strings | Thematic/search keywords |
| `sources` | Array of Strings | Bibliographic/provenance references |
| `notes` | String | Internal editorial notes |

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

**Syntax for Multi-Word Tags:**
Use **a single YAML string** containing spaces for multi-word tags. This is standard YAML syntax and fits Persian typography naturally.
```yaml
tags:
  - "عشق الهی"
  - "سیر و سلوک"
  - "وحدت وجود"
```

**Persian Typography Guidelines:**
* Use **normal spaces** between separate words: `"عشق الهی"`.
* Use the **Zero-Width Non-Joiner (ZWNJ / نیم‌فاصله)** where Persian orthography expects compound word division: `"می‌خانه"`, `"گفت‌وگو"`, `"دل‌بر"`.

**Normalization Rules:**
* Trim leading and trailing whitespace.
* Collapse repeated internal spaces to a single standard space.
* Standardize on a single canonical spelling (e.g., prefer `سلوک` consistently, avoiding alternate representations unless a distinction is explicitly intended).
* Do not encode multi-word tags with hyphens or underscores (such as `عشق-الهی`). Tags must remain human-readable labels; web presentation systems should generate URL slugs automatically.

##### `sources`
* **Type:** Array of Strings  
* **Required:** No (defaults implicitly to `[]` if omitted)  
* **Purpose:** Documents bibliographic and provenance information for the text, editions, commentaries, manuscripts, and online resources used.

**Design Philosophy:**
* **Simplified Structure:** Each source is a plain string.
* **Contributor Responsibility:** Contributors format citations as appropriate.
* **No Validation Overhead:** Maintainers and automated validation scripts do not parse or enforce strict citation structure.
* **Maximum Flexibility:** Accommodates any citation style, methodology, or local standard (e.g., APA, MLA, Chicago, or custom Persian formats).

**Guidelines for Contributors:**
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

**Style Guide Recommendations:**
System documentation should encourage consistent formatting standards:
* **Online Sources:** `نام منبع: URL (مشاهده: تاریخ)`
* **Books:** `عنوان، ویراستار/مؤلف، ناشر، مکان، سال، صفحات`
* **Manuscripts:** `مخزن، شماره نسخه، تاریخ، شماره برگ`

##### `notes`
* **Type:** String (multi-line supported)  
* **Required:** No  
* **Purpose:** For editorial notes, placeholders, or clarifications. This field is designated for internal workflow tracking and is not displayed publicly.

---

### 20. Document Heading Hierarchy

The document uses the following structure:

``` text
H1
└── H2 (optional)
    └── H3 content block
        ├── H4 متن
        ├── H4 واژگان و ترکیبات (optional)
        ├── H4 معنی روان (optional)
        ├── H4 نکات / Notes (optional)
        └── H4 ارجاعات (optional)
```

#### H1

- Exactly one H1 is required.
- The H1 is the page's primary displayed title.
- It may be more elaborate or stylistically formatted than the concise front-matter `title`.

#### H2

- H2 is optional.
- It provides one level of headings for major subdivisions within a literary document (narrative unit).
- No deeper heading level is permitted for this purpose.

#### H3

Every content block begins with exactly one H3 heading.

The H3 contains only a number or a number range:

``` markdown
### 1
```
or:
``` markdown
### 1-2
```

ASCII numerals and Persian numerals are both valid. ASCII numerals are recommended for authoring. The compiler normalizes Persian digits to ASCII for canonical indexing, sorting, and anchors.

The number identifies the semantic content block rather than necessarily representing a literal couplet count.

Numeric ranges are used for enjambed units or other content blocks spanning multiple sequential source numbers.

------------------------------------------------------------------------

## Part 5: Content Blocks and Excerpts

### 21. Content Block

A **content block** is the fundamental reading-and-recording unit inside a literary document.

Every content block contains:

1.  a mandatory excerpt block;
2.  an optional glossary;
3.  optional meaning;
4.  optional notes;
5.  optional references.

``` text
content block
├── excerpt
├── glossary                optional
├── meaning                 optional
├── notes                   optional
└── references              optional
```

There is no requirement for horizontal rules between content blocks.

The H3 heading itself provides the structural boundary.

### 22. Excerpt

The `متن` H4 section contains the excerpt itself.

Exactly four excerpt types are supported:

| Type | Persian | Definition |
|---|---|---|
| **Prose** | نثر | One or more prose sentences treated as a single excerpt for annotation or without it |
| **Hemistich** | مصرع | A standalone poetic hemistich treated as an independent excerpt |
| **Couplet** | بیت | Two poetic hemistichs constituting a normal couplet |
| **Enjambed unit** | موقوف‌المعانی | A poetic unit whose meaning extends beyond the normal boundary of a couplet or standalone hemistich |

#### 22.1 Prose

Prose may consist of one or more sentences when they are treated as a single reading/annotation unit or without annotation.

#### 22.2 Hemistich

A single poetic hemistich may be represented independently when it is itself the appropriate excerpt.

#### 22.3 Couplet

A normal couplet consists of two hemistichs and is treated as one excerpt.

#### 22.4 Enjambed Unit

An enjambed unit consists of multiple poetic hemistichs whose meaning extends beyond the normal boundary of a couplet or standalone hemistich.

The numeric H3 range identifies the source span represented by the unit.

### 23. Excerpt Formatting

#### Prose

Plain prose paragraph.

#### Hemistich

One bold line:

``` markdown
**text**
```

#### Couplet

Two bold lines separated by one blank line:

``` markdown
**hemistich one**

**hemistich two**
```

#### Enjambed Unit

More than two bold hemistich lines, each separated by one blank line:

``` markdown
**hemistich one**

**hemistich two**

**hemistich three**

**hemistich four**
```

The compiler determines the excerpt type from the physical structure of `متن`. No explicit excerpt-type field is required.

### 24. Optional Content-Block Components

#### `واژگان و ترکیبات` --- Glossary

- Provides explanations of difficult words, phrases, and idioms.
- It uses a Markdown table:
	``` markdown
	| واژه/ترکیب | معنی/توضیح |
	|---:|---|
	| **واژه** | توضیح |
	```

#### `معنی روان` --- Meaning

- Provides a fluent contemporary-Persian rendering of the excerpt.
- It should convey the meaning rather than mechanically reproduce the original word order.

#### `نکات` --- Notes

- Provides optional reading notes, including literary, thematic, stylistic, linguistic, historical, or other observations useful to the reader.
- The broader term **Notes** is preferred to the narrower `نکات ادبی`, because the project's purpose is reading and recording rather than formal literary scholarship.

#### `ارجاعات` --- References

Provides references to related literary passages, sources, or external material.

No rigid citation format is imposed.

---

## Part 6: Contextual Composition

### 25. Mixed Content

- A literary document may contain any sequence of the four supported excerpt types. For example:
	``` text
	prose
	couplet
	couplet
	enjambed unit
	prose
	hemistich
	```
- No document-level `type` declaration is required.
- Each content block is independently classified from its `متن` structure.

### 26. H2 Sections

- H2 headings may group content blocks into one high-level division of a literary document.

- They are optional and do not represent the literary hierarchy of the corpus. Corpus-level structural divisions belong in directories and their `meta.yml` files.

---

## Part 7: Validation Rules

### 27. Corpus and Directory Validation

For every directory under `corpus/`:

-   `meta.yml` must exist.
-   `meta.yml` must contain valid YAML.
-   `schema` must equal `"1.0.0"`.
-   `type` must be valid.
-   `title` must be a non-empty string.
-   `children` must be an array of strings.
-   Every `children` entry must identify an immediate filesystem child.
-   No child may appear more than once.
-   `meta.yml` itself must not appear in `children`.
-   Every child must be permitted by the parent directory's type.
-   Omitted filesystem children are permitted and are excluded from the public literary hierarchy.
-   Unknown metadata fields must not cause validation failure.

#### Directory type validation

``` text
root
└── poet/

poet
└── work/

work
├── structural_division/
└── literary-document.md

structural_division
├── structural_division/
└── literary-document.md
```

At the `work` and `structural_division` levels, directories and Markdown documents may be freely mixed and ordered through `children`.

### 28. Literary Document Validation

A literary document is valid only if:

-   it begins directly with YAML front matter;
-   `schema` is `"1.0.0"`;
-   `title` is present;
-   `status` is valid;
-   at least one contributor is specified;
-   every contributor ID references an existing contributor file;
-   exactly one H1 exists;
-   H2 is optional;
-   every content block begins with an H3;
-   every H3 is a bare number or numeric range;
-   every content block contains `#### متن`;
-   every `متن` conforms to exactly one of the four excerpt types;
-   optional H4 sections use the standardized vocabulary;
-   Setext headings are prohibited;
-   horizontal rules are not required between content blocks.

### 29. Deterministic Compilation

The compiler must derive the same TypeScript representation from the same valid repository state.

The compiler must:

1.  traverse the filesystem;
2.  validate directory types;
3.  read each directory's `meta.yml`;
4.  use `children` to construct the literary hierarchy and canonical order;
5.  ignore filesystem children omitted from `children`;
6.  parse literary documents;
7.  classify each content block from its `#### متن` structure;
8.  normalize Persian numerals in H3 identifiers to ASCII if any;
9.  preserve author-authored Persian text;
10. produce deterministic TypeScript objects.

---

## Part 8: Separation of Concerns

The schema deliberately separates three kinds of information.

### Literary hierarchy

- Stored in:
	1. Physical filesystem hierarchy of the `corpus/` folder.
	2. The mandatory `meta.yml` file inside every sub-folders of the `corpus/`
- and especially the `children` field of `meta.yml`.

This determines where a poet, work, structural division, or literary document belongs and in what order it appears.

### Literary document metadata

- Stored in Markdown front matter.
- This describes the document itself and its editorial state.

### Reading notes

- Stored in content blocks.

- These contain the actual excerpt and the reader's glossary, meaning, notes, and references.

- Information derivable from filesystem structure or Git history should not be duplicated in literary-document front matter.

---

## Part 9: Git Integration

Git provides historical information that does not belong in the literary schema.

For example:

``` bash
git log -1 --format="%ai" -- path/to/file.md
```

can provide the last modification timestamp.

Likewise:

``` bash
git log --follow --format="%h|%ai|%an|%s" -- path/to/file.md
```

can provide revision history.

Such information should be derived at compilation or presentation time rather than manually maintained in literary documents.

---

## Part 10: Complete Example

A simplified corpus may look like:

``` text
corpus/
├── meta.yml
│
├── عطار-نیشابوری/
│   ├── meta.yml
│   │
│   └── منطق-الطیر/
│       ├── meta.yml
│       │
│       ├── مقدمه/
│       │   ├── meta.yml
│       │   ├── مجمع-مرغان.md
│       │   ├── خطاب-هدهد.md
│       │   └── بخش-فرعی/
│       │       ├── meta.yml
│       │       └── حکایت.md
│       │
│       └── هفت-وادی/
│           ├── meta.yml
│           ├── وادی-طلب.md
│           └── وادی-عشق.md
│
└── سعدی/
    ├── meta.yml
    │
    └── گلستان/
        ├── meta.yml
        ├── دیباچه.md
        ├── باب-اول/
        │   ├── meta.yml
        │   ├── حکایت-۱.md
        │   ├── حکایت-۲.md
        │   └── بخش-فرعی/
        │       ├── meta.yml
        │       └── حکایت-۳.md
        ├── باب-دوم/
        │   ├── meta.yml
        │   └── حکایت-۱.md
        └── خاتمه.md
```

The important point is that the physical tree and the literary hierarchy are related but not identical: `meta.yml:children` declares the authoritative literary table of contents at every directory.

---

## Part 11: Canonical Literary Vocabulary

The schema uses the following terminology.

**Literary hierarchy:**
The conceptual hierarchy represented by `corpus/`.

**Poet:**
The top-level literary person represented under `corpus/`.

**Work:**
A distinct literary work.

**Structural division:**
A division used by the particular work to organize itself, such as دفتر, باب, فصل, بخش, or روضه.

**Literary unit:**
A recognizable literary or narrative unit such as حکایت, داستان, غزل, قصیده, رباعی, or دیباچه.

**Literary document:**
The Markdown representation of a literary unit.

**Content block:**
One annotatable or non-annotatable reading unit within a literary document.

**Excerpt:**
The original literary text in a content block.

**Excerpt types:**
1.  Prose --- نثر
2.  Hemistich --- مصرع
3.  Couplet --- بیت
4.  Enjambed unit --- موقوف‌المعانی

---

## Part 12: Schema Version

This specification is **version 1.0.0**.

The version applies uniformly to:

-   corpus directory metadata;
-   poet metadata;
-   work metadata;
-   structural-division metadata;
-   contributor profiles;
-   literary-document front matter;
-   content-block structure.

These components are not independently versioned.
