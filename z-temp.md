# Pearls of Persian: Presentation Layer Specification v1.0.0

---

## 1. Design Principles

### 1.1 Core Philosophy
The presentation layer serves classical Persian literature with the same reverence and clarity that the content layer preserves it. Every design decision prioritizes **readability, scholarly rigor, and timeless aesthetics** over trends or visual flourish.

### 1.2 Guiding Principles

| Principle | Description |
|-----------|-------------|
| **Content First** | Typography, spacing, and hierarchy serve the poetry, not the interface. Visual elements never compete with the text. |
| **Scholarly Precision** | Metadata, sources, and contributors are treated as first-class information, not footnotes. |
| **Timeless Aesthetics** | Design draws from classical Persian manuscript traditions: generous margins, clear hierarchy, ornamental restraint. |
| **Performance as a Feature** | Pages load instantly. No JavaScript is required for core reading experience. Progressive enhancement only. |
| **Accessible by Default** | RTL support, semantic HTML, keyboard navigation, and screen reader compatibility are non-negotiable from day one. |
| **Separation of Concerns** | The presentation layer is a **rendering engine**, not a content editor. All content modification happens in the repository. |

### 1.3 Non-Goals
- **Not a web app**: No client-side routing, no hydration of complex state, no "app shell" architecture.
- **Not a CMS**: No in-browser editing, no admin panel, no user-generated content submission.
- **Not a social platform**: No comments, likes, or user profiles (contributors are metadata, not accounts).

---

## 2. URL Structure

### 2.1 Mapping Principle
URLs mirror the physical `content/` directory structure exactly, using Persian script slugs derived from directory and file names.

### 2.2 URL Format
```
https://pearls-of-persian.com/<poet>/<book>/<section>/<document>
```

**Examples:**
```
/عطار-نیشابوری
/عطار-نیشابوری/منطق-الطیر
/عطار-نیشابوری/منطق-الطیر/مقدمه
/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان
```

### 2.3 Slug Generation Rules
- Use the exact directory/file name (minus `.md` extension) as the URL segment
- Preserve Persian characters, hyphens, and ZWNJ (نیم‌فاصله) exactly as authored
- No transliteration, no Latin fallbacks
- URL-encode as UTF-8 (browsers handle display automatically)

### 2.4 Special Routes

| Route | Purpose |
|-------|---------|
| `/` | Site root: featured collections, recent additions, project introduction |
| `/search` | Full-text search interface |
| `/contributors` | Alphabetical directory of all contributors with their profiles |
| `/contributors/<id>` | Individual contributor detail page |
| `/about` | Project mission, schema documentation, contribution guidelines |

### 2.5 Canonicalization
- Trailing slashes are normalized (redirect `/عطار-نیشابوری/` → `/عطار-نیشابوری`)
- All URLs are case-sensitive (Persian text is case-aware)
- No alternate slug formats (no "pretty" English URLs)

---

## 3. Navigation

### 3.1 Breadcrumb Structure
Breadcrumbs appear at the top of every page, except for the Home page, above the site header, built dynamically from the `meta.yml:title` chain.

**Visual format:**
```
گوهرهای پارسی با محسن  >  عطار نیشابوری  >  منطق‌الطیر  >  مقدمه  >  مجمع مرغان
```

**Semantic HTML:**
```html
<nav aria-label="مسیر صفحه">
  <ol class="breadcrumbs">
    <li><a href="/">گوهرهای پارسی با محسن</a></li>
    <li><a href="/عطار-نیشابوری">عطار نیشابوری</a></li>
    <li><a href="/عطار-نیشابوری/منطق-الطیر">منطق‌الطیر</a></li>
    <li><a href="/عطار-نیشابوری/منطق-الطیر/مقدمه">مقدمه</a></li>
    <li aria-current="page">مجمع مرغان</li>
  </ol>
</nav>
```

**Rules:**
- Root (`/`) always uses the site title from root `meta.yml`
- Each segment uses `meta.yml:title` (not the directory name)
- Current page is not a link, uses `aria-current="page"`
- Separator is ` > ` with proper spacing

### 3.2 Sidebar Navigation (Hierarchy Tree)
On directory-level pages (`poet`, `book`, `section`, etc.) and also document pages, a collapsible (responsive) sidebar, on the right side of the HTML page, displays the canonical structure defined by `meta.yml:children` arrays.

