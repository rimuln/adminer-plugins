# Adminer plugins

Plugins for [Adminer](https://www.adminer.org/) 5+ focused on foreign keys.

| Plugin | Description |
|---|---|
| [`foreign-tooltip.php`](foreign-tooltip.php) | Displays the referenced row in a tooltip when hovering a foreign key value in select, similar to phpPgAdmin. |
| [`edit-foreign-search.php`](edit-foreign-search.php) | Selects a foreign key in the edit form from a searchable, paged list showing the data of the referenced rows. |

## Installation

Upload the plugin to the `adminer-plugins/` directory next to `adminer.php`. Plugins without required parameters are loaded automatically. To configure them or to specify their order, use `adminer-plugins.php`:

```php
<?php // adminer-plugins.php
return array(
	new AdminerForeignTooltip(20, 80), // max columns, max length of a value
	new AdminerEditForeignSearch(20, 3), // rows per page, text columns in the description
);
```

If `adminer-plugins.php` returns an array, list every plugin you want to use in it.

## foreign-tooltip

Adds a `title` to foreign key links in select with the referenced row:

```
klub
id: 1
nazev: SK Praha
mesto: Praha
```

- The referenced rows are fetched by one query per foreign key for the whole page.
- Works together with the official `select-foreign` plugin.
- Blob columns are skipped.

## edit-foreign-search

Replaces the foreign key input in the edit form by:

- the original input with the key value (typing a key looks up its description),
- a search field with the description of the referenced row (first text columns),
- a dropdown list `key  description`, paged, filtered by the key and the description, controlled by mouse or keyboard (arrows, Enter, Esc, PageUp, PageDown).

Choosing the NULL function clears the value, choosing a row resets the NULL function.

The list is loaded by `fetch` from the form URL, answered in `afterConnect()` (after login) with a CSRF token check. The script uses Adminer's CSP nonce.

## Limitations

- Only single-column foreign keys in the current database (other schemas are supported).
- Tested with PostgreSQL. Search uses `ILIKE` in PostgreSQL and `LIKE` elsewhere; MySQL, SQLite, MS SQL and Oracle are untested.

## License

[Apache License 2.0](LICENSE-APACHE) or [GPL 2](LICENSE-GPL2), same as Adminer.
