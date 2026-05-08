name: Update README Projects

on:
push:
schedule:
- cron: "0 */6 * * *"
workflow_dispatch:

jobs:
update:
runs-on: ubuntu-latest

```
permissions:
  contents: write

steps:
  - name: Checkout
    uses: actions/checkout@v4

  - name: Install jq
    run: sudo apt-get update && sudo apt-get install -y jq

  - name: Fetch repositories
    run: |
      set -e

      curl -s https://api.github.com/users/kasi8112/repos -o repos.json

      jq -r '
        sort_by(.pushed_at)
        | reverse
        | map(select(.fork == false))
        | (
            map(select(.name != "kasi8112"))[:5]
            + map(select(.name == "kasi8112"))
          )
        | unique_by(.name)
        | .[]
        | "- " + .name + ": " + (.description // "No description")
      ' repos.json > projects.txt

  - name: Update README
    run: |
      python3 << 'EOF'
      import re

      with open("README.md", "r", encoding="utf-8") as f:
          content = f.read()

      with open("projects.txt", "r", encoding="utf-8") as f:
          projects = f.read().strip()

      pattern = r"(<!--START_PROJECTS-->)(.*?)(<!--END_PROJECTS-->)"

      replacement = f"\\1\n{projects}\n\\3"

      updated = re.sub(pattern, replacement, content, flags=re.S)

      with open("README.md", "w", encoding="utf-8") as f:
          f.write(updated)
      EOF

  - name: Commit changes
    run: |
      git config user.name "github-actions"
      git config user.email "actions@github.com"

      git add README.md

      git diff --cached --quiet || git commit -m "Auto update projects"

      git push
```
