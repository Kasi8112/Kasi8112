name: Update README Projects

on:
  push:
  schedule:
    - cron: "0 */6 * * *"
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install jq
        run: sudo apt-get update && sudo apt-get install -y jq

      - name: Fetch repos
        run: |
          set -e
          curl -s https://api.github.com/users/kasi8112/repos -o repos.json

          if [ ! -s repos.json ]; then
            echo "API returned empty response"
            exit 1
          fi

          jq -r '
            sort_by(.pushed_at) | reverse
            | map(select(.fork == false))
            | .[:5]
            | .[]
            | "- [" + .name + "](" + .html_url + "): " + (.description // "No description")
          ' repos.json > projects.txt

      - name: Update README
        run: |
          set -e
          awk '
          /<!--START_PROJECTS-->/ {
            print;
            while ((getline line < "projects.txt") > 0) print line;
            next
          }
          /<!--END_PROJECTS-->/ {print; next}
          {print}
          ' README.md > new.md

          mv new.md README.md

      - name: Commit changes
        run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git add README.md
          git commit -m "Auto update projects" || echo "No changes"
          git push          )
        | unique_by(.name)
        | .[]
        | "- [" + .name + "](" + .html_url + "): " + (.description // "No description")
      ' repos.json > projects.txt

  - name: Update README section
    run: |
      awk '
      BEGIN {inside=0}

      /<!--START_PROJECTS-->/ {
- [AVR-Projects](https://github.com/Kasi8112/AVR-Projects): Contains the projects developed using AVR microcontrollers
- [Kasi8112](https://github.com/Kasi8112/Kasi8112): Dashboard
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
