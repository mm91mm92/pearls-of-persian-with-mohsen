# Pearls of Persian with Mohsen: Presentation Layer Specification

**Specification Version: 1.1.0**  
**Target Stack: Astro + TypeScript + Cloudflare Pages**

---

## 1. Design Principles

### 1.1 Core Philosophy
The presentation layer serves classical Persian literature with the same academic rigor and clarity that the content layer preserves it. Every design decision prioritizes **readability, structural consistency, and timeless aesthetics** over ephemeral trends.

### 1.2 Guiding Principles

| Principle | Description |
| :--- | :--- |
| **Content First** | Typography, spacing, and hierarchy exist to serve the literary text. Visual styling must never compete with or distract from the reading experience. |
| **Scholarly Precision** | Metadata, sources, and contributors are treated as first-class information. Citation and attribution are prioritized, not marginalized. |
| **Timeless Aesthetics** | The visual design draws inspiration from classical Persian manuscript traditions, utilizing warm, readable backdrops, generous margins, and subtle, intentional accents. |
| **Static-First Performance** | Pages must load rapidly. The core reading experience must function with zero client-side JavaScript. JavaScript is reserved for progressive enhancement. |
| **Accessible by Default** | Semantic HTML, comprehensive right-to-left (RTL) layout support, keyboard navigation, and screen reader compatibility are mandatory core features. |
| **Separation of Concerns** | The presentation layer is strictly a **deterministic rendering engine**. Content must never be edited, generated, or modified within this layer. |
| **Progressive Disclosure** | The primary excerpt (`#### متن`) is the focal point. Explanatory annotations are consolidated into a collapsed-by-default container to optimize readability and minimize cognitive load. |
| **Strict Rendering From Schema** | Document structure is rendered precisely as defined in `SCHEMA.md` v1.0.0. Optional sections are rendered only if present; no placeholder blocks are displayed. |

### 1.3 Non-Goals

| Principle | Description |
| :--- | :--- |
| **No Client-Side Routing** | The site utilizes static-first HTML pages, avoiding heavy app-shell routing or complex state hydration. |
| **No Content Management System (CMS)** | The system does not support in-browser editing, administrative login panels, or database writes. All content changes must occur in the repository. |
| **No Social Layer** | The application excludes social interactions such as comments, user profiles, or likes. Contributors represent structured metadata, not interactive user accounts. |

---

### 1.4 Baseline HTML & Global Configuration

```html
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>گوهرهای پارسی با محسن</title>
</head>
<body class="ppwm-body">
</body>
</html>
```

```css
:root {
  --ppwm-breakpoint-mobile: 48rem; /* 768px */
}
```

#### Shared System Labels (`src/constants/labels.ts`)
```typescript
export const LABELS: Record<string, string> = {
  // Statuses
  draft: "پیش‌نویس",
  final: "تکمیل‌شده",
  archived: "بایگانی‌شده",

  // Content Block H4 Headings
  excerpt: "متن",
  glossary: "واژگان و ترکیبات",
  meaning: "معنی روان",
  notes: "نکات",
  references: "ارجاعات",

  // Metadata Block
  metadata: "جزئیات سند",
  lastModified: "آخرین به‌روزرسانی",
  checksum: "شناسه رمزنگاری",
  status: "وضعیت",
  contributors: "مشارکت‌کنندگان",
  prosody: "وزن عروضی",
  tags: "برچسب‌ها",
  sources: "منابع",

  // Actions
  toggleMetadata: "تغییر وضعیت اطلاعات",
  toggleSection: "تغییر وضعیت بخش",
  copyLink: "کپی پیوند",
  editOnGithub: "ویرایش در گیت‌هاب",
};
```

---

## 2. URL Structure

### 2.1 Canonical Mapping
URLs map directly to physical filesystem paths under the `corpus/` directory, utilizing Persian-script slugs derived from directory and file names.

* **Directory Index Page:** Corresponds to a directory containing `meta.yml`
  * URL format: `/<path-to-directory>/`
  * Example: `/سعدی/گلستان/` or `/عطار-نیشابوری/منطق-الطیر/مقدمه/`
