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

```
------------------------####### Show All Connections #######--------------------------
SELECT 
    spid,
    dbid,
    DB_NAME(dbid) as DBName,
    loginame as LoginName,
    hostname,
    program_name,
    status,
    last_batch
FROM
    sys.sysprocesses
WHERE 
    dbid > 0 
    AND DB_NAME(dbid)='CloudPOS_DB_GHORERBAZAR'
ORDER BY 
    last_batch DESC
```

```
------------------------####### Kill All Connections Except Your Own #######--------------------------
DECLARE @CurrentSessionID INT;
SELECT @CurrentSessionID = @@SPID;

DECLARE @killCommand VARCHAR(MAX) = '';

SELECT @killCommand = @killCommand + 'KILL ' + CONVERT(VARCHAR(5), session_id) + '; '
FROM sys.dm_exec_sessions
WHERE (status LIKE '%sleeping%' OR status LIKE '%suspended%') AND  database_id = DB_ID('CloudPOS_DB_GHORERBAZAR') -- Your database name
  AND session_id <> @CurrentSessionID; -- Exclude the current session

EXEC(@killCommand);
```

```
------------------------####### c# method #######--------------------------
public bool RefreshDatabaseConnections()
        {
            using (var command = _context.CreateCommand())
            {
                var conStr = ConfigurationManager.ConnectionStrings["DB_CONN"].ToString();
                System.Data.SqlClient.SqlConnectionStringBuilder builder = new System.Data.SqlClient.SqlConnectionStringBuilder(conStr);
                string databaseName = builder.InitialCatalog;

                StringBuilder sb = new StringBuilder();

                sb.AppendLine(@" DECLARE @CurrentSessionID INT;
                SELECT @CurrentSessionID = @@SPID;

                DECLARE @killCommand VARCHAR(MAX) = '';

                SELECT @killCommand = @killCommand + 'KILL ' + CONVERT(VARCHAR(5), session_id) + '; '
                FROM sys.dm_exec_sessions
                WHERE (status LIKE '%sleeping%' OR status LIKE '%suspended%') AND  database_id = DB_ID('" + databaseName + @"') -- Your database name
                AND session_id <> @CurrentSessionID; -- Exclude the current session

                EXEC(@killCommand); ");

                command.CommandText = sb.ToString();

                return this.ExecuteNonQueryWithoutTransaction(command);
            }
        }

 public bool ExecuteNonQueryWithoutTransaction(IDbCommand command)
        {
            try
            {
                int rowsAffected = command.ExecuteNonQuery();
                //return rowsAffected >= 0; // Return true if the command executed successfully
                return true;
            }
            catch (Exception ex)
            {
                throw ex;
            }
            finally
            {
            }
        }
```
