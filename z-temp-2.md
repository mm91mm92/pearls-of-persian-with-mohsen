## Change 6: Change `schema` from Integer (`1`) to Float (`1.1`) for Compatibility Signaling

### What it is
You want `schema` to carry more information than just “version 1”, so that:
- **Backward-compatible** changes can increment a minor version (e.g., `1.0 → 1.1`)
- **Backward-incompatible** changes can increment a major version (e.g., `1.x → 2.0`)

### Assessment: ✅ Approve the goal, ❌ do not recommend using a float
The *idea* (distinguishing compatible vs incompatible revisions) is solid and worth doing early.  
But encoding versions as a **float number** (e.g., `1.1`) is a fragile choice in YAML/tooling and will eventually create avoidable problems.

### Why a float is not a good fit (critical issues)
1. **YAML numeric ambiguity (`1`, `1.0`, and sometimes `1.00`)**
   - Many parsers treat `1`, `1.0`, and `1.00` as the *same numeric value*.
   - That undermines your intent to precisely represent versions and compare them reliably.

2. **Floating-point issues and formatting drift**
   - Even if you write `1.10`, numerically that’s indistinguishable from `1.1`.
   - Some serializers may reformat `1.10` to `1.1`, silently changing meaning if you ever relied on formatting.

3. **Versioning runs into “decimal math” limitations**
   - Humans read versions as dotted components, not decimals.
   - With floats you eventually face confusing transitions like `1.9 → 1.10` (is that “one point ten” or “one point one”?).

### Recommended refinement (what I *am* okay with)
Use **semantic versions as a string**, not a float:

```yaml
schema: "1.1.0"
```

This achieves exactly what you want—clean separation of compatibility types—without numeric pitfalls.

#### Why this is better
- **Unambiguous:** `"1.1.0"` is a string; no parser will collapse it into `1`
- **Standard:** aligns with how most specs evolve (SemVer-style)
- **Future-proof:** lets you use:
  - **MAJOR** for breaking changes
  - **MINOR** for backward-compatible feature additions (like `prosody`)
  - **PATCH** for clarifications, typo fixes, example improvements

### Compatibility meaning (how the spec should interpret it)
If you adopt `"MAJOR.MINOR.PATCH"`:

- **Backward-incompatible change → bump MAJOR**
  - Renaming required fields, changing required headings, changing parsing rules such that old files become invalid.
- **Backward-compatible change → bump MINOR**
  - Adding optional fields, making formerly-required sections optional, adding additional allowed text forms.
- **Documentation-only or non-behavioral updates → bump PATCH**
  - Examples, wording clarity, non-normative notes.

### How this interacts with your other approved changes
Given the changes you’re making (adding optional fields, loosening requirements, supporting enjambed verses), these are predominantly **backward-compatible** in spirit—so a minor bump fits well (e.g., from `"1.0.0"` to `"1.1.0"`).

### Bottom line
- I agree you should evolve beyond `schema: 1`.
- I do **not** recommend `schema: 1.1` as a float.
- The clean, robust way to get exactly your compatibility signaling is:

```yaml
schema: "1.1.0"
```