* **Literary Document Page:** Corresponds to a Markdown file (`*.md`)
  * URL format: `/<path-to-file-without-extension>/`
  * Example: `/سعدی/گلستان/دیباچه/` or `/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان/`

### 2.2 Encoding & Titles
* Path segments are derived from directory or file names and are URL-encoded by the web server.
* The user interface must display human-readable, decoded Persian titles sourced from:
  * `meta.yml:title` for directory levels (`root`, `poet`, `work`, `structural_division`).
  * Front matter `title` for navigation elements (breadcrumbs, sidebars) and document H1 for primary page display.

### 2.3 Canonical URL and Trailing Slash
* **Trailing Slashes are Canonical:** All canonical URLs must end with a trailing slash.
* **Enforced Redirection:** Requests without a trailing slash must perform a permanent 301 redirect to the canonical trailing slash path.
* **Case-Sensitivity:** URLs are evaluated with case-sensitivity to preserve exact Persian character mappings.

### 2.4 Special Routes

| Route | Purpose |
| :--- | :--- |
| `/` | Site root: Project introduction, featured works, and directory index. |
| `/search/` | Full-text search interface. |
| `/contributors/` | Directory of all project contributors. |
| `/contributors/<id>/` | Individual contributor profile detail page. |
| `/about/` | Project mission, schema documentation, and guidelines. |
| `/copyright/` | Copyright statement and intellectual property policies. |
| `/license/` | Terms and open-source licenses applied to site resources. |

---

## 3. Navigation

### 3.1 Breadcrumbs
Breadcrumbs are displayed at the top of pages (excluding Home, License, and Copyright pages). They are constructed dynamically from the directory hierarchy using `meta.yml:title` definitions and Markdown front matter `title`.

**Visual Format (RTL):**
```text
گوهرهای پارسی با محسن  ‹  سعدی  ‹  گلستان  ‹  باب اول  ‹  حکایت اول
```

### 3.2 Sidebar Navigation Tree (`<SidebarTree>`)
On directory index pages and literary document pages, a responsive sidebar displays the canonical structure of the repository and highlights the active location.

**Rules:**
* Render elements strictly in the order defined by `meta.yml:children`. Alphabetical or filesystem sorting is prohibited.
* The active location within the hierarchy must be highlighted and expanded to reveal immediate children.
* Other expandable nodes remain collapsed by default.

### 3.3 Next / Previous Navigation
For literary document pages nested within a work, sequential navigation controls are rendered at the bottom of the page to allow readers to traverse the work in its canonical sequence.

**Traversal Logic:**
* **Sibling Traversal:** "Next" moves to the next sibling listed in the parent directory's `meta.yml:children`.
* **Structural Boundary Traversal:** If the current document is the last child in its section's `children` array, "Next" traverses to the first document of the subsequent section (obeying parent `meta.yml:children` arrays recursively across nested depths).
* **Filtering:** Items omitted from `children` arrays are completely bypassed.

---

## 4. Page Layouts

### 4.1 Layout Scaffolding Components

#### `<RootLayout>`
Basic three-row grid: `<SiteHeader>`, `<MiddleContainer>`, and `<SiteFooter>`. Used by Home, Search, Copyright, and License pages.

#### `<SemiNavLayout>`
Extends `<RootLayout>` by injecting breadcrumb navigation at the top of the content area. Used by Contributors index and Contributor Profile pages.

#### `<FullNavLayout>`
Extends `<SemiNavLayout>` by docking `<SidebarTree>` alongside the main content on wide viewports. The sidebar collapses into a slide-out drawer on small viewports.

---

### 4.2 Page Wireframes

#### Home Page (`/`)
* **Layout:** `<RootLayout>`
* **Sections:** Banner / Site Intro, Recent Additions (from Git history), Featured Works, Poet & Work Directories.

#### Poet Page (`/corpus/<poet>/`)
* **Layout:** `<FullNavLayout>`
* **Sections:** Poet Title & Biography (`description`), Works List (ordered by `children`).

