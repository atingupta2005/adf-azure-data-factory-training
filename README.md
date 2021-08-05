Clone Git Repo:
https://github.com/atingupta2005/adf-apr-21.git

Specify details of Git while creating your Data Factory

1-azure-data-factory-mapping-data-flows-basics (basics branch)
-----------------------------------------------
1. Create blank SQL Database - (dbatinapr21)
	Server Name: sqlsvrapr21.database.windows.net
Note: make sure to enable firewall settings in Networking section to Public Endpoint and Allow Client IP and Azure Services
	
2. Create cars table in the database
CREATE TABLE Cars (
	Make nvarchar(100),
	Model nvarchar(200),
	Type nvarchar(100),
	Origin nvarchar(100),
	DriveTrain nvarchar(100),
	Length decimal(18,0)
)

3. Create a new Data Factory (adfatinapr21)
4. Create storage account (sadfapr21), create container (input) and upload cars.csv to it
5. Create linked service to connect to storage account - (lsStorage)
6. Create linked service to connect to sql server - (lsSQL)
7. Add dataset to connect to sql table - (outTable)
8. Create dblob dataset to connect to cars.csv	- (inputCSV)
9. Create pipeline
10. Add copy data activity to the pipeline
11. Configure copy data activity
12. Run pipeline
13. Verify data is there in the cars table
select * from [dbo].[Cars]

2-azure-data-factory-parametrization (parametrization branch)
-----------------------------------------------
1. Create planes tables in SQL database
CREATE TABLE Planes (
	ICAO nvarchar(200),
	IATA nvarchar(200),
	MAKER nvarchar(200),
	DESCRIPTION nvarchar(200)
)
3. Add parameter to dataset - inputCSV
	- fileName
4. Use parameter in the connection tab
5. Add parameter to dataset - outputTable
	- tableName
5. Upload planes.csv to storage account\input container
6. Use parameter in the connection tab
11. Now add parameters to the pipeline itself
12. Use those parameters in copy data activity
13. Now while running pipeline, it will ask the inputs
14. Add trigger to pipeline
15. Publish pipeline
16. Run pipeline using trigger
17. Monitor pipeline execution in Monitor tab


3-azure-data-factory-mapping-data-flows (data-flows branch)
-----------------------------------------------
1. Create another container in storage account - output
2. Upload movies.csv to input
3. Create/update linked services
6. Create dataset to input\movies.csv (dsInMovies)
7. Create dataset to output\ (dsOutMovies)
	- Select first row as header
	- Disable import schema
8. Create a Data Flow resource
9. Add Source -> dsInMovies dataset
10. Enable Data flow debug
11. Wait for 5-10 min
12. Select step and open tab Data Preview
13. Add new step - Derived Column. Name it - YearTitleExtraction
14. Enter column Name (Year) and Add expression to it
	toInteger(trim(right(title, 6), '()'))
15. Preview data
16. Add new column - Title and add expression
	toString(left(title, length(title)-6))
17. Preview the data on each step to verify changes
18. Add new Step - Sink
19. Specify dataset - dsOutMovies
20. Add a new Pipeline (pipeline2) and add Dataflow to it
21. Run Pipeline and Refer to output container and open file - part*
22. To create a single file open dataflow
23. Open sink step and specify "Output to Single File" in Settings tab


4-azure-data-factory-self-hosted-integration-runtime - (ir branch)
-----------------------------------------------
1. Create a data factory
2. Open Manage Tab\connections\intergration runtimes and create new
3. Create Self Hosted Integration runtime (irSelfHosted)
4. Download and install on VM
	- https://www.microsoft.com/en-us/download/details.aspx?id=39717
	Version: 5.4
	
5. Register by specifying the key
6. Verify the connection status in Portal
7. Create a folder on the VM:
	C:\ir
7. Create a linked service to read file (lsirFile)
	Integration Runtime: Self Hosted
	Host: C:\ir
	Username: dsvmApr21\atingupta2005
	Password: <Password of VM>
8. Put a cars.csv in C:\ir
9. Create a dataset of type Fileservice and connect to that file (dsirInput)
10. Create a dataset of type Fileservice and connect to the folder and specify a new file name - carscopy.csv (dsirOut)
11. Create a pipeline (pipeline2) and Copy Data activity and configure it
12. Notice that we can copy content on the VM



Get Metadata Activity (branch getMetaData)
-----------------------
1. Create/Update the Linked Service
1. Create a new Pipeline (pipeline3)
2. Add activity - Get Metadata
3. Connect to the File Linked Service created using Self Hosted Runtime
4. Create a dataset to represent files in folder (dsFolderIn)
5. This dataset should refer to the folder - c:\ir
6. Connect activity with dataset and add new Field in the activity - Child items
7. Publish and Run pipeline
8.  Inspect the output JSON
9. Add a foreach activity
10. In Setting\Items Add dynamic content and put below expression:
	@activity('Get Metadata1').output.childitems
11. Create a variable in pipeline level - (fileName). It will be used in ForEach
12. Open For each activity
13. Add Set Variable activity
14. Use @item().name to refer to the current item in For each loop
15. Debug pipeline and verify the output

Filter Activity (Filter_if_append branch):
----------------
1. Create/Update Linked Service
1. Create a new dataset to refer to the output folder in storage account (dsOutFolderSA)
1. In previous pipeline add a filter activity after Get Metadata and remove for each
2. Add dynamic content and put below expression:
	@activity('Get Metadata1').output.childitems
2. Add a file on the VM where Self Hosted IR is setup. File name should start with - cars
3. Add a filter condition
	@startswith(item().name, 'c')
4. Debug pipeline and see the JSON output of the activities
5. Add For each loop and configure
	@activity('Filter1').output.Value
6. Now go into For Each activity and create Copy Data activity
	Source: dsFolderIn, Wildcard Path, @item().name
	Sink: dsOutFolderSA
	

Refer: pipeline3
