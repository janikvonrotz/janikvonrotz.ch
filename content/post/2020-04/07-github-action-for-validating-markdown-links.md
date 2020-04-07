---
title: "Github Action for validating markdown links"
slug: github-action-for-validating-markdown-links
date: 2020-04-07T14:18:35+02:00
categories:
 - Scripting
tags:
 - scripting
 - github
 - action
 - quality assurance
images:
 - "/images/interwined zippers.jpg"
---

GitHub Action are free computing resources to run CI/CD jobs that build, lint, test or deploy a software project. On the [Awesome PowerShell](https://github.com/janikvonrotz/awesome-powershell) I asked contributors to submit a PR for a quality check job. Not much later [Frederik Hjorslev](https://github.com/hjorslev) submitted a nice solution.
<!--more-->

The first file he submitted was the script that analyzes any markdown file, extracts links and requests them.

**DeadLinksAnalyzer.ps1**

```powershell
function Test-MarkdownLinks([String]$Path) {
    $unreachable = @()
    # Get markdown files recursively
    $files = Get-ChildItem -Path $Path -Recurse -Include "*.md"

    $files | ForEach-Object {
        $fileName = $_.Name
        Write-Host "Analyzing $fileName"

        $urls = Select-String -Path $_ -Pattern "\[.+\]\((http.*?)\)" | ForEach-Object { $_.matches.Groups[1] } | Select-Object

        $urls | ForEach-Object {
            $url = $_.Value
            Write-Host "Requesting url $url"

            try {
                $request = Invoke-WebRequest -Uri $url
            } catch {
                Write-Warning -Message "Found dead url $url in $fileName"
                $unreachable += $url
            }
        }
    }

    # Output urls
    return $unreachable
}

$DeadLinks = Test-MarkdownLinks -Path ".\readme.md"
if ($DeadLinks) {
    Write-Host -Object '--- DEAD LINKS FOUND ---' -ForegroundColor Red
    foreach ($DeadLink in $DeadLinks) {
        Write-Host -Object $DeadLink -ForegroundColor Red
    }
    exit 1
}
```

Dead links are reported at the end and the scripts exit with 1 if one or more findings occured.

The other file he submitted is the GitHub Action configuration file.

**.github/workflows/.github/workflows/**

```yml
# This is a basic workflow to help you get started with Actions

name: Quality Assurance

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  pull_request:
    branches: [ master ]
  schedule:
  # * is a special character in YAML so you have to quote this string
  - cron:  '0 0 * * 0'

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  analyze-readme:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      - name: Analyze Links
        shell: pwsh
        run: |
          ./DeadLinksAnalyzer.ps1
```

This Action executes the the analyzer script every week and for every pull request. This ensures that contributors submit valid links.

Pretty neat. Thanks to all the Awesome PowerShell contributors.
