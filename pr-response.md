# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end -->

## Comment 1 — Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py`, following the project's verb_to_noun naming convention used by `add_to_collection()`. Updated the docstring and the one call site in `routes/watchlist/watchlist.py` (both the import and the function call).
**How I verified:** Used VS Code's project-wide search (Ctrl+Shift+F) for `save_to_watchlist` to confirm zero remaining references. Ran `pytest tests/ -v` — all tests passed.

## Comment 2 — Deduplication
**What I did:** Added a check in `add_to_watchlist()` that queries for an existing `WatchlistEntry` with the same `user_id` and `film_id` before creating a new one. If found, raises a new `AlreadyInWatchlistError` exception, following the same pattern as `AlreadyInCollectionError` in `add_to_collection()`.
**How I verified:** Ran the full test suite (`pytest tests/ -v`) to confirm no existing tests broke. Compared the logic shape directly against `add_to_collection()`'s dedup pattern.

## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` modeled directly on `test_collection.py`. Added `test_add_to_watchlist_nonexistent_film_raises` (the specifically requested test), plus two supporting tests: `test_add_to_watchlist_creates_entry` and `test_add_to_watchlist_duplicate_raises` (covering the dedup logic from Comment 2). Reused the same fixture pattern (`app`, `sample_user`, `sample_film`) as the collection tests.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — all 3 pass. Ran full suite `pytest tests/ -v` — all 7 pass with no regressions.

## Comment 4 — Default visibility
## Comment 4 — Default visibility
**My position:** I'm keeping `public=True` as the default for `WatchlistEntry`.

**Reasoning:** CineLog is a social, film-tracking app — the value of watchlists is largely social: friends can see what you want to watch, which makes it easy to coordinate watch parties, get recommendations, and start conversations. If watchlists defaulted to private, this discovery behavior would only happen for the small number of users who actively dig into settings to make their list public — most users never do this. Defaulting to public optimizes for the core social/discovery use case that CineLog is built around.

**Tradeoff acknowledged:** The cost of defaulting to public is that some users may not want their taste in films visible to others by default — they might be watching something personal, unusual, or just prefer privacy as a baseline. A public default means they're exposed unless they actively find and change the setting, rather than choosing to share when they're ready.

## Comment 5 — Sort order
**My position:** I'm keeping the watchlist sorted alphabetically by film title rather than switching to date-added order.

**Reasoning:** The watchlist endpoint currently has no search or filtering capability — `GET /watchlist/<user_id>` returns the entire list with no query parameters. Without search, alphabetical order is effectively the only way a user can scan a list and quickly locate a specific film they're looking for. If the list defaulted to date-added order instead, finding a specific title in a longer watchlist would require scrolling through the whole thing with no predictable structure to rely on.

**Tradeoff acknowledged:** I recognize the maintainer's point that most users primarily want to see what they added recently, and another reviewer raised a similar point about wanting to quickly find the oldest unwatched item. Recency is a valid and probably more common use case than searching for a specific title. My position is weaker at larger scale — as watchlists grow, "what's new" likely matters more to most users day-to-day than alphabetical browsing. If the team wanted, a good middle ground would be to keep alphabetical as the default for now (since there's no search feature to fall back on) while treating a `sort` query parameter (supporting both `date_added` and `title`) as a near-term follow-up, so users aren't locked into either behavior.
## Comment 6 — Rebase
## Comment 6 — Rebase
**What conflicted:** Running `git rebase origin/main` surfaced an add/add conflict on `.gitignore` (both branches independently added one with slightly different entries), which I resolved by merging both sets of entries into a single file. After that resolved, the rebase completed without further conflict markers — but checking the result afterward, I found that `models.py` had been fully replaced by main's version during the rebase, which silently dropped the `WatchlistEntry` class entirely (since that model never existed on main, only on my feature branch).

**How I resolved it:** I manually re-added the `WatchlistEntry` class to `models.py`, updating `film_id` from `db.Integer` to `db.String(36)` to match the new UUID-based `Film.id` column, consistent with how `CollectionEntry.film_id` was already updated by the refactor. I also updated my test file's fake film ID from an integer placeholder to a UUID-formatted string, and corrected a stale docstring in `add_to_watchlist()` that still described `film_id` as an integer.

**How I verified no conflict remains:** Ran `git status` to confirm a clean working tree post-rebase, with no leftover conflict markers in any file. Ran the full test suite (`pytest tests/ -v`) — all 7 tests pass, including the nonexistent-film test with the corrected UUID-format fake ID.
## PR Description
<!-- Written at the end -->