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
I used VSCodium and Codeium to develop Python code. VSCode is my editor of choice and Codeium is a well integrated AI-tool that helps writing code. While it solved a lot of problems for me, especially writing boilerplate code, I became more and more frustrated using this setup.

No longer I wanted to understand the actual problem or piece of code, but just to prompt out a solution. Often I was eagerly waiting for the auto-complete feature to fix my code. I copied pieces of code to LLM chats in the browser and then updated the code in the editor. This workflow didn't feel right. This isn't coding.

<!--more-->

The obvious solution would be to to stop using these AI-tools and write code as I used to. On one side I really would like to do that, but on the other side, I feel pressured to use this technology. I believe in the LLM technology and the promise of more efficient coding. However, I do not like the products and services built around LLMs.

This is also what annoyed me about Codeium. Every prompt is funneled through their service. If they change the terms of agreement the only option is to agree. Coding should not be on a providers agreement. Especially if the terms are to their favor.
## Inspiration

So I was looking for different angles on this topic. I was not looking for alternatives. Alternatives often don't solve the actual problem. While reading Hacker News (HN) I got an inspiration. In comment section of [Yek: Serialize your code repo (or part of it) to feed into any LLM ](https://news.ycombinator.com/item?id=42753302) I stumbled this:

> <https://github.com/simonw/files-to-prompt/> seems to work fine especially for Claude as it follows the recommendations and format.

In the HN sphere Simon Willison (guy in the picture) is well-known for his [blog](https://simonwillison.net/) and insights. He writes a column on how he uses LLMs to code and most importantly is he the author of the [llm](https://github.com/simonw/llm) command line interface (cli).

The link in the command shows an example on how Simon feeds multiple files into an LLM using his cli. The llm cli allows he to switch between remote models and even use local LLMs! There is a well maintained plugin directory to attach any kind of models.
## Files to prompt

So I started to introduce the files-to-prompt solution into my development workflow. My worklow is heavily based on [taskfile.build](https://taskfile.build/). For every project there is a task script file to run selected commands.

For the [Odoo.Build](https://odoo.build/) project I added a new command:

```bash
printf "| %-${cmd_width}s | %-${opt_width}s | %-${desc_width}s |\n" "update-with-llm" "[path][prompt]" "Update files with LLM prompt."
```

And here is the function that does the magic:

```bash
function update-with-llm() {
    if test -z "$1"; then 
        echo "\$1 is empty."; 
        exit 1; 
    fi

    # Get files from path
    if [ -f "$1" ]; then
        FILES="$1"
    elif [ -d "$1" ]; then
        FILES=$(find "$1" -type f \( -name "*.md" -o -name "*.yml" -o -name "*.yaml" -o -name "*.conf" -o -name "*.nginx" \) \
            | grep -vFf <(git -C "$1" ls-files --ignored --exclude-standard --others))
    else
        echo "$1 is neither a file nor a directory."
        exit 1
    fi
    echo -e "Loaded these files into prompt:\n\n$FILES\n"

    # Prompt task description
    if [ -z "$2" ]; then
        read -p "Enter the task description: " TASK_DESCRIPTION
    else
        TASK_DESCRIPTION="$2"
    fi

    # Prepare the files content for prompt
    FILE_CONTENTS=""
    for FILE in $FILES; do
        FILE_CONTENTS+="<<<$FILE>>>
$(cat "$FILE")

"
    done

    # Define prompt content
    mkdir -p tmp
    PROMPT_FILE="tmp/llm_update"
    echo -e "\nWrite prompt to $PROMPT_FILE."
    cat << EOF > "$PROMPT_FILE"
Look at the code files below and do the following:

$TASK_DESCRIPTION

Output all files that you need to change in full again, including your changes. 
In the same format as I provide the files below. Under no circumstances output any other text, 
no additional infos, no code formatting chars. Only the code in the given format.

Here are the files:

$FILE_CONTENTS
EOF

    # Run the llm command
    echo -e "Send prompt and wait for the response of the $LLM_MODEL LLM."
    RESULT=$(cat "$PROMPT_FILE" | llm -m "$LLM_MODEL")

    # Check if result is empty
    if [ -z "$RESULT" ]; then
        echo "No response from the model. Exiting."
        exit 1
    fi

    # Save the result to a file
    RESULT_FILE="tmp/llm_update_result"
    echo "$RESULT" > "$RESULT_FILE"
    echo -e "Saved response to $RESULT_FILE.\n"

    # Show a preview of result file
    less "$RESULT_FILE"

    # Ask for confirmation before updating files
    read -p "Do you want to apply these updates to the files? (y/n): " CONFIRM
    if [[ ! "$CONFIRM" =~ ^[Yy]$ ]]; then
        exit 0
    fi

    # Parse the response from the file and update the files directly
    echo -e "Parsing the response and updating files...\n"
    CURRENT_FILE=""
    while IFS= read -r LINE; do
        if [[ $LINE =~ ^"<<<"(.*)">>>" ]]; then
            CURRENT_FILE="${BASH_REMATCH[1]}"
            echo "Update file $CURRENT_FILE."
            > "$CURRENT_FILE"
        elif [[ -n $CURRENT_FILE ]]; then
            echo "$LINE" >> "$CURRENT_FILE"
        fi
    done < "$RESULT_FILE"
}
```

Executing the function does:

* Collect all Python and XML files from the provided path.
* Ask for task instructions.
* Generate a prompt with the instruction and files that asks for a response with file content only.
* Pass the prompt to the `llm` command.
* Store the response into a file.
* Show a preview of the file with `less`.
* Ask if the changes should be applied.
* Apply the changes on confirmation.

## In Action

Let me show the function in action. In this scenario I have an Odoo module `board_user_acl` and would like to rename a definition.

First I run the `task` file with the `update-with-llm` command and give the path to the module as parameter:

```bash
task update-with-llm addons/server_tools/board_user_acl "Rename restricted to restrict"
Loaded these files into prompt:

addons/server_tools/board_user_acl/__manifest__.py
addons/server_tools/board_user_acl/security/security.xml
addons/server_tools/board_user_acl/views/menu.xml
```

The function asks for a task description:

```bash
Enter the task description: Rename restricted to restrict
```

And then it waits for the response.

```bash
Write prompt to tmp/llm_update.
Send prompt and wait for the response of the gpt-4o LLM.
Saved response to tmp/llm_update_result.
```

Once its done, a preview of the updated files is shown:

```text
<<<addons/server_tools/board_user_acl/__manifest__.py>>>
{
    "name": "Board User ACL",
    "summary": """
        Restrict access to dashboards app.
    """,
    "author": "Mint System GmbH",
    "website": "https://www.mint-system.ch",
    "category": "Technical",
    "version": "16.0.1.0.0",
    "license": "AGPL-3",
    "depends": ["base_user_acl"],
    "data": ["security/security.xml", "views/menu.xml"],
    "installable": True,
    "application": False,
    "auto_install": False,
    "images": ["images/screen.png"],
}
tmp/llm_update_result (END)
```

Quit the less preview with `q` and confirm the change.

```bash
Parsing the response and updating files...

Update file addons/server_tools/board_user_acl/__manifest__.py.
```

That's it.

I will most likely add new commands and other prompts.

*Edits:*
* Removed prompt response with git patch file as it was often not working correctly.
* Renamed `llm-update` to `update-with-llm`
* Updated how files are searched
