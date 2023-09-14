# Step 6: Clone GitHub Repository

At this point, the repository should have been created and can now be cloned by all team members.
You now need to clone the repository to your machine and set up TomCat to run the artifacts.

Follow these steps to clone the repository to a folder on your computer: 
[view instructions](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository-from-github/cloning-a-repository)

Once you have cloned the repository, launch IntelliJ and select Open:

![](resources/6_github_clone_1.png)

Select the folder you have just cloned from the GitHub repository:

![](resources/6_github_clone_2.png)

You may be prompted if you want IntelliJ to trust this Maven project, select Trust Project:

![](resources/6_github_clone_3.png)

In the terminal run `mvn clean install`. This will compile the code and package it into a war. Upon successful build, you
can see that a target folder is created with a war file.

Set up the tomcat in your local environment mentioned in [Step 4](4_create_project.md) to deploy your application locally. 

Once you can see that your server is running on port 8080 you can view this in the browser. The project should be
deployed to localhost:

![](resources/4_create_project_11.png)


```{admonition} What's Next
Please proceed to [Step 7: Connect IntelliJ Project to PostgreSQL](7_connect_intellij_postgresql.md).
```
