# **Mater Automating Salesforce Data Manipulation with Node.js: A Weekend Project**

Imagine having the power to programmatically set up, manipulate, and retrieve Salesforce data with just a few lines of code. Whether you're a QA engineer looking to create realistic test data, a developer automating data setup for demos, or a tech-savvy business analyst exploring Salesforce's vast API capabilities, this guide is for you.

Salesforce is a cornerstone of customer relationship management, but working with its data manually can be time-consuming and prone to errors. Enter **Node.js and Salesforce REST APIs**—a powerful combination that empowers you to interact with Salesforce programmatically. From creating test leads to fetching data for validations, Node.js scripts streamline workflows, reduce manual effort, and enable efficient testing and automation.

In this two-part series, you'll learn how to build a robust, reusable foundation for interacting with Salesforce data. **In Part 1**, we’ll cover:
1. Setting up a Salesforce Developer Edition org for free.
2. Creating a secure connection to Salesforce via a Connected App.
3. Writing Node.js scripts to authenticate, create, and retrieve Salesforce data.

This guide goes beyond theory, giving you a practical, hands-on project that you can complete in just a few hours. By the end of this article, you'll have a fully functional setup to programmatically create leads in Salesforce and retrieve their details, ready to be leveraged in Part 2 for writing comprehensive end-to-end tests.

---

## **Why Should You Care?**

1. **Streamline Testing**: Developers and QA engineers often need to test Salesforce workflows or integrations. This guide provides a way to programmatically create and retrieve data, reducing dependency on manual data entry.
   
2. **Automate and Save Time**: Instead of manually entering records for demos or testing, automate the process with reusable scripts. This is especially beneficial for teams working with multiple environments like staging and production.

3. **Learn a Real-World Use Case**: Understand how to work with APIs, manage authentication securely, and build a foundation for advanced Salesforce automation.

4. **For Developers, QA Engineers, and Salesforce Admins**: Whether you're coding new integrations, validating business rules, or automating repetitive tasks, these skills are a game-changer.

## **Why This Project?**

Imagine you’re a QA engineer or developer tasked with testing workflows in Salesforce. You need to create realistic test data, validate it across multiple environments (e.g., Sandbox, Staging, and Production), and ensure consistency. Doing this manually is error-prone and tedious, especially if the workflow involves complex data relationships or large volumes of records.

Here’s how this project solves that:
1. **Automated Data Creation**: Programmatically create records in Salesforce with just a script.
2. **Data Retrieval and Validation**: Fetch details of created records to ensure correctness.
3. **Reusable Configuration**: Easily adapt the setup for different environments.
4. **Audit and Debugging**: Save all created data to timestamped files for traceability.

---

### **What You’ll Achieve**
- Gain a strong understanding of Salesforce’s REST API and OAuth2 authentication.
- Build reusable scripts to programmatically create and retrieve Salesforce data.
- Lay the groundwork for writing automated end-to-end tests in Part 2.

This isn’t YAT, yet another tutorial; it’s your step-by-step playbook to mastering Salesforce automation with Node.js. Let’s dive in!

---

## **What You’ll be doing**

1. How to set up a **free Salesforce Developer Edition** for testing.
2. How to create a **Connected App** to enable secure programmatic access.
3. How to authenticate with Salesforce using the **OAuth2 Username-Password flow**.
4. How to programmatically **create and retrieve Salesforce Leads**.
5. How to use **Node.js libraries** like `axios`, `qs`, `fs`, and `path` to manage data creation, retrieval, and storage.

---

## **Step 1: Set Up a Free Salesforce Developer Edition**

To interact with Salesforce programmatically, you need access to an instance. A **Developer Edition** is a free, fully functional Salesforce environment designed for developers to test, explore, and build applications.