#### Work Page (`/corpus/<poet>/<work>/`)
* **Layout:** `<FullNavLayout>`
* **Sections:** Work Title, Poet Name as Subtitle, Description, Table of Contents Tree (reflecting `children` mixed files and `structural_division` folders).

#### Structural Division Page (`/corpus/<poet>/<work>/.../<structural_division>/`)
* **Layout:** `<FullNavLayout>`
* **Sections:** Division Title, Breadcrumb Subtitle, Description, TOC list/tree of subordinate divisions and literary units.

#### Literary Document Page (`/corpus/.../<document>/`)
* **Layout:** `<FullNavLayout>`
* **Sections:** H1 Title, `<MetadataBlock>`, Internal Page TOC (from body H2s, if any), Content Blocks (`<ContentBlock>`), Bottom Navigation Bar ("Previous" / "Next" / "Edit on GitHub").

#### Contributor Profile Page (`/contributors/<id>/`)
* **Layout:** `<SemiNavLayout>`
* **Sections:** Title Container, Info Container (bio, location, affiliation), Contact & Links Container, Contributions Matrix (grouped by role).

---

## 5. Data Types

### 5.1 Domain Models

```typescript
/**
 * Repositories hierarchy node types matching SCHEMA v1.0.0
 */
export type DirectoryType = 'root' | 'poet' | 'work' | 'structural_division';
export type WorkflowStatus = 'draft' | 'final' | 'archived';
export type ExcerptType = 'prose' | 'hemistich' | 'couplet' | 'enjambed';

/**
 * Single contributor profile record from contributors/<id>.yml
 */
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

/**
 * Document front matter and build metadata
 */
export interface Metadata {
  checksum: string;
  lastModified: string;
  status: WorkflowStatus;
  contributors: Array<{ contributor: Contributor; role: string }>;
  prosody?: string;
  prosody_name?: string;
  tags?: string[];
  sources?: string[];
  notes?: string;
}

/**
 * Excerpt block structure parsed from #### متن
 */
export interface ExcerptBlock {
  type: ExcerptType;
  lines: string[];
}

/**
 * Single entry in the glossary table
 */
export interface GlossaryEntry {
  term: string;
  explanation: string;
}

/**
 * Single annotatable content block (### <number>)
 */
export interface ContentBlockData {
  number: string;
  excerpt: ExcerptBlock;
  glossary?: GlossaryEntry[];
  meaning?: string;
  notes?: string[];
  references?: string[];
}

/**
 * Recursive Table of Contents item
 */
export interface TocItem {
  id: string;
  title: string;
  url: string;
  type: DirectoryType | 'literary_document';
  children?: TocItem[];
}
```

---

## 6. Component Specifications

### 6.1 Content Block Component (`<ContentBlock>`)
* **Purpose:** Renders an individual numbered annotation block (`### <number>`).
* **Progressive Disclosure:** Wraps annotations in a collapsible `<details>` element (closed by default) to keep the primary excerpt front and center.

