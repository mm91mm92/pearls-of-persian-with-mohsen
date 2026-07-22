1. Start numbers from ------.
2. Please attribute sources you used for your annotation, paraphrase, etc. in `sources` field of the front matter.




```markdown
---
schema: 1
title: "<concise_gui_title>"
status: "<draft|final|archived>"

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
Template for annotated poem documents.

- One Markdown file should represent one logical document (e.g., a chapter, a ghazal, or another coherent unit).
- Repeat the verse/sentence block for each verse/sentence.
- Optional sections (e.g., words or phrases, literary notes, or references) may be omitted if not applicable.
- The order among sections (H4 headings) heading are important.
-->

# <stylized_page_title>

<!-- The H2 headings are optional for major, one-level divisions among verses and/or sentences -->
## <subtitle_1>

<!-- The verse/sentence block -->
### <persian_numeral>

<!-- The text of the verse or the sentence, this heading is compulsory -->
#### متن

<first_hemistich_or_prose_sentence>

<second_hemistich_if_verse>

<!-- This H4 can be ommitted if there is no difficult word or phrase to explain -->
#### واژگان و ترکیبات

| واژه/ترکیب | معنی/توضیح |
|---:|---|
| **<word_1>** | <explanation_1> |

<!-- This H4 heading is mandatory -->
#### معنی روان

<prose_translation>

<!-- This H4 heading is optional and can be ommitted -->
#### نکات ادبی

<literary_commentary>

<!-- This H4 heading is optional and can be ommitted -->
#### ارجاعات

- [<internal_reference_text>](relative/path/to/file.md)
- <external_citation>

---

<!-- Repeat verse/sentence block for the next verse or sentence -->

```

















```markdown
---
title: "<document_title>"
status: "<draft|reviewed|published>"

contributors: 
  - id: "<the_id_of_contributor1>"
    role: "<the_role_of_contributor1>"
  - id: "<the_id_of_contributor2>"
    role: "<the_role_of_contributor2>"

tags: 
  - <tag_1>             # optional
  - <tag_2>             # optional
  - <tag_3>             # optional

sources: 
  - <src_1>             # optional
  - <src_2>             # optional

notes: |                # optional                  
  <string>   
  <string>  
---

<!--
Template for annotated poem documents.

- One Markdown file should represent one logical document (e.g., a chapter, a ghazal, or another coherent unit).
- Repeat the verse block for each verse.
- Optional sections (e.g., literary notes or references) may be omitted if not applicable.
-->

# \<title>

<!-- Optional: Add additional subtitle sections as needed -->

## <subtitle_1>

<!-- The start of the verse block -->
### بیت <verse_number>

#### متن

**<first_hemistich>**

**<second_hemistich>**

#### واژگان و ترکیبات

| واژه/ترکیب | معنی/توضیح |
|---:|---|
| **<word_phrase_1>** | <explanation_1> |
| **<word_phrase_2>** | <explanation_2> |

#### معنی روان

<fluent_meaning_n1>

#### نکات ادبی

<optional_literary_notes>

#### ارجاعات

<optional_references>

<!-- The end of the verse block -->

---

<!-- Repeat the verse block for the next verse -->

```