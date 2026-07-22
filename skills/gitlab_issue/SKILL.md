---
name: gitlab_issue
description: Help creating or update issue on the gitlab issue board. The create a small buildable blocks.
metadata:
  last_modified: 22-07-2026
---

# GitLab Issue

Help the user create or update GitLab issues. The issue describes a feature/bug/improvement or action which helps with implementing it and pushes the project forward.

# Process
## 1. Gather context
Use the already available information presented by the user. Gather descriptions, specs and designs. Use information from already provided issues or existing issue information.

## 2. Inspect code base (Optional)
Inspect the code base to gather the current state of the project. Make sure there's a clear picture of the needed work that needs to be done. Issue titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

## 3. Quiz the user
Ask the user 1 question at a time to refine the issue.

Ask the user:
- About missing information
- About the correctness of dependencies

## 4. Issue ticket format

Use the following issue structure for writing/updating issue in GitLab:

```markdown
# Description

A clear description of the issue.

# Designs

Images or links to the designs needed for the implementation. (Optional)

# API Specs

Describes the needed endpoint and models in a simple markdown structure needed for the feature. (Optional)

# Acceptance criteria

Provide a bullet point list of acceptance criteria which provide validation basis for validating the implementation.

# Notes

Place remark and note here.
```

## 5. Publish the Issue
Use the GitLab MCP to publish the issue on the correct issue board for the correct project.
