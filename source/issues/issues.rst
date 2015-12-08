Issues
=======
Q1. I'm getting the error message ``sqlite3.OperationalError: database is locked`` when running the generated pipeline on a cluster.

S1. You need to set ``DEFAULT_RUFFUS_HISTORY_FILE`` to null:

	.. code-block:: bash

		export DEFAULT_RUFFUS_HISTORY_FILE=''
		
