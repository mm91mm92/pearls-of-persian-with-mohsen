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

This layout remains unchanged on viewport from small to wide.

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

This layout remains unchanged on viewport from small to wide.

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
* A link to the repository on the GitHub.

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
*   **Layout:** Uses `<SemiNavLayout>`. The `<Breadcrumbs>` displays: `گوهرهای پارسی با محسن ‹ مشارکت‌کنندگان ‹ <id>`, where the final segment is unlinked. The `<MainContainer>` is split into four distinct content blocks:
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
*   **Source:** A `.md` file containing front matter, an H1 heading, and content blocks interspersed with H2 headings.
*   **URL:** `/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان/`
*   **Layout:** Uses `<FullNavLayout>`. The `<SidebarTree>` represents the document's location within the hierarchy. The `<MainContainer>` consists of the following components:
    *   **Title:** Renders the page's primary H1 heading.
    *   **Metadata:** A vertical [[#6.7 Document Metadata Component (`<MetadataBlock>`)|card component]] displaying status, active contributors, prosody, and sources.
    *   **Table of Contents:** Auto-generated link list constructed solely from H2 headings (omitted if no H2 headings exist).
    *   **Literary Division:** Vertically stacks the H2 section headings and standard `<ContentBlock>` elements.
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

### 5.1 `Toc`
Provides the backbone for a multi-level, non-uniform depth TOC throughout the presentation layer of the project:
```typescript
export interface TocItem {
  title: string;
  url: string;
  children?: TocItem[]; // Optional nested items
}

// The root TOC is simply an array of items
export type Toc = TocItem[];
```

### 5.2 `Contributor`
Packs all pieces of information for a specific contributor. The `id` is the filename without the extension, while the other fields are read from the file content.
```typescript
interface Contributor {
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
  links?: Record<string, string>; // Flat map platform-to-string dictionary
}
```

### 5.3 `Metadata`
This data type represents the metadata (the front matter) of literary documents. The `checksum` and `lastModified` fields are extracted from the Git history during build, while the other fields are collected from the front matter.
```typescript
interface Metadata {
  checksum: string;
  lastModified: string;
  status: string;
  contributors: Array<{ contributor: Contributor; role: string }>;
  prosody?: string;
  prosody_name?: string;
  tags?: string[];
  sources?: string[];
  notes?: string;
}
```

---

## 6. Component Specifications

### 6.1 Content Block Component (`<ContentBlock>`)

**Purpose:** Renders an individual H3 block (`### <number_or_number_range>`) containing the numeric identifier and standard literary annotations.

**Progressive Disclosure:** To reduce cognitive load and prioritize text readability, annotations (glossary, meanings, notes, references) are consolidated into a unified, collapsible `<details>` accordion component that is closed by default.

**Configurable Headings:** To avoid hardcoding localized strings (such as `واژگان و ترکیبات`, `معنی روان`, `نکات ادبی`, and `ارجاعات`) inside the component markup, these labels must be dynamically injected from a global UI localization configuration object (e.g., `labels` / `uiLabels` dictionary).

**Props:**
```typescript
interface ContentBlockProps {
  number: string; // The bare numeric identifier (e.g., "۱" or "۴-۵")
  excerpt: ExcerptProps;
  glossary?: GlossaryProps;
  meaning?: string;
  literaryNotes?: string[];
  references?: string[];
  labels: Record<string, string>; // Injected labels for dynamic headings
}
```

**Jinja2 / Astro Implementation Template:**
```jinja2
{% for cb in contBlocks %}
  <div class="ppwm-content-block" id="block-{{ cb.number }}">
    {% if cb.glossary or cb.meaning or cb.literaryNotes or cb.references %}
      <details class="ppwm-annotations-details">
        <summary class="ppwm-annotations-summary">
          <span class="ppwm-block-number-badge">{{ cb.number }}</span>
          <span class="ppwm-excerpt-inline">{{ cb.excerpt }}</span>
        </summary>
        
        <div class="ppwm-annotations-content">
          {% if cb.glossary %}
            <div class="ppwm-lit-glossary">
              <h4 class="ppwm-lit-h4">{{ labels.glossary_title }}</h4>
              {{ cb.glossary }}
            </div>
          {% endif %}
          
          {% if cb.meaning %}
            <div class="ppwm-lit-meaning">
              <h4 class="ppwm-lit-h4">{{ labels.meaning_title }}</h4>
              <p class="ppwm-prose-text">{{ cb.meaning }}</p>
            </div>
          {% endif %}
          
          {% if cb.literaryNotes %}
            <div class="ppwm-lit-notes">
              <h4 class="ppwm-lit-h4">{{ labels.notes_title }}</h4>
              <ol class="ppwm-notes-list">
                {% for note in cb.literaryNotes %}
                  <li>{{ note }}</li>
                {% endfor %}
              </ol>
            </div>
          {% endif %}
          
          {% if cb.references %}
            <div class="ppwm-lit-refs">
              <h4 class="ppwm-lit-h4">{{ labels.references_title }}</h4>
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
        <span class="ppwm-excerpt-standalone">{{ cb.excerpt }}</span>
      </div>
    {% endif %}
  </div>
{% endfor %}
```

---

### 6.2 Excerpt Component (`<ExcerptBlock>`)

**Purpose:** Renders the compulsory `#### متن` section, dynamically formatting the markup depending on its structural type.

**Props:**
```typescript
interface ExcerptProps {
  type: 'prose' | 'hemistich' | 'verse' | 'enjambed';
  lines: string[];
}
```

**Rendering Logic:**
1.  **`type: 'prose'`**  
    Renders as a standard, right-aligned `<p>` element with prose-optimized line-height. No bolding is applied.
2.  **`type: 'hemistich'`**  
    Renders as a single, bolded (`**text**`), centered line.
3.  **`type: 'verse'`**  
    Renders as a traditional Persian verse. On desktop viewports, it displays as a balanced, two-column grid (first left-aligned hemistich on the right column, second right-aligned hemistich on the left column). On mobile viewports, it falls back to a stacked vertical layout.
4.  **`type: 'enjambed'`**  
    Renders as multiple verses stacked vertically, preserving the two-column grid alignment on desktop and vertical stacking on mobile. An explicit margin-bottom separates distinct verse rows while keeping them grouped under a single H3 block.
    *   **Even-hemistich enjambed:** Renders hemistichs sequentially in pairs.
    *   **Odd-hemistich enjambed:** Renders paired hemistichs sequentially, with the remaining trailing single hemistich rendered centered at the bottom on wide viewports, or stacked on mobile.

**CSS Variable Integration:**
```css
.ppwm-excerpt {
  font-size: var(--ppwm-ftsz-excerpt);
}
```

---

### 6.3 Glossary Component (`<GlossaryBlock>`)

**Purpose:** Renders glossary entries inside a semantic table.

**Props:**
```typescript
interface GlossaryProps {
  entries: Array<{ term: string; explanation: string }>;
}
```

**HTML Output:**
```html
<div class="ppwm-glossary-block">
  <table class="ppwm-glossary-table">
    <tbody>
      <tr>
        <th class="ppwm-glossary-term">سیمرغ</th>
        <td class="ppwm-glossary-explanation">پرنده‌ای افسانه‌ای که در ادبیات عرفانی نماد ذات حق است</td>
      </tr>
    </tbody>
  </table>
</div>
```

**CSS Variable Integration:**
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
Renders a consistent, stylized title block for directory and content pages.

**Props:**
```typescript
interface TitleProps {
  title: string;
  subtitle?: string;
  description?: string;
}
```

**CSS Variable Sizing Map:**
*   `title` field maps to: `var(--ppwm-ftsz-h1)`
*   `subtitle` field maps to: `var(--ppwm-ftsz-h2)`
*   `description` field maps to: `var(--ppwm-ftsz-text)`

---

### 6.5 Breadcrumbs Component (`<Breadcrumbs>`)
**Purpose:** Contextual hierarchical navigation (see [Section 3.1](#31-breadcrumb-structure)).

**Props:**
```typescript
interface BreadcrumbsProps {
  segments: Array<{ title: string; href: string }>
}
```

**Rules:**
*   All segments must contain `title` and `href` fields.
*   Except for the final segment (the active page), all breadcrumb elements must be rendered as active hyperlinks.
*   Breadcrumb segments utilize the `var(--ppwm-ftsz-crumb)` sizing token.

---

### 6.6 Table of Contents Component (`<TocBlock>`)
Renders a hierarchical table of contents with variable-depth links. If no structural depth exists, it falls back to rendering a flat list.

**Props:**
```typescript
interface TocProps {
   toc: Toc;
}
```

**CSS Sizing Variable:**
*   Font size maps to: `var(--ppwm-ftsz-text)`

---

### 6.7 Document Metadata Component (`<MetadataBlock>`)
Renders the visual presentation layer of the `Metadata` data type, utilizing icons and standard labels for status badges, contributors, prosody patterns, and bibliography.

```typescript
interface MetadataProps {
   metadata: Metadata;
}
```

---

### 6.8 Font Size Control Component (`<FontSizeControl>`)
Provides high-readability interactive controls allowing readers to dynamically adjust the line size of Persian content. Values are managed via CSS custom properties (`var(--ppwm-ftsz-verse)` and `var(--ppwm-ftsz-body)`) and persisted in local storage.

---

### 6.9 Copy Link Component (`<CopyLink>`)
Renders a minimal button that copies the page's canonical URL or a specific verse-level anchor link (`#block-1`) to the clipboard.

---

### 6.10 Status Badge Component (`<StatusBadge>`)
Visual indicator for document workflow state.

**Visual Mapping:**

| Status | Color CSS Variable | Persian Label | Icon |
|--------|-------|--------------|------|
| `draft` | `var(--ppwm-status-draft)` | پیش‌نویس | 🟡 |
| `final` | `var(--ppwm-status-final)` | تکمیل‌شده | ✅ |
| `archived` | `var(--ppwm-status-archived)` | بایگانی‌شده | ⚫ |

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
| Metadata | `var(--ppwm-ppwm-ftsz-metadata)` | Compact meta-panel typography |
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