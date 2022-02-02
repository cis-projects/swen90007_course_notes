# Create Your First Web Application

This tutorial has been made for students unfamiliar with creating web applications. It will familiarise you with
IntelliJ, Java Servlet Pages (JSPs), Maven, and PostgreSQL.

For some people, this may be the first time you have encountered the tools we will be using in the project.
Below is a list of extra resources you can use to become more familiar with them.

| Tool                              | Use                                                                                                                                 | Resources                                                                                                                                                                                          |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| GitHub                            | Version control                                                                                                                     | 1. An interactive and visual tool to learn git commands: [view](https://learngitbranching.js.org)</br>2. GitHub Learning Lab: [view](https://lab.github.com/githubtraining/introduction-to-github) |
| Maven                             | Build automation tool                                                                                                               | Learn how to use Maven in 5-minutes: [view](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)                                                                            |
| Heroku                            | Platform as a Service (PaaS)                                                                                                        | Developer Centre: [view](https://devcenter.heroku.com/categories/reference)                                                                                                                        |
| PostgreSQL                        | Database                                                                                                                            | 1. PostgreSQL tutorial: [view](https://www.postgresqltutorial.com)</br>2. PostgreSQL tutorials and exercises: [view](https://www.postgresql.org/docs/online-resources/)                            |
| Java Database Connectivity (JDBC) | An application programming interface (API) for the programming language Java, which defines how a client may access a database.     |                                                                                                                                                                                                    |
| Java Servlet Pages (JSPs)         | Server-side programming technology that enables the creation of dynamic, platform-independent method for building Web applications. |                                                                                                                                                                                                    |
| Tomcat                            | Server and container to execute Java web applications.                                                                              |                                                                                                                                                                                                    |

```{note}
If you first want to learn the theory behind servlets, JSPs, and containers, check out 
[JSPs and Servlets](jsp_servlets.md).
```

## Create a Web Application in IntelliJ

Please complete the below steps to create your first project:

[Step 1: Install IntelliJ Professional Edition](../setup_dev/1_intellij_install.md)

[Step 2: Download Tomcat](../setup_dev/2_tomcat_download.md)

[Step 3: Setup PostgreSQL](../setup_dev/3_postgresql_setup.md)

[Step 4: Create Project in IntelliJ](../setup_dev/4_create_project.md)

## Create a Servlet

In this example, the project is titled 'test'. Expand the directory as shown below and create a servlet:

![](resources/first_web_app_1.png)

We are going to create a LoginServlet:

![](resources/first_web_app_2.png)

Title the servlet LoginServlet and change the value to be '/login' - the value becomes 
the URL of the servlet. For example, I can access the servlet by running the Tomcat configuration and 
navigating to: localhost.com:8080/login

![](resources/first_web_app_3.png)

In the doGet method, add the following code:
````
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws
ServletException, IOException {
    response.setContentType("text/html");
    System.out.println("Hello from Get method");
    PrintWriter writer = response.getWriter();
    writer.println("<h3> Hello in HTML</h3>");
}
````

![](resources/first_web_app_7.png)

Run the Tomcat configuration:

![](resources/first_web_app_4.png)

If a browser does not launch, open one and navigate to localhost:8080/<project_name>_war_exploded/login:

![](resources/first_web_app_5.png)

You will see the doGet() method you just edited:

![](resources/first_web_app_6.png)

You just created your first servlet!

## Create a JSP

Right click in the webapp directory to create a new JSP:

![](resources/first_web_app_8.png)

Enter a name for the JSP:

![](resources/first_web_app_9.png)

You have created your first JSP:

![](resources/first_web_app_10.png)

However, without a servlet to serve the JSP, it will not be accessible.

In the login JSP you just created, write something (it does not matter what). Something in the body, so it is 
visible when you navigate to the page:

![](resources/first_web_app_12.png)

Open the LoginServlet again and remove the code in the doGet() method and add the following:
````
response.sendRedirect("login.jsp");
````

![](resources/first_web_app_14.png)

Run the Tomcat configuration:

![](resources/first_web_app_4.png)

A browser window should automatically open. Navigate to: localhost:8080/<project_name>_war_exploded/login

![](resources/first_web_app_11.png)

The Login Servlet is now redirecting you to the login JSP you just created:

![](resources/first_web_app_13.png)

### Passing Parameters to a Servlet

There are two ways to pass parameters (arguments) to a servlet:
`````{admonition} Using doGet() Servlet Method
:class: note, dropdown

Parameters are passed to the doGet() method as URL arguments:
````
?[parameterName1]=[parameterValue1]&[parameterName2]=[parameterValue2]
````

In the LoginServlet, remove the doGet() method body and add:
````
response.setContentType("text/html");
System.out.println("Hello from GET method in LoginServlet");
String user = request.getParameter("userName");
String pass = request.getParameter("passWord");
PrintWriter writer = response.getWriter();
writer.println("<h3> Hello from Get "+user+ " " +pass+ "</h3>");
````

![](resources/first_web_app_15.png)

As an example, navigate to:
````
http://localhost:8080/<project_name>_war_exploded/login?userName=luke&passWord=test
````

![](resources/first_web_app_16.png)

The servlet will print the values you passed as parameters:

![](resources/first_web_app_17.png)
`````

`````{admonition} Using doPost() Servlet Method
:class: note, dropdown

Open index.jsp, remove the text in the body and add:
```html
<form action = "login" method = "post">
    User name: <input type = "text" name = "userName"><br/>
    Password: <input type = "password" name = "passWord"><br/>
    <input type = "submit" value = "Login">
</form>
```

![](resources/first_web_app_18.png)

Open the LoginServlet and add the below into the doPost() method:
```
response.setContentType("text/html");
System.out.println("Hello from Post method in LoginServlet");
String user = request.getParameter("userName");
String pass = request.getParameter("passWord");
PrintWriter writer = response.getWriter();
writer.println("<h3> Hello from Post: Your user name is: "+user+", Your password is: " +pass+
        "</h3>");
```

![](resources/first_web_app_19.png)

Run the Tomcat configuration:

![](resources/first_web_app_4.png)

It should load the index.jsp by default:

![](resources/first_web_app_20.png)

Enter a username and password and select Login:

![](resources/first_web_app_21.png)

The index.jsp will post your username and password to the doGet() method of the LoginServlet, which, in turn, will print
them to HTML:

![](resources/first_web_app_22.png)
````
`````

## Create Connection with Local PostgreSQL
```{important}
Make sure you launch pgAdmin and have the database instance running on your computer otherwise all queries 
will fail.
```

You should have already connected to a local PostgreSQL instance.  
To open the database view:

![](resources/first_web_app_23.png)

Open a query console:

![](resources/first_web_app_24.png)

Run the follow SQL query to create a new table for users:
```sql
CREATE TABLE users (
    username    text,
    password text
);
```

It should return successfully, and you should now be able to see the new table in the database view:

![](resources/first_web_app_25.png)

Run the following SQL query to create a test user:
```sql
INSERT INTO users(username, password) VALUES ('lrosa', 'test');
```

Create a new Java file and title it whatever you like:

![](resources/first_web_app_26.png)

![](resources/first_web_app_27.png)

Copy this to the newly created file but make sure to change the database URL, user, and password to match your own:
```
import java.sql.*;

public class JDBCtest {
        private final String url = <insert URL>;
        private final String user = <insert user>;
        private final String password = <insert password>;
        /**
         * Connect to the PostgreSQL database
         * @return a Connection object
         */
        public void connect() {
            String sql = "SELECT * FROM users;";
            PreparedStatement findStatement = null;
            ResultSet rs = null;
            Connection conn = null;
            try {
                DriverManager.registerDriver(new org.postgresql.Driver());
                conn = DriverManager.getConnection(url, user, password);
                findStatement = conn.prepareStatement(sql);
                findStatement.execute();
                rs = findStatement.getResultSet();
                rs.next();
                String username = rs.getString(1);
                System.out.println(username);
            } catch (SQLException e) {
                e.printStackTrace();
            }
    }

    public static void main(String[] args) {
        JDBCtest app = new JDBCtest();
        app.connect();
    }
}
```

![](resources/first_web_app_28.png)

Select Run:

![](resources/first_web_app_29.png)

![](resources/first_web_app_30.png)

It will take a few seconds to run but then should return a successful query:

![](resources/first_web_app_31.png)

You have now created a table in your local PostgreSQL instance, created a connection to it and queried data stored in 
the users table.
You can now build queries on top of this.

## Connect to Heroku PostgreSQL

You must have completed [Step 8: Deploy Project to Heroku](../setup_dev/8_heroku_deploy.md) 
before attempting this.

Now that you've deployed to Heroku, you must change the database credentials in order to access Heroku's PostgreSQL 
instance.

Change the JDBCtest class to be:
```
try {
    DriverManager.registerDriver(new org.postgresql.Driver());
    String DB_CONNECTION = System.getenv().get("JDBC_DATABASE_URL");
    Connection dbConnection = DriverManager.getConnection(DB_CONNECTION);
    return dbConnection;
    } catch (SQLException e) {
        <write error message here>
    }
}
```

For more help, go to [Heroku's Developer Centre](https://devcenter.heroku.com/articles/connecting-to-relational-databases-on-heroku-with-java).