**Visual behavior:**
- Current location is highlighted and auto-expanded
- Other expandable nodes are collapsed
- Clicking a directory title or a document title navigates to its index page or its literary page repectively
- Expand/collapse icons indicate child presence
- Respects `meta.yml:children` order exactly (no alphabetical sorting)

**Rendering logic:**
```typescript
function buildNavTree(metaYml) {
  return metaYml.children.map(child => ({
    title: getChildMeta(child).title,
    href: buildUrl(child),
    children: getChildMeta(child).children || []
  }))
}
```

### 3.3 Table of Contents (Document-Level)
Literary documents (`.md` files) display an auto-generated table of contents based on H2 headings.

**Structure:**
- Only shows H2 heading, ignoring H3 (content blocks) and H4 (literary explanations) headings
- Smooth scroll to anchor on click
- Active section is highlighted during scroll

**Example TOC:**
```
در این صفحه:
  خطاب به هدهد
  خطاب به موسیچه
  خطاب به طوطی
```

---

## 4. Page Types

### 4.1 Root Page (`type: root`)
**URL:** `/`

**Purpose:** Entry point to the site; introduces the project and provides access to top-level collections.

**Content sections:**
- Hero: Site title, tagline, brief mission statement
- Recent additions: Last 5 documents added (derived from Git history or build timestamp)
- Favorites: The top 5 favorite (the most visited) literary documents
- All poets: A list (maybe a fantasy list view) featuring all available poets
- About link, contributor link, schema documentation link

**Metadata displayed:**
- None (this is an index, not a document)

---

### 4.2 Poet Page (`type: poet`)
**URL:** `/عطار-نیشابوری`

**Purpose:** Overview of a poet's life, works, and metadata; gateway to their books.

**Content sections:**
- **Breadcrumb:** Must show the name of the poet after the title of the project:
	```
	گوهرهای پارسی با مخین > عطار نیشابوری
	```
In HTML:
	```html
	<nav aria-label="مسیر صفحه">
	  <ol class="breadcrumbs">
	    <li><a href="/">گوهرهای پارسی با محسن</a></li>
	    <li aria-current="page">عطار نیشابوری</li>
	  </ol>
	</nav>
	```
- **Header:**
  - Poet name (from `meta.yml:title`)
  - Dates, biographical summary (from `meta.yml:description`)
  - Tags (from `meta.yml:tags`)
- **Works list:**
  - Ordered list derived from `meta.yml:children`
  - Each work shows title, type badge (e.g., مثنوی, دیوان), and optional description
  - Click navigates to book page

**Metadata displayed:**
- `description` (if present)
- `tags` (if present)

**Example:**
```
عطار نیشابوری
فریدالدین ابوحامد محمد عطار نیشابوری (متوفی حدود ۶۱۸ ه.ق.) 
از بزرگترین شاعران و عارفان ایران است.

آثار او در این مجموعه:
  • منطق‌الطیر (مثنوی)
  • الهی‌نامه (مثنوی)
```

---

### 4.3 Book Page (`type: book`)
**URL:** `/عطار-نیشابوری/منطق-الطیر`

**Purpose:** Introduce a literary work; provide access to its structural divisions.

**Content sections:**
- **Breadcrumb:** 
	```
	گوهرهای پارسی با مخین > عطار نیشابوری > منطق‌الطیر
	```
In HTML:
	```html
	<nav aria-label="مسیر صفحه">
	  <ol class="breadcrumbs">
	    <li><a href="/">گوهرهای پارسی با محسن</a></li>
	    <li><a href="/">عطار نیشابوری</a></li>
	    <li aria-current="page">منطق‌الطیر</li>
	  </ol>
	</nav>
	```
- **Header:**
  - Book title (derived from current `meta.yml:title`)
  - Author name (derived from parent poet `meta.yml:title`)
  - Description (from `meta.yml:description`)
  - Tags
- **Structure navigator:**
  - Tree or flat list of children (offices, chapters, sections)
  - Each item shows title + child count

**Metadata displayed:**
- `description` (if present)
- `tags` (if present)

---

### 4.4 Section/Chapter/Office Pages (`type: section | chapter | office`)
**URL:** `/عطار-نیشابوری/منطق-الطیر/مقدمه`

