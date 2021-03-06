![](images/300/Lab300_image100.PNG)

Update December 28, 2018

## UniDirectional and DDL Replication
## Introduction

This lab walk you through unidirectional and DDL replication between to database schemas using Goldengate 18.1 micro services web interface in a Ravello environment.

![](images/300/Lab300_image104.PNG)

This lab supports the following use cases:
-	Migration of on-premise pluggable databases to a cloud based environment.
-	Rapid creation of test or development pluggable database copies in the Cloud.
-	Rapid replication of on-premise data for reporting and analytics.
-	Rapid re-fresh of selected on-premise schemas for test and/or development activities and/or reporting and analytics.

- To log issues and view the Lab Guide source, go to the [github oracle](https://github.com/oracle/learning-library/tree/master/workshops/dbcs) repository.

## Objectives

-   Migrate a pluggable database from on-premise to the Cloud.
-   Migrate a schema using Oracle Data Pump.
-   Migrate data using a Transportable Tablespace.
-   Copy data using Database Links.

## Required Artifacts

Lab 7a: Configure Uni-Directional Replication (Integrated Extract)

Objective:

This lab is in two parts.  The first part will setup the Integrated Extract for Oracle GoldenGate 18c Service Architecture for a uni-directional configuration using the SOE schema in OGGOOW181 and OGGOOW182 PDBs. 

Time: 25 minutes

Steps:

1.	Open Firefox and login to the Service Manager using the Administrator account you setup during deployment (Figure 7a-1). Port number will vary depending on what you used during setup.

For Ravello Environment <br />
http://localhost:16000 <br />


Figure 7a-1:

![](images/300/Lab300_image110.PNG) 
 

2.	After logging in, find and open the Administration Server for your first deployment.  In this example, the first deployment is Atlanta (Figure 7a-2).  When the page is completely open, you should be at a page where you can see Extracts/Replicats clearly.
Note: You will be required to login again.  Use the same Administrator account that was used with the Service Manager.

Figure 7a-2:

![](images/300/Lab300_image120.PNG) 
 

3.	Before you can create an Extract, you need to setup a credential alias for the GoldenGate user (C##GGATE).  This is done from the Configuration menu option in the grey bar on the left of the screen (Figure 7a-3).

Figure 7a-3:

![](images/300/Lab300_image130.PNG) 
 

4.	On the Configuration page, select the plus ( + ) sign to begin adding a credential.  At this point, you will be able to add a Credential Alias (Figure 7a-4).  You will need to add the alias for a user that will connect to CDB (ORCL) and PDB (OGGOOW181).  The CDB alias will be used to connect to the database to read the required files for extraction operations, and the PDB1 user will be used to add TRANDATA to the schemas used in replication.

Figure 7a-4:

![](images/300/Lab300_image150.PNG) 
 

You will notice that a Domain name and Credential Alias were added along with the User ID and Password.  After adding the user to the credential store, you will reference it via its domain name and credential alias.

You will need to create two (2) credential aliases for your Atlanta deployment. The first credential will be for the CDB(ORCL) database and the second will be for the PDB(OGGOOW181) database. The table below shows what needs to be added:


Credential Domain	Credential Alias	UserID	Password
SGGATE	SGGATE	C##GGATE@OGGOOW181	ggate
CDBGGATE	CDBGGATE	C##GGATE@ORCL	ggate
 

5.	Verify that the credentials you just created work.  There is a little database icon under Action in the table.  Click on this for each Credential Alias and you should be able to login to the database (Figure 7a-5).

Figure 7a-5:

![](images/300/Lab300_image160.PNG) 
 

6.	Add SCHEMATRANDATA to the SOE schema using the SGGATE Credential Alias.  
After logging into the database as described in step 5 for OGGOOW181, find the Trandata section.  Click on the plus ( + ) sign and make sure that the radio button for Schema is selected (Figure 7a-6).  At this point, you provide the Schema Name, enable All Columns and Scheduling Columns, and click Submit.

Figure 7a-6:

![](images/300/Lab300_image170.PNG) 
 

You will notice that after you click Submit, there is no return message that states the operation was successful.  You can verify that SCHEMATRANDATA has been added by looking searching by Schema (Figure 7a-7).  To do this, click on the magnifying glass and provide the Schema name.

Figure 7a-7:

![](images/300/Lab300_image180.PNG) 
 

7.	Add the Protocol user.
Since we are on the Credential screen, let’s go ahead and add a Protocol user.  A Protocol user is the user that the Distribution Server will use to communicate with the Receiver Server over an unsecure connection.
As you did in Step 4, click the plus sign ( + ) next to the word Credentials.  Then provide the connection information needed (Figure 7a-8), notice that you will be using the Service Manager login in this credential.

Figure 7a-8:

![](images/300/Lab300_image190.PNG) 
 

For now, just leave this login alone.  It will be used in a later step. 

8.	Add the Integrated Extract.
Navigate back to the Overview page of the Administration Server (Figure 7a-9).  Then click on the plus sign ( + ) in the box for Extracts.

Figure 7a-9:

![](images/300/Lab300_image300.PNG) 


After clicking the plus sign ( + ), you are taken to the Add Extract page (Figure 7a-10).  Here you can choose from three different types of Extracts.  You will be installing an Integrated Extract.  Click Next.

Figure 7a-10:

![](images/300/Lab300_image210.PNG) 


The next page of the Add Extract process, is to provide the basic information for the Extract. Items required have a star ( * ) next to them.  Provide the required information and then click Next (Figure 7a-11).  Keep in mind that the credentials needed to register the Extract need to be against the CDB (ORCL). Use the CDB domain and alias that you setup previously.

When using the CDB credential, at the bottom of the page, you will be presented with a box where you can select the PDB that will be used. This will only appear when you have a valid credential for the CDB.  Once you see this box, make sure you select OGGOOW181. 

Figure 7a-11:

![](images/300/Lab300_image220.PNG) 

![](images/300/Lab300_image225.PNG) 
 

On the last page of the Add Extract process, you are presented with a parameter file (Figure 7a-12).  The parameter file is partially filled out, but missing the TABLE parameters. Insert the following list of TABLE parameter values into the parameter file.
DDL INCLUDE ALL; <br />
SOURCECATALOG OGGOOW181 <br />
TABLE SOE.*; <br />

Notes: ~/Desktop/Software/extract.prm has these contents for copying.

You can also include specific table names for capturing the data changes, but extract will skip the create table DDLs. Sample TABLE parameter vales are given below.

DDL INCLUDE ALL; <br />
SOURCECATALOG OGGOOW181 <br />
TABLE SOE.ADDRESSES; <br />
TABLE SOE.CUSTOMERS; <br />
TABLE SOE.ORDERS; <br />
TABLE SOE.ORDER_ITEMS; <br />
TABLE SOE.CARD_DETAILS; <br />
TABLE SOE.LOGON; <br />
TABLE SOE.PRODUCT_INFORMATION; <br />
TABLE SOE.INVENTORIES; <br />
TABLE SOE.PRODUCT_DESCRIPTIONS; <br />
TABLE SOE.WAREHOUSES; <br />
TABLE SOE.ORDERENTRY_METADATA; <br />

Once the TABLE statements are added, click Create and Run at the bottom of the page.

Figure 7a-12:
 
![](images/300/Lab300_image230.PNG) 

The Administration Server page will refresh when the process is done registering the Extract with the database, and will show that the Extract is up and running (Figure 7a-13).

Figure 7a-13:
 
![](images/300/Lab300_image240.PNG) 

Lab 7b: Configure Uni-Directional Replication (Distribution Server)

Objective:
This lab will walk you through how to setup a Path within the Distribution Server.

Time: 10 minutes

Steps:
1.	Start from the Service Manager page (Figure 7b-1).

Figure 7b-1:

![](images/300/Lab300_image250.PNG) 


2.	Open the Distribution Server page for your first deployment (Figure 7b-2).

Figure 7b-2:

![](images/300/Lab300_image260.PNG) 

3.	Click the plus sign ( + ) to add a new Distribution Path (Figure 7b-3).

Figure 7b-3:

![](images/300/Lab300_image270.PNG) 

4.	On the Add Path page, fill in the required information (Figure 7b-4).  Make note that the default protocol for distribution service is secure websockets (wss).  You will need to change this to websockets (ws).

Figure 7b-4:

![](images/300/Lab300_image280.PNG) 

![](images/300/Lab300_image285.PNG) 

Notice the drop down with the values WS, WSS, UDT and OGG.  These are the protocols you can select to use for transport.  Since you are setting up an unsecure uni-directional replication, make sure you select WS, then provide the following target information:
Hostname: localhost
Port: <2nd deployment’s receiver server port>
Trail File: <any two letter value>
Domain: <credential you created in the Admin Server for WS>
Alias: <credential you created in the Admin Server for WS>
After filling out the form, click Create and Run at the bottom of the page.

5.	If everything works as expected, your Distribution Path should be up and running.  You should be able to see clearly the source and target on this page (Figure 7b-5).

Figure 7b-5:
 
![](images/300/Lab300_image290.PNG) 


Lab 7c: Configure Uni-Directional Replication (Receiver Server)

Objective:
In this lab, you will configure the Receiver Server for the target database, which will receive the trail from the Distribution Path that you created on the source deployment.

Time: 5 minutes

Steps:
1.	Start from the Service Manager page for your second deployment (Figure 7c-1).

Figure 7c-1:
 
![](images/300/Lab300_image305.PNG) 

2.	Click on the Receiver Server link to open the Receiver Server page (Figure 7c-2).  Verify that everything is configured.

Figure 7c-2:

![](images/300/Lab300_image310.PNG) 


Lab 7d: Configure Uni-Directional Replication (Integrated Replicat)

Object:
In this lab you will configure the Integrated Replicat for the second deployment.

Time: 25 minutes

Steps:
1.	Starting from the Service Manager page (Figure 7d-1).

Figure 7d-1:
 
![](images/300/Lab300_image320.PNG) 
 
2.	Open the Administration Server for the second deployment by clicking on the link (Figure 7d-2).

Figure 7d-2:

![](images/300/Lab300_image330.PNG) 

3.	Open the Configuration option to add your credentials needed to connect to PDB2 (OGGOOW182) (Figure 7d-3).  After creating the credential, login and verify that it works.
You will need to create 1 credential for the user to connect to PDB2.  We will use the same common user as before, C##GGATE@OGGOOW182, with password ggate.  Click Submit when finished.

Figure 7d-3:
 
![](images/300/Lab300_image340.PNG) 


4.	Navigate back to the Overview page on the Administration Server.  Here you will begin to create your Integrated Replicat (Figure 7d-4).  Click the plus sign ( + ) to open the Add Replicat process.

Figure 7d-4:
 
![](images/300/Lab300_image350.PNG) 


5.	With the Add Replicat page open, you want to create an Integrated Replicat.  Make sure the radio button is selected and click Next (Figure 7d-5).

Figure 7d-5:
 
![](images/300/Lab300_image360.PNG) 


6.	Fill in the Replicat options form with the required information (Figure 7d-6).  Your trail name should match the trail name you saw in the Receiver Server.  Once you are done filling everything out, click the Next button at the bottom of the screen.

Figure 7d-6:
 
![](images/300/Lab300_image370.PNG) 

7.	You are next taken to the Parameter File page.  On this page, you will notice that a sample parameter file is provided (Figure 7d-7).  You will have to remove the MAP statement and replace it with the information below:

MAP OGGOOW181.SOE.*, TARGET SOE.*; <br />

Figure 7d-7:
 
![](images/300/Lab300_image380.PNG) 

You can also specify individual table name as given below. <br />

MAP PDB1.SOE.CUSTOMERS, TARGET SOE.CUSTOMERS; <br />
MAP PDB1.SOE.ADDRESSES, TARGET SOE.ADDRESSES;  <br />
MAP PDB1.SOE.ORDERS, TARGET SOE.ORDERS;<br />
MAP PDB1.SOE.ORDER_ITEMS, TARGET SOE.ORDER_ITEMS; <br />
MAP PDB1.SOE.CARD_DETAILS, TARGET SOE.CARD_DETAILS; <br />
MAP PDB1.SOE.LOGON, TARGET SOE.LOGON; <br />
MAP PDB1.SOE.PRODUCT_INFORMATION, TARGET SOE.PRODUCT_INFORMATION; <br />
MAP PDB1.SOE.INVENTORIES, TARGET SOE.INVENTORIES; <br />
MAP PDB1.SOE.PRODUCT_DESCRIPTIONS, TARGET SOE.PRODUCT_DESCRIPTIONS; <br />
MAP PDB1.SOE.WAREHOUSES, TARGET SOE.WAREHOUSES; <br />
MAP PDB1.SOE.ORDERENTRY_METADATA, TARGET SOE.ORDERENTRY_METADATA; <br />
Notes: ~/Desktop/Software/replicat.prm has these contents for copying.
Once the parameter file has been updated, click the Create and Run button at the bottom.


At this point, you should have a fully functional uni-directional replication environment. You can start Swingbench and begin testing.  See Appendix A for further instructions.


Lab 7e: DML and DDL Replication Samples

Objective: In this lab we will perform few DML and DDL operations on source pdb and check if those operations are properly replicated to target database.

prerequisite: Source and target database should be in sync. Extract, pump and replicat should be up and running.

Time: 15 mins

Steps:

1. Logon to OGGOOW181 and OGGOOW182 pdbs using SOE user.

![](images/300/Lab300_image400.PNG) 


2. Verify whether tables are in sync between source and target databases.

![](images/300/Lab300_image405.PNG) 

![](images/300/Lab300_image410.PNG) 

3. Create a employee table in OGGOOW181.

![](images/300/Lab300_image415.PNG) 

4. Go to target admin server page, click on actions on replicate and select details option.

![](images/300/Lab300_image420.PNG) 

5. Click on statistics tab and check the DDL Mapped count.

![](images/300/Lab300_image425.PNG) 

6. Perform few insert operations on source tables in OGGOOW181 pdb database and check if the inserts are replicated to target tables.

![](images/300/Lab300_image430.PNG) 

![](images/300/Lab300_image435.PNG) 

7. Perform few updates and deletes operations on source table and check if the operations are replicated to target database.

![](images/300/Lab300_image440.PNG) 

8. Execute the below alter commands and verify the statistics on extract and replicat.

![](images/300/Lab300_image445.PNG) 

9. Execute truncate operation on employee table, verify the statistics and count in the target pdb.

![](images/300/Lab300_image455.PNG) 

![](images/300/Lab300_image460.PNG) 

![](images/300/Lab300_image465.PNG)

10. Similarly, execute drop command on the employee table and check the results in the target database.

![](images/300/Lab300_image470.PNG)

![](images/300/Lab300_image475.PNG)

![](images/300/Lab300_image480.PNG)

The above error is because employee table is not present in the target database. Drop command is executed successfully in target database.

This completes the DML and DDL replication sample operations lab.












