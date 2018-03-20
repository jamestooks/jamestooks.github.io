---
layout: post
title:  "MySQL: Delete duplicates"
date:   2018-03-19 13:43:31 +0000
categories: mysql
tags: mysql
blurb: How to remove duplicates from a database table
---

I needed to add a unique key to a column on an existing table.  When I tried to do so, it failed because the values in that column were not unique.  After looking at the data, I could simply remove any duplicates before adding my unique key.

The query to run is below:

```mysql
DELETE t FROM table AS t
LEFT JOIN (
    SELECT MAX(updated_at) AS max_updated_at, name
    FROM table
    GROUP BY name
) AS tmp
ON t.updated_at = tmp.max_updated_at
AND t.name = tmp.name
WHERE tmp.max_updated_at IS NULL
```

This query works by retrieving the last updated version of each row that needs to be unique.  This is because we will only keep the most recently updated rows.  The left join means that all rows will be analysed for deletion and the WHERE clause ensures we only delete the rows that do not match the most recently updated ones.
