# bcgw-last-refresh-date
code and scripts for setting last_modified_date of BCGW resources in the BC Data Catalogue
## Data Model
The data model consists of four tables, stored in the BCGW APP_UTILITY schema.
### Table and Column Definitions
|Table Name|Column Name|Data Type|Nullable||Comment|
|---|---|---|---|---|---|
|EDRP_REPL_DATES_OVERRIDES|||||Last refresh dates for tables and views that are not replicated through the normal mechanisms (FME, SDR, Materialized Views) and thus have to be enumerated separately, e.g., TRIM_TEXT_ANNO is replicated manually through ArcMap.|
||EDRP_RDO_SYSID|NUMBER(10,0)|No|1|A system generated unique identification number.|
||OBJECT_OWNER|VARCHAR2(30 BYTE)|No|2|The schema of the table/view whose last refresh date value is being specified.|
||OBJECT_NAME|VARCHAR2(30 BYTE)|No|3|The name of the table/view whose last refresh time value is being specified.|
||LAST_REFRESH_DATE|DATE|No|4|The specified last refresh date for table or view.|
|EDRP_REPL_EXCLUSION_OBJS|||||List of tables and views to be ignored when calculating the last refresh date of other objects that depend on them.  Examples are code tables and tables that underlie multiple objects (e.g., GSR_OCCUPANTS_SP underlies all the GSR occupant type views.)|
||EDRP_REO_SYSID|NUMBER(10,0)|No|1|A system generated unique identification number.|
||OBJECT_OWNER|VARCHAR2(30 BYTE)|No|2|The owner of the object (table/view) to be ignored when calculating the last refresh time any object that depends on it.|
||OBJECT_NAME|VARCHAR2(30 BYTE)|No|3|The name of the object (table/view) to be ignored when calculating the last refresh time any object that depends on it.|
|EDRP_REPL_REFRESH_DATES|||||BC Geographic Warehouse latest data refresh times associated with tables, views, and materialized views defined in the BC Data Catalogue.  There is one row for each combination of BC Data Catalogue - referenced table/view/materialized view ("Subject object") and any other table/view/materialized view that it is built from ("Parent object"), according to the Oracle system view all_dependencies.|
||EDRP_RRD_SYSID|NUMBER(10,0)|No|1|A system-generated unique identification number.|
||OBJECT_OWNER|VARCHAR2(30 BYTE)|No|2|The schema in which the subject table, view, or materialized view resides.|
||OBJECT_NAME|VARCHAR2(30 BYTE)|No|3|The name of the subject table, view, or materialized view.|
||OBJECT_TYPE|VARCHAR2(20 BYTE)|No|4|The type of the subject table, view, or materialized view, i.e., TABLE, VIEW, MVW.|
||PARENT_OBJECT_OWNER|VARCHAR2(30 BYTE)|No|5|The schema in which an object that the subject table, view, or materialized view depends on resides.|
||PARENT_OBJECT_NAME|VARCHAR2(30 BYTE)|No|6|The name of the object that the subject table, view, or materialized view depends on.|
||PARENT_OBJECT_TYPE|VARCHAR2(20 BYTE)|No|7|The type of the object that the subject table, view, or materialized view depends on, i.e., TABLE, VIEW, MVW.|
||FME_TIMESTAMP|DATE|Yes|8|The most recent datetime of an FME load that loaded more than 0 records into PARENT_OBJECT_NAME.|
||TAB_MOD_TIMESTAMP|DATE|Yes|9|The latest datetime of a change to the contents of PARENT_OBJECT_NAME, according to the Oracle system view DBA_TAB_MODIFICATIONS.|
||MVW_TIMESTAMP|DATE|Yes|10|The most recent materialized refresh of PARENT_OBJECT_NAME, if it is a materialized view.|
||DWM_TIMESTAMP|DATE|Yes|11|The most recent refresh datetime of PARENT_OBJECT_NAME, according to the DWM view.|
||SDR_TIMESTAMP|DATE|Yes|12|The most recent SDR refresh datetime of PARENT_OBJECT_NAME, if the object is replicated using SDR.|
||MIN_TIMESTAMP|DATE|Yes|13|The earliest of all the timestamps other than TAB_MOD_TIMESTAMP.  If all the other timestamps are NULL, though, then MIN_TIMESTAMP is set to TAB_MOD_TIMESTAMP.|
||MAX_TIMESTAMP|DATE|Yes|14|The latest of all the timestamps other than TAB_MOD_TIMESTAMP.  If all the other timestamps are NULL, though, then MIN_TIMESTAMP is set to TAB_MOD_TIMESTAMP.|
|EDRP_REPL_REFRESH_DATES_ROLLUP|||||BC Geographic Warehouse latest data refresh times associated with tables, views, and materialized views defined in the BC Data Catalogue.  There is one row for each BC Data Catalogue - referenced table/view/materialized view (i.e., "Subject object"). This is a rollup of all EDRP_REFRESH_DATES records associated with that Subject object.|
||EDRP_RRDR_SYSID|NUMBER(10,0)|No|1|A system generated unique identification number.|
||OBJECT_OWNER|VARCHAR2(30 BYTE)|No|2|The schema of the table/view whose last refresh date value is being specified.|
||OBJECT_NAME|VARCHAR2(30 BYTE)|No|3|The name of the table/view whose last refresh time value is being specified.|
||LAST_REFRESH_DATE|DATE|Yes|4|The specified last refresh date for table or view.|
||OBJECT_OWNER_AND_NAME|VARCHAR2(65 BYTE)|Yes|5|Object owner and name concatenated: OBJECT_OWNER || '.' || OBJECT_NAME.|
||LAST_REFRESH_DATE_STRING|VARCHAR2(50 BYTE)|Yes|6|LAST_REFRESH_DATE formatted as text: YYYY-MON-DD HH:MI.|
### Data Model Diagram
![data model](ER%20Diagram.png)
## Calculation of the last_refresh_date values
![processing flow](Processing%20Flow.png)
## Updating the BC Data Catalogue resource last_modified value
![updating resource](Updating%20Resource.png)
