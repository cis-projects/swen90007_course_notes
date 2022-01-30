# Step 2: Download Tomcat

---

Tomcat provides an HTTP web server environment in which Java code can run.

To download Tomcat 9, follow this link:

> [Apache Tomcat® download](https://tomcat.apache.org/download-90.cgi)

Unzip the downloaded file somewhere easy to access (for example, your home directory).

> **If you are on Mac**, make sure you download the .tar.gz extension file. If you don't, you will likely 
encounter "error=13 Catalina.sh cannot run" when executing the Tomcat server in IntelliJ. This is saying that the IDE 
does not have correct permissions to run the file (this occurs when you use a .zip extension file).

> **If you are on Windows**  
> Setup CATALINA_HOME variable path to your unzipped folder. Example: C:\Users\oliveirae\apache-tomcat-9.0.50  
> 
> *Start and Stop Tomcat Server on Windows 10, 8 and 7*  
> After successful installation, go to BIN folder directly under Tomcat folder. You will find two batch files with 
> names startup.bat and shutdown.bat. You can create shortcuts of these batch files on the desktop or inside Startup 
> Menu for easily starting and stopping Tomcat server whenever required.  
> 
> Alternatively, you can simply use Command Prompt to start and stop Tomcat server:  
> C:\Users\oliveirae\apache-tomcat-9.0.50\bin>startup.bat  
> C:\Users\oliveirae\apache-tomcat-9.0.50\bin>shutdown.bat 
> 
> *Testing Tomcat Installation*  
> Open any web browser like Google Chrome. Go to 'http://localhost:8080'. You should see Tomcat page.

---

> Please proceed to [Step 3: Setup PostgreSQL](3_postgresql_setup.md).
