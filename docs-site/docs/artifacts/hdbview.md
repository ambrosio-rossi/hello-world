---
title: HDBView
---

HDBView
===

## Overview
---

The information on this page will help you learn how to develop the design-time data-persistence model for an XSK application using the HDBView syntax.

## Details About the Parser
---

!!! warning "Caution"

    There are currently some [issues](https://github.com/SAP/xsk/issues/108) in the behavior of the HANA version 1.x of the parser:

    - Currently, the parser takes into accouцnt if a given property is mandatory.
    - If the property order is misplaced, the parser will still parse the values.
    - If more than one value of one property is provided, then only the last one is taken.     

## Reference
---

!!! note "SAP Help Portal"

    For more information, see [Create an SQL View](https://help.sap.com/viewer/cc2b23beaa3344aebffa2f6e717df049/2.0.03/en-US/78dcb8e2f4f14b53b0d333bcd24f8721.html).

## Sample
---

!!! warning "Caution"

    [Reserved keywords](https://help.sap.com/viewer/f381aa9c4b99457fb3c6b53a2fd29c02/2.0.02/en-US/888f434c5e494d6c9af82bc5db854a14.html), used as table or column names in queries, must be escaped.

=== "HANA XS Classic syntax"

    ```sql
    schema="MYSCHEMA";
    query="SELECT T1.\"Column2\" FROM \"MYSCHEMA\".\"acme.com.test.views::MY_VIEW1\" AS T1 LEFT JOIN \"MYSCHEMA\".\"acme.com.test.views::MY_VIEW2\" AS T2 ON T1.\"Column1\" = T2.\"Column1\"";
    depends_on=["acme.com.test.views::MY_VIEW1", "acme.com.test.views::MY_VIEW2"];
    ```

=== "HANA XS Advanced syntax"

    ```sql
    VIEW "hdb_view.db::ItemsByOrderHANAv2"
    AS SELECT "Id" FROM "hdb_view::Item";
    ```

**Which syntax does the parser support?**

| HANA XS Classic syntax  | HANA XS Advanced syntax        | Comments     |
|-------------------------|--------------------------------|--------------|
| schema                  | schema                         |              |
| query                   | query                          |              |
| public                  | public                         | default=true |
| depends_on              | depends_on                     |              |
| depends_on_table        | depends_on_table               |              |
| depends_on_view         | depends_on_view                |              |

!!! warning "Caution"

    [Reserved keywords](https://help.sap.com/viewer/f381aa9c4b99457fb3c6b53a2fd29c02/2.0.02/en-US/888f434c5e494d6c9af82bc5db854a14.html), used as table or column names in queries, must be escaped.
