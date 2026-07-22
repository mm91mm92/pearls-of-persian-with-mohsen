# Pearls of Persian with Mohsen: Presentation Layer Specification

**Presentation Specification Version: 1.0.0**
**Repository Schema Version: 1.0.0**

---

### 1. Purpose and Scope

The presentation layer renders the literary corpus defined by `SCHEMA.md` as a static, readable, accessible website.

The project is a **personal reading-and-recording system** for Persian literary works. The presentation layer therefore prioritizes:

* reading comfort;
* clear literary hierarchy;
* unobtrusive presentation of reading notes;
* Persian-first typography;
* predictable navigation;
* accessibility;
* static-site performance.

The presentation layer does not define or reinterpret the literary hierarchy. It consumes the hierarchy produced by compilation from the filesystem and directory metadata.

The presentation layer must not modify literary content.

---

### 2. Relationship to the Repository Schema

[[SCHEMA]] is the authoritative specification for the repository.

The presentation layer consumes its compiled representation.

The relationship is:

```text
Repository
    │
    ├── corpus/
    ├── contributors/
    ├── COPYRIGHT.md
    └── LICENSE
          │
          ▼
      Compilation
          │
          ├── Literary hierarchy
          ├── Literary documents
          ├── Contributors
          └── Presentation Manifest
                    │
                    ▼
                 Astro
                    │
                    ▼
              Static website
```

The presentation layer must not introduce a competing literary hierarchy.

In particular:

* `corpus/` remains the source of the literary hierarchy;
* `meta.yml:children` remains authoritative for membership and order;
* literary documents remain the source of their own content and metadata;
* Git remains the source of Git-derived historical information;
* the Presentation Manifest describes what the presentation layer needs from the compiled corpus.

The repository schema currently uses version `1.0.0` uniformly across directory metadata, contributor profiles, literary-document front matter, and content-block structure.

---

## 3. Presentation Principles

### 3.1 Reading-first

The presentation exists to support reading.

Typography, spacing, navigation, metadata, and interactive features must remain subordinate to the literary text.

### 3.2 Content-first

The literary content is the primary visual object.

Visual styling must never compete with the excerpt itself.

### 3.3 Persian-first, not Persian-only

Persian typography and RTL layout are first-class requirements.

Latin-script content remains fully supported where required for metadata, URLs, identifiers, code, contributor information, and external links.

### 3.4 Structural fidelity

The presentation must preserve the literary hierarchy produced by compilation.

The order supplied by `children` must never be replaced by alphabetical or implicit filesystem ordering. The schema explicitly makes `children` authoritative for membership and canonical order.

### 3.5 Presentation independence

The content layer should not contain presentation instructions merely to achieve a visual effect.

Conversely, the presentation layer must not infer literary structure from visual appearance when that structure is already defined by the schema.

### 3.6 Progressive disclosure

The excerpt itself is the primary reading object.

Glossaries, meanings, notes, and references are secondary material and should normally be visually subordinate and, where appropriate, collapsible.

### 3.7 Static-first

The core reading experience must work with server-generated/static HTML and without client-side JavaScript.

JavaScript is used only for progressive enhancement such as:

* collapsible controls;
* font-size controls;
* theme selection;
* enhanced search;
* dynamic navigation conveniences.

### 3.8 Accessible by default

Semantic HTML, keyboard accessibility, sufficient focus indication, screen-reader-compatible controls, and correct RTL semantics are mandatory.

No important information may be conveyed by color alone.

### 3.9 Deterministic rendering

The same compiled repository state must produce the same presentation output.

Presentation code must not make arbitrary editorial decisions during rendering.

---

## 4. Non-Goals

The presentation layer is not:

* a CMS;
* an in-browser editing system;
* a database-backed application;
* a social platform;
* a collaborative annotation system;
* a scholarly textual-critical interface;
* a manuscript collation interface;
* a multilingual parallel-text interface;
* a single-page application.

The site does not require client-side routing.

---

## 5. Presentation Manifest

### 5.1 Purpose

The **Presentation Manifest** is the compiled contract between the corpus compiler and the presentation layer.

It contains presentation-oriented information derived from the repository and compilation process.

It is not an additional authoring format.

Authors do not edit the Presentation Manifest manually.

The compiler generates it from:

1. the filesystem;
2. directory `meta.yml` files;
3. literary-document front matter;
4. contributor profiles;
5. literary-document content;
6. Git-derived information where applicable;
7. fixed presentation configuration.

---

### 5.2 Responsibilities

The Presentation Manifest provides the presentation layer with:

* the site identity;
* the compiled literary hierarchy;
* canonical URLs;
* page-level navigation;
* literary-document metadata required for rendering;
* contributor data required for rendering;
* document-level previous/next navigation;
* search-index data or search-index generation inputs;
* presentation configuration and labels.

It must not become a duplicate of the complete repository schema.

---

### 5.3 Literary hierarchy in the manifest

The manifest represents the compiled literary hierarchy using typed TOC nodes.

The hierarchy is:

```text
root
└── poet
    └── work
        ├── literary_document
        └── structural_division
            ├── literary_document
            └── structural_division
                └── literary_document
```

A `work` may directly contain literary documents, structural divisions, or any mixture of them.

A `structural_division` may recursively contain structural divisions and literary documents.

A literary document is always a leaf.

This directly reflects the Directory Inclusivity Rule in the repository schema.

---

### 5.4 Recommended manifest data model

The compiled manifest should use plain serializable objects.

