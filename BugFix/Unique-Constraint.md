# ISSUE FACED
Table lacking unique constraint  

`` 
Postgres rejected the INSERT because your INSERT ... ON CONFLICT (date,shift) clause names a conflict target that does not exist — there is no UNIQUE constraint or unique index on (date, shift), so Postgres can't know how to detect a conflict. The SQLSTATE 42P10 means "invalid ON CONFLICT target". 
``
  
`` What ON CONFLICT expects INSERT ... ON CONFLICT (col1, col2) DO ... requires that (col1, col2) be protected by a UNIQUE constraint or UNIQUE INDEX (or an exclusion constraint). That index is what Postgres uses to detect a "conflict" (a row with the same key). If no such index/constraint exists, Postgres throws 42P10. Why it happened in your migration The migration that creates the table did not add a UNIQUE index or constraint on ("date","shift"). 
``
  
`` 
Later migration(s) that bulk-load CSV rows use INSERT ... ON CONFLICT (date,shift) DO NOTHING to avoid duplicate loads. Because the unique constraint was missing, Postgres refused to run the ON CONFLICT load — it can't perform conflict checks without an index. Consequences
``

## The fix
``
CREATE UNIQUE INDEX IF NOT EXISTS idx_production_date_shift_unique
ON public.production_summary ("date", "shift");
``
The command creates a **unique index** on the columns `(date, shift)` in the `production_summary` table. This means Postgres will now enforce that each combination of `date` and `shift` appears only once in the table.  
With this unique index in place, Postgres can use it to detect when an `INSERT` tries to add a duplicate `(date, shift)` pair. As a result, the `INSERT ... ON CONFLICT (date, shift) DO NOTHING` statement can now correctly identify conflicts and skip inserting duplicates, instead of failing with an error.
