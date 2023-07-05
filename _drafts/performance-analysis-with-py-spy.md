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

### Checking with the odoo profiler
```
modified: models/micrologistic_settlement.py
@ models/micrologistic_settlement.py:24 @
...
from odoo.tools.profiler import profile


class MicrologisticSettlement(models.Model):
@ models/micrologistic_settlement.py:106 @ class MicrologisticSettlement(models.Model):
            )
        return action

    @profile
    def undo_settling(self):
        for settlement in self:
            for picking in settlement.picking_ids:
@ models/micrologistic_settlement.py:125 @ class MicrologisticSettlement(models.Model):
        return True

```

```
odoo_1                        | 2023-07-05 14:50:06,187 1 INFO devel odoo.tools.profiler:
odoo_1                        | calls     queries   ms
odoo_1                        | micrologistic.settlement --------------------- /opt/odoo/auto/addons/logistic_transport/models/micrologistic_settlement.py, 106
odoo_1                        |
odoo_1                        | 1         0         0.01          @profile
odoo_1                        |                                   def undo_settling(self):
odoo_1                        | 3         0         0.03              for settlement in self:
odoo_1                        | 26        25        408.55                for picking in settlement.picking_ids:
odoo_1                        | 24        0         0.08                      picking.write(
odoo_1                        |                                                   {
odoo_1                        | 24        0         0.07                              "micrologistic_amount": 0,
odoo_1                        | 24        0         0.09                              'carry_rates': '',
odoo_1                        | 24        319       44426.83                          'rates_values': 0,
odoo_1                        |                                                   }
odoo_1                        |                                               )
odoo_1                        | 1         0         0.0                   settlement.write(
odoo_1                        |                                               {
odoo_1                        | 1         1         2.9                           "total_amount": 0,
odoo_1                        |                                               }
odoo_1                        |                                           )
odoo_1                        | 1         0         0.0               return True
odoo_1                        |
odoo_1                        | Total:
odoo_1                        | 1         345       44838.56
```

```
odoo_1                        | 2023-07-05 15:24:01,817 1 INFO devel odoo.tools.profiler:
odoo_1                        | calls     queries   ms
odoo_1                        | micrologistic.settlement --------------------- /opt/odoo/auto/addons/logistic_transport/models/micrologistic_settlement.py, 134
odoo_1                        |
odoo_1                        | 1         0         0.01          @api.multi
odoo_1                        |                                   @profile
odoo_1                        |                                   def action_draft(self):
odoo_1                        | 1         2         43.97             self.undo_settling()
odoo_1                        | 1         0         0.01              q1 = "UPDATE stock_picking SET microsettlement_id=NULL WHERE microsettlement_id IN %(ids)s"
odoo_1                        | 1         0         0.0               q_p = {'ids': self._ids}
odoo_1                        | 1         1         45.55             self._cr.execute(q1, q_p)
odoo_1                        | 1         0         0.01              q2 = "DELETE FROM micrologistic_settlement_route_line WHERE microsettlement_id IN %(ids)s"
odoo_1                        | 1         1         0.45              self._cr.execute(q2, q_p)
odoo_1                        |                                       # self.picking_ids.write({"microsettlement_id": False})
odoo_1                        |                                       # self.route_lines.unlink()
odoo_1                        | 1         0         0.0               q3 = "UPDATE micrologistic_settlement SET state='draft' WHERE id IN %(ids)s"
odoo_1                        | 1         1         0.47              self._cr.execute(q3, q_p)
odoo_1                        | 1         0         0.0               return True
odoo_1                        |
odoo_1                        | Total:
odoo_1                        | 1         5         90.48
```

## Conclusion
In this blog post, we introduced the powerful utility py-spy, which empowers developers to analyze stacktraces and identify performance bottlenecks effortlessly. We discussed the approach to problem-solving by inspecting both Python code and the database using py-spy. By leveraging this tool effectively, you can optimize your application's performance and enhance the user experience.

We hope this article provides valuable insights into using py-spy and enables you to tackle performance issues with confidence. Happy coding!

Stay tuned for more informative blog posts on optimizing Python applications.

## Resources
* https://github.com/benfred/py-spy
* https://www.postgresql.org/docs/current/pgstatstatements.html
