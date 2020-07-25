# Solution to The Assignment

### Question 1
1. SELECT COUNT(u_id) AS total_users FROM users;
   


### Question 2
SELECT
    COUNT(transfer_id) AS total_cfa_transfers
FROM
    transfers
WHERE
    send_amount_currency = 'CFA';
        

### Question 3
SELECT 
    COUNT(DISTINCT u_id)
FROM
    transfers
WHERE
    send_amount_currency = 'CFA';



### Question 4
SELECT 
    DATE_PART('month', when_created) AS atx_month, COUNT(atx_id) AS atx_total
FROM 
    agent_transactions 
WHERE 
    DATE_PART('year',when_created) = 2018 
GROUP BY atx_month;




#### Some notes on questions 5 to 7:
I understand that the "over the course of the last week" in the questions refer to data recorded in the
the last 7 days (1 week) as we have in the database. It is on this premise that I use the
WITH clause to get the data from the range and subsequently use it for the queries.

### Question 5
WITH last_wk_atx AS (
                    SELECT * FROM agent_transactions 
                    WHERE when_created BETWEEN 
                        ((SELECT MAX(when_created) FROM agent_transactions) - interval '6 day')
                          AND 
                        ((SELECT MAX(when_created) FROM agent_transactions) + interval '1 day')
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


 
### Question 6
WITH last_wk_atx AS (
                    SELECT * FROM agent_transactions 
                    WHERE when_created BETWEEN 
                        ((SELECT MAX(when_created) FROM agent_transactions) - interval '6 day')
                          AND 
                        ((SELECT MAX(when_created) FROM agent_transactions) + interval '1 day')
                   )     
SELECT
    agt.city city, COUNT(atx.atx_id) atx_volume
FROM
    last_wk_atx AS atx
JOIN 
    agents as agt
ON
    atx.agent_id = agt.agent_id
GROUP BY 
    agt.city 
ORDER BY 2 DESC;


### Question 7
WITH last_wk_atx AS (
                    SELECT * FROM agent_transactions 
                    WHERE when_created BETWEEN 
                        ((SELECT MAX(when_created) FROM agent_transactions) - interval '6 day')
                          AND 
                        ((SELECT MAX(when_created) FROM agent_transactions) + interval '1 day')
                   )
       
SELECT
    agt.country, agt.city, SUM(atx.amount) atx_volume
FROM 
    last_wk_atx AS atx
JOIN
    agents AS agt
ON
    atx.agent_id = agt.agent_id
GROUP BY
    agt.city, agt.country
ORDER BY 3 DESC;


#### Some notes on questions 8 & 9


### Question 8
SELECT
    w.ledger_location AS country, txfr.kind AS transfer_kind, SUM(txfr.send_amount_scalar) AS send_volume
FROM
    transfers txfr
JOIN
    wallets w
ON
    txfr.source_wallet_id = w.wallet_id
GROUP BY w.ledger_location, txfr.kind;



### Question 9
SELECT
    w.ledger_location AS country, txfr.kind AS transfer_kind, SUM(txfr.send_amount_scalar) AS send_volume,
    COUNT(txfr.send_amount_scalar) AS transaction_count, COUNT(DISTINCT txfr.u_id) AS unique_senders_number
FROM
    transfers txfr
JOIN
    wallets w
ON
    txfr.source_wallet_id = w.wallet_id
GROUP BY w.ledger_location, txfr.kind;


### Question 10
SELECT
    w.wallet_id, SUM(txfr.send_amount_scalar) total_transfers
FROM
    wallets w
JOIN
    transfers txfr
ON
    w.wallet_id = txfr.source_wallet_id
WHERE
    txfr.send_amount_scalar > 10000000 AND txfr.send_amount_currency = 'CFA'
GROUP BY  w.wallet_id;