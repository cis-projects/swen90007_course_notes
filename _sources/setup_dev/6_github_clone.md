# Step 6: Clone GitHub Repository

---

At this point, the repository should have been created and can now be cloned by all team members.
You now need to clone the repository to your machine and set up TomCat to run the artifacts.

Follow these steps to clone the repository to a folder on your computer: [view instructions](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository-from-github/cloning-a-repository)

Once you have cloned the repository, launch IntelliJ and select Open:

![](resources/6_github_clone_1.png)

Select the folder you have just cloned from the GitHub repository:

![](resources/6_github_clone_2.png)

You may be prompted if you want IntelliJ to trust this Maven project, select Trust Project:

![](resources/6_github_clone_3.png)

Now select Add Configuration:

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
> *A Web application can be deployed to the TomCat server as an exploded directory where files and folders are presented 
in the file system as separate items. A WAR file is a Web Archive file. An exploded WAR file means the structure is the 
exact same as an archive file but not zipped into an archive form.*

Select Apply then OK:

![](resources/6_github_clone_14.png)

To test the artifact is being deployed to the server correctly, select Add Configuration then select the newly created 
TomCat configuration:

![](resources/6_github_clone_15.png)

Select Run:

![](resources/6_github_clone_16.png)

The program is now deployed to localhost:

![](resources/6_github_clone_17.png)

---

> Please proceed to [Step 7: Connect IntelliJ Project to PostgreSQL](7_connect_intellij_postgresql.md).
