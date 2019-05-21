
This example runs on MapR 6.0.1 ,  Spark 2.2.1 and greater

Install and fire up the Sandbox using the instructions here: http://maprdocs.mapr.com/home/SandboxHadoop/c_sandbox_overview.html. 

____________________________________________________________________

Step 1: Log into Sandbox, create data directory, MapR-ES Topic and MapR-DB table:

Use an SSH client such as Putty (Windows) or Terminal (Mac) to login. See below for an example:
use userid: mapr and password: mapr.

For VMWare use:  $ ssh mapr@ipaddress 

For Virtualbox use:  $ ssh mapr@127.0.0.1 -p 2222 

after logging into the sandbox At the Sandbox unix command line:
 
Create a directory for the data for this project

mkdir /mapr/demo.mapr.com/data

____________________________________________________________________

Step 2: Copy the data file to the MapR sandbox or your MapR cluster

 
Copy the data file from the project data folder to the sandbox using scp to this directory /user/mapr/data/uber.csv on the sandbox:

For VMWare use:  $ scp  *.json  mapr@<ipaddress>:/mapr/demo.mapr.com/data/.
For Virtualbox use:  $ scp -P 2222 data/*.json  mapr@127.0.0.1:/mapr/demo.mapr.com/data/.

this will put the data file into the cluster directory: 
/mapr/<cluster-name>/data/

____________________________________________________________________

Step 3: To run the code in the Spark Shell:
 
/opt/mapr/spark/spark-2.3.1/bin/spark-shell --master local[2]
 
 - For Yarn you should change --master parameter to yarn-client - "--master yarn-client"

The you can copy paste code from the scripts directory to run the examples :

look at and copy code from these files:
Chapter 2  dataset 
Chapter 5 machinelearning
Chapter 9 graphframe1 graphframe2

____________________________________________________________________

Step 4: To submit the code as a spark application: Build project, Copy the jar files

Build project with maven and/or load into your IDE and build. 
You can build this project with Maven using IDEs like Intellij, Eclipse, NetBeans, and then copy the JAR file to your MapR Sandbox, or you can install Maven on your sandbox and build from the Linux command line, 
for more information on maven, eclipse or netbeans use google search. 

This creates the following jar in the target directory.

mapr-spark-flightdelay-1.0.jar 

After building the project on your laptop, you can use scp to copy your JAR file from the project target folder to the MapR Sandbox:

From your laptop command line or with a scp tool :

use userid: mapr and password: mapr.

For VMWare use:  $ scp  nameoffile.jar  mapr@ipaddress:/home/mapr/. 

For Virtualbox use:  $ scp -P 2222 target/*.jar  mapr@127.0.0.1:/home/mapr/.  

this will put the jar file into the mapr user home directory: 
/home/mapr/


_____________________________________________________________________

Step 5:

 To run the application code for Chapter 2 Datasets,  DataFrames and Spark SQL


From the Sandbox command line :

/opt/mapr/spark/spark-2.2.1/bin/spark-submit --class dataset.Flight --master local[2]  mapr-spark-flightdelay-1.0.jar 

This will read  from the file "/mapr/demo.mapr.com/data/flights20170102.json" 

You can optionally pass the file as an input parameter   (take a look at the code to see what it does)

____________________________________________________________________

 To run the application code for Chapter 5 Machine Learning Classification


From the Sandbox command line :

/opt/mapr/spark/spark-2.2.1/bin/spark-submit --class machinelearning.Flight --master local[2]  mapr-spark-flightdelay-1.0.jar 

This will read  from the file mfs:///mapr/demo.mapr.com/data/uber.csv  

You can optionally pass the file as an input parameter   (take a look at the code to see what it does)

____________________________________________________________________

 To run the application code for Chapter 9 GraphFrames


From the Sandbox command line :

/opt/mapr/spark/spark-2.2.1/bin/spark-submit --packages graphframes:graphframes:0.5.0-spark2.1-s_2.11 --class graphx.Flight --master local[2]  mapr-spark-flightdelay-1.0.jar 

This will read  from the file mfs:///mapr/demo.mapr.com/data/uber.csv  

You can optionally pass the file as an input parameter   (take a look at the code to see what it does)

_____________________________________________________________________
Additional code examples not in book:

Structured Streaming with MapR-ES and MapR-DB :

use the mapr command line interface to create a stream, a topic, get info and create a table:

maprcli stream create -path /apps/stream -produceperm p -consumeperm p -topicperm p
maprcli stream topic create -path /apps/stream -topic flights  

to get info on the ubers topic :
maprcli stream topic info -path /apps/stream -topic flights

Create the MapR-DB Table which will get written to

maprcli table create -path /apps/flighttable -tabletype json -defaultreadperm p -defaultwriteperm p

Run the Streaming code to publish events to the topic:

java -cp ./mapr-spark-flightdelay-1.0.jar:`mapr classpath` streams.MsgProducer

This client will read lines from the file in "/mapr/demo.mapr.com/data/uber.csv" and publish them to the topic /apps/uberstream:ubers. 
You can optionally pass the file and topic as input parameters <file topic> 

Optional: run the MapR Streams Java consumer to see what was published :

java -cp mapr-spark-flightdelay-1.0.jar:`mapr classpath` streams.MsgConsumer 

_____________________________________________________________________________

Run the  the Spark Structured Streaming client to consume events enrich them and write them to MapR-DB
(in separate consoles if you want to run at the same time)


/opt/mapr/spark/spark-2.2.1/bin/spark-submit --class stream.StructuredStreamingConsumer --master local[2] \
 mapr-spark-flightdelay-1.0.jar 

This spark streaming client will consume from the topic /apps/uberstream:ubers, enrich from the saved model at
/mapr/demo.mapr.com/data/ubermodel and write to the table /apps/ubertable.
You can optionally pass the  input parameters <topic model table> 
 
You can use ctl-c to stop


In another window while the Streaming code is running, run the code to Query from MapR-DB 

/opt/mapr/spark/spark-2.2.1/bin/spark-submit --class sparkmaprdb.QueryFlight --master local[2] \
 mapr-spark-flightdelay-1.0.jar 

 Use the Mapr-DB shell to query the data

start the hbase shell and scan to see results: 

$ /opt/mapr/bin/mapr dbshell

maprdb mapr:> jsonoptions --pretty true --withtags false

maprdb mapr:> find /apps/flighttable --limit 5


____________________________________________________________________


 To run the application code for the blog GraphFrames with MapR-DB 

Create the MapR-DB Table which will get written to 
(if you created it for the exercise above, delete it maprcli table delete -path /apps/flighttable)

maprcli table create -path /apps/flighttable -tabletype json -defaultreadperm p -defaultwriteperm p

/opt/mapr/spark/spark-2.3.1/bin/spark-shell --class graphmaprdb.WriteFlight --master local[2]  mapr-spark-flightdelay-1.0.jar 

This will read  from the file "/mapr/demo.mapr.com/data/flightdata2018.json" 
and write to /apps/flighttable  you can pass in the file and table name as arguments

To read from MapR-DB into GraphFrames

/opt/mapr/spark/spark-2.3.1/bin/spark-shell --class graphmaprdb.Flight --master local[2]  mapr-spark-flightdelay-1.0.jar 

This will read  from the /apps/flighttable and the airports.json file  you can pass in the file and table name as arguments

