# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Standalone plugins for [Adminer](https://www.adminer.org/) 6.0+ (foreign keys) plus a fixed copy of the Nette design. There is no build, package manager or test suite: each `*.php` file at the root is one self-contained plugin that users drop into `adminer-plugins/` next to `adminer.php`. `README.md` is the user documentation — keep it in sync when behavior, constructor parameters or limitations change.

## Checking changes

- Syntax check: `php -l foreign-tooltip.php` (PHP is installed locally).
- There are no automated tests. Verify changes manually in a running Adminer 6 against a real database (so far tested only on PostgreSQL; MySQL, SQLite, MS SQL and Oracle are untested — say so rather than claiming support).

## Plugin conventions (follow the existing files)

- Global class `Adminer<Name>` extending `Adminer\Plugin`; every Adminer function/constant is called fully qualified (`Adminer\q()`, `Adminer\idf_escape()`, `Adminer\DB`, `Adminer\JUSH`, …) because the file itself is not namespaced.
- Requires Adminer 6 APIs (`afterConnect()`, `is_blob()`); don't use anything that would break on 6.0.
- Header docblock: one-line description, `@link https://github.com/rimuln/adminer-plugins`, `@author Lumír Návrat`, dual Apache-2.0/GPL-2 `@license` lines.
- `protected $translations` at the end of the class: `''` key is the plugin description; `cs` is written by the author, the other languages are marked `// Claude Opus 5.5`. Use `$this->lang()` for UI strings.
- Tab indentation, LF line endings (`.gitattributes`), `array()` syntax, Adminer's coding style.
- Only single-column foreign keys in the current database are handled; other schemas are reached by temporarily calling `Adminer\set_schema()` (guarded by `function_exists`) and restoring it — `fields()`/`table()` work in the current schema.
- Queries that are optional (tooltips, descriptions) pass `""` as the error argument of `get_rows()` so failures are silent.

## Architecture notes

- **foreign-tooltip.php**: `rowDescriptions()` collects all FK values on the page and fetches referenced rows with one `IN (...)` query per foreign key, caching tooltip text; `selectVal()` re-enters `Adminer\adminer()->selectVal()` (guarded by `$inside`) so other plugins (e.g. official `select-foreign`) format the value first, then injects `title=` into the resulting `<a>`. The FK value is read from the link's `where[0][val]` because the displayed text may already be replaced by a description.
- **edit-foreign-search.php**: `editInput()` renders the combo (key input + search input + hidden list) and prints the CSS/JS once (`assets()`, JS via `Adminer\script()` for the CSP nonce). The JS `fetch`es POST requests with `fk_search` to the form URL; `afterConnect()` answers them as JSON (after login, with `Adminer\verify_token()` CSRF check) and `exit`s. Search uses `ILIKE` on pgsql, `LIKE` elsewhere; paging fetches `pageSize + 1` rows to compute `more`.
- **designs/nette/adminer.css**: upstream Adminer 6.1.1 file with the Adminer 6 fix kept as a separate commit on top of the unchanged copy — keep upstream content and local fixes in separate commits so the diff stays reviewable.

## Repo notes

- Deployment to the author's hosting is manual (they upload the files themselves).