```typescript
export type TocNode =
  | RootNode
  | PoetNode
  | WorkNode
  | StructuralDivisionNode
  | LiteraryDocumentNode;

interface TocNodeBase {
  id: string;
  title: string;
  url: string;
}

export interface RootNode extends TocNodeBase {
  type: "root";
  children: PoetNode[];
}

export interface PoetNode extends TocNodeBase {
  type: "poet";
  children: WorkNode[];
}

export interface WorkNode extends TocNodeBase {
  type: "work";
  children: WorkChild[];
}

export interface StructuralDivisionNode extends TocNodeBase {
  type: "structural_division";
  children: StructuralDivisionChild[];
}

export interface LiteraryDocumentNode extends TocNodeBase {
  type: "literary_document";
}

export type WorkChild =
  | StructuralDivisionNode
  | LiteraryDocumentNode;

export type StructuralDivisionChild =
  | StructuralDivisionNode
  | LiteraryDocumentNode;
```

The manifest should not contain circular `parent` references.

The manifest should not embed all content blocks of every literary document merely to support navigation.

Content blocks belong to literary-document data, not to the literary TOC.

---

### 5.5 Document navigation

Sequential document navigation is derived from the canonical document order of the work.

The compiler or presentation utility should conceptually flatten a work into:

```text
Document 1
Document 2
Document 3
...
Document N
```

using recursive traversal of `children`.

Thus:

```text
work
├── document A
├── division X
│   ├── document B
│   ├── document C
│   └── division Y
│       └── document D
└── document E
```

produces:

```text
A → B → C → D → E
```

The previous/next relationships are therefore:

```text
A.previous = null
A.next     = B

B.previous = A
B.next     = C

C.previous = B
C.next     = D

D.previous = C
D.next     = E

E.previous = D
E.next     = null
```

Filesystem items omitted from `children` do not participate in the sequence.

---

## 6. URL Model

### 6.1 Corpus URLs

Corpus URLs correspond to their position within the literary hierarchy.

The default mapping is:

```text
corpus/<poet>/<work>/...
        ↓
/<poet>/<work>/...
```

Directory pages use their directory path.

Literary documents use their path without the `.md` extension.

Examples:

| Filesystem path | URL |
| --- | --- |
| `corpus/سعدی/` | `/سعدی/` |
| `corpus/سعدی/گلستان/` | `/سعدی/گلستان/` |
| `corpus/سعدی/گلستان/باب-اول/` | `/سعدی/گلستان/باب-اول/` |
| `corpus/سعدی/گلستان/باب-اول/حکایت-۱.md` | `/سعدی/گلستان/باب-اول/حکایت-۱/` |

The public URL is separate from the repository-relative identifier.

---

### 6.2 Trailing slash

Canonical site URLs use trailing slashes.

```text
/سعدی/گلستان/
```

is canonical rather than:

```text
/سعدی/گلستان
```

The implementation should configure the static hosting layer and Astro routing consistently with this convention.

---

### 6.3 URL encoding

Persian filesystem names may appear as Persian path segments.

Browsers and HTTP clients may URL-encode these characters internally.

The displayed navigation labels must remain human-readable Persian titles rather than percent-encoded strings.

---

## 7. Site-wide Navigation

### 7.1 Site Header

`<SiteHeader>` appears throughout the site.

It provides:

* site identity;
* link to the home page;
* global search;
* theme control;
* font-size controls where enabled.

The header must remain visually subordinate to the literary content on reading pages.

---

### 7.2 Site Footer

`<SiteFooter>` provides:

* contributor-directory link;
* copyright information;
* license information where appropriate;
* repository link;
* repository schema version.

The displayed schema version must come from the compiled repository schema version rather than being independently hard-coded in multiple templates.

For example:

```text
تهیه‌شده با طرح نسخهٔ ۱.۰.۰
```

---

## 8. Breadcrumbs

`<Breadcrumbs>` provides the reader with the current location within the literary hierarchy.

Example:

```text
گوهرهای پارسی با محسن  > سعدی > گلستان > باب اول > حکایت اول
```

The breadcrumb trail is generated from the compiled hierarchy.

Rules:

1. The site title is the first segment.
2. Intermediate corpus nodes link to their pages.
3. The current page is the final segment.
4. The current page is not linked.
5. The current page uses `aria-current="page"`.
6. Titles come from compiled titles, not filesystem names.

Example:

```html
<nav aria-label="مسیر صفحه">
  <ol class="ppwm-breadcrumbs">
    <li>
      <a href="/">گوهرهای پارسی با محسن</a>
    </li>

    <li>
      <a href="/سعدی/">سعدی</a>
    </li>

    <li>
      <a href="/سعدی/گلستان/">گلستان</a>
    </li>

    <li>
      <a href="/سعدی/گلستان/باب-اول/">باب اول</a>
    </li>

    <li aria-current="page">
      حکایت اول
    </li>
  </ol>
</nav>
```

---

## 9. Literary Hierarchy Navigation

### 9.1 Sidebar Tree

`<SidebarTree>` displays the current work's literary hierarchy.

It must use the compiled TOC rather than independently scanning the filesystem.

The tree must preserve the exact order represented by `children`.

It must distinguish visually between:

* structural divisions;
* literary documents.

Structural divisions and literary documents must remain conceptually distinct even when both appear as links in the same tree.

---

### 9.2 Expansion

Nodes containing children may be collapsible.

The current path must be expanded sufficiently to expose the current page.

Other branches may be collapsed by default.

The collapse mechanism must remain usable without destroying basic navigation when JavaScript is unavailable.

---

### 9.3 Work Table of Contents

A work page displays the work's literary hierarchy.

Example:

```text
گلستان

1. دیباچه
2. باب اول
   2.1 حکایت اول
   2.2 حکایت دوم
   2.3 بخش فرعی
       2.3.1 حکایت سوم
3. باب دوم
   3.1 حکایت اول
4. خاتمه
```

