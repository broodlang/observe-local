# observe-local — guidance for Claude

An `observe` `Provider` that keeps batches in one gen server (`:observe-local`) and a
hatch page over a snapshot. Read `src/observe-local.blsp`'s header for the design.

- **The folds and the queries are pure over the state map** and every one takes an
  optional snapshot. Test them with `(empty-state)`, never through the process; only the
  one `:isolated` group starts it.
- **Everything is bounded** (`*max-…*`). A new cap follows the existing policy: oldest
  goes, and the drop is counted or visible on the page. Nothing here may grow with traffic.
- The page is `web/template` Hiccup, self-contained (inline CSS, no assets, no JS), and
  everything a user could have typed goes through the renderer's escaping — the "everything
  is escaped" test guards it.
- `observe` and hatch are pinned by commit until they are on the registry.
- Run the one test file (the full suite is blocked on this machine) and `nest format`
  before committing. No trailing `!`, no abbreviations.
