# Mega Menu Builder

`mega_menu_builder`: Splits the children of a menu item into balanced columns
for a mega menu panel.

The module has no UI, configuration, hooks or templates. It provides one
service, `mega_menu_builder.builder`, that a theme calls from a menu preprocess
hook. Given a top-level menu item and a number of columns, it decides which
second-level items go in which column so the columns hold roughly the same
number of links. `mainspring_theme` uses it to build the main menu.

Supports Drupal `^10 || ^11`.

## Usage

Call the builder for each top-level item and pass the result to the template:

```php
function mytheme_preprocess_menu(&$variables) {
  $builder = \Drupal::service('mega_menu_builder.builder');
  foreach ($variables['items'] as $id => $item) {
    $variables['items'][$id]['columns_data'] = $builder->build($item, 3);
  }
}
```

`build($item, $cols, $options = [])`:

- `$item`: a menu item from the preprocess `items` array. Its `below`
  children are what gets distributed. An item with no children returns `[]`.
- `$cols`: the number of columns, from 1 to 5. Values outside that range
  are treated as 5 (0 or less) or capped at 5.
- `$options['multiplier']`: how much bigger than all the other items put
  together one item has to be before it gets a "split" column of its own
  (default `2`).

It returns:

```php
[
  'number_of_columns' => 3,
  // Column number (starting at 1) => the menu ids of the second-level items
  // in that column, in menu order.
  'columns' => [
    1 => ['menu_link_content:abc' => 'menu_link_content:abc', ...],
    2 => [...],
    3 => [...],
  ],
  // Only when one item is large enough to be split: its column number.
  'split_col' => 2,
]
```

Each item counts as one link plus one per child link. Columns are filled in
menu order, so items are never reordered; when the links can't divide evenly
the first column gets the extra ones. `columns` can have fewer entries than
`number_of_columns` when there are fewer second-level items than columns. A
split column needs at least 3 columns, and with exactly 3 the split item must
be first or last.

## Install

Add the repository and require the package in the site's root `composer.json`:

```json
"repositories": [
    { "type": "vcs", "url": "https://github.com/fosterinteractive/mega_menu_builder" }
]
```

```bash
composer require fosterinteractive/mega_menu_builder:<version>
drush en mega_menu_builder
```

It installs to `web/modules/contrib/mega_menu_builder`.

The repo is private: the site's Composer token (e.g. `COMPOSER_AUTH` on the
host) must belong to the `core` or `extended` team, or downloads fail with a 404.

## Branches

- `main`: the latest release line.
- `1.1.x`: Drupal `^10 || ^11` (1.1.0 and later).
- `1.0.x`: the last Drupal `^9.2 || ^10` code (1.0.3), for sites that
  aren't on Drupal 10 yet.

## Releasing

The version lives in two places that must match: the git tag (what Composer
installs) and `version:` in `mega_menu_builder.info.yml` (what Drupal shows).

1. Merge the changes to `main`.
2. **Actions → Release → Run workflow**, and enter the new version, e.g.
   `1.1.2` (three numbers, no `v` prefix). Optionally fill in **Summary**: a
   line or two for the top of the release notes.
3. Fast-forward the release line's branch (e.g. `1.1.x`) to `main`.

The workflow sets `version:` in the `.info.yml`, commits "Release 1.1.2" to
`main`, creates and pushes the `1.1.2` tag, and publishes a GitHub release. The
release notes are the summary plus the commit messages since the previous tag,
so write commit messages that read well there. It refuses a malformed version
or one that's already tagged.

You can edit the notes afterwards on the repo's **Releases** page; that doesn't
touch the tag or the code. Composer installs from the tag, not the release.

**Don't tag by hand,** and don't create a new tag from **Releases → Draft a new
release** (that also skips the `.info.yml` bump). If you have to, bump
`version:` in the `.info.yml` to the same number and commit it first. The
**Check tag** workflow fails any pushed tag that doesn't match the `.info.yml`.

Don't add a `version` field to `composer.json`; Composer reads the version from
the tag.