The numbering shown here is **presentation-generated numbering**.

It is not part of the repository hierarchy and must not be written into `meta.yml`.

---

## 10. Page Types

The presentation layer recognizes the following principal page types:

1. Home page
2. Poet page
3. Work page
4. Structural-division page
5. Literary-document page
6. Contributors directory
7. Contributor profile
8. Search page
9. Copyright page
10. License page
11. About/documentation pages where implemented

---

## 11. Home Page

### 11.1 Source

The site root corresponds to:

```text
corpus/meta.yml
```

and the compiled root presentation data.

### 11.2 URL

```text
/
```

### 11.3 Layout

Uses `<RootLayout>`. The `<MiddleContainer>` is divided into the following sub-containers:

*   **Banner:** Site introduction, project description, and goals.
*   **Recent Additions:** The 5 most recently updated or added literary documents (derived from build metadata or Git history).
*   **Favorites:** A curated list of featured or highly visited works.
*   **Directories:** Navigation access points to top-level poets, books, and genres.

**Layout on small screens:**
```text
┌────────────────────┐
│    SiteHeader      │
├────────────────────┤
│       Banner       │
├────────────────────┤
│                    │
│  Recent Additions  │
│                    │
├────────────────────┤
│                    │
│         Favorites  │
│                    │
├────────────────────┤
│                    │
│       Directories  │
│                    │
├────────────────────┤
│    SiteFooter      │
└────────────────────┘
```

**Layout on ordinary/wide screens:**
```text
┌─────────────────────────────────┐
│            SiteHeader           │
├─────────────────────────────────┤
│              Banner             │
├───────────────┬─────────────────┤
│               ┃                 │
│    Favorites  ┃         Recent  │
│               ┃      Additions  │
│               ┃                 │
├───────────────┴─────────────────┤
│                                 │
│                    Directories  │
│                                 │
├─────────────────────────────────┤
│            SiteFooter           │
└─────────────────────────────────┘
```


"Recent" information should be derived from compilation/Git data rather than manually maintained front matter.

The home page must not invent literary hierarchy that is absent from the corpus.

---

## 12. Poet Page

### 12.1 Source

```text
corpus/<poet>/meta.yml
```

Like:

```text
/عطار-نیشابوری/
```

### 12.2 Presentation

Uses `<FullNavLayout>`. The `<SidebarTree>` displays the poet's location within the `corpus/` hierarchy. The `<MainContainer>` consists of the following sub-containers, stacked vertically regardless of screen size:
    *   **Title:** The poet's name (`title`) and biography (`description`).
    *   **Works List:** Ordered list of literary works derived from the directory's `children` array, hyperlinking to their respective work pages.

**Layout on small screens:**
```text
┌─────────────────────────┐
│        SiteHeader       │
├─────────────────────────┤
│            Breadcrumbs  │
├─────────────────────────│
│                  Title  │
├─────────────────────────│
│                         │
│                  Works  │
│                         │
├─────────────────────────┤
│        SiteFooter       │
└─────────────────────────┘
```

**Layout on ordinary/wide screens:**
```text
┌────────────────────────────────────────┐
│               SiteHeader               │
├─────────────────────────┬──────────────┤
│            Breadcrumbs  │              │
├─────────────────────────┤              │
│                  Title  │              │
├─────────────────────────┤  SidebarTree │
│                         │              │
│                  Works  │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

---

## 13. Work Page

### 13.1 Source

```text
corpus/<poet>/<work>/meta.yml
```

Example:

```text
/عطار-نیشابوری/منطق-الطیر/
```

### 13.2 Presentation

Uses `<FullNavLayout>`. The `<SidebarTree>` represents the work's location within the literary hierarchy, and the `<MainContainer>` consists of the following stacked sub-containers:
    *   **Title:** The work title, parent poet name as subtitle, and the `description` metadata (if present).
    *   **TOC:** A hierarchical, variable-depth structural tree of links showing all available structural divisions alongside with the ultimate narrative units. The TOC is generated from the work's compiled literary hierarchy. The presentation must not assume that a work begins with a structural division. A work may have:
```text
work
├── literary document
├── structural division
└── literary document
```
	because the repository schema explicitly permits such mixtures.

**Layout on small screens:**
```text
┌─────────────────────────┐
│        SiteHeader       │
├─────────────────────────┤
│            Breadcrumbs  │
├─────────────────────────│
│                  Title  │
├─────────────────────────│
│                         │
│                    TOC  │
│                         │
├─────────────────────────┤
│        SiteFooter       │
└─────────────────────────┘
```

**Layout on ordinary/wide screens:**
```text
┌────────────────────────────────────────┐
│               SiteHeader               │
├─────────────────────────┬──────────────┤
│            Breadcrumbs  │              │
├─────────────────────────┤              │
│                  Title  │              │
├─────────────────────────┤  SidebarTree │
│                         │              │
│                    TOC  │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

---

## 14. Structural-Division Page

### 14.1 Source

```text
corpus/<poet>/<work>/<structural_divisions>/meta.yml
```

Structural divisions may occur at arbitrary depth.

For example:

```text
work
└── باب
    └── فصل
        └── بخش
            └── literary document
```

The presentation layer must not impose a maximum depth.

### 14.2 Presentation