```astro
---
// src/components/ContentBlock.astro
import ExcerptBlockComponent from './ExcerptBlock.astro';
import GlossaryBlockComponent from './GlossaryBlock.astro';
import type { ContentBlockData } from '../types';
import { LABELS } from '../constants/labels';

interface Props {
  cb: ContentBlockData;
}

const { cb } = Astro.props;
const hasAnnotations = Boolean(cb.glossary?.length || cb.meaning || cb.notes?.length || cb.references?.length);
---

<div class="ppwm-content-block" id={`block-${cb.number}`}>
  {hasAnnotations ? (
    <details class="ppwm-annotations-details">
      <summary class="ppwm-annotations-summary">
        <span class="ppwm-block-number-badge">{cb.number}</span>
        <span class="ppwm-excerpt-inline">
          <ExcerptBlockComponent eb={cb.excerpt} />
        </span>
      </summary>
      
      <div class="ppwm-annotations-content">
        {cb.glossary && cb.glossary.length > 0 && (
          <div class="ppwm-lit-section">
            <h4 class="ppwm-lit-h4">{LABELS.glossary}</h4>
            <GlossaryBlockComponent entries={cb.glossary} />
          </div>
        )}
        
        {cb.meaning && (
          <div class="ppwm-lit-section">
            <h4 class="ppwm-lit-h4">{LABELS.meaning}</h4>
            <p class="ppwm-prose-text">{cb.meaning}</p>
          </div>
        )}
        
        {cb.notes && cb.notes.length > 0 && (
          <div class="ppwm-lit-section">
            <h4 class="ppwm-lit-h4">{LABELS.notes}</h4>
            <ul class="ppwm-notes-list">
              {cb.notes.map((note) => <li>{note}</li>)}
            </ul>
          </div>
        )}
        
        {cb.references && cb.references.length > 0 && (
          <div class="ppwm-lit-section">
            <h4 class="ppwm-lit-h4">{LABELS.references}</h4>
            <ul class="ppwm-refs-list">
              {cb.references.map((ref) => <li>{ref}</li>)}
            </ul>
          </div>
        )}
      </div>
    </details>
  ) : (
    <div class="ppwm-excerpt-standalone-wrapper">
      <span class="ppwm-block-number-badge">{cb.number}</span>
      <div class="ppwm-excerpt-standalone">
        <ExcerptBlockComponent eb={cb.excerpt} />
      </div>
    </div>
  )}
</div>
```

---

### 6.2 Excerpt Component (`<ExcerptBlock>`)
* **Purpose:** Renders the compulsory excerpt (`#### متن`), choosing the correct CSS layout (prose vs. poetry hemistichs/couplets/enjambment) based on the structural type derived by the compiler.

```astro
---
// src/components/ExcerptBlock.astro
import type { ExcerptBlock as ExcerptBlockType } from '../types';

interface Props {
  eb: ExcerptBlockType;
}

const { eb } = Astro.props;
const nVerses = Math.floor(eb.lines.length / 2);
const hasTrailingHemistich = eb.lines.length % 2 !== 0;
---

{eb.type === 'prose' && (
  <p class="ppwm-prose-excerpt">{eb.lines[0]}</p>
)}

{eb.type === 'hemistich' && (
  <div class="ppwm-hemistich-excerpt">
    <span class="ppwm-hemistich-line">{eb.lines[0]}</span>
  </div>
)}

{eb.type === 'couplet' && (
  <div class="ppwm-verse-excerpt">
    <div class="ppwm-couplet">
      <span class="ppwm-hemistich ppwm-hemistich-odd">{eb.lines[0]}</span>
      <span class="ppwm-hemistich ppwm-hemistich-even">{eb.lines[1]}</span>
    </div>
  </div>
)}

{eb.type === 'enjambed' && (
  <div class="ppwm-enjambed-excerpt">
    {Array.from({ length: nVerses }).map((_, i) => (
      <div class="ppwm-couplet">
        <span class="ppwm-hemistich ppwm-hemistich-odd">{eb.lines[i * 2]}</span>
        <span class="ppwm-hemistich ppwm-hemistich-even">{eb.lines[i * 2 + 1]}</span>
      </div>
    ))}
    {hasTrailingHemistich && (
      <div class="ppwm-trailing-hemistich">
        <span class="ppwm-hemistich ppwm-hemistich-odd">{eb.lines[eb.lines.length - 1]}</span>
      </div>
    )}
  </div>
)}
```

