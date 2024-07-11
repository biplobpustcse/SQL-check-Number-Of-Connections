# SQL-check-Number-Of-Connections

```
--SQL check Number Of Connections 
SELECT 
    DB_NAME(dbid) as DBName, 
    COUNT(dbid) as NumberOfConnections,
    loginame as LoginName
FROM
    sys.sysprocesses
WHERE 
    dbid > 0 
	AND DB_NAME(dbid)='TestDB'--database name
GROUP BY 
    dbid, loginame
```
