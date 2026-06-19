# Releasing the Helm chart

The `helm/` chart is published to GitHub Pages via
[`helm/chart-releaser-action`](https://github.com/helm/chart-releaser-action).
The workflow lives at `.github/workflows/release-helm-chart.yml`.

## How it works

On a push to the `helm-chart-release` branch that touches `helm/**` (or a
manual **Run workflow** dispatch), the workflow:

1. Packages the chart found at `helm/Chart.yaml` (`charts_dir: .` scans the
   repo root for top-level chart directories).
2. Creates a GitHub Release named `agent-sandbox-<chart-version>` with the
   `.tgz` attached.
3. Updates `index.yaml` on the `gh-pages` branch so the chart is consumable as
   a Helm repo.

A chart version is only released once. To publish an update, bump `version:`
in `helm/Chart.yaml` — chart-releaser skips versions that already have a
release.

## One-time repository setup

These steps are done once in the GitHub repo (the workflow cannot do them):

1. **Create the `gh-pages` branch** (chart-releaser pushes `index.yaml` here):

   ```bash
   git checkout --orphan gh-pages
   git rm -rf .
   echo "# Helm chart repository" > index.md
   git add index.md
   git commit -m "Initialize gh-pages for Helm chart repository"
   git push origin gh-pages
   git checkout helm-chart-release
   ```

2. **Enable GitHub Pages**: Settings → Pages → Source = *Deploy from a branch*,
   Branch = `gh-pages` / `(root)`.

3. **Allow Actions to write**: Settings → Actions → General → Workflow
   permissions = *Read and write permissions*. (The workflow also requests
   `contents: write` explicitly.)

## Consuming the chart

Once published (replace `<owner>`):

```bash
helm repo add agent-sandbox https://<owner>.github.io/agent-sandbox
helm repo update
helm install agent-sandbox agent-sandbox/agent-sandbox \
  --set image.tag=v0.4.6
```

> `image.tag` is required — the chart has no default tag.
