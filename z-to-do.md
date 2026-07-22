# To-do list

## Content Layer

1. **Add the optional `prosody` field to metadata:**  
	```
	prosody: "فعولن فعولن فعولن فعل"  # وزن شعر
	```

2. Currently a verse/sentence block has five sections logically ordered as:
	1. Text
	2. Words & Phrases
	3. Fluent Meaning
	4. Literary Notes
	5. References
Of these subdivisions, "Text" and "Fluent Meaning" are only necessary ones. How about making all but Text optional? Some verses and sentences are clear and does not need any explanation in the form of words and phrases or contemporary prose.

3. Any verse/sentence block has a subdivision called Fluent Meaning. What is the best title for it: Fluent Meaning, Contemporary Prose, or something else?

4. The current design of verse/sentence block does not support "موقوف‌المعانی" verses. How about defining the syntax of Text section as:
	1. **Prose:** Contains only one line,
	2. **Verse:** Contains an event number of lines with only **one empty line** between every two consecutive text lines.

5. We used Poem Document to refer to files containing literary text and their annotations. How about rename it from Poem Document to Literary Document? This name suggests that it can contains poems and/or prose.

## Presentation Layer

1. The display of poem document in the presentation: 
	1. Every verse/sentence block without any optional annotation/explanation (Words & Phrases, Fluent Meaning, Literary Notes, or References) must be shown as a paragraph (`<p>` element).
	2. With at least one such explanatory subdivisions, it must e shown as a Disclosure Widget with the verse/verses/sentence as **Summary label** and explanations/annotations as Disclosure Content.
