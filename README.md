# rjtech-sap-abap-skills

The team's SAP ABAP Agent Skills and conventions. The MCP server
[rjtech-mcp-sap-abap](https://github.com/RJTechRamjee/rjtech-mcp-sap-abap) reads this repo
directly from GitHub. Within about 5 minutes of a merge to `main`, every developer's Copilot
has the change. Nobody needs to pull anything.

- **New to the team?** Set up the server once: [TEAM-SETUP.md](https://github.com/RJTechRamjee/rjtech-mcp-sap-abap/blob/main/TEAM-SETUP.md)
- **Adding or changing a skill?** See [CONTRIBUTING.md](CONTRIBUTING.md)

## Layout

```
skills/<skill-name>/SKILL.md          instructions (frontmatter: name, description)
skills/<skill-name>/references/*.md   long templates, loaded only when needed
team/conventions.md                   naming and working rules, always given to the model
team/team.yaml                        prefixes for the ADT guard, projects, packages, systems
```

Only text files (`.md .txt .abap .cds .json .yaml .yml .xml`) under `skills/` and `team/` are served.
Never put passwords, tokens or customer data in this repo: it is public.

## Skills

| Skill | Use it for |
|---|---|
| `rap-managed-bo` | Managed RAP business object with draft (tables, views, BDEF, behavior pool) |
| `cds-view-entities` | CDS view entities, associations, access control |
| `rap-odata-v4-service` | Service definition/binding, Fiori UI annotations |
| `abap-unit-rap` | ABAP Unit for RAP, CDS and SQL test doubles |
| `enhancement-spot-badi` | Enhancement spots and BAdIs in ABAP Cloud |
