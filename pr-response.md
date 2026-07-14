# PR Response Doc – CineLog Watchlist Feature

## AI Usage

I used AI tools to help orient myself in the codebase, understand existing service and test patterns, and check whether my commit messages followed conventional commit format. I verified all suggestions against the actual project code before applying changes.

## Comment 1 – Rename

**What I did: renamed the watchlist service function from `save\_to\_watchlist()` to `add\_to\_watchlist()` so it follows the project’s existing verb-to-noun naming convention, matching the pattern used by `add\_to\_collection()`.**


**How I verified: I checked both the service file and the watchlist route file to make sure all references were updated. The call site in `routes/watchlist/watchlist.py` now imports and calls `add\_to\_watchlist()`, and `services/watchlist\_service.py` now defines the function with the new name. I also searched for `save\_to\_watchlist` afterward to confirm no old references remained, then ran `pytest tests/ -v`.**

## Comment 2 – Deduplication

**What I did: I added an `AlreadyInWatchlistError` and updated `add\_to\_watchlist()` so it checks for an existing `WatchlistEntry` with the same `user\_id` and `film\_id` before creating a new one. If a duplicate is found, the function raises `AlreadyInWatchlistError` instead of silently creating another row.**


**How I verified:I modeled the check after the existing `add\_to\_collection()` logic in `services/collection\_service.py`, which first confirms the film exists, then queries for an existing user/film entry before creating a new one. I ran `pytest tests/ -v` after the change to confirm the existing collection tests still passed.**

## Comment 3 – Missing test

**What I did: I created a new `tests/test\_watchlist.py` file and added `test\_add\_to\_watchlist\_nonexistent\_film\_raises`. The test confirms that calling `add\_to\_watchlist()` with a `film\_id` that does not exist raises `FilmNotFoundError`.**


**How I verified: I modeled the test after `test\_add\_to\_collection\_nonexistent\_film\_raises` in `tests/test\_collection.py`, using the same fixture style and assertion pattern. I first ran `pytest tests/test\_watchlist.py -v` to confirm the new test passed by itself, then ran `pytest tests/ -v` to confirm the full test suite passed.**

## Comment 4 – Default visibility

**My position: I am keeping the default watchlist visibility as `public=True`.**


**Reasoning: CineLog is designed as a community film tracking app, so a public default makes sense for a watchlist feature because it helps users share what they plan to watch and makes their profile more useful to other users. A watchlist is less sensitive than something like private notes or account settings; it is part of the social discovery value of the app. Keeping the default public also keeps behavior simple for callers because adding a film to a watchlist immediately makes it visible in the same way other community-facing film activity is visible.**


**Tradeoff acknowledged: The tradeoff is that a private default would give users more control and avoid accidentally exposing what someone plans to watch. That is a valid privacy concern. However, for CineLog’s current community-focused context, I think the better default is public because it supports sharing and discovery. If privacy becomes a larger product requirement later, I would support adding an explicit `public` parameter so callers can choose visibility instead of relying only on the default.S**

## Comment 5 – Sort order

**My position: I am keeping the watchlist sorted alphabetically by film title.**


**Reasoning: For a watchlist, users are often looking for a specific movie they already saved, not just reviewing their most recent activity. Alphabetical order makes the list predictable and scannable as it grows, especially when a user has many saved films. This also matches the current implementation in `get\_watchlist()`, which joins films and orders by `Film.title.asc()`.**


**Engagement with reviewer’s point: I understand the maintainer’s point that many users may want to see what they added recently, and sorting by `date\_added` would support that behavior better. However, I see that as more useful for an activity feed or “recently added” view than for the main watchlist. For the main watchlist endpoint, I think alphabetical order is the better default because it optimizes for finding saved films consistently. A future improvement could add an optional sort parameter, such as `sort=recent` or `sort=title`, but I would keep the default alphabetical for now.**

## Comment 6 – Rebase

**What conflicted:
How I resolved it:
How I verified no conflict remains:**

## PR Description

