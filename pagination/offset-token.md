
On Using Offset Token vs Numeric Offset in Pagination Queries
=============================================================

# The Problem

It is not possible to provide efficient offset-limit queries in all the cases.
Sometimes it is possible to create an index however that index might drastically slow
down insert operations.

For all the queries like ``SELECT /*... fields ...*/ FROM /*...*/ WHERE /*...*/ OFFSET n LIMIT m`` large ``n`` might result in pretty expensive execution plan. This happens because queries like that, albeit optimized, still result in selecting ``m+n`` entities just to get n-th element.

## The Workaround

Use specialized offset token built using specific field or combination of fields in the query.

Ex.: consider query:

```sql
SELECT name FROM some_table WHERE foo=X ORDER BY bar
```

If ``bar`` field is unique in such a query (and corresponding index exists for ``foo`` and ``bar`` fields) the following can be used as an alternative to the ``offset-limit`` query:

```sql
SELECT name FROM some_table WHERE foo=X AND bar=PREVIOUS_VALUE_OF_BAR ORDER BY bar LIMIT m
```

The query above uses previous value of ``bar`` fetched by previous select. For the first select that value would be ``NULL``.

This query combines efficiency (as database will use index to 'jump' to the given value of ``bar``) and still provides offset-limit-alike functionality.

The drawback is obvious - we can't jump to the arbitrary offset. But this is rarely needed in the real UI as users often view only very first fetched pages.

