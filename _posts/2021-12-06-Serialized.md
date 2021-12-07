---
layout: post
title:  "Sales Orders with Serialized Inventory Shipped in Last 30 days"
date:   2021-12-05 15:23:10 -0600
categories: Netsuite SuiteQL
---


I had a request to see all sales orders where a serialized inventory item was shipped. The client wanted to see the serial numbers shipped by Sales Order.

You can add any field you want to see such as item desciption, quantity, etc.

We must first join an Item Fullfilment to Inventory Assignment and then to an Inventory Number.

From the Fullfilemnt Line Items we can get to the sales order it was 'createdfrom'.


{% highlight sql %}
Select so.trandisplayname as salesorder, so.id, IFS.inventorynumber as serialnumber, IFS.trandate as DateFulfilled from 
	(	
		select IF.tranid, IN.inventorynumber, IF.id, IF.trandisplayname, IF.trandate, IF.CreatedBy, IF.entity, TL.createdfrom
		FROM Transaction IF
		INNER JOIN TransactionLine TL ON (TL.Transaction = IF.ID)
		INNER JOIN inventoryAssignment IA on (IA.transactionline = TL.id and IA.Transaction = IF.ID)
		INNER JOIN inventoryNumber IN on  (IA.inventorynumber = IN.id)
		WHERE IF.type='ItemShip' 
		order by IF.trandate desc
	
	) as IFS
	INNER JOIN Transaction so on ( IFS.createdfrom = so.id)
	where IFS.trandate > sysdate-30
	order by IFS.trandate desc
{% endhighlight %}
