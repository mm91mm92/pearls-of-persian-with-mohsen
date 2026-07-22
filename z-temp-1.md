# Pearls of Persian: Presentation Layer Specification v1.0.0

## 1. Design Principles

### 1.1 Core Philosophy
The presentation layer serves classical Persian literature with academic rigor and timeless aesthetics. Every design decision prioritizes **readability, structural consistency, and scholarly precision**.

### 1.2 Guiding Principles

| Principle | Implementation |
|-----------|----------------|
| **Content First** | Typography and spacing serve the literary text. Visual styling never competes with reading experience. |
| **Scholarly Precision** | Metadata, sources, and attribution are first-class UI elements, not afterthoughts. |
| **Timeless Aesthetics** | Design draws from Persian manuscript traditions: warm backgrounds, generous margins, intentional accents. |
| **Static-First Performance** | Core reading experience works without JavaScript. JS enhances, never enables. |
| **Accessible by Default** | Semantic HTML, RTL layout, keyboard navigation, and screen reader support are mandatory. |
| **Strict Schema Compliance** | Presentation is a deterministic rendering engine. No content generation or modification occurs here. |
| **Progressive Disclosure** | Primary excerpts (`متن`) are focal. Annotations collapse by default to reduce cognitive load. |

### 1.3 Non-Goals
- No client-side routing or SPA architecture
- No CMS, admin panels, or in-browser editing
- No social features (comments, profiles, reactions)
- No dynamic content generation

---

## 2. Type System

### 2.1 Naming Convention

**Rule:** Component input types use `Props` suffix. Domain data types do not.

```typescript
// ✅ Component Props (input to render functions)
interface ContentBlockProps { /* ... */ }
interface TitleProps { /* ... */ }

// ✅ Domain Data (business logic objects)
interface Contributor { /* ... */ }
interface Excerpt { /* ... */ }
```

### 2.2 Domain Data Types

These represent **data structures** compiled from the corpus schema, not component inputs.

```typescript
/**
 * URL classification for routing logic
 */
export type UrlType = '404' | 'site' | 'corpus';

export interface Url {
  text: string;
  type: UrlType;
}

/**
 * Table of Contents tree structure
 */
export interface TocNode {
  title: string;
  url: string;
  children?: TocNode[];
}

/**
 * Flattened TOC with navigation methods
 */
export interface Toc {
  root: TocNode[];
  
  /** Sequential next document in reading order */
  getNext(current: TocNode): TocNode | null;
  
  /** Sequential previous document in reading order */
  getPrev(current: TocNode): TocNode | null;
}

/**
 * Contributor profile (from contributors/<id>.yml)
 */
export interface Contributor {
  id: string;
  name: string;
  name_fa: string;
  email: string;
  website?: string;
  location?: string;
  affiliation?: string;
  affiliation_fa?: string;
  bio?: string;
  bio_fa?: string;
  preferred_roles?: string[];
  links?: Record<string, string>;
}

/**
 * Literary document metadata (from front matter + Git)
 */
export interface DocumentMetadata {
  checksum: string;
  lastModified: string; // ISO 8601
  status: 'draft' | 'final' | 'archived';
  contributors: Array<{
    contributor: Contributor;
    role: string;
  }>;
  prosody?: string;
  prosody_name?: string;
  tags?: string[];
  sources?: string[];
  notes?: string; // Internal only, never displayed
}

/**
 * Excerpt text structure (from #### متن parsing)
 */
export type ExcerptType = 'prose' | 'hemistich' | 'couplet' | 'enjambed';

export interface Excerpt {
  type: ExcerptType;
  lines: string[];
}

/**
 * Glossary entry (from #### واژگان و ترکیبات)
 */
export interface GlossaryEntry {
  term: string;
  explanation: string;
}
```

### 2.3 Component Props Types

