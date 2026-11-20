---
description: Names of global repository objects - classes, interfaces, exception classes, DDIC (tables, structures, data elements, domains, table types, search helps, lock objects), reports, transactions, function groups and modules, IDocs, message classes, enhancements, authorization and number range objects, CMOD projects. Load before creating or naming any global object outside RAP/CDS.
---

# Global objects

General rules:

- Pattern `ZRJ_<MOD>_<SUB>_<TYPE>_<NAME>`. `<MOD>` and `<SUB>` are codes from `modules` in
  `project/project.yaml` (here `SD` / `BIL`). `<TYPE>` is the code in the tables below. `<NAME>`
  is English, upper case, words separated by `_`.
- **Room for `<NAME>`** in the tables assumes a 2-letter module and a 3-letter submodule
  (`ZRJ_SD_BIL_` = 11 characters). With a 2-letter submodule there's one more character.
- Name the business thing, not the technique: `ZRJ_SD_BIL_CL_INVOICE_SPLIT`, not `ZRJ_SD_BIL_CL_HELPER`.
- Use the same `<NAME>` across related objects of one feature (`ZRJ_SD_BIL_INV` table,
  `ZRJ_SD_BIL_S_INVOICE` structure, `ZRJ_SD_BIL_TT_INVOICE` table type).
- Abbreviate consistently (`INV` invoice, `ORD` order, `CUST` customer, `MAT` material) instead of
  cutting names off. Put the full meaning in the object's description.
- Tables, structures, table types and data elements share one DDIC namespace, which is why
  each one has its own type code.

## Classes and interfaces

| Object | Pattern | Max | Room for `<NAME>` | Example |
|---|---|---|---|---|
| Class | `ZRJ_<MOD>_<SUB>_CL_<NAME>` | 30 | 16 | `ZRJ_SD_BIL_CL_INVOICE_SPLIT` |
| Abstract base class | `ZRJ_<MOD>_<SUB>_CL_<NAME>_BASE` | 30 | 11 | `ZRJ_SD_PRC_CL_DISCOUNT_BASE` |
| Factory class | `ZRJ_<MOD>_<SUB>_CL_<NAME>_FACT` | 30 | 11 | `ZRJ_SD_PRC_CL_DISCOUNT_FACT` |
| Interface | `ZRJ_<MOD>_<SUB>_IF_<NAME>` | 30 | 16 | `ZRJ_SD_PRC_IF_DISCOUNT` |
| Exception class | `ZRJ_<MOD>_<SUB>_CX_<NAME>` | 30 | 16 | `ZRJ_SD_SLS_CX_CREDIT_BLOCK` |
| Unit test data class (global, `FOR TESTING`) | `ZRJ_<MOD>_<SUB>_CL_<NAME>_TD` | 30 | 13 | `ZRJ_SD_BIL_CL_INVOICE_TD` |

- Exception classes inherit from `CX_STATIC_CHECK` (caller must handle), `CX_DYNAMIC_CHECK`
  (caller can prevent it) or `CX_NO_CHECK` (should never happen), and use a message class for texts.
- Interface components are not prefixed with the interface name; callers use `zrj_..._if_x~method`.

## Dictionary (DDIC)

