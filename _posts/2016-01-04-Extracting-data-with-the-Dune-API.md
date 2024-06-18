---
layout: post
title: Extracting data with the Dune API
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
Here I show how to extract a dataset created on Dune.com. I show both the python setup and the Dune SQL, which can be filtered with a paramenter, that can be controlled when querying the API with python.

## SQL on Dune.com

Here is an example of a table being created with Dune. I am fetching all transactions ever done on Instadapp and Defisaver on ethereum mainnet.
You can see I am using a parameter in the where statements to be able to filter on what timeframe I want when I fetch the data using python.

```sql
/* INSTADAPP */

with instadapp_accounts as (
    SELECT distinct account from instadapp_dsa_ethereum.InstaIndex_evt_LogAccountCreated
),

id_txs as (
    select block_time as "timestamp", hash as tx_hash, 'Instadapp' as frontend
    from ethereum.transactions
    join instadapp_accounts on to = account
    where block_time > from_unixtime({{day_limit}}, 'UTC')
    group by 1,2,3
),

/* DEFISAVER */

ds_txs as (
    select evt_block_time as "timestamp", evt_tx_hash as tx_hash, 'Defisaver' as frontend
    from defisaver_ethereum.DefisaverLogger_evt_LogEvent
    where evt_block_time > from_unixtime({{day_limit}}, 'UTC')
    group by 1, 2, 3
    
    union all
    
    select evt_block_time as "timestamp", evt_tx_hash as tx_hash, 'Defisaver' as frontend
    from defisaver_ethereum.DefisaverLogger_evt_RecipeEvent
    where evt_block_time > from_unixtime({{day_limit}}, 'UTC')
    group by 1, 2, 3
    
    union all
    
    select evt_block_time as "timestamp", evt_tx_hash as tx_hash, 'Defisaver' as frontend
    from defisaver_ethereum.DefisaverLogger_evt_ActionDirectEvent
    where evt_block_time > from_unixtime({{day_limit}}, 'UTC')
    group by 1, 2, 3
)

select * from (
    select * from id_txs
    union all
    select * from ds_txs
)
order by timestamp desc
```

You can see I am using a parameter in the where statements to be able to filter on what timeframe I want when I fetch the data using python.

```sql
where evt_block_time > from_unixtime({{day_limit}}, 'UTC')
```

## Fetch data with python

In python I am using the requests library to connect to Dune's API:

```python
from requests import get, post
```

To create the connection string we need a Dune API key and a function colleting the various attributes into a string we can use.

```python
HEADER = {"x-dune-api-key" : config('DUNE_API_KEY')}
```

```python
BASE_URL = "https://api.dune.com/api/v1/"

def make_api_url(module, action, ID):
    """
    We shall use this function to generate a URL to call the API.
    """

    url = BASE_URL + module + "/" + ID + "/" + action

    return url
```

To execute the query and fetch the results we need 3 functions. One to provide us with an execution ID that contains the action we need performed on which Dune query.
Another to get the status of the request. We need to wait for Dune to finish runnig the SQL query before we try to fetch the data.
Lastly, we need the function we run once the data is ready to be fetched.

```python
def execute_query_with_params(query_id, param_dict):
    """
    Takes in the query ID. And a dictionary containing parameter values.
    Calls the API to execute the query.
    Returns the execution ID of the instance which is executing the query.
    """

    url = make_api_url("query", "execute", query_id)
    response = post(url, headers=HEADER, json={"query_parameters" : param_dict})
    execution_id = response.json()['execution_id']

    return execution_id

def get_query_status(execution_id):
    """
    Takes in an execution ID.
    Fetches the status of query execution using the API
    Returns the status response object
    """

    url = make_api_url("execution", "status", execution_id)
    response = get(url, headers=HEADER)

    return response

def get_query_results(execution_id):
    """
    Takes in an execution ID.
    Fetches the results returned from the query using the API
    Returns the results response object
    """

    url = make_api_url("execution", "results", execution_id)
    response = get(url, headers=HEADER)

    return response
```

In this case we also need to define the day_limit timestamp which should be in unix format here.

```python
parameters = {'day_limit':[unix timestamp]}
```
```python
execution_id = execute_query_with_params("xyz", parameters)
```

Now that we have the execution ID we can put it all together in the final function that starts the action on the Dune server and loops over the query status every 10 seconds until we get 'QUERY_STATE_COMPLETED' and the data can be fetched. Result will be a pandas dataframe.

```python
def queryDataWait(execution_id):

    tries = 50
    while tries > 0:
        print(get_query_status(execution_id).json()['state'])
        if get_query_status(execution_id).json()['state'] == 'QUERY_STATE_COMPLETED':
            response = get_query_results(execution_id)
            data = pd.DataFrame(response.json()['result']['rows'])
            tries = 0
            return data
        else:
            time.sleep(10) 
            tries -= 1                
            continue
```

To initiate everything we simply run this:

```python
df = queryDataWait(execution_id)
```

When running the function it will print the status every 10 seconds until it is completed.

QUERY_STATE_EXECUTING
QUERY_STATE_EXECUTING
QUERY_STATE_EXECUTING
QUERY_STATE_COMPLETED