```css
/* Styling for Excerpt Block */
.ppwm-prose-excerpt,
.ppwm-hemistich-line,
.ppwm-hemistich {
  font-size: var(--ppwm-ftsz-excerpt);
}

.ppwm-prose-excerpt {
  text-align: justify;
  line-height: 1.8;
  margin: 0;
}

.ppwm-hemistich-excerpt {
  text-align: center;
  margin: 0;
}

.ppwm-hemistich-line {
  font-weight: bold;
  display: block;
}

.ppwm-couplet {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  direction: rtl;
}

.ppwm-hemistich-odd {
  text-align: left;
  font-weight: bold;
}

.ppwm-hemistich-even {
  text-align: right;
  font-weight: bold;
}

.ppwm-enjambed-excerpt .ppwm-couplet {
  margin-bottom: 0.75rem;
}

.ppwm-trailing-hemistich {
  text-align: center;
}

@media (max-width: 48rem) {
  .ppwm-couplet {
    grid-template-columns: 1fr;
    gap: 0.5rem;
  }
  .ppwm-hemistich-odd,
  .ppwm-hemistich-even {
    text-align: center;
  }
}
```

---

### 6.3 Glossary Component (`<GlossaryBlock>`)

```astro
---
// src/components/GlossaryBlock.astro
import type { GlossaryEntry } from '../types';

interface Props {
  entries: GlossaryEntry[];
}

const { entries } = Astro.props;
---

<div class="ppwm-glossary-block">
  <table class="ppwm-glossary-table">
    <tbody>
      {entries.map((entry) => (
        <tr>
          <th class="ppwm-glossary-term">{entry.term}</th>
          <td class="ppwm-glossary-explanation">{entry.explanation}</td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

---

### 6.4 Title Component (`<TitleBlock>`)

```astro
---
// src/components/TitleBlock.astro
interface Props {
  title: string;
  subtitle?: string;
  description?: string;
}

const { title, subtitle, description } = Astro.props;
---

<header class="ppwm-title-block">
  <h1 class="ppwm-title-h1">{title}</h1>
  {subtitle && <p class="ppwm-title-subtitle">{subtitle}</p>}
  {description && <p class="ppwm-title-description">{description}</p>}
</header>
```

---

### 6.5 Document Metadata Component (`<MetadataBlock>`)

```astro
---
// src/components/MetadataBlock.astro
import type { Metadata } from '../types';
import { LABELS } from '../constants/labels';

interface Props {
  metadata: Metadata;
  startCollapsed?: boolean;
}

const { metadata, startCollapsed = false } = Astro.props;

const statusIcons = {
  draft: '🟡',
  final: '✅',
  archived: '⚫',
};
---

<div class={`ppwm-metadata-block ${startCollapsed ? 'ppwm-metadata-collapsed' : ''}`}>
  <div class="ppwm-metadata-header">
    <span class="ppwm-metadata-title">{LABELS.metadata}</span>
    <button
      type="button"
      class="ppwm-metadata-toggle"
      aria-expanded={startCollapsed ? 'false' : 'true'}
      aria-label={LABELS.toggleMetadata}
    >
      <svg class="ppwm-metadata-caret" viewBox="0 0 16 16" aria-hidden="true">
        <path d="M4 6l4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"></path>
      </svg>
    </button>
  </div>

  <div class="ppwm-metadata-content">
    <div class="ppwm-metadata-grid">
      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">{LABELS.lastModified}:</span>
        <span class="ppwm-meta-value">{metadata.lastModified}</span>
      </div>

      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">{LABELS.status}:</span>
        <span class="ppwm-meta-value ppwm-meta-status" data-status={metadata.status}>
          <span aria-hidden="true">{statusIcons[metadata.status]}</span>
          <span>{LABELS[metadata.status]}</span>
        </span>
      </div>

      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">{LABELS.checksum}:</span>
        <code class="ppwm-meta-value ppwm-meta-hash">{metadata.checksum}</code>
      </div>

      <div class="ppwm-meta-row">
        <span class="ppwm-meta-label">{LABELS.contributors}:</span>
        <ul class="ppwm-meta-value ppwm-meta-contributors-list">
          {metadata.contributors.map((item) => (
            <li>
              <a href={`/contributors/${item.contributor.id}/`}>
                {item.contributor.name_fa}
              </a>
              <span class="ppwm-contributor-role">({item.role})</span>
            </li>
          ))}
        </ul>
      </div>

      {(metadata.prosody || metadata.prosody_name) && (
        <div class="ppwm-meta-row">
          <span class="ppwm-meta-label">{LABELS.prosody}:</span>
          <span class="ppwm-meta-value">
            {metadata.prosody_name} {metadata.prosody ? `(${metadata.prosody})` : ''}
          </span>
        </div>
      )}

      {metadata.tags && metadata.tags.length > 0 && (
        <div class="ppwm-meta-row">
          <span class="ppwm-meta-label">{LABELS.tags}:</span>
          <div class="ppwm-meta-value ppwm-meta-tags-list">
            {metadata.tags.map((tag) => <span class="ppwm-tag-chip">{tag}</span>)}
          </div>
        </div>
      )}

      {metadata.sources && metadata.sources.length > 0 && (
        <div class="ppwm-meta-row ppwm-meta-sources-row">
          <span class="ppwm-meta-label">{LABELS.sources}:</span>
          <ul class="ppwm-meta-value ppwm-meta-sources-list">
            {metadata.sources.map((source) => <li>{source}</li>)}
          </ul>
        </div>
      )}
    </div>
  </div>
