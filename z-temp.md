# Pearls of Persian with Mohsen: Presentation Layer Specification v1.0.0

---

## 1. Design Principles

### 1.1 Core Philosophy
The presentation layer serves classical Persian literature with the same academic rigor and clarity that the content layer preserves it. Every design decision prioritizes **readability, structural consistency, and timeless aesthetics** over ephemeral trends.

### 1.2 Guiding Principles

| Principle | Description |
|-----------|-------------|
| **Content First** | Typography, spacing, and hierarchy exist to serve the literary text. Visual styling must never compete with or distract from the reading experience. |
| **Scholarly Precision** | Metadata, sources, and contributors are treated as first-class information. Citation and attribution are prioritized, not marginalized. |
| **Timeless Aesthetics** | The visual design draws inspiration from classical Persian manuscript traditions, utilizing warm, readable backdrops, generous margins, and subtle, intentional accents. |
| **Static-First Performance** | Pages must load rapidly. The core reading experience must function with zero client-side JavaScript. JavaScript is reserved for progressive enhancement. |
| **Accessible by Default** | Semantic HTML, comprehensive right-to-left (RTL) layout support, keyboard navigation, and screen reader compatibility are mandatory core features. |
| **Separation of Concerns** | The presentation layer is strictly a **deterministic rendering engine**. Content must never be edited, generated, or modified within this layer. |
| **Progressive Disclosure** | The primary literary text (`متن`) is the focal point. Explanatory annotations are consolidated into a collapsed-by-default container to optimize readability and minimize cognitive load. |
| **Strict Rendering From Schema**| Document structure is rendered precisely as defined. Optional sections are rendered only if present; no placeholder blocks are displayed. |

### 1.3 Non-Goals
* **No Client-Side Routing:** The site utilizes static-first HTML pages, avoiding heavy app-shell routing or complex state hydration.
* **No Content Management System (CMS):** The system does not support in-browser editing, administrative login panels, or database writes. All content changes must occur in the repository.
* **No Social Layer:** The application excludes social interactions such as comments, user profiles, or likes. Contributors represent structured metadata, not interactive user accounts.

---

### 1.4 Page

**HTML:**

```html
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title></title>
</head>
  <body>
  </body>
</html>
```

**CSS:**

```css
:root {
  /* Define breakpoints once. Using 'rem' is recommended as it scales
     with the user's base font size, unlike 'px'. 48rem = 768px at default 16px. */
  --ppwm-breakpoint-mobile: 48rem;
}
```

**Jinja2:**
Environment variables:
```
labels: Record<string, string> = {
	draft: "پیش‌نویس",
	final: "تکمیل‌شده",
	archived: "بایگانی‌شده",
  
	glossary: "واژگان و ترکیبات",
	meaning: "معنی روان",
	literaryNotes: "نکات ادبی",
	references: "ارجاعات",
	
	metadata: "جزئیات",
	lastModified: "آخرین به‌روزرسانی",
	checksum: "شناسه‌رمزنگاری",
	status: "وضعیت",
	contributors: "مشارکت‌کنندگان",
	prosody: "وزن عروضی",
	tags: "برچسب‌ها",
	sources: "منابع",
}
```

---

## 2. URL Structure

### 2.1 Canonical Mapping
URLs map directly to physical filesystem paths under the `content/` directory, utilizing Persian-script slugs derived from directory and file names.

- **Directory Page:** Corresponds to a directory containing `meta.yml`
  - URL format: `/<path-to-directory>/`
  - Example: `/عطار-نیشابوری/منطق-الطیر/`
- **Literary Document Page:** Corresponds to a Markdown file (`*.md`)
  - URL format: `/<path-to-file-without-extension>/`
  - Example: `/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان/`

### 2.2 Encoding
* Path segments are derived from directory or file names and are URL-encoded by the web server.
* The user interface must display human-readable, decoded Persian titles sourced from:
  * `meta.yml:title` for directory levels.
  * Front matter `title` (navigation label) and document H1 (page display title) for literary files.

### 2.3 Canonical URL and Trailing Slash
* **Trailing Slashes are Canonical:** All canonical URLs must end with a trailing slash (e.g., `/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان/`).
* **Enforced Redirection:** Requests without a trailing slash (e.g., `/عطار-نیشابوری/منطق-الطیر`) must perform a permanent 301 redirect to the canonical trailing slash path.
* **Case-Sensitivity:** URLs are evaluated with case-sensitivity to accommodate Persian character mappings.

### 2.4 Special Routes

| Route | Purpose |
|-------|---------|
| `/` | Site root: Project introduction, featured works, and collections. |
| `/search/` | Full-text search interface. |
| `/contributors/` | Alphabetical directory of all project contributors. |
| `/contributors/<id>/` | Individual contributor profile detail page. |
| `/about/` | Project mission, schema documentation, and guidelines. |

---

## 3. Navigation

### 3.1 Breadcrumbs
Breadcrumbs are displayed at the top of every page (excluding the Home page), positioned below the site header and above the page title. They are constructed dynamically from the directory hierarchy using `meta.yml:title` definitions.

**Visual Format (RTL):**
```text
گوهرهای پارسی با محسن  ‹  عطار نیشابوری  ‹  منطق‌الطیر  ‹  مقدمه  ‹  مجمع مرغان
```

**Semantic HTML:**
```html
<nav aria-label="مسیر صفحه">
  <ol class="ppwm-breadcrumbs">
    <li><a href="/">گوهرهای پارسی با محسن</a></li>
    <li><a href="/عطار-نیشابوری/">عطار نیشابوری</a></li>
    <li><a href="/عطار-نیشابوری/منطق-الطیر/">منطق‌الطیر</a></li>
    <li><a href="/عطار-نیشابوری/منطق-الطیر/مقدمه/">مقدمه</a></li>
    <li aria-current="page">مجمع مرغان</li>
  </ol>
</nav>
```

**Rules:**
* The root segment (`/`) always maps to the site title defined in the root `meta.yml`.
* Intermediate segments display their corresponding `meta.yml:title` values and link to their index pages.
* The current page segment represents the final breadcrumb item. It must not be an active link and must feature `aria-current="page"`.
* The breadcrumb separator is a consistent RTL character (such as `‹` or `/`) padded with standard spacing.

### 3.2 Sidebar Navigation (Tree)
On directory index pages and individual document pages, a collapsible, responsive sidebar displays the canonical structure of the repository and the location of the current page within it.

**Rules:**
* The sidebar must render elements in the exact order defined by the `meta.yml:children` array. Alphabetical or filesystem-implicit sorting is prohibited.
* The current location within the hierarchy must be highlighted and auto-expanded to display immediate children.
* Other expandable nodes must remain collapsed by default.
* Clicking a directory node navigates to its index page; clicking a document node navigates to the literary document page.

### 3.3 Next / Previous Navigation
For literary document pages nested within a book directory, sequential navigation controls are rendered at the bottom of the page to allow readers to traverse the work in its canonical sequence.

