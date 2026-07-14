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

## git log --oneline screenshot

<img width="786" height="418" alt="Screenshot 2026-07-14 at 18 14 53" src="https://github.com/user-attachments/assets/3d009ebb-7f36-4dab-b9f9-913b1b9ded5f" />