Uses `<FullNavLayout>`. The `<SidebarTree>` displays the structural division's location within the literary hierarchy. There might be any number of levels of structural divisions, while some literary works do not have such divisions and their TOC is a flat list of narrative units. The `<MainContainer>` consists of the following stacked sub-containers:
    *   **Title:** The section title; a multiline subtitle of the poet name, and book title, the parent structural division(s) title (if any); and the `description` metadata as the description.
    *   **TOC:** A list or hierarchy depending on whether this structural division has any further structural division as its children:
	      * **List:** If a structural division contains only literary documents, its TOC may be presented as a simple list, exposes the ultimate literary documents on this branch.
	      * **Hierarchy:** If it contains further structural divisions, the TOC is presented as a nested hierarchy.

**Layout on small screens:**
```text
┌─────────────────────────┐
│        SiteHeader       │
├─────────────────────────┤
│            Breadcrumbs  │
├─────────────────────────│
│                  Title  │
├─────────────────────────│
│                         │
│                    TOC  │
│                         │
├─────────────────────────┤
│        SiteFooter       │
└─────────────────────────┘
```

**Layout on ordinary/wide screens:**
```text
┌────────────────────────────────────────┐
│               SiteHeader               │
├─────────────────────────┬──────────────┤
│            Breadcrumbs  │              │
├─────────────────────────┤              │
│                  Title  │              │
├─────────────────────────┤  SidebarTree │
│                         │              │
│                    TOC  │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

---

## 15. Literary Document Page

### 15.1 Source

A Markdown literary document containing:

* YAML front matter;
* one H1;
* optional H2 sections;
* one or more content blocks.

This corresponds directly to the literary-document structure defined by the repository schema.

---

### 15.2 Page structure

Uses `<FullNavLayout>`. The `<SidebarTree>` represents the document's location within the hierarchy. The `<MainContainer>` container is divided into the following sub-containers:
    *   **Title:** Renders the page's only H1 heading.
    *   **Metadata:** A vertical card component displaying status, active contributors, prosody, and sources.
    *   **Table of Contents:** Auto-generated link list constructed solely from H2 headings (omitted if no H2 headings exist).
    *   **Content container:** Vertically stacks the H2 section headings and standard `<ContentBlock>` blocks.
    *   **Button Bar:** Renders "Previous" and "Next" sequential navigation buttons, alongside an edit link pointing to the source file in the remote content repository.

**Layout on Small Screens:**
```text
┌─────────────────────────────────┐
│           SiteHeader            │
├─────────────────────────────────┤
│                    Breadcrumbs  │
├─────────────────────────────────┤
│                          Title  │
├─────────────────────────────────┤
│                       Metadata  │
├─────────────────────────────────┤
│                            TOC  │
├─────────────────────────────────┤
│                                 │
│                       Contents  │
│                                 │
├─────────────────────────────────┤
│           Button bar            │
├─────────────────────────────────┤
│           SiteFooter            │
└─────────────────────────────────┘
```

**Layout on ordinary/wide screens:**
```text
┌──────────────────────────────────────────────────────────────┐
│                        SiteHeader                            │
├──────────────────────────────────────┬───────────────────────┤
│                         Breadcrumbs  │                       │
├──────────────────────────────────────┤                       │
│                               Title  │                       │
├──────────────────────────────────────┤                       │
│                            Metadata  │                       │
├──────────────────────────────────────┤          SidebarTree  │
│                                 TOC  │                       │
├──────────────────────────────────────┤                       │
│                                      │                       │
│                            Contents  │                       │
│                                      │                       │
├──────────────────────────────────────┤                       │
│             Buttons bar              │                       │
├──────────────────────────────────────┴───────────────────────┤
│                        SiteFooter                            │
└──────────────────────────────────────────────────────────────┘
```

The literary content remains the visual focus.

---

## 16. Document Title

The page title is taken from the document's H1.

The front-matter `title` is the concise metadata title used for navigation and hierarchy-related presentation.

The two values may differ because the schema explicitly permits the H1 to be more elaborate than the front-matter title.

---

## 17. Document Metadata

The metadata component displays information from the literary document's front matter and, where appropriate, compilation-derived information.

The repository schema defines:

```text
status
contributors
prosody
prosody_name
tags
sources
notes
```

as the relevant document metadata.

The presentation layer must not invent additional repository metadata fields merely for display convenience.

Git-derived information may be displayed when available.

For example:

```text
آخرین به‌روزرسانی
```

may be generated from Git history.

The repository schema explicitly prefers deriving such historical information from Git rather than manually storing it in literary documents.

---

## 18. Metadata Presentation

Metadata should normally be presented in a collapsible panel.

Recommended behavior:

```text
جزئیات                                      ▾
────────────────────────────────────────────
وضعیت             تکمیل‌شده
مشارکت‌کنندگان    محسن ملک‌محمدی (محقق)
وزن عروضی         ...
برچسب‌ها           ...
منابع             ...
```

The panel may begin collapsed on literary-document pages.

It must remain accessible by keyboard and screen reader.

A status indicator must use text or another semantic indication in addition to color.

---

## 19. Document-Level Table of Contents

The document-level TOC is distinct from the literary hierarchy.

It is generated only from H2 headings inside the Markdown document.

The distinction is:

```text
Literary hierarchy
    corpus → poet → work → structural division → literary document

Document TOC
    H1 → H2
```

H1 is not included in the document TOC.

If there are no H2 headings, the document TOC is omitted.

The document TOC must never be confused with the work's literary hierarchy.

---

## 20. Content Blocks

A content block is the fundamental reading-and-recording unit within a literary document.

The repository schema requires:

```text
H3
├── H4 متن
├── H4 واژگان و ترکیبات
├── H4 معنی روان
├── H4 نکات
└── H4 ارجاعات
```

with optional annotation sections.

The presentation layer renders these blocks without changing their semantic structure.

There is no requirement for horizontal rules between content blocks.

The H3 boundary and spacing provide the visual separation.

---

## 21. Content-Block Presentation

When annotations exist, the primary excerpt remains visible while the annotations are presented through progressive disclosure.

Conceptually:

```text
[ 75   متن اصلی ◄]
```

```text
[ 75   متن اصلی ▼]

