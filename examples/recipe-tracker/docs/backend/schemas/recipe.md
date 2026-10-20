----
name: recipe schema
type: schema
----

# Recipe

The single data model for the app. One row per recipe in SQLite.

## Fields

- `id: str` — uuid4 generated at create time.
- `title: str` — required, 1-200 chars.
- `ingredients: list[str]` — one string per ingredient line, free-form.
- `steps: list[str]` — one string per step, free-form.
- `tags: list[str]` — free-form strings, lowercase. Empty list if none.
- `created_at: str` — ISO-8601 timestamp, set at create, immutable.
- `updated_at: str` — ISO-8601 timestamp, set on every edit.

Pydantic class name: `Recipe` (matches this md title).

## Pairs with

- `docs/backend/apis/recipes.md` — request and response shapes.
- `src/api/schemas/recipe.py` — the pydantic class itself.

## V2

- `image_urls: list[str]` — signed storage URLs, set after upload.
- `cook_count: int` — incremented when user hits a "cooked this" button on the detail page.
