---
layout: post
title: Extracting data from a GraphQL endpoint
---

In this example I will be extracting data from a GraphQL endpoint.
Also, I will show the use of asyncronous execution, enabling the code to wait for the API server before continuing. Lastly, the code will show how I am only fetching only the newest data since the last run and appending this to the database.

## Setup

First we import the GraphQL library, json, asyncio and make sure we an API key available.

```python
from gql import gql, Client
from gql.transport.aiohttp import AIOHTTPTransport
import json
import asyncio
```

Below we generate the connection string and set up the transport and client.

```python
# fetch API key from environment variables
api_key=config('GRAPH_API_KEY')

# create url
graph_url="https://subgraph.com/{}/analytics/api".format(api_key)

# Select your transport with a defined url endpoint
transport = AIOHTTPTransport(url=graph_url)

# Create a GraphQL client using the defined transport
client = Client(transport=transport, fetch_schema_from_transport=True)
```

## Load latest data

If we need to automate some kind of filtering, we can interact with the API using variables. For example if we only want the newest data since the last run, we can query the latest block in the Snowflake database and only get data from after this block from the API.

First we define the variable.

```python
variables = {
    "skip" : 0,
    "first" : 10000,
    "block" : 0
    }
```

The we load the latest block in the database.

```python
sql = """ select max(BLOCK) from "DATA_LOAD"."PUBLIC"."{}" """.format(dataset_name)

curs.execute(sql)
mb = curs.fetch_pandas_all()
```

Now we can overwrite the block variable if we only need data after the last update.

```python
variables['block'] = int(mb['MAX(BLOCK)'].values[0])
```

## Load data

We can now define the function to get the data. The result from the API is in json format, but as you can see this function converts the data into a pandas dataframe.

```python
async def get_data(query, variables, subset):
    all_data = []
    temp_variables = variables.copy()

    # fetch 10000 rows of data at every loop until there is no more data to fetch
    while True:
        data = await client.execute_async(query, variable_values=temp_variables)
        data = pd.DataFrame(data[subset])

        if data.empty:
            break
        else:
            all_data.append(data)

        # Increase the skip for the next iteration
        temp_variables["skip"] += 10000
        
    # concat all dataframes into one
    all_data = pd.concat(all_data)
    
    return all_data
```

To retrieve the data we have to define a GraphQl query. Contrary to traditional REST APIs you can define much more presicely what you want to retrieve. Kind of like you can with a direct database connection.
As you can see there is a "where" statement that filters the data on a specific protocol and blockNumber greater than a certain block (blockNumber_gt).

```python
query = gql(
  """
  query
    analyticsEvents ($skip : Int!, $first:Int!, $block: BigInt!) {
        events (skip:$skip, first:$first, where: { protocol:"xxx", blockNumber_gt:$block }) {
            id
            blockNumber
            protocol
            txHash
            timestamp
            user {
                id
                address
                type
            }
        }
    } 
"""
)
```

Below we run the "get_data()" function and fetch the data.

```python
triggers = get_data(query, variables, 'events')
triggers = asyncio.run(triggers)
```

Note that you might have to deal with nested data. In the query you can see there is a user table nested inside the events table. To deal with this we have to "flatten" the table to make sure all columns are on the same level and each cell of the dataframe only have one value. So we expand the dataset with the nested user dataset like so:

```python
triggers = pd.concat([triggers.drop(['user'], axis=1).reset_index(drop=True), pd.json_normalize(triggers['user'])], axis=1)
```

The data has now been successfully extracted from the endpoint using GraphQL and is ready to use for further processing.