واژگان و ترکیبات
...

معنی روان
...

نکات
...

ارجاعات
...
```

When no annotations exist:

```text
75   متن اصلی
```

The presentation must not force readers to open an annotation panel merely to read the literary text.

---

## 22. Excerpt Rendering

The schema defines exactly four excerpt types:

| English | فارسی |
|---|---|
| Prose | نثر |
| Hemistich | مصرع |
| Couplet | بیت |
| Enjambed unit | موقوف‌المعانی |

The presentation layer must therefore use the same four-way classification.

No separate `verse` type is used for `بیت`.

No separate `enjambed passage` type exists in schema version 1.0.0.

The excerpt type is determined during compilation from the physical structure of `متن`; it is not authored as a separate field.

---

## 23. Prose

Prose is rendered as ordinary Persian reading text.

```html
<p class="ppwm-excerpt ppwm-excerpt-prose">
  ...
</p>
```

The presentation should optimize:

* line length;
* line height;
* paragraph spacing;
* comfortable Persian reading.

---

## 24. Hemistich

A standalone hemistich is rendered as a centered poetic line.

```html
<div class="ppwm-excerpt ppwm-excerpt-hemistich">
  <span>...</span>
</div>
```

---

## 25. Couplet

A couplet consists of two poetic hemistichs.

On sufficiently wide screens, the two hemistichs may be displayed in two columns:

```text
          مصرع دوم        مصرع اول
```

The first hemistich occupies the right side in RTL presentation.

On narrow screens, the two hemistichs may stack vertically.

The presentation must preserve their semantic order regardless of visual layout.

---

## 26. Enjambed Unit

An enjambed unit contains multiple poetic hemistichs whose meaning extends beyond the normal boundary of a couplet or standalone hemistich.

The compiler provides the excerpt's classified structure.

The presentation renders the individual hemistichs while making their belonging to one reading unit visually clear.

The presentation must not falsely break the unit into independent couplets merely because the physical rendering happens to use two-column rows.

---

## 27. Glossary

The `واژگان و ترکیبات` section is rendered as a semantic HTML table.

```html
<table>
  <tbody>
    <tr>
      <th>واژه/ترکیب</th>
      <td>معنی/توضیح</td>
    </tr>
  </tbody>
</table>
```

The glossary is secondary to the excerpt and should normally appear inside the annotation area.

The left column must be left-aligned while the left column is supposed to appear right-aligned.

---

## 28. Meaning

The `معنی روان` section displays the reader's fluent contemporary-Persian rendering.

It should be visually distinguishable from the original excerpt while remaining subordinate to it.

The presentation must not label this material as a scholarly translation unless the underlying content explicitly establishes that function.

---

## 29. Notes

The `نکات` section displays optional reading notes.

Notes may concern:

* literary observations;
* themes;
* language;
* style;
* history;
* context;
* other observations useful to the reader.

The presentation should use the generic concept of notes rather than implying that all notes are formal literary scholarship.

It is very suitable that this annotation section appear as an ordered/numbered list.

---

## 30. References

The `ارجاعات` section displays references supplied by the literary document.

No particular bibliographic rendering style is imposed by the presentation layer unless a future schema version specifies one.

It is very suitable that this annotation section appear as an ordered/numbered list.

---

## 31. Previous / Next Navigation

Literary-document pages provide sequential navigation through the containing work.

Recommended presentation:

```text
پیشین: <عنوان_سند_ادبی_قبلی>                       پسین: <عنوان_سند_ادبی_بعدی>ی
```

The exact visual arrangement must respect RTL semantics.

Navigation is based on the canonical literary sequence derived from `children`.

It therefore crosses structural-division boundaries automatically.

For example:

```text
دیباچه
باب اول
    حکایت ۱
    حکایت ۲
    بخش فرعی
        حکایت ۳
باب دوم
    حکایت ۱
