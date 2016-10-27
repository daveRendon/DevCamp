# Hands on Lab - Modern Applications (Java)

## Overview

City Power & Light is a sample application that allows citizens to to report "incidents" that have occurred in their community.  It includes a landing screen, a dashboard, and a form for reporting new incidents with an optional photo.  The application is implemented with several components:

* Front end web application contains the user interface and business logic.  This component has been implemented three times in .NET, NodeJS, and Java.
* WebAPI is shared across the front ends and exposes the backend DocumentDB
* DocumentDB is used as the data persistence layer

In this lab, you will work with an existing API to connect to the web application front end. This will allow you perform CRUD operations for incidents. You will also configure additional Azure features for Redis Cache, Azure Storage Queues, and Azure Blob Storage. 

This guide uses [Eclipse](https://www.eclipse.org) for editing, however please feel free to use your editor of choice.

## Objectives

In this hands-on lab, you will learn how to:

* Use Eclipse to connect to an API
* Deploy the application to an Azure Web App
* Modify a view to add caching
* Modify code to add queuing and blob storage

## Prerequisites

* The source for the starter app is located in the HOL\java\modern-cloud-apps\start folder.
* The finished project is located in the HOL\java\modern-cloud-apps\end folder.
* Deployed the starter ARM Template
* Established a development machine either on-premises or in Azure

## Exercises

This hands-on-lab has the following exercises:

* Exercise 1: Integrate the API
* Exercise 2: Add a caching layer
* Exercise 3: Write images to Azure Blob storage

---
### Exercise 1: Integrate the API

1. In your development virtual machine, open a command prompt window and navigate to the `c:\DevCamp\HOL\java\02-modern-cloud-apps\start` folder 
1. Run `gradle eclipse` in the terminal window to restore all dependencies and configure the
   project paths for Eclipse

    ![image](./media/image-001.png)

1. Once package restoration completes, open Eclipse 

    ![image](./media/image-002.png)

    Import the HOL Start folder using the menu item `File/import`, and choose the Gradle project wizard and click `Next`:

    ![image](./media/2016-10-24_14-35-05.png)

    On the gradle welcome page click next, and on the `Import Gradle Project` page, choose the `c:\DevCamp\HOL\java\02-modern-cloud-apps\start` directory.  Click 'Finish`:

    ![image](./media/2016-10-24_14-38-30.png)

    You will be asked whether to use the existing project descriptor, or to create a new one.  Keep the existing one:

    ![image](./media/2016-10-24_14-41-33.png)
  

1. Let's run the application in Debug Mode.  Click the Debug icon on
   the top toolbar, then select "Debug Configurations...".

    ![image](./media/image-003.png)

   Click on "Spring Boot App" and click the + icon in the top left to create a new run configuration.  
   
   ![image](./media/2016-10-24_14-46-43.png)
   
   Give the run configuration a name, such as `Run DevCamp App`, choose the Start project, click `Search` and choose devCamp.WebApp.DevcampApplication for the main type.

    ![image](./media/2016-10-24_14-51-00.png)

   Click "Apply" and "Debug".  In the console pane you should see
   something like this:

    ![image](./media/image-003b.png)

1. Open a browser and navigate to `http://localhost:8080`. You should now see the running application

    ![image](./media/image-004.png)

1. On the Dashboard page, notice how the incidents are stubbed in.

    ![image](./media/image-005.png)

    As part of the original ARM template we deployed an ASP.NET WebAPI that queries a DocumentDB Collection. Let's integrate that API so that the incidents are dynamically pulled from a data store.

1. In the [Azure Portal](https://portal.azure.com) navigate to the resource group that you created with the original ARM template.  Resource Groups can be found on the left hand toolbar -> More Services -> Resource Groups.

    Select the API app that begins with the name **incidentsapi** followed by a random string of characters.

    ![image](./media/image-006.png)

1. The window that slides out is called a **blade** and contains information and configuration options for the resource.

    On the top toolbar, select **Browse** to open the API in a new browser window.

    ![image](./media/image-007.png)

    You should be greeted by the default ASP.NET landing page. Capture
    the URL in notepad or other text editor.
    ![image](./media/image-008.png)

1. Since we provisioned a new instance of DocumentDB, there are not any records to use as sample data.  To generate sample data, our API has a route that can be hit at any time to reset the documents in our collection.  In the browser, add `/incidents/sampledata` to your API's URL to generate sample documents.  The API will respond with a block of JSON that looks like this:
    ```JSON
    {"Version":{"_Major":1,"_Minor":1,"_Build":-1,"_Revision":-1},"Content":{"Message":"Initialized sample data","Id":"187a8c66-dae6-406a-9d6e-7654459689c4","Timestamp":"2016-10-24T19:55:02.9495008Z","Headers":[{"Key":"Content-Type","Value":["application/json; charset=utf-8"]}]},"StatusCode":200,"ReasonPhrase":"OK","Headers":[],"RequestMessage":null,"IsSuccessStatusCode":true}
    ```

1. After navigating to the sampledata route, let's verify that the documents were created in DocumentDB. In the Azure Portal, navigate to the Resource Group blade and select the DocumentDB resource.

    ![image](./media/image-010.png)

    Select the one database, and then select the **incidents** collection.

    ![image](./media/image-011.png)

    In the Collection blade, select **Document Explorer** from the top toolbar.

    ![image](./media/image-012.png)

    The Document Explorer is an easy way to view the documents inside of a collection via the browser. Select the first record to see the JSON body of the document.

    ![image](./media/image-013.png)

    We can see that several incidents have been created and are now available to the API.

1. Back in Eclipse, let's begin integrating the API into our code.  We will need to query the API's endpoint URL, and we have options of where to store that string.  While we could insert it directly into our code, a better practice is to abstract such a configuration setting into an environment variable.

    Stop the debugger by pressing the red "stop" square, and open the
    run configuration you created earlier.  Click the "Environment"
    tab.  This section defines key/value pairs that will be passed
    into environment variables whenever the debugger is launched. Add
    an entry for `INCIDENT_API_URL` and set the value to the ASP.NET
    WebAPI that we earlier loaded into the browser (and captured in
    notepad). It should look like this: `http://incidentapib6prykosg3fjk.azurewebsites.net/`, but with your own website name.  Click OK to save the
    environment variable, then apply and close.

    ![image](./media/image-009.png)

    Now that the URL is loaded as an environment variable, we can
    access it from our application by calling `System.getenv("INCIDENT_API_URL")`.  We will repeat this process several times to configure our application with Azure services.

    > Our ARM Template already configured an environment variable for the Azure Web App that will soon run our application

1. Several components will work together to call and display the
   incidents in the database.  First we will need an object to hold
   the data associated with each incident.  We've supplied that object
   in devCamp.WebApp.IncidentAPIClient.Models.IncedentBean.Java.  Open
   that file and look at the properties and methods.

1. Create the class `devCamp.WebApp.IncidentAPIClient.IncidentAPIClient.java` with the
   following code:

    ```java
    package devCamp.WebApp.IncidentAPIClient;

    import java.util.List;

    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.springframework.cache.annotation.CacheEvict;
    import org.springframework.core.ParameterizedTypeReference;
    import org.springframework.http.HttpMethod;
    import org.springframework.http.ResponseEntity;
    import org.springframework.http.converter.StringHttpMessageConverter;
    import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
    import org.springframework.web.client.RestTemplate;

    import devCamp.WebApp.IncidentAPIClient.Models.IncidentBean;

    public class IncidentAPIClient {
        private Log log = LogFactory.getLog(IncidentAPIClient.class);
        private String baseURI;

        public String getBaseURI() {
            return baseURI;
        }

        public void setBaseURI(String baseURI) {
            this.baseURI = baseURI;
        }

        public IncidentBean CreateIncident(IncidentBean incident) {
            //call REST API to create the incident
            final String uri = baseURI+"/incidents";
            RestTemplate restTemplate = new RestTemplate();
            restTemplate.getMessageConverters().add(new MappingJackson2HttpMessageConverter());
            restTemplate.getMessageConverters().add(new StringHttpMessageConverter());
            
            IncidentBean createdBean = restTemplate.postForObject(uri, incident, IncidentBean.class);
            return createdBean;
        }

        public List<IncidentBean> GetAllIncidents() {
            log.info("Performing get /incidents web service");
            final String uri = baseURI+"/incidents";
            RestTemplate restTemplate = new RestTemplate();

            ResponseEntity<List<IncidentBean>> IncidentResponse =
                    restTemplate.exchange(uri,
                                HttpMethod.GET, null, new ParameterizedTypeReference<List<IncidentBean>>() {
                        });

            return IncidentResponse.getBody();
        }

        public IncidentBean GetById(String incidentId) {
                //call REST API to create the incident
                final String uri = String.format("%s/incidents/%s", baseURI,incidentId);
                RestTemplate restTemplate = new RestTemplate();
                restTemplate.getMessageConverters().add(new MappingJackson2HttpMessageConverter());
                restTemplate.getMessageConverters().add(new StringHttpMessageConverter());
                
                IncidentBean retval = restTemplate.getForObject(uri, IncidentBean.class);
                
                return retval;		
            }
        
            public IncidentBean UpdateIncident(String incidentId,IncidentBean newIncident){
                //call REST API to create the incident
                final String uri = baseURI+"/incidents";
                RestTemplate restTemplate = new RestTemplate();
                restTemplate.getMessageConverters().add(new MappingJackson2HttpMessageConverter());
                restTemplate.getMessageConverters().add(new StringHttpMessageConverter());
                
                IncidentBean retval = null;
                return retval;		
            }
            
        public IncidentAPIClient(String baseURI) {
            if (baseURI == null){
                //throw argument null exception
            }
            this.baseURI = baseURI;

        }
    }

    ```

    This class uses the
    [RestTemplate](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html) library
    to generate a HTTP GET to the API endpoint, and to convert the
    return javascript into a java object.  In this case, we've
    specified that it should return a `List<IncidentBean>`.

    The class also contains functions to create a new incident, update an incident, and get a single incident by ID.  These 
    invoke the appropriate REST api call.

1.  Next, create an object to create an IncidentAPIClient with the
    proper URI. Create the class `devCamp.WebApp.Utils.IncidentAPIHelper`:

    ```java
    package devCamp.WebApp.Utils;

    import devCamp.WebApp.IncidentAPIClient.IncidentAPIClient;

    public class IncidentAPIHelper {
        public static IncidentAPIClient getIncidentAPIClient() {

            String apiurl= System.getenv("INCIDENT_API_URL");
            return new IncidentAPIClient(apiurl);
        }
    }
    ```

    >this class will create an incident of the IncidentAPIClient, configured so that it will pull the configuration from the OS environment variable.

1. Open `DevCamp.WebApp.Controllers.DashboardController.java`. The dashboard function in this class is called when the user hits the `/dashboard` url.  In the function we are currently populating some dummy data to display on the dashboard.  We are going to change this to call the API, and display the retrieved data in the dashboard.

    In the dashboard function,
    comment out this section of code that generated the dummy
    dashboard data:

    ```java
    ArrayList<IncidentBean> theList = new ArrayList<>();
    for (int i = 1;i<=3;++i){
        IncidentBean bean = new IncidentBean();
        bean.setId("12345");
        bean.setStreet("123 Main St.");
        bean.setFirstName("Jane");
        bean.setLastName("Doe");
        bean.setCreated("1/01/2016");
        theList.add(bean);
    }
    ```

    Insert this code to call the GetAllIncidents API and put the
    resulting list of IncidentBean in the model.

    ```java
    IncidentAPIClient client = IncidentAPIHelper.getIncidentAPIClient();
    List<IncidentBean> theList = client.GetAllIncidents();
    model.addAttribute("allIncidents",theList);
    ```

    > You will notice that IncidentAPIClient and IncidentAPIHelper will be underlined in red, indicating that they are currently undefined in this class.  Either click on the red icon with the X on the left side of the code, or hover over the code to get the error correction menu.  For each, choose the appropriate import to add to this class:

    ![image](./media/2016-10-24_21-03-58.png)

1. In addition to displaying incidents, the application also provides a form to enter in new Incidents.  
The POST from the form is handled by the `IncidentController.java` class.  
Scroll to the Create function of `devCamp.WebApp.Controllers.IncidentController.java` and locate this line of code:
    ```java
    IncidentBean result = null;
    ```

    Change this to call the function in the IncidentAPIClient:
    ```java
    IncidentBean result = IncidentAPIHelper.getIncidentAPIClient().CreateIncident(incident);
    ```

    Before we test this code, lets take a look at the HTML template for the dashboard
    page, located in
    `src/main/resources/templates/Dashboard/index.html`. The following
    section loops through all of the incidents in the allIncidents
    object in the model, and formats them nicely for the display.

    ```HTML
    <div th:each="incident : ${allIncidents}">
        <div class="col-sm-4">

            <div class="panel panel-default">
                <div class="panel-heading">
                    Outage <span th:text="${incident.Id}"></span>
                </div>
                <table class="table">
                    <tr>
                        <th>Address</th>
                        <td><span th:text="${incident.Street}"></span></td>
                    </tr>
                    <tr>
                        <th>Contact</th>
                        <td><a href="tel:14174444444"><span
                                th:text="${incident.FirstName}"></span> <span
                                th:text="${incident.LastName}"></span></a></td>
                    </tr>
                    <tr>
                        <th>Reported</th>
                        <td><span th:text="${incident.Created}"></span></td>
                    </tr>
                </table>
            </div>
        </div>
    </div>
    ```

    >We aren't making any changes to this file at this point, we are just verifying that the dashboard display simply pulls all the incidents in the model object, and formats them for HTML display.

1. Run the application via the Debug Tab in Eclipse and check the
   dashboard page at http://localhost:8080/dashboard.

    ![image](./media/image-015.png)

The cards now represent data returned from our API, replacing the static mockup code.  You can also click on `Report Outage`, enter the information requested, then come back to the dashboard display to verify that your outage was saved.

---
## Exercise 2: Add a caching layer
Querying our API is a big step forward, but caching the data in memory would increase 
performance and reduce the load on our API.  Azure offers a managed (PaaS) 
service called [Azure Redis Cache](https://azure.microsoft.com/en-us/services/cache/).

We deployed an instance of Azure Redis Cache in the ARM Template, but
need to add application logic. Spring has great support for caching,
and can easily use Azure Redis Cache to hold the data.


1. First, let's add our Redis information to local environment variables. In the [Azure Portal](https://portal.azure.com) navigate to the Resource Group and select the Redis instance.

    ![image](./media/image-016.png)

    On the Redis blade, note the **Host Name**, then select the **key icon** and note the **Primary Key**.

    ![image](./media/image-017.png)

    On the Redis blade, expand **Ports* and note the Non-SSL port 6379 and SSL Port of 6380.

    ![image](./media/image-018.png)

    In Eclipse open the run configuration, click the environment tab
    and add four variables for `REDISCACHE_HOSTNAME`,
    `REDISCACHE_PRIMARY_KEY`, `REDISCACHE_PORT`, and
    `REDISCACHE_SSLPORT`.  Click apply and close.  Your environment variables should look like this:

    ![image](./media/2016-10-24_21-33-35.png)

    We will use these variables to configure a Redis client.

1. To add caching support to your Spring application, open the build.gradle
   file and add the following entries under dependencies:
   ```java
    compile("javax.cache:cache-api")
    compile('org.springframework.data:spring-data-redis')
    compile('redis.clients:jedis')
    compile('org.springframework.boot:spring-boot-starter-cache')
    ```

    To make sure that Eclipse knows about the new packages we added to
    the buld, run the `ide/eclipse` gradle task in the `gradle tasks`
    window. Then right-click on the project in the project explorer,
    close the project, and then open it again.

    In Spring, you can apply caching to a Spring
   [Service](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html). 
   We need to create a Java class to represent this service, so create a new Java class named 
   `devCamp.WebApp.IncidentAPIClient.IncidentService.java` with this code:

   ```java

    package devCamp.WebApp.IncidentAPIClient;

    import java.util.List;

    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.springframework.cache.annotation.CacheEvict;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.stereotype.Service;

    import devCamp.WebApp.IncidentAPIClient.Models.IncidentBean;
    import devCamp.WebApp.Utils.IncidentAPIHelper;

    @Service
    public class IncidentService {

        private Log log = LogFactory.getLog(IncidentService.class);

        @Cacheable("incidents")
        public List<IncidentBean> GetAllIncidents() {
            IncidentAPIClient client = IncidentAPIHelper.getIncidentAPIClient();
            return client.GetAllIncidents();
        }

        @CacheEvict(cacheNames="incidents", allEntries=true)
        public IncidentBean CreateIncident(IncidentBean incident) {
            return IncidentAPIHelper.getIncidentAPIClient().CreateIncident(incident);		
        }
        
        @CacheEvict(cacheNames="incidents", allEntries=true)
        public IncidentBean UpdateIncident(String incidentId,IncidentBean newIncident){
            return IncidentAPIHelper.getIncidentAPIClient().UpdateIncident(incidentId,newIncident);
        }
        
        public IncidentBean GetById(String incidentId) {
            return IncidentAPIHelper.getIncidentAPIClient().GetById(incidentId);		
        }

        @CacheEvict(cacheNames="incidents", allEntries=true)
        public void ClearCache() {
        }
        
    }
   ```

    The `@Service` annotation tells Spring that this is a service
    class, and the `@Cacheable` annotation tells spring that the
    result of the GetAllIncidents is cachable and will automatically
    use the cached version if available.

    The `@CacheEvict` annotation on the other API calls tells Spring to clear the cache 
    when those functions are called; these are the ones that make changes to the Incident database.

    We still have to configure Spring caching to use Azure Redis
    Cache. To do this, create a new class
    devCamp.WebApp.CacheConfig.java with this code:

    ```java

    package devCamp.WebApp;

    import java.util.Arrays;
    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.annotation.CachingConfigurerSupport;
    import org.springframework.cache.annotation.EnableCaching;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.data.redis.cache.RedisCacheManager;
    import org.springframework.data.redis.connection.RedisConnection;
    import org.springframework.data.redis.connection.RedisConnectionFactory;
    import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
    import org.springframework.data.redis.core.RedisTemplate;

    import redis.clients.jedis.JedisPoolConfig;

    @Configuration
    @EnableCaching
    public class CacheConfig extends CachingConfigurerSupport {
        private Log log = LogFactory.getLog(CacheConfig.class);

        @Bean
        public JedisConnectionFactory redisConnectionFactory() {
                JedisPoolConfig poolConfig = new JedisPoolConfig();
                poolConfig.setMaxTotal(5);
                poolConfig.setTestOnBorrow(true);
                poolConfig.setTestOnReturn(true);
                JedisConnectionFactory ob = new JedisConnectionFactory(poolConfig);
                ob.setUsePool(true);
                String redishost = System.getenv("REDISCACHE_HOSTNAME");
                log.info("REDISCACHE_HOSTNAME="+redishost);
                ob.setHostName(redishost);
                String redisport = System.getenv("REDISCACHE_PORT");
                log.info("REDISCACHE_PORT="+redisport);
                try {
                    ob.setPort(Integer.parseInt(  redisport));
                } catch (NumberFormatException e1) {
                    // if the port is not in the ENV, use the default
                    ob.setPort(6379);
                }
                String rediskey = System.getenv("REDISCACHE_PRIMARY_KEY");
                log.info("REDISCACHE_PRIMARY_KEY="+rediskey);
                ob.setPassword(rediskey);
                ob.afterPropertiesSet();
                RedisTemplate<Object,Object> tmp = new RedisTemplate<>();
                tmp.setConnectionFactory(ob);

                //make sure redis connection is working
                try {
                    String msg = tmp.getConnectionFactory().getConnection().ping();
                    log.info("redis ping response="+msg);
                } catch (Exception e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                return ob;
            }

        @Bean(name="redisTemplate")
        public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory cf) {
            RedisTemplate<String, String> redisTemplate = new RedisTemplate<String, String>();
            redisTemplate.setConnectionFactory(cf);
            return redisTemplate;
        }

        @Bean
        public CacheManager cacheManager() {
            RedisCacheManager manager =new RedisCacheManager(redisTemplate(redisConnectionFactory()));
            manager.setDefaultExpiration(300);
            return manager;
        }
    }
    ```

    There is a lot going on in this class.  The `@Configuration`
    annotation tells Spring that this class declares one or more beans
    that will generate bean and service definitions.  The
    `@EnableCaching` annotation enables Spring's annotation driving
    caching mechanism for the application.

    The `CacheConfig` class contains beans that will configure the
    annotation driven caching. The `redisConnectionFactory` function
    creates a new `JedisConnectionFactory` with the appropriate
    connection to the Azure Redis cache. It also does a test to make
    sure it is properly communicating with the cache.

    The `cacheManager` function configures Spring to use the
    redisConnectionFactory function to connect to the cache.  It also
    configures the default cache expiration time to 300 seconds.

    All application requests for the dashboard will now first try to
    use Azure Redis Cache. Under high traffic, this will improve page
    performance and decrease the API's scaling needs.

1. Change the `devCamp.WebApp.Controllers.DashboardController.java`
class to use the IncidentService rather than the IncidentAPIClient
directly. To do this add these lines inside the DashboardController class:

    ```java
    @Autowired
    IncidentService service;
    ```
    >you will also have to resolve the import for devCamp.WebApp.IncidentAPIClient.IncidentService

    Also, change these two lines:
    ```java
    IncidentAPIClient client = IncidentApiHelper.getIncidentAPIClient();
    ArrayList<IncidentBean> theList = client.GetAllIncidents();
    ```
    to this:

    ```java
    List<IncidentBean> theList = service.GetAllIncidents();
    ```

    These changes will ensure that the IncidentAPI is called via the service, rather than directly.  This is required to make Spring caching annotations work properly.

1. We need to change the `IncidentController.java` class to use the IncidentService 
   object also.  Add these lines inside the class definition:

    ```java
    @Autowired
    IncidentService service;
    ```

    Find this line inside of the create function:

      ```java
    IncidentBean result = IncidentAPIHelper.getIncidentAPIClient().CreateIncident(incident);
    ```

    Change it to this:
    ```java
    IncidentBean result = service.CreateIncident(incident);
    ```

   >Again, you will have to resolve the import for devCamp.WebApp.IncidentAPIClient.IncidentService

   These changes ensure that we call the IncidentAPI via the service, rather than directly.

1. To test the application using the Azure Redis Cache, note that in
   the IncidentAPIClient class, the `GetAllincidents` function has
   this code at the top:

   ```java
    log.info("Performing get /incidents web service");
   ```

   This will print a log message every time the API is called. Start
   the application and in your browser go to
   `http://localhost:8080/dashboard`. Look at your console out window
   in Eclipse, it should end with a line that says

   ```
   Performing get /incidents web service
   ```

If you refresh your page in the browser, you should not get another log message, since the actual API code will not be called for 300 seconds.

> if you refresh or click `Dashboard` before the previous request has completed, you may get two log messages indicating the web service has been called.  This is expected behavior, since the write to cache will happen when the request has completed.

---
### Exercise 3: Write images to Azure Blob Storage

When a new incident is reported, the user can attach a photo.  In this exercise we will process that image and upload it into an Azure Blob Storage Container.

1. The [Azure Storage SDK](https://github.com/Azure/azure-storage-java) 
    makes it easy to access Azure storage from within Azure applicatons.
    First, lets establish environment variables that we can use in the applicaiton
    for configuration.  To get the necessary values, open the [Azure Portal](https://portal.azrue.com) and open the Resource Group.  Select the Storage Account beginning with `incidentblobstg`.

    > The other storage accounts are used for diagnostics data and virtual machine disks

    ![image](./media/image-019.png)

    Select **Access Keys** and note the **key1** for the storage account.

    ![image](./media/image-020.png)

     In Eclipse open the run configuration, click the environment tab
    and add the following environment variables:
    * `AZURE_STORAGE_ACCOUNT` is the name of the Azure Storage Account resource
    * `AZURE_STORAGE_ACCESS_KEY` is **key1** from the Access Keys blade
    * `AZURE_STORAGE_BLOB_CONTAINER` is the name of the container that will be used. Storage Accounts use containers to group 
    sets of blobs together.  For this demo let's use `images` as the Container name
    * `AZURE_STORAGE_QUEUE` is the name of the Azure Storage Queue resource.  For this demo we will use `thumbnails`.

    Your `Run configurations` window in Eclipse should contain these environment variables:

    ![image](./media/2016-10-24_22-02-32.png)

    Add the following lines to the dependencies in build.gradle:
    ```java
    compile('com.microsoft.azure:azure-storage:4.4.0')
    compile('com.microsoft.azure:azure-svc-mgmt-storage:0.9.5')
    ```

    Run the `ide/eclipse` gradle task in the `gradle tasks`
    window. Then right-click on the project in the project explorer,
    close the project, and then open it again.

1. Today we are working with Azure Storage Blobs, but in the future we may 
decide to extend our application use Azure Stage Tables or Azure Storage 
Queues.  To better organize our code, let's create a storage interaction 
class.  Create `devCamp.WebApp.StorageAPIClient.StorageAPIClient.java` and 
paste in the following code: 

    ```java
    package devCamp.WebApp.StorageAPIClient;

    import org.apache.commons.io.FilenameUtils;
    import org.codehaus.jettison.json.JSONException;
    import org.codehaus.jettison.json.JSONObject;
    import org.springframework.web.multipart.MultipartFile;
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;
    import com.microsoft.azure.storage.queue.CloudQueue;
    import com.microsoft.azure.storage.queue.CloudQueueClient;
    import com.microsoft.azure.storage.queue.CloudQueueMessage;

    import java.io.*;
    import java.net.URISyntaxException;
    import java.security.InvalidKeyException;

    import javax.ws.rs.core.UriBuilder;

    public class StorageAPIClient {

        //configuration values from the system Environment
        private String account;
        private String key;
        private String azureStorageContainer;
        private String azureStorageQueue;
        private String blobStorageConnectionString;
        
        public StorageAPIClient(String account, String key, String azureStorageContainer, String azureStorageQueue) {
            this.account = account;
            this.key = key;
            this.azureStorageContainer = azureStorageContainer;
            this.azureStorageQueue = azureStorageQueue;
            blobStorageConnectionString = String.format("DefaultEndpointsProtocol=http;AccountName=%s;AccountKey=%s", account,key);
        }

	
        public void AddMessageToQueue(String IncidentId, String ImageFileName)
        {
            CloudStorageAccount storageAccount;
            try {
                storageAccount = CloudStorageAccount.parse(blobStorageConnectionString);
                CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

                CloudQueue msgQ = queueClient.getQueueReference(azureStorageQueue);
                msgQ.createIfNotExists();
        
                JSONObject json = new JSONObject()
                .put("IncidentId", IncidentId)
                .put("BlobContainerName", azureStorageContainer)
                .put("BlobName",getIncidentBlobFilename(IncidentId,ImageFileName));
        
                String msgPayload = json.toString();
                CloudQueueMessage qMsg = new CloudQueueMessage(msgPayload);
                msgQ.addMessage(qMsg);
            } catch (InvalidKeyException | URISyntaxException | StorageException | JSONException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
	
        public  String UploadFileToBlobStorage(String IncidentId, MultipartFile imageFile){
            CloudStorageAccount account;
            try {
                account = CloudStorageAccount.parse(blobStorageConnectionString);
                CloudBlobClient serviceClient = account.createCloudBlobClient();

                // Container name must be lower case.
                CloudBlobContainer container = serviceClient.getContainerReference(azureStorageContainer);
                container.createIfNotExists();

                // Set anonymous access on the container.
                BlobContainerPermissions containerPermissions = new BlobContainerPermissions();
                containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);
                container.uploadPermissions(containerPermissions);

                // 
                CloudBlockBlob imgBlob = container.getBlockBlobReference(getIncidentBlobFilename(IncidentId,imageFile.getOriginalFilename()));
                imgBlob.getProperties().setContentType(imageFile.getContentType());
                imgBlob.upload(imageFile.getInputStream(),imageFile.getSize());
                UriBuilder builder = UriBuilder.fromUri(imgBlob.getUri());
                builder.scheme("https");
                return builder.toString();
            } catch (InvalidKeyException | URISyntaxException | StorageException | IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } 
            return null;
        }

        private String getIncidentBlobFilename(String IncidentId,String FileName) {
            String fileExt = FilenameUtils.getExtension(FileName);
            // TODO check this against the .NET code - 
            //	with this code in, we're generating filenames that don't have a period between the incident id and the extension 
    //		if (fileExt.startsWith(".")){
    //			fileExt = fileExt.substring(1);
    //		}
            return String.format("%s.%s", IncidentId,fileExt);
        }
    }


    ```

1. Next lets create a configuration class to handle retrieving the 
environment variables and creating a StorageAPIClient. Create 
`devCamp.WebApp.Utils.StorageAPIHelper.java` and paste in the following code:
    ```java
    package devCamp.WebApp.Utils;

    import devCamp.WebApp.StorageAPIClient.StorageAPIClient;

    public class StorageAPIHelper {

        public static StorageAPIClient getStorageAPIClient() {

            String account = System.getenv("AZURE_STORAGE_ACCOUNT");;
            String key = System.getenv("AZURE_STORAGE_ACCESS_KEY");;
            String queue = System.getenv("AZURE_STORAGE_QUEUE");
            String container = System.getenv("AZURE_STORAGE_BLOB_CONTAINER");
            
            return new StorageAPIClient(account, key,container,queue);	
        }
    }

    ```

1. Now lets arrange for the controller that manages new incidents to call the Storage API.  Open up 
`devCamp.WebApp.Controllers.IncidentController.java`.  Find the code inside the Create function:
    ```java
        if (fileName != null) {
    ```

    Add this code after the if statement:
    ```java
        //now upload the file to blob storage 
        log.info("uploading to blob");
        StorageAPIHelper.getStorageAPIClient().UploadFileToBlobStorage(IncidentID, imageFile);
        //add a event into the queue to resize and attach to incident
        log.info("adding to queue");
        StorageAPIHelper.getStorageAPIClient().AddMessageToQueue(IncidentID, fileName);
    ```

1. We should be ready to test the storage changes at this point.  Run or debug the application within Eclipse, and open a browser window.  Before you go to the application, use your favorite search engine and download an image you can post with the incident. Then, navigate to `http://localhost:8080/new` (or click on Report Outage).  Fill out the form and hit the **Submit** button.

    ![image](./media/image-021.png)

    You should be redirected to the Dashboard screen, which will contain your new Incident.  

1. Let's install the Microsoft Azure Storage Explorer.  Go to `http://storageexplorer.com/`, 
1. In the Microsoft Azure Storage Explorer, navigate to your Storage Account and ensure that the blob was created.

    ![image](./media/image-022.png)

  You can also use the Azure Storage Explorer to view the `thumbnails` queue, and verify that there is an entry for the image we uploaded.  It is also safe to delete the images and queue entries using Azure Storage Explorer, and enter new Incidents for testing.

Our application can now create new incidents and upload related images to Azure Blob Storage.  It will also put an 
entry into an Azure queue, to invoke an image resizing process, for example. In a later demo, we'll show how 
an [Azure Function](https://azure.microsoft.com/en-us/services/functions/) can be invoked via a queue entry to 
do tasks such as this.

---
Copyright 2016 Microsoft Corporation. All rights reserved. Except where otherwise noted, these materials are licensed under the terms of the MIT License. You may use them according to the license as is most appropriate for your project. The terms of this license can be found at https://opensource.org/licenses/MIT.
