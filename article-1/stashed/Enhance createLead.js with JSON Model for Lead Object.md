### **Updated Solution: Enhanced `createLead.js` with JSON Model for Lead Object**

This implementation will:
1. Use a JSON model file (`leadModel.json`) to define the lead structure, including an address.
2. Populate the lead creation data in `createlead.js` from this JSON file.
3. Retrieve the details of the newly created lead.
4. Pretty-print the lead's details on the console.
5. Save the lead's details to a date-time-stamped file.

---

### **Step 1: Create `leadModel.json`**
Create a JSON file named `leadModel.json` to define the structure of the lead, including the address fields.

#### **`leadModel.json`**
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

---

### **Step 2: Updated `createlead.js`**

#### **Code:**
```javascript
const axios = require('axios');
const fs = require('fs');
const path = require('path');
const authenticateSalesforce = require('./dataconnect'); // Import authentication

/**
 * Create a new lead in Salesforce.
 * @param {string} instanceUrl - Salesforce instance URL.
 * @param {string} accessToken - Salesforce access token.
 * @param {object} leadData - Lead data to create.
 * @returns {Promise<object>} - Response from Salesforce containing the lead ID.
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
        return response.data;
    } catch (error) {
        console.error('Error creating lead:', error.response?.data || error.message);
        throw error;
    }
};

/**
 * Get details of a lead by ID.
 * @param {string} instanceUrl - Salesforce instance URL.
 * @param {string} accessToken - Salesforce access token.
 * @param {string} leadId - ID of the lead to retrieve.
 * @returns {Promise<object>} - Response containing lead details.
 */
const getLeadDetails = async (instanceUrl, accessToken, leadId) => {
    const endpoint = `${instanceUrl}/services/data/v57.0/sobjects/Lead/${leadId}`;
    const headers = {
        Authorization: `Bearer ${accessToken}`
    };

    try {
        const response = await axios.get(endpoint, { headers });
        console.log('Lead details retrieved successfully:');
        console.log(JSON.stringify(response.data, null, 2)); // Pretty-print the response
        return response.data;
    } catch (error) {
        console.error('Error retrieving lead details:', error.response?.data || error.message);
        throw error;
    }
};

/**
 * Save lead details to a date-time stamped file.
 * @param {object} leadDetails - The lead details to save.
 */
const saveLeadToFile = (leadDetails) => {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const filename = `lead-${timestamp}.json`;
    const filepath = path.resolve(__dirname, filename);

    try {
        fs.writeFileSync(filepath, JSON.stringify(leadDetails, null, 2), 'utf8');
        console.log(`Lead details saved to file: ${filename}`);
    } catch (error) {
        console.error('Error saving lead details to file:', error.message);
    }
};

(async () => {
    try {
        // Step 1: Authenticate with Salesforce
        const authResponse = await authenticateSalesforce();
        const { instance_url, access_token } = authResponse;

        // Step 2: Read lead data from leadModel.json
        const leadModelPath = path.resolve(__dirname, 'leadModel.json');
        if (!fs.existsSync(leadModelPath)) {
            console.error('Error: leadModel.json not found. Please create the file with lead data.');
            process.exit(1);
        }

        const leadData = JSON.parse(fs.readFileSync(leadModelPath, 'utf8'));

        // Step 3: Create the lead
        const createResponse = await createLead(instance_url, access_token, leadData);
        const leadId = createResponse.id;

        // Step 4: Retrieve the created lead's details
        const leadDetails = await getLeadDetails(instance_url, access_token, leadId);

        // Step 5: Save lead details to a date-time stamped file
        saveLeadToFile(leadDetails);
    } catch (error) {
        console.error('Error:', error.message);
    }
})();
```

---

### **Explanation of the Code**

1. **Read Lead Data from `leadModel.json`**:
   - The lead creation data is dynamically loaded from `leadModel.json`.
   - If the file is missing, an error message prompts the user to create it.

2. **Create the Lead**:
   - Sends the lead data to the Salesforce REST API.
   - Logs the lead creation response, including the generated Lead ID.

3. **Retrieve Lead Details**:
   - Fetches the full details of the newly created lead by its ID.
   - Pretty-prints the response in the console for easy debugging.

4. **Save Lead Details**:
   - Saves the retrieved lead details to a date-time-stamped JSON file.

---

### **Step 3: Run the Script**

1. **Install Dependencies**:
   ```bash
   npm install axios
   ```

2. **Ensure Files Exist**:
   - Place `leadModel.json` in the same directory as `createlead.js` and `dataconnect.js`.

3. **Run the Script**:
   ```bash
   node createlead.js
   ```

---

### **Expected Output**

#### Console:
1. **Lead Created**:
   ```plaintext
   Lead created successfully: { id: '00Qxx0000001gP3EAI', success: true, errors: [] }
   ```

2. **Lead Details** (Pretty-Printed):
   ```json
   {
       "Id": "00Qxx0000001gP3EAI",
       "FirstName": "John",
       "LastName": "Doe",
       "Company": "Doe Enterprises",
       "Email": "john.doe@example.com",
       "Phone": "1234567890",
       "Street": "123 Main Street",
       "City": "San Francisco",
       "State": "CA",
       "PostalCode": "94105",
       "Country": "USA",
       ...
   }
   ```

3. **File Saved**:
   ```plaintext
   Lead details saved to file: lead-2023-12-03T08-45-00.json
   ```

---

### **Summary**

This implementation ensures:
1. **Dynamic Input**: Lead creation data is sourced from a JSON model, making it easy to update.
2. **Comprehensive Workflow**: Includes lead creation, retrieval, and saving details for audit or testing purposes.
3. **User-Friendly**: Provides clear error messages and guidance if files are missing or misconfigured.
