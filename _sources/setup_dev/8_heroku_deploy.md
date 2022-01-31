# Step 8: Deploy Project to Heroku

## Set Up Repository to Work with Heroku

Open the project in IntelliJ and right-click on the project outermost folder and select New -> File:

![](resources/8_heroku_deploy_10.png)

Title the file 'system.properties':

![](resources/8_heroku_deploy_11.png)

Enter the following into the file and save it:
````
java.runtime.version=11
````

![](resources/8_heroku_deploy_12.png)

Once again, right-click on the project outermost folder and select New -> File:

![](resources/8_heroku_deploy_10.png)

Title the new file 'Procfile':

![](resources/8_heroku_deploy_13.png)

If it prompts you to register file type association, select the first option and press OK:

![](resources/8_heroku_deploy_14.png)

In the file, enter:
````
web: java $JAVA_OPTS -jar target/dependency/webapp-runner.jar $WEBAPP_RUNNER_OPTS --port $PORT target/demo-1.0-SNAPSHOT.war
````

![](resources/8_heroku_deploy_15.png)

## Deploy to Heroku

```{caution}
There are a number of ways to deploy the application to Heroku. You will need to decide as a team which option to 
choose.

***Note that the methods are not cross-compatible - you cannot deploy with more than one option without making changes 
to the project and the repository.***
```

`````{admonition} Option 1: Deploy From GitHub
:class: note, dropdown

```{note}
Only one team member needs to set this up.
```

Hooking Heroku into your GitHub repository is an easy choice and reduces work for all team members, as it does not
require them to set up anything for deployment.
If you choose to hook Heroku to the main branch of the repository, it will rebuild the application on every push to
main. ***Be careful about pushing changes to GitHub after project submission. Changes will be automatically deployed
and might impact the markers' ability to mark your project (i.e., if it
introduces a new bug).***

First create a Heroku account [here](https://www.heroku.com). ***Only one team member needs to do this.***

Select Create New Application:

![](resources/8_heroku_deploy_8.png)

Enter an application name and select Create App:

![](resources/8_heroku_deploy_9.png)

Update the pom.xml file to add the Heroku webapp-runner dependency:
```
<artifactItem>
    <groupId>com.heroku</groupId>
    <artifactId>webapp-runner</artifactId>
    <version>9.0.41.0</version>
    <destFileName>webapp-runner.jar</destFileName>
<artifactItem>
```
The full file is below for ease. You *should not* paste the entire file as it will overwrite your local properties.
````{admonition} pom.xml File
:class: note, dropdown

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>demo</name>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
        <junit.version>5.7.1</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals><goal>copy</goal></goals>
                        <configuration>
                            <artifactItems>
                                <artifactItem>
                                    <groupId>com.heroku</groupId>
                                    <artifactId>webapp-runner</artifactId>
                                    <version>9.0.41.0</version>
                                    <destFileName>webapp-runner.jar</destFileName>
                                </artifactItem>
                            </artifactItems>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>com.heroku.sdk</groupId>
                <artifactId>heroku-maven-plugin</artifactId>
                <version>3.0.3</version>
            </plugin>
        </plugins>
    </build>
</project>
````

> Push this change to the repository.

Login to Heroku and select the Application:

![](resources/8_heroku_deploy_16.png)

Select Deploy:

![](resources/8_heroku_deploy_17.png)

Select Connect to GitHub and follow the prompts:

![](resources/8_heroku_deploy_20.png)

Once connected, select drop-down menu and choose the GitHub Classroom SWEN900072021:

![](resources/8_heroku_deploy_39.png)

> You must be Admin of the project repository in order for Heroku to be able to connect to it.

Select Search. Once Heroku finds the repository, select Connect:

![](resources/8_heroku_deploy_18.png)

![](resources/8_heroku_deploy_19.png)

Follow the prompts to enable automatic deployment and select a branch you wish to deploy *(this is up to your team to
decide)*:

![](resources/8_heroku_deploy_23.png)

It will take up a minute or two to build, but once done, select View to view your application:

![](resources/8_heroku_deploy_24.png)

> Now the code on the branch selected will be deployed to Heroku.
`````

<details>
<summary>Option 2: Deploy From Terminal/IDE</summary>

> All team members should set this up in order to deploy the application.

