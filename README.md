# PSQL Guides
Handy documentation and guides for working with PostgreSQL (psql)

[Create NPI Table SQL Script] (./npi_table.sql)

Skip the rest if you just want the .sql script file.

## Creating the NPI Database
One of the great things about working in health care is the massive amount of publicly available data. 

For example, the [Centers for Medicare & Medicaid Services] (https://www.cms.gov/), or CMS, releases huge amounts of data on a regular basis on their doctors, payments, health plans, etc.

This guide is will go through the process of downloading the NPI file from CMS and loading it into a psql database for analysis.
The guide is intended for people using Windows.

### Setting up the PSQL Database
1. Download the NPI file from the [CMS Website] (http://download.cms.gov/nppes/NPI_Files.html). It's about 550MB as of December 2015.
2. Use winzip, winrar, or 7-zip to decompress the file to the directory of your choice. The default windows zip explorer will not work!
3. There should be 4 files:
   - The main CSV file (5 GB or so decompressed)
   - A header CSV file (much smaller)
   - Two PDF files explaining the data
4. Download the graphic installer for [PostgreSQL for Windows] (http://www.postgresql.org/download/windows/) and install it
5. Follow the prompts and use the default options. Pick a good password.
6. Open pgAdmin. On the left (Object Browser) you should see the server you just created it and below that,
   - Databases 
   - Tablespaces
   - GroupRoles
   - LoginRoles
7. Right click on the Databases to create a new database. I named it 'CMS', leave everything else as default. 
8. Right click on LoginRoles to create a new role with only the 'Can Login' privilege. This will be the role which users login to perform queries. Create as many roles as needed (one per person 'johnnyk' or one per role 'salesops').
9. The next part is a little technical, you'll have to edit the pg_hba.conf file to allow remote logins.
   - Best to speak with the IT team on what should be the correct setting. Do you want to only allow local connections, local + VPN, all? All connections is a bad idea.
   - Once you have the ip address, go to the PSQL install directory (Mine is C:\Program Files\PostgreSQL\9.4\data) and open the pg_hba.conf file
   - Add the following line below the 'host all all 127.0.0.1 ...' line. Replace 10.10.29.0/24 with what your IT team provides you
      - host all all 10.10.29.0/24 md5
   - Save the file
   - Restart the PSQL server. The easiest way to do this is to restart computer. You can also restart by going into Control Panel -> Administrative Tools -> Services. Look for the PostgreSQL entry, stop the service, start the service again.
   - Change the firewall setting to allow incoming UDP connections at the PSQL port (default is 5432)
10. Test the connection is working by launching PSQL Shell. When prompted for the server address, type in the actual IP address of the server instead of 'localhost' to connect. If not, you'll have to work with IT to see what the solution is.

At this point, you have created a PSQL server which allows remote connections and an empty database. You can now login from the PSQL shell and execute the [create NPI table SQL Script] (./npi_table.sql) to create the NPI table, or go to the next section to create your own script (in case the NPI file has changed since December 2015).

```
Make sure to grant access to the roles you've created! 
```
The command is 'GRANT SELECT ON npi TO johnnyk;' where johnnyk is the user role. 
You can choose what types of actions each user is allowed to do, but SELECT should be sufficient for most users.

### Creating the NPI Table with Correct Columns and Field Data Types

1. Open the header CSV file using a programming oriented text editor. [Notepad++] (https://notepad-plus-plus.org/) is one such tool.
2. Follow the directions [here] (https://gist.github.com/ramons03/6152802), use extended search and replace ',' with '\r\n'
   - This should create a new line for each single column
3. Search and replace space with underscore '_'. I also replaced longer titles with shorter words and deleted everything between '(' and ')'. I wanted to create shorter column names.
   - Provider -> Prvdr
   - Business -> Biz
   - Healthcare -> HC
4. Type 'CREATE TABLE npi (' as the first line to initial the create table command. In this case, I've named the table 'npi'
5. Provide each column with the field type. This part is tedious and manual. Refer to the PDFs that came with the NPI file for what each column contains.
   - Some things to note: Other than NPI, I've choosen to define all other columns as varchar. This is because the CSV file handles both empty and null values as "". If you set Replacement NPI as bigint and the value is "", PSQL would try to import it as an empty string instead of a NULL. This can be fixed if you open the database file and search and replace all "" with another value.
6. Make sure the last column doesn't have a ',' after it, but instead a ')'
7. Save file with a .sql extension. I called mine npi_table.sql
8. Copy this file to the scripts directory within the PostgreSQL install directory.
9. Run the PostgreSQL shell and log into the server. 
10. Run \q npi_table.sql
   - It should respond with TABLE CREATED
   - If not, check the directory the .sql file is in. You may have to do something like \q C:\User\Desktop\npi_table.sql if the script is saved to the desktop.
11. Open pgAdmin and check the table is created under Database -> Schema -> Tables

> Make sure to choose the correct database! In this case, 'CMS'. See step 7 of Creating the NPI Database

### Importing Data Into the NPI Table

1. Right click on the table and select import.
2. Select the large CSV file
3. Choose CSV as the file format
4. Under Misc. Options check the header box
5. Select ',' as the delimiter
6. Press Import. This will take a long time.
7. Assuming everything worked, there should be no error messages and you'll be able to press "Done".
8. Open pgAdmin, right click on the npi table, select View Data -> View First 100 Rows to make sure the data is present
9. If everything is there, that's it!

```
Make sure to grant access to the roles you've created! 
```
The command is 'GRANT SELECT ON npi TO johnnyk;' where johnnyk is the user role. 
You can choose what types of actions each user is allowed to do, but SELECT should be sufficient for most users.
