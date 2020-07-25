# Solution to The Assignment

### Question 1
```sql
SELECT COUNT(u_id) AS total_users FROM users;
```


### Question 2
```sql
SELECT
    COUNT(transfer_id) AS total_cfa_transfers
FROM
    transfers
WHERE
    send_amount_currency = 'CFA';
```

### Question 3
```sql
SELECT 
    COUNT(DISTINCT u_id) AS cfa_users
FROM
    transfers
WHERE
    send_amount_currency = 'CFA';
```


### Question 4
```sql
SELECT 
    DATE_PART('month', when_created) AS atx_month, COUNT(atx_id) AS atx_total
FROM 
    agent_transactions 
WHERE 
    DATE_PART('year',when_created) = 2018 
GROUP BY atx_month;
```



#### Some notes on questions 5 to 7:
I understand that the "over the course of the last week" in the questions refer to data recorded in the
the last 7 days (1 week) as we have in the database. It is on this premise that I use the
WITH clause to get the data from the range and subsequently use it for the queries.

### Question 5
```sql
WITH last_wk_atx AS (
    SELECT * FROM agent_transactions 
    WHERE
        when_created > ((SELECT MAX(when_created) FROM agent_transactions) - interval '7 day')
            AND 
        when_created <= (SELECT MAX(when_created) FROM agent_transactions)
    ) 
SELECT
    COUNT(DISTINCT tb1.agent_id) total_net_depositors, COUNT(DISTINCT tb2.agent_id) total_net_withdrawers 
FROM 
    (SELECT 
        agent_id
     FROM
        last_wk_atx 
     GROUP BY 
        agent_id 
     HAVING SUM(amount) < 0) AS tb1, 
    (SELECT 
        agent_id 
     FROM 
        last_wk_atx
     GROUP BY
        agent_id
     HAVING SUM(amount) > 0) AS tb2;
```

#### Some Notes on questions 6 to 10 concerning the volume of transfers or transactions ,etc
For the volume of transactions I'm torn between finding the number of transactions with COUNT or SUM of the 'amount'.
So I used both.<br/> 
This is because transaction_count had been asked in Question 9 after the send volume was asked in Question 8
 
### Question 6
```sql
WITH last_wk_atx AS (
    SELECT * FROM agent_transactions 
    WHERE
        when_created > ((SELECT MAX(when_created) FROM agent_transactions) - interval '7 day')
            AND 
        when_created <= (SELECT MAX(when_created) FROM agent_transactions)
    )     
SELECT
    agt.city city, COUNT(atx.atx_id) atx_volume, SUM(atx.amount) atx_amount_volume
FROM
    last_wk_atx AS atx
JOIN 
    agents as agt
ON
    atx.agent_id = agt.agent_id
GROUP BY 
    agt.city 
ORDER BY 2 DESC, 3 DESC;
```

### Question 7
```sql
WITH last_wk_atx AS (
    SELECT * FROM agent_transactions 
    WHERE
        when_created > ((SELECT MAX(when_created) FROM agent_transactions) - interval '7 day')
            AND 
        when_created <= (SELECT MAX(when_created) FROM agent_transactions)
    ) 

SELECT
    agt.country, agt.city, COUNT(atx.atx_id) atx_volume, SUM(atx.amount) atx_amount_volume
FROM 
    last_wk_atx AS atx
JOIN
    agents AS agt
ON
    atx.agent_id = agt.agent_id
GROUP BY
    agt.city, agt.country
ORDER BY 3 DESC, 4 DESC;
```

#### Some notes on questions 8 & 9
I understand "the past week" to be the week before 'the last week' as recorded in the database.
Therefore if I'm querying the database, I use the interval from 14 days ago and 7 days ago relative to the (MAX day in the database), 

### Question 8
```sql
WITH past_wk_txfr AS (
	 SELECT * FROM transfers
     WHERE 
	 when_created > ((SELECT MAX(when_created) FROM transfers) - interval '14 day')
     AND 
     when_created <= ((SELECT MAX(when_created) FROM transfers) - interval '7 day')
)
SELECT
    w.ledger_location AS country, txfr.kind AS transfer_kind, SUM(txfr.send_amount_scalar) AS send_volume
FROM
    past_wk_txfr txfr
JOIN
    wallets w
ON
    txfr.source_wallet_id = w.wallet_id
GROUP BY w.ledger_location, txfr.kind;
```


### Question 9
I was torn between using the source_wallet_id or the u_id to compute the unique_senders. I went ahead to use the u_id anyways.
```sql
WITH past_wk_txfr AS (
	 SELECT * FROM transfers
     WHERE 
	 when_created > ((SELECT MAX(when_created) FROM transfers) - interval '14 day')
     AND 
     when_created <= ((SELECT MAX(when_created) FROM transfers) - interval '7 day')
)
SELECT
    w.ledger_location AS country, txfr.kind AS transfer_kind, SUM(txfr.send_amount_scalar) AS send_volume,
    COUNT(txfr.transfer_id) AS transaction_count, COUNT(DISTINCT txfr.u_id) AS unique_senders_number
FROM
    transfers txfr
JOIN
    wallets w
ON
    txfr.source_wallet_id = w.wallet_id
GROUP BY w.ledger_location, txfr.kind;
```

### Question 10
```sql
WITH last_month_txfr AS (
  SELECT * FROM transfers
     WHERE 
 when_created > ((SELECT MAX(when_created) FROM transfers) - interval '1 month')
     AND 
     when_created <= (SELECT MAX(when_created) FROM transfers)
)
SELECT
    w.wallet_id, SUM(txfr.send_amount_scalar) total_transfers
FROM
    wallets w
JOIN
    last_month_txfr txfr
ON
    w.wallet_id = txfr.source_wallet_id
WHERE
    txfr.send_amount_scalar > 10000000 AND txfr.send_amount_currency = 'CFA'
GROUP BY  w.wallet_id;
```