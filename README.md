# Letterboxd-list-scraper

A tool for scraping Letterboxd lists from a simple URL. The output is a file with film titles, release year, director, cast, owner rating, average rating and a whole lot more (see example CSVs and JSONs in `/example_output/`). 

Version v2.2.0 supports the scraping of:
- **Lists** (e.g. `https://letterboxd.com/bjornbork/list/het-huis-anubis/`)
- **Watchlists** (e.g. `https://letterboxd.com/joelhaver/watchlist/`)
- **User films** (e.g. `https://letterboxd.com/mscorsese/films/`)
- **Generic Letterboxd films** (e.g. `https://letterboxd.com/films/popular/this/week/genre/documentary/`)
- **Roles** (e.g. `https://letterboxd.com/actor/willem-dafoe/`)

## Rate limiting and blocked requests

To avoid being blocked by Letterboxd, requests are rate limited to one every 12 seconds across all threads. Each film takes up to 3 requests, so expect roughly 36 seconds per film (about 2.5 hours for a 250-film list). Multiple lists can be scraped concurrently using separate threads (default max of 4 threads, but this is configurable), but they share the same rate limit.

If Letterboxd still responds with HTTP 429 (too many requests), all threads back off for the duration given in the `Retry-After` header (or 30, 60, 90... seconds if none is given) and the request is retried up to 5 times. Connection errors and timeouts are retried the same way.

The per-film statistics (`Watches`, `List_appearances`, `Likes`, `Fans`, the rating histogram and `Total_ratings`) come from separate endpoints that Cloudflare sometimes blocks with HTTP 403. When this happens those columns are filled with the not-found value (`nan` in CSV, `null` in JSON) and the scraper stops requesting them for the rest of the run, so the remaining fields are still scraped at 1 request per film.

Films that fail to scrape for any other reason are skipped, listed at the end of the run, and the program then exits with a non-zero status so the incomplete output is not mistaken for a full one.

## Getting Started

### Dependencies

Requires Python 3.x, numpy, BeautifulSoup (bs4), requests, tqdm and **lxml**.

If dependencies are not met it is recommended to install everything needed in one go using `pip install -r requirements.txt` (ideally in a clean virtual environment).

### Installing

* Clone the repository and work in there.

### Executing program

* Execute the program by running `python -m listscraper [options] [list-url]` on the command line in the project directory. 

    Multiple list URLs can be provided, separated by a space. The output file(s) can then be found in the folder `/scraper_outputs/`, which will be created if not already present.
    Some of the optional flags are:
    - `-p` or `--pages` can be used to select specific pages.
    - `-on` or `--output-name` can be used to give the output file(s) a user-specified name.
    - `-f` or `--file` can be used to import a .txt file with multiple list URLs that should be scraped.
    - `-op` or `--output-path` can be used to write the output file(s) to a desired directory.
    - `-ofe` or `--output-file-extension` can be used to specify what type of file is outputted (support for CSV and json).
    - `--concat` will concatenate all films of the given lists and output them in a single file.
    - `--html-dir` can be used to read the list pages from a directory of `.html` files saved from your browser, instead of downloading them (useful when Letterboxd blocks the list page itself). Requires exactly one list URL on the command line; `-p` is ignored.

> [!NOTE]
> Please use `python -m listscraper --help` for a full list of all available flags including extensive descriptions on how to use them.

> [!TIP]
> Scraping multiple lists is most easily done by running `python -m listscraper -f <file>` with a custom .txt file that contains the URL on each newline. Each newline can take unique `-p` and `-on` optional flags. For an example of such a file please see `target_lists.txt`.

> [!IMPORTANT]
> Program currently does not support the scraping of extremely long generic Letterboxd pages (e.g. `https://letterboxd.com/films/popular/this/week/genre/documentary/`, which contains ~152000 films). To circumvent this, please use the `-p` flag to make a smaller page selection.

## TODO

* Add further options for output, currently supports CSV and json.
* Add scrape functionality for user top 4 and diary.
* Add `-u <username>` flag that scrapes the diary, top 4, films and lists of a single user.
* Add a `--meta-data` flag to print original list name, scrape date, username above CSV header.
* Optimize thread usage to increase scrape speed.


## Authors

Arno Lafontaine  

## Acknowledgments

Thanks to BBotml for the inspiration for this project https://github.com/BBottoml/Letterboxd-friend-ranker.
