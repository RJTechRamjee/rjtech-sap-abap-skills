# Templates: enhancement spot and BAdI

Example: a single-use BAdI `ZRJ_BADI_DISCOUNT` in spot `ZRJ_ES_TRAVEL` decides the discount percentage for a travel, filtered by country. Keep the `ZRJ_<TYPE>_` prefix; replace the names after it and the parameters with the user's.

## 1. BAdI interface

```abap
INTERFACE zrj_if_discount PUBLIC.
  INTERFACES if_badi_interface.

  TYPES ty_percent TYPE p LENGTH 5 DECIMALS 2.

  "! Returns the discount in percent for a travel
  "! @parameter customer_id    | Customer of the travel
  "! @parameter total_price    | Travel price before discount
  "! @parameter discount       | Discount in percent (0-100)
  METHODS get_discount
    IMPORTING customer_id     TYPE /dmo/customer_id
              total_price     TYPE /dmo/total_price
    RETURNING VALUE(discount) TYPE ty_percent.
ENDINTERFACE.
```

`RETURNING` is fine here because the BAdI is single use. For a multiple-use BAdI use `CHANGING` instead (see section 6).

## 2. Fallback class

```abap
CLASS zrj_cl_discount_fallback DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    INTERFACES zrj_if_discount.
ENDCLASS.

CLASS zrj_cl_discount_fallback IMPLEMENTATION.
  METHOD zrj_if_discount~get_discount.
    discount = 0.
  ENDMETHOD.
ENDCLASS.
```

## 3. Enhancement spot and BAdI definition (wizard)

| Field | Value |
|---|---|
| Enhancement spot | `ZRJ_ES_TRAVEL` |
| BAdI definition | `ZRJ_BADI_DISCOUNT` |
| Interface | `ZRJ_IF_DISCOUNT` |
| Usage | Single Use |
| Fallback class | `ZRJ_CL_DISCOUNT_FALLBACK` |
| Filter | `COUNTRY`, type `LAND1` (CHAR 3) |

## 4. Caller

Put the BAdI access in a small class behind an interface, so callers can be unit-tested with a double (section 7) instead of a real BAdI.

```abap
INTERFACE zrj_if_discount_provider PUBLIC.
  METHODS get_discount
    IMPORTING country         TYPE land1
              customer_id     TYPE /dmo/customer_id
              total_price     TYPE /dmo/total_price
    RETURNING VALUE(discount) TYPE zrj_if_discount=>ty_percent.
ENDINTERFACE.
```

```abap
CLASS zrj_cl_discount_provider DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    INTERFACES zrj_if_discount_provider.
ENDCLASS.

CLASS zrj_cl_discount_provider IMPLEMENTATION.
  METHOD zrj_if_discount_provider~get_discount.
    DATA badi TYPE REF TO zrj_badi_discount.

    TRY.
        GET BADI badi FILTERS country = country.
        CALL BADI badi->get_discount
          EXPORTING customer_id = customer_id
                    total_price = total_price
          RECEIVING discount    = discount.
      CATCH cx_badi_not_implemented cx_badi_multiply_implemented.
        " Only reachable without a fallback class or with two active
        " implementations for the same filter value
        discount = 0.
    ENDTRY.
  ENDMETHOD.
ENDCLASS.
```

The BAdI name (`zrj_badi_discount`), not the interface, is the reference type for `GET BADI`.

## 5. Implementation for Germany

Implementing class:

```abap
CLASS zrj_cl_discount_de DEFINITION PUBLIC FINAL CREATE PUBLIC.
  PUBLIC SECTION.
    INTERFACES zrj_if_discount.
ENDCLASS.

CLASS zrj_cl_discount_de IMPLEMENTATION.
  METHOD zrj_if_discount~get_discount.
    discount = COND #( WHEN total_price >= 5000 THEN 10
                       WHEN total_price >= 1000 THEN 5
                       ELSE 0 ).
  ENDMETHOD.
ENDCLASS.
```

Enhancement implementation (wizard):

