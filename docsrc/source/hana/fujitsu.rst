-> im Standard werden 4 Partitionen angelegt

create Tables
y04adm@hdb10y04-0001:/usr/sap/Y04/HDB10/exe/python_support> python fillHDB_vers3.py --host=lavdb10y04001 --port=31015 --user=SYSTEM --password=Pa55w0rd --maxTables=50 --tablePrefix=FTS_ --maxJobs=200 --recordCount=1000000

Delete Tables
python fillHDB_vers3.py --delete --host=lavdb10y04001 --port=31015 --user=SYSTEM --password=Pa55w0rd --tablePrefix=FTS_