خاتمه
```

produces:

```text
دیباچه
↓
حکایت ۱
↓
حکایت ۲
↓
حکایت ۳
↓
حکایت ۱ باب دوم
↓
خاتمه
```

This is preferable to implementing navigation as a collection of special cases for individual directory depths.

---

## 32. Reading Progress

The presentation may display the current document's position within its work.

For example:

```text
۳ از ۴۸
```

The position is based on the flattened canonical sequence of literary documents.

It must not be calculated from filesystem enumeration order.

---

## 33. Contributors Directory Page

### 33.1 Source

```text
Project root
│
└── contributors/
```

### 33.2 URL

```text
/contributors/
```

The page lists contributors according to the presentation's configured ordering.

Contributor profiles are separate from the literary hierarchy.

### 33.3 Layout

It uses `<SemiNavLayout>`. The `<Breadcrumbs>` displays: `گوهرهای پارسی با محسن ‹ مشارکت‌کنندگان`. The final segment (`مشارکت‌کنندگان`) is unlinked. The `<MainContainer>` renders the global alphabetical list of contributors after project maintainers at the top of the list.

```text
┌────────────────────────────┐
│         SiteHeader         │
├────────────────────────────┤
│               Breadcrumbs  │
├────────────────────────────┤
│                            │
│              Contributors  │
│                            │
├────────────────────────────┤
│        SiteFooter          │
└────────────────────────────┘
```


---

## 34. Contributor Profile

### 34.1 Source

```text
contributors/<id>.yml
```

### 34.2 URL

```text
/contributors/<id>/
```

A contributor profile may contain:

1. identity information;
2. contact and external links;
3. biography and affiliation;
4. contributions to the project.

The presentation should distinguish Persian and Latin representations where both are available.

---

## 35. Contributor References

A literary document references contributors by ID and role.

The presentation resolves those IDs against compiled contributor profiles.

For example:

```text
مشارکت‌کنندگان
محسن ملک‌محمدی (محقق)
```

The contributor's displayed name should link to the corresponding contributor profile.

---

## 36. Search

Search is a build-time generated static index.

The index may include:

* poet titles;
* work titles;
* structural-division titles;
* literary-document navigation titles;
* literary-document H1 titles;
* directory tags;
* document tags;
* excerpt text;
* fluent meanings.

Search should respect the distinction between original literary text and reading notes.

The index should not unnecessarily include internal editorial notes.

The search interface may use JavaScript for enhanced interaction, but the search architecture must remain compatible with a static site.

---

## 37. Site Routes

The principal routes are:

| Route                            | Purpose                     |
| -------------------------------- | --------------------------- |
| `/`                              | Home page                   |
| `/contributors/`                 | Contributors directory      |
| `/contributors/<id>/`            | Contributor profile         |
| `/search/`                       | Search                      |
| `/about/`                        | About/project documentation |
| `/copyright/`                    | Copyright information       |
| `/license/`                      | License information         |
| `/<poet>/`                       | Poet page                   |
| `/<poet>/<work>/`                | Work page                   |
| `/<poet>/<work>/.../`            | Structural-division page    |
| `/<poet>/<work>/.../<document>/` | Literary-document page      |

The exact route-generation implementation belongs to Astro rather than to the repository schema.

---

## 38. Layout System

The site uses three principal layout levels.

### 38.1 `<RootLayout>`

Provides:

```text
SiteHeader
Main content
SiteFooter
```

```text
┌────────────────────────┐
│      SiteHeader        │
├────────────────────────┤
│                        │
│      Main content      │
│                        │
├────────────────────────┤
│      SiteFooter        │
└────────────────────────┘
```

---

### 38.2 `<SemiNavLayout>`

Adds breadcrumbs:

```text
┌────────────────────────┐
│      SiteHeader        │
├────────────────────────┤
│       Breadcrumbs      │
├────────────────────────┤
│                        │
│      Main content      │
│                        │
├────────────────────────┤
│      SiteFooter        │
└────────────────────────┘
```

---

### 38.3 `<FullNavLayout>`

Adds the literary hierarchy sidebar.

On wide screens:

```text
┌────────────────────────────────────────┐
│              SiteHeader                │
├─────────────────────────┬──────────────┤
│       Breadcrumbs       │              │
├─────────────────────────┤              │
│                         │              │
│      Main content       │ SidebarTree  │
│                         │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

On small screens, the sidebar becomes a collapsible navigation drawer.

---

## 39. Responsive Behavior

The site is designed for three broad presentation conditions:

* small screens;
* ordinary desktop screens;
* wide desktop screens.

The implementation should use responsive CSS rather than separate mobile and desktop page implementations.

The literary text must remain readable at every width.

For example:

```css
:root {
  --ppwm-breakpoint-mobile: 48rem;
}
```

Logical CSS properties should be preferred:

```css
margin-inline-start
margin-inline-end
padding-inline
text-align: start
text-align: end
```

rather than hard-coding left/right behavior wherever possible.

---

## 40. Typography

### 40.1 Persian font stack

The primary font stack should support Persian and Arabic script.

A suitable starting point is:

```css
font-family:
  "Vazirmatn",
  "Noto Naskh Arabic",
  "Traditional Arabic",
  system-ui,
  sans-serif;
```

The exact final font choice is an implementation decision and should not be treated as part of the repository schema.

---

### 40.2 Latin font stack

Latin-script interface elements may use:

```css
font-family:
  "Inter",
  system-ui,
  sans-serif;
```

---

### 40.3 Monospace

Identifiers, code, Git information, and schema values may use:

```css
font-family:
  "Fira Code",
  "Cascadia Code",
  monospace;
```

---

### 40.4 Line height

The presentation should begin with approximately:

```text
Poetry:  2.0
Prose:   1.8
UI:      1.5
```

These are presentation defaults, not schema constraints.

---

### 40.5 Fluid font sizes

Font sizes should use CSS `clamp()` where appropriate so that reading text scales smoothly across viewport sizes.

Suggested variables include:

```text
--ppwm-ftsz-h1
--ppwm-ftsz-h2
--ppwm-ftsz-excerpt
--ppwm-ftsz-text
--ppwm-ftsz-explanation
--ppwm-ftsz-metadata
--ppwm-ftsz-crumb
```

The exact values belong to the implementation stylesheet.

---

## 41. RTL Semantics

The root document must use:

```html
<html lang="fa" dir="rtl">
```

Persian literary content is rendered in RTL.

However, bidirectional content such as:

* URLs;
* email addresses;
* Git identifiers;
* Latin names;
* code;
* technical identifiers

must be rendered with appropriate bidi isolation where necessary.

The presentation must not assume that every string is Persian merely because the page is RTL.

---

## 42. Theme

The site may support light and dark themes.

Theme selection is a presentation concern and must not affect repository content.

The preferred implementation is CSS custom properties with a theme selector such as:

```css
[data-theme="dark"] {
  ...
}
```

The theme control is progressive enhancement and must not prevent reading when JavaScript is unavailable.

---

## 43. Color

The visual language may draw subtle inspiration from Persian manuscript traditions, but the presentation should avoid turning the site into a decorative imitation of a manuscript.

