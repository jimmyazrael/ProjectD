Cooking Postgresql
==================

All kinds of litte tricks.

.. highlight:: psql

Create Autoincre Sequence
-------------------------

.. code-block:: psql

	CREATE SEQUENCE memotender_pk_seq;  /* Note thers's no quote here */
	ALTER TABLE memotender ALTER pk SET DEFAULT NEXTVAL('memotender_pk_seq');

Although any name for the sequence can be used, convention is to use 
``TABLE_NAME + COLUMN_NAME + 'seq'``.

Listing All Triggers in a DB
----------------------------

.. code-block:: psql

	select trg.tgname,
        CASE trg.tgtype::integer & 66
            WHEN 2 THEN 'BEFORE'
            WHEN 64 THEN 'INSTEAD OF'
            ELSE 'AFTER'
        end as trigger_type,
       	case trg.tgtype::integer & cast(28 as int2)
         when 16 then 'UPDATE'
         when 8 then 'DELETE'
         when 4 then 'INSERT'
         when 20 then 'INSERT, UPDATE'
         when 28 then 'INSERT, UPDATE, DELETE'
         when 24 then 'UPDATE, DELETE'
         when 12 then 'INSERT, DELETE'
       	end as trigger_event,
       	ns.nspname||'.'||tbl.relname as trigger_table,
       	obj_description(trg.oid) as remarks,
         case
          when trg.tgenabled='O' then 'ENABLED'
            else 'DISABLED'
        end as status,
        case trg.tgtype::integer & 1
          when 1 then 'ROW'::text
          else 'STATEMENT'::text
        end as trigger_level
	FROM pg_trigger trg
	 JOIN pg_class tbl on trg.tgrelid = tbl.oid
	 JOIN pg_namespace ns ON ns.oid = tbl.relnamespace
	WHERE trg.tgname not like 'RI_ConstraintTrigger%'
	  AND trg.tgname not like 'pg_sync_pg%';

