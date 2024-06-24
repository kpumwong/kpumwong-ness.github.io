---
layout: post
title: Programming decision tree to label users
---

This example shows how I am labeling users based on a number of criteria defined in a decision tree. This is to be able to determine whether the users are active or inactive week by week and makes it possible to chart the developement over time.

## Introduction

The difinition of a user in DeFi will probably just be a wallet address. A real person might have multiple wallets, but we can never know this if there has been no KYC done, due to the annonymous nature of the blockchain technology.

We have want to define some arbitrary crietia for when we determine a user to be active or inactive based on the nature of the business. If a user stops using the platform the data on the blockchain will not tell us anything directly. Their wallet will exist forever and they might even decide to start using it again after a year of inactivity.

## Desition tree

The concept of a decision tree can be seen below.

<img src="/images/Desicion_tree/decision_tree.png" alt="Decision Tree" class="fit image">

Imagine we use this structure to determine whether a user is active or inactive.
As a very simple example, let us we first ask whether the user has some collateral locked and then whether it has done at least 1 transaction in the last 3 months. If we can say yes to both the user is active.
If it has no collateral locked it will drop out of the decision tree immediately be defined as inactive and labelled as "Closed". If the user HAS locked collateral, but no transactions in the last 3 months it is also inactive and further label it as "Idle".


## Data preparation

I will not go through the preparation of the data except for one little example. We can imagine we already have a weekly state dataset prepared, so we know how much value locked a user has week by week. Much like this example here:
<a href="https://kasper-birkelund.github.io/2016/01/03/Calculating-a-weekly-wallet-state-dataset/" target="_blank">Calculating a weekly wallet state dataset</a>

We should also know whether the user had any collateral volume (absolute value of deposits and withdraws). The user might deposit 10 ETH and withdraw 10 ETH in the same week. Even though by the end of the week the users has no locked colleral, it was still active in this week.

We then need to prepare some new columns with specific attibutes, that can help us determine if the user is active or inactive in each week.
For example we can calulate a rolling count of the transaction each user has performed over the last 90 days.

```python
df["TX_COUNT_90"] = df.groupby(['USER', 'DATE_WEEK'])["ANY_TX_COUNT"].transform(lambda x: x.rolling('90D', 1).sum())
```

More attributes will probably be needed, but let us keep the example simple.

## Defining logic for the decision nodes

Now we can define some logic resulting in a True or False boolean Series corresponding to the decision tree nodes.

```python
value = ((df['COLLATERAL_VALUE_LOCKED_USD'] > 0) | (df['ABS_DEPOSIT_WITHDRAW_USD'] != 0))
no_value = ((df['COLLATERAL_VALUE_LOCKED_USD'] == 0) & (df['ABS_DEPOSIT_WITHDRAW_USD'] == 0))
txs = (df['ANY_TX_COUNT_90'] > 0)
no_txs = (df['ANY_TX_COUNT_90'] == 0)
```

To create the column determining whether the user is active or not ("Active user" column), we can use the above logic in a straight forward way like so.

```python
df['ACTIVE_USER'] = np.where(value & txs, True, False)
```

## Indicator for user activity last week

Both for charting and further labelling it is good to have a lagging indicator of the active user column.

```python
df['ACTIVE_USER_LAST_WEEK'] = df.groupby(['OWNER', 'PERIOD'])['ACTIVE_USER'].transform('shift')
```

Let us also define some logic around this lagging indicator, that we can use in the next step.

```python
not_active_last_week = (df['ACTIVE_USER_LAST_WEEK'] == False)
active_last_week = (df['ACTIVE_USER_LAST_WEEK'] == True)
```

## first transaction

In the next step we also should have some logic on when the users performed their first transaction.

```python
first_tx = (ut['FIRST_USER_TX'] == 1)
not_first_tx = (ut['FIRST_USER_TX'] == 0)
```

## User labels for more granularity

It can be interesting to see what caused the user to become active or inactive in any given week. Is it a new user, continuous active user, reactivated user, closed user, idle user?
We can now use the logic created above to do this as so.

```python
df['USER_LABEL'] = np.where(value & txs not_active_last_week, 'Reactivated user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(value & no_txs & not_active_last_week, 'Reactivated user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(value & txs & active_last_week, 'Continuous user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(value & no_txs & active_last_week, 'Continuous user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(first_tx, 'New Oasis user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(no_value, 'Closed user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(no_value & active_last_week, 'NEW Closed user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(no_txs, 'Idle user', df['USER_LABEL'])
df['USER_LABEL'] = np.where(no_txs & active_last_week, 'NEW Idle user', df['USER_LABEL'])
```

For example if a user was active in any given week, but closed its position the week after, we can label it as "NEW Closed user" in the specific week. These labels keep track of the users' behavior week by week and is interesting for analytics.