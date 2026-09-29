# crawlcove-action

A GitHub Action that runs an SEO crawl of your staging or preview URL on every pull request and fails the check on SEO regressions — broken internal links, missing titles, accidental `noindex`, redirect chains — with the details posted as a PR comment.

## Usage

```yaml
- uses: CrawlCove/crawlcove-action@v1
  with:
    url: https://preview-123.example.com/
```

That crawls up to 100 pages, fails the check on the first broken link, missing title, noindex page or 2+-hop redirect chain, and posts (then keeps updating) one comment on the PR:

> ## CrawlCove SEO crawl: ❌ failed
> Crawled **42** pages from `https://preview-123.example.com/`. The gated checks total **3**, threshold 1.
>
> | Check | Count | Fails the build |
> |---|---:|:---:|
> | Broken links | 2 | yes |
> | Missing titles | 1 | yes |
> | Noindex pages | 0 | yes |
> | Redirect chains (2+ hops) | 0 | yes |
>
> <details><summary>Broken links (2)</summary> … </details>

The PR comment needs `permissions: pull-requests: write` on the job; everything else works with the default token. See [`examples/`](examples/) for a Vercel/Netlify preview trigger, a Next.js build crawled inside the job, and a scheduled WordPress staging crawl.

## Inputs

| Input | Default | What it does |
|---|---|---|
| `url` | *(required)* | Where to start crawling. Same-origin links are followed breadth-first. |
| `max-pages` | `100` | Stop after this many pages. |
| `fail-on` | all four | Comma-separated checks that count towards failure: `broken-links`, `missing-titles`, `noindex`, `redirect-chains`. `none` reports without ever failing. |
| `threshold` | `1` | Fail once the selected checks total this many issues. |
| `ignore-robots` | `false` | Crawl URLs robots.txt disallows. For hosts you own — a staging site behind `Disallow: /`, a local build. |
| `concurrency` | `4` | Simultaneous requests. Lower it for shared hosting. |
| `timeout` | `15000` | Per-request timeout, ms. |
| `comment` | `true` | Post and update the PR comment (pull_request events only). |
| `github-token` | `${{ github.token }}` | Token for the comment. |
| `cli-version` | `v1.1.0` | Git ref of `crawlcove-cli` to install. |

## Outputs

| Output | Meaning |
|---|---|
| `exit-code` | `0` clean, `1` the gated checks reached the threshold, `2` the crawl could not run (usage error, or robots.txt disallows the start URL — set `ignore-robots` if you own it). |
| `pages` | Pages crawled. |
| `broken-links` | Broken internal links: each link to a 4xx/5xx/unreachable page, plus each such page itself. |
| `missing-titles` | Pages with no `<title>`. |
| `noindex` | Pages with a robots `noindex` meta tag. |
| `redirect-chains` | URLs that went through 2 or more redirects (a single redirect is normal and not flagged). |
| `report-path` | Path to the full JSON report — one object per page, field names shared with [crawlcove-export-spec](https://github.com/CrawlCove/crawlcove-export-spec). Upload it with `actions/upload-artifact` to keep it. |

Read outputs from a later step with `${{ steps.<id>.outputs.broken-links }}`.

## How it decides

The action installs [crawlcove-cli](https://github.com/CrawlCove/crawlcove-cli) and runs `crawlcove crawl <url>`. robots.txt is respected by default (a `User-agent: crawlcove-cli` group is honoured over `*`), so a preview host that blocks all crawlers yields exit code 2 and a "could not run" summary rather than a false pass — use `ignore-robots: 'true'` there. Previews are often deliberately `noindex`; drop `noindex` from `fail-on` in that case rather than turning the check off everywhere.

## Works with CrawlCove

This action is the CI half of [Crawl Cove](https://crawlcove.com/?utm_source=github&utm_medium=crawlcove-action), a desktop SEO crawler for Windows and Mac. The action catches regressions before they merge; the desktop app gives you the full site audit — every page, every finding, fixes ranked by impact, history over time, Search Console data alongside. Open the same URL there when a check fails and you want the whole picture.

## Related tools

- [crawlcove-cli](https://github.com/CrawlCove/crawlcove-cli) — the command line crawler this action runs.
- [crawlcove-export-spec](https://github.com/CrawlCove/crawlcove-export-spec) — the JSON Schema and CSV column reference the report's page shape follows.
- [crawl-cove-connector](https://github.com/CrawlCove/crawl-cove-connector) — WordPress plugin that applies Crawl Cove's approved fixes to Yoast, Rank Math, SEOPress or AIOSEO.

## License

MIT — see [LICENSE](LICENSE).