### **Instructions**:
1. **Sign Up**:
   - Visit the [Salesforce Developer Signup Page](https://developer.salesforce.com/signup).
   - Fill out the form with your details (First Name, Last Name, Email, etc.).
   - Check your email for a verification link, then log in to your new Salesforce Developer Edition.

2. **Explore Your Environment**:
   - Once logged in, you’ll have access to the Salesforce interface, including Leads, Accounts, and Contacts.
   - This environment comes pre-configured with a small set of sample data, which you can expand or modify.

---

## **Step 2: Create a Connected App**

A **Connected App** enables external applications to securely connect to Salesforce using OAuth2.

### **Instructions**:
1. **Navigate to App Manager**:
   - Log in to your Developer Edition.
   - Click the gear icon (⚙️) in the top-right corner and select **Setup**.
   - Search for **App Manager** in the Quick Find box and click it.

2. **Create a New Connected App**:
   - Click **New Connected App**.
   - Fill out the following:
     - **Connected App Name**: `DataConnectApp`.
     - **API Name**: Auto-populates.
     - **Contact Email**: Provide your email. This is where you will receive your security token

3. **Enable OAuth Settings**:
   - Check **Enable OAuth Settings**.
   - Set the **Callback URL** to `http://localhost:3000` (or your app’s domain; recommended)
   - --If the url, after you log into salesforce.com, is https://my-sompany-name.develop.lightning.force.com/whatever
   - -- then you callback url is one with subdomain: https://my-sompany-name
   - Add these **OAuth Scopes**:
     - `Full Access (full)`.
     - `API (api)`.

4. **Save and Wait**:
   - Save your app. It may take a few minutes to activate.
   - Note down the **Consumer Key** and **Consumer Secret** for use in the next steps.
   - -- Under the option: API (Enable OAuth Settings) >> Consumer Key and Secret, click the button [Manage Consumer Details]
	
---

## **Step 3: Write Authentication Code**

The Salesforce OAuth2 **Username-Password Flow** is ideal for programmatic interactions where user intervention is not required. We’ll write a script to authenticate with Salesforce and retrieve an access token.

### **Configuration File: `config.json`**
First, create a configuration file to store your credentials:
```json
{
  "grant_type": "password",
  "client_id": "your_consumer_key",
  "client_secret": "your_consumer_secret",
  "username": "your_salesforce_username",
  "password": "your_salesforce_password + security_token"
}
```
### Getting the Security Token. Under Setup find "Reset My Security Token" and click the button [Reset Security Token]. New toke will be emailed to you in the email under Personal Information section

### **Authentication Script: `dataconnect.js`**
```javascript
const axios = require('axios');
const qs = require('qs');
const fs = require('fs');
const path = require('path');

const authenticateSalesforce = async () => {
    const configPath = path.resolve(__dirname, 'config.json');
    if (!fs.existsSync(configPath)) {
        console.error('Configuration file not found. Please create a "config.json" file.');
        process.exit(1);
    }
    const config = JSON.parse(fs.readFileSync(configPath, 'utf8'));
    const data = qs.stringify(config);

    try {
        const response = await axios.post(
            'https://login.salesforce.com/services/oauth2/token',
            data,
            { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } }
        );
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

## **Step 4: Programmatically Create and Retrieve Leads**

### **Define Lead Model: `leadModel.json`**
```json
{
  "FirstName": "John",
  "LastName": "Doe",
  "Company": "Doe Enterprises",
  "Email": "john.doe@example.com",
  "Phone": "1234567890",
  "Status": "Open - Not Contacted",
  "Street": "123 Main Street",
  "City": "San Francisco",
  "State": "CA",
  "PostalCode": "94105",
  "Country": "USA"
}
```

### **Script: `createlead.js`**
```javascript
const axios = require('axios');
const fs = require('fs');
const path = require('path');
const authenticateSalesforce = require('./dataconnect');

const createLead = async (instanceUrl, accessToken, leadData) => {
    const endpoint = `${instanceUrl}/services/data/v57.0/sobjects/Lead`;
    const headers = { Authorization: `Bearer ${accessToken}`, 'Content-Type': 'application/json' };
    try {
        const response = await axios.post(endpoint, leadData, { headers });
        console.log('Lead created successfully:', response.data);
        return response.data.id;
    } catch (error) {
        console.error('Error creating lead:', error.response?.data || error.message);
        throw error;
    }
};

const getLeadDetails = async (instanceUrl, accessToken, leadId) => {
    const endpoint = `${instanceUrl}/services/data/v57.0/sobjects/Lead/${leadId}`;
    const headers = { Authorization: `Bearer ${accessToken}` };
    try {
        const response = await axios.get(endpoint, { headers });
        console.log('Lead details retrieved:', JSON.stringify(response.data, null, 2));
        return response.data;
    } catch (error) {
        console.error('Error retrieving lead details:', error.response?.data || error.message);
        throw error;
    }
};

(async () => {
    try {
        const authResponse = await authenticateSalesforce();
        const { instance_url, access_token } = authResponse;

        const leadData = JSON.parse(fs.readFileSync('leadModel.json', 'utf8'));
        const leadId = await createLead(instance_url, access_token, leadData);

        const leadDetails = await getLeadDetails(instance_url, access_token, leadId);

        const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
        fs.writeFileSync(`lead-${timestamp}.json`, JSON.stringify(leadDetails, null, 2), 'utf8');
        console.log(`Lead details saved to lead-${timestamp}.json`);
    } catch (error) {
        console.error('Error:', error.message);
    }
})();
```

---

## **Final Output**
1. **Console**:
   - Authentication success.
   - Lead creation confirmation with the ID.
   - Pretty-printed lead details.

2. **File Output**:
   - A JSON file named `lead-YYYY-MM-DDTHH-MM-SS.json` containing the lead’s details.

---

## **Conclusion**
This weekend project equips you with the skills to automate Salesforce data manipulation using Node.js. You’ve learned how to authenticate, create leads programmatically, retrieve their details, and save them for auditing. These foundational skills can be expanded into more advanced workflows, including end-to-end testing, which we’ll explore in **Part 2**.
