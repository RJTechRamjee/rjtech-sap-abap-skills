# Team conventions

These rules apply to every task and override names or examples in individual skills.
Machine-readable settings (allowed prefixes, projects, packages) are in `team/team.yaml`.

## Platform

- ABAP Cloud only: released APIs, no classic statements (no `ENHANCEMENT-POINT`, no direct DB access to SAP tables).
- RAP with `strict ( 2 )`, CDS view entities (never `define view`), OData V4.

## Naming: `ZRJ_<TYPE>_<NAME>`

Every object starts with `ZRJ_`, followed by an object-type code. Never use `ZCL_RJ_` or `ZIF_RJ_`.

| Object | Pattern | Example |
|---|---|---|
| Class | `ZRJ_CL_<NAME>` | `ZRJ_CL_DISCOUNT_DE` |
| Interface | `ZRJ_IF_<NAME>` | `ZRJ_IF_DISCOUNT` |
| Database table / draft table | `ZRJ_<NAME>` / `ZRJ_<NAME>_D` (max 16 chars) | `ZRJ_TRAVEL`, `ZRJ_TRAVEL_D` |
| Interface (basic) view | `ZRJ_I_<Name>` | `ZRJ_I_TravelCustomer` |
| RAP BO (restricted) view | `ZRJ_R_<Name>` | `ZRJ_R_Travel` |
| Projection / consumption view | `ZRJ_C_<Name>` | `ZRJ_C_Travel` |
| Behavior pool | `ZRJ_BP_R_<NAME>` | `ZRJ_BP_R_TRAVEL` |
| Service definition (UI / API) | `ZRJ_UI_<NAME>` / `ZRJ_API_<NAME>` | `ZRJ_UI_TRAVEL` |
| Service binding | `<service definition>_O4` | `ZRJ_UI_TRAVEL_O4` |
| Enhancement spot / BAdI definition | `ZRJ_ES_<NAME>` / `ZRJ_BADI_<NAME>` | `ZRJ_ES_TRAVEL` |
| Enhancement / BAdI implementation | `ZRJ_EI_<NAME>_<VARIANT>` / `ZRJ_BI_<NAME>_<VARIANT>` | `ZRJ_BI_DISCOUNT_DE` |
| Message class | `ZRJ_<NAME>` | `ZRJ_TRAVEL` |

Class and interface names are at most 30 characters, including the prefix, so keep `<NAME>` short.

## Packages and transports

- Put objects in the package of their project (see `projects` in `team/team.yaml`). Ask the user if the project is unclear.
- `$TMP` (or another `$` package) only for experiments that will not be transported.
- Transportable packages need a transport request. Always ask the user for it; never guess one.

## Working with the user

- Confirm object names, package and transport before creating anything in the SAP system.
- After creating objects, list what was created and whether activation succeeded.
