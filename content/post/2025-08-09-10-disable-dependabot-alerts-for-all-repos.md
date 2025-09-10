---
title: Disable dependabot alerts for all repos
slug: 2025-08-09-10-disable-dependabot-alerts-for-all-repos
date: 2025-09-10T10:45:30+02:00
categories:
  - Software development
tags:
  - noice
  - cancelling
  - github
  - microsoft
images:
  - /images/clippy-vs-code.png
draft: false
---
It is well known that GitHub dependabot alerts and PRs are less than helpful. For hubbers the dependabot is very similar to what clippy was to the office users. It tries to help, but is very distracting for solving the actual problem.

Disabling dependabot alerts for one repo is simple. Got to this page `https://github.com/$GITHUB_USERNAME/$REPO/settings/security_analysis` and click *disable*. But doing this for a 100 or 1000 repos is not feasible. We need a script to automate this process. Let me show you how.

<!--more-->

In order to run the scripts you need to create a personal access token to access the GitHub API. Create a token with read/write access to user and repo here: <https://github.com/settings/tokens>

And then you are ready to configure and run the script. Simply change the `$GITHUB_USERNAME` and set the `$GITHUB_TOKEN` variables.

```bash
#!/bin/bash

GITHUB_USERNAME="janikvonrotz"
GITHUB_TOKEN="*******"

GITHUB_TOTAL_REPOS=$(curl -s -H "Authorization: token $GITHUB_TOKEN" "https://api.github.com/user" | jq '.public_repos + .total_private_repos')
GITHUB_PAGINATION=100
GITHUB_NEEDED_PAGES=$(( (GITHUB_TOTAL_REPOS + GITHUB_PAGINATION - 1) / GITHUB_PAGINATION ))

echo "Found $GITHUB_TOTAL_REPOS repos, processing over $GITHUB_NEEDED_PAGES page(s)..."

for (( PAGE=1; PAGE<=GITHUB_NEEDED_PAGES; PAGE++ )); do
    REPOS=$(curl -s -H "Authorization: token $GITHUB_TOKEN" \
        "https://api.github.com/user/repos?per_page=$GITHUB_PAGINATION&page=$PAGE&type=owner")
    
    echo "$REPOS" | jq -r '.[] | select(.owner.login == "'"$GITHUB_USERNAME"'") | .full_name' | while read REPO_FULL; do
    
        echo "Disabling dependabot for: $REPO_FULL"
        
        # Disable dependabot vulnerability alerts
        curl -s -X DELETE \
            -H "Authorization: token $GITHUB_TOKEN" \
            -H "Accept: application/vnd.github.v3+json" \
            "https://api.github.com/repos/$REPO_FULL/vulnerability-alerts" \
            -w " -> Status: %{http_code}\n" -o /dev/null

        # Disable dependabot automated security fixes (PRs)
        curl -s -X DELETE \
            -H "Authorization: token $GITHUB_TOKEN" \
            -H "Accept: application/vnd.github.v3+json" \
            "https://api.github.com/repos/$REPO_FULL/automated-security-fixes" \
            -w " -> Status: %{http_code}\n" -o /dev/null
    done
done
```

The script will create list of repos connected to your account. Then it loops through the list disabling the alerts.

The script above only works for personal accounts. If you want to disable the alerts for all repos of an organisation, use this script:

```bash
#!/bin/bash

# Configuration
ORG_NAME="Mint-System"
GITHUB_TOKEN="*******"

GITHUB_TOTAL_REPOS=$(curl -s -H "Authorization: token $GITHUB_TOKEN" \
    "https://api.github.com/orgs/$ORG_NAME" | jq -r '.public_repos + .total_private_repos')
GITHUB_PAGINATION=100
GITHUB_NEEDED_PAGES=$(( (GITHUB_TOTAL_REPOS + GITHUB_PAGINATION - 1) / GITHUB_PAGINATION ))

echo "Found $GITHUB_TOTAL_REPOS repos in org '$ORG_NAME', processing over $GITHUB_NEEDED_PAGES page(s)..."

# Loop through each page of organization repositories
for (( PAGE=1; PAGE<=GITHUB_NEEDED_PAGES; PAGE++ )); do
    REPOS=$(curl -s -H "Authorization: token $GITHUB_TOKEN" \
        "https://api.github.com/orgs/$ORG_NAME/repos?per_page=$GITHUB_PAGINATION&page=$PAGE&type=public")

    echo "$REPOS" | jq -r '.[] | .full_name' | while read REPO_FULL; do
    
        echo "Disabling dependabot for: $REPO_FULL"

        # Disable dependabot vulnerability alerts
        curl -s -X DELETE \
            -H "Authorization: token $GITHUB_TOKEN" \
            -H "Accept: application/vnd.github.v3+json" \
            "https://api.github.com/repos/$REPO_FULL/vulnerability-alerts" \
            -w " -> Status: %{http_code}\n" -o /dev/null

        # Disable dependabot automated security fixes (PRs)
        curl -s -X DELETE \
            -H "Authorization: token $GITHUB_TOKEN" \
            -H "Accept: application/vnd.github.v3+json" \
            "https://api.github.com/repos/$REPO_FULL/automated-security-fixes" \
            -w " -> Status: %{http_code}\n" -o /dev/null
    done
done
```

You can use the same access token. Simply set the org name and your are good to go.