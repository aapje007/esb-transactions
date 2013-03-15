# Transactions

## Overview
This example will show you how to leverage the JTA transaction manager provided by JBoss Fuse when working with JMS
or JTA Camel endpoints.  We will setup a route that reads messages from a queue and inserts information into a database
using JTA and deploy that onto JBoss Fuse 6.0.0.

## What You Will Learn
In studying this example you will learn:
- how to set up an XA-aware DataSource
- how to configure a JTA persistence unit
- how to define a transactional route
- how to leverage JBoss Fuse's JTA and JPA support in your routes

## Prerequisites
Before building and running this example you need:

* Maven 3.0.4 or higher
* JDK 1.6
* JBoss Fuse 6.0.0
* Apache Derby 10.9.1.0 or higher

## Files in the Example
* `pom.xml` - the Maven POM file for building the example
* `database` - contains the persistence unit definition and the JPA entity beans
* `datasource` - contains the JDBC data source definition
* `features` - contains the Apache Karaf features definition that allows for easy installation of this example
* `routing` - contains the transactional Camel routes

For more information about these Maven modules, have a look at the README.md file in every module directory.

## Setting up the database server
For this example, we will be using Apache Derby as our database server.  Before installing the demo, we need to set up
the server and create the database tables we will be using.

We will refer to the directory that contains your Apache Derby installation as `DERBY_HOME`

### Start the network server
Start Apache Derby's network server with

* on Linux/Unix/MacOS: `DERBY_HOME/bin/startNetworkServer`
* on Windows: `DERBY_HOME\bin\startNetworkServer.bat`

### Create the database tables
Open Derby's interactive shell:

* on Linux/Unix/MacOS: `DERBY_HOME/ij`
* on Windows: `DERBY_HOME\ij.bat`

In the shell, run these two commands:

    ij> connect 'jdbc:derby://localhost:1527/transactions;create=true';
    ij> CREATE TABLE flights (
          number VARCHAR(12) NOT NULL,
          departure VARCHAR(3),
          arrival VARCHAR(3),
          PRIMARY KEY (number)
        );

To be sure the tables got created successfully, run one more command:

    ij> select * from flights;

If the latter command shows you the empty table you just created, you're ready to move along with this demo.  Leave `ij`
running for now, we will use it again later to verify that our messages are being processed correctly.

## Building the Example
In the directory where this README.md file is found, run `mvn clean install` to build the example.

## Running the Example
We will refer to the directory that contains your JBoss Fuse installation as `$FUSE_HOME`.

### Configuring additional users
Before we can start JBoss Fuse, we have to make sure we configure a user we can use later on to connect to the embedded
message broker and send messages to a queue.  Edit the `$FUSE_HOME/etc/users.properties` file and add a line that says:

    admin=admin,admin

The syntax for this line is &lt;userid&gt;=&lt;password&gt;,&lt;group&gt;, so we're creating a user called `admin` with a password `admin`
who's a member of the `admin` group.

### Start JBoss Fuse
Start JBoss Fuse with

* on Linux/Unix/MacOS: `bin/fuse`
* on Windows: `bin\fuse.bat`

### Adding the features repository
To allow for easy installation of the example, we created a features descriptor.  On JBoss Fuse's console, add the
extra features repository with this command:

    FuseESB:karaf@root> features:addurl mvn:org.fusesource.example.transactions/features/1.0-SNAPSHOT/xml/features

### Install the example using the feature
First, install the feature itself using this command:

    FuseESB:karaf@root> features:install transactions-openjpa-demo

Using `osgi:list` in the console, you should now see this demo's bundles at the bottom of the list.


### Using jconsole to send JMS messages
Open `jconsole` and connect to the running JBoss Fuse instance.  If the instance is running locally, connect to
the process called `org.apache.karaf.main.Main`.

On the MBeans tab, navigate to `org.apache.activemq` &rarr; `fusemq` &rarr; `Queue` &rarr; `Input.Flights`.  Send a few
messages to the queue using the `sendTextMessage(String body, String user, String password)` operation.  For the second
and third parameter, use the username and password you configured earlier.  The first parameter will become the flight ID
in the database, so just use your imagination for that one ;)

### Verifying the result
Now, head back to `ij` and run this SQL query:

    ij> select * from flights;

You will see new database rows for every message you sent, using the message body as the flight number.

## More information
For more information see:

* JBoss Fuse 6.0.0 - [EIP Component Reference](https://access.redhat.com/knowledge/docs/en-US/JBoss_Fuse/6.0.EA/html/EIP_Component_Reference/files/front.html)