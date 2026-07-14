# PR Response Doc — CineLog Watchlist Feature

## AI Usage
I used Claude Code throughout this project. Most importantly, at the start I asked it to walk me through the repo module by module — what `models.py`, `services/`, `routes/`, and `tests/` were each responsible for — so I had a mental map before touching anything.

Beyond that, I used AI for:
- **Git commands when stuck.** Rebasing, resolving conflict markers, aborting a rebase mid-`squash`, and cleaning up commit messages to conventional format — I asked AI to explain what each command did and why, not just to run it for me.
- **Locating the default visibility setting.** When I decided `public=True` should be the default, I asked AI to confirm where in the code that default is actually applied (`WatchlistEntry.public = db.Column(..., default=True)` in `models.py`) so my Comment 4 reasoning matched the real behavior.
- **Verifying the feature end-to-end.** I asked AI to run `pytest tests/ -v` and confirm all 6 tests passed after the `film_id` UUID fix, so I knew the type change hadn't silently broken anything downstream.

## Comment 1 — Rename
**What I did:**
Renamed all occurences of save_to_watchlist() to add_to_watchlist() to follow the project's naming convention. I used the VSCode's internal shortcut of Command+Shift+F to look for all the current occurences of save_to_watchlist() and then renamed each on individually to add_to_watchlist()
**How I verified:**
Checked in the entire repo for all occurences of save_to_watchlist(), verified there were none left. I used the VSCode's internal shortcut of Command+Shift+F to search across the repositories for any remaining occurences of save_to_watchlist() and found none.

## Comment 2 — Deduplication
**What I did:**
Added a deduplication logic to query in the `WatchListEntry` model if a particular `user_id` and `film_id` was already present. If it was present, we throw `AlreadyInWatchListError`, else we make an entry in the WatchList DB. I modeled it on `add_to_collection()` in `services/collection_service.py`, which does the same `.filter_by().first()` check before inserting.
**How I verified:**
I added a testcase in `test_watchlist.py` that handles the deduplication scenario - we ensure to try and add a duplicate entry which then throws `AlreadyInWatchListError`. After that, we also verify that the count of the particular record is 1 in `WatchListEntry`.

## Comment 3 — Missing test
**What I did:**
Added a test case `test_add_to_watchlist_nonexistent_film_raises` to the file `test_watchlist.py` which addresses the missing test case that checks for non existent film to be added to the watchlist and the right error to be thrown. I modeled it on `test_add_to_collection_nonexistent_film_raises` in `tests/test_collection.py`.
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
I agree with the comment here. I will change the `get_watchlist()` in `services/watchlist_service.py` to sort by `date_added` descending instead of `Film.title` ascending, so the newest additions show up first.

**Reasoning:**
When I consider about how I user a watchlist myself, it feels more like a queue than a library. What I usually want to see is "what did I just add that I haven't watched yet", not what's alphabetically in order. I don't exactly remember all the titles to actually go through the watchlist like a dictionary. Since it is a to-do list for movies, it makes sense to have the most recent on the top. 

**Engagement with reviewer's point:**
The @dev-lead's point is that recent additions are what users mostly care about, and I agree with it. Alphabetically ordered could look more aesthetic and ordered, but nobody opens their watchlist to look at a sorted list, they just want to puck something to watch. Recency is a better signal for that than the first letter of the title. If we later find that users are hunting for a specific film by name, a search box is an easier solution for them to find the title, than alphabetical order.

## Comment 6 — Rebase
**What conflicted:**
I had 3 files show me conflicts. One was `.gitignore`, one was this file - `pr-response.md` and the third being the `models.py` file.
**How I resolved it:**
I was able to use the VSCode UI to check for the conflict files. The first one that I saw was the `.gitignore` file, I chose to use my current version as it had more items than the main branch, and then I marked it resolved it. I ran `git rebase --continue`, which then showed me a conflict in `pr-response.md`. I chose my version again, to not lose out on all the entries I had made in this file. Once done, I staged the changes using git add `filename`, and then hit `git rebase --continue`. I saw the UUID conflict in `models.py`. I made sure that WatchList Entry remained in the file, and I also made sure than the Films entry had the UUID change come in from Main. Once this was done I hit `git rebase --continue`. I had to push my changes, and for this I had to do a `git pull` first, followed by `git push`.
**How I verified no conflict remains:**
I checked my branch status with `git status`, which showed me no signs of being in a git rebase state. I also tried running `git rebase origin/main` again, which gave me the output of `Current branch feature/watchlist is up to date.` I also manually compared the code in `main` branch in the original repo and in my feature/watchlist branch. I also didn't see any merge commits.

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->
What this feature does

Adds a watchlist — a per-user list of films saved for later viewing, kept separate from the existing collection (which tracks films already watched). Users can add a film once, view their list back in newest-first order, and each entry carries a public visibility flag. Two endpoints ship under the /watchlist blueprint: POST /watchlist/<user_id>/add to save a film, and GET /watchlist/<user_id> to retrieve the list. Duplicate adds raise AlreadyInWatchlistError; adding a nonexistent film raises FilmNotFoundError.

Design decisions

- Default visibility is public=True. CineLog is a social product, and requiring users to toggle a flag to share means the social loop rarely starts. A watchlist is low-sensitivity ("films I want to watch") compared to ratings or reviews, and this PR ships no public-read endpoint — so nothing is actually exposed until the sharing UI lands alongside a clear public indicator. If a public read endpoint lands separately from that UI, existing rows should be backfilled to public=False first so no one is retroactively opted in.
- Sort order is date_added descending. get_watchlist returns newest entries first rather than alphabetical by title. A watchlist behaves more like a to-do queue than a library — the common question is "what did I just add?" not "which film starts with G?" If title lookup becomes a need later, a search box is a better answer than an alphabetical sort.

How to manually test

1. Activate the venv and start the server: source .venv/bin/activate && flask --app app run.
2. Seed one user and two films via the Flask shell or your seed script; note their UUIDs.
3. Add the first film: curl -X POST http://localhost:5000/watchlist/<USER_UUID>/add -H "Content-Type: application/json" -d '{"film_id": "<FILM_UUID_1>"}'. Expect a success response with "public": true.
4. Add the second film with the same command and <FILM_UUID_2>.
5. View the watchlist: curl http://localhost:5000/watchlist/<USER_UUID>. Confirm the second film appears first — validates the sort-order decision.
6. Confirm every entry in that response has "public": true — validates the visibility default.
7. Retry step 3 with <FILM_UUID_1> a second time. Expect a 4xx "already in watchlist" error, and re-run step 5 to confirm no duplicate row was created.
8. Retry step 3 with a random UUID that doesn't match any film. Expect a 4xx "film not found" error.
9. Run the automated suite: pytest tests/ -v. Expect all 6 tests to pass (4 collection, 2 watchlist).

## git log --oneline screenshot

<img width="932" height="604" alt="Screenshot 2026-07-14 at 18 56 54" src="https://github.com/user-attachments/assets/971a389f-1014-40e7-bec4-0d56ae964b57" />