**Purpose:** Intermediate navigation level; lists documents within this division.

**Content sections:**
- **Breadcrumb:** 
	```
	گوهرهای پارسی با مخین > عطار نیشابوری > منطق‌الطیر > مقدمه
	```
In HTML:
	```html
	<nav aria-label="مسیر صفحه">
	  <ol class="breadcrumbs">
	    <li><a href="/">گوهرهای پارسی با محسن</a></li>
	    <li><a href="/">عطار نیشابوری</a></li>
	    <li><a href="/">منطق‌الطیر</a></li>
	    <li aria-current="page">مقدمه</li>
	  </ol>
	</nav>
	```
- **Header:**
  - Section title
  - Optional description
- **Document list:**
  - Ordered by `meta.yml:children`
  - Each entry shows:
    - Document title (front matter `title`)
    - Status badge (`draft`, `final`, `archived`)

**Metadata displayed:**
- `description` (if present)
- `tags` (if present)

---

### 4.5 Literary Document Page (Leaf `.md` file)
**URL:** `/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان`

**Purpose:** The core reading experience; displays annotated poetry or prose.

**Content sections (in order):**

#### Breadcrumbs

#### Header
- **Document title** (H1 from Markdown body, not front matter `title`)
- **Metadata panel:**
  - "Last updated" timestamp (from Git history or build time)
  - Status badge
  - Contributors (linked to their profiles)
  - Tags (linked to tag filter pages, if implemented)
  - Prosody notation (if `prosody` field exists)
  - Sources (bibliographic references)

#### Table of Contents
- Auto-generated from H2 headings ignoring H3 and H4 headings
- Collapsible (responsive) on the left side of the page

#### Content Body
- H2 sections (subtitles) render as visual dividers if existed

#### Footer
- Link to edit this document on GitHub
- Schema version used (e.g., "تهیه‌شده با طرح نسخهٔ ۱.۱.۰")

**Metadata displayed:**
- `status` (badge)
- `contributors` (list with roles)
- `tags` (linked chips)
- `prosody` + `prosody_name` (if present)
- `sources` (formatted bibliography)

---

### 4.6 Contributor Profile Page
**URL:** `/contributors/mm91mm92`

**Purpose:** Display a contributor's identity, bio, and body of work within the project.

**Content sections:**
- **Header:**
  - Persian name (large, primary)
  - Latin name (smaller, secondary)
  - Affiliation (if present)
  - Location (if present)
- **Biography:**
  - Persian bio (primary)
  - English bio (secondary, collapsed by default if both exist)
- **Links:**
  - External profiles (GitHub, ORCID, etc.)
- **Contributions:**
  - List of documents this contributor worked on, grouped by role
  - Each entry links to the document and shows their role

**Example:**
```
محسن ملک‌محمدی
Mohsen Malekmohammadi

دانشگاه پیام‌نور

درباره:
پژوهشگر و نگه‌دارندهٔ پروژه «گوهرهای پارسی با محسن» با تخصص در
شعر کلاسیک فارسی و ادبیات عرفانی.

پیوندها:
  GitHub: <github_string>
  ORCID: <orcid_string>
  LinkedIn: <orcid_string>

مشارکت‌ها (۲۳ سند):
  محقق:
    • منطق‌الطیر / مقدمه / مجمع مرغان
    • منطق‌الطیر / مقدمه / خطاب هدهد
```

---

### 4.7 Search Results Page
**URL:** `/search?q=عشق+الهی`

**Purpose:** Display matching documents for user query.

**Content sections:**
- **Search input** (pre-filled with query)
- **Results count** (e.g., "۱۲ نتیجه برای «عشق الهی»")
- **Filters** (optional):
  - By poet
  - By book
  - By status (`final` only, etc.)
- **Result cards:**
  - Document title
  - Breadcrumb path
  - Matching snippet (highlighted)
  - Link to document

**Ranking:**
- Exact phrase match > word match
- Title match > body match
- `final` status > `draft`

---

## 5. Components

### 5.1 Core Layout Components

#### `<SiteHeader>`
**Purpose:** Top-level site navigation and branding.

**Contents:**
- Site logo/title (linked to `/`)
- Main navigation links (About, Contributors, Search)
- Dark mode toggle

**Behavior:**
- Sticky on scroll (optional; test for readability)
- Collapses to hamburger menu on mobile

