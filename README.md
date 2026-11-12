# CardScope link preview action

Shows what X, LinkedIn, Slack, Discord, iMessage, Bluesky, Threads, Mastodon, WhatsApp, Telegram, and Microsoft Teams will display for your URLs, from a GitHub workflow. Composite action (bash, curl, jq). Call it as `uses: danroot/cardscope-action@v1`.

A free key covers 300 previews a month. Get one at [cardscope.dev/api](https://cardscope.dev/api).

It calls `https://cardscope.dev/v1/preview?source=action` once per URL and prints a markdown table (app, layout, title, whether the image is set, issue count). The same table is appended to `$GITHUB_STEP_SUMMARY`. With `comment: true` on a pull request, it posts or updates one comment marked `<!-- cardscope-link-preview -->`, using `GITHUB_TOKEN`.

The API key is a secret. Store it as `CARDSCOPE_API_KEY`. Do not echo it.

## Inputs

| Input | Required | Default | |
| --- | --- | --- | --- |
| `urls` | yes | | One absolute URL per line |
| `api-key` | yes | | `cs_live_…` |
| `platforms` | no | all 11 | Comma-separated ids, such as `x,slack` |
| `fail-on` | no | `error` | `error`, `warning`, or `never` |
| `comment` | no | `false` | `true` posts or updates one PR comment |

`fail-on: error` fails the step when any issue has severity `error`, or when the preview request itself fails. `warning` also fails on severity `warn`. `never` always exits 0 after writing the table.

## Pull request

```yaml
name: Link previews
on:
  pull_request:
permissions:
  contents: read
  pull-requests: write
jobs:
  cardscope:
    runs-on: ubuntu-latest
    steps:
      - uses: danroot/cardscope-action@v1
        with:
          urls: |
            https://example.com
            https://example.com/pricing
          api-key: ${{ secrets.CARDSCOPE_API_KEY }}
          fail-on: warning
          comment: true
```

## Vercel or Netlify `deployment_status`

Vercel and Netlify emit `deployment_status` when a deploy finishes. `environment_url` is the deployed origin. The comment is posted only when the workflow also has a pull request number (`pull_request`, or `refs/pull/N/merge`). A deployment event without a PR still writes the job summary.

```yaml
name: Link previews on deploy
on:
  deployment_status:
  pull_request:
permissions:
  contents: read
  pull-requests: write
jobs:
  cardscope:
    if: github.event_name == 'pull_request' || github.event.deployment_status.state == 'success'
    runs-on: ubuntu-latest
    steps:
      - uses: danroot/cardscope-action@v1
        with:
          urls: |
            ${{ github.event.deployment_status.environment_url || 'https://example.com' }}
          api-key: ${{ secrets.CARDSCOPE_API_KEY }}
          fail-on: error
          comment: true
```

Get a key at [cardscope.dev/api](https://cardscope.dev/api). The reference is [cardscope.dev/docs/api](https://cardscope.dev/docs/api).
