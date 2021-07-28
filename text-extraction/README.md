# General Text Extraction
This folder contains a number of scripts for extracting and analyzing text from documents.

* `layout-analyzer.py`: produces a new PDF showing the layout of all text and images.

* `pdf2text.py`: "naive" or "native" text extraction, which simply prints the text as encountered in the document page. In many cases this may lead to text not appearing in the intended reading order or even to illegible text.

* `pdf2textblocks.py`: extracts the text portioned in so-called "blocks", as collected by the underlying MuPDF library. These text blocks are sorted by their accompanying coordinates to establish a "Western" reading order: top-left to bottom-right. In many cases, this output should produce satisfactory results in reading order, while maintaining a high extraction speed. There are however cases, where this expectation cannot be met. For example, multi-column text or text in tables will not show up satisfactorily.

* `fitzcli.py`: is a duplicate of the PyMuPDF batch / CLI module. So it offers all functions and commands described [here](https://pymupdf.readthedocs.io/en/latest/module.html) **_plus_** the new command `gettext`, which offers text extraction from arbitrary MuPDF documents. Most importantly, you can now etract text in a **_layout-preserving_** manner. The next section describes this in detail.

# Layout-preserving Text Extraction

Via its subcommand `gettext`, script `fitzcli.py` offers text extraction in different formats. Of special interest is probably **_layout preservation_**, which produces text as close to the original physical layout as possible, thus surrounding areas where there are images, or reproducing text in tables and multi-column text. Numerous document files (especially PDFs) contain "irregular" text like
* simulation of bold / shadowed text by double "printing" it with a small horizontal / vertical shift or inclination
* arbitrary arrangements of each single character to prevent or impede copy-pasting text from within a PDF viewer window (what you see is **_not_** what you get in such cases)
* unintended specification errors like writing spaces over preceeding non-space characters
* etc.

Many of these pesky situations are being corrected.

In this folder you find examples for PDFs and corresponding text output images for your review.

## Invocation

In its simplest form, the following the following extracts text from all pages of `filename.ext` and generates file `filename.txt` in "UTF-8" encoding, where all pages contain text in the original physical layout.

`python fitzcli.py gettext filename.ext`

> Version PyMuPDF 1.18.16 will support exactly the same via `python -m fitz gettext filename.ext`.

```
python fitzcli.py gettext -h
usage: fitzcli.py gettext [-h] [-password PASSWORD] [-mode {simple,blocks,layout}] [-pages PAGES] [-noligatures]
                          [-whitespace] [-extra-spaces] [-noformfeed] [-skip-empty] [-output OUTPUT] [-grid GRID]
                          input

----------------- extract text in various formatting modes ----------------

positional arguments:
  input                 input document filename

optional arguments:
  -h, --help            show this help message and exit
  -password PASSWORD    password for input document
  -mode {simple,blocks,layout}
                        mode: simple, block sort, or layout (default)
  -pages PAGES          select pages, format: 1,5-7,50-N
  -noligatures          expand ligature characters (default False)
  -whitespace           keep whitespace characters (default False)
  -extra-spaces         fill gaps with spaces (default False)
  -noformfeed           write linefeeds, no formfeeds (default False)
  -skip-empty           suppress pages with no text (default False)
  -output OUTPUT        store text in this file (default inputfilename.txt)
  -grid GRID            merge lines if closer than this (default 2)
```

The output filename defaults to the input with its extension replaced by ``.txt``.
As with other commands, you can select page ranges in ``mutool`` format as indicated above.

* **mode:** select a formatting mode -- default is "layout". Output of "simple" is the same as for script `pdf2text.py`, and "blocks" produces the output of `pdf2textblocks.py. So this script is an extended replacement for all of them.
* **noligatures:** corresponds to **not** `TEXT_PRESERVE_LIGATURES`. If specified, ligatures (present in advanced fonts: glyphs combining multiple characters like "fi") are split up into their components (i.e. "f", "i"). Default is passing them through.
* **whitespace:** corresponds to `TEXT_PRESERVE_WHITESPACE`. If specified, all white space characters (like tabs) are replaced with one or more spaces. Default is passing them through.
* **extra-spaces:**  corresponds to **not** `TEXT_INHIBIT_SPACES`. If specified, large gaps between adjacent characters will be filled with one or more spaces. Default is off.
* **noformfeed:**  instead of ``hex(12)`` (formfeed), write linebreaks ``\n`` at end of output pages.
* **skip-empty:**  skip pages with no text.
* **grid:** lines with a vertical coordinate difference of no more than this value (float, in points) will be merged into the same output line. In addition, lines with a ``bbox.height < grid`` **will be ignored**. Only relevant for "layout" mode. **Use with care:** the default 2 should be adequate in most cases. If **too large**, lines intended to be different will result in garbled and / or incomplete merged output -- plus lines may be suppressed unintendedly. If **too low**, separate, artifact output lines may be generated for text spans just because they are coded in a different font with slightly deviating properties.

> Command options may be abbreviated as long as no ambiguities are introduced. So instead of:
```
... -output text.txt -noligatures -noformfeed -whitespace -grid 3 -extra-spaces ...
```
> you can also write: ``... -o text.txt -nol -nof -w -g 3 -e ...``.
