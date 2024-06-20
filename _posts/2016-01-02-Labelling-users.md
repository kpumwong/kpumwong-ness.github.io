---
layout: post
title: Programming decision tree to label users
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
This example shows how I am labbeling users based on a number of criteria defined in a decision tree. This is to be able to determine whether they are active or inactive week by week and makes it possible to chart the developement over time.

## Introduction

It is important to get an idea about how many users are active or inactive from week to week. In DeFi a user will probably just be defined as a wallet address. A real person might have multiple wallets, but we can never know this if there has been no KYC done, due to the annonymous nature of the blockchain technology.

We might want to define some crietia for when we determine a user to be active or inactive. Because if a user stops using the platform the data on the blockchain will not tell us anything directly. Their wallet will exist forever and they might even decide to start using it again after a year of inactivity.

## Desition tree

How to define whether a user is active or inactive is an arbitrary business decision. Depending on the products offered to the users this definition might not be the same.

To make it simple and just to explain the overall concept imagine we want to define an active user as someone show has some collateral locked value and has done at least 1 transaction in the last 3 months.

## Data preparation

I will not go through all the steps needed to prepare the dataset. We can imagine we already have a weekly state dataset prepared, so we know how much value locked a user has week by week. Much like this example here:
We should also know whether the user had any collateral volume (absolute value of deposits and withdraws). The user might deposit 10 ETH and withdraw 10 ETH in the same week to bet on some short term volatility. Even though by the end of the week the users has no locked colleral it was still active in this week.

We then need to prepare some new columns with specific attibutes that can help us determine if the user is active or inactive in each week.
For example we can calulate a rolling count of the transaction each user has performed over the last 90 days.

```python
ut["TX_COUNT_90"] = ut.groupby(['USER', 'DATE_WEEK'])["ANY_TX_COUNT"].transform(lambda x: x.rolling('90D', 1).sum())
```

More attributes will probably be needed, but let us keep the example simple.

Now we can define some logic resulting in a True or False boolean Series. For example a user has had some value in a given week if it had either som locked value or collateral volume.


```python
value = ((ut['COLLATERAL_VALUE_LOCKED_USD'] > 0) | (ut['ABS_DEPOSIT_WITHDRAW_USD'] != 0))
no_value = ((ut['COLLATERAL_VALUE_LOCKED_USD'] == 0) & (ut['ABS_DEPOSIT_WITHDRAW_USD'] == 0))
txs = (ut['ANY_TX_COUNT_90'] > 0)
no_txs = (ut['ANY_TX_COUNT_90'] == 0)
first_tx = (ut['FIRST_USER_TX'] == 1)
not_first_tx = (ut['FIRST_USER_TX'] == 0)
```


To create a final "Active user" column we can use the above logiv in a straight forward way like so.

```python
ut['ACTIVE_USER'] = np.where(value & txs, True, False)
ut['ACTIVE_USER'] = np.where(first_tx, True, ut['ACTIVE_USER'])
```

Both for charting and further labelling it is good to have a lagging indicator of the active user column.

```python
ut['ACTIVE_USER_LAST_WEEK'] = ut.groupby(['OWNER', 'PERIOD'])['ACTIVE_USER'].transform('shift')
```

Let us define more logic around this lagging indicator.

```python
not_active_last_week = (ut['ACTIVE_USER_LAST_WEEK'] == False)
active_last_week = (ut['ACTIVE_USER_LAST_WEEK'] == True)
```

## User labels more granularity

We can also define some labels with more granularity. It can be interesting to see what caused the user to become active or inactive in any given week. Is it a new user, continuous active user, reactivated user, closed user, idle user.
As you can see we can use the lagging indicator whether the user was active last week to see if there was a change in label. For example if the user was active last week, but closed its position this week we can label it as "NEW Closed user". This change in user behavior is interesting for analytics so we can look into why this might have happened.

```python
ut['USER_LABEL'] = np.where(value & txs not_active_last_week, 'Reactivated user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(value & no_txs & not_active_last_week, 'Reactivated user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(value & txs & active_last_week, 'Continuous user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(value & no_txs & active_last_week, 'Continuous user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(first_tx, 'New Oasis user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(no_value, 'Closed user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(no_value & active_last_week, 'NEW Closed user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(no_txs, 'Idle user', ut['USER_LABEL'])
ut['USER_LABEL'] = np.where(no_txs & active_last_week, 'NEW Idle user', ut['USER_LABEL'])
```