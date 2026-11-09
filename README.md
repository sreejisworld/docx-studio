# Docx Studio

A Word-like editor for `.docx` files that runs entirely in the browser.
No install, no build step, no server, and nothing leaves your machine.

**[Open the editor](https://sreejisworld.github.io/docx-studio/)**

Or [download `index.html`][dl] and double-click it. One file, works
offline, nothing else needed.

[dl]: https://raw.githubusercontent.com/sreejisworld/docx-studio/main/index.html

## The two save modes

The mode you are in decides what the saved file looks like. It is shown
in the top bar and again in the status bar.

**Faithful** patches your edits into the original package. Every
paragraph and run is rebuilt from the properties it was opened with, and
only `word/document.xml` is rewritten. Styles, numbering, headers,
footers, theme, settings and images come back byte for byte. Use this on
documents that must come back looking exactly as they went in.

**Free** rebuilds the whole document from what is on screen, with its own
styles and numbering. You get the full ribbon, including arbitrary fonts,
colours, tables and pictures. Formatting the reader does not model may
normalise on save, so a named style it does not know (`Intense Quote`,
say) comes back as direct formatting rather than that style.

Switching from Free back to Faithful reloads the original file, because
Faithful has to start from the package as it was opened. The app asks
first.

## What it reads

Paragraph and character styles with the full `basedOn` chain, direct run
formatting (font, size, bold, italic, underline styles, strikethrough,
colour, highlight, shading, caps, super- and subscript), paragraph
alignment, indents, spacing, borders and shading, numbered and bulleted
lists including numbering defined on a style, tables with column widths,
merges, borders and cell shading, inline and VML images, hyperlinks,
footnote and comment markers, content controls, tracked changes, page
size and margins, and the first header and footer.

Fields such as a table of contents or a page number are kept intact and
shown as their cached text. They are not editable in place, which is why
they appear shaded. Delete one as a whole or leave it for Word to
refresh.

## Fonts

A document names fonts; it does not carry them. If the machine does not
have one, the browser substitutes silently and the page looks nothing
like Word with no indication why. So the app measures which of the fonts
a document asks for are actually present, gives each a considered
fallback chain rather than a bare `sans-serif`, and tells you what it
substituted: a notice on open, and a `Not on this machine` row in the
File panel.

This matters most for **Aptos**, the Microsoft 365 default since 2024.
It ships with Office rather than with Windows, so a browser usually
cannot use it and every document written in current Word is affected.
Install Aptos system-wide if you want exact fidelity.

Substitution is display only. Both save modes still write the original
font name, so a document that says Aptos still says Aptos after a round
trip and renders correctly when it gets back to Word. Fonts the machine
lacks stay selectable in the ribbon, marked `not installed`, for the
same reason.

## Tracked changes

A document under revision carries its history in the file. Faithful mode
keeps it: insertions stay wrapped in `w:ins` with their original author,
date and id, and deletions stay as `w:del` with the deleted text. The
editor shows them the way Word's markup view does, insertions marked and
deletions struck through, and both are read-only so an edit cannot half
accept one. Accept or reject them in Word.

Free mode rebuilds the document from what is on screen, so it writes the
text as it stands and the revision marks do not survive. Use Faithful
mode on anything still under review.

## Limits worth knowing

- Layout is a single continuous column at the real page width and
  margins, with dashed rules marking where pages would break. The page
  count in the status bar is an estimate from content height, not Word's
  own pagination.
- Headers and footers are shown for reference and are not editable here.
- Insert Table and Insert Picture are Free mode only. In Faithful mode
  they would need new parts and relationships that the original package
  does not have.
- List buttons in Faithful mode reuse a numbering definition the file
  already contains. A file with no `numbering.xml` cannot gain lists
  without switching to Free mode.
- Where a numbering part defines only its first level, deeper levels get
  a synthesised marker (1. / a. / i.) rather than none.
- Fields flatten to their cached text in Free mode. Faithful keeps them.
- `.doc` (Word 97-2003) is a different, binary format. Open it in Word
  and save as `.docx` first.
- Pasting inserts plain text, so nothing unexpected arrives with it.

## How it works

No libraries. Zip reading and writing use the browser's native
`DecompressionStream` and `CompressionStream`, so the app has no network
dependency at all; the webfont link is the only optional one and it falls
back to system fonts offline. OOXML is parsed with `DOMParser`, rendered
into a `contenteditable` sheet, and written back either by mutating the
original XML tree (Faithful) or by generating a new package (Free).

Needs a current Chrome, Edge, Firefox or Safari.

## Verified

Round-tripped against eleven fixtures covering headings, mixed run
formatting, tabs, indents, bulleted and numbered lists, tables, headers
and footers, inline PNG and JPEG images, hyperlinks, merged cells
(`gridSpan` and `vMerge`), simple and complex fields, tracked insertions
and deletions, content controls, CJK, Devanagari, Arabic and XML escape
sequences, multilevel numbering, a package carrying no styles, numbering,
theme or settings, and an archive whose entries are all stored rather
than deflated. 104 checks, none failing.

- **Faithful**: every package part survives and only `word/document.xml`
  changes, for every fixture. All 18 namespace declarations and
  `mc:Ignorable` are preserved on the document root. Image bytes,
  hyperlink relationships, merge structure, field markup, content
  controls and revision marks all come back intact.
- **Free**: produces a valid package, re-embeds images with resolving
  relationships and correct content types, writes hyperlink
  relationships, and rebuilds lists as real `w:numPr` numbering.

## Licence

MIT. See [LICENSE](LICENSE).
