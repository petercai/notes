

To configure **Tomcat 9** to allow **file system browsing**, you essentially need to set up a **web application** or a **context** that maps to a directory on your file system and serves its contents over HTTP. Here's a step-by-step guide:

---

### ✅ Option 1: Use the Default `webapps` Directory

Tomcat serves static files from the `webapps` directory by default. You can:

1. **Create a folder** inside `webapps`, e.g., `filebrowser`.
2. **Place files or subdirectories** you want to browse inside `webapps/filebrowser`.
3. Access it via: [http://localhost:8080/filebrowser/](http://localhost:8080/filebrowser/)

Tomcat will serve static files and allow browsing if `index.html` is missing and directory listings are enabled.

---

### ✅ Option 2: Enable Directory Listings

By default, Tomcat **disables directory listings**. To enable them:

1. Open the `web.xml` file located in: $CATALINA_HOME/conf/web.xml
    
2. Find the `<servlet>` definition for the **default servlet**:
    
    <servlet>
    
        <servlet-name>default</servlet-name>
    
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    
        …
    
    </servlet>
    
3. Add or modify the `<init-param>`:
    
    <init-param>
    
        <param-name>listings</param-name>
    
        <param-value>true</param-value>
    
    </init-param>
    
4. Restart Tomcat.
    

---

### ✅ Option 3: Create a Custom Context to Map to Any Directory

If you want to serve files from a directory **outside** `webapps`:

1. Create a new context XML file in: $CATALINA_HOME/conf/Catalina/localhost/
    
2. Name it something like `files.xml` and add:
    
    <Context docBase="/absolute/path/to/your/files" path="/files" reloadable="true">
    
        <Parameter name="listings" value="true" override="false"/>
    
    </Context>
    
3. Ensure the directory has read permissions for Tomcat.
    
4. Restart Tomcat and access via: [http://localhost:8080/files/](http://localhost:8080/files/)
    

---

### 🔐 Security Considerations

- Be cautious when exposing file system directories.
- Restrict access using `<security-constraint>` in `web.xml` if needed.
- Avoid exposing sensitive directories.

---

Would you like help creating a sample `Context` file or modifying your `web.xml`? Or are you trying to browse a specific folder like `/var/www/files`?