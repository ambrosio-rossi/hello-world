---
title: HDBSequence
---

HDBSequence
===

## Overview
---

The information on this page will help you lesrn how to develop the design-time data-persistence model for an XSK application using the HDBSequence syntax.

## Reference
---

!!! note "SAP Help Portal"

    - SAP HANA XS Classic: For more information, see [Create a Sequence](https://help.sap.com/viewer/cc2b23beaa3344aebffa2f6e717df049/2.0.03/en-US/a1e95af655ee4e00bd9183518d1fa5c5.html).
    - SAP HANA XS Advanced: For more information, see [CREATE SEQUENCE Statement (Data Definition)](https://help.sap.com/viewer/4fe29514fd584807ac9f2a04f6754767/1.0.12/en-US/20d509277519101489029c064d468c5d.html).

## Sample
---

=== "HANA XS Classic syntax"

    ```json
    schema= "TEST_DUMMY";
    start_with= 10;
    maxvalue= 30;
    nomaxvalue=false;
    minvalue= 1;
    nominvalue=true;
    cycles= false;
    reset_by= "SELECT T1.\"Column2\" FROM \"MYSCHEMA\".\"com.acme.test.tables::MY_TABLE1\" AS T1 LEFT JOIN \"MYSCHEMA\".\"com.acme.test.tables::MY_TABLE2\" AS T2 ON T1.\"Column1\" = T2.\"Column1\"";
    depends_on=["com.acme.test.tables::MY_TABLE1", "com.acme.test.tables::MY_TABLE2"];
    ```

=== "HANA XS Advanced syntax"

    ```sql
    SEQUENCE "com.sap.hana.example::CUSTOMER_ID" 
    RESET BY 
    SELECT IFNULL(MAX(ID), 0) + 1 FROM "com.sap.hana.example::CUSTOMERS"
    ```

**Configuration Schema**

| Name       | Description                                                 | Type     | Default value | Required |
|------------|-------------------------------------------------------------|:--------:|:-------------:|:--------:|
| schema | Name of the schema that contains the sequence you are defining. |  String  |     None      |   Yes    |
| increment_by | Specifies that the sequence increments by a defined value.  | Integer  |      1        |    No    |
| start_with | Specifies that the sequence starts with a specific value. Cannot be less than the minimum value. | Integer  |  1  | No |
| maxvalue | Specifies that the sequence stops at a specific maximum value.  | Integer  |  Different for each DB | No |
| nomaxvalue | Specifies that the sequence does not stop at any specific maximum value. | Boolean | False | No |
| minvalue | Specifies that the sequence stops at a specific minimum value.  | Integer  |      1        | No |
| nominvalue | Specifies that the sequence does not stop at any specific minimum value. |    Boolean    |  False  |  No  |
| cycles | Enables you to specify whether the sequence number will be restarted after it reaches its maximum or minimum value. | Boolean | None | No |
| reset_by | Enables you to reset the sequence using a query on any view, table or even table function. However, any dependency must be declared explicitly, for example, with the depends_on_view or depends_on_table keyword. If the table or view specified in the dependency does not exist, the activation of the sequence object in the repository fails. | String | None | No |
| public | Specifies if it is public or not. |    Boolean    | False | No |
| depends_on_table | Enables you to define a dependency to one or more specific tables. | List(String) | None | No |
| depends_on_view | Enables you to define a dependency to one or more specific views. | List(String) | None | No |
| depends_on | Enables you to define a dependency to one or more specific tables or views, for example, when using the reset_by option to specify the query to use when resetting the sequence. | List(String) | None | No |

!!! warning "Caution"

    [Reserved keywords](https://help.sap.com/viewer/f381aa9c4b99457fb3c6b53a2fd29c02/2.0.02/en-US/888f434c5e494d6c9af82bc5db854a14.html), used as table or column names in queries, must be escaped.
