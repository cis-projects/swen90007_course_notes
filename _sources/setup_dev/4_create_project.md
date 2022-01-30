# Step 4: Create Project in IntelliJ

```{attention}
First confirm you have a Java Virtual Machine (JVM) installed on your machine. For Macs, the JVM can normally be found 
in the following directory: {Macintosh HD}/Library/Java/JavaVirtualMachines. If there is no JVM installed on your 
machine, please download one first before attempting these steps.
```

Launch IntelliJ and select New Project:

![](resources/4_create_project_1.png)

Select Java Enterprise and enter the information as shown below.

```{important}
Make sure you create the project in a directory you will remember. You will need to push it to GitHub.
```

![](resources/4_create_project_2.png)

```{important}
IntelliJ comes bundled with Java Enterprise Edition. If it's not in this list, it means the installation was likely 
corrupted. Try uninstalling and re-installing IntelliJ.
```

In the same window, select New to create a new Application Server:

![](resources/4_create_project_3.png)

Select Tomcat Server:

![](resources/4_create_project_4.png)

Select Browse:

![](resources/4_create_project_5.png)

Navigate to wherever you unzipped the Tomcat server you downloaded in step 2. Select the outermost folder then select 
Open:

![](resources/4_create_project_6.png)

Select OK:

![](resources/4_create_project_7.png)

Select Next:

![](resources/4_create_project_8.png)

Make sure Servlet is selected then select Finish:

![](resources/4_create_project_9.png)

IntelliJ will create a HelloWorld Servlet project by default.

```{important}
Once IntelliJ has finished creating the project (this could take a minute or more), you should test Tomcat has 
been set up successfully.
```

In the upper-right corner, select Run on the Tomcat configuration:

![](resources/4_create_project_10.png)

````{note} If You Cannot Select Run Configuration
:class: dropdown

Select Add Configuration:

![](resources/6_github_clone_4.png)

Add a new TomCat local configuration:

![](resources/6_github_clone_5.png)

Select Configure (Application Server may or may not already be populated - ignore it):

![](resources/6_github_clone_6.png)

Select the Open icon:

![](resources/6_github_clone_7.png)

Select the folder where you unzipped the TomCat directory in [Step 2: Download Tomcat](2_tomcat_download.md):

![](resources/6_github_clone_8.png)

Select OK:
![](resources/6_github_clone_9.png)

Select Deployment:

![](resources/6_github_clone_10.png)

Select the Add icon:

![](resources/6_github_clone_11.png)

Select Artifact:

![](resources/6_github_clone_12.png)

Select demo:war exploded:

![](resources/6_github_clone_13.png)

```{note}
A Web application can be deployed to the TomCat server as an exploded directory where files and folders 
are presented in the file system as separate items. A WAR file is a Web Archive file. An exploded WAR file means 
the structure is the exact same as an archive file but not zipped into an archive form.*
```

Select Apply then OK:

![](resources/6_github_clone_14.png)
````

The project should be deployed to localhost:

![](resources/4_create_project_11.png)

To shut down the server, return to IntelliJ and select Stop:

![](resources/4_create_project_12.png)

```{admonition} What's Next
Please proceed to [Step 5: Setup GitHub Repository](5_github_setup.md).
```
