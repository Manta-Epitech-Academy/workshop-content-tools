# workshop-content-tools

Convention **2.0** for workshop subject repos, and the parser that enforces it.

- **[CONTENT_CONVENTION.md](CONTENT_CONVENTION.md)** — the format: what `subject.yaml` holds, how
  structure is derived from markdown headings and `<!-- ws: ... -->` markers, how exercises,
  hints, quizzes, instructor summaries and images are expressed.
- **`ws_parser.py`** — the parser and the linter, in one file. **The platform importer runs this
  same parser**, so a repo that passes CI is a repo the platform can import. That is the whole
  point of publishing it rather than reimplementing the rules in a schema.
- **`generate_readme.py`** — a subject's README is derived from its content, not written by hand.

Successor to [workshop-metadata-tools](https://github.com/Manta-Epitech-Academy/workshop-metadata-tools)
(schema 1.x). The big change: **there is no `toc`.** Structure comes from the markdown itself, so
the entire class of "the toc drifted from the headings" errors disappears, and with it
`check_toc.py`.

## Use it in a subject repo

Add `.github/workflows/verify-content.yaml`:

```yaml
name: Verify content
on:
  push:
    branches: [main]
  pull_request:

jobs:
  verify:
    uses: kevin-cazal/workshop-content-tools/.github/workflows/verify-content-reusable.yml@main
```

Pin `@main` to a tag or commit for a repo backing a scheduled session, so the rules cannot move
under you between now and the workshop.

Inputs, all optional: `subject-dir` (default `.`), `tools-ref` (default `main`),
`python-version` (default `3.12`).

## Use it locally

```bash
pip install -r requirements.txt

python3 ws_parser.py path/to/subject     # lint; non-zero exit on any problem
python3 generate_readme.py path/to/subject
python3 generate_readme.py path/to/subject --check    # for CI
```

The linter reports missing quiz answers, unparseable regexes, quiz answers matching no marker,
duplicate exercise ids across the whole subject, and referenced images that do not exist.

## What a subject looks like

```
my-subject/
├── subject.yaml        # identity, runtime, platform defaults, document order
├── intro.md            # entrypoint: index page and first step
├── main.md             # content, any number of files
├── quiz_answers.yaml   # answers, kept out of the documents
├── img/…               # images, referenced relative to the document
└── README.md           # GENERATED
```

`examples/minimal/` is the smallest thing that passes, and is linted by this repository's own CI.
