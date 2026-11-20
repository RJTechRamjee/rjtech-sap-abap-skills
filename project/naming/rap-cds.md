---
description: Names for RAP and CDS - view entity layers (I_/R_/C_), abstract and custom entities, value help and text views, CDS fields, keys, associations and parameters, behavior definitions, aliases, behavior pools and local handler classes, actions, functions, determinations, validations, draft tables, metadata extensions, access controls, service definitions and bindings. Load before creating or naming any RAP or CDS artifact.
---

# RAP and CDS

General rules:

- CDS entity names use `ZRJ_<MOD>_<SUB>_<LAYER>_<Name>`. `<MOD>` and `<SUB>` are codes from
  `modules` in `project/project.yaml`. The `<Name>` part is CamelCase and singular
  (`ZRJ_SD_SLS_R_SalesOrder`, not `ZRJ_SD_SLS_R_SALESORDERS`). ABAP repository names are upper
  case anyway, but the CamelCase spelling is what appears in code and in OData.
- Max 30 characters for every CDS and RAP object name, including the prefix.
  **Room for `<Name>`** below assumes a 2-letter module and a 3-letter submodule (`ZRJ_SD_SLS_`).
- One business object = one `<Name>` used in every layer: `ZRJ_SD_SLS_ORD` (table) →
  `ZRJ_SD_SLS_R_SalesOrder` → `ZRJ_SD_SLS_C_SalesOrder` → `ZRJ_SD_SLS_UI_SALESORDER` →
  `ZRJ_SD_SLS_UI_SALESORDER_O4`.
- Skill templates use `ZRJ_R_Travel` and similar names: insert `<MOD>_<SUB>_` after `ZRJ_`
  (`ZRJ_CA_DMO_R_Travel`).

## View entities and other CDS entities

| Object | Pattern | Room for `<Name>` | Example |
|---|---|---|---|
| Interface (basic / composite) view | `ZRJ_<MOD>_<SUB>_I_<Name>` | 17 | `ZRJ_SD_SLS_I_SalesOrder` |
| RAP BO (restricted, transactional) view | `ZRJ_<MOD>_<SUB>_R_<Name>` | 17 | `ZRJ_SD_SLS_R_SalesOrder`, `ZRJ_SD_SLS_R_SalesOrderItem` |
| Projection / consumption view | `ZRJ_<MOD>_<SUB>_C_<Name>` | 17 | `ZRJ_SD_SLS_C_SalesOrder` |
| Projection view for an API service | `ZRJ_<MOD>_<SUB>_C_<Name>API` (`_A_` is taken by abstract entities) | 14 | `ZRJ_SD_SLS_C_SalesOrderAPI` |
| Value help view | `ZRJ_<MOD>_<SUB>_I_<Name>VH` | 15 | `ZRJ_SD_SLS_I_OrderStatusVH` |
| Text view (language-dependent texts) | `ZRJ_<MOD>_<SUB>_I_<Name>Text` | 13 | `ZRJ_SD_SLS_I_OrderStatusText` |
| Analytical cube / dimension / query | `..._I_<Name>Cube` / `..._I_<Name>Dim` / `..._C_<Name>Qry` | 13 / 14 / 14 | `ZRJ_SD_BIL_I_InvoiceCube` |
| Abstract entity (action / function parameter or result) | `ZRJ_<MOD>_<SUB>_A_<Name>` | 17 | `ZRJ_SD_PRC_A_DiscountParam` |
| Custom entity (data from ABAP) | `ZRJ_<MOD>_<SUB>_CE_<Name>` | 16 | `ZRJ_MM_IM_CE_StockOverview` |
| Query provider class for a custom entity | `ZRJ_<MOD>_<SUB>_CL_<NAME>_QRY` | 12 | `ZRJ_MM_IM_CL_STOCK_OVW_QRY` |
| Table function / its implementing class | `ZRJ_<MOD>_<SUB>_TF_<Name>` / `ZRJ_<MOD>_<SUB>_CL_TF_<NAME>` | 16 / 13 | `ZRJ_FI_AR_TF_OpenItems` |
| Metadata extension | Same name as the view it annotates | | `ZRJ_SD_SLS_C_SalesOrder` |
| Access control (DCL) | Same name as the protected view | | `ZRJ_SD_SLS_R_SalesOrder` |
| CDS extension | `ZRJ_<MOD>_<SUB>_X_<Name>` | 17 | `ZRJ_SD_SLS_X_OrderPriority` |

- Never create the old `define view` (DDIC-based) views. No `_DDL` / `_CDS` suffixes.
- `@EndUserText.label` is a short English label (`'Travel - BO view'`); it's what users see in tools.

## Fields, keys, associations, parameters

