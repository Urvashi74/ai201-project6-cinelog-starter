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

## Comment 3 — Missing test
**What I did:**
**How I verified:**

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