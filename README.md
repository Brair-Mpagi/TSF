<div align="center">

# TutorialStudyForge (TSF)

**Turn an online programming tutorial into a clean, offline study guide: one PDF and one editable Word document.**

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Output](https://img.shields.io/badge/output-PDF%20%7C%20DOCX%20%7C%20Markdown-informational)
![Single file](https://img.shields.io/badge/script-tsf.py-success)
![Tests](https://img.shields.io/badge/tests-offline%20pytest-brightgreen)
![License](https://img.shields.io/badge/license-not%20chosen-lightgrey)

</div>

TSF reads a tutorial's table of contents (for example the *SQL Tutorial* on GeeksforGeeks, the *SQL* course on
W3Schools, or the *Java Tutorial* on JavaTpoint), downloads every topic politely, and compiles them into one
document with a cover, a contents page, parts, chapters, exact code blocks, tables and images.

It is a **personal study tool**. Read [Responsible use](#responsible-use) before pointing it at a site.

---

## Contents

- [Features](#features)
- [Install](#install)
- [Quick start](#quick-start)
- [Usage](#usage)
- [Output](#output)
- [Supported sites](#supported-sites)
- [Fixing or adding a site](#fixing-or-adding-a-site)
- [How it works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Responsible use](#responsible-use)
- [Development](#development)
- [License](#license)

## Features

- **PDF** with cover (title and subtitle), contents, bookmarks, page numbers, selectable text.
- **Editable Word document (`.docx`)** with real Word styles (Heading 1-3, Code, lists, tables), working
  numbered lists, a Contents field and header/footer, so you can restyle and edit it freely.
- **Parts like the source site**: a main title, a subtitle, then sections ("Parts") each with a short description
  and their topics. Section names come from the page's own headings.
- **Guided mode**: run it with no arguments and it asks for the address, shows the sections, lets you pick
  (`1-5,8` or `not 12,13`), and builds.
- **Works across tutorial sites**: built-in rules for several well-known sites plus generic detectors for the rest
  (article body, side menu, heading-based sections).
- **Exact code**: spaces, tabs, line breaks and comments are preserved; line-number gutters are removed.
- **Honest results**: every run ends as *Completed*, *Completed with warnings* or *Partial*. Failed or duplicate
  pages are listed in a report, never hidden. Partial builds are named `-PARTIAL`.
- **Polite fetching**: obeys `robots.txt`, one request per second per site, caches pages, never retries after
  403/429, and blocks private-network addresses.
- Optional **Markdown** and **HTML** exports.

## Install

Requires Python 3.10 or newer.

```bash
git clone <your-repo-url> TutorialStudyForge
cd TutorialStudyForge
python3 -m venv .venv && source .venv/bin/activate      # optional but recommended
pip install -r requirements.txt
python3 tsf.py selftest                                  # builds a sample PDF + Word file
```

PDF output uses WeasyPrint, which needs the Pango library. On Debian/Ubuntu:
`sudo apt install libpango-1.0-0 libpangoft2-1.0-0`. On macOS: `brew install pango`. If you only want Word,
use `--docx --no-pdf`.

## Quick start

```bash
python3 tsf.py                                          # guided mode, asks everything
python3 tsf.py https://www.geeksforgeeks.org/linux-tutorial/    # guided mode for one tutorial
```

Guided mode asks for: the address, which sections to include, title and subtitle, whether to also save Word and
Markdown, then writes an editable `topics.txt` you can reorder or trim before it builds.

Non-interactive:

```bash
# everything, as PDF + Word + Markdown, no questions
python3 tsf.py go https://www.w3schools.com/sql/default.asp --yes --docx --markdown

# Word only
python3 tsf.py build --index https://www.javatpoint.com/java-tutorial --docx --no-pdf
```

## Usage

| Command | What it does |
|---|---|
| `python3 tsf.py` / `go [URL]` | Guided mode (default). |
| `python3 tsf.py discover URL -o topics.txt` | Write the topic list to an editable file. |
| `python3 tsf.py build --chapters topics.txt` | Build from that file (after you edit it). |
| `python3 tsf.py build --index URL` | Discover and build in one go. |
| `python3 tsf.py build --url URL [--url URL ...]` | Build from specific article addresses. |
| `python3 tsf.py build --html page.html ...` | Build from pages you saved yourself. |
| `python3 tsf.py inspect URL` | Show how one page is read (for debugging). |
| `python3 tsf.py sites` | List sites with built-in rules. |
| `python3 tsf.py selftest` | Check your installation. |

Common options (`python3 tsf.py build --help` lists all):

| Option | Meaning |
|---|---|
| `--docx` / `--no-pdf` | Also write Word / skip the PDF (needs `--docx`). |
| `--markdown`, `--html-export` | Extra output formats. |
| `--sections "1-5,8"` or `"not 12,13"` | Choose sections by number. |
| `--skip-sections REGEX` | Drop sections by name. |
| `--select 1-5,8` | Choose topics by number. |
| `--include REGEX`, `--exclude REGEX` | Filter topics by title or address, e.g. `--exclude 'quiz|interview'`. |
| `--title`, `--subtitle`, `--author` | Cover text. |
| `--toc-depth {1,2}`, `--no-chapter-breaks` | Contents depth; keep chapters flowing without page breaks. |
| `--page-size {A4,Letter}`, `--landscape`, `--font-size`, `--margin-mm`, `--code-cols` | Layout. |
| `--out-dir DIR` | Output folder. |
| `--allow-partial` | Produce a document even if some chapters failed (named `-PARTIAL`). |
| `--ack-terms HOST` | Confirm you reviewed that site's terms (asked once if omitted). |
| `--refresh`, `--offline` | Ignore the cache / use only the cache. |
| `--selector`, `--link-selector`, `--section-selector`, `--profiles FILE` | Fix extraction, see below. |
| `-y`, `--yes` | No questions. |

## Output

```
<out-dir>/<tutorial-name>/
├── <tutorial-name>.pdf          # the study guide
├── <tutorial-name>.docx         # editable Word version (with --docx)
├── <tutorial-name>.md           # with --markdown
├── completeness.md / .json      # what was included, skipped, duplicated or failed
├── chapters.json                # the topic list used
├── manifest.json                # files, hashes, settings, versions
└── build.log
```

**Word note:** the Contents page is a real Word field. When you open the file, Word asks whether to update fields;
choose **Yes** to refresh page numbers. After you edit headings, right-click the contents and choose
*Update Field*. LibreOffice shows the pre-filled entries without asking.

## Supported sites

| Site | Example start address | Notes |
|---|---|---|
| GeeksforGeeks | `https://www.geeksforgeeks.org/sql-tutorial/` | Sections and descriptions follow the page. |
| W3Schools | `https://www.w3schools.com/sql/default.asp` | Left-menu topics; `div.w3-code` blocks. |
| TutorialsPoint | `https://www.tutorialspoint.com/sql/index.htm` | Chapter list. |
| JavaTpoint | `https://www.javatpoint.com/java-tutorial` | Left-menu topics. |
| Programiz | `https://www.programiz.com/python-programming` | |
| MDN Web Docs | `https://developer.mozilla.org/en-US/docs/Learn_web_development` | Openly licensed content. |
| Python documentation | `https://docs.python.org/3/tutorial/index.html` | Openly licensed. |
| Read the Docs / Sphinx | `https://<project>.readthedocs.io/en/latest/` | Any Sphinx site. |
| Real Python, Baeldung, Oracle Java Tutorials | see `python3 tsf.py sites` | |
| Anything else | | Generic detectors: article body, side menu, heading sections. |

> **Important:** the built-in rules are written from each site's usual page layout and were **tested against
> local mock pages, not against the live sites**. Sites change their HTML. Always run `discover` first and check
> the list. If something is off, use the fixes below.

## Fixing or adding a site

1. See what TSF finds: `python3 tsf.py discover URL -o topics.txt`, and `python3 tsf.py inspect <article-URL>`.
2. Fix it with a flag:
   - wrong article body: `--selector "div.content"`
   - topics not found: `--link-selector "#menu"`
   - sections not detected: `--section-selector "h2.part"`
3. Make it permanent with a profiles file (merged over the built-ins, keyed by domain):

```json
{
  "example.com": {
    "name": "Example Tutorials",
    "content": ["div.tutorial-body"],
    "toc": ["nav.course-menu"],
    "remove": [".ads", ".next-prev"],
    "code_selectors": ["div.code-sample"]
  }
}
```

```bash
python3 tsf.py go https://example.com/course/ --profiles profiles.json
```

Pull requests with verified profiles are welcome.

## How it works

1. **Discover**: find the topic list (site rules, then content links, then side menu, then whole-page fallback)
   and group topics into sections from the headings above them.
2. **Fetch**: polite, cached, safety-checked downloads.
3. **Extract**: each page becomes typed blocks (headings, paragraphs, code, lists, tables, images, quotes,
   equations). Navigation, ads and comments are removed; confidence is scored.
4. **Build**: the same blocks feed the PDF (HTML to PDF), Word and Markdown writers.
5. **Verify**: the PDF and Word file are reopened and checked (page count, chapter titles, code text, bookmarks).
   The result is reported honestly.

## Troubleshooting

| Problem | Fix |
|---|---|
| `Only http and https URLs are allowed` | Pass a real address starting with `https://`. Do not type placeholders like `LINUX_URL` literally. |
| `can't open file ... tsf` | Include `.py`: `python3 tsf.py ...`, or `chmod +x tsf.py` and run `./tsf.py`. |
| Cannot reach the site / DNS error | Check your connection or proxy, then retry (cached pages are reused). |
| 403 / 429 from a site | TSF stops asking that host. Save pages in your browser and use `--html`. |
| Too few topics found | Try `--link-selector`, or paste a different start page of the tutorial. |
| Wrong text included | Use `--selector` or `--drop-text REGEX`; check with `inspect`. |
| Result named `-PARTIAL` | Some chapters failed; see `completeness.md`, then re-run (cache makes it quick). |
| Images missing (`[Image not available - access_denied ...]`) | The run prints why. TSF sends the article as `Referer` (needed by many image servers), reads lazy-loaded, `srcset`, `<picture>`, inline and SVG images, and an image refusal never blocks the page host. If a site still refuses (403/robots), TSF respects it: use `--html` with pages saved from your browser (**Save as > Webpage, complete**). Raise the cap with `--max-images N` (default 3000). |
| SVG diagrams missing in Word | SVG shows in the PDF. For Word run `pip install cairosvg` (or install `rsvg-convert`); without it Word shows a note. |
| PDF fails on import of WeasyPrint | Install Pango (see [Install](#install)) or use `--docx --no-pdf`. |

## Responsible use

- Tutorial content belongs to its authors. Several sites (including W3Schools, TutorialsPoint, JavaTpoint and
  GeeksforGeeks) restrict automated copying or republishing in their terms. TSF asks you to confirm you reviewed
  the terms for each site and is meant for **personal offline study only**.
- **Do not redistribute** generated documents. The `.gitignore` excludes `*.pdf` and `*.docx` for that reason.
- Prefer openly licensed sources (MDN, Python docs, many Read the Docs projects) when you want to share.
- TSF honours `robots.txt`, rate-limits itself and never works around blocks or logins.

## Development

```bash
pip install -r requirements-dev.txt
python3 -m pytest -q tests        # offline: uses a local fixture website, no internet needed
python3 tsf.py selftest
```

The whole tool is the single file `tsf.py`. The tests cover URL safety, extraction exactness, section detection,
guided mode, PDF and Word output.

## License

No license has been chosen yet. Add a `LICENSE` file (MIT is a common choice for tools like this) before
publishing. Generated study guides remain subject to the licenses of their source sites.
