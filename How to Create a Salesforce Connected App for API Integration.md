### **How to Create a Salesforce Connected App for API Integration**

A Salesforce Connected App allows external applications to integrate with Salesforce using APIs. Here are step-by-step instructions to create a Connected App:

---

### **Step 1: Log in to Salesforce**
1. Log in to your Salesforce account.
   - Use a **Sandbox** environment for testing or **Production** for live usage.

---

### **Step 2: Navigate to the App Manager**
1. In Salesforce, click on the gear icon (⚙️) in the top-right corner.
2. Select **Setup** from the dropdown.
3. In the **Quick Find** search bar, type **App Manager**.
4. Click on **App Manager** under the **Apps** section.

---

### **Step 3: Create a New Connected App**
1. In the App Manager, click the **New Connected App** button (top-right corner).
2. Fill out the **Basic Information**:
   - **Connected App Name**: Provide a name for your app (e.g., `My Integration App`).
   - **API Name**: This will auto-populate based on the app name. You can edit it if necessary.
   - **Contact Email**: Enter a valid email address where Salesforce can contact you.

---

### **Step 4: Configure API Integration Settings**
1. Scroll to the **API (Enable OAuth Settings)** section.
2. Check the box **Enable OAuth Settings**.
3. Configure the following:
   - **Callback URL**:
     - This is the redirect URI where Salesforce sends the OAuth token after authentication.
     - For local testing, use: `http://localhost:3000` (replace with your app's domain in production).
   - **Selected OAuth Scopes**:
     - Add the necessary scopes by selecting them and clicking **Add**:
       - `Full Access (full)`
       - `API (api)`
       - Optionally, add others like `Refresh Token (refresh_token)` if needed.
4. **Select Access Policies**:
   - Check **Require Secret for Web Server Flow**.
   - Optionally, select **IP Relaxation** or other policies based on your security requirements.

---

### **Step 5: Save and Confirm**
1. Scroll to the bottom and click **Save**.
2. Salesforce will create the connected app, but it may take **2-10 minutes** for the app to propagate.

---

### **Step 6: View App Details**
1. After saving, you will see a confirmation screen. Click **Continue** to return to the App Manager.
2. In the App Manager, find your app in the list (use the search bar if necessary).
3. Click the **Dropdown Arrow** next to your app and select **View**.

---

### **Step 7: Note Down Key Details**
On the app details page:
1. **Consumer Key**: Copy this value. It acts as the app’s unique identifier.
2. **Consumer Secret**: Click **Click to Reveal** and copy this value. This is the app’s password.

---

### **Step 8: Additional Settings (Optional)**
1. If you need to make changes later:
   - Click **Manage Connected Apps** in the App Manager.
   - Find your app and click **Edit**.
2. For production use, consider setting IP restrictions or adding a more specific callback URL.

---

### **Step 9: Testing the Connected App**
Test the app by performing an OAuth flow using tools like **Postman**, **cURL**, or your preferred programming language.

**Example OAuth Token Request (Postman)**:
1. Create a POST request to:
   ```
   https://login.salesforce.com/services/oauth2/token
   ```
   - For **Sandbox**, use:
     ```
     https://test.salesforce.com/services/oauth2/token
     ```

2. Add the following body parameters:
   The body must be a x-www-form-urlencoded containing the following parameters:
   - `grant_type`: `password`
   - `client_id`: The **Consumer Key** from Step 7.
   - `client_secret`: The **Consumer Secret** from Step 7.
   - `username`: Your Salesforce username.
   - `password`: Your Salesforce password + security token.

4. Test the request. If successful, you will receive a JSON response containing the `access_token` and other details.

---

### **Key Notes**
1. **Security Token**:
   - If your Salesforce org requires a security token, append it to your password in the OAuth request.
   - Example: If your password is `Password123` and your token is `ABCDEF`, send `Password123ABCDEF`.

2. **Callback URL**:
   - For production, replace `http://localhost:3000` with your actual application domain.
   - Ensure the URL matches exactly (case-sensitive) to avoid authentication errors.

3. **OAuth Scopes**:
   - `Full Access (full)` grants complete access to Salesforce data but should be used cautiously.
   - For limited access, choose `API (api)` or specific object-level scopes.

---

### **Final Output**
After completing these steps, your Salesforce Connected App is ready for use. You can now integrate with Salesforce APIs using the **Consumer Key**, **Consumer Secret**, and **Access Token**.
