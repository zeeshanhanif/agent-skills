# Word (.docx) Generation

Generate `docs/srs.docx` and `docs/use-cases.docx` from the **finalized** markdown
sources, as the last step (Phase 7). Because the markdown is the checkpointed
source of truth, the Word files are a pure conversion — never author content
directly in Word.

## Primary method — Pandoc

Pandoc is the cleanest markdown→docx path and renders the tables and headings an
SRS needs.

```bash
pandoc docs/srs.md -o docs/srs.docx
pandoc docs/use-cases.md -o docs/use-cases.docx
```

For a more polished result, generate (once) and reuse a reference document for
styling, and add a table of contents:

```bash
# create a style reference once, then customize its styles in Word if desired
pandoc -o reference.docx --print-default-data-file reference.docx

pandoc docs/srs.md \
  --reference-doc=reference.docx \
  --toc --toc-depth=3 \
  -o docs/srs.docx
```

**Mermaid diagrams in the use-case doc:** pandoc does not render Mermaid code
blocks into images by default — they'll appear as code. Options, in order of
preference:
- Use a Mermaid-aware filter if available in the environment (e.g.,
  `mermaid-filter`): `pandoc docs/use-cases.md -F mermaid-filter -o docs/use-cases.docx`.
- Otherwise, pre-render each diagram to PNG/SVG (if `mmdc` /
  `@mermaid-js/mermaid-cli` is installed) and reference the images in a
  docx-specific copy of the markdown.
- If neither is available, note to the user that the `.docx` shows the diagram as
  Mermaid source and the rendered diagrams live in the markdown version (which
  renders on GitHub); offer to install a renderer.

## Check availability first

Before converting, check what's installed and adapt:

```bash
command -v pandoc   # primary
command -v mmdc     # mermaid CLI, for diagram images
```

If `pandoc` is missing, tell the user it's needed for `.docx` output and how to
install it (on macOS: `brew install pandoc`; or `npm i -g @mermaid-js/mermaid-cli`
for the diagram renderer). Do not silently skip the Word deliverables — they were
explicitly requested.

## Fallback — python-docx

If pandoc cannot be installed, generate the docx programmatically with
`python-docx` (`pip install python-docx --break-system-packages`): parse the
finalized markdown and write headings, paragraphs, and tables. This is more code
and styles tables less richly than pandoc, so prefer pandoc when available.

## Verify

After generating, confirm both files exist and are non-trivial in size, and
report their paths to the user as deliverables alongside the markdown sources.
