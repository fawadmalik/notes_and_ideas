# How to run Apex code from an external programming language
To run Apex code from an external programming language, you need to interact with the Salesforce platform via its **REST API** or **SOAP API**. The process involves executing the Apex code asynchronously using an **Apex REST resource** or synchronously via the **Execute Anonymous API**.

### **Option 1: Using the REST API**

1. **Expose the Apex Code via an Apex REST Resource**
   Create an Apex class annotated with `@RestResource` to make it callable via an HTTP endpoint.

   **Example Apex Class**:
   ```java
   @RestResource(urlMapping='/runApexCode')
   global class ApexExecutor {
       @HttpPost
       global static String executeApex(String input) {
           // Your Apex logic here
           System.debug('Input received: ' + input);
           return 'Executed successfully with input: ' + input;
       }
   }
   ```

2. **Authenticate to Salesforce**
   Use OAuth 2.0 to get an access token. Authentication involves:
   - Obtaining the client ID and secret from your Salesforce connected app.
   - Using the username-password flow to get a session token.

   **Authentication Example in Python**:
   ```python
   import requests

   auth_url = "https://login.salesforce.com/services/oauth2/token"
   params = {
       'grant_type': 'password',
       'client_id': '<your_client_id>',
       'client_secret': '<your_client_secret>',
       'username': '<your_username>',
       'password': '<your_password>'
   }

   response = requests.post(auth_url, data=params)
   access_token = response.json().get('access_token')
   instance_url = response.json().get('instance_url')
   ```

3. **Call the Apex REST Resource**
   Use the access token to make a POST request to the Apex REST endpoint.

   **Example in Python**:
   ```python
   import requests

   headers = {
       'Authorization': f'Bearer {access_token}',
       'Content-Type': 'application/json'
   }
   payload = {'input': 'Hello from Python'}

   response = requests.post(f'{instance_url}/services/apexrest/runApexCode', headers=headers, json=payload)
   print(response.json())
   ```

---

### **Option 2: Using the SOAP API**

1. **Authenticate Using SOAP**
   Use the Salesforce WSDL (Web Services Description Language) to interact with Salesforce SOAP APIs. Download the WSDL from your Salesforce org and import it into your programming environment.

   **Example Authentication in Java**:
   ```java
   PartnerConnection connection = Connector.newConnection(new ConnectorConfig() {{
       setUsername("<your_username>");
       setPassword("<your_password+security_token>");
       setAuthEndpoint("https://login.salesforce.com/services/Soap/u/57.0");
   }});
   ```

2. **Run Apex Using the Execute Anonymous API**
   Use the `executeAnonymous` method provided by the SOAP API to run your Apex code directly.

   **Example Apex Code Execution in Java**:
   ```java
   String apexCode = "System.debug('Hello from SOAP API!');";
   ExecuteAnonymousResult result = connection.executeAnonymous(apexCode);

   if (result.isSuccess()) {
       System.out.println("Apex executed successfully.");
   } else {
       System.err.println("Error: " + result.getCompileProblem());
   }
   ```

---

### **Option 3: Using Salesforce's Developer Console**

While not directly via an external programming language, the Salesforce **Developer Console** allows you to run Apex code directly:
1. Open Developer Console in Salesforce.
2. Navigate to **Debug > Open Execute Anonymous Window**.
3. Enter your Apex code and execute it.

---

### **Choosing a Programming Language**

You can use almost any programming language that supports HTTP requests or SOAP:
- **Python**: Use libraries like `requests` (REST) or `zeep` (SOAP).
- **Java**: Use Salesforce's `Force.com WSC` library.
- **C#**: Use WSDL-based SOAP services or `HttpClient` for REST.
- **JavaScript/Node.js**: Use `axios` or `fetch` for REST API calls.

---

### **Best Practices**
- Always authenticate securely using OAuth.
- Use parameterized input to prevent injection vulnerabilities.
- Log execution results for debugging and monitoring.
- Consider error handling for API failures or Apex exceptions.
