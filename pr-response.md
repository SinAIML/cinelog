# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

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
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->