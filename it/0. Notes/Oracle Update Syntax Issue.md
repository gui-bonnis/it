Why this update is not working in oracle UPDATE tes.titpa SET RGM_ALUN = subquery.RGM_ALUN FROM (SELECT tadp.rgm_alun, tadp.ID_TITPA, tadp.NR_TITUL FROM tes.titpa tadp where tadp.cod_inst = 49 AND tadp.tipo_tit = 2 AND tadp.rgm_alun = 18322531 AND tadp.NR_TITUL = '24102/06' ) AS subquery WHERE subquery.ID_TITPA = titpag.ID_TITPA;

###### ChatGPT said:

ChatGPT

In Oracle, the `UPDATE ... FROM` syntax is not supported. Unlike some other SQL databases like PostgreSQL, Oracle has a different syntax for updating a table based on a subquery.

You can achieve the same effect using a `MERGE` statement or a correlated subquery in Oracle.

Here's how you can rewrite your query using a `MERGE` statement:

sql

Copy code

`MERGE INTO tes.titpa tgt USING (SELECT tadp.rgm_alun, tadp.ID_TITPA, tadp.NR_TITUL        FROM tes.titpa tadp        WHERE tadp.cod_inst = 49          AND tadp.tipo_tit = 2          AND tadp.rgm_alun = 18322531          AND tadp.NR_TITUL = '24102/06'       ) src ON (tgt.ID_TITPA = src.ID_TITPA) WHEN MATCHED THEN   UPDATE SET tgt.RGM_ALUN = src.RGM_ALUN;`

Alternatively, using a correlated subquery, it would look like this:

sql

Copy code

`UPDATE tes.titpa tgt SET tgt.RGM_ALUN = (SELECT src.RGM_ALUN                     FROM tes.titpa src                     WHERE src.cod_inst = 49                       AND src.tipo_tit = 2                       AND src.rgm_alun = 18322531                       AND src.NR_TITUL = '24102/06'                       AND src.ID_TITPA = tgt.ID_TITPA) WHERE EXISTS (SELECT 1               FROM tes.titpa src               WHERE src.cod_inst = 49                 AND src.tipo_tit = 2                 AND src.rgm_alun = 18322531                 AND src.NR_TITUL = '24102/06'                 AND src.ID_TITPA = tgt.ID_TITPA);`

Both approaches should work in Oracle. The `MERGE` statement is more straightforward and is often preferred for clarity.