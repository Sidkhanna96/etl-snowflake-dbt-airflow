- ELT Pipelines (Extract Load Transform):
  - Extract (ADF, Spark) -> Pull data from APIs, DBs or files
  - Load (Snowflake, Databricks) -> Store data here
  - Transform (DBT) -> Transform the data from one form to another
  - Orchestrator (Airflow) -> Trigger pipelines on scheduled basis to perform transformation

What I did:

- Snowflake - Load
  - Created a data warehouse and database along with assigning [roles](#sample-code)
  - Leveraged the sample data from snowflake to transform data into created database

- DBT - Transform
  - `dbt init`
  - Created a Staging folder containing the first part of the Transformation process -> to fetch data from the warehouse (Snowflake Sample TPCH in this case)
    - created sources yaml to [fetch](./dbt-dag-airflow/dags/dbt/data_pipeline/models/staging/tpch_sources.yml) from the datawarehouse -> What DB -> What tables
    - Converted the data from sample snowflake DB into better naming strategy (Did this for both - lineItems and orders)
    - `dbt run -s stg_tpch_orders.sql` when this command is ran it creates a schema of the same name in the dbt_db folder
  - Created a marts folder for containing business logic:
    - Fetch all of this through ref sd opposed to source command to conver the business logic of the dbt_db into a separate table
  - Created macros functions for repeated multi use locations
  - Created a fact tables:
    - Containing numeric values of the quantitive instances of given set of sources containing foreign keys for other tables
    - Grain - what one row in the table represents

- Airflow:
  - `astro dev init`
  - For running the staging and marts scripts on a schedule basis - i.e. tranforming the data from the db warehouses from various table and then placing it in our source db table
  - Configuration:
    - install astronomer - `brew install astro`
    - Create a dockerfile for running on a docker container and its instance
    - dbt_dag.py under dags folder to connect to the db warehouse
    - dbt_project.yml - define the dbt tags
    - Store the DBT transform code into dags/dbt/data_pipeline
  - run the astro instance - `astro dev`

### Sample Code

Snowflake:

```sql
use role accountadmin;

create warehouse if not exists dbt_wh with warehouse_size='x-small'; -- Read heavy system to store data from various different sources - its a separate entity from a DB
create database if not exists dbt_db;
create role if not exists dbt_role;

show grants on warehouse dbt_wh;

grant usage on warehouse dbt_wh to role dbt_role;
grant role dbt_role to user skidhs;
grant all on database dbt_db to role dbt_role;

use role dbt_role;

create schema if not exists dbt_db.dbt_schema;

use role accountadmin;

drop warehouse if exists dbt_wh;
drop database if exists dbt_db;
drop role if exists dbt_role;
```