---

#### `<Breadcrumbs>`
**Purpose:** Contextual navigation (see §3.1).

**Props:**
```typescript
interface BreadcrumbsProps {
  segments: Array<{ title: string; href: string }>
}
```

---

#### `<MetadataPanel>`
**Purpose:** Display document front matter metadata.

**Props:**
```typescript
interface MetadataPanelProps {
  last_modified: string
  status: 'draft' | 'final' | 'archived'
  poet: string
  book: string
  contributors: Array<{ id: string; name_fa: string; role: string }>
  tags?: string[]
  prosody?: string
  prosody_name?: string
  sources?: string[]
}
```

**Visual design:**
- Compact, sidebar-style on desktop
- Full-width panel on mobile
- Each metadata type uses an icon + label

---

### 5.2 Content Block Components

#### `<ContentBlock>`

**Purpose:** Represents a content block in the literary document.

**Props:**
```typescript
interface LiteraryTextBlockProps {
  number: string          // e.g., "۱"
  excerpt: ExcerptBlock
  vocabulary?: VocabEntry[]
  meaning?: string
  literaryNotes?: string
  references?: string[]
}
```

**Rendering:** 

#### `<ExcerptBlock>`

**Props:**
```typescript
interface ExcerptBlock {
  isVerse: boolean
  lines: string[]
}
```

**Rendering:** the compiler must determine whether the corresponding `#### متن` represents a piece of prose or a sequence of hemistichs (verse). Three situations are possible:

1. Prose (`isVerse` is `false` and there is only one `lines`): The component must be rendered as a simple right-aligned `<p>` element.
2. Even-hemistich verse (`isVerse` is `true` and there are even number of `lines`): The component must be rendered as two columns with odd hemistichs on the right column (the column itself must be left-aligned) and even hemistichs on the left column (the column itself must be right-aligned). The implementation, of course, must be responsive, i.e. on small screens all hemistichs must be stacked  on top of each other preserving their order in `lines` field. Any implementation which ensures this is sufficient such as CSS flex, grid, or HTML tables.
3. Odd-hemistich verse (`isVerse` is `true` and `lines` has an odd number of elements): The odd number can be decomposed as `odd = even + the last single hemistich`. The even part must be rendered according to the previous situation and the last single hemistich must be implemented as a `<p>` element after that, of course this trailing hemistich must be center-aligned.

#### `<VerseBlock>`
**Purpose:** Render a single verse (بیت) with proper Persian typography and spacing.

**Props:**
```typescript
interface VerseBlockProps {
  number: string          // e.g., "۱"
  hemistichs: [string, string]
  vocabulary?: VocabEntry[]
  meaning?: string
  literaryNotes?: string
  references?: string[]
}
```

**Rendering:**
- **Text section:**
  ```html
  <div class="verse-text">
    <p class="hemistich">عشق آمد و چون خون در رگ و پوست من دوید</p>
    <p class="hemistich">مرا تهی کرد و او را پُر کرد از دوست من دوید</p>
  </div>
  ```
- Hemistichs are center-aligned by default
- Optional: use justify-space-between for traditional manuscript look

**Optional sections rendered conditionally:**
- Vocabulary table (if `vocabulary` exists)
- Meaning block (if `meaning` exists)
- Literary notes (if `literaryNotes` exists)
- References (if `references` exists)

---

#### `<EnjambedVerseBlock>`
**Purpose:** Render موقوف‌المعانی (multi-couplet) blocks.

**Props:**
```typescript
interface EnjambedVerseBlockProps {
  number: string
  hemistichs: string[]    // 4+ items, even count
  vocabulary?: VocabEntry[]
  meaning?: string
  literaryNotes?: string
  references?: string[]
}
```

**Rendering:**
- All hemistichs in a continuous flow (not separated into couplets visually)
- Optional: subtle visual connector (e.g., left border) to indicate continuity

---

#### `<ProseBlock>`
**Purpose:** Render a prose sentence block.

**Props:**
```typescript
interface ProseBlockProps {
  number: string
  text: string
  vocabulary?: VocabEntry[]
  meaning?: string
  literaryNotes?: string
  references?: string[]
}
```

**Rendering:**
- Text is justified (not centered like verses)
- Optional subtle background to distinguish from verse

---

#### `<VocabularyTable>`
**Purpose:** Render glossary of difficult words/phrases.

**Props:**
```typescript
interface VocabularyTableProps {
  entries: Array<{ term: string; explanation: string }>
}
```

**HTML structure:**
```html
<table class="vocabulary">
  <tbody>
    <tr>
      <th>سیمرغ</th>
      <td>پرنده‌ای افسانه‌ای که در ادبیات عرفانی نماد ذات حق است</td>
    </tr>
  </tbody>
</table>
```

**Styling:**
- Right-aligned term column (bold)
- Left-aligned explanation column
- Responsive: stacks on mobile

---

#### `<ProsodyNotation>`
**Purpose:** Display metrical pattern with proper formatting.

**Props:**
```typescript
interface ProsodyNotationProps {
  pattern: string       // e.g., "فعولن فعولن فعولن فعل"
  name?: string         // e.g., "رمل مثمن محذوف"
}
```

**Rendering:**
```
وزن: فعولن فعولن فعولن فعل
     (رمل مثمن محذوف)
```

---

#### `<ContributorCard>`
**Purpose:** Display contributor info in lists or inline.

**Props:**
```typescript
interface ContributorCardProps {
  id: string
  name_fa: string
  role: string
  affiliation_fa?: string
  compact?: boolean    // Inline vs. full card
}
```

---

#### `<StatusBadge>`
**Purpose:** Visual indicator for document workflow state.

**Props:**
```typescript
interface StatusBadgeProps {
  status: 'draft' | 'final' | 'archived'
}
```

**Visual mapping:**
| Status | Color | Persian Label | Icon |
|--------|-------|--------------|------|
| `draft` | Yellow | پیش‌نویس | 🟡 |
| `final` | Green | تکمیل‌شده | ✅ |
| `archived` | Gray | بایگانی‌شده | ⚫ |

---

### 5.3 Interactive Components

#### `<SearchBox>`
**Purpose:** Full-text search input with autocomplete.

**Behavior:**
- Debounced input (300ms)
- Shows top 5 suggestions on focus
- Submits to `/search?q=...`

---

#### `<TableOfContents>`
**Purpose:** Auto-generated TOC with scroll spy.

**Behavior:**
- Sticky positioning on desktop
- Highlights current section during scroll
- Smooth scroll to anchor on click

---

#### `<ThemeToggle>`
**Purpose:** Switch between light and dark modes.

**Behavior:**
- Persists preference to `localStorage`
- Respects `prefers-color-scheme` by default
- No flash of unstyled content (inline script in `<head>`)

---

## 6. Typography

### 6.1 Font Stack
**Primary (Persian & Arabic):**
```css
font-family: 'Vazirmatn', 'Noto Naskh Arabic', 'Traditional Arabic', 'Scheherazade New', system-ui, sans-serif;
```

**Latin (UI & metadata):**
```css
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
```

**Monospace (code, Git SHAs):**
```css
font-family: 'Fira Code', 'Cascadia Code', Consolas, monospace;
```

### 6.2 Font Sizing (Fluid Scale)
| Element | Desktop | Mobile | CSS Variable |
|---------|---------|--------|--------------|
| H1 (Document title) | 2.5rem | 2rem | `--font-size-h1` |
| H2 (Subtitle) | 2rem | 1.75rem | `--font-size-h2` |
| H3 (Block number) | 1.5rem | 1.25rem | `--font-size-h3` |
| Body (prose) | 1.25rem | 1.125rem | `--font-size-body` |
| Body (verse) | 1.5rem | 1.25rem | `--font-size-verse` |
| Small (metadata) | 0.875rem | 0.875rem | `--font-size-small` |

**Rationale:**
- Verses are larger than prose to honor their centrality
- Generous sizing aids readability of Persian script
- Fluid scaling via `clamp()` for smooth responsive behavior

### 6.3 Line Height
| Context | Line Height | Reasoning |
|---------|-------------|-----------|
| Verse (hemistichs) | 2.0 | Generous spacing for contemplation |
| Prose | 1.8 | Readable but more compact than verse |
| UI text | 1.5 | Standard interface spacing |

### 6.4 Text Alignment
- **Verses:** Center-aligned (default Persian manuscript tradition)
- **Prose:** Justified with hyphenation disabled (Persian doesn't hyphenate well)
- **Metadata, UI:** Right-aligned (RTL primary direction)

### 6.5 Special Typography Rules
- **ZWNJ (نیم‌فاصله):** Rendered correctly; no breaking or collapsing
- **Kashida:** Avoided in web fonts (inconsistent rendering)
- **Diacritics:** Supported if present in source text (not added by presentation layer)

---

## 7. Colors

### 7.1 Design Philosophy
Color palette draws inspiration from **classical Persian manuscripts**: warm cream/beige backgrounds, deep ink blacks, and subtle accent colors derived from traditional miniature painting.

### 7.2 Light Mode Palette
| Token | Hex | Usage |
|-------|-----|-------|
| `--bg-primary` | `#F9F6F1` | Main background (cream) |
| `--bg-secondary` | `#EFE9E0` | Sidebar, cards |
| `--text-primary` | `#1A1614` | Body text, verse |
| `--text-secondary` | `#4A4542` | Metadata, captions |
| `--accent-primary` | `#8B4513` | Links, active states (سبز کهربایی) |
| `--accent-secondary` | `#2C5F7C` | Badges, highlights (لاجوردی) |
| `--border` | `#D4C7B8` | Dividers, card borders |

### 7.3 Dark Mode Palette
| Token | Hex | Usage |
|-------|-----|-------|
| `--bg-primary` | `#1A1614` | Main background |
| `--bg-secondary` | `#2A2421` | Sidebar, cards |
| `--text-primary` | `#E8E2D9` | Body text, verse |
| `--text-secondary` | `#B8AEA3` | Metadata, captions |
| `--accent-primary` | `#D4A574` | Links, active states |
| `--accent-secondary` | `#6B9FBB` | Badges, highlights |
| `--border` | `#3A342F` | Dividers, card borders |

### 7.4 Semantic Colors (Status Badges)
| Status | Light | Dark |
|--------|-------|------|
| Draft | `#FFC107` | `#FFB300` |
| Final | `#4CAF50` | `#66BB6A` |
| Archived | `#9E9E9E` | `#757575` |

### 7.5 Color Accessibility
- All text/background pairs meet **WCAG AA** contrast (4.5:1 minimum)
- Links meet **WCAG AAA** contrast (7:1 minimum)
- Status badges use both color + icon for colorblind users

---

## 8. Responsive Design

### 8.1 Breakpoints
```css
/* Mobile-first approach */
--bp-sm: 640px   /* Tablet portrait */
--bp-md: 768px   /* Tablet landscape */
--bp-lg: 1024px  /* Desktop */
--bp-xl: 1280px  /* Wide desktop */
```

### 8.2 Layout Behavior by Breakpoint

#### Mobile (`< 640px`)
- Single column
- Sidebar navigation becomes hamburger menu
- Breadcrumbs collapse (show only current + parent)
- TOC hidden by default (expandable)
- Verse font size slightly reduced
- Metadata panel full-width, collapsed by default

#### Tablet (`640px – 1024px`)
- Two-column where appropriate (sidebar + content)
- Breadcrumbs show full path
- TOC appears as sticky panel
- Verse maintains larger sizing

#### Desktop (`> 1024px`)
- Three-column: sidebar + content + TOC
- Maximum content width: `65ch` (for prose), `45ch` (for verse)
- Generous margins (manuscript-inspired whitespace)

### 8.3 Touch Targets
- Minimum tap target: `44×44px` (WCAG guideline)
- Applies to: buttons, links, menu items, TOC links

### 8.4 Images & Media
- No decorative images in v1.0 (content purity)
- Future: contributor avatars, manuscript scans (lazy-loaded, responsive)

---

## 9. Search

### 9.1 Implementation Strategy
**Phase 1 (MVP):** Client-side static search using prebuilt index.

**Why static search:**
- Entire corpus is small enough to index in JSON (~5–10 MB even for 1000+ documents)
- No server costs, no rate limits, no API dependencies
- Works offline (PWA-ready)
- Fast enough for instant-as-you-type

**Index structure:**
```json
{
  "documents": [
    {
      "id": "عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان",
      "title": "مجمع مرغان",
      "poet": "عطار نیشابوری",
      "book": "منطق‌الطیر",
      "text": "همه مرغان عالم طالب آن شده‌اند که سیمرغ را یکدلانه ببینند...",
      "tags": ["هدهد", "سیر و سلوک"],
      "status": "final"
    }
  ]
}
```

**Library:** `minisearch` or `lunr.js` (both support Persian/RTL)

### 9.2 Indexing Rules
**What gets indexed:**
- Document title (high weight)
- H2 subtitles (medium weight)
- Verse/prose text (medium weight)
- Vocabulary terms + explanations (low weight)
- Tags (high weight)

**What doesn't get indexed:**
- Metadata labels (e.g., "محقق:", "وزن:")
- UI text
- Contributor bios (indexed separately)

### 9.3 Search Features
**MVP:**
- Full-text search
- Persian stemming (if library supports it; otherwise exact + prefix match)
- Phrase search (quoted strings)
- Filter by status (`final` only toggle)

**Future:**
- Filter by poet, book, tags
- Advanced syntax (`poet:مولوی tag:عشق`)
- Search within a specific book

### 9.4 Search UX
- **Input:** Top-right corner of header (always visible)
- **Autocomplete:** Top 5 suggestions appear on focus
- **Results page:** Shows snippets with highlighted matches
- **No results:** Suggests spelling alternatives or shows random featured poems

---

## 10. Accessibility

### 10.1 Semantic HTML
- All pages use proper heading hierarchy (`<h1>` → `<h2>` → `<h3>`, no skipping)
- Lists use `<ul>`/`<ol>`, not `<div>` chains
- Navigation uses `<nav>` with `aria-label`
- Main content wrapped in `<main>`
- Breadcrumbs use `<ol>` with `aria-label="مسیر صفحه"`

### 10.2 Keyboard Navigation
- All interactive elements reachable via `Tab`
- Visible focus indicators (high-contrast outline)
- Skip links: "برو به محتوا" at top of page
- Modal traps: dialogs trap focus until closed
- Sidebar/menu: `Escape` to close

### 10.3 Screen Reader Support
- `aria-current="page"` on active breadcrumb
- `aria-label` on icon-only buttons (e.g., theme toggle, search)
- `role="status"` on live search results count
- `alt` text on all images (if/when added)
- Language tags: `lang="fa"` on Persian content, `lang="en"` on Latin

### 10.4 RTL (Right-to-Left) Support
- `dir="rtl"` on `<html>` element
- All layouts flex/grid with `start`/`end` (not `left`/`right`)
- Text alignment uses logical properties (`text-align: start`)
- Icons/arrows flip direction in RTL (CSS `transform: scaleX(-1)`)

### 10.5 Motion & Animation
- Respects `prefers-reduced-motion`
- Smooth scroll disabled for users who prefer reduced motion
- No auto-playing animations

### 10.6 Color & Contrast
- WCAG AA minimum (already specified in §7.5)
- Status badges use icon + text (not color alone)
- Links underlined by default (not relying on color alone)

---

## 11. SEO

### 11.1 Meta Tags (Per Page)
```html
<title>مجمع مرغان - منطق‌الطیر - عطار نیشابوری | گوهرهای پارسی</title>
<meta name="description" content="تحلیل و تفسیر بیت‌های آغازین منطق‌الطیر درباره مجمع پرندگان و جستجوی سیمرغ">
<meta name="keywords" content="عطار نیشابوری, منطق‌الطیر, هدهد, سیر و سلوک, عرفان">
<meta name="author" content="محسن ملک‌محمدی">
<link rel="canonical" href="https://pearls-of-persian.com/عطار-نیشابوری/منطق-الطیر/مقدمه/مجمع-مرغان">
```

### 11.2 Open Graph (Social Sharing)
```html
<meta property="og:title" content="مجمع مرغان - منطق‌الطیر - عطار نیشابوری">
<meta property="og:description" content="تحلیل و تفسیر بیت‌های آغازین منطق‌الطیر">
<meta property="og:type" content="article">
<meta property="og:url" content="https://pearls-of-persian.com/...">
<meta property="og:locale" content="fa_IR">
```

### 11.3 Structured Data (JSON-LD)
```json
{
  "@context": "https://schema.org",
  "@type": "ScholarlyArticle",
  "headline": "مجمع مرغان",
  "author": {
    "@type": "Person",
    "name": "محسن ملک‌محمدی"
  },
  "about": {
    "@type": "Book",
    "name": "منطق‌الطیر",
    "author": "عطار نیشابوری"
  },
  "inLanguage": "fa"
}
```

### 11.4 Sitemap
- Auto-generated `sitemap.xml` from content tree
- Includes all `final` documents
- Updates on every build
- Submitted to Google Search Console

### 11.5 robots.txt
```
User-agent: *
Allow: /
Disallow: /search
Sitemap: https://pearls-of-persian.com/sitemap.xml
```

---

## 12. Performance

### 12.1 Core Web Vitals Targets
| Metric | Target | Strategy |
|--------|--------|----------|
| **LCP** (Largest Contentful Paint) | < 2.5s | Static generation, optimized fonts, minimal CSS |
| **FID** (First Input Delay) | < 100ms | No blocking JS, progressive enhancement |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Font loading with `font-display: swap`, no layout-shifting ads |

### 12.2 Asset Optimization
**Fonts:**
- Subset to Persian + Arabic + Latin ranges
- Preload critical font files
- Use `woff2` format exclusively
- Font display: `swap` (show fallback immediately, swap when loaded)

**CSS:**
- Critical CSS inlined in `<head>`
- Non-critical CSS loaded async
- No CSS frameworks (custom, minimal styles only)

**JavaScript:**
- Zero JS required for reading (static HTML + CSS only)
- Search, theme toggle, TOC interactions: progressive enhancement
- Total JS budget: < 50 KB (minified + gzipped)

**Images (future):**
- WebP with JPEG fallback
- Lazy loading (`loading="lazy"`)
- Responsive images (`srcset`)

### 12.3 Caching Strategy
- **Cloudflare Pages default caching** (static assets cached at edge)
- HTML: short cache (5 minutes) with stale-while-revalidate
- CSS/JS: long cache (1 year) with content hashing in filenames
- Fonts: long cache (1 year)

### 12.4 Build Optimization
- Incremental builds (only rebuild changed documents)
- Parallel processing of content parsing
- Minification: HTML, CSS, JS
- Tree-shaking unused code

---

## 13. Internationalization

### 13.1 Current Scope (v1.0)
**Primary language:** Persian (Farsi)  
**Secondary language:** English (for technical metadata, contributor bios, UI labels if needed)

**No multi-language content planned.** The literary corpus is Persian; UI is Persian-first.

### 13.2 Language Handling
- `lang="fa"` on Persian content
- `lang="en"` on Latin/English snippets (e.g., contributor names, Git SHAs)
- Mixed-direction handling (e.g., English bio within Persian page)

### 13.3 Future i18n (If Needed)
If the project expands to include:
- English translations of poems
- Arabic works
- Tajik/Dari variants

Then implement:
- URL structure: `/en/attar/conference-of-birds`
- Language switcher in header
- Separate content folders or multilingual frontmatter

---

## 14. Future Features

### 14.1 Phase 2 (6–12 months)
- **Advanced search filters** (by poet, book, prosody type, tags)
- **Annotations system** (collaborative scholarly notes, similar to Genius.com)
- **Audio recitation** (embedded audio for selected poems)
- **Manuscript scans** (side-by-side view with transcription)

### 14.2 Phase 3 (12–24 months)
- **User accounts** (save favorites, track reading progress)
- **API** (expose content as JSON for third-party apps)
- **Mobile app** (offline-first PWA or native app)
- **Interactive prosody analyzer** (visualize metrical patterns)

### 14.3 Not Planned
- **Comments/forums** (keep focus on scholarly work, not social features)
- **Ads or monetization** (non-commercial, academic project)
- **User-submitted content** (all contributions via Git/PR only)

---

## Appendix: Layout-to-Type Mapping

| `meta.yml:type` | Page Layout | Key Features |
|-----------------|-------------|--------------|
| `root` | Homepage | Hero, featured collections, recent additions |
| `poet` | Poet index | Bio, works list, tags |
| `book` | Book index | Description, structure navigator |
| `office` | Section index | Document list, descriptions |
| `chapter` | Section index | Document list, descriptions |
| `section` | Section index | Document list, descriptions |
| `genre` | Section index | Document list (غزلیات, رباعیات, etc.) |
| `collection` | Section index | Generic container for uncategorized groups |
| (`.md` file) | Literary document | Full annotated text with metadata, TOC, verse blocks |

---

**End of Presentation Layer Specification v1.0.0**