**Traversing Logic:**
* **Standard Traversal:** "Next" navigates to the next sibling in the parent directory’s `meta.yml:children` array.
* **Structural Boundaries:** If the current document is the final child in its section's `children` array, "Next" must navigate to the first document of the subsequent section folder (obeying the order of sections defined in the book's `meta.yml:children` array). This recursive traversal logic applies across arbitrary nested directory depths.
* **Filtering:** Hidden filesystem items omitted from `children` arrays are completely bypassed during navigation generation.

### 3.4 Table of Contents (Document-Level)
Each literary document page displays an auto-generated table of contents constructed solely from the H2 headings in the Markdown body.

**Rules:**
* H1, H3, and H4 headings are ignored by the Table of Contents.
* Clicking a TOC anchor must trigger a smooth scroll to the corresponding section.
* The active H2 section must be dynamically highlighted in the TOC as the reader scrolls through the page.

---

## 4. Page Layouts & Types

### 4.1 Layout Scaffolding Components
The layout scaffolding ensures a consistent Header and Footer across all pages.

#### `<RootLayout>`
Divides the page grid into three primary vertical sections:
1.  `<SiteHeader>` (Branding and search)
2.  `<MiddleContainer>` (Primary page slot)
3.  `<SiteFooter>` (Copyright and links)

```text
┌─────────────────┐
│   SiteHeader    │
├─────────────────┤
│                 │
│ MiddleContainer │
│                 │
├─────────────────┤
│   SiteFooter    │
└─────────────────┘
```

This layout remains unchanged on viewports from small to wide.

#### `<SemiNavLayout>`
Extends `<RootLayout>` and inserts breadcrumbs navigation at the top of the `<MiddleContainer>`:
```text
┌─────────────────┐
│   SiteHeader    │
├─────────────────┤
│   Breadcrumbs   │
├─────────────────┤
│                 │
│  MainContainer  │
│                 │
├─────────────────┤
│   SiteFooter    │
└─────────────────┘
```

This layout remains unchanged on viewports from small to wide.

#### `<FullNavLayout>`
Extends `<SemiNavLayout>` and docks `<SidebarTree>` to the right of the `<Breadcrumbs>` and `<MainContainer>`. The sidebar collapses into a slide-out drawer on small viewports.

**Layout on small screens:**
```text
┌────────────────────────────┐
│         SiteHeader         │
├────────────────────────────┤
│        Breadcrumbs         │
├────────────────────────────┤
│                            │
│       MainContainer        │
│                            │
├────────────────────────────┤
│        SiteFooter          │
└────────────────────────────┘
```

**Layout on ordinary/wide screens:**
```text
┌────────────────────────────────────────┐
│               SiteHeader               │
├─────────────────────────┬──────────────┤
│       Breadcrumbs       │              │
├─────────────────────────┤              │
│                         │              │
│                         │ SidebarTree  │
│     MainContainer       │              │
│                         │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

---

The `<SiteHeader>` and `<SiteFooter>` components remain persistent across the entire site and render the following structural information:

**`<SiteHeader>`:**
* Site logo, site title, and branding (the logo redirects to the Home page).
* Dynamic search input box.
* Dark/Light theme toggle button.
* Font size adjustment controls: Buttons to incrementally increase or decrease font size.

**`<SiteFooter>`:** 
* A link to the contributors directory page.
* The active compiler schema version (e.g., `"تهیه‌شده با طرح نسخهٔ ۱.۰.۰"`).
* The copyright notice and a link pointing directly to the copyright document in the remote repository.
* A link to the repository on GitHub.

---

### 4.2 Home Page (Root Index)
*   **Source:** `content/meta.yml`
*   **URL:** `/`
*   **Layout:** Uses `<RootLayout>`. The `<MiddleContainer>` is divided into the following sub-containers:
    *   **Banner:** Site introduction, project description, and scholarly goals.
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
│ Recent Additions   │
│                    │
├────────────────────┤
│                    │
│ Favorites          │
│                    │
├────────────────────┤
│                    │
│ Directories        │
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
│ Favorites     ┃ Recent          │
│               ┃ Additions       │
│               ┃                 │
├───────────────┴─────────────────┤
│                                 │
│ Directories                     │
│                                 │
├─────────────────────────────────┤
│            SiteFooter           │
└─────────────────────────────────┘
```

---

### 4.3 Contributors Directory Page
*   **Source:** `contributors/`
*   **URL:** `/contributors/`
*   **Layout:** Uses `<SemiNavLayout>`. The `<Breadcrumbs>` displays: `گوهرهای پارسی با محسن ‹ مشارکت‌کنندگان`. The final segment (`مشارکت‌کنندگان`) is unlinked. The `<MainContainer>` renders the global alphabetical directory of project contributors.

```text
┌────────────────────────────┐
│         SiteHeader         │
├────────────────────────────┐
│        Breadcrumbs         │
├────────────────────────────┤
│                            │
│ Contributors               │
│                            │
├────────────────────────────┤
│        SiteFooter          │
└────────────────────────────┘
```

---

### 4.4 Contributor Profile Page
*   **Source:** `contributors/<id>.yml`
*   **URL:** `/contributors/mm91mm92/`
*   **Layout:** Uses `<SemiNavLayout>`. The `<Breadcrumbs>` displays: `گوهرهای پارسی با محسن ‹ مشارکت‌کنندگان ‹ <id>`, where the final segment is unlinked. The `<MainContainer>` is split into four distinct sub-container:
    *   **Title Container:** Large Persian name, smaller Latin name, professional location, and institutional affiliation.
    *   **Biography Container:** Persian biography text, accompanied by an optional English translation (collapsed by default).
    *   **Links Container:** Standardized external profiles (GitHub, LinkedIn, ORCID, etc.) rendered from the `links` flat dictionary.
    *   **Contributions Container:** A list of all literary documents in the repository associated with the contributor, grouped by their assigned role.

**Layout on small screens:**
```text
┌────────────────────────────┐
│         SiteHeader         │
├────────────────────────────┤
│        Breadcrumbs         │
├────────────────────────────┤
│ Title                      │
├────────────────────────────┤
│ Biography                  │
├────────────────────────────┤
│ Links                      │
├────────────────────────────┤
│ Contributions              │
├────────────────────────────┤
│        SiteFooter          │
└────────────────────────────┘
```

**Layout on ordinary/wide screens:**
```text
┌────────────────────────────┐
│         SiteHeader         │
├────────────────────────────┤
│        Breadcrumbs         │
├────────────────────────────┤
│ Title                      │
├──────────────┬─────────────┤
│ Biography    │ Links       │
├──────────────┴─────────────┤
│ Contributions              │
├────────────────────────────┤
│        SiteFooter          │
└────────────────────────────┘
```

---

### 4.5 Poet Page
*   **Source:** `content/<poet>/meta.yml`
*   **URL:** `/عطار-نیشابوری/`
*   **Layout:** Uses `<FullNavLayout>`. The `<SidebarTree>` displays the poet's location within the `content/` hierarchy. The `<MainContainer>` consists of the following sub-containers, stacked vertically regardless of screen size:
    *   **Title:** The poet's name (`title`) and biography (`description`).
    *   **Works List:** Ordered list of literary works derived from the directory's `children` array, linking to their respective book pages.