| Field | Value |
|---|---|
| Enhancement implementation | `ZRJ_EI_DISCOUNT_DE` |
| Enhancement spot | `ZRJ_ES_TRAVEL` |
| BAdI implementation | `ZRJ_BI_DISCOUNT_DE` |
| BAdI definition | `ZRJ_BADI_DISCOUNT` |
| Implementing class | `ZRJ_CL_DISCOUNT_DE` |
| Filter condition | `COUNTRY = DE` |
| Active | checked |

## 6. Multiple-use variant

Every active implementation runs, in an undefined order. Results come back through `CHANGING`.

```abap
INTERFACE zrj_if_travel_check PUBLIC.
  INTERFACES if_badi_interface.

  TYPES ty_messages TYPE STANDARD TABLE OF string WITH EMPTY KEY.

  METHODS check
    IMPORTING travel_id TYPE /dmo/travel_id
    CHANGING  messages  TYPE ty_messages.
ENDINTERFACE.
```

```abap
DATA badi     TYPE REF TO zrj_badi_travel_check.
DATA messages TYPE zrj_if_travel_check=>ty_messages.

GET BADI badi.
CALL BADI badi->check
  EXPORTING travel_id = travel_id
  CHANGING  messages  = messages.
```

No `TRY` needed: without implementations `CALL BADI` does nothing.

## 7. ABAP Unit

Test the implementation class directly; it is a plain class:

```abap
CLASS ltc_discount_de DEFINITION FINAL FOR TESTING
  RISK LEVEL HARMLESS DURATION SHORT.
  PRIVATE SECTION.
    DATA cut TYPE REF TO zrj_if_discount.
    METHODS setup.
    METHODS ten_percent_from_5000 FOR TESTING.
    METHODS none_below_1000 FOR TESTING.
ENDCLASS.

CLASS ltc_discount_de IMPLEMENTATION.
  METHOD setup.
    cut = NEW zrj_cl_discount_de( ).
  ENDMETHOD.

  METHOD ten_percent_from_5000.
    cl_abap_unit_assert=>assert_equals(
      act = cut->get_discount( customer_id = '000001' total_price = 5000 )
      exp = 10 ).
  ENDMETHOD.

  METHOD none_below_1000.
    cl_abap_unit_assert=>assert_equals(
      act = cut->get_discount( customer_id = '000001' total_price = 999 )
      exp = 0 ).
  ENDMETHOD.
ENDCLASS.
```

Code that **uses** the discount depends on `zrj_if_discount_provider` (constructor injection), so its tests replace the BAdI with a double and do not depend on which implementations are active in the system:

```abap
DATA(provider) = CAST zrj_if_discount_provider(
  cl_abap_testdouble=>create( 'ZRJ_IF_DISCOUNT_PROVIDER' ) ).
cl_abap_testdouble=>configure_call( provider )->returning( 5 ).
provider->get_discount( country = 'DE' customer_id = '000001' total_price = 100 ).

DATA(cut) = NEW zrj_cl_travel_pricing( discount_provider = provider ).
```

`zrj_cl_travel_pricing` stands for the user's consuming class. See the `abap-unit-rap` skill for general test rules.

## 8. Classic only: explicit enhancement point and section

Standard ABAP (on-premise) only; not valid in ABAP Cloud. `ZRJ_ES_TRAVEL_SRC` is a simple enhancement spot (not a BAdI spot).

```abap
" Declarations an implementation may add
ENHANCEMENT-POINT zrj_ep_travel_data SPOTS zrj_es_travel_src STATIC.

" Code an implementation may add to
ENHANCEMENT-POINT zrj_ep_travel_before_save SPOTS zrj_es_travel_src.

" Default code an implementation may replace
ENHANCEMENT-SECTION zrj_sec_discount SPOTS zrj_es_travel_src.
  discount = 0.
END-ENHANCEMENT-SECTION.
```

An implementation of the point (created from the point in the editor) looks like:

```abap
ENHANCEMENT 1 zrj_ei_travel_src_de.
  IF country = 'DE'.
    discount = 5.
  ENDIF.
ENDENHANCEMENT.
```