| Object | Pattern | Max | Room for `<NAME>` | Example |
|---|---|---|---|---|
| Database table | `ZRJ_<MOD>_<SUB>_<NAME>` | 16 | 5 | `ZRJ_SD_BIL_INV` |
| Draft table (RAP) | `<TABLE>_D` | 16 | 3 | `ZRJ_SD_BIL_INV_D` |
| Customizing / text table | `<TABLE>_C` / `<TABLE>_T` | 16 | 3 | `ZRJ_SD_BIL_TYP_C` |
| Structure | `ZRJ_<MOD>_<SUB>_S_<NAME>` | 30 | 17 | `ZRJ_SD_BIL_S_INVOICE` |
| Table type | `ZRJ_<MOD>_<SUB>_TT_<NAME>` | 30 | 16 | `ZRJ_SD_BIL_TT_INVOICE` |
| Range table type | `ZRJ_<MOD>_<SUB>_RT_<NAME>` | 30 | 16 | `ZRJ_SD_BIL_RT_INVOICE_ID` |
| Data element | `ZRJ_<MOD>_<SUB>_DE_<NAME>` | 30 | 16 | `ZRJ_SD_BIL_DE_SPLIT_REASON` |
| Domain | `ZRJ_<MOD>_<SUB>_DO_<NAME>` | 30 | 16 | `ZRJ_SD_BIL_DO_SPLIT_REASON` |
| Search help (elementary / collective) | `ZRJ_<MOD>_<SUB>_SH_<NAME>` / `_SHC_<NAME>` | 30 | 16 / 15 | `ZRJ_SD_BIL_SH_INVOICE` |
| Lock object | `E<TABLE>` (SAP requires `EZ`) | 16 | table name max 15 | `EZRJ_SD_BIL_INV` |
| Table maintenance function group | `ZRJ_<MOD>_<SUB>_TM_<NAME>` | 26 | 12 | `ZRJ_SD_BIL_TM_TYPES` |

- Table names are short. Use 3 characters for a table that gets a draft table, text table or
  customizing table, so `_D`, `_T` or `_C` still fits.
- The first key field is `client` (`abap.clnt`). Field names are lower case snake_case,
  max 16 characters (`invoice_id`, `split_reason`). They don't carry module codes.
- Reuse SAP data elements where they fit. Create your own data element when you need your own
  labels or domain values. Domains with fixed values: one domain per value list.

## Programs and transactions

| Object | Pattern | Max | Room for `<NAME>` | Example |
|---|---|---|---|---|
| Executable report | `ZRJ_<MOD>_<SUB>_RP_<NAME>` | 30 | 16 | `ZRJ_FI_AR_RP_DUNNING_CHECK` |
| Include | `<REPORT>_<SUFFIX>`: `_TOP` (data), `_SEL` (selection screen), `_CL` (local classes), `_F01` (forms, legacy only) | 40 | | `ZRJ_FI_AR_RP_DUNNING_CHECK_CL` |
| Module pool | `SAPMZRJ_<MOD>_<SUB>_<NAME>` | 30 | 15 | `SAPMZRJ_SD_SHP_LOADING` |
| Transaction code | `ZRJ_<MOD>_<SUB>_<NAME>` | 20 | 9 | `ZRJ_SD_SHP_LOAD` |
| Report variant | `<NAME>` (belongs to the report, no prefix needed) | 14 | 14 | `DAILY_DE` |
| Background job | `<REPORT>_<VARIANT>` | 32 | | `ZRJ_FI_AR_RP_DUNNING_CHECK_DAILY` |

- New logic goes into local or global classes; the report only reads the selection screen and
  calls `NEW lcl_app( )->run( )`. No new `FORM` routines.

## Function groups and function modules

| Object | Pattern | Max | Room for `<NAME>` | Example |
|---|---|---|---|---|
| Function group | `ZRJ_<MOD>_<SUB>_FG_<NAME>` | 26 | 12 | `ZRJ_SD_BIL_FG_INVOICE` |
| Function module (SAP requires `Z_`) | `Z_RJ_<MOD>_<SUB>_<NAME>` | 30 | 18 | `Z_RJ_SD_BIL_INVOICE_GET` |
| RFC-enabled function module | `Z_RJ_<MOD>_<SUB>_RFC_<NAME>` | 30 | 14 | `Z_RJ_SD_BIL_RFC_INVOICE_GET` |
| Customer BAPI | `Z_RJ_<MOD>_<SUB>_BAPI_<NAME>` | 30 | 13 | `Z_RJ_SD_BIL_BAPI_INV_CREATE` |
| Update function module | `Z_RJ_<MOD>_<SUB>_UPD_<NAME>` | 30 | 14 | `Z_RJ_SD_BIL_UPD_INV_SAVE` |

- Name function modules `<object>_<verb>` so a function group's modules sort together.
- Use function modules only where a class can't do the job: RFC, update task, BAPI,
  classic exits. Keep the logic in a class and let the function module delegate to it.
- Parameters follow the prefixes in `local-definitions`. No `TABLES` parameters in new code.

