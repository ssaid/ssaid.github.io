---
layout: post
title: Identifying and Solving Performance Issues with py-spy
categories:
- linux
- odoo
- postgres
tags:
- tools
- py-spy
---
In today's blog post, we will explore the powerful utility called py-spy, which allows developers to dive deep into the stacktrace and effortlessly identify the root causes of performance issues. We'll discuss how to approach problems using py-spy and showcase its effectiveness in introspecting both Python code and the database. Let's get started!

## How to Approach the Problem?

### Introspecting Python Code
When encountering an issue, it's crucial to determine whether the problem lies within the database or the code. With py-spy, you can easily perform an introspection of the Python stacktrace. Here's an example command you can use:

```bash
sudo env "PATH=$PATH" py-spy top --pid 1716361
```

The output will provide valuable insights into the execution flow, enabling you to pinpoint potential bottlenecks. For instance:

```
  0.00%   0.00%   0.000s     3278s   _call_kw_multi (odoo/api.py)
  0.00%   0.00%   0.070s     3278s   action_confirm (logistic_transport/models/micrologistic_settlement.py)
  0.00%   0.00%   0.000s     3278s   call_button (web/controllers/main.py)
  0.00%   0.00%   0.890s     3194s   write_full (auditlog/models/rule.py)
  0.00%   0.00%    1.12s     2938s   wrapper (odoo/sql_db.py)
  0.00%   0.00%    2924s     2935s   execute (odoo/sql_db.py)
  0.00%   0.00%   10.85s     2828s   read (odoo/models.py)
  0.00%   0.00%   10.76s     2759s   _read_from_database (odoo/models.py)
```

From the above example, we can infer that there is a problem with the `read` method. Let's proceed to the next step to gain further insights.

### Introspecting the Database
To analyze the database performance, we can leverage the versatile tool known as `pg_stat_statements` from PostgreSQL. Here's an example SQL query:

```sql
devel=# SELECT query, calls, total_exec_time, rows, 100.0 * shared_blks_hit /
               nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
          FROM pg_stat_statements ORDER BY total_exec_time DESC LIMIT 5;
```

Executing the query above will provide valuable information about the most time-consuming queries, allowing you to identify potential bottlenecks. For instance, you might discover queries that are executed excessively, resulting in full reads. 

In your case, the issue is related to the `auditlog` module in Odoo, which traces changes made to objects. When your method interacts with a large number of objects, it can lead to performance degradation. In such scenarios, it's essential to dig deeper into the `auditlog` code and find a context flag that can be utilized to disable it selectively.

Here's an example output after taking appropriate measures:

```
  0.00%   0.00%   0.000s    461.2s   _call_kw_multi (odoo/api.py)
  0.00%   0.00%   0.000s    461.2s   _call_kw (web/controllers/main.py)
  0.00%   0.00%   0.000s    461.0s   call_button (web/controllers/main.py)
  0.00%   0.00%   0.060s    461.0s   action_confirm (logistic_transport/models/micrologistic_settlement.py)
```

## Conclusion
In this blog post, we introduced the powerful utility py-spy, which empowers developers to analyze stacktraces and identify performance bottlenecks effortlessly. We discussed the approach to problem-solving by inspecting both Python code and the database using py-spy. By leveraging this tool effectively, you can optimize your application's performance and enhance the user experience.

We hope this article provides valuable insights into using py-spy and enables you to tackle performance issues with confidence. Happy coding!

Stay tuned for more informative blog posts on optimizing Python applications.

## Resources
* https://github.com/benfred/py-spy
* https://www.postgresql.org/docs/current/pgstatstatements.html
