# warmup.rocks — CDN Cache Warmer Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-warmup.rocks%20CDN%20Cache%20Warmer-dc0000?logo=github)](https://github.com/marketplace/actions/warmup-rocks-cdn-cache-warmer)

Trigger a cache warm run on [warmup.rocks](https://warmup.rocks) right after your
deploy. Deploys purge your CDN cache — this Action makes sure it's hot again
before the first visitor arrives, instead of waiting for the next scheduled run.

## Setup

1. In your [warmup.rocks dashboard](https://warmup.rocks/app/), open
   **Project → Settings → Deploy hook** and click **Generate hook URL**.
2. In your GitHub repository, add the URL as an encrypted secret named
   `WARMUP_HOOK_URL` (**Settings → Secrets and variables → Actions**).
3. Add the step to your deploy workflow:

```yaml
- name: Warm CDN cache
  uses: warmup-rocks/warm-action@v1
  with:
    hook-url: ${{ secrets.WARMUP_HOOK_URL }}
```

## Full example

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # … your build & deploy steps …

      - name: Warm CDN cache
        uses: warmup-rocks/warm-action@v1
        with:
          hook-url: ${{ secrets.WARMUP_HOOK_URL }}
```

## Inputs

| Input      | Required | Description                                                                 |
| ---------- | -------- | --------------------------------------------------------------------------- |
| `hook-url` | yes      | Your project's deploy-hook URL. Always pass it via an encrypted secret.     |

## Outputs

| Output    | Description                                                                                  |
| --------- | -------------------------------------------------------------------------------------------- |
| `run-id`  | ID of the warm run that was started (empty if skipped).                                       |
| `skipped` | `already-running` while a warm pass is in progress, `cooldown` within 5 min of the last hook. |

Skips are **not** failures — they mean your cache is already being warmed.
The step only fails if the hook URL is unreachable, invalid or rejected.

## How it works

The Action sends a single `POST` request to your hook URL. warmup.rocks then
requests your pages from 90+ global edge locations, exactly like your scheduled
warm runs. The run appears in your dashboard with the trigger `deploy`.

- Hook runs are rate-limited to one per project every 5 minutes, so rapid
  deploy sequences (retries, matrix builds) don't cause a run avalanche.
- The hook URL is the only credential. You can rotate or disable it anytime
  under **Project → Settings → Deploy hook**.

## Docs & support

- [Deploy hooks documentation](https://warmup.rocks/docs/deploy-hooks)
- [Contact](https://warmup.rocks/contact)

## License

[MIT](LICENSE)
