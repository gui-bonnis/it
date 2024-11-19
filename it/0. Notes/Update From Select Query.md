How to create an update from a select

###### ChatGPT said:

ChatGPT

To create an update from a select in SQL, you typically use a subquery in the `UPDATE` statement. This allows you to update the values in a table based on the results of a `SELECT` query. The syntax can vary slightly depending on the SQL dialect you're using, but the general approach is consistent.

Here is a general template:

sql

Copy code

`UPDATE target_table SET column1 = subquery.column1,     column2 = subquery.column2,     ... FROM (SELECT column1, column2, ...       FROM source_table       WHERE condition) AS subquery WHERE target_table.id = subquery.id;`

In PostgreSQL, for example:

sql

Copy code

`UPDATE employees SET salary = subquery.new_salary FROM (SELECT id, salary * 1.1 AS new_salary       FROM employees       WHERE department = 'Sales') AS subquery WHERE employees.id = subquery.id;`

In this example:

- `employees` is the table being updated.
- The `subquery` calculates the new salary for employees in the 'Sales' department by increasing their current salary by 10%.
- The `FROM` clause is used to reference the `subquery` in the `UPDATE` statement.
- The `WHERE` clause ensures that the update only affects the rows in the `employees` table that match the IDs from the `subquery`.

If you're using Liquibase to manage database changes, you can write this SQL in a `<sql>` tag within a `<changeSet>` in your Liquibase changelog file.

Let me know if you need a more specific example or have a different use case in mind!