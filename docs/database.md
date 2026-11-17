# The database: what bites

Official: <https://core.telegram.org/bots/serverless.md#the-database>. Drizzle-like DSL from `sdk/db`, SQLite underneath, one database per bot.

## Query builder gotchas (verified with CLI 0.2.0)

- **Every call is a Promise**: `.all()`, `.get()`, `.values()`, `.run()`, `db.$count()`, `db.run/all/get`. Forgetting `await` gives you a Promise object, not rows.
- **`.returning().run()` returns an array** of rows: `const [row] = await db.insert(t).values(v).returning().run();`
- **`desc()` / `asc()` with a `sql` fragment produce invalid SQL** (`no such column: "undefined"`). Write the direction inside the fragment instead:
  ```js
  .orderBy(sql`COALESCE(${t.answeredAt}, ${t.created}) DESC`)   // ✅
  .orderBy(desc(sql`COALESCE(...)`))                              // ❌
  ```
- Raw `sql` queries are not bound to a table: booleans come back as `0/1`, JSON as text, timestamps as numbers.
- `db.all(sql\`…\`)` keys rows by column name — alias your columns (`AS n`) and beware of duplicate names in joins (use `db.values`).
- `onConflictDoNothing().returning().run()` returns `[]` when the row already existed — a handy atomic "claim" (e.g. "am I the first update of this album?").
- An `update(...).where(...).returning().run()` returns the changed rows: use its length to know whether a conditional update (e.g. `status = 'open'`) matched — useful to avoid double processing when two people press the same button.
- Batch inserts are a single statement: chunk big lists (SQLite variable limit).
- `json_extract()` works in raw SQL against `json()` columns (they are stored as TEXT).

## Schema and migrations

- `push` uploads `schema.js` but **never** changes the database. Run `migrate` afterwards (or `migrate --local` before the first push).
- Adding tables/columns/indexes = **safe**. Applied with `migrate --safe`.
- Removing something: mark it **`.deprecated('reason')`** (table, column or index), push, then `migrate --yes` (each drop is a "warning"). Deleting the declaration alone drops nothing — the table shows up as `?  not in schema`. After the drop succeeds, remove the declaration.
- Changing a column type is **manual**: create a new column/table, copy, swap — with `db.run()`.
- Foreign keys: `.references()` / `foreignKey()` **throw at declaration**; the schema won't deploy. Use plain integer columns and keep integrity in code (delete children before parents).
- A multi-column unique index: `uniqueIndex('uq_x').on(t.a, t.b)` works.

## Patterns

- Per-user data in a multi-user bot: add a `tenant_id` (the Telegram user ID) to every table and index it; always filter by it.
- Key/value settings: a `settings` table with `key` primary key and a `json()` value; upsert with `onConflictDoUpdate`.
- Per-user preferences: a `json()` column `prefs` on the users table, merged with defaults in code.
