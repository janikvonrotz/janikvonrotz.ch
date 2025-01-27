---
title: Work with LLMs on the command line
slug: work-with-llms-on-the-command-line
date: 2025-01-27T16:28:35+01:00
categories:
  - Software development
  - Large Language Model
tags:
  - terminal
  - llm
  - bash
  - script
  - task
  - taskfile
images:
  - /images/simon-willison-live-coding-llm.jpg
draft: false
---
I used VSCodium and Codeium to develope Python code. VSCode is my editor of choice and Codeium is a well integrated AI-tool that helps writing code. While it solved a lot of problems for me, especially writing boilerplate code, I became more and more frustrated using this setup.

No longer I wanted to understand the actual problem or piece of code, but just to prompt out a solution. Often I was eagerly waiting for the auto-complete feature to fix my code. I copied pieces of code to LLM chats in the browser and then updated the code in the editor. This workflow didn't feel right. This isn't coding.

<!--more-->

The obvious solution would be to to stop using these AI-tools and write code as I used to. On one side I really would like to do that, but on the other side, I feel pressured to use this technology. I believe in this technologly and the promise of efficiency, however, I do not like the products and services.

This is also what annoyed me about Codeium. Every prompt is funneled through their service. If they change the terms of agreement the only option is to agree. You should know how this story goes.

## Inspiration

So I was looking for different angles on this topic. I was not looking for alternatives. Alternatives often don't solve the actual problem. While reading Hacker News (HN) I got an inspiration. In comment section of [Yek: Serialize your code repo (or part of it) to feed into any LLM ](https://news.ycombinator.com/item?id=42753302) I stumbled this:

> <https://github.com/simonw/files-to-prompt/> seems to work fine especially for Claude as it follows the recommendations and format.

In the HN sphere Simon Willison (guy in the picture) is well-known for his [blog](https://simonwillison.net/) and insights. He writes a column on how he uses LLMs to code and most importantly is he the author of the [llm](https://github.com/simonw/llm) command line interface (cli).

The link in the command shows an example on how Simon feeds multiple files into an LLM using his cli. The llm cli allows he to switch between remote models and even use local LLMs! There is a well maintained plugin directory to attach any kind of models.
## Files to prompt

So I started to introduce the files-to-prompt solution into my development workflow. My worklow is heavily based on [taskfile.build](https://taskfile.build/). For every project there is a task script file to run selected commands.

For the [Odoo.Build](https://odoo.build/) project I added a new command:

```bash
printf "| %-${cmd_width}s | %-${opt_width}s | %-${desc_width}s |\n" "llm-update" "[path]" "Feed module files with prompt to LLM and apply updates with git path."
```

And here is the function that does the magic:

```bash
function llm-update() {
    if test -z "$1"; then 
        echo "\$1 is empty."; 
        exit 1; 
    fi

    # Get list of files
    FILES=$(find "$1" -type f \( -name "*.py" -o -name "*.xml" \))

    echo -e "Loaded these files into prompt:\n\n$FILES\n"

    # Get task description
    read -p "Enter the task description: " TASK_DESCRIPTION

    # Prepare the files content for prompt
    FILE_CONTENTS=""
    for file in $FILES; do
        FILE_CONTENTS+="<<< $file >>>
$(cat "$file")

"
    done

    # Define the prompt
    PROMPT="EOF
Look at the code files below and do the following:

$TASK_DESCRIPTION

Output your response in the format of a git patch file. The patch should include only the changes to the files, and it should be applicable with 'git apply'. Do not include any explanations or extra information outside of the patch format.

Here are the files:

$FILE_CONTENTS
EOF"

    # Run the llm command with the prompt
    echo -e "\nWait for the response of the $LLM_MODEL llm."
    RESULT=$(llm -m "$LLM_MODEL" "$PROMPT")

    # Check if the result is empty
    if test -z "$RESULT"; then
        echo "No response from the model. Exiting."
        exit 1
    fi

    # Save the patch to a file
    PATCH_FILE="tmp/llm_changes.patch"
    echo "$RESULT" > "$PATCH_FILE"
    echo -e "\nSaved patch to $PATCH_FILE.\n"

    # Preview patch file and ask to apply
    git apply --check "$PATCH_FILE"
    less "$PATCH_FILE"
    read -p "Do you want to apply this patch? (y/n): " CONFIRM
    if [[ "$CONFIRM" =~ ^[Yy]$ ]]; then
        git apply "$PATCH_FILE"
    fi
}
```

Executing the function does:

* Collect all Python and XML files from the provided path.
* Ask for task instructions.
* Generate a prompt with the instruction and files that asks for a response in the style of a git patch.
* Pass the prompt to the `llm` command.
* Store the response as git patch file.
* Show a preview of the file with `less`.
* Ask if the patch should be applied.
* Apply the patch on confirmation.
## In Action

Let me show the function in action. In this scenario I have an Odoo module `hr_employee_user_acl` and would like to rename a definition.

First I run the `task` file with the command `llm-update` and give the path to the module as parameter:

```bash
task llm-update addons/hr/hr_employee_user_acl/
Loaded these files into prompt:

addons/hr/hr_employee_user_acl/__manifest__.py
addons/hr/hr_employee_user_acl/security/security.xml
addons/hr/hr_employee_user_acl/views/menu.xml
```

The function asks for a task description:

```bash
Enter the task description: rename restricted to restrict
```

And then it waits for the response.

```bash
Wait for the response of the deepseek-coder llm.
```

Once its done, a preview of the patch is shown:

```bash
```patch
diff --git a/addons/hr/hr_employee_user_acl/__manifest__.py b/addons/hr/hr_employee_user_acl/__manifest__.py
index 1234567..89abcde 100644
--- a/addons/hr/hr_employee_user_acl/__manifest__.py
+++ b/addons/hr/hr_employee_user_acl/__manifest__.py
@@ -1,7 +1,7 @@
 {
     "name": "HR Employee User ACL",
     "summary": """
-        Restricted access to employees app.
+        Restrict access to employees app.
     """,
     "author": "Mint System GmbH, Odoo Community Association (OCA)",
     "website": "https://www.mint-system.ch",

tmp/llm_changes.patch (END)
```

Quit the less preview with `q` and confirm the change.

```bash
Saved patch to tmp/llm_changes.patch.

Do you want to apply this patch? (y/n): y
Applied patch file.
```

That's it.

I will most likely add new commands and other prompts.