## IDocs

| Object | Pattern | Max | Room for `<NAME>` | Example |
|---|---|---|---|---|
| Message type (WE81) | `ZRJ_<MOD>_<SUB>_<NAME>` | 30 | 19 | `ZRJ_SD_BIL_INVOICE` |
| Basic type (WE30) | `ZRJ_<MOD>_<SUB>_<NAME><NN>` (version number) | 30 | 17 | `ZRJ_SD_BIL_INVOICE01` |
| Extension of an SAP basic type | `ZRJ_<MOD>_<SUB>_<BASIC TYPE>_X<NN>` | 30 | 15 | `ZRJ_SD_SLS_ORDERS05_X01` |
| Segment type (WE31, SAP requires `Z1`) | `Z1RJ_<MOD>_<SUB>_<NAME>` | 27 | 15 | `Z1RJ_SD_BIL_INV_HEAD` |
| Inbound / outbound process code | `Z` + 3 characters, no module (too short) | 4 | | `ZBIN` |
| Inbound function module | `Z_RJ_<MOD>_<SUB>_IDOC_IN_<NAME>` | 30 | 10 | `Z_RJ_SD_BIL_IDOC_IN_INVOICE` |
| Outbound function module | `Z_RJ_<MOD>_<SUB>_IDOC_OUT_<NAME>` | 30 | 9 | `Z_RJ_SD_BIL_IDOC_OUT_INVOIC` |

- Segment names say what the level holds: `_HEAD`, `_ITEM`, `_PARTNER`, `_TEXT`.
- Process codes can't carry the module, so list each one with its message type in the
  application's documentation.

## Messages, enhancements, authorizations

| Object | Pattern | Max | Room for `<NAME>` | Example |
|---|---|---|---|---|
| Message class (one per submodule) | `ZRJ_<MOD>_<SUB>` | 20 | | `ZRJ_SD_BIL` |
| Message class (own one for a large feature) | `ZRJ_<MOD>_<SUB>_<NAME>` | 20 | 9 | `ZRJ_SD_BIL_SPLIT` |
| Enhancement spot / BAdI definition | `ZRJ_<MOD>_<SUB>_ES_<NAME>` / `_BADI_<NAME>` | 30 | 16 / 14 | `ZRJ_SD_PRC_ES_DISCOUNT` |
| Enhancement / BAdI implementation | `ZRJ_<MOD>_<SUB>_EI_<NAME>` / `_BI_<NAME>` (variant at the end) | 30 | 16 | `ZRJ_SD_PRC_BI_DISCOUNT_DE` |
| BAdI implementing class | `ZRJ_<MOD>_<SUB>_CL_<NAME>` (same `<NAME>` as the implementation) | 30 | 16 | `ZRJ_SD_PRC_CL_DISCOUNT_DE` |
| Implicit enhancement implementation | `ZRJ_<MOD>_<SUB>_EI_<SAP OBJECT>_<PURPOSE>` | 30 | 16 | `ZRJ_SD_SLS_EI_MV45A_PRICING` |
| Application log object / subobject | `ZRJ_<MOD>_<SUB>` / `<NAME>` | 20 | | `ZRJ_SD_BIL` / `SPLIT` |

Objects with a 10- or 8-character limit use the **compact code**: `ZRJ<MOD><SUB>` without
underscores (`ZRJSDBIL`, 8 characters with a 3-letter submodule).

| Object | Pattern | Max | Example |
|---|---|---|---|
| Authorization object | `ZRJ<MOD><SUB><X>` (1-2 characters for the object) | 10 | `ZRJSDBILIV` |
| Authorization field | Reuse SAP fields (`BUKRS`, `VKORG`, `ACTVT`); own fields like authorization objects | 10 | |
| Number range object | `ZRJ<MOD><SUB><X>` | 10 | `ZRJSDBILIN` |
| CMOD project (classic user exits) | `ZRJ<MOD><SUB>`, one per submodule | 8 | `ZRJSDBIL` |

- Message numbers: `000`-`099` general, then one block of 100 per class or feature that uses
  the message class. Texts are full sentences with `&1` to `&4` placeholders.
