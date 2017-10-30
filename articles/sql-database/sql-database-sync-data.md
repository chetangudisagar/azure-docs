---
title: Sync data (Preview) | Microsoft Docs
description: This overview introduces Azure SQL Data Sync (Preview).
services: sql-database
documentationcenter: ''
author: douglaslms
manager: craigg
editor: ''

ms.assetid: 
ms.service: sql-database
ms.custom: load & move data
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/27/2017
ms.author: douglasl

---
# Sync data across multiple cloud and on-premises databases with SQL Data Sync

SQL Data Sync is a service built on Azure SQL Database that lets you synchronize the data you select bi-directionally across multiple SQL databases and SQL Server instances.

Data Sync is based around the concept of a Sync Group. A Sync Group is a group of databases that you want to synchronize.

A Sync Group has the following properties:

-   The **Sync Schema** describes which data is being synchronized.

-   The **Sync Direction** can be bi-directional or can flow in only one direction. That is, the Sync Direction can be *Hub to Member* or *Member to Hub*, or both.

-   The **Sync Interval** is how often synchronization occurs.

-   The **Conflict Resolution Policy** is a group level policy, which can be *Hub wins* or *Member wins*.

Data Sync uses a hub and spoke topology to synchronize data. You define one of the databases in the group as the Hub Database. The rest of the databases are member databases. Sync occurs only between the Hub and individual members.
-   The **Hub Database** must be an Azure SQL Database.
-   The **member databases** can be either SQL Databases, on-premises SQL Server databases, or SQL Server instances on Azure virtual machines.
-   The **Sync Database** contains the metadata and log for Data Sync. The Sync Database has to be an Azure SQL Database located in the same region as the Hub Database. The Sync Database is customer created and customer owned.

> [!NOTE]
> If you're using an on premises database, you have to [configure a local agent.](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-get-started-sql-data-sync)

![Sync data between databases](media/sql-database-sync-data/sync-data-overview.png)

## When to use Data Sync

Data Sync is useful in cases where data needs to be kept up to date across several Azure SQL Databases or SQL Server databases. Here are the main use cases for Data Sync:

-   **Hybrid Data Synchronization:** With Data Sync, you can keep data synchronized between your on-premises databases and Azure SQL Databases to enable hybrid applications. This capability may appeal to customers who are considering moving to the cloud and would like to put some of their application in Azure.

-   **Distributed Applications:** In many cases, it's beneficial to separate different workloads across different databases. For example, if you have a large production database, but you also need to run a reporting or analytics workload on this data, it's helpful to have a second database for this additional workload. This approach minimizes the performance impact on your production workload. You can use Data Sync to keep these two databases synchronized.

-   **Globally Distributed Applications:** Many businesses span several regions and even several countries. To minimize network latency, it's best to have your data in a region close to you. With Data Sync, you can easily keep databases in regions around the world synchronized.

We don't recommend Data Sync for the following scenarios:

-   Disaster Recovery

-   Read Scale

-   ETL (OLTP to OLAP)

-   Migration from on-premises SQL Server to Azure SQL Database

## How does Data Sync work? 

-   **Tracking data changes:** Data Sync tracks changes using insert, update, and delete triggers. The changes are recorded in a side table in the user database.

-   **Synchronizing data:** Data Sync is designed in a Hub and Spoke model. The Hub syncs with each member individually. Changes from the Hub are downloaded to the member and then changes from the member are uploaded to the Hub.

-   **Resolving conflicts:** Data Sync provides two options for conflict resolution, *Hub wins* or *Member wins*.
    -   If you select *Hub wins*, the changes in the hub always overwrite changes in the member.
    -   If you select *Member wins*, the changes in the member overwrite changes in the hub. If there's more than one member, the final value depends on which member syncs first.

## Limitations and considerations

### Performance impact
Data Sync uses insert, update, and delete triggers to track changes. It creates side tables in the user database for change tracking. These change tracking activities have an impact on your database workload. Assess your service tier and upgrade if needed.

### Eventual consistency
Since Data Sync is trigger-based, transactional consistency is not guaranteed. Microsoft guarantees that all changes are made eventually and that Data Sync does not cause data loss.

### Unsupported data types

-   FileStream

-   SQL/CLR UDT

-   XMLSchemaCollection (XML supported)