```typescript
/**
 * Title block (page headers)
 */
export interface TitleProps {
  title: string;
  subtitle?: string;
  description?: string;
}

/**
 * Breadcrumb navigation
 */
export interface BreadcrumbSegment {
  title: string;
  href: string;
}

export interface BreadcrumbsProps {
  segments: BreadcrumbSegment[];
  current: string; // Non-linked final segment
}

/**
 * Hierarchical TOC display
 */
export interface TocProps {
  nodes: TocNode[];
}

/**
 * Document metadata card
 */
export interface MetadataProps {
  metadata: DocumentMetadata;
  startCollapsed?: boolean;
}

/**
 * Content block (H3 + متن + annotations)
 */
export interface ContentBlockProps {
  number: string; // e.g., "1" or "123-124"
  excerpt: Excerpt;
  glossary?: GlossaryEntry[];
  meaning?: string;
  notes?: string[];
  references?: string[];
}

/**
 * Excerpt rendering
 */
export interface ExcerptProps {
  excerpt: Excerpt;
}

/**
 * Glossary table
 */
export interface GlossaryProps {
  entries: GlossaryEntry[];
}

/**
 * Contributor profile display
 */
export interface ContributorProps {
  contributor: Contributor;
  hideName?: boolean; // When used inside TitleBlock
  showArticles?: boolean; // Show contribution list
}

/**
 * Font size controls
 */
export interface FontControlsProps {
  minSize?: number; // default: 14
  maxSize?: number; // default: 24
  step?: number; // default: 2
}
```

---

## 3. URL Structure & Routing

### 3.1 Canonical Mapping

All URLs derive from filesystem paths under `corpus/`:

```
/                                    → corpus/meta.yml (root index)
/عطار-نیشابوری/                      → corpus/عطار-نیشابوری/meta.yml
/عطار-نیشابوری/منطق-الطیر/            → corpus/عطار-نیشابوری/منطق-الطیر/meta.yml
/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان/  → .../مجمع-مرغان.md
```

### 3.2 Encoding Rules
- Path segments are URL-encoded by the server
- Display titles come from `meta.yml:title` or front matter `title`
- **All canonical URLs end with trailing slash** (`/`)
- Non-trailing requests → 301 redirect to canonical

### 3.3 Special Routes

| Route | Purpose |
|-------|---------|
| `/` | Project homepage |
| `/search/` | Search interface |
| `/contributors/` | Contributor directory |
| `/contributors/<id>/` | Individual profile |
| `/copyright/` | Copyright notice |
| `/license/` | License terms |

---

## 4. CSS Architecture

### 4.1 Custom Property System

```css
:root {
  /* ═══════════════════════════════════════════
     Typography Scale (Fluid)
     ═══════════════════════════════════════════ */
  --ppwm-ftsz-h1: clamp(2rem, 4vw + 1rem, 3rem);
  --ppwm-ftsz-h2: clamp(1.5rem, 3vw + 0.75rem, 2.25rem);
  --ppwm-ftsz-excerpt: clamp(1.125rem, 2vw + 0.5rem, 1.5rem);
  --ppwm-ftsz-text: clamp(1rem, 1.5vw + 0.5rem, 1.125rem);
  --ppwm-ftsz-metadata: clamp(0.875rem, 1vw + 0.5rem, 1rem);
  --ppwm-ftsz-crumb: clamp(0.8125rem, 1vw + 0.4rem, 0.9375rem);

  /* ═══════════════════════════════════════════
     Responsive Breakpoint
     ═══════════════════════════════════════════ */
  --ppwm-bp-mobile: 48rem; /* 768px at 16px base */

  /* ═══════════════════════════════════════════
     Light Theme Colors
     ═══════════════════════════════════════════ */
  --ppwm-bg-primary: #F9F6F1;
  --ppwm-bg-secondary: #EFE9E0;
  --ppwm-text-primary: #1A1614;
  --ppwm-text-secondary: #4A4542;
  --ppwm-accent-primary: #8B4513;
  --ppwm-accent-secondary: #2C5F7C;
  --ppwm-border: #D4C7B8;

  /* ═══════════════════════════════════════════
     Status Colors
     ═══════════════════════════════════════════ */
  --ppwm-status-draft: #FFC107;
  --ppwm-status-final: #4CAF50;
  --ppwm-status-archived: #9E9E9E;
}

[data-theme="dark"] {
  --ppwm-bg-primary: #1A1614;
  --ppwm-bg-secondary: #2A2421;
  --ppwm-text-primary: #E8E2D9;
  --ppwm-text-secondary: #B8AEA3;
  --ppwm-accent-primary: #D4A574;
  --ppwm-accent-secondary: #6B9FBB;
  --ppwm-border: #3A342F;

  --ppwm-status-draft: #FFB300;
  --ppwm-status-final: #66BB6A;
  --ppwm-status-archived: #757575;
}
```

### 4.2 Font Stacks

