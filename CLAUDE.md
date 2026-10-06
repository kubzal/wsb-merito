# CLAUDE.md

Course-materials website for classes taught by Jakub Zalewski at Uniwersytet WSB Merito (Warszawa). Static MkDocs site (Material theme), published at https://kubzal.github.io/wsb-merito. All content is in **Polish**; there is no application code.

## Commands

```sh
source .venv/bin/activate        # local venv (gitignored); pip install -r requirements.txt if missing
make preview                     # = mkdocs serve, live preview at http://127.0.0.1:8000
mkdocs build --strict            # catch broken links / pages missing from nav before pushing
```

Deployment is automatic: every push to `main` runs [.github/workflows/ci.yml](.github/workflows/ci.yml) → `mkdocs gh-deploy --force` (gh-pages branch). Pushing to `main` = publishing to students. Note CI installs unpinned `mkdocs-material`, while [requirements.txt](requirements.txt) pins 9.7.7 locally.

## Layout

- [mkdocs.yml](mkdocs.yml) — config **and the explicit `nav`**. A new page does not appear on the site until it is added to `nav`.
- [docs/index.md](docs/index.md) — home page ("Hello world!"); lists current courses and the archive. Update it when courses change.
- `docs/semestr_zimowy_2026_27/<przedmiot>/` — current semester. Each course has `index.md` (title "Wprowadzenie": schedule, grading rules) and `zajecia_N.md` per class.
- `docs/archiwum/<przedmiot>/` — previous semester (letni 2025/26). Sometimes also `projekt.md` / `projekt_zaliczeniowy.md`.
- Course assets live next to the pages: `dane/` (CSV/XLSX datasets, linked relatively and also fetched in notebooks via the full `https://kubzal.github.io/wsb-merito/...` URL), `obrazki/` (images).
- [overrides/partials/tabs-item.html](overrides/partials/tabs-item.html) + [docs/stylesheets/extra.css](docs/stylesheets/extra.css) — customisation: top-level tabs with children (e.g. "Archiwum") get a hover dropdown listing courses; on wide screens the left sidebar shows only the active course.
- `drafts/` — **gitignored, private**. Semester plans, notes, and reference solutions (`drafts/zadania/<przedmiot>/zajecia_N_rozwiazanie.ipynb`, marked "nie udostępniać studentom"). Never move anything from here into `docs/` or commit it unless explicitly asked.

Directory names are ASCII slugs of the Polish course name (`uczenie_glebokie`, `zarzadzanie_big_data`); nav titles keep Polish diacritics.

## Content conventions

Follow the existing pages closely — new material should look like it was written by the same person.

- Address students informally with "Ty" in lessons (`Po dzisiejszych zajęciach będziesz w stanie:`), "Państwo" only on the home page. Plain, practical, slightly humorous tone.
- Course `index.md`: `# Wprowadzenie`, an `!!! info "Semestr zimowy 2026/27"` box, `## Plan zajęć` as a bullet list in the form
  `- **Zajęcia N (_YYYY-MM-DD_ godz. HH:MM - HH:MM - Xh):** topics`, then `## Zasady zaliczenia` (100 pts, grading table), `## Czego potrzebujesz`.
- Lesson `zajecia_N.md`: `# Zajęcia N`, `## <topic>` with a short intro and learning outcomes, `## Plan na dziś` table (block | time), sections separated by `---`, a `## Ściągawka`, then `## Zadanie N — <title> (6 pkt)` with **Środowisko:** / **Oddanie:** lines, two parts (`### Część 1 — … (ok. 20 min)`), `### Wskazówki`, `### Punktacja (6 pkt)` table.
- Tasks are personalised by student album number, e.g. `NR_ALBUMU = 123456` → `MIESIAC = NR_ALBUMU % 12 + 1`, so solutions can't be copied.
- Typical tools referenced: Google Colab, pandas/numpy, scikit-learn, DuckDB, Databricks Free Edition, BigQuery Sandbox; submissions go through Moodle.
- Markdown extensions available: admonitions (`!!! info|note|warning|danger|tip "Title"`, body indented 4 spaces), `pymdownx.superfences`/`highlight` code blocks, `attr_list`, Material emoji/icons (`:material-email:`, `:material-file-delimited:`), `pymdownx.snippets`.
- Hard-wrap prose at ~85 characters, as existing files do. Use the em dash `—` and Polish quotes `„…"`.

## Git

Commit messages are short, lowercase Polish: `<przedmiot> - zajecia N` (e.g. `systemy big data - zajecia 1`, `python w ds - zajecia 5 (opis)`). Work happens directly on `main`; since that deploys immediately, confirm with the user before pushing.