The primary goal is readable contemporary presentation of classical literature.

Color must not be the sole means of conveying:

* workflow status;
* links;
* focus;
* warnings;
* hierarchy.

---

## 44. Accessibility

The presentation must provide:

* semantic landmarks;
* meaningful heading hierarchy;
* keyboard-accessible controls;
* visible focus indicators;
* accessible labels for icon-only controls;
* semantic tables for glossaries;
* ordered/unordered lists for lists;
* `aria-current` for current navigation;
* appropriate button semantics for expandable controls;
* sufficient contrast;
* no information conveyed solely by color.

The content hierarchy must remain understandable without CSS.

---

## 45. Progressive Enhancement

Interactive features must degrade gracefully.

For example:

| Feature             | Without JavaScript | With JavaScript               |
| ------------------- | ------------------ | ----------------------------- |
| Literary navigation | Fully usable       | Enhanced                      |
| Breadcrumbs         | Fully usable       | Unchanged                     |
| TOC links           | Fully usable       | Smooth scrolling may be added |
| Metadata            | Visible            | Collapsible                   |
| Annotations         | Visible/readable   | Collapsible                   |
| Sidebar             | Navigable          | Expand/collapse               |
| Theme               | Default theme      | User-selectable               |
| Font size           | Default size       | User-adjustable               |
| Search              | Static/search page | Enhanced interaction          |

JavaScript must not be required for basic literary reading.

---

## 46. Component Data Types

The presentation layer should distinguish **domain data** from **component props**.

For example:

```typescript
export interface Excerpt {
  type: "prose" | "hemistich" | "couplet" | "enjambed";
  lines: string[];
}
```

Component-specific types use the `Props` suffix:

```typescript
export interface ContentBlockProps {
  number: string;
  excerpt: Excerpt;
  glossary?: GlossaryProps;
  meaning?: string;
  notes?: string[];
  references?: string[];
}
```

This distinction prevents component APIs from becoming confused with corpus-domain types.

---

## 47. Core Component Types

Recommended component interfaces include:

```typescript
export interface ContentBlockProps {
  number: string;
  excerpt: Excerpt;
  glossary?: GlossaryProps;
  meaning?: string;
  notes?: string[];
  references?: string[];
}

export interface GlossaryEntry {
  term: string;
  explanation: string;
}

export interface GlossaryProps {
  entries: GlossaryEntry[];
}

export interface TitleProps {
  title: string;
  subtitle?: string;
  description?: string;
}

export interface BreadcrumbSegment {
  title: string;
  href: string;
  current?: boolean;
}

export interface BreadcrumbsProps {
  segments: BreadcrumbSegment[];
}

export interface TocProps {
  root: TocNode;
}

export interface MetadataProps {
  metadata: DocumentMetadata;
}

export interface FontSizeControlProps {
  minSize?: number;
  maxSize?: number;
  step?: number;
}

export interface CopyLinkProps {
  targetUrl?: string;
  label?: string;
}
```

---

## 48. Document Metadata Type

The presentation representation of document metadata should correspond to the repository schema rather than inventing a parallel metadata model.

For example:

```typescript
export interface DocumentMetadata {
  status: "draft" | "final" | "archived";

  contributors: Array<{
    contributor: Contributor;
    role: string;
  }>;

  prosody?: string;
  prosody_name?: string;
  tags?: string[];
  sources?: string[];
  notes?: string;

  lastModified?: string;
}
```

`lastModified` is presentation/compilation data derived from Git and is not a literary-document front-matter field.

A checksum should not be displayed as required document metadata unless a future specification explicitly introduces it.

---

## 49. Contributor Type

The presentation representation of a contributor may be:

```typescript
export interface Contributor {
  id: string;
  name: string;
  name_fa: string;
  email: string;

  affiliation?: string;
  affiliation_fa?: string;

  website?: string;
  location?: string;

  bio?: string;
  bio_fa?: string;

  preferred_roles?: string[];

  links?: Record<string, string>;
}
```

The presentation layer may additionally derive a list of contributions for a contributor profile by searching compiled literary-document references.

That derived list is not written into contributor YAML.

---

## 50. Content Block Component

`<ContentBlock>` renders one `###` content block.

Its responsibilities are:

1. display the block identifier;
2. render the excerpt;
3. display optional annotations;
4. preserve the order of the authored material.

It must not:

* reorder content;
* invent annotations;
* classify content independently of the compiler;
* add horizontal rules merely to separate blocks.

---

## 51. Excerpt Component

`<Excerpt>` renders the classified excerpt.

Its implementation branches on:

```typescript
excerpt.type
```

using:

```text
prose
hemistich
couplet
enjambed
```

The component should not use `verse` as the internal name for `couplet`.

---

## 52. Breadcrumbs Component

`<Breadcrumbs>` renders the compiled breadcrumb segments.

The component does not inspect the filesystem.

Its sole responsibility is presentation.

---

## 53. TOC Component

`<TocBlock>` renders the compiled literary hierarchy.

It must:

* preserve child order;
* support arbitrary structural-division depth;
* distinguish structural divisions from literary documents;
* link each node to its canonical URL;
* optionally collapse branches;
* identify the current node.

The TOC component must not create its own ordering algorithm.

---

## 54. Metadata Component

`<MetadataBlock>` renders document metadata.

Recommended behavior:

* collapsed by default;
* expandable through a semantic button;
* two-column presentation on sufficiently wide screens;
* one-column presentation on small screens.

For RTL presentation, labels may be visually left-aligned while values are right-aligned where this improves scanability.

---

## 55. Contributor Component

`<ContributorBlock>` renders a contributor profile.

A contributor profile may be divided into:

#### Identity and biography

Includes:

* Persian and Latin names;
* location;
* affiliation;
* biography.

#### Links

Includes:

* email;
* website;
* external links from `links`.

#### Contributions

Displays the contributor's project contributions, grouped by role where such derived information is available.

The component should not require the contributor YAML file to contain a manually maintained list of contributions.

---

## 56. Header Components

`<SiteHeader>` contains:

* branding;
* global search;
* theme control;
* font-size controls.

The header should remain compact enough not to dominate reading pages.

---

## 57. Footer Components

`<SiteFooter>` contains:

* contributors link;
* repository link;
* copyright link;
* license link where appropriate;
* schema-version indicator.

The active repository schema version is supplied by the Presentation Manifest.

---

## 58. Copy-Link Control

`<CopyLink>` may copy:

* the current page URL;
* a specific content-block URL;
* another canonical presentation URL.

It must remain a progressive enhancement.

A normal hyperlink must remain available wherever copying a link would otherwise be the only means of access.

---

## 59. Search Index Manifest Data

The Presentation Manifest may expose search-index entries such as:

```typescript
export interface SearchEntry {
  id: string;
  url: string;
  title: string;
  breadcrumb: BreadcrumbSegment[];
  tags?: string[];
  text?: string;
}
```

The exact search engine/indexing implementation is independent of the repository schema.

---

## 60. Presentation Configuration

Presentation-specific configuration should be kept separate from literary content.

Examples include:

```typescript
export interface PresentationConfig {
  siteTitle: string;
  defaultTheme: "light" | "dark" | "system";
  enableSearch: boolean;
  enableThemeToggle: boolean;
  enableFontSizeControl: boolean;
}
```

These values are not literary metadata.

They should not be added to `corpus/meta.yml` merely because the presentation layer needs them.

---

## 61. Compilation Boundary

The compiler is responsible for transforming repository information into presentation-ready data.

The boundary should conceptually be:

```text
SCHEMA
  │
  ▼
Repository compiler
  │
  ├── validate
  ├── construct literary hierarchy
  ├── parse documents
  ├── resolve contributors
  ├── classify excerpts
  ├── derive navigation
  └── generate Presentation Manifest
          │
          ▼
       Astro
```

Astro components should consume the compiled objects rather than independently reconstructing repository semantics.

---

## 62. Determinism Requirements

Given the same valid repository state, compilation must produce the same Presentation Manifest.

In particular:

* child ordering must be deterministic;
* URLs must be deterministic;
* document IDs must be deterministic;
* previous/next navigation must be deterministic;
* breadcrumb paths must be deterministic;
* search-index entries must be deterministic.

No presentation component should depend on incidental filesystem enumeration order.

---

## 63. Error Handling

Invalid repository states should be rejected during compilation rather than silently repaired by the presentation layer.

Examples include:

* missing `meta.yml`;
* invalid directory type;
* invalid `children`;
* duplicate children;
* nonexistent child references;
* invalid literary-document structure;
* unresolved contributor IDs.

The presentation layer should assume that the objects it receives have already passed schema validation.

---

## 64. Separation of Data Responsibilities

The project maintains three important boundaries:

```text
Literary hierarchy
        │
        ▼
corpus/ + meta.yml:children

Literary document
        │
        ▼
Markdown + front matter + content blocks

Presentation
        │
        ▼
Presentation Manifest + Astro components
```

The same information should not be redundantly authored in more than one of these layers.

---

## 65. Versioning

The repository schema and presentation specification are related but distinct.

The repository schema is currently:

```text
1.0.0
```

and applies uniformly to the repository's schema components.

The Presentation Specification has its own version because it describes the website implementation rather than the repository data format.

The current values are therefore:

```text
Repository Schema:       1.0.0
Presentation Specification: 1.0.0
```

A change to the presentation layer does not automatically constitute a change to the repository schema.

Conversely, a repository-schema change may require corresponding presentation changes.

---

## 66. Implementation Principle

The desired implementation can be summarized as:

> **The repository describes the literature; the compiler describes the compiled corpus; the Presentation Manifest describes what the website needs; Astro describes how it is displayed.**

The presentation layer should therefore remain deliberately thin.

It should not become:

* another content authoring system;
* another hierarchy database;
* another metadata schema;
* or another source of literary truth.

Its purpose is simply to make the compiled corpus pleasant to read, navigate, search, and revisit.

---

## 67. Summary Architecture

```text
┌──────────────────────────────────────────┐
│              Repository                  │
│                                          │
│  corpus/                                 │
│  contributors/                           │
│  COPYRIGHT.md                            │
│  LICENSE                                 │
└─────────────────────┬────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────┐
│                Compiler                  │
│                                          │
│  validation                              │
│  hierarchy construction                  │
│  document parsing                        │
│  excerpt classification                  │
│  contributor resolution                  │
│  Git-derived information                 │
│  navigation derivation                   │
└─────────────────────┬────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────┐
│         Presentation Manifest             │
│                                          │
│  literary TOC                            │
│  URLs                                     │
│  breadcrumbs                              │
│  document navigation                      │
│  document presentation data               │
│  contributors                             │
│  search data                              │
│  presentation configuration               │
└─────────────────────┬────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────┐
│                  Astro                   │
│                                          │
│  layouts                                  │
│  components                               │
│  CSS                                      │
│  progressive enhancement                  │
└─────────────────────┬────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────┐
│             Static Website               │
│                                          │
│  readable                                │
│  navigable                               │
│  accessible                              │
│  Persian-first                           │
│  fast                                    │
└──────────────────────────────────────────┘
```

**End of Presentation Layer Specification v1.0.0**
