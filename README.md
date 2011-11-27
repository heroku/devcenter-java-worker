# Run Java worker processes on Heroku

Some applications can benifit from splitting logic into two components?

1. A web component that is consumed by the end-user
2. A non-web component or background process to supplement the web component.

The non-web component of an application is called a [Worker](http://devcenter.heroku.com/articles/process-model#mapping_the_unix_process_model_to_web_apps) process in Heroku.  This article explains the running of a Java worker process on Heroku.

## Prerequisites

* Basic Java knowledge, including an installed version of the JVM and Maven.
* Basic Git knowledge, including an installed version of Git.
* A Java web application. If you don't have one follow the first step to create an example. Otherwise skip that step.

## Components of a Java worker process

A java worker process on Heroku comprises of 4 parts:

1. Application code
2. A maven build file (pom.xml) that defines how the application will be built along with which dependencies are required
3. A Procfile defining how the process is launched

## Create an application

Create a simple Java application using mvn archetype:create:

    :::term
    $ mvn archetype:create -DgroupId=com.myexamples -DartifactId=herokujavaworker

This will create the project directories, "pom.xml" and the associated test directories. This folder structure will look like:

    project
    ¦   pom.xml
    ¦
    +---src
    ¦   +---main
    ¦   ¦   +---java
    ¦   ¦       +---com
    ¦   ¦           +---myexamples
    ¦   ¦                   App.java
    ¦   ¦
    ¦   +---test
    ¦       +---java
    ¦           +---com
    ¦               +---myexamples
    ¦                       AppTest.java


A class called App.java is also created. This is the main entry point for the application. The App.java that maven creates will look like:

    :::java
    package com.myexamples;
    
    /**
     * Hello world!
     *
     */
    public class App 
    {
        public static void main( String[] args )
        {
            System.out.println( "Hello World!" );
        }
    }


## Configure Maven

Add the [maven appassembler](http://mojo.codehaus.org/appassembler/appassembler-maven-plugin/) plugin to the pom.xml:

    <plugin>
          <groupId>org.codehaus.mojo</groupId>
    	    <artifactId>appassembler-maven-plugin</artifactId>
    	    <version>1.1.1</version>
    	    <configuration> 
      		  <assembleDirectory>target</assembleDirectory> 
      		  <generateRepository>false</generateRepository>
      		  <extraJvmArguments>-Xmx256m</extraJvmArguments>
      		  <programs>
      			  <program>
      				  <mainClass>com.myexamples.App</mainClass>
      				  <name>app</name>
      			  </program>
      		  </programs>
    	    </configuration>
        	<executions>
        		<execution>
        			<phase>package</phase>
        			<goals>
        				<goal>assemble</goal>
        			</goals>
        		</execution>  			
        	</executions>
        </plugin>



If you have renamed App.java or are using a different class as your main entry point, make sure you change the "mainClass" parameter above to reflect the fully qualified name of that class.

You are now ready to add any additional business logic to your application. 

## Run your Application

To build your application simply run:

    :::term
    $ mvn install

This compiles your java classes and also generates a script called "app.sh" that you can use to run your Java application. To run the applicaiton use the command:

    :::term
    $ sh target/bin/app.sh

That's it. You are now ready to deploy this java application to Heroku.

# Deploy your Application to Heroku

## Create a Procfile

You declare how you want your application executed in `Procfile` in the project root. Create this file with a single line:

    :::term
    worker: sh target/bin/app.sh

## Deploy to Heroku

Commit your changes to Git:

    :::term
    $ git add .
    $ git commit -m "Ready to deploy"

Create the app on the Cedar stack:

    :::term
    $ heroku create --stack cedar
    Creating high-lightning-129... done, stack is cedar
    http://high-lightning-129.herokuapp.com/ | git@heroku.com:high-lightning-129.git
    Git remote heroku added

Deploy your code:

    :::term
    Counting objects: 227, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (117/117), done.
    Writing objects: 100% (227/227), 101.06 KiB, done.
    Total 227 (delta 99), reused 220 (delta 98)

    -----> Heroku receiving push
    -----> Java app detected
    -----> Installing Maven 3.0.3..... done
    -----> Installing settings.xml..... done
    -----> executing .maven/bin/mvn -B -Duser.home=/tmp/build_1jems2so86ck4 -s .m2/settings.xml -DskipTests=true clean install
           [INFO] Scanning for projects...
           [INFO]                                                                         
           [INFO] ------------------------------------------------------------------------
           [INFO] Building petclinic 0.1.0.BUILD-SNAPSHOT
           [INFO] ------------------------------------------------------------------------
           ...
           [INFO] ------------------------------------------------------------------------
           [INFO] BUILD SUCCESS
           [INFO] ------------------------------------------------------------------------
           [INFO] Total time: 36.612s
           [INFO] Finished at: Tue Aug 30 04:03:02 UTC 2011
           [INFO] Final Memory: 19M/287M
           [INFO] ------------------------------------------------------------------------
    -----> Discovering process types
           Procfile declares types -> web
    -----> Compiled slug size is 62.7MB
    -----> Launching... done, v5
           http://pure-window-800.herokuapp.com deployed to Heroku


Congratulations! Your  app should now be up and running on Heroku. To check the status of your app, run the command:

    :::term
    $ heroku ps
    
To look at the application logs, runt he command:

    :::term
    $ heroku logs --tail

# Types of Worker processes

The article covered how to get started with a simple Java worker process. A worker process can be executed in 3 contexts on Heroku:

1. A long running java application that is waiting on events (either through a database or a message queue)
2. A scheduled java application that is invoked through the [Heroku Scheduler](http://addons.heroku.com/scheduler)
3. A one time execution i.e. 1 off admin process.

Each of these contexts are valid uses of a worker process and depending on your use case your could choose to use one of them for your application.

