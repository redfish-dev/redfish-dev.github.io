---
layout: post
title:  "NetSuite SquiteQL columns to text via LISTAGG"
date:   2021-12-05 15:23:10 -0600
categories: Netsuite SuiteQL
---

If you want to see all item fulfilments for a sales order on one line versus a row for each:

In our example Sales Order 2193 has Items fulfilled on IF 47 and IF 71.
Starting with 

SELECT T.tranid
	FROM Transaction T
	INNER JOIN TransactionLine TL ON (TL.Transaction = T.ID)
	WHERE T.Type = 'ItemShip'  and TL.CreatedFrom=2193
	group by T.tranid

We will get 2 rows
IF47
IF71

Now we want these in one text result in one row.
We will use Oracle’s Lis Aggregator function LISTAGG

https://docs.oracle.com/cd/E11882_01/server.112/e41084/functions089.htm#SQLRF30030

We can wrap the sample query above as follows:

SELECT LISTAGG (T2.tranid, ', ')
WITHIN GROUP (ORDER BY T2.tranid ) "tranid"
from
(
	SELECT T.tranid
	FROM Transaction T
	INNER JOIN TransactionLine TL ON (TL.Transaction = T.ID)
	WHERE T.Type = 'ItemShip'  and TL.CreatedFrom=2193
	group by T.tranid
) as T2

Now the result is “IF47, IF71”
