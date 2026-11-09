# YouTube Channel Data

Get YouTube channel details, statistics and up to 100 recent videos from a handle. Export as JSON and JSONL.

Powered by the [FlashData YouTube Channel API](https://flashdata.dev/docs/sources/youtube_channel?utm_source=github-actions&utm_medium=integration&utm_campaign=intent_gtm_2026q4&utm_content=youtube_channel-task). A recent-video snapshot is not the complete channel archive.

## Quick start

Add a [FlashData API key](https://flashdata.dev/signup?utm_source=github-actions&utm_medium=integration&utm_campaign=intent_gtm_2026q4&utm_content=youtube_channel-task) as the repository secret `FLASHDATA_API_KEY`. API calls require FlashData Credits ([pricing](https://flashdata.dev/pricing?utm_source=github-actions&utm_medium=integration&utm_campaign=intent_gtm_2026q4&utm_content=youtube_channel-task)). Then add this workflow:

```yaml
name: YouTube Channel Data
on:
  workflow_dispatch: {}
permissions:
  contents: read
jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - name: YouTube Channel Data
        id: data
        uses: rocosen/flashdata-youtube-channel-action@v0.1.1
        with:
          api-key: ${{ secrets.FLASHDATA_API_KEY }}
          handle: "@GoogleDevelopers"
          options-json: '{"video_limit":5}'
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: youtube-channel
          path: ${{ steps.data.outputs.artifact-directory }}
          retention-days: 7
```

Run it from the **Actions** tab. Download the exported files from the run’s **Artifacts**, or use the output paths in later steps. Requires an up-to-date runner with Node 24 support; no dependency installation is needed. For production, pin actions to a reviewed commit SHA.

## Inputs

| Input | Required | Description |
| --- | --- | --- |
| `api-key` | Yes | FlashData API Key. Pass secrets.FLASHDATA_API_KEY; requests use paid Credits. |
| `handle` | Yes | YouTube channel handle, for example @GoogleDevelopers. |
| `options-json` | No | Optional JSON object with only source-specific options listed in the README. Never include credentials here. |

### Options

Pass these fields in `options-json`:

`video_limit` (integer 1–100, default 20).

Invalid options fail before an API request. One paid query per run; no automatic pagination or retries.

## Outputs

- `response.json`: API response, including available result fields and usage.
- `records.jsonl`: result objects, one per line.
- `manifest.json`: source, operation, timestamps, record count and export warnings, plus query/options and reported usage.

Returned API fields: results[]: channelId, channelHandle, name, channelUrl, statistics, videos; usage.credits.

Empty results produce an empty JSONL file and `result-count: 0`. Optional fields may be absent.

| Step output | Description |
| --- | --- |
| `artifact-directory` | Absolute directory with response.json, records.jsonl and manifest.json. |
| `results-file` | Absolute path to the original response JSON (sanitized lifecycle fields for download jobs). |
| `records-file` | Absolute path to JSONL records for downstream steps. Empty file for an empty result. |
| `result-count` | Number of exported records; not the requested result count. |
| `source` | The fixed FlashData source used by this action. |

## Pricing and errors

The action is free to install. FlashData API calls use paid Credits; GitHub runner/artifact costs are separate. [Current pricing](https://flashdata.dev/pricing?utm_source=github-actions&utm_medium=integration&utm_campaign=intent_gtm_2026q4&utm_content=youtube_channel-task).

401: check the API key. 402: add Credits. 429: rate/quota limit. After a timeout, check [FlashData Jobs](https://flashdata.dev?utm_source=github-actions&utm_medium=integration&utm_campaign=intent_gtm_2026q4&utm_content=youtube_channel-task) before rerunning: a request may already have been accepted. Paid POST requests are not retried. Requests have a 65-second timeout and a 16-MiB response limit.

## Development

`npm ci` → `npm run check` → `npm test` → `npm run build`. Tests use offline fixtures. Commit the bundle with runtime changes.
