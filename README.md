# bcgw-last-refresh-date
This repository describes code and scripts used for setting last_modified_date of BCGW resources in the BC Data Catalogue. The code and scripts themselves are stored in https://gogs.data.gov.bc.ca/datasets/edrp_app.
## Data Model
The data model consists of four tables and a stored procedure package, stored in the BCGW APP_UTILITY schema of the (Oracle) production BC Geographic Warehouse.
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
### Stored Procedure Package
#### EDRP_REPL_BCGW_LAST_REFRESH functions
|Method|Description|
|----|----|
|get_replication_times_1_obj|table-value function which, for an input object owner and object name, returns a table of BCGW objects. Each row contains the input parameters, the owner and name of a (parent) object upon which this (child) object depends, the type of parent object (table, view, materialized view), and the last refresh dates of the child and parent objects from the all_tab_modifications, app_utility.fme_status_log, app_utility.edrp_sdr_status_vw, app_utility.mvw_overview_vw, app_utility.dwm_status_log_mvw, views/tables|
|get_dbamod_time|get the most recent timestamp for an input object from all_tab_modifications|
|get_fme_time|get the most recent timestamp for an input object from app_utility.fme_status_log|
|get_fme_time_recent|get the most recent timestamp for an input object from app_utility.fme_status_log, restricted to updates happening since EDRP_REPL_REFRESH_DATES_ROLLUP was last updated|
|get_mvw_time|get the most recent timestamp for an input object from app_utility.mvw_overview_vw|
|get_sdr_time|get the most recent timestamp for an input object from app_utility.edrp_sdr_status_vw|
|get_dwm_time|get the most recent timestamp for an input object from app_utility.dwm_status_log_mvw|
|get_last_refresh_date|get the most recent of the timestamps from get_fme_time_recent() and get_replication_times_1_obj() |
### Data Model Diagram
![data model](ER%20Diagram.png)
## Calculation of the last_refresh_date values
EDRP_REPL_REFRESH_DATES_ROLLUP and EDRP_REPL_REFRESH_DATE are calculated daily (see FME ETL Server schedules for exact time) using FME (repository BCGW_SCHEDULED, FMW script edrp_repl_refresh_dates_rollup_bcdc_api_bcgw.fmw) 
![processing flow](Processing%20Flow.png)
## Updating the BC Data Catalogue resource last_modified value
This is also implemented by the same FME script as above (repository BCGW_SCHEDULED, FMW script edrp_repl_refresh_dates_rollup_bcdc_api_bcgw.fmw) 
![updating resource](Updating%20Resource.png)