```css
/* Persian/Arabic primary */
--ppwm-font-primary: 'Vazirmatn', 'Noto Naskh Arabic', 'Traditional Arabic', system-ui, sans-serif;

/* Latin (UI/metadata) */
--ppwm-font-latin: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

/* Monospace (hashes/code) */
--ppwm-font-mono: 'Fira Code', 'Cascadia Code', 'Consolas', monospace;
```

### 4.3 Line Heights

```css
--ppwm-lh-verse: 2.0;    /* Poetry */
--ppwm-lh-prose: 1.8;    /* Prose & meanings */
--ppwm-lh-ui: 1.5;       /* Interface elements */
```

---

## 5. Component Specifications

### 5.1 Layout Scaffolding

#### `<RootLayout>`
Base HTML structure for all pages.

```html
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ page_title }}</title>
  <link rel="stylesheet" href="/styles/main.css">
</head>
<body class="ppwm-body">
  <header class="ppwm-site-header">
    <!-- SiteHeader component -->
  </header>
  
  <main class="ppwm-middle-container">
    {{ slot }}
  </main>
  
  <footer class="ppwm-site-footer">
    <!-- SiteFooter component -->
  </footer>
  
  <script src="/scripts/main.js" defer></script>
</body>
</html>
```

#### `<SemiNavLayout>`
Adds breadcrumbs above content.

```html
<!-- extends RootLayout -->
<nav aria-label="مسیر صفحه" class="ppwm-breadcrumbs-wrapper">
  <Breadcrumbs {...breadcrumbsProps} />
</nav>

<div class="ppwm-main-container">
  {{ slot }}
</div>
```

#### `<FullNavLayout>`
Adds sidebar tree (responsive).

```html
<!-- extends RootLayout -->
<div class="ppwm-layout-fullnav">
  <nav aria-label="مسیر صفحه" class="ppwm-breadcrumbs-wrapper">
    <Breadcrumbs {...breadcrumbsProps} />
  </nav>
  
  <div class="ppwm-layout-body">
    <div class="ppwm-main-container">
      {{ slot }}
    </div>
    
    <aside class="ppwm-sidebar-wrapper" aria-label="فهرست محتوا">
      <SidebarTree {...tocProps} />
    </aside>
  </div>
</div>
```

**CSS:**
```css
.ppwm-layout-fullnav {
  display: grid;
  grid-template-rows: auto 1fr;
  min-height: 100vh;
}

.ppwm-layout-body {
  display: grid;
  grid-template-columns: 1fr 20rem;
  gap: 2rem;
  padding: 2rem;
}

@media (max-width: 48rem) {
  .ppwm-layout-body {
    grid-template-columns: 1fr;
  }
  
  .ppwm-sidebar-wrapper {
    position: fixed;
    inset-block-start: 0;
    inset-inline-end: -100%;
    width: 80%;
    height: 100vh;
    background: var(--ppwm-bg-secondary);
    transition: inset-inline-end 0.3s ease;
    z-index: 1000;
  }
  
  .ppwm-sidebar-wrapper.is-open {
    inset-inline-end: 0;
  }
}
```

---

### 5.2 Core Components

Due to length constraints, I'll provide the **complete, production-ready** versions of the most critical components:

#### `<ContentBlock>`

**Template:**
```html
<div class="ppwm-content-block" id="block-{{ number }}">
  {% if has_annotations %}
    <details class="ppwm-annotations" open="{{ not start_collapsed }}">
      <summary class="ppwm-annotations-summary">
        <span class="ppwm-block-number">{{ number }}</span>
        <div class="ppwm-excerpt-preview">
          <Excerpt excerpt="{{ excerpt }}" />
        </div>
      </summary>
      
      <div class="ppwm-annotations-content">
        {% if glossary %}
          <section class="ppwm-annotation-section">
            <h4>واژگان و ترکیبات</h4>
            <Glossary entries="{{ glossary }}" />
          </section>
        {% endif %}
        
        {% if meaning %}
          <section class="ppwm-annotation-section">
            <h4>معنی روان</h4>
            <p class="ppwm-meaning-text">{{ meaning }}</p>
          </section>
        {% endif %}
        
        {% if notes %}
          <section class="ppwm-annotation-section">
            <h4>نکات</h4>
            <ol class="ppwm-notes-list">
              {% for note in notes %}
                <li>{{ note }}</li>
              {% endfor %}
            </ol>
          </section>
        {% endif %}
        
        {% if references %}
          <section class="ppwm-annotation-section">
            <h4>ارجاعات</h4>
            <ul class="ppwm-refs-list">
              {% for ref in references %}
                <li>{{ ref }}</li>
              {% endfor %}
            </ul>
          </section>
        {% endif %}
      </div>
    </details>
  {% else %}
    <div class="ppwm-excerpt-standalone">
      <span class="ppwm-block-number">{{ number }}</span>
      <Excerpt excerpt="{{ excerpt }}" />
    </div>
  {% endif %}
</div>
```