**Layout on small screens:**
```text
┌─────────────────────────┐
│        SiteHeader       │
├─────────────────────────┤
│       Breadcrumbs       │
├─────────────────────────│
│          Title          │
├─────────────────────────│
│                         │
│          Works          │
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
│       Breadcrumbs       │              │
├─────────────────────────┤              │
│          Title          │              │
├─────────────────────────┤ SidebarTree  │
│                         │              │
│          Works          │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

---

### 4.6 Book Page
*   **Source:** `content/<poet>/<book>/meta.yml`
*   **URL:** `/عطار-نیشابوری/منطق-الطیر/`
*   **Layout:** Uses `<FullNavLayout>`. The `<SidebarTree>` represents the book's location within the hierarchy, and the `<MainContainer>` consists of the following stacked components:
    *   **Title:** The book title, parent poet name as subtitle, and the `description` metadata (if present).
    *   **TOC:** A hierarchical, variable-depth structural tree of links showing chapters, offices, or sections, along with their respective document links.

**Layout on small screens:**
```text
┌─────────────────────────┐
│        SiteHeader       │
├─────────────────────────┤
│       Breadcrumbs       │
├─────────────────────────│
│          Title          │
├─────────────────────────│
│                         │
│           TOC           │
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
│       Breadcrumbs       │              │
├─────────────────────────┤              │
│          Title          │              │
├─────────────────────────┤ SidebarTree  │
│                         │              │
│           TOC           │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

---

### 4.7 Section / Chapter / Office Page
*   **Source:** `content/<poet>/<book>/<section>/meta.yml`
*   **URL:** `/عطار-نیشابوری/منطق-الطیر/مقدمه/`
*   **Layout:** Uses `<FullNavLayout>`. The `<SidebarTree>` displays the section's location within the hierarchy. The `<MainContainer>` consists of the following stacked components:
    *   **Title:** The section title; a multiline subtitle of the parent section(s) title (if any), the poet name, and book title; and the `description` metadata as description.
    *   **TOC:** A list or hierarchy depending on whether this section/chapter/office has some section/chapter/office under it:
      * **List:** there is no further section/chapter/office and a list exposes the literary documents under it.
      * **Hierarchy:** there is at least one more level of section/chapter/office, so a hierarchy exhibits them alongside the ultimate literary documents.

**Layout on small screens:**
```text
┌─────────────────────────┐
│        SiteHeader       │
├─────────────────────────┤
│       Breadcrumbs       │
├─────────────────────────│
│          Title          │
├─────────────────────────│
│                         │
│           TOC           │
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
│       Breadcrumbs       │              │
├─────────────────────────┤              │
│          Title          │              │
├─────────────────────────┤ SidebarTree  │
│                         │              │
│           TOC           │              │
│                         │              │
├─────────────────────────┴──────────────┤
│              SiteFooter                │
└────────────────────────────────────────┘
```

---