-   Cursor, Timestamp, Hierarchyid

### Requirements

-   Each table must have a primary key. Don't change the value of the primary key in any row. If you have to do this, delete the row and recreate it with the new primary key value. 

-   A table cannot have an identity column that is not the primary key.

-   The names of objects (databases, tables, and columns) cannot contain the printable characters period (.), left square bracket ([), or right square bracket (]).

-   Snapshot isolation must be enabled. For more info, see [Snapshot Isolation in SQL Server](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql/snapshot-isolation-in-sql-server).

### Limitations on service and database dimensions

|                                                                 |                        |                             |
|-----------------------------------------------------------------|------------------------|-----------------------------|
| **Dimensions**                                                      | **Limit**              | **Workaround**              |
| Maximum number of sync groups any database can belong to.       | 5                      |                             |
| Maximum number of endpoints in a single sync group              | 30                     | Create multiple sync groups |
| Maximum number of on-premises endpoints in a single sync group. | 5                      | Create multiple sync groups |
| Database, table, schema, and column names                       | 50 characters per name |                             |
| Tables in a sync group                                          | 500                    | Create multiple sync groups |
| Columns in a table in a sync group                              | 1000                   |                             |
| Data row size on a table                                        | 24 Mb                  |                             |
| Minimum sync interval                                           | 5 Minutes              |                             |

## Common questions

### How frequently can Data Sync synchronize my data? 
The minimum frequency is every five minutes.

### Can I use Data Sync to sync between SQL Server on-premises databases only? 
Not directly. You can sync between SQL Server on-premises databases indirectly, however, by creating a Hub database in Azure, and then adding the on-premises databases to the sync group.
   
### Can I use Data Sync to seed data from my production database to an empty database, and then keep them synchronized? 
Yes. Create the schema manually in the new database by scripting it from the original. After you create the schema, add the tables to a sync group to copy the data and keep it synced.

### Why do I see tables that I did not create?  
Data Sync creates side tables in your database for change tracking. Don't delete them or Data Sync stops working.
   
### I got an error message that said "cannot insert the value NULL into the column \<column\>. Column does not allow nulls." What does this mean, and how can I fix the error? 
This error message indicates one of the two following issues:
1.  There may be a table without a primary key. To fix this issue, add a primary key to all the tables you're syncing.
2.  There may be a WHERE clause in your CREATE INDEX statement. Sync does not handle this condition. To fix this issue, remove the WHERE clause or manually make the changes to all databases. 
 
### How does Data Sync handle circular references? That is, when the same data is synced in multiple sync groups, and keeps changing as a result?
Data Sync doesn’t handle circular references. Be sure to avoid them. 

### How can I export and import a database with Data Sync?
After you export a database as a .bacpac file and import it to create a new database, you have to do the following two things to use Data Sync in the new database:
1.  Clean up the Data Sync objects and side tables on the **new database** by using [this script](https://github.com/Microsoft/sql-server-samples/blob/master/samples/features/sql-data-sync/clean_up_data_sync_objects.sql). This script deletes all of the required Data Sync objects from the database.
2.  Recreate the sync group with the new database. If you no longer need the old sync group, delete it.

## Next steps

For more info about SQL Data Sync, see:

-   [Getting Started with SQL Data Sync](sql-database-get-started-sql-data-sync.md)

-   Complete PowerShell examples that show how to configure SQL Data Sync:
    -   [Use PowerShell to sync between multiple Azure SQL databases](scripts/sql-database-sync-data-between-sql-databases.md)
    -   [Use PowerShell to sync between an Azure SQL Database and a SQL Server on-premises database](scripts/sql-database-sync-data-between-azure-onprem.md)

-   [Download the complete SQL Data Sync technical documentation](https://github.com/Microsoft/sql-server-samples/raw/master/samples/features/sql-data-sync/Data_Sync_Preview_full_documentation.pdf?raw=true)

-   [Download the SQL Data Sync REST API documentation](https://github.com/Microsoft/sql-server-samples/raw/master/samples/features/sql-data-sync/Data_Sync_Preview_REST_API.pdf?raw=true)

For more info about SQL Database, see:

-   [SQL Database Overview](sql-database-technical-overview.md)

-   [Database Lifecycle Management](https://msdn.microsoft.com/library/jj907294.aspx)
