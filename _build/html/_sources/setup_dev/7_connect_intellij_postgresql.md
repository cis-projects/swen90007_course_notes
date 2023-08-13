# Step 7: Connect IntelliJ Project to Local PostgreSQL Instance

## Add PostgreSQL Maven Dependency

You must update the pom.xml file to include the Maven dependency for PostgreSQL:
````
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.27</version>
</dependency>
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

Check whether the postgres driver is installed, otherwise install it.
![](resources/7_connect_intellij_postgresql_10.jpeg)

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
The next step is to setup Docker and then deploy the application to Render. We recommend you deploy a sample application
on Render as Part 1B assessment and start early on this. 
Proceed to [Step 8: Setup Docker for Java Application](9_setup_docker.md).
```