### 4.8 Literary Document Page
*   **Source:** A `.md` file containing front matter, an H1 heading, and prose/verse blocks interspersed with H2 headings.
*   **URL:** `/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان/`
*   **Layout:** Uses `<FullNavLayout>`. The `<SidebarTree>` represents the document's location within the hierarchy. The `<MainContainer>` consists of the following components:
    *   **Title:** Renders the page's primary H1 heading.
    *   **Metadata:** A vertical [[#6.7 Document Metadata Component (`<MetadataBlock>`)|card component]] displaying status, active contributors, prosody, and sources.
    *   **Table of Contents:** Auto-generated link list constructed solely from H2 headings (omitted if no H2 headings exist).
    *   **Literary Division:** Vertically stacks the H2 section headings and standard `<ProseVerseBlock>` elements.
    *   **Button Bar:** Renders "Previous" and "Next" sequential navigation buttons, alongside an edit link pointing to the source file in the remote content repository.

**Layout on Small Screens:**
```text
┌─────────────────────────────────┐
│           SiteHeader            │
├─────────────────────────────────┤
│           Breadcrumbs           │
├─────────────────────────────────┤
│              Title              │
├─────────────────────────────────┤
│            Metadata             │
├─────────────────────────────────┤
│               TOC               │
├─────────────────────────────────┤
│                                 │
│        Literary Division        │
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
│              Breadcrumbs             │                       │
├──────────────────────────────────────┤                       │
│                Title                 │                       │
├──────────────────────────────────────┤                       │
│              Metadata                │                       │
├──────────────────────────────────────┤      SidebarTree      │
│                TOC                   │                       │
├──────────────────────────────────────┤                       │
│                                      │                       │
│              Literary                │                       │
│                                      │                       │
├──────────────────────────────────────┤                       │
│             Buttons bar              │                       │
├──────────────────────────────────────┴───────────────────────┤
│                        SiteFooter                            │
└──────────────────────────────────────────────────────────────┘
```

---

### 4.9 Search Results Page
*   **URL:** `/search/?q=عشق+الهی`
*   **Layout:** Uses `<RootLayout>`.
*   **Content Sections:**
    *   **Search Box:** Input pre-filled with the active query.
    *   **Results Banner:** Displays matching result counts using Persian numerals.
    *   **Filtering Controls:** Interactive toggles to filter search results by poet, book, tags, or workflow status.
    *   **Results List:** Cards displaying document titles, complete breadcrumb paths, highlighted matching content snippets, and metadata tags.

---

## 5. Data Types

### 5.1. `Url` 

Bundles and exposes useful information about the URL at hand:

```typescript
export interface Url {
  text: string;
  
  '404' | 'site' | 'content' getType();
}
```

* **`getType`:** Specifies the type of this URL: 
	* **`content`:** The URL refers to a poet, a literary work, a section/chapter/office, or a literary document.
	* **`404`:** The URL is not points to an existing resource on the website.
	* **`site`:**

### 5.1 `TocItem` & `Toc`
Provides the recursive, typed structure representing the multi-level, non-uniform table of contents used throughout directory indexes and navigational blocks.
```typescript
export interface TocItem {
  title: string;
  url: string;
  children?: TocItem[];
}

export type Toc {
  children: TocItem[];
  
  TocItem next(TocItem);
  TocItem prev(TocItem);
}
```

`next` and `prev` returns the next and previous TOC item (heading) which is typographically written in a book: 
* **Next heading:** the next sibling if it is not the last children, if it is the last children, the next sibling of its nearest ancestor whose is not the last sibling.
* **Previous heading:** the same logic applies here.

### 5.2 `Contributor`
Binds all data associated with a single project contributor. The unique identifier is extracted from the profile filename.
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
  links?: Record<string, string>; // Flat platform-to-string dictionary map
}
```

### 5.3 `Metadata`
Binds all metadata fields associated with an individual literary document, including front matter fields and compilation variables derived from Git history.
```typescript
export interface Metadata {
  checksum: string;
  lastModified: string;
  status: 'draft' | 'final' | 'archived';
  contributors: Array<{ contributor: Contributor; role: string }>;
  prosody?: string;
  prosody_name?: string;
  tags?: string[];
  sources?: string[];
  notes?: string;
}
```

### 5.4. `ExcerptProps`
Properties required to format the core text segment of a literary block.
```typescript
export interface ExcerptProps {
  type: 'prose' | 'hemistich' | 'verse' | 'enjambed';
  lines: string[];
}
```

### 5.5. `GlossaryProps` & `GlossaryEntry`
Defines glossary mapping for annotated prose/verse blocks.
```typescript
export interface GlossaryEntry {
  term: string;
  explanation: string;
}

export interface GlossaryProps {
  entries: GlossaryEntry[];
}
```

### 5.6. `ProseVerseProps`
Properties defining an individual numbered section of text and its corresponding collapsible explanations.
```typescript
export interface ProseVerseProps {
  number: string;
  excerpt: ExcerptProps;
  glossary?: GlossaryProps;
  meaning?: string;
  literaryNotes?: string[];
  references?: string[];
}
```

### 5.7. `TitleProps`
Properties for configuring the page title header component.
```typescript
export interface TitleProps {
  title: string;
  subtitle?: string;
  description?: string;
}
```

### 5.8. `BreadcrumbsProps`
Properties containing segment mappings for the hierarchical breadcrumb track.
```typescript
export interface BreadcrumbSegment {
  title: string;
  href: string;
}

export interface BreadcrumbsProps {
  segments: BreadcrumbSegment[];
}
```

### 5.9. `TocProps`
Properties for the dynamic page table of contents.
```typescript
export interface TocProps {
  toc: Toc;
}
```

### 5.10. `MetadataProps`
Properties required to display a document's historical, cryptographic, and scholarly metadata. The optional `startCollapsed` specifies whether the component must be rendered in a collapsed state.
```typescript
export interface MetadataProps {
  metadata: Metadata;
  startCollapsed?: boolean;
}
```

### 5.11. `FontSizeControlProps`
Properties configuration for font adjustment panels.
```typescript
export interface FontSizeControlProps {
  minSize?: number;
  maxSize?: number;
  step?: number;
}
```

### 5.12. `CopyLinkProps`
Properties for configure share and copy permalinks.
```typescript
export interface CopyLinkProps {
  targetUrl?: string;
  label?: string;
}
```

### 5.13. `Poet`

Packs all the information about a poet from the `content/` folder:

```typescript
export interface Poet {
  name: string;
  description?: string;
  tags?: string[];
  books: Book[];
}
```

### 5.14. `Book`

Packs all the information about a book from the `content/` folder:

```typescript
export interface Book {
  title: string;
  description?: string;
  tags?: string[];
  toc: Toc;
}
```

The most important property is `toc`, all the other properties is a simple reflect from the corresponding `meta.yml` file in the `content/`. `toc` is actually the filesystem under the book which is visible through `meta.yml:children`.

---

## 6. Component Specifications

### 6.1 Prose/Verse Block Component (`<ProseVerseBlock>`)
*   **Purpose:** Renders an individual annotated prose or poetry section (`### <number>`).
*   **Progressive Disclosure:** Exposes annotations within a collapsible container closed by default to prevent cognitive fatigue.
*   **Props:** `ProseVerseProps`
*   **Jinja2 / Astro Template:**
```jinja2
{#
This template uses the following environment and context variables:
environment = {
  labels: Record<string, string>;
}
context = {
  cb: ProseVerseProps;
}
#}

<div class="ppwm-content-block" id="block-{{ cb.number }}">
  {% if cb.glossary or cb.meaning or cb.literaryNotes or cb.references %}
    <details class="ppwm-annotations-details">
      <summary class="ppwm-annotations-summary">
        <span class="ppwm-block-number-badge">{{ cb.number }}</span>
        <span class="ppwm-excerpt-inline">
          {% include "ExcerptBlock" with cb.excerpt %}
        </span>
      </summary>
      
      <div class="ppwm-annotations-content">
        {% if cb.glossary %}
          <div class="ppwm-lit-glossary">
            <h4 class="ppwm-lit-h4">{{ labels.glossary }}</h4>
            {% include "GlossaryBlock" with cb.glossary %}
          </div>
        {% endif %}
        
        {% if cb.meaning %}
          <div class="ppwm-lit-meaning">
            <h4 class="ppwm-lit-h4">{{ labels.meaning }}</h4>
            <p class="ppwm-prose-text">{{ cb.meaning }}</p>
          </div>
        {% endif %}
        
        {% if cb.literaryNotes %}
          <div class="ppwm-lit-notes">
            <h4 class="ppwm-lit-h4">{{ labels.literaryNotes }}</h4>
            <ol class="ppwm-notes-list">
              {% for note in cb.literaryNotes %}
                <li>{{ note }}</li>
              {% endfor %}
            </ol>
          </div>
        {% endif %}
        
        {% if cb.references %}
          <div class="ppwm-lit-refs">
            <h4 class="ppwm-lit-h4">{{ labels.references }}</h4>
            <ul class="ppwm-refs-list">
              {% for ref in cb.references %}
                <li>{{ ref }}</li>
              {% endfor %}
            </ul>
          </div>
        {% endif %}
      </div>
    </details>
  {% else %}
    <div class="ppwm-excerpt-standalone-wrapper">
      <span class="ppwm-block-number-badge">{{ cb.number }}</span>
      <span class="ppwm-excerpt-standalone">
        {% include "ExcerptBlock" with cb.excerpt %}
      </span>
    </div>
  {% endif %}
</div>
```

---

### 6.2 Excerpt Component (`<ExcerptBlock>`)
*   **Purpose:** Renders the compulsory text section (`متن`), dynamically determining prose vs. poetic structural alignment.
*   **Props:** `ExcerptProps`
*   **Jinja2 / Astro Template:**
```jinja2
{#
This template uses the following environment and context variables:
environment = {
  labels: Record<string, string>;
}
context = {
  eb: ExcerptBlock;
}
#}

{% if eb.type == 'prose' %}
  <p class="ppwm-prose-excerpt">{{ eb.lines[0] }}</p>
{% elif eb.type == 'hemistich' %}
  <div class="ppwm-hemistich-excerpt">
    <span class="ppwm-hemistich-line">{{ eb.lines[0] }}</span>
  </div>
{% elif eb.type == 'verse' %}
  <div class="ppwm-verse-excerpt">
    <div class="ppwm-couplet">
      <span class="ppwm-hemistich ppwm-hemistich-odd">{{ eb.lines[0] }}</span>
      <span class="ppwm-hemistich ppwm-hemistich-even">{{ eb.lines[1] }}</span>
    </div>
  </div>
{% elif eb.type == 'enjambed' %}
  <div class="ppwm-enjambed-excerpt">
    {% set nVerses = (eb.lines|length // 2) %}
    {% for i in range(0, nVerses) %}
      <div class="ppwm-couplet">
        <span class="ppwm-hemistich ppwm-hemistich-odd">{{ eb.lines[i * 2] }}</span>
        <span class="ppwm-hemistich ppwm-hemistich-even">{{ eb.lines[(i * 2) + 1] }}</span>
      </div>
    {% endfor %}
    {% if eb.lines|length % 2 != 0 %}
      <div class="ppwm-trailing-hemistich">
        <span class="ppwm-hemistich ppwm-hemistich-odd">{{ eb.lines|last }}</span>
      </div>
    {% endif %}
  </div>
{% endif %}
```

*   **CSS Variable Integration:**
```css
/* =========================================
   Base CSS Variable Integration
   ========================================= */
.ppwm-prose-excerpt,
.ppwm-hemistich-line,
.ppwm-hemistich {
  font-size: var(--ppwm-ftsz-excerpt);
}

/* =========================================
   1. Prose
   ========================================= */
.ppwm-prose-excerpt {
  text-align: right;
  line-height: 1.8; /* Prose-optimized line-height for Persian text */
  font-weight: normal;
  margin: 0;
}

/* =========================================
   2. Hemistich
   ========================================= */
.ppwm-hemistich-excerpt {
  text-align: center;
  margin: 0;
}

.ppwm-hemistich-line {
  font-weight: bold;
  display: block;
}

/* =========================================
   3. Verse
   ========================================= */
.ppwm-verse-excerpt {
  margin: 0;
}

.ppwm-couplet {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem; /* Balanced gap for traditional Persian verse */
  direction: rtl; /* Ensures correct column ordering (1st=right, 2nd=left) */
}

.ppwm-hemistich {
  display: block;
}

/* 
  Note: Implemented exactly as requested: 
  - First hemistich (right column) is left-aligned.
  - Second hemistich (left column) is right-aligned.
  (If standard "outward" alignment is preferred, simply swap 'left' and 'right' below)
*/
.ppwm-hemistich-odd {
  text-align: left;
}

.ppwm-hemistich-even {
  text-align: right;
}

/* =========================================
   4. Enjambed
   ========================================= */
.ppwm-enjambed-excerpt {
  display: flex;
  flex-direction: column;
}

.ppwm-enjambed-excerpt .ppwm-couplet {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  direction: rtl;
  margin-bottom: 1rem; /* Explicit margin-bottom separates distinct verse rows */
}

.ppwm-enjambed-excerpt .ppwm-trailing-hemistich {
  grid-column: 1 / -1;
  margin-top: 0.5rem;
}

/* Center the trailing hemistich on wide viewports */
.ppwm-enjambed-excerpt .ppwm-trailing-hemistich .ppwm-hemistich {
  text-align: center;
}

/* =========================================
   Mobile Viewports (Stacked Vertical Layout)
   ========================================= */
@media (max-width: 768px) {
  .ppwm-couplet,
  .ppwm-enjambed-excerpt .ppwm-couplet {
    grid-template-columns: 1fr;
    gap: 0.5rem;
  }

  /* Center text for stacked single-column poetry readability */
  .ppwm-hemistich-odd,
  .ppwm-hemistich-even,
  .ppwm-enjambed-excerpt .ppwm-hemistich-odd,
  .ppwm-enjambed-excerpt .ppwm-hemistich-even {
    text-align: center;
  }

  .ppwm-enjambed-excerpt .ppwm-couplet {
    margin-bottom: 1.5rem; /* Slightly larger separation on mobile for clarity */
  }

  .ppwm-enjambed-excerpt .ppwm-trailing-hemistich .ppwm-hemistich {
    text-align: center;
    margin-top: 0.25rem;
  }
}
```

---

### 6.3 Glossary Component (`<GlossaryBlock>`)
*   **Purpose:** Renders word definitions in a standard glossary table.
*   **Props:** `GlossaryProps`
*   **Jinja2 / Astro Template:**
```jinja2
{#
This template uses the following context variables:
context = {
  entries: GlossaryEntry[];
}
#}

<div class="ppwm-glossary-block">
  <table class="ppwm-glossary-table">
    <tbody>
      {% for entry in entries %}
        <tr>
          <th class="ppwm-glossary-term">{{ entry.term }}</th>
          <td class="ppwm-glossary-explanation">{{ entry.explanation }}</td>
        </tr>
      {% endfor %}
    </tbody>
  </table>
</div>
```
*   **CSS Variable Integration:**
```css
.ppwm-glossary-term {
  font-size: var(--ppwm-ftsz-text);
  font-weight: bold;
}
.ppwm-glossary-explanation {
  font-size: var(--ppwm-ftsz-text);
}
```

---

### 6.4 Title Component (`<TitleBlock>`)
*   **Purpose:** Standardizes structural and typographic rendering of all page titles, subtitles, and descriptions.
*   **Props:** `TitleProps`
*   **Jinja2 / Astro Template:**
```jinja2
{#
This template uses the following context variables:
context = {
  tb: TitleProps;
}
#}
<div class="ppwm-title-block">
  <h1 class="ppwm-title-h1">{{ tb.title }}</h1>
  {% if tb.subtitle %}
    <p class="ppwm-title-subtitle">{{ tb.subtitle }}</p>
  {% endif %}
  {% if tb.description %}
    <p class="ppwm-title-description">{{ tb.description }}</p>
  {% endif %}
</div>
```
*   **CSS Variable Sizing Map:**
```css
.ppwm-title-h1 { font-size: var(--ppwm-ftsz-h1); }
.ppwm-title-subtitle { font-size: var(--ppwm-ftsz-h2); }
.ppwm-title-description { font-size: var(--ppwm-ftsz-text); }
```

---

### 6.5 Breadcrumbs Component (`<Breadcrumbs>`)
*   **Purpose:** Contextual hierarchical navigation path.
*   **Props:** `BreadcrumbsProps`
*   **Jinja2 / Astro Template:**
```jinja2
{#
This template uses the following environment and context variables:
environment = {
  labels: Record<string, string>;
}
context = {
  bcp: BreadcrumbsProps;
}
#}

<nav aria-label="مسیر صفحه">
  <ol class="ppwm-breadcrumbs">
    {% for segment in bcp.segments %}
      {% if loop.last %}
        <li aria-current="page" class="ppwm-breadcrumb-active">{{ segment.title }}</li>
      {% else %}
        <li>
          <a href="{{ segment.href }}" class="ppwm-breadcrumb-link">{{ segment.title }}</a>
        </li>
      {% endif %}
    {% endfor %}
  </ol>
</nav>
```
*   **CSS Variable Sizing:**
```css
.ppwm-breadcrumbs li {
  font-size: var(--ppwm-ftsz-crumb);
}
```

---

### 6.6 Table of Content Component (`<TocBlock>`)
**Purpose:** Renders dynamic recursive tree directories.

**Props:** `TocProps`

**Jinja2 / Astro Template:**
```jinja2
{#
This template uses the following environment and context variables:

environment = {
  labels: Record<string, string>;
}

context = {
  toc: TocProps;
}
#}

{% macro render_toc_list(items, prefix='', level=1) %}
  <ul class="ppwm-toc-list" data-level="{{ level }}">
    {% for item in items %}
      {% set number = prefix ~ loop.index %}
      {% set safe_id = number | replace('.', '-') %}
      {% set has_children = item.children is defined and item.children %}

      <li class="ppwm-toc-item{% if has_children %} ppwm-toc-has-children{% endif %}">
        <div class="ppwm-toc-row">

          <a href="{{ item.url }}" class="ppwm-toc-link">
            <span class="ppwm-toc-number">{{ number }}</span>
            <span class="ppwm-toc-title">{{ item.title }}</span>
          </a>

          {% if has_children %}
            <button
              type="button"
              class="ppwm-toc-toggle"
              aria-expanded="true"
              aria-controls="ppwm-toc-sub-{{ safe_id }}"
              aria-label="{{ environment.labels.toggle_section or 'Toggle section' }}"
            >
              <svg
                class="ppwm-toc-caret"
                viewBox="0 0 16 16"
                aria-hidden="true"
              >
                <path
                  d="M4 6l4 4 4-4"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="1.5"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                />
              </svg>
            </button>
          {% endif %}

        </div>

        {% if has_children %}
          <div
            class="ppwm-toc-children"
            id="ppwm-toc-sub-{{ safe_id }}"
          >
            {{ render_toc_list(
              item.children,
              number ~ '.',
              level + 1
            ) }}
          </div>
        {% endif %}

      </li>
    {% endfor %}
  </ul>
{% endmacro %}


<div class="ppwm-toc-block">
  {{ render_toc_list(toc) }}
</div>
```

**CSS Variable Sizing:**
```css
.ppwm-toc-list {
  list-style: none;
  margin: 0;
  padding-left: 0;
}

.ppwm-toc-list:not([data-level="1"]) {
  padding-left: 1.25em;
}

.ppwm-toc-row {
  display: flex;
  align-items: center;
  gap: 0.4em;
}

.ppwm-toc-link {
  display: inline-flex;
  gap: 0.5em;
  min-width: 0;
  flex: 1;

  font-size: var(--ppwm-ftsz-text);
  text-decoration: none;
}

.ppwm-toc-link:hover {
  text-decoration: underline;
}

.ppwm-toc-number {
  flex-shrink: 0;

  color: var(--ppwm-clr-toc-number, currentColor);
  opacity: 0.65;
  font-variant-numeric: tabular-nums;
}

.ppwm-toc-toggle {
  all: unset;

  display: inline-flex;
  align-items: center;
  justify-content: center;

  width: 1.5em;
  height: 1.5em;

  cursor: pointer;
  flex-shrink: 0;
}

.ppwm-toc-toggle:focus-visible {
  outline: 2px solid currentColor;
  border-radius: 4px;
}

.ppwm-toc-caret {
  width: 0.7em;
  height: 0.7em;

  transition: transform 150ms ease;
}

.ppwm-toc-toggle[aria-expanded="false"] .ppwm-toc-caret {
  transform: rotate(-90deg);
}


/*
 * Collapse/expand animation.
 *
 * The nested <ul> is the grid item. Changing its row from
 * 1fr to 0fr gives us a CSS-only height transition.
 */
.ppwm-toc-children {
  display: grid;
  grid-template-rows: 1fr;

  transition: grid-template-rows 200ms ease;
}

.ppwm-toc-children > .ppwm-toc-list {
  min-height: 0;
  overflow: hidden;
}

.ppwm-toc-item.ppwm-toc-collapsed > .ppwm-toc-children {
  grid-template-rows: 0fr;
}
```

**JavaScript:**
```javascript
document.addEventListener('DOMContentLoaded', () => {
  document.querySelectorAll('.ppwm-toc-block').forEach((block) => {
    block.addEventListener('click', (event) => {
      const toggle = event.target.closest('.ppwm-toc-toggle');

      if (!toggle || !block.contains(toggle)) {
        return;
      }

      const item = toggle.closest('.ppwm-toc-item');

      if (!item) {
        return;
      }

      const expanded =
        toggle.getAttribute('aria-expanded') === 'true';

      toggle.setAttribute(
        'aria-expanded',
        String(!expanded)
      );

      item.classList.toggle(
        'ppwm-toc-collapsed',
        expanded
      );
    });
  });
});
```

---

### 6.7 Document Metadata Component (`<MetadataBlock>`)
*   **Purpose:** Renders comprehensive document version, cryptographic hash, tags, and bibliography details.
*   **Props:** `MetadataProps`
*   **Jinja2 / Astro Template:**
```jinja2
{#
This template uses the following environment and context variables:

environment = {
  labels: Record<string, string>;
  icons: Record<string, Record<string, string>>;
}

context = {
  metadata: Metadata;
  start_collapsed: boolean;
}
#}

<div
  class="ppwm-metadata-block{% if start_collapsed %} ppwm-metadata-collapsed{% endif %}"
>

  <div class="ppwm-metadata-header">
    <span class="ppwm-metadata-title">
      {{ labels.metadata }}
    </span>

    <button
      type="button"
      class="ppwm-metadata-toggle"
      aria-expanded="{{ 'false' if start_collapsed else 'true' }}"
      aria-controls="ppwm-metadata-content"
      aria-label="{{ labels.toggle_metadata or 'تغییر وضعیت اطلاعات' }}"
    >
      <svg
        class="ppwm-metadata-caret"
        viewBox="0 0 16 16"
        aria-hidden="true"
      >
        <path
          d="M4 6l4 4 4-4"
          fill="none"
          stroke="currentColor"
          stroke-width="1.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>
    </button>
  </div>


  <div
    class="ppwm-metadata-content"
    id="ppwm-metadata-content"
  >
    <div class="ppwm-metadata-grid">

      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">
          {{ labels.lastModified }}
        </span>

        <span class="ppwm-meta-value">
          {{ metadata.lastModified }}
        </span>
      </div>


      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">
          {{ labels.status }}
        </span>

        <span
          class="ppwm-meta-value ppwm-meta-status"
          data-status="{{ metadata.status }}"
        >
          <span
            class="ppwm-meta-status-icon"
            aria-hidden="true"
          >
            {{ icons.status[metadata.status] | safe }}
          </span>

          <span class="ppwm-meta-status-label">
            {{ labels[metadata.status] }}
          </span>
        </span>
      </div>


      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">
          {{ labels.checksum }}
        </span>

        <code class="ppwm-meta-value ppwm-meta-hash">
          {{ metadata.checksum }}
        </code>
      </div>


      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">
          {{ labels.contributors }}
        </span>

        <ul class="ppwm-meta-value ppwm-meta-contributors-list">
          {% for item in metadata.contributors %}
            <li class="ppwm-meta-contributor">
              <a
                href="/contributors/{{ item.contributor.id }}/"
                class="ppwm-contributor-link"
              >
                {{ item.contributor.name_fa }}
              </a>

              <span class="ppwm-contributor-role">
                ({{ item.role }})
              </span>
            </li>
          {% endfor %}
        </ul>
      </div>


      {% if metadata.prosody or metadata.prosody_name %}
        <div class="ppwm-meta-row">
          <span class="ppwm-meta-label">
            {{ labels.prosody }}
          </span>

          <span class="ppwm-meta-value">
            {% if metadata.prosody_name %}
              {{ metadata.prosody_name }}

              {% if metadata.prosody %}
                ({{ metadata.prosody }})
              {% endif %}
            {% else %}
              {{ metadata.prosody }}
            {% endif %}
          </span>
        </div>
      {% endif %}


      {% if metadata.tags %}
        <div class="ppwm-meta-row">
          <span class="ppwm-meta-label">
            {{ labels.tags }}
          </span>

          <div class="ppwm-meta-value ppwm-meta-tags-list">
            {% for tag in metadata.tags %}
              <span class="ppwm-tag-chip">
                {{ tag }}
              </span>
            {% endfor %}
          </div>
        </div>
      {% endif %}


      {% if metadata.sources %}
        <div class="ppwm-meta-row ppwm-meta-sources-row">
          <span class="ppwm-meta-label">
            {{ labels.sources }}
          </span>

          <ul class="ppwm-meta-value ppwm-meta-sources-list">
            {% for source in metadata.sources %}
              <li>{{ source }}</li>
            {% endfor %}
          </ul>
        </div>
      {% endif %}

    </div>
  </div>

</div>
```
*   **CSS Variable Sizing:**
```css
.ppwm-metadata-block {
  font-size: var(--ppwm-ftsz-metadata);
}


/* ─────────────────────────────────────────────
   Header
   ───────────────────────────────────────────── */

.ppwm-metadata-header {
  display: flex;
  align-items: center;
  justify-content: space-between;

  padding-block-end: 0.75em;
  border-block-end: 1px solid
    var(--ppwm-clr-border, rgba(0, 0, 0, 0.1));

  margin-block-end: 0.75em;
}

.ppwm-metadata-title {
  font-weight: 600;
}

.ppwm-metadata-toggle {
  all: unset;

  display: inline-flex;
  align-items: center;
  justify-content: center;

  width: 1.6em;
  height: 1.6em;

  cursor: pointer;
  flex-shrink: 0;
  border-radius: 4px;
}

.ppwm-metadata-toggle:focus-visible {
  outline: 2px solid var(--ppwm-clr-focus, currentColor);
}

.ppwm-metadata-caret {
  width: 0.8em;
  height: 0.8em;

  transition: transform 150ms ease;
}

.ppwm-metadata-toggle[aria-expanded="false"] .ppwm-metadata-caret {
  transform: rotate(-90deg);
}


/* ─────────────────────────────────────────────
   Collapsible content
   ───────────────────────────────────────────── */

.ppwm-metadata-content {
  display: grid;
  grid-template-rows: 1fr;

  overflow: hidden;

  transition:
    grid-template-rows 200ms ease,
    opacity 200ms ease;
}

.ppwm-metadata-collapsed .ppwm-metadata-content {
  grid-template-rows: 0fr;
  opacity: 0;
}


/* ─────────────────────────────────────────────
   Responsive two-column grid
   ───────────────────────────────────────────── */

.ppwm-metadata-grid {
  min-height: 0;

  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));

  column-gap: 2em;
  row-gap: 0.75em;
}


/* ─────────────────────────────────────────────
   Individual metadata rows
   ───────────────────────────────────────────── */

.ppwm-meta-row {
  display: grid;

  /*
   * The document is RTL, therefore:
   *
   *   label → right column
   *   value → left column
   *
   * The explicit text alignment below gives the
   * label and value the desired visual alignment.
   */
  grid-template-columns: auto minmax(0, 1fr);

  align-items: baseline;
  gap: 0.5em;

  min-width: 0;
}

.ppwm-meta-label {
  min-width: 0;

  text-align: left;
  font-weight: 600;
}

.ppwm-meta-value {
  min-width: 0;

  text-align: right;
}


/* ─────────────────────────────────────────────
   Status
   ───────────────────────────────────────────── */

.ppwm-meta-status {
  display: inline-flex;
  align-items: center;
  justify-content: flex-start;

  gap: 0.35em;
}

.ppwm-meta-status-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.ppwm-meta-status-icon svg {
  width: 1em;
  height: 1em;
}


/* ─────────────────────────────────────────────
   Checksum
   ───────────────────────────────────────────── */

.ppwm-meta-hash {
  overflow-wrap: anywhere;
}


/* ─────────────────────────────────────────────
   Contributors
   ───────────────────────────────────────────── */

.ppwm-meta-contributors-list {
  margin-block: 0;
  padding-inline-start: 1.25em;

  text-align: right;
}

.ppwm-meta-contributor {
  padding-inline-start: 0.15em;
}

.ppwm-contributor-role {
  opacity: 0.7;
}


/* ─────────────────────────────────────────────
   Tags
   ───────────────────────────────────────────── */

.ppwm-meta-tags-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;

  gap: 0.35em;
}


/* ─────────────────────────────────────────────
   Sources
   ───────────────────────────────────────────── */

.ppwm-meta-sources-list {
  margin-block: 0;
  padding-inline-start: 1.25em;

  text-align: right;
}


/* ─────────────────────────────────────────────
   Small screens
   ───────────────────────────────────────────── */

@media (max-width: 48rem) {
  .ppwm-metadata-grid {
    grid-template-columns: 1fr;
  }
}
```

**JavaScript:**

```javascript
document.addEventListener('DOMContentLoaded', () => {
  document.querySelectorAll('.ppwm-metadata-block').forEach((block) => {
    const toggle = block.querySelector('.ppwm-metadata-toggle');

    if (!toggle) {
      return;
    }

    toggle.addEventListener('click', () => {
      const expanded =
        toggle.getAttribute('aria-expanded') === 'true';

      toggle.setAttribute(
        'aria-expanded',
        String(!expanded)
      );

      block.classList.toggle(
        'ppwm-metadata-collapsed',
        expanded
      );
    });
  });
});
```

---

### 6.8 Font Size Control Component (`<FontSizeControl>`)
*   **Purpose:** Client-side interface control to adjust text scaling.
*   **Props:** `FontSizeControlProps`
*   **HTML Structure:**
```html
<div class="ppwm-font-size-control">
  <button class="ppwm-font-btn ppwm-btn-decrease" aria-label="کاهش اندازه قلم">−</button>
  <button class="ppwm-font-btn ppwm-btn-increase" aria-label="افزایش اندازه قلم">+</button>
</div>
```

---

### 6.9 Copy Link Component (`<CopyLink>`)
*   **Purpose:** Renders page or section link share buttons.
*   **Props:** `CopyLinkProps`
*   **HTML Structure:**
```html
<button class="ppwm-copy-link" data-url="{{ targetUrl }}" aria-label="کپی کردن پیوند">
  <span class="ppwm-copy-icon" aria-hidden="true">🔗</span>
  <span class="ppwm-copy-label">{{ label }}</span>
</button>
```

---

### 6.10 Layout & Header/Footer Components

#### `<RootLayout>`
*   **Purpose:** Top-level HTML wrapping and primary structural layout.
*   **Jinja2 / Astro Template:**
```jinja2
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ page_title }}</title>
  <link rel="stylesheet" href="/styles/main.css">
</head>
<body class="ppwm-body">
  {% include "SiteHeader" %}
  <main class="ppwm-middle-container">
    {{ content }}
  </main>
  {% include "SiteFooter" %}
</body>
</html>
```

#### `<SemiNavLayout>`
*   **Purpose:** Layout injecting breadcrumb tracks above primary content arrays.
*   **Jinja2 / Astro Template:**
```jinja2
{% extends "RootLayout" %}
{% block content %}
  {% include "Breadcrumbs" with breadcrumbs %}
  <div class="ppwm-main-container">
    {{ main_content }}
  </div>
{% endblock %}
```

#### `<FullNavLayout>`
*   **Purpose:** Three-column viewport layout linking navigation trees.
*   **Jinja2 / Astro Template:**
```jinja2
{% extends "RootLayout" %}
{% block content %}
  <div class="ppwm-layout-fullnav">
    <div class="ppwm-breadcrumbs-row">
      {% include "Breadcrumbs" with breadcrumbs %}
    </div>
    <div class="ppwm-layout-body-wrapper">
      <div class="ppwm-main-container">
        {{ main_content }}
      </div>
      <aside class="ppwm-sidebartree-wrapper" id="sidebar-tree">
        {% include "SidebarTree" with sidebar_tree_data %}
      </aside>
    </div>
  </div>
{% endblock %}
```

#### `<SiteHeader>`
*   **Purpose:** Standard site navigation, search wrapper, and theme configurations.
*   **Jinja2 / Astro Template:**
```jinja2
<header class="ppwm-site-header">
  <div class="ppwm-header-branding">
    <a href="/" class="ppwm-logo-link" aria-label="صفحه اصلی گوهرهای پارسی با محسن">
      <img src="/assets/logo.svg" alt="نشان‌واره گوهرهای پارسی با محسن" class="ppwm-logo">
      <span class="ppwm-site-title">گوهرهای پارسی با محسن</span>
    </a>
  </div>
  <div class="ppwm-header-controls">
    {% include "SearchBox" %}
    {% include "ThemeToggle" %}
    {% include "FontSizeControl" %}
  </div>
</header>
```

#### `<SiteFooter>`
*   **Purpose:** Renders attribution, compiler schemas, licenses, and repository links.
*   **Jinja2 / Astro Template:**
```jinja2
<footer class="ppwm-site-footer">
  <div class="ppwm-footer-links">
    <a href="/contributors/" class="ppwm-footer-link">مشارکت‌کنندگان</a>
    <a href="https://github.com/mohsen/pearls-of-persian" class="ppwm-footer-link" target="_blank" rel="noopener noreferrer">مخزن گیت‌هاب</a>
  </div>
  <div class="ppwm-footer-schema">
    <span>{{ schema_version_label }}</span>
  </div>
  <div class="ppwm-footer-copyright">
    <a href="/copyright/" class="ppwm-footer-link">بیانیه حق تکثیر و شرایط بهره‌برداری</a>
  </div>
</footer>
```

---

## 7. Typography

### 7.1 Font Stack
```css
/* Primary (Persian & Arabic) */
font-family: 'Vazirmatn', 'Noto Naskh Arabic', 'Traditional Arabic', system-ui, sans-serif;

/* Latin (Metadata & Code) */
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

/* Monospace (Git SHAs, Schemas) */
font-family: 'Fira Code', 'Cascadia Code', monospace;
```

### 7.2 Line Height
*   **Verse Text (`متن` - poetry):** `2.0` (provides comfortable breathing room for Persian script).
*   **Prose Text (`متن` - prose & meanings):** `1.8` (standard for Persian reading blocks).
*   **UI Elements:** `1.5` (optimized for compact interface alignment).

### 7.3 Font Sizing
Fluid typographic scales must be implemented using CSS `clamp()` to scale smoothly between mobile and desktop viewport boundaries.

| Element | CSS Variable | Notes |
| ------- | ------------ | ----- |
| H1 | `var(--ppwm-ftsz-h1)` | Page title in the `Title` block |
| H2 | `var(--ppwm-ftsz-h2)` | Page subtitle in the `Title` block |
| Excerpt | `var(--ppwm-ftsz-excerpt)` | For both the excerpt and the H3 number |
| Literary Explanations | `var(--ppwm-ftsz-explanation)` | Used for commentary and notes headings |
| Text | `var(--ppwm-ftsz-text)` | Normal inline text |
| Metadata | `var(--ppwm-ftsz-metadata)` | Compact meta-panel typography |
| Breadcrumbs segments | `var(--ppwm-ftsz-crumb)` | Navigational path typography |

---

## 8. Color & Theming

### 8.1 Light Theme
The palette is modeled after warm Persian manuscript paper (cream/beige) and deep carbon-ink writing tones.

```css
:root {
  --ppwm-bg-primary: #F9F6F1;       /* Main cream background */
  --ppwm-bg-secondary: #EFE9E0;     /* Sidebar & metadata cards */
  --ppwm-text-primary: #1A1614;     /* Ink black primary text */
  --ppwm-text-secondary: #4A4542;   /* Charcoal metadata text */
  --ppwm-accent-primary: #8B4513;   /* Amber-brown primary brand links */
  --ppwm-accent-secondary: #2C5F7C; /* Deep blue highlight (Lajevardi) */
  --ppwm-border: #D4C7B8;           /* Muted dividers */
}
```

### 8.2 Dark Theme
```css
[data-theme="dark"] {
  --ppwm-bg-primary: #1A1614;
  --ppwm-bg-secondary: #2A2421;
  --ppwm-text-primary: #E8E2D9;
  --ppwm-text-secondary: #B8AEA3;
  --ppwm-accent-primary: #D4A574;
  --ppwm-accent-secondary: #6B9FBB;
  --ppwm-border: #3A342F;
}
```

### 8.3 Semantic Status Colors
```css
:root {
  --ppwm-status-draft: #FFC107;
  --ppwm-status-final: #4CAF50;
  --ppwm-status-archived: #9E9E9E;
}

[data-theme="dark"] {
  --ppwm-status-draft: #FFB300;
  --ppwm-status-final: #66BB6A;
  --ppwm-status-archived: #757575;
}
```

---

## 9. Search

### 9.1 Indexing Strategy
Search is implemented as a build-time generated static index.

Index parameters must include:
*   Directory titles (`meta.yml:title`)
*   Document navigation titles (front matter `title`)
*   Document H1 (page display title)
*   Metadata tags (directory and document level)
*   Full-text body content of `#### متن` and `#### معنی روان`

### 9.2 UX
*   Persistent global search input positioned in the `<SiteHeader>`.
*   Results layout renders matching titles, dynamic highlighted content matches, breadcrumbs, and tag pills.

### 9.3 Performance
*   **Lazy Load:** The search index is fetched dynamically only when the user first focuses on the search input box.
*   **Asset Weight Optimization:** Very verbose editorial notes are excluded from index serialization to keep payload sizes minimal.

---

## 10. Accessibility & Internationalization

*   **HTML Attributes:** The root `<html>` tag must feature `dir="rtl"` and `lang="fa"`.
*   **Semantic Structures:** Glossary tables must use semantic `<table>`, `<th>`, and `<td>` tags. Annotation lists must be contained in `<ol>` or `<ul>` structures.
*   **Logical Styles:** Layout alignments must use logical CSS properties (e.g., `margin-inline-start`, `text-align: start`) rather than physical directions.
*   **No Color-Only Cues:** Status badges, text highlights, and inline links must be identifiable through semantic symbols, icons, or text styling (such as underlines) rather than color shifts alone.

---

## 11. Performance Targets

*   **Largest Contentful Paint (LCP):** < 1.5s
*   **Cumulative Layout Shift (CLS):** < 0.1 (Strict layout dimension bounding on fonts and components).
*   **Font Optimization:** All Persian font assets must be subset to exclude Latin weight-drifts and compiled in the `woff2` format with `font-display: swap` configured.

---

**End of Presentation Layer Specification v1.1.0**

Please critically check the structure and the contents of this version against the previous compiled one, and make necessary changes. 