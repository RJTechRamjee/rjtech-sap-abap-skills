---
description: Names inside ABAP code - local classes and interfaces, test classes, variables, constants, field symbols, data references, internal tables, structures, local types, class attributes, methods, method and function module parameters (importing, exporting, changing, returning), exceptions, selection screens. Load before writing or reviewing ABAP source code.
---

# Local definitions

General rules:

- Names are lower case in code, English, words separated by `_`, and say what the thing holds
  (`lt_open_bookings`, not `lt_data` or `itab`).
- The prefix shows **scope** (first letter) and **kind** (second letter). The table below is
  the full system; everything else in this file is built from it.
- Max 30 characters per name. Avoid abbreviations except common ones (`id`, `uuid`, `qty`, `amt`).
- Inline declarations follow the same prefixes: `DATA(lt_travels) = ...`,
  `LOOP AT lt_travels INTO DATA(ls_travel)`, `FIELD-SYMBOL(<ls_travel>)`.
- RAP handler methods keep the framework's parameter names (`keys`, `result`, `failed`, ...); see `rap-cds`.

## Prefix system

Scope (first letter):

| Letter | Scope |
|---|---|
| `l` | Local: declared inside a method, function module or form |
| `g` | Global in a program (report `TOP` include, function group `TOP`) or static class attribute (`CLASS-DATA`) |
| `m` | Instance attribute of a class (`DATA` in a class) |
| `i` / `e` / `c` / `r` | Parameter: importing / exporting / changing / returning |

Kind (second letter):

| Letter | Kind | Example |
|---|---|---|
| `v` | Elementary value (char, numeric, date, string, `abap_bool`) | `lv_travel_id` |
| `s` | Structure | `ls_travel` |
| `t` | Internal table | `lt_bookings` |
| `o` | Object reference | `lo_calculator` |
| `r` | Data reference (`REF TO data` or typed) | `lr_travel` |
| `x` | Exception object | `lx_error` |
| `c` | Constant (scope letter + `c`) | `lc_max_rows`, `gc_status_open` |

Combined:

| | Elementary | Structure | Table | Object | Data ref | Constant |
|---|---|---|---|---|---|---|
| Local | `lv_` | `ls_` | `lt_` | `lo_` | `lr_` | `lc_` |
| Program global / static attribute | `gv_` | `gs_` | `gt_` | `go_` | `gr_` | `gc_` |
| Instance attribute | `mv_` | `ms_` | `mt_` | `mo_` | `mr_` | - |
| Importing | `iv_` | `is_` | `it_` | `io_` | `ir_` | - |
| Exporting | `ev_` | `es_` | `et_` | `eo_` | `er_` | - |
| Changing | `cv_` | `cs_` | `ct_` | `co_` | `cr_` | - |
| Returning | `rv_` | `rs_` | `rt_` | `ro_` | `rr_` | - |

## Field symbols and loops

| What | Pattern | Example |
|---|---|---|
| Field symbol | `<l?_name>` using the kind letter | `<ls_travel>`, `<lv_amount>`, `<lt_items>` |
| Generic field symbol | `<lg_name>` (`TYPE any`) | `<lg_component>` |
| Loop work area | `ls_<singular of table>` | `LOOP AT lt_travels INTO DATA(ls_travel)` |
| Loop index | `lv_index`, `lv_tabix` | |
| Group key (`LOOP AT GROUP`) | `ls_group_<key>` | `ls_group_customer` |

## Constants

- Local constants `lc_`, program constants `gc_`.
- Class and interface constants are `c_` (they are always used with the class name:
  `zrj_sd_sls_cl_order=>c_status_open`, `zrj_sd_sls_if_status=>c_open`).
- Group related values in one constant structure:
  `CONSTANTS: BEGIN OF c_status, open TYPE c LENGTH 1 VALUE 'O', accepted ... END OF c_status.`
  Used as `c_status-open`.
- Prefer an `ENUM` (`ty_e_<name>`) when the values have no external meaning.

## Types

| What | Pattern | Example |
|---|---|---|
| Elementary type | `ty_<name>` | `ty_percent` |
| Structure type | `ty_s_<name>` | `ty_s_travel` |
| Table type | `ty_t_<name>` | `ty_t_travels` |
| Range table type | `ty_r_<name>` | `ty_r_travel_id` |
| Enumeration | `ty_e_<name>` | `ty_e_status` |
| Reference type | `ty_ref_<name>` | `ty_ref_travel` |

