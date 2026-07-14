# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
Renamed all occurences of save_to_watchlist() to add_to_watchlist() to follow the project's naming convention.
**How I verified:**
Checked in the entire repo for all occurences of save_to_watchlist(), verified there were none left.

## Comment 2 — Deduplication
**What I did:**
Added a deduplication logic to query in the `WatchListEntry` model if a particular `user_id` and `film_id` was already present. If it was present, we throw `AlreadyInWatchListError`, else we make an entry in the WatchList DB.
**How I verified:**
I added a testcase in `test_watchlist.py` that handles the deduplication scenario - we ensure to try and add a duplicate entry which then throws `AlreadyInWatchListError`. After that, we also verify that the count of the particular record is 1 in `WatchListEntry`.

## Comment 3 — Missing test
**What I did:**
Added a test case `test_add_to_watchlist_nonexistent_film_raises` to the file `test_watchlist.py` which addresses the missing test case that checks for non existent film to be added to the watchlist and the right error to be thrown.
**How I verified:**
I ran the test case with the command `pytest tests/test_watchlist.py -v`. The test passed. For a good measure, I also ran the entire test suite `pytest tests/ -v`.

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