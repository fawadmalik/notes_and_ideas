# **Getting Started with Salesforce Data Manipulation Using Node.js**

Salesforce is one of the most popular platforms for managing customer relationships, and its powerful API allows developers to programmatically interact with Salesforce data. Whether you're a developer looking to automate data setup for testing or exploring Salesforce's capabilities, this two-part series will guide you. 

In **Part 1**, we’ll focus on setting up your environment, connecting to Salesforce using OAuth2, and programmatically creating and retrieving data. By the end of this guide, you will have a fully functioning Node.js project to work with Salesforce data, which sets the stage for **Part 2**: writing end-to-end tests for Salesforce applications.

---

## **Big Picture: Weekend Project Plan**
### **Goal**:
1. **Sign Up for a Free Salesforce Developer Account**: Access a personal Salesforce instance for development.
2. **Set Up a Connected App**: Enable secure programmatic access via Salesforce's APIs.
3. **Write Authentication Code**: Use Node.js to connect to Salesforce using OAuth2.
4. **Create and Retrieve Data**: Write code to create leads in Salesforce and retrieve them for validation.

### **Expected Time Commitment**:
- **Setup Salesforce Developer Instance**: 1 hour.
- **Configure Connected App and Understand Authentication**: 1.5 hours.
- **Write Node.js Scripts for Authentication and Lead Creation**: 3.5 hours.

---

## **Step 1: Sign Up for a Free Salesforce Developer Account**
To interact with Salesforce, you need access to an org. Salesforce provides a free **Developer Edition** that is perfect for testing and development.

### **Instructions**:
1. Visit the [Salesforce Developer Signup Page](https://developer.salesforce.com/signup).
2. Fill out the form with your details:
   - First Name, Last Name, Email, and desired Username (e.g., `yourname@devinstance.com`).
3. Check your email for the verification link.
4. Log in to your new Salesforce Developer Edition org.

### **Why Do You Need This?**
The Developer Edition provides full access to Salesforce's platform features, including:
- API access.
- Custom object creation.
- Workflow automation.

---

## **Step 2: Set Up a Connected App in Salesforce**
A **Connected App** allows external applications to securely connect to Salesforce via APIs using OAuth2.

### **Instructions**:
1. **Log in to Salesforce**:
   - Go to your Developer Edition org.

2. **Navigate to App Manager**:
   - Click the gear icon (⚙️) in the top-right corner.
   - Select **Setup**.
   - In the **Quick Find** box, type **App Manager** and click it.

3. **Create a New Connected App**:
   - Click the **New Connected App** button.
   - Fill in the following details:
     - **Connected App Name**: `NodeTestApp`.
     - **API Name**: Auto-populates from the app name.
     - **Contact Email**: Provide your email.

4. **Enable OAuth Settings**:
   - Check **Enable OAuth Settings**.
   - Set the **Callback URL** to `http://localhost:3000` (or your app's domain).
   - Select **OAuth Scopes**:
     - `Full Access (full)`.
     - `API (api)`.

5. **Save and Wait**:
   - Save your Connected App.
   - It may take 2-10 minutes for the app to become active.

6. **Note Down Key Details**:
   - Once the app is active, click **Manage Consumer Details**.
   - Copy the **Consumer Key** and **Consumer Secret**.

---

## **Step 3: Write Authentication Code**

The first step in programmatically interacting with Salesforce is to authenticate using the **OAuth2 Username-Password Flow**.

### **Code Example: `dataconnect.js`**
The following code handles authentication and retrieves an access token from Salesforce:

```javascript
const axios = require('axios');
const qs = require('qs');

/**
 * Authenticate with Salesforce using OAuth2.
 * @returns {Promise<object>} JSON response containing the access token and instance URL.
 */
const authenticateSalesforce = async () => {
    const data = qs.stringify({
        grant_type: 'password',
        client_id: 'your_consumer_key', // Replace with your Consumer Key
        client_secret: 'your_consumer_secret', // Replace with your Consumer Secret
        username: 'your_salesforce_username', // Replace with your Salesforce username
        password: 'your_salesforce_password_and_token' // Replace with your password + security token
    });

    const config = {
        method: 'post',
        url: 'https://login.salesforce.com/services/oauth2/token',
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        data: data
    };

    try {
        const response = await axios.request(config);
        console.log('Authentication successful:', response.data);
        return response.data;
    } catch (error) {
        console.error('Authentication failed:', error.response?.data || error.message);
        throw error;
    }
};

module.exports = authenticateSalesforce;
```

---

## **Step 4: Create a Lead Programmatically**

Now that you can authenticate, let’s write a script to create a lead in Salesforce.

### **Code Example: `createlead.js`**

```javascript
const axios = require('axios');
const authenticateSalesforce = require('./dataconnect');

/**
 * Create a new lead in Salesforce.
 * @param {string} instanceUrl - Salesforce instance URL.
 * @param {string} accessToken - Salesforce access token.
 * @param {object} leadData - Lead data to create.
 */
const createLead = async (instanceUrl, accessToken, leadData) => {
    const endpoint = `${instanceUrl}/services/data/v57.0/sobjects/Lead`;
    const headers = {
        Authorization: `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
    };

    try {
        const response = await axios.post(endpoint, leadData, { headers });
        console.log('Lead created successfully:', response.data);
    } catch (error) {
        console.error('Error creating lead:', error.response?.data || error.message);
    }
};

(async () => {
    try {
        // Authenticate and retrieve credentials
        const authResponse = await authenticateSalesforce();
        const { instance_url, access_token } = authResponse;

        // Define lead data
        const leadData = {
            FirstName: 'John',
            LastName: 'Doe',
            Company: 'Doe Enterprises',
            Email: 'john.doe@example.com',
            Phone: '1234567890',
            Status: 'Open - Not Contacted'
        };

        // Create the lead
        await createLead(instance_url, access_token, leadData);
    } catch (error) {
        console.error('Error:', error.message);
    }
})();
```

---

## **Step 5: Test the Code**

1. **Install Dependencies**:
   ```bash
   npm install axios qs
   ```

2. **Run `createlead.js`**:
   ```bash
   node createlead.js
   ```

---

## **Summary**

### **What You Learned in Part 1**:
- **Setting up Salesforce Developer Edition**: Access your free Salesforce instance.
- **Configuring a Connected App**: Enable secure API access via OAuth2.
- **Authentication with Salesforce**: Use OAuth2 Username-Password Flow to get access tokens.
- **Creating Leads Programmatically**: Interact with Salesforce data using REST APIs.

### **What’s Next in Part 2**:
- Use the leads created in Part 1 to write end-to-end tests.
- Test Salesforce workflows, validations, and triggers programmatically.

With these foundational skills, you're now ready to start building automated test setups for Salesforce applications!