**CSS:**
```css
.ppwm-content-block {
  margin-block-end: 2rem;
  padding: 1.5rem;
  background: var(--ppwm-bg-secondary);
  border-radius: 0.5rem;
}

.ppwm-block-number {
  display: inline-block;
  min-width: 3ch;
  font-size: var(--ppwm-ftsz-text);
  font-weight: 700;
  color: var(--ppwm-accent-primary);
  text-align: center;
}

.ppwm-annotations-summary {
  cursor: pointer;
  list-style: none;
  display: flex;
  gap: 1rem;
  align-items: baseline;
}

.ppwm-annotations-summary::-webkit-details-marker {
  display: none;
}

.ppwm-annotation-section {
  margin-block-start: 1.5rem;
  padding-block-start: 1.5rem;
  border-block-start: 1px solid var(--ppwm-border);
}

.ppwm-annotation-section h4 {
  font-size: var(--ppwm-ftsz-text);
  margin-block-end: 0.75rem;
  color: var(--ppwm-text-secondary);
}
```

---

#### `<Excerpt>`

**Template:**
```html
{% if type == 'prose' %}
  <p class="ppwm-excerpt-prose">{{ lines[0] }}</p>

{% elif type == 'hemistich' %}
  <div class="ppwm-excerpt-hemistich">
    <span class="ppwm-hemistich-line">{{ lines[0] }}</span>
  </div>

{% elif type == 'couplet' %}
  <div class="ppwm-excerpt-couplet">
    <span class="ppwm-hemistich ppwm-hemistich-1">{{ lines[0] }}</span>
    <span class="ppwm-hemistich ppwm-hemistich-2">{{ lines[1] }}</span>
  </div>

{% elif type == 'enjambed' %}
  <div class="ppwm-excerpt-enjambed">
    {% for i in range(0, lines|length, 2) %}
      <div class="ppwm-couplet-row">
        <span class="ppwm-hemistich ppwm-hemistich-1">{{ lines[i] }}</span>
        {% if i + 1 < lines|length %}
          <span class="ppwm-hemistich ppwm-hemistich-2">{{ lines[i+1] }}</span>
        {% endif %}
      </div>
    {% endfor %}
  </div>
{% endif %}
```

**CSS:**
```css
.ppwm-excerpt-prose {
  font-size: var(--ppwm-ftsz-excerpt);
  line-height: var(--ppwm-lh-prose);
  text-align: justify;
  text-align-last: right;
}

.ppwm-excerpt-hemistich {
  font-size: var(--ppwm-ftsz-excerpt);
  line-height: var(--ppwm-lh-verse);
  text-align: center;
  font-weight: 700;
}

.ppwm-excerpt-couplet,
.ppwm-couplet-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  font-size: var(--ppwm-ftsz-excerpt);
  line-height: var(--ppwm-lh-verse);
  font-weight: 700;
}

.ppwm-hemistich-1 {
  text-align: left;
}

.ppwm-hemistich-2 {
  text-align: right;
}

@media (max-width: 48rem) {
  .ppwm-excerpt-couplet,
  .ppwm-couplet-row {
    grid-template-columns: 1fr;
    text-align: center;
  }
  
  .ppwm-hemistich-1,
  .ppwm-hemistich-2 {
    text-align: center;
  }
}
```

---

## Final Recommendations

### What to do NOW:
1. **Delete unused types** (`Poet`, `Work`) from your spec
2. **Standardize naming**: All component inputs get `Props` suffix
3. **Complete missing components**: I can provide full specs for `<SidebarTree>`, `<MetadataBlock>`, `<ContributorBlock>` if needed
4. **Document your Astro/Jinja2 hybrid**: Clarify which template engine you're actually using

### What NOT to do:
- Don't try to pass entire `Work` objects to components — pass only the data needed for that specific render
- Don't create circular dependencies (e.g., `Work` containing `Poet` containing `Work[]`)
- Don't mix domain logic into presentation components

Would you like me to continue with the remaining components (`<MetadataBlock>`, `<SidebarTree>`, `<ContributorBlock>`, etc.)?