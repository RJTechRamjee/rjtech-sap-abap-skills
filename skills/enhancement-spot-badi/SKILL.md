---
name: enhancement-spot-badi
description: Define your own extension point in ABAP - enhancement spot, BAdI definition (interface, filters, fallback class), the GET BADI / CALL BADI caller code, and a BAdI implementation to test it. Includes a short classic-only note on explicit enhancement points/sections. Use when the user asks to create an enhancement spot, BAdI definition, BAdI implementation, enhancement point or a custom extension/exit in their own code.
---

# Enhancement spot and BAdI definition

Makes your own code extensible: a caller asks a BAdI for an implementation at runtime, and other developers (or other software components) plug in their logic without changing the caller. In ABAP Cloud a BAdI is the **only** extension point you can define yourself. Full code templates are in `references/templates.md`; load it before generating code.

## Who creates what

- **Source objects** (BAdI interface, fallback class, implementing class, caller): if the `adt_*` tools are available, create them yourself with `adt_create_interface` / `adt_create_class`, fix activation errors with `adt_update_source`, and check existing code with `adt_read_source`. Without the tools, hand the code over for pasting.
- **Enhancement spot (ENHS) and enhancement implementation (ENHO)** are form-based objects with no tool yet. The user creates them in the Eclipse ADT wizard: give them a table with the exact value for every field (see "Wizard values" below) and wait for them to confirm before continuing with the steps that depend on them.

## Before you start

Ask (or infer from context) and confirm with the user:
1. **Name** (after the fixed prefix `ZRJ_<TYPE>_`), e.g. `DISCOUNT`.
2. **Package** and **transport request**. ABAP Cloud: package with ABAP language version *ABAP for Cloud Development*.
3. **What the extension decides or adds**: this defines the interface methods and their parameters.
4. **Single use or multiple use** (see rules).
5. **Filters**: do implementations apply only for certain values (country, company code, document type)? Each filter needs a name and a type.
6. **Fallback**: should there be default logic when nobody has implemented the BAdI? Default: yes for single use.

## Naming

Every object starts with the prefix `ZRJ_`, followed by an object-type code: `ZRJ_<TYPE>_<NAME>`.

| Object | Pattern | Example |
|---|---|---|
| Enhancement spot | `ZRJ_ES_<NAME>` | `ZRJ_ES_TRAVEL` |
| BAdI definition | `ZRJ_BADI_<NAME>` | `ZRJ_BADI_DISCOUNT` |
| BAdI interface | `ZRJ_IF_<NAME>` | `ZRJ_IF_DISCOUNT` |
| Fallback class | `ZRJ_CL_<NAME>_FALLBACK` | `ZRJ_CL_DISCOUNT_FALLBACK` |
| Enhancement implementation | `ZRJ_EI_<NAME>_<VARIANT>` | `ZRJ_EI_DISCOUNT_DE` |
| BAdI implementation | `ZRJ_BI_<NAME>_<VARIANT>` | `ZRJ_BI_DISCOUNT_DE` |
| Implementing class | `ZRJ_CL_<NAME>_<VARIANT>` | `ZRJ_CL_DISCOUNT_DE` |
| Other classes / interfaces (caller, provider) | `ZRJ_CL_<NAME>` / `ZRJ_IF_<NAME>` | `ZRJ_CL_DISCOUNT_PROVIDER` |

Class and interface names have a **maximum of 30 characters**, including the prefix, so keep `<NAME>` short (`ZRJ_CL_` + `_FALLBACK` already use 16).

One enhancement spot can hold several BAdI definitions that belong together (e.g. all extension points of one application).

## Build order

Activate after each step. Each step needs the previous objects to be active.

1. **BAdI interface**: a normal global interface with `INTERFACES if_badi_interface.` plus the methods. Without `if_badi_interface` the BAdI definition will not activate.
2. **Fallback class** (optional): implements the BAdI interface with the default logic.
3. **Enhancement spot + BAdI definition** (user, in the wizard): create the ENHS, add the BAdI definition, set usage, interface, fallback class and filters. Activate.
4. **Caller code**: `GET BADI` / `CALL BADI` where the extension should run. Needs the active BAdI, because the BAdI name is the reference type.
5. **Implementation for testing**: implementing class (code), then the enhancement implementation + BAdI implementation (user, in the wizard) with filter values. Activate.
6. **Test**: run the caller with and without an active implementation; ABAP Unit test as in the templates.