</div>

<script>
  document.querySelectorAll('.ppwm-metadata-toggle').forEach((button) => {
    button.addEventListener('click', () => {
      const block = button.closest('.ppwm-metadata-block');
      if (!block) return;
      const isExpanded = button.getAttribute('aria-expanded') === 'true';
      button.setAttribute('aria-expanded', String(!isExpanded));
      block.classList.toggle('ppwm-metadata-collapsed', isExpanded);
    });
  });
</script>
```

---

### 6.10 Contributor Component (`<ContributorBlock>`)

* **Purpose:** Displays an individual contributor profile in three distinct sections:
  1. Personal Information (Name, Location, Affiliations, Bio).
  2. Contact & Social Handles (Email, Website, and the `links` flat dictionary).
  3. Contribution Matrix (Articles grouped by role).

```astro
---
// src/components/ContributorBlock.astro
import type { Contributor } from '../types';

interface Props {
  contributor: Contributor;
  hideName?: boolean;
  contributionsByRole?: Record<string, Array<{ title: string; url: string }>>;
}

const { contributor, hideName = false, contributionsByRole = {} } = Astro.props;
---

<div class="ppwm-contributor-block" id={`contributor-${contributor.id}`}>
  {/* Section 1: Bio & Institutional Info */}
  <section class="ppwm-contrib-section ppwm-contrib-bio-section">
    {!hideName && (
      <div class="ppwm-contrib-header">
        <h2 class="ppwm-contrib-name-fa">{contributor.name_fa}</h2>
        <span class="ppwm-contrib-name-en">{contributor.name}</span>
      </div>
    )}

    <div class="ppwm-contrib-details">
      {contributor.location && (
        <div class="ppwm-contrib-detail-item">
          <strong>مکان:</strong> {contributor.location}
        </div>
      )}

      {(contributor.affiliation_fa || contributor.affiliation) && (
        <div class="ppwm-contrib-detail-item">
          <strong>وابستگی سازمانی:</strong>{' '}
          {contributor.affiliation_fa || contributor.affiliation}
        </div>
      )}

      {contributor.bio_fa && (
        <p class="ppwm-contrib-bio">{contributor.bio_fa}</p>
      )}
    </div>
  </section>

  {/* Section 2: Links & Platform Identifiers */}
  <section class="ppwm-contrib-section ppwm-contrib-links-section" dir="ltr">
    <h3 class="ppwm-contrib-subsection-title">Contact & Profiles</h3>
    <ul class="ppwm-contrib-links-list">
      <li>
        <strong>Email:</strong> <a href={`mailto:${contributor.email}`}>{contributor.email}</a>
      </li>

      {contributor.website && (
        <li>
          <strong>Website:</strong>{' '}
          <a href={contributor.website} target="_blank" rel="noopener noreferrer">
            {contributor.website}
          </a>
        </li>
      )}

      {contributor.links &&
        Object.entries(contributor.links).map(([platform, val]) => (
          <li>
            <strong class="ppwm-platform-name">{platform}:</strong>{' '}
            {val.startsWith('http') ? (
              <a href={val} target="_blank" rel="noopener noreferrer">
                {val}
              </a>
            ) : (
              <span>{val}</span>
            )}
          </li>
        ))}
    </ul>
  </section>

  {/* Section 3: Contribution Matrix */}
  {Object.keys(contributionsByRole).length > 0 && (
    <section class="ppwm-contrib-section ppwm-contrib-matrix-section">
      <h3 class="ppwm-contrib-subsection-title">مشارکت‌ها در پروژه</h3>
      {Object.entries(contributionsByRole).map(([role, articles]) => (
        <div class="ppwm-contrib-role-group">
          <h4 class="ppwm-contrib-role-heading">{role} ({articles.length})</h4>
          <ul class="ppwm-contrib-articles-list">
            {articles.map((art) => (
              <li>
                <a href={art.url}>{art.title}</a>
              </li>
            ))}
          </ul>
        </div>
      ))}
    </section>
  )}
