# mdview

A static HTML viewer for Markdown files with a sidebar built from all headers and subheaders.

![Two files side by side, with an opened folder in the sidebar](docs/screenshot-split.png)

## What it does

- **Contents:** a sidebar tree of every heading (H1–H4), with nesting lines and fold/unfold per section. The section you're reading is highlighted as you scroll.
- **Folders:** open a folder and browse all its Markdown files as a tree. Relative links between them (`[setup](guides/setup.md#install)`) open as tabs.
- **History:** every file you open is kept under **Recent**, and your open tabs and panes come back when you reopen the viewer.
- **Side by side:** up to three panes, each with its own tabs. **Split** opens the current file next to itself; Option-click (or ⌘/Ctrl-click) a file to open it in another pane. Contents follows the pane you're in.
- Filter headings or files by typing (press `/` to jump to the filter)
- Light and dark themes, following the system by default
- Syntax highlighting, GFM tables and task lists
- Works on phones: the sidebar becomes a drawer

![Light theme](docs/screenshot-light.png)

| Dark theme, filtering headings | Phone |
|---|---|
| ![Dark theme with filter](docs/screenshot-dark.png) | ![Mobile drawer](docs/screenshot-mobile.png) |

## Usage

### Bake a Markdown file into a standalone page

```bash
./mdview notes.md            # writes notes.html next to it
./mdview docs/               # converts every .md under docs/
./mdview notes.md -o out.html --open
```

The output is a single `.html` file that opens straight from disk. Links between converted `.md` files are rewritten to the `.html` pages, and relative images keep working because the page sits next to the source.

To have it on your PATH:

```bash
ln -s "$PWD/mdview" /usr/local/bin/mdview
```

### Open files and folders in the viewer

Open `viewer.html` in a browser and drop Markdown files or a whole folder on it, or use **Open files** / **Open folder**. Folders skip hidden directories and `node_modules`.

History, folders and layout are stored in the browser (IndexedDB) for that page, so they're per browser and per location of `viewer.html`.

- **Chrome, Edge, Arc:** files and folders are re-read from disk whenever you open them, so you always see the current version. After a browser restart, the first click asks you to allow access again. Use ↻ next to a folder to pick up newly added files.
- **Firefox, Safari:** these browsers can't keep access to files, so the viewer stores a copy when you open something. Open the file or folder again to see changes.

When the folder is served over HTTP, `viewer.html?src=path/to/file.md` loads that file:

```bash
python3 -m http.server
# http://localhost:8000/viewer.html?src=examples/runbook.md
```

## Keyboard

| Key | Action |
|-----|--------|
| `/` | Filter headings |
| `Esc` | Leave the filter |
| `[` | Show or hide the sidebar |

## Notes

- Requires Python 3 for the `mdview` command; no packages needed.
- Rendering uses [marked](https://marked.js.org) and [highlight.js](https://highlightjs.org) from a CDN, so pages need a network connection the first time.
- Rendered HTML goes through [DOMPurify](https://github.com/cure53/DOMPurify), so scripts inside a Markdown file don't run.
- `?src=` only loads files from the same site as the viewer.