## Wizard values

Hand these to the user as a table filled with the real names.

**Enhancement spot** (*File > New > Other ABAP Repository Object > Enhancements > Enhancement Spot*): Package, Name, Description, BAdI Interface "none" at creation; then transport request. In the spot editor choose **Add BAdI Definition**:

| Field | Value |
|---|---|
| BAdI Definition | `ZRJ_BADI_DISCOUNT` |
| Description | short text |
| Interface | `ZRJ_IF_DISCOUNT` |
| Usage | Single Use / Multiple Use |
| Fallback class | `ZRJ_CL_DISCOUNT_FALLBACK` (Use Fallback Class checked) |
| Instance creation | New instance each call, unless implementations hold no state and performance matters (then Reuse) |
| Filters | name, type (data element or built-in type, length), e.g. `COUNTRY` / `LAND1` |

**BAdI implementation** (*New > Other ABAP Repository Object > Enhancements > BAdI Enhancement Implementation*): Package, Name, Description, Enhancement Spot; then **Add BAdI Implementation**: BAdI definition, implementation name, implementing class, filter condition (e.g. `COUNTRY = DE`), **Active** checked.

Field labels differ slightly between ADT releases; match by meaning.

## Rules

- **Single use** when exactly one result is needed (a value, a decision). `GET BADI` fails if there is no implementation and no fallback (`cx_badi_not_implemented`) or more than one active implementation for the filter (`cx_badi_multiply_implemented`). Give single-use BAdIs a fallback class so the caller always works.
- **Multiple use** when every implementation should run (notifications, extra checks, adding fields). Methods may only have `IMPORTING` and `CHANGING` parameters, no `EXPORTING` or `RETURNING`. With no implementation `CALL BADI` simply does nothing.
- Design the interface as a stable contract: implementations in other packages/components depend on it. Add new methods instead of changing existing signatures; give new optional parameters defaults.
- Pass data in, get results out through parameters. Do not let implementations read or change the caller's global state.
- Filter names in `GET BADI ... FILTERS` must match the BAdI definition exactly; filter values must fit the filter type.
- `GET BADI` once, then `CALL BADI` as often as needed. Do not call `GET BADI` inside a tight loop.
- **ABAP Cloud**: only released types in the interface. For implementations from another software component (or key users), release the BAdI definition for extensibility (*API State > Use in Cloud Development*, contract C0) in the spot editor.
- Never call implementations directly (`NEW zrj_cl_..._de( )->method( )`) in productive code; always go through the BAdI so activation and filters apply.

## Classic only: explicit enhancement points and sections

`ENHANCEMENT-POINT` and `ENHANCEMENT-SECTION` exist only in **classic ABAP (on-premise, language version Standard ABAP)**. They are **not allowed in ABAP Cloud** and go against clean core. Recommend a BAdI instead; describe this path only if the user insists on classic code.

- They need a **simple (non-BAdI) enhancement spot** to belong to.
- `ENHANCEMENT-POINT <name> SPOTS <spot> [STATIC].` marks a place where implementations can **add** code.
- `ENHANCEMENT-SECTION <name> SPOTS <spot> [STATIC]. ... END-ENHANCEMENT-SECTION.` marks code an implementation can **replace**. Only one active implementation replaces the section.
- `STATIC` is for declarations (data, types) and is processed even when the Switch Framework switch is off; non-static is for executable code.
- Implementations are source-code plug-ins in an enhancement implementation (`ENHANCEMENT 1 <impl>. ... ENDENHANCEMENT.`), created from the enhancement point in the ABAP editor. If the user's ADT release lacks this, use SAP GUI (SE80 or SE38, *Edit > Enhancement Operations*), embedded in Eclipse via Ctrl+6.
- Implicit enhancement points (start/end of routines, end of structures) need no definition. They are for modifying SAP code and are not part of this skill.

## Checklist before handing over

- [ ] Interface contains `INTERFACES if_badi_interface.`
- [ ] Multiple-use BAdI methods have no `EXPORTING`/`RETURNING` parameters.
- [ ] Single-use BAdI has a fallback class, or the caller catches `cx_badi_not_implemented`.
- [ ] Caller uses the exact filter names of the definition.
- [ ] Spot, definition, classes and implementation all active and on the same transport (or the user knows which goes where).
- [ ] Tested with an active implementation, and with it deactivated (fallback or no-op path).
