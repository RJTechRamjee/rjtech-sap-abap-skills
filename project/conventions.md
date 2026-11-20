# Project conventions

These rules apply to every task and override names or examples in individual skills.
Machine-readable settings (allowed prefixes, modules, applications, packages, systems) are in
`project/project.yaml`.

## Platform

- New development is ABAP Cloud: released APIs only, no direct access to SAP tables, RAP with
  `strict ( 2 )`, CDS view entities (never `define view`), OData V4.
- Classic objects (reports, function modules, IDocs, classic DDIC) only where the system or
  package uses the Standard ABAP language version, for example when extending existing
  on-premise code. Say so explicitly when you choose classic over ABAP Cloud.

## Naming: `ZRJ_<MOD>_<SUB>_<TYPE>_<NAME>`

Every repository object starts with `ZRJ_`, then the **module** and **submodule** code, then an
object-type code, then the name:

```
ZRJ_SD_BIL_CL_INVOICE_SPLIT
│   │  │   │  └─ <NAME>: what it is, English
│   │  │   └──── <TYPE>: object type (CL = class); some objects have none (tables, tcodes)
│   │  └──────── <SUB>:  submodule (BIL = Billing)
│   └─────────── <MOD>:  module (SD = Sales and Distribution)
└─────────────── project prefix
```

- Module and submodule codes come only from `modules` in `project/project.yaml`. Take them from
  the application the object belongs to (`applications`). If it's unclear, ask the user. If a
  code is missing, ask the user to add it to `project.yaml`; never invent one.
- Code shared by several modules uses `CA_GEN`; demo and training code uses `CA_DMO`.
- Never use `ZCL_RJ_`, `ZIF_RJ_` or `ZRJ_<TYPE>_` without module and submodule.
- `ZRJ_SD_BIL_` already uses 11 of the 30 characters most names allow (16 for tables), so keep
  `<NAME>` short and abbreviate consistently. The topics give the room left per object type.
- A few objects need a different start because SAP requires it (lock objects `EZRJ_`,
  function modules `Z_RJ_`, IDoc segments `Z1RJ_`) or because they're too short (authorization
  and number range objects, CMOD projects): see `global-objects`.
- Skill templates show names like `ZRJ_R_Travel`: insert `<MOD>_<SUB>_` after `ZRJ_` when you
  use them (`ZRJ_CA_DMO_R_Travel`).

The detailed rules are split into naming topics. **Load the matching topic before you name
anything** (`get_project_conventions` with `topic`):

| Topic | Covers |
|---|---|
| `global-objects` | Classes, interfaces, exception classes, DDIC (tables, structures, data elements, domains, table types), reports, transactions, function groups and modules, IDocs, message classes, enhancements, authorization objects |
| `rap-cds` | CDS view entities (I_/R_/C_ layers), fields and associations, behavior definitions and pools, actions, determinations, validations, services, value helps, draft tables |
| `local-definitions` | Local classes, variables, constants, field symbols, internal tables, types, method and function module parameters, selection screens, test classes |

Most-used names at a glance:

| Object | Pattern | Example |
|---|---|---|
| Class / interface | `ZRJ_<MOD>_<SUB>_CL_<NAME>` / `_IF_<NAME>` | `ZRJ_SD_BIL_CL_INVOICE_SPLIT` |
| Database table / draft table | `ZRJ_<MOD>_<SUB>_<NAME>` / `..._D` (max 16) | `ZRJ_SD_SLS_ORD`, `ZRJ_SD_SLS_ORD_D` |
| CDS interface / BO / projection view | `ZRJ_<MOD>_<SUB>_I_` / `_R_` / `_C_<Name>` | `ZRJ_SD_SLS_R_SalesOrder` |
| Behavior pool | `ZRJ_<MOD>_<SUB>_BP_R_<NAME>` | `ZRJ_SD_SLS_BP_R_SALESORDER` |
| Local variable / structure / table | `lv_` / `ls_` / `lt_<name>` | `lt_bookings` |
| Importing / exporting / returning | `iv_`, `is_`, `it_` / `ev_`, `es_`, `et_` / `rv_`, `rs_`, `rt_` | `iv_travel_id` |

## Packages and transports

- Package hierarchy: `ZRJ_<MOD>` (structure package) > `ZRJ_<MOD>_<SUB>` > optional
  `ZRJ_<MOD>_<SUB>_<APP>` per application (see `applications` in `project/project.yaml`).
  Ask the user if the application is unclear.
- An object's module and submodule match its package's.
- `$TMP` (or another `$` package) only for experiments that will not be transported.
- Transportable packages need a transport request. Always ask the user for it; never guess one.

## Working with the user

- Confirm object names, package and transport before creating anything in the SAP system.
- After creating objects, list what was created and whether activation succeeded.
