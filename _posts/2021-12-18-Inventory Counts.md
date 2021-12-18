---
layout: post
title:  "Item Counts in SuiteQL such as QuantityAvailable"
date:   2021-12-18 06:21:10 -0600
categories: Netsuite SuiteQL
---

<script async src="https://www.googletagmanager.com/gtag/js?id=G-T43W5QQ2KS"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-T43W5QQ2KS');
</script>



Depending on your account settings, some fields such as quantityavailable may have moved off of the item table and may be found in itemCounts .

If you are getting "Field 'QuantityAvailable' for record 'item' was not found. Reason: REMOVED - Field is removed" you might find your counts on the inventoryitemlocations record.

In order to get a count of available accross multiple locations you need to pull from inventoryitemlocations.

To get a sum across all inventory locations:

{% highlight sql %}

Select 
	Item.itemid, 
	Item.description,   
	QA as "Avail", 
	QC as "Committed", 
	QOH as "OnHand", 
	QOO as "OnOrder"
	from item left outer join
		(
				SELECT 
					item, 
					SUM(quantityavailable) as QA, 
					Sum(QuantityCommitted) as QC, 
					Sum(QuantityOnHand) as QOH, 
					SUM(QuantityOnOrder) as QOO 
					FROM inventoryitemlocations group by item
		) as itemCounts
		on (itemCounts.item = item.id)
	order by Item.itemid

{% endhighlight %}