Download and install the Heroku CLI [here](https://devcenter.heroku.com/articles/heroku-cli).

Before we can deploy to Heroku, we must create a Heroku application. *This part should only be done by one team member
(each team member does not need to have their own Heroku application)*.

Once you have installed the Heroku CLI, open a terminal and log into Heroku:
````
heroku login
````

Before the project can be deployed to Heroku, we need to add a Heroku Maven dependency.
The code to add to the pom file is:
````
<plugin>
    <groupId>com.heroku.sdk</groupId>
    <artifactId>heroku-maven-plugin</artifactId>
    <version>3.0.3</version>
</plugin>
````
The full file is below for ease. You *should not* paste the entire file as it will overwrite your local properties.
<details>
<summary>pom.xml file</summary>

````
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>demo</name>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
        <junit.version>5.7.1</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.1</version>
            </plugin>
            <plugin>
                <groupId>com.heroku.sdk</groupId>
                <artifactId>heroku-maven-plugin</artifactId>
                <version>3.0.3</version>
            </plugin>
        </plugins>
    </build>
</project>
````

</details>

Open a terminal window again, navigate to the root of the Git repository.  
To create the repository as a Heroku application, enter:
````
heroku create
````
This will create the application on Heroku:
    
![](resources/8_heroku_deploy_1.png)
> *You can rename the application (and change the URL) by logging into Heroku and going to settings.*

To deploy it to the URL above, enter the following into terminal:
````
mvn clean heroku:deploy-war
````
It will take a few minutes to build:

![](resources/8_heroku_deploy_2.png)
Now you can go to the URL above, and your project will be deployed.

> Each time you want to deploy to Heroku, you need to run these same commands.

To make life easier, you can create a Heroku configuration in IntelliJ to allow you to deploy changes to the application 
with only one click. ***This is entirely optional.***

Open the project in IntelliJ and select Add Configuration:

![](resources/8_heroku_deploy_3.png)

Select the icon for Add and then scroll down and select Maven:

![](resources/8_heroku_deploy_4.png)

Enter the following into the Command Line:
````
heroku:deploy-war
````

![](resources/8_heroku_deploy_5.png)

Select Run:

![](resources/8_heroku_deploy_6.png)

It will take a few minutes to build, but once done, the application will be viewable at the URL:

![](resources/8_heroku_deploy_7.png)

Now everytime you make changes and want to deploy, you can select Run Configuration.
</details>

<details>
<summary>Option 3: Deploy Using WAR File</summary>

This is not covered, but you can find details [here](https://devcenter.heroku.com/articles/war-deployment#deployment-with-the-heroku-cli).
</details>

> Even after deploying to Heroku, you can still use the local TomCat configuration we created in 
> [Step 4: Setup PostgreSQL](4_create_project.md) to deploy changes locally before pushing to Heroku.

---

### Set Up the PostgreSQL Database

We can no longer use the local instance of the PostgreSQL database, so we have to migrate to a Heroku managed PostgreSQL 
instance.

Log into Heroku and select the application. Then select Resources:

![](resources/8_heroku_deploy_25.png)

Search and select Heroku PostgreSQL:

![](resources/8_heroku_deploy_26.png)

Select Submit Order Form (make sure you have selected the free hobby-development database):

![](resources/8_heroku_deploy_27.png)

Once it has been added on to the application, launch Heroku Postgres

![](resources/8_heroku_deploy_28.png)

Select Settings:

![](resources/8_heroku_deploy_29.png)

Select View Credentials:

![](resources/8_heroku_deploy_30.png)

Launch pgAdmin on your computer. Right-click the server and select Create -> Server:

![](resources/8_heroku_deploy_31.png)

In the pop-up dialog box, enter a name (it does not matter what you choose):

![](resources/8_heroku_deploy_32.png)

Select the Connection tab and enter the credentials provided by Heroku: 

![](resources/8_heroku_deploy_34.png)

![](resources/8_heroku_deploy_33.png)

Select the Advanced tab and enter the name of the database in the DB Restriction field. If you do not do this, you will 
see thousands of other databases managed by Heroku. Select Save:

![](resources/8_heroku_deploy_35.png)

In order to test the connection, select the newly connected database and right-click on Tables and select Query Tool:

![](resources/8_heroku_deploy_36.png)

Enter a random SQL query and click the Run icon:

![](resources/8_heroku_deploy_37.png)

It should return a successful query, and the table should now be visible once you refresh:

![](resources/8_heroku_deploy_38.png)

---