</div>
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
* **Poetry (`#### متن`):** `2.0` (optimal breathing room for diacritics and Persian script).
* **Prose (`#### متن` & meanings):** `1.8`.
* **UI Controls & Metadata:** `1.5`.

### 7.3 Fluid Typography Scale
Fluid typography uses CSS `clamp()` to transition smoothly across mobile and desktop viewports:

```css
:root {
  --ppwm-ftsz-h1: clamp(1.75rem, 4vw, 2.5rem);
  --ppwm-ftsz-h2: clamp(1.25rem, 3vw, 1.75rem);
  --ppwm-ftsz-excerpt: clamp(1.15rem, 2.5vw, 1.4rem);
  --ppwm-ftsz-text: clamp(1rem, 2vw, 1.125rem);
  --ppwm-ftsz-metadata: clamp(0.85rem, 1.5vw, 0.95rem);
  --ppwm-ftsz-crumb: clamp(0.8rem, 1.2vw, 0.9rem);
}
```

---

## 8. Color & Theming

### 8.1 Light Theme (Classical Persian Manuscript Palette)
```css
:root {
  --ppwm-bg-primary: #F9F6F1;       /* Cream manuscript paper */
  --ppwm-bg-secondary: #EFE9E0;     /* Soft card backdrop */
  --ppwm-text-primary: #1A1614;     /* Carbon ink text */
  --ppwm-text-secondary: #4A4542;   /* Muted annotation text */
  --ppwm-accent-primary: #8B4513;   /* Warm terracotta brown */
  --ppwm-accent-secondary: #2C5F7C; /* Lajevardi blue accent */
  --ppwm-border: #D4C7B8;           /* Paper dividers */
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

---

## 9. Search & Indexing Strategy

### 9.1 Build-Time Indexing
Search uses a static build-time index generated by **Pagefind**.

**Indexed Document Signals:**
* Directory titles (`meta.yml:title`)
* Document navigation title (`frontmatter.title`) and Page Title (`H1`)
* Metadata tags (directory and document levels)
* Full text of `#### متن` and `#### معنی روان`

### 9.2 Search Text Normalization
To deliver accurate results for Persian text, the build pipeline normalizes characters before writing the search index:

```typescript
export function normalizePersianSearchText(text: string): string {
  return text
    .replace(/[\u064B-\u065F]/g, '') // Strip Tashkeel / Diacritics
    .replace(/\u200C/g, '')          // Strip ZWNJ for unified token matching
    .replace(/ي/g, 'ی')               // Unify Arabic Yeh to Persian Yeh
    .replace(/ك/g, 'ک');              // Unify Arabic Kaf to Persian Keheh
}
```

---

## 10. Performance Targets

* **First Contentful Paint (FCP):** < 0.8s
* **Largest Contentful Paint (LCP):** < 1.2s
* **Cumulative Layout Shift (CLS):** `0.0`
* **Font Delivery:** All Persian fonts must be subset and served locally in `.woff2` format with `font-display: swap`.

---

**End of Presentation Layer Specification v1.1.0**