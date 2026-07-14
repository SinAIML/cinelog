# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- How AI was used during this work -->
- Drafted the PR response text (positions, reasoning, and examples) and iterated wording for reviewer clarity.
- Explored visibility and ordering tradeoffs and produced concrete example scenarios (Alice/Bob) to help inform the default decision.
- Suggested concrete code changes and test assertions (newest-first ordering, optional `?sort` param, visibility filtering) and verified where enforcement is required in the service/route layer.
- Generated recommended manual testing steps and example API calls for reviewer verification.

## Comment 1 — Rename
**What I did:**
- Changed all references of save_to_watchlist() to add_to_watchlist()
**How I verified:**
- I went to each reference to verify it's changed from save_to_watchlist() to add_to_watchlist()

## Comment 2 — Deduplication
**What I did:**
- Added deduplication logic to add_to_watchlist() in services/watchlist_service.py. Used the same pattern from collection_services.py. 
**How I verified:**
- Added a Class "AlreadyInWatchlistError" at the beginning of the file which can be utilized in add_to_watchlist function to raise an exception when film is already in the watchlist. 
- Ran pytest tests/test_watchlist.py -v and it passed successfully.

## Comment 3 — Missing test
**What I did:**
- Added test_watchlist.py following the same pattern in test_collection.py
**How I verified:**
- Ran pytest tests/test_watchlist.py -v and it passed successfully.

## Comment 4 — Default visibility
**My position:**
- default=True or default=False only affects what new rows store.
- The real access control comes from whether get_watchlist() filters by public=True
**Reasoning:**
- Since this is a community film tracking app, “I want others to see my list” → public by default. Good for “social discovery” or “my favorites list” experiences. 
- optionally allow private/hidden mode if the user chooses.
**Example Scenarios:**
1) Public by default
- Alice adds “Inception” to her watchlist.
- The entry is stored as public=True.
- Bob requests GET /watchlist/<alice_id>.
- Bob immediately sees “Inception”.
- If Alice wants some items hidden later, she must set those entries to public=False.
    **Result:**
    - Alice’s list is visible immediately
    - privacy requires explicit hiding

2) Mixed / per-item visibility
- Alice adds “Inception” to her watchlist and leaves it public (public=True).
- Alice also adds “Secret Project” and marks it private (public=False).
- Bob requests GET /watchlist/<alice_id>.
- Bob sees “Inception” but does not see “Secret Project”.
- Alice can still view her full watchlist including both items if there is an owner-only view.
    **Result:**
    - Alice controls visibility per film
    - Bob sees only the shared entries

**Tradeoff acknowledged:**
- private by defalut is not possible.

## Comment 5 — Sort order
**My position:**
- I think sort order by date-added makes more sense to see the latest first.
- UX: users see what they recently saved at the top (better for “what I just found” workflows).
- Current code: get_watchlist() uses .order_by(Film.title.asc()), so lists are alphabetical.
- Change needed: order by the watchlist entry timestamp (date_added) instead.
**Reasoning:**
- Showing newest items first aligns with how users typically interact with saved content — they want to quickly access what they most recently discovered. Newest-first reduces friction for the common "I just found this — where did I save it?" flow and matches other "save for later" UX patterns.
**Engagement with reviewer's point:**
- I understand the argument for alphabetical ordering when users are browsing or looking for a known title. To accommodate both needs, I propose newest-first as the default and adding an optional `?sort=title` (or `?sort=alpha`) query parameter for alphabetical results. This keeps the default experience focused on recency while preserving discoverability for title-based browsing.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
Feature overview
- Adds a basic per-user watchlist feature with endpoints to add and view watchlist items.

What changed
- New business logic in `services/watchlist_service.py`: `add_to_watchlist()` (with deduplication) and `get_watchlist()`.
- New routes in `routes/watchlist/watchlist.py`: `GET /watchlist/<user_id>` and `POST /watchlist/<user_id>/add`.
- Data model: `WatchlistEntry` (id, user_id, film_id, date_added, public).
- Tests: `tests/test_watchlist.py` added following the `test_collection.py` patterns.

Design decisions
- Default visibility: public by default for social/discovery-first UX. Private/hidden mode is supported per-item via the `public` flag; enabling true private-by-default would also require changing the model default and enforcing `public=True` filtering on the public-facing endpoint.
- Default sort order: newest-first (`date_added` descending) to prioritize recently saved items. An optional query parameter (e.g. `?sort=title`) can be added to support alphabetical views.

Manual testing steps
1. Run unit tests:
```bash
pytest tests/test_watchlist.py -q
```
2. Start the app and exercise endpoints:
```bash
# add a watchlist item
curl -X POST -H "Content-Type: application/json" -d '{"film_id": 1}' http://localhost:5000/watchlist/<user_id>/add

# view public watchlist
curl http://localhost:5000/watchlist/<user_id>
```

Notes and follow-ups
- This branch uses integer `film_id`s (pre-refactor). When rebasing onto main (which migrated film IDs to UUID), update callsites and tests accordingly.
- To make private-by-default real, set the model default to `public=False` and ensure `get_watchlist()` or the route filters by `public=True` for non-owner viewers.