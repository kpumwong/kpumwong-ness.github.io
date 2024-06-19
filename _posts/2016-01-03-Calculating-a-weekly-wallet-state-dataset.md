---
layout: post
title: Calculating a weekly wallet state dataset
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
Here I will show how to create a dataset from blockchain DeFi borrowing/lending transactions containing the state of all positions (or value locked) at any given point in time. We do this to be able to plot TVM for example (Total Value Managed), which is an important metric to keep track of.

## Transaction dataset

Blockchain data consists of events emitted to the blockchain. In the borrowing and lending world of DeFi users take up positions using a collateral token like Ethereum and perform events like deposits, withdraws, drawing debt and paying back debt. We can also call these "operations" on the blockchain. All transactions on the blockchain has a unique transaction hash where multiple events or operations can occur.

Important columns that a dataset like this would contain, and for the purpose of this example, are timestamp, transaction_hash, operation, position_id, collateral_volume, principal_volume.
Collateral volume would be ETH deposited (positive amount) and withdrawn (negative amount). Principal volume would be draw debt (positive amount) and pay back debt (negative amount).
For the sake of simplicity let's say all amounts are already converted to USD based on the token price at the time of transaction.

## Cummulative sum

First let us make sure we have only one row per transaction hash. We don't need the granularity of operations.

```python
df = df.groupby(['TIMESTAMP', 'POSITION_ID', 'COLLATERAL_TOKEN', 'TX_HASH']).agg({'COLLATERAL': 'sum', 'PRINCIPAL':'sum'}).reset_index()
```

Now we need to do a cummulative sum so we have the value locked at the time of each transaction.

```python
v_status[['LOCKED_COLLATERAL', 'LOCKED_DEBT']] = v_status.groupby('POSITION_ID')[['COLLATERAL_VOLUME', 'PRINCIPAL_VOLUME']].cumsum()
```

The challenge is now that we need to be able to know the value locked for each position even in the weeks where the user did not do any transactions. For example if a user deposited 10 ETH in week 1 and withdrew 5 ETH in week 6, the value locked in the weeks in between will be 10 ETH. We need to be able to see this in the dataset.

# Expanding dataset with value locked for all weeks

First we need to calculate which week each timestamp belong to so we can group on this afterwards. The new timestamp column will contain the date of the Sunday in the week where the transaction occured.

```python
df['TIMESTAMP'] = pd.to_datetime(df['TIMESTAMP'].dt.to_period('W-SUN').dt.to_timestamp('W-SUN'), utc=True)
```
Now we can group on this to and take the last value of the cummulative sum. We know know what the value locked was at the end of each week a transaction occured.

```python
df = df.groupby(['POSITION_ID', 'TIMESTAMP']).agg({'LOCKED_COLLATERAL':'last', 'LOCKED_DEBT':'last'}).reset_index()
```

We now need to expand the dataset to also include the weeks where there were no transactions happening as there might still be value locked in the positions.
First let's make sure the data is sorted on timestamp, as we are going to fill forwards values.

```python
df = df.sort_values('TIMESTAMP').set_index(['TIMESTAMP', 'POSITION_ID'])
```

First we unstack to get all dates for all positions. We use 'del' as default is 0. We want to be able to distinguish the collateral_locked 0s in the original dataset, as this would be the user closing the position.

```python
df = df.unstack(fill_value='del')
```

Now we use asfreq to make sure all weeks are represented for each position. Even weeks were no transaction happened.

```python
df = df.asfreq('W-SUN', fill_value='del')
```

We can now stack the dataset again and we are almost done.

```python
df = df.stack().sort_index(level=1).reset_index()
```

The dataset now has the correct representation of weeks for all positions, but the locked values are only present in the weeks where a transaction happened. So we need to forward fill values for each position. As we have used 'del' as fill value 0s will be forward filled correctly. But we need to replace 'del' with np.nan so we can use the ffill/bfill method.

```python
df = df.replace('del', np.nan)
```

```python
df['LOCKED_COLLATERAL'] = df.groupby('POSITION_ID')['LOCKED_COLLATERAL'].ffill()
df['LOCKED_DEBT'] = df.groupby('POSITION_ID')['LOCKED_DEBT'].ffill()
```

We can now see the collateral and debt status of each position at the end of any given week.

You might want to filter out positions with 0 collateral value locked as these make the dataset unnecessaryly big. However, the first 0 occuring after a week with collateral locked tells us that the position was closed in this week, so this week has analytical importance for that position.