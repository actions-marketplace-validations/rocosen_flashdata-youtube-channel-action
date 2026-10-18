# YouTube Channel Snapshot - FlashData

Save channel metadata and a bounded recent-video snapshot for a channel-monitoring workflow.

This action calls the `youtube_channel` FlashData API and writes files for subsequent GitHub Actions steps. It does not invoke a language model. A recent-video snapshot is not the complete channel archive.

## Quick start

1. [Create a FlashData account](https://flashdata.dev/signup), obtain an API Key, and fund the required Credits. [API setup and examples](https://flashdata.dev/docs/sources/youtube_channel).
2. Add FLASHDATA_API_KEY as a repository or environment Actions secret. Do not use a plaintext workflow input for the key.
3. Add this manually triggered workflow and review its query before running it. Pin actions to a reviewed full commit SHA for production; the example uses a version tag for readability.

```yaml
name: YouTube Channel Snapshot - FlashData
on:
  workflow_dispatch: {}
permissions:
  contents: read
jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - name: Save channel metadata and a bounded recent-video snapshot for a
          channel-monitoring workflow.
        id: data
        uses: rocosen/flashdata-youtube-channel-action@v0.1.0
        with:
          api-key: ${{ secrets.FLASHDATA_API_KEY }}
          handle: "@GoogleDevelopers"
          options-json: '{"video_limit":5}'
      - name: Save data for the next pipeline stage
        uses: actions/upload-artifact@v4
        with:
          name: youtube-channel
          path: ${{ steps.data.outputs.artifact-directory }}
          retention-days: 7
```

The action is bundled for Node 24; no npm install or Python setup is required in your consuming workflow. Use an up-to-date GitHub runner. No schedule is enabled by this example.

## Inputs

| Input | Required | Meaning |
| --- | --- | --- |
| `api-key` | Yes | FlashData API Key. Pass secrets.FLASHDATA_API_KEY; requests use paid Credits. |
| `handle` | Yes | YouTube channel handle, for example @GoogleDevelopers. |
| `options-json` | No | Optional JSON object with only source-specific options listed in the README. Never include credentials here. |

### Source options

`video_limit` (integer 1–100, default 20).

Unknown fields, incorrect types and invalid ranges fail before an API request. The action makes one paid query without automatic pagination or retries.

## Outputs

| Output | Meaning |
| --- | --- |
| `artifact-directory` | Absolute directory with response.json, records.jsonl and manifest.json. |
| `results-file` | Absolute path to the original response JSON (sanitized lifecycle fields for download jobs). |
| `records-file` | Absolute path to JSONL records for downstream steps. Empty file for an empty result. |
| `result-count` | Number of exported records; not the requested result count. |
| `source` | The fixed FlashData source used by this action. |

Raw API fields: results[]: channelId, channelHandle, name, channelUrl, statistics, videos; usage.credits. records.jsonl exports the original result objects. manifest.json records the source, operation, timestamps, exported count and missing-collection warnings. For ordinary queries it also includes the query/options and API-reported usage.

Empty results produce an empty JSONL file and result-count 0. Missing fields stay missing; response.json preserves the original result envelope. The supplied upload-artifact step stores these files for 7 days. Only use queries and datasets appropriate for the repository’s artifact access level; private signed delivery links may be present in download manifests.

## Cost and failure handling

The MIT action is free to install. FlashData API calls use paid Credits; GitHub runner/artifact, model (if you add one) and customer storage costs are separate. [Current pricing](https://flashdata.dev/pricing). Extended YouTube Search has a separate rate; media delivery is billed by delivered volume. Do not infer price from sample usage fields.

A timeout, HTTP error or response-processing failure may happen after a paid request was accepted. Check [FlashData](https://flashdata.dev) Jobs before rerunning; the action does not retry POST requests. HTTP 401 means check the Key; 402 means insufficient Credits; 429 means rate/quota limit. Keep any download job ID and resume with a read operation. Do not use workflow reruns as polling. Requests have a 65-second per-request timeout and a 16-MiB response limit.

## Development and validation

Run `npm ci`, `npm run check`, `npm test` and `npm run build`. Offline tests use labeled fixtures and make no production API calls. A local or CI test pass is not proof of a paid live run or Marketplace publication. Commit the bundled dist/index.cjs together with source changes.

[Source-specific documentation](https://flashdata.dev/docs/sources/youtube_channel) · [Pricing](https://flashdata.dev/pricing) · [Sign up](https://flashdata.dev/signup)
