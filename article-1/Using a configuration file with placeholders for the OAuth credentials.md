To make `dataconnect.js` more flexible and user-friendly, we can introduce a configuration file. This file will hold placeholders for the OAuth credentials, making it easy for users to update their details without modifying the code.

---

### **Step 1: Configuration File**

Create a file named `config.json` in the same directory as `dataconnect.js`. This file will store the OAuth credentials with placeholders for the user to replace.

#### **config.json**
```json
{
  "grant_type": "password",
  "client_id": "your_client_id_here",
  "client_secret": "your_client_secret_here",
  "username": "your_salesforce_username_here",
  "password": "your_salesforce_password_and_security_token_here"
}
```

---

### **Step 2: Update `dataconnect.js`**

Modify `dataconnect.js` to:
1. Read the credentials from `config.json`.
2. Handle cases where `config.json` is missing or improperly configured.
3. Provide clear instructions on how to create the `config.json` file if it’s not found.

#### **Updated `dataconnect.js`**
```javascript
const axios = require('axios');
const qs = require('qs');
const fs = require('fs');
const path = require('path');

/**
 * Authenticate with Salesforce using OAuth2.
 * @returns {Promise<object>} JSON response containing the access token and instance URL.
 */
const authenticateSalesforce = async () => {
    const configPath = path.resolve(__dirname, 'config.json');

    // Check if the configuration file exists
    if (!fs.existsSync(configPath)) {
        console.error(`
Error: Configuration file not found.

Please create a file named "config.json" in the same directory as this script with the following content:

{
  "grant_type": "password",
  "client_id": "your_client_id_here",
  "client_secret": "your_client_secret_here",
  "username": "your_salesforce_username_here",
  "password": "your_salesforce_password_and_security_token_here"
}
        `);
        process.exit(1);
    }

    // Read and parse the configuration file
    let config;
    try {
        config = JSON.parse(fs.readFileSync(configPath, 'utf8'));
    } catch (error) {
        console.error('Error reading configuration file:', error.message);
        process.exit(1);
    }

    // Ensure all required fields are present
    const requiredFields = ['grant_type', 'client_id', 'client_secret', 'username', 'password'];
    for (const field of requiredFields) {
        if (!config[field]) {
            console.error(`Error: Missing required field "${field}" in configuration file.`);
            process.exit(1);
        }
    }

    // Prepare the OAuth request
    const data = qs.stringify(config);
    const requestConfig = {
        method: 'post',
        url: 'https://login.salesforce.com/services/oauth2/token',
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        data: data
    };

    // Make the request
    try {
        const response = await axios.request(requestConfig);
        console.log('Authentication successful:', response.data);
        return response.data; // Return the JSON response
    } catch (error) {
        console.error('Authentication failed:', error.response?.data || error.message);
        throw error;
    }
};

module.exports = authenticateSalesforce;
```

---

### **How This Works**

1. **`config.json` as a Centralized Configuration**:
   - Users update `config.json` with their credentials.
   - If the file is missing, `dataconnect.js` prints detailed instructions on how to create it.

2. **Validation**:
   - Ensures all required fields (`grant_type`, `client_id`, etc.) are present in `config.json`.

3. **Error Handling**:
   - Handles missing file, invalid JSON, and incomplete configurations gracefully.

---

### **Example Workflow**

1. **Create `config.json`**:
   - Add your Salesforce OAuth credentials to the `config.json` file.

2. **Run `dataconnect.js`**:
   ```bash
   node dataconnect.js
   ```

3. **Output**:
   - **If Successful**:
     ```plaintext
     Authentication successful: { access_token: '...', instance_url: '...', ... }
     ```
   - **If Missing or Misconfigured `config.json`**:
     ```plaintext
     Error: Configuration file not found.

     Please create a file named "config.json" in the same directory as this script with the following content:

     {
       "grant_type": "password",
       "client_id": "your_client_id_here",
       "client_secret": "your_client_secret_here",
       "username": "your_salesforce_username_here",
       "password": "your_salesforce_password_and_security_token_here"
     }
     ```

---

### **Benefits**
- **User-Friendly**: Simplifies the process of updating credentials.
- **Reusable**: Configuration file can be reused across projects.
- **Error-Proof**: Validates input and provides clear feedback for missing fields.

With this enhancement, `dataconnect.js` becomes a robust and user-friendly utility for authenticating with Salesforce!
