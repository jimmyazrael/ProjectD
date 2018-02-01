Aerospike
=========

Disable lua Caching for Developement ONLY
-----------------------------------------

It saves you the extra effort to manually refresh the cache each time
you update the lua file.

In ``aerospike.conf``,  add:

.. code-block:: py

	mod-lua {
	  cache-enabled false
	}

Do remember to change it to true for production.

Set Config at Run-time
----------------------

.. code-block:: sh

	asinfo -v "set-config:context=service;query-batch-size=10000"
