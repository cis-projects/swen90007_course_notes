# Step 7: Connect IntelliJ Project to Local PostgreSQL Instance

## Add PostgreSQL Maven Dependency

You must update the pom.xml file to include the Maven dependency for PostgreSQL:
````
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.14</version>
</dependency>
````

The full file is below for ease. You *should not* paste the entire file as it will overwrite your local properties.
````{admonition} pom.xml File
:class: note, dropdown

```
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
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.14</version>
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
```
````

## Add PostgreSQL to IntelliJ

To connect the project to the local instance of PostgreSQL, open the Database view:

![](resources/7_connect_intellij_postgresql_1.png)

Select 'New':

![](resources/7_connect_intellij_postgresql_2.png)

Select 'PostgreSQL':

![](resources/7_connect_intellij_postgresql_3.png)

Enter the details of your local PostgreSQL instance:

![](resources/7_connect_intellij_postgresql_4.png)

Select 'Test Connection' to test the details you entered are correct:

![](resources/7_connect_intellij_postgresql_5.png)

Select 'OK':

![](resources/7_connect_intellij_postgresql_6.png)

The console will open. Run an SQL query to make sure the database has been connected successfully. Enter any query and select 'Run':

![](resources/7_connect_intellij_postgresql_7.png)

If successful, there will be a tick next to the SQL query, and it will display the query in the console:

![](resources/7_connect_intellij_postgresql_8.png)

The newly created table should be viewable in pgAdmin:

![](resources/7_connect_intellij_postgresql_9.png)

```{admonition} What's Next
The next step is to deploy the application to Heroku. This does not need to be done now - in fact we recommend you 
begin developing the application prior to deployment.

However, if you wish to proceed now please proceed to [Step 8: Deploy Project to Heroku](8_heroku_deploy.md).
```
