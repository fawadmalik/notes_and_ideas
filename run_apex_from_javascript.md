## Here’s a complete JavaScript solution to run Apex code from an external application using Salesforce’s **REST API**.

---

### **Solution Overview**
The solution involves:
1. Authenticating to Salesforce using OAuth2.
2. Calling an Apex REST endpoint that runs your Apex code.

---

### **Prerequisites**
1. **Salesforce Connected App**:
   - Create a connected app in Salesforce.
   - Enable OAuth 2.0 and add the following:
     - Callback URL: `http://localhost:3000` (or your app's domain).
     - Scopes: `Full Access (full)` or `API Access (api)`.

2. **Apex REST Resource**:
   - Expose your Apex code via a REST resource.

   **Example Apex Class**:
   ```java
   @RestResource(urlMapping='/runApex')
   global class ApexExecutor {
       @HttpPost
       global static String executeApex(String input) {
           System.debug('Received input: ' + input);
           // Add Apex logic here
           return 'Executed successfully with input: ' + input;
       }
   }
   ```

---

### **Full JavaScript Solution**

#### **1. Setting Up Authentication**

Use OAuth 2.0 to authenticate and obtain an access token.

```javascript
const axios = require('axios');

const authenticateSalesforce = async () => {
    const authUrl = 'https://login.salesforce.com/services/oauth2/token';
    const params = {
        grant_type: 'password',
        client_id: '<your_client_id>', // Replace with your connected app client ID
        client_secret: '<your_client_secret>', // Replace with your connected app client secret
        username: '<your_username>', // Replace with your Salesforce username
        password: '<your_password>' // Replace with your Salesforce password + security token
    };

    try {
        const response = await axios.post(authUrl, new URLSearchParams(params));
        return {
            accessToken: response.data.access_token,
            instanceUrl: response.data.instance_url
        };
    } catch (error) {
        console.error('Error during authentication:', error.response?.data || error.message);
        throw error;
    }
};
```

---

#### **2. Calling the Apex REST Endpoint**

Once authenticated, use the access token to call the Apex REST resource.

```javascript
const callApexRest = async (instanceUrl, accessToken, input) => {
    const apexRestUrl = `${instanceUrl}/services/apexrest/runApex`;
    const headers = {
        Authorization: `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
    };

    const payload = { input };

    try {
        const response = await axios.post(apexRestUrl, payload, { headers });
        console.log('Apex execution result:', response.data);
        return response.data;
    } catch (error) {
        console.error('Error during Apex REST call:', error.response?.data || error.message);
        throw error;
    }
};
```

---

#### **3. Complete JavaScript Program**

Combine authentication and REST API calls into a single program.

```javascript
const axios = require('axios');

const authenticateSalesforce = async () => {
    const authUrl = 'https://login.salesforce.com/services/oauth2/token';
    const params = {
        grant_type: 'password',
        client_id: '<your_client_id>',
        client_secret: '<your_client_secret>',
        username: '<your_username>',
        password: '<your_password>'
    };

    try {
        const response = await axios.post(authUrl, new URLSearchParams(params));
        return {
            accessToken: response.data.access_token,
            instanceUrl: response.data.instance_url
        };
    } catch (error) {
        console.error('Error during authentication:', error.response?.data || error.message);
        throw error;
    }
};

const callApexRest = async (instanceUrl, accessToken, input) => {
    const apexRestUrl = `${instanceUrl}/services/apexrest/runApex`;
    const headers = {
        Authorization: `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
    };

    const payload = { input };

    try {
        const response = await axios.post(apexRestUrl, payload, { headers });
        console.log('Apex execution result:', response.data);
        return response.data;
    } catch (error) {
        console.error('Error during Apex REST call:', error.response?.data || error.message);
        throw error;
    }
};

const runApex = async () => {
    try {
        const { accessToken, instanceUrl } = await authenticateSalesforce();
        const result = await callApexRest(instanceUrl, accessToken, 'Hello from JavaScript');
        console.log('Final result:', result);
    } catch (error) {
        console.error('Error executing Apex:', error.message);
    }
};

runApex();
```

---

### **Steps to Execute**
1. **Install Node.js**:
   Ensure Node.js is installed on your system.

2. **Install Axios**:
   ```bash
   npm install axios
   ```

3. **Run the Program**:
   Save the code to a file (e.g., `runApex.js`) and execute:
   ```bash
   node runApex.js
   ```

---

### **Expected Output**
When the Apex REST resource is successfully called:
```plaintext
Apex execution result: Executed successfully with input: Hello from JavaScript
Final result: Executed successfully with input: Hello from JavaScript
```

---

### **Best Practices**
1. **Secure Secrets**:
   - Use environment variables to store sensitive data (e.g., `client_id`, `client_secret`).
   - Example:
     ```javascript
     const clientId = process.env.SALESFORCE_CLIENT_ID;
     ```

2. **Error Handling**:
   - Include detailed logging for API errors to diagnose issues quickly.

3. **Timeouts**:
   - Add timeouts to `axios` requests to handle network delays.

4. **Test with Sandbox**:
   - Use a Salesforce Sandbox environment for testing before running against production.

This solution provides a complete workflow for running Apex code through JavaScript using Salesforce's REST API.
