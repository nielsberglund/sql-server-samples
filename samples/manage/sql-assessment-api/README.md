# SQL Assessment API samples

Contains samples for customizing SQL Assessment API. Learn more about the API on the [SQL Assessment API docs page](https://docs.microsoft.com/en-us/sql/sql-assessment-api/sql-assessment-api-overview).

## config.json

This is the default set of checks shipped with SQL Assessment API. Feel free to open issues to have us fix or add checks. Also, we're happy to see your pull requests to this file.

## DisablingBuiltInChecks_sample.json

Contains two parts. First shows how you can disable a specified check by its ID. The second disables all the checks with the "TraceFlag" tag.

## MakingCustomChecks_sample.json

Demonstrates how to make a custom rule set containing two checks. The sample contains two sections: `checks` and `probes`. `Checks` is for check (or rule) definitions. Usually, checks or rules are best practices or a company's internal policies that should be applied to SQL Server. Here's one of the checks from this sample with comments on each property:

```
{
  "target": {                                           //Object to describe which SQL Server object this check is applied.
    "type": "Database",                                     //This check targets at Database object.
    "version": "[12.0,)",                                   //Applies to SQL Server 2014 and higher.
                                                            //Another example: "[12.0,13.0)" reads as "any SQL Server with version >= 12.0 and < 13.0.
    "platform": "Windows",                                  //Applies to SQL Server on Windows.
    "name": { "not": "/^(master|msdb)$/" }                  //Applies to any database but master and msdb.
  },
  "id": "CustomCheck1",                                 //Check ID.
  "tags": [ "InternalBestPracticeSet", "Performance" ], //Tags combine checks in different subsets.
  "displayName": "Query Store should be on",            //Short name for check.
  "description": "The SQL Server Query Store feature provides you with insight on query plan choice and performance. It simplifies performance troubleshooting by helping you quickly find performance differences caused by query plan changes. /n Query Store automatically captures a history of queries, plans, and runtime statistics, and retains these for your review. It separates data by time windows so you can see database usage patterns and understand when query plan changes happened on the server.",
                                                        //Some more detailed explanation of the best practice or policy.
  "message": "Turn Query Store option on to improve query performance troubleshooting.",
                                                        //Usually, it's for recommendation what the user should do if the check fires up
  "helpLink": "https://docs.microsoft.com/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store",
                                                        //Reference material
  "probes": [ "DatabaseConfiguration" ],                //List of probes that are used to get the required data for this check.
                                                        //Probes will be explained below.
  "condition": "@is_query_store_on"                     //Check will pass if condition is true. Otherwise, the check fires up.
}
```

`Probes` describe how and where get required data to perform a check. For this, you can use T-SQL queries as well as methods from assemblies. The probe below uses a T-SQL query.
```
"probes":{
  "DatabaseConfiguration": [                            //Probe name that is used to reference the probe from a check.
                                                        //Probe can have a few implementations that will be used for different targets.
                                                        //This probe has two implementations for different version of SQL Server.
    {
      "type": "SQL",                                    //Probe uses a T-SQL query to get the required data
      "target": {
        "type": "Database",                             //Targets at database
        "version": "(,12.0)",                           //This implementation is for SQL Server before 2014
        "platform": "Windows"                           //Targets at SQL on Windows
      },
      "implementation": {                               //Implementation object with a T-SQL query.
                                                        //sys.databases of SQL Server before 2014 doesn't have the field is_query_store_on so we replace it with 0.
        "query": "SELECT db.[is_auto_create_stats_on] AS is_auto_create_stats_on, db.[is_auto_update_stats_on] AS is_auto_update_stats_on, 0 AS is_query_store_on FROM sys.databases AS db WHERE db.[name]='@DatabaseName'"
      }
    },
    {                                                   //Second implementation
      "type": "SQL",
      "target": {
        "type": "Database",
        "version": "[12.0,)",                           //This implementation is for SQL Server 2014 and up.
        "platform": "Windows"
      },
      "implementation": {                               //Query of the second implementation.
        "query": "SELECT db.[is_auto_create_stats_on] AS is_auto_create_stats_on, db.[is_auto_update_stats_on] AS is_auto_update_stats_on, db.[is_query_store_on] AS is_query_store_on FROM sys.databases AS db WHERE db.[name]='@DatabaseName'"
      }
    }
  ]
}
```
