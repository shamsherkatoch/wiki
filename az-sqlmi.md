Introduction

KB document for doing Native SQL Backup and Restore operations on Azure
SQL on Azure SQL Managed Instance (MI) and using Azure Storage account
for back/restore media.

Pre-requisites Below is related information of backup/restore for your
reference (Please prepare a storage account to store the database backup
files when doing backup/restore) - \* Backup MI DB to URL \* Create
storage account and container: Create a resource manager azure storage
account in the same region as your Azure SQL MI and BLOB container name
\"dbbackup\". See
https://docs.microsoft.com/en-us/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal
\* Verify that Storage account firewall allows SQL MI subnets under
Networking \> Firewall \> Selected networks \* \* \* Obtain shared
access signature (SAS token): \* Click on \"Shared Access Signature\"

\* Choose start and expiry date/time and click on \"Generate SAS and
connection string\" button

\* Copy the SAS token at safe location for later usage \* Current token
has expiry of 15-oct-2022 and stored on Secret Server. TSS \> Server and
Storage Operations \> AZURE \> Azure Hub Storage Account - SAS Token -
SQL MI DB\'s Backup/Restore

Note that when you copy the SAS token, it starts with question mark ?
You do NOT need this question mark (?). If you keep it, the credential
you create next step will not work KEEP THIS SAS TOKEN IN SAFE LOCATION,
YOU WON\'T BE ABLE TO ACCESS SAME TOKEN AGAIN. IF LOST, YOU WILL NEED TO
GENERATE NEW TOKEN AGAIN AND CREATE NEW CREDENITALS AS IN BELOW STEP.

Backup \* Login to Azure HUB VM AUACDC10RDS002 \"10.61.5.5\" where SSMS
is installed and connect to relevant SQL MI. \* Azure SQL MI owns all
backup chains. We are only creating the COPY of database. Create
credential Create credential for the access to storage account/container
in which you\'d like to store your database backups using SQL Query. (
For https://sqlvm.blob.core.windows.net/dbbackup storage account , SAS
credentials are already there so we can use that -\> TSS \> Server and
Storage Operations \> AZURE \>Azure HUB Storage Account (sqlvm) - SAS
Token - SQL MI DB\'s Backup/Restore ) /\* Create SAS credentials for SQL
MI to use

CREATE CREDENTIAL \[Error! Hyperlink reference not valid.\] WITH
IDENTITY = \'SHARED ACCESS SIGNATURE\' , SECRET = \'sv=SAS_token\'
\--make sure you remove? In the SAS token copied from previous step.

EXAMPLE USE \[master\] CREATE CREDENTIAL
\[https://sqlvm.blob.core.windows.net/dbbackup\] WITH IDENTITY=\'SHARED
ACCESS SIGNATURE\', SECRET =
\'sv=2020-08-04&ss=bf&srt=sco&sp=rwdlactfx&se=2022-10-15T11:42:36Z&st=2021-10-15T03:42:36Z&spr=https&sig=BgjpmlS9a%2BZNre7ZFs1RT5pY\-\-\-\-\-\-\-\--gi%2FNoORRk%3D\';

Backup using SQL query /\*turn off encryption on MI DB\*/ ALTER DATABASE
\<database_name\> SET ENCRYPTION OFF;

EXAMPLE: ALTER DATABASE \[FEG\] SET ENCRYPTION OFF; GO /\*confirm the
encryption is turned off on the DB\*/ SELECT db.name, db.is_encrypted,
dm.encryption_state, dm.percent_complete, dm.key_algorithm,
dm.key_length FROM sys.databases db LEFT OUTER JOIN
sys.dm_database_encryption_keys dm ON db.database_id = dm.database_id;
/\*If you see against database \"is_encrypted\" to 0 but still
\"encypted_state\" to 1 then you will need to run below command use
\[DATABASE_NAME\] DROP DATABASE ENCRYPTION KEY EXAMPLE: use \[FEG\] DROP
DATABASE ENCRYPTION KEY /\*backup database BACKUP DATABASE
\<database_name\> TO URL =
N\'https://\<storage_account\>.blob.core.windows.net/\<container\>/\<backup_name\>.bak\'
with copy_only, compression; EXAMPLE: BACKUP DATABASE \[FEG\] TO URL =
N\'https://sqlvm.blob.core.windows.net/dbbackup/FEG_DEV-ver2-14-10-21.bak\'
with copy_only, compression;

/\*turn ON encryption on MI DB after verifying that backup is there on
Storage account container\*/ ALTER DATABASE \<database_name\> SET
ENCRYPTION ON; GO EXAMPLE: ALTER DATABASE \[FEG\] SET ENCRYPTION ON; GO

Backup using SSMS Wizard

Once Credential setup completed using \"Create Credential\" step then
you can use SSMS backup wizard to do the backup. \* Once connected to
the SQL MI, Right click database \> Tasks \> Backup \*

\* Select General \> Database \> URL \> Add \> Select Azure storage
container credentials from drop down that you have created in first step
\> Change backup file name if required \> OK \*

Restore

Restore using SSMS Wizard \* Select Database \> Tasks \> Restore \>
Database \* \*

\* General \> Device \> \... \> URL \> ADD

\* Select Drop down for existing Storage account credentials created in
Step 1 for Backup.

\* Paste SAS token saved in Last step of Prerequisites SAS token
creation step.

\*

\* Select backup file to restore from storage account on next step.

Template - System Overview 1/02/2024 12:43 AM Page 2 of 4 For Official
Use Only

