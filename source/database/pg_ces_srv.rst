CESPOS Server Postgressql Operations
====================================

Database Replication with londiste3
-----------------------------------

Refer to the doc for the installation guide and simple HOWTO: 
https://github.com/markokr/skytools/tree/master/doc

Root Node
^^^^^^^^^

1. Dump DB schema
*****************

.. code-block:: bash

		pg_dump -h localhost -U username -s RetailPOSLaurenServer -f RetailPOSLaurenServer_schema

or just dump everything and use ``pg_restore`` on the leaf node.

2. Create Node and Ticker
*************************
Refer below, as this time I will just run everything on the leaf node to alleviate the old machine's load.

.. attention::
	Remember to update ``pg_hba.conf`` for leaf node connection.

Leaf Node
^^^^^^^^^

1. Create Database and Restore from File
****************************************

.. code-block:: psql

	CREATE DATABASE RetailPOSLaurenServer TEMPLATE template0;

.. code-block:: bash

	pg_restore -d RetailPOSLaurenServer -U username -h localhost -c -v RetailPOSLaurenServer.sql.tar

2. Create Root Node
*******************
Create a file ``retailpos_master.ini``:

.. code-block:: ini

	[londiste3]
	job_name = retailpos_master
	db = dbname=RetailPOSLaurenServer host=pos.regalglory.com user=username
	public_node_location = dbname=RetailPOSLaurenServer host=pos.regalglory.com user=username
	queue_name = replika
	parallel_copies = 8
	logfile = /opt/londiste/log/retailpos_master.log
	pidfile = /opt/londiste/pid/retailpos_master.pid

.. code-block:: bash

	londiste3 retailpos_master.ini create-root node_root

3. Create Leaf Node
*******************
Create a file ``retailpos_leaf.ini``:

.. code-block:: ini

	[londiste3]
	job_name = retailpos_leaf
	db = dbname=RetailPOSLaurenServer host=127.0.0.1 user=username password=password
	public_node_location = dbname=RetailPOSLaurenServer host=139.196.220.211 user=username
	initial_provider_location = dbname=RetailPOSLaurenServer host=pos.regalglory.com user=username password=password
	queue_name = replika
	logfile = /opt/londiste/log/retailpos_leaf.log
	pidfile = /opt/londiste/pid/retailpos_leaf.pid

.. attention::
	Again, remember to modify ``pg_hba.conf`` on this machine for localhost connection as well. Or you WILL definitely hit the wall at the speed of 250 km/h. You will see running the command below. Now enjoy the frustration!

Now you may create the leaf node:

.. code-block:: bash

	londiste3 retailpos_leaf.ini create-leaf node_leaf

4. Setup and Start the Ticker:
******************************
Create a file ``pgqd.ini``:

.. code-block:: ini

	[pgqd]
	base_connstr = host=pos.regalglory.com user=username
	initial_database = lontest
	logfile = /opt/londiste/log/pgqd.log
	pidfile = /opt/londiste/pid/pgqd.pid

Start it as a daemon:

.. code-block:: bash
	
	pgqd -d pgqd.ini

5. Start Both Nodes and Add Tables
**********************************

.. code-block:: bash

	londiste3 -d retailpos_master.ini worker
	londiste3 -d retailpos_leaf.ini worker 

Now add the tables. If you're feeling lucky, just use:

.. code-block:: bash

	londiste3 retailpos_master.ini add-table --all
	londiste3 retailpos_leaf.ini add-table --all

Usually you're not that lucky, or you won't be here reading boring words from me.
Tables without primary key CANNOT be added and will terminate the above process.

To manually add them one by one seems tedious. Try this to save some of your life for you to 
waste later:

First select out all the table names and paste them in a file named ``whatever.txt`` LINE BY LINE. Then:

.. code-block:: bash

	cat whatever.txt | while read line; do londiste3 retailpos_master.ini add-table $line; done;

Brutally remove a node
*************************

If for some mysterious reason, londiste3 drop-node doesn't work, meanwhile, if anything goes wrong and you are blaming the nodes and are threatening to kill them, then the brutal way is the only way:

.. code-block:: psql

	drop schema londiste cascade;
	drop schema pgq      cascade;
	drop schema pgq_ext  cascade;
	drop schema pgq_node cascade;   
	
Peacefully remove a node
***************************

Thanks for keeping on reading and not being tempted by raw brutality. It means you're quite a peaceful person. Let's end the nodes' life peacefully, with style:

.. code-block:: bash

	londiste3 node_configuration.ini drop-node node_name


Grant All Privileges On All Tables in PG 8.3
--------------------------------------------

It's stupid. It's meaningless. It's painful.
Who use 8.3 these days?!

Anyway, here's how:

.. code-block:: psql

	GRANT ALL PRIVILEGES ON DATABASE lontest to username;

Get all the table names here:

.. code-block:: psql

	SELECT array(
	 SELECT table_schema::text ||'.'||table_name::text 
	 FROM information_schema.tables 
	 WHERE table_schema = 'erp' OR table_schema = 'public' 
	 ORDER BY 1);

Then **MANUALLY** copy and paste in the command below:

.. code-block:: psql

	GRANT ALL PRIVILEGES ON TABLE paste_list_here TO username;

Info on the Table ``stocktransaction``
--------------------------------------
=========	=========	=======
typevalue	flowvalue	fomtype
=========	=========	=======
2			1			PD
2			1			PIR
5			2			PR
1			2			SA
1			2			SR
4			1			SR
1			2			SX
4			1			SX
6			3			TA
0			0			TB
8			3			TC
3			3			TI
3			3			TO
5			2			TR
12			2			WD
12			2			WID
12			2			WR
=========	=========	=======
