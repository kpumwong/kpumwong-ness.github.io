---
layout: post
title: Using python to interact with Snowflake
---

Simple example of how to use the python snowflake-connector library to interact with a Snowflake database. Both reading and writing data.

## Setup

To get started we need the snowflake-connector library imported and the credentials to allow for connection to the database.

Here we import the library:

```python
import snowflake.connector
from snowflake.connector.pandas_tools import write_pandas
```

And here we create the connection string and cursor where sensitive information like the username and password can be read from a .env file using config().

```python
conn = snowflake.connector.connect(
    user=config('SNOW_USER'),
    password=config('SNOW_PASSWORD'),
    account=config('SNOW_ACCOUNT'),
    database='ANALYTICS',
    schema='PUBLIC',
    warehouse='COMPUTE_WH',
    role='ACCOUNTADMIN'
)
curs = conn.cursor()
```

## Read data

Using the cursor, we can execute any supported SQL code remotely on the Snowflake database and fetch the results like so:

```python
sql =  """ select * from "DATABASE"."SCHEMA"."TABLE/VIEW"; """

curs.execute(sql)
df = curs.fetch_pandas_all()
```

The query results will be written directly into a pandas dataframe for further processing.

## Write data

To write a pandas dataframe into a table in Snowflake we simply need the write_pandas() function.
If the database, schema and table names are not already defined in the connection string, we can define it inside the function. You can also write the dataframe in chunks if it is a big dataset. Lastly, we save "success, nchunks and nrows" to be able to print out whether it was successful and how many chunks and rows were written. Usually you would print this to a log.

```python
success, nchunks, nrows, _ = write_pandas(conn=conn, df=dataframe_name, database='ANALYTICS', schema='PUBLIC', table_name=dataset_name, chunk_size=1000000)
```

```python
logging.info('--- ' + dataset_name + ' --- Success: ' + str(success) + ',' + ' Chunks: ' + str(nchunks) + ',' + ' Rows: ' + str(nrows) + ' Finished! Data written to Snowflake!')
```

If you want to have more control over how the table is being created in Snowflake you can create it manually using the cursor. Below you can see a small example.

```python
sql = """
create or replace table "ANALYTICS"."PUBLIC"."{}" COPY GRANTS (
  BLOCK FLOAT,
  TIMESTAMP TIMESTAMP_NTZ,
  TX_HASH VARCHAR(16777216),
  USER_ADDRESS VARCHAR(16777216),
  COLLATERAL_VOLUME FLOAT
  )
""".format(dataset_name)

curs.execute(sql)
```