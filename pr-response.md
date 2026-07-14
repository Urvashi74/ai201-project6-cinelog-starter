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
Watchlists should be defaulting to `public = True`. I think when a user adds to their watchlist, it should be public unless the user specifically sets to to False.

**Reasoning:**
CineLog seems to a social app. It would make sense that the users share their watchlist by default. Keeping it public will generate more views and sharing and create the social aspect. If the watchlist is private until you toggle it, most users don't care about toggling it to make it public and this would mean the social aspect of sharing your watchlist is never kicked off. Making the defualt to public also helps with the intent of "add a film, share it" in one single actions.

**Tradeoff acknowledged:**
The tradeoff I can this of is that a user who thinks that the watchlists are private, and this would then result in a leak of their data without them realising it. To help this out, we can mention upfront in the UI, during watchlist creation - so the user knows that their list is public by default. The tradeoff is asymmetric: a user who wanted private and got public cannot un-leak, while a user who wanted public and got private is one toggle away. I'm accepting that asymmetry because a watchlist is "films I want to watch" — low-sensitivity compared to ratings or reviews — and because the feature has no public read endpoint in this PR, so nothing is exposed until the sharing UI ships alongside a clear indicator. If a public endpoint lands separately from the sharing UI, we should backfill existing rows to public=False first so users aren't retroactively opted in.

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