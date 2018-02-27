---
layout: post
title:  "MySQL: Selecting one random item from each category"
date:   2018-02-27 15:57:27 +0000
categories: mysql
tags: mysql
blurb: How to select just one random item from each category using only MySQL
---

I recently had a requirement to select a single item from each category in my database.  However, with each select I needed to select a random item.

Code first, explanation is underneath:

```mysql
SELECT * FROM (
    SELECT c.id AS category_id, c.category, i.id, i.name
    FROM categories c
    INNER JOIN items i ON c.id = i.category_id
    ORDER BY RAND()
) AS sorted_items
GROUP BY category_id
```

The randomisation is in a sub-query.  This is because a GROUP BY statement always happens before an ORDER BY if they are part of the same query.  If this happens, we would end up grouping by the category ID first which means we would not be able to randomly choose an item within that category.
