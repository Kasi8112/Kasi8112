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
        | "- [" + .name + "](" + .html_url + "): " + (.description // "No description")
      ' repos.json > projects.txt

  - name: Update README section
    run: |
      awk '
      BEGIN {inside=0}

      /<!--START_PROJECTS-->/ {
        print
        while ((getline line < "projects.txt") > 0)
          print line
        inside=1
        next
      }

      /<!--END_PROJECTS-->/ {
        inside=0
        print
        next
      }

      !inside {
        print
      }
      ' README.md > temp.md

      mv temp.md README.md

  - name: Commit changes
    run: |
      git config user.name "github-actions"
      git config user.email "actions@github.com"

      git add README.md

      git diff --cached --quiet || git commit -m "Auto update projects"

      git push
```