- The same patterns apply to types in a class or interface (`zrj_sd_prc_if_discount=>ty_s_result`).
- A type used by more than one class goes into an interface or a DDIC type (`ZRJ_<MOD>_<SUB>_S_` /
  `_TT_`, see `global-objects`), not into each class.
- Ranges: `lt_r_travel_id TYPE RANGE OF ...`; selection screen select-options are `s_` (below).

## Local classes and interfaces

| What | Pattern | Example |
|---|---|---|
| Local class | `lcl_<name>` | `lcl_app`, `lcl_file_reader` |
| Local interface | `lif_<name>` | `lif_reader` |
| Local exception class | `lcx_<name>` | `lcx_invalid_input` |
| Local test class | `ltcl_<class under test or feature>` | `ltcl_discount_calculator` |
| Local test double | `ltd_<name>` | `ltd_travel_repository` |
| Local test helper / data builder | `lth_<name>` | `lth_travel_builder` |
| RAP handler / saver | `lhc_<alias>` / `lsc_<root>` | see `rap-cds` |
| Local event handler class | `lcl_<event source>_handler` | `lcl_alv_handler` |

## Methods

| Kind | Pattern | Example |
|---|---|---|
| Getter / setter | `get_<thing>` / `set_<thing>` | `get_total_price` |
| Boolean check | `is_<state>` / `has_<thing>` / `can_<action>`, returning `rv_result TYPE abap_bool` | `is_valid`, `has_bookings` |
| Check that raises | `check_<thing>` (raises an exception, no return value) | `check_dates` |
| Action | `<verb>_<object>` | `calculate_discount`, `read_bookings`, `send_mail` |
| Factory | `create` / `create_<variant>` / `get_instance` | `create_for_country` |
| Conversion | `to_<target>` / `from_<source>` | `to_json` |
| Event handler | `on_<event>` | `on_double_click` |
| Test method | What is expected, in words | `discount_for_de_is_ten_percent` |
| Test setup | `setup`, `teardown`, `class_setup`, `class_teardown` (ABAP Unit names) | |

- Use a `RETURNING` parameter when a method has one result; name it `r<kind>_<name>` or simply
  `rv_result` / `rt_result` / `ro_result` if the method name already says what it is.
- Pass several results as one structure (`rs_`) rather than several `EXPORTING` parameters.
- Errors are class-based exceptions (`RAISING zrj_<mod>_<sub>_cx_...`), not `EXCEPTIONS` or return codes.

## Method and function module parameters

| Direction | Elementary | Structure | Table | Object | Example |
|---|---|---|---|---|---|
| `IMPORTING` | `iv_` | `is_` | `it_` | `io_` | `iv_travel_id`, `it_keys` |
| `EXPORTING` | `ev_` | `es_` | `et_` | `eo_` | `et_messages` |
| `CHANGING` | `cv_` | `cs_` | `ct_` | `co_` | `cs_travel` |
| `RETURNING` | `rv_` | `rs_` | `rt_` | `ro_` | `rv_total` |
| `TABLES` (function modules, legacy only) | - | - | `t_` | - | `t_items` |

- Optional parameters have a sensible `DEFAULT`; boolean flags are `iv_<flag>` of `abap_bool`
  (`iv_test_mode`), never `iv_flag`.
- Function modules use the same prefixes. New function modules have no `TABLES` parameters and
  no classic `EXCEPTIONS`, except where SAP requires them (RFC, BAPI: `return` table of type
  `BAPIRET2_T`, named `et_return`).

## Selection screens (reports)

Selection screen names have a hard limit of 8 characters.

| Element | Pattern | Example |
|---|---|---|
| Parameter | `p_<name>` | `p_date` |
| Select-option | `s_<name>` | `s_trvid` |
| Checkbox | `cb_<name>` | `cb_test` |
| Radio button / group | `rb_<name>` / `rg_<name>` | `rb_all` |
| Block | `b<nn>` | `b01` |
| Selection screen text | Set in the text elements, never hard-coded | |

- Read the selection screen into a structure (`ls_params`) once and pass it to the class that
  does the work; don't read `p_` / `s_` fields inside classes.

## Messages and texts

- Messages come from a message class (`MESSAGE e001(zrj_sd_bil)`), never hard-coded text.
- Text symbols `TEXT-<nnn>` for literal texts in reports.
