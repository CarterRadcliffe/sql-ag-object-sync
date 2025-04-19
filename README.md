# sql-ag-object-sync
Powershell script to automate the copying of objects between sql server always on availability group nodes. Uses DBATools.io module as the base.

Prerequisites:
* Central Management Server with availability group listener names stored in one folder
* DBATools.io powershell module installed
* User running process will need domain admin on the sql server machines and SA in the database instances

Notes: 
* Script will dynamically determine which node is primary of each AG and which node(s) are secondary
* Sync-DbaAvailabilityGroup is the base command for the sync
* I have this script setup to ignore the DBMail and specific logins upon sync
* This will drop and recreate all jobs on secondaries

Links:
* DBAtools module - https://dbatools.io
