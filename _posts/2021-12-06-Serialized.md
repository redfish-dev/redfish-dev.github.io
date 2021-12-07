---
layout: post
title:  "Sales Orders with Serialized Inventory Shipped in Last 30 days"
date:   2021-12-06 15:23:10 -0600
categories: Netsuite SuiteQL
---

<script async src="https://www.googletagmanager.com/gtag/js?id=G-T43W5QQ2KS"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-T43W5QQ2KS');
</script>



I had a request to see all sales orders where a serialized inventory item was shipped. The client wanted to see the serial numbers shipped by Sales Order.

You can add any field you want to see such as item description, quantity, etc.

We must first join an Item Fulfillment to Inventory Assignment and then to an Inventory Number.

From the Fulfillment Line Items we can get to the sales order it was ‘createdfrom’.



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

