# Install your profile refresh

The GitHub integration rejected repository writes, so these changes are prepared but are not live.

## 1. Upload the design

Open https://github.com/hatchakala/hatchakala and choose **Add file → Upload files**.
Drag in **README.md** and the whole **assets** folder from this package.
Commit directly to **main**. Keep the header at **assets/header.svg**.

The README includes the snake image links. They will begin working after step 2 completes.

## 2. Add the snake workflow

Open https://github.com/hatchakala/hatchakala/new/main
Enter **.github/workflows/snake.yml** as the filename.
Copy the YAML below into the editor and commit to **main**.

The same workflow is included at .github/workflows/snake.yml in this package.
Folders starting with a dot are hidden in macOS Finder, so the copy below is provided for convenience.

## 3. Check the first run

Open https://github.com/hatchakala/hatchakala/actions
Look for **Update contribution snake**. Adding the workflow to main should start it automatically.
If it does not, select the workflow and choose **Run workflow → main → Run workflow**.

After the run succeeds, refresh https://github.com/hatchakala.
GitHub may briefly cache the previous README or images.

The workflow refreshes the contribution calendar daily at 06:23 UTC and can also be run manually.
It publishes two SVGs to a separate **output** branch and selects a light or dark version automatically.
It uses the repository's built-in GitHub token, so no personal access token or extra secret is needed.
Scheduled runs are subject to GitHub's scheduling delays and inactivity rules.

## Files

- README.md: concise introduction, four project highlights, skills, and snake embed
- assets/header.svg: custom animated orbital header with a reduced-motion alternative
- .github/workflows/snake.yml: daily generation and publishing
- SETUP.md: this guide; it does not need to be uploaded

## Workflow to copy

```yaml
name: Update contribution snake

on:
  schedule:
    - cron: "23 6 * * *"
  workflow_dispatch:
  push:
    branches: [main]
    paths:
      - .github/workflows/snake.yml

permissions:
  contents: write

concurrency:
  group: contribution-snake
  cancel-in-progress: false

jobs:
  generate:
    name: Generate and publish snake
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: Generate light and dark animations
        uses: Platane/snk/svg-only@d8f6715049803e982ee5ff501b6b9b7d5deeb09b # v3
        with:
          github_user_name: hatchakala
          outputs: |
            dist/github-snake.svg?color_snake=#7C3AED&color_dots=#E8EDF4,#C4B5FD,#A78BFA,#7C3AED,#0891B2
            dist/github-snake-dark.svg?palette=github-dark&color_snake=#22D3EE&color_dots=#161B22,#312E81,#5B21B6,#7C3AED,#A78BFA

      - name: Publish SVGs to the output branch
        working-directory: dist
        env:
          GH_TOKEN: ${{ github.token }}
        shell: bash
        run: |
          set -euo pipefail
          test -s github-snake.svg
          test -s github-snake-dark.svg
          git init --initial-branch=output
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git remote add origin "https://x-access-token:${GH_TOKEN}@github.com/${GITHUB_REPOSITORY}.git"
          if git ls-remote --exit-code --heads origin output > /dev/null; then
            git fetch --depth=1 origin output
            git reset --soft FETCH_HEAD
          fi
          git add -- github-snake.svg github-snake-dark.svg
          if git diff --cached --quiet; then
            echo "Contribution animations are already current."
            exit 0
          fi
          git commit -m "Refresh contribution snake"
          git push origin HEAD:output
```

## Components

- Snake generator: https://github.com/Platane/snk
- Typing animation: https://github.com/DenverCoder1/readme-typing-svg
- Skill icons: https://skillicons.dev
- Badges: https://shields.io

The workflow syntax, publisher logic, and header rendering were checked locally.
The actual contribution animation still needs its first successful GitHub Actions run.

