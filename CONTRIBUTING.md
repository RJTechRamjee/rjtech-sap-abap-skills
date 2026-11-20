# Adding or changing a skill

Everything merged to `main` goes live for the whole team within about 5 minutes, so changes go
through a pull request and an automatic check.

## 1. Create the skill

```
skills/my-new-skill/
├── SKILL.md
└── references/          (optional: long templates)
    └── templates.md
```

`SKILL.md` starts with frontmatter:

```markdown
---
name: my-new-skill
description: Creates X in ABAP Cloud. Use when the user asks to build, change or test X.
---

# Steps
1. ...
```

Rules:

- The folder name uses lowercase letters, digits and hyphens, and `name` matches it exactly.
- `description` is what Copilot uses to pick the skill, so say **what** it does and **when** to use it.
- Keep `SKILL.md` about workflow and rules. Move long code to `references/` and mention the file in `SKILL.md`.
- Follow `project/conventions.md` and the naming topics in `project/naming/` (`ZRJ_<MOD>_<SUB>_<TYPE>_<NAME>`; new module or submodule codes go into `modules` in `project/project.yaml` first). Point to other skills by name (for example "see the `rap-managed-bo` skill"), and don't rename existing skills.
- Target ABAP Cloud: released APIs, RAP `strict ( 2 )`, CDS view entities, OData V4.

## 2. Test it locally before pushing (optional)

Point your MCP server at this checkout instead of GitHub. In `%USERPROFILE%\.rjtech-mcp\config.env`, add:

```
LOCAL_CONTENT_DIR="C:\path\to\rjtech-sap-abap-skills"
```

Restart the MCP server in your IDE. Edits now show up within about 10 seconds. Remove the line when you are done, or you will stop getting the team's updates.

Check the repo the same way the pull request check does:

```powershell
npx -y https://github.com/RJTechRamjee/rjtech-mcp-sap-abap/releases/latest/download/rjtech-mcp-sap-abap.tgz validate .
```

## 3. Open a pull request

Push a branch and open a pull request. The **Validate skills** check must pass, and
someone should review the content. After the merge, everyone has it within about 5 minutes.
In Copilot, the tool `skills_status` (with `refresh: true`) shows which version is loaded.

## Changing project conventions

- Core rules for the model (platform, prefix, packages, how to work): `project/conventions.md`.
  It goes into every session, so keep it short and put details into a naming topic.
- Detailed naming rules: `project/naming/<topic>.md`. Each file needs a frontmatter
  `description` that says what it covers and when to load it; the model sees only that list
  until it loads a topic. A new file becomes a new topic; mention it in the topic table in
  `project/conventions.md`.
- Allowed prefixes for the ADT write tools, applications, packages, systems: `project/project.yaml`.
  The validate check rejects an invalid `project.yaml`.