| Element | Rule | Example |
|---|---|---|
| Database table field | lower snake_case, max 16 characters | `travel_uuid`, `begin_date` |
| CDS element (alias) | UpperCamelCase, English, no abbreviations except common ones (`ID`, `UUID`) | `travel_id as TravelID` |
| UUID key | `<Entity>UUID`; the parent key in a child entity keeps the parent's name | `TravelUUID` in `ZRJ_R_Booking` |
| Semantic ID | `<Entity>ID` | `TravelID`, `BookingID` |
| Amount + currency | `<Something>Price`/`Amount` + `CurrencyCode` | `TotalPrice`, `CurrencyCode` |
| Quantity + unit | `<Something>Quantity` + `<Something>Unit` | `OrderQuantity`, `OrderQuantityUnit` |
| Admin fields (table) | `created_by`, `created_at`, `local_last_changed_by`, `local_last_changed_at`, `last_changed_at` | |
| Admin fields (CDS) | `CreatedBy`, `CreatedAt`, `LocalLastChangedBy`, `LocalLastChangedAt`, `LastChangedAt` | |
| Association / composition | `_<Target>` in UpperCamelCase, singular for to-one, plural allowed for to-many | `_Customer`, `_Booking` |
| Parent association | `_<Parent>` | `_Travel` |
| Text association | `_Text` | |
| CDS parameter | `P_<Name>` | `P_KeyDate` |
| Calculated / virtual element | Name the result, not the formula | `RemainingDays` |

## Behavior definitions

| Object | Pattern | Room for `<NAME>` | Example |
|---|---|---|---|
| Behavior definition | Same name as its root entity | | `ZRJ_SD_SLS_R_SalesOrder`, `ZRJ_SD_SLS_C_SalesOrder` |
| Entity alias | UpperCamelCase, singular, no prefix, no module | | `alias SalesOrder`, `alias Item` |
| Behavior pool (BO) | `ZRJ_<MOD>_<SUB>_BP_R_<NAME>` (one per behavior definition) | 14 | `ZRJ_SD_SLS_BP_R_SALESORDER` |
| Behavior pool (projection, only if needed) | `ZRJ_<MOD>_<SUB>_BP_C_<NAME>` | 14 | `ZRJ_SD_SLS_BP_C_SALESORDER` |
| Draft table | `<TABLE>_D` (max 16, see `global-objects`) | 3 | `ZRJ_SD_SLS_ORD_D` |
| Local handler class (in the pool) | `lhc_<alias>` | | `lhc_salesorder`, `lhc_item` |
| Local saver class (in the pool) | `lsc_<alias of the root>` | | `lsc_salesorder` |
| Business event | `<object><PastTense>`, lowerCamelCase | | `salesOrderReleased` |

Operations in the behavior definition are lowerCamelCase, start with a verb, and carry no
module codes (they live inside the BO):

| Operation | Pattern | Example |
|---|---|---|
| Action | `<verb><Object>` | `releaseOrder`, `rejectOrder`, `copyOrder` |
| Factory action | `copy<Entity>` / `create<Entity>From<Source>` | `copySalesOrder` |
| Function | `get<Something>` | `getTotalPrice` |
| Determination | `set<Field>` or `calculate<Field>` | `setSalesOrderID`, `calculateTotalPrice` |
| Validation | `validate<Field or Rule>` | `validateCustomer`, `validateDates` |
| Side effect target / feature control | use the field and action names above | |

- The handler method has the same name as the operation (`METHODS validateDates FOR VALIDATE ON SAVE`).
- Standard draft actions keep their SAP names: `Edit`, `Activate`, `Discard`, `Resume`, `Prepare`.
- Handler method parameters keep the RAP framework names (`keys`, `result`, `mapped`, `failed`,
  `reported`, `requested_authorizations`); prefixes from `local-definitions` apply to everything
  you declare yourself.

## Services

| Object | Pattern | Room for `<NAME>` | Example |
|---|---|---|---|
| Service definition for a Fiori UI | `ZRJ_<MOD>_<SUB>_UI_<NAME>` | 13 (so the binding fits) | `ZRJ_SD_SLS_UI_SALESORDER` |
| Service definition for an API | `ZRJ_<MOD>_<SUB>_API_<NAME>` | 12 | `ZRJ_SD_SLS_API_SALESORDER` |
| Service binding (OData V4) | `<service definition>_O4` | | `ZRJ_SD_SLS_UI_SALESORDER_O4` |
| Service binding (OData V2, only if a client needs it) | `<service definition>_O2` | | `ZRJ_SD_SLS_UI_SALESORDER_O2` |
| Exposed entity name in the service | The alias, no prefix, no module | | `expose ZRJ_SD_SLS_C_SalesOrder as SalesOrder` |
| Communication scenario / IAM app | `ZRJ_<MOD>_<SUB>_<NAME>_CSCEN` / `_IAM` | | `ZRJ_SD_SLS_ORDER_CSCEN` |
