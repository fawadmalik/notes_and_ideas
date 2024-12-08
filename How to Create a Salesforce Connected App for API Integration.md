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

---

### **Troubleshooting**
Most people report the "unsupported_grant_type" error after running this sestup
{"error": "unsupported_grant_type", "error_description": "grant type not supported"}
The error **"unsupported_grant_type"** means that Salesforce does not recognize the `grant_type` specified in your POST request. This usually happens when:

1. The `grant_type` parameter is incorrect.
2. The `Content-Type` header or request payload format is incorrect.

Here’s how to troubleshoot and resolve the issue:

---

### **Step-by-Step Solution**

#### **1. Verify the Correct `grant_type`**
The value for the `grant_type` must match the OAuth 2.0 flow you are using. Common values include:
- `password`: For the **Username-Password Flow**.
- `authorization_code`: For the **Authorization Code Flow**.
- `refresh_token`: For obtaining new tokens using a refresh token.

For the **Username-Password Flow**, the `grant_type` must be:
```plaintext
grant_type=password
```

---

#### **2. Ensure Proper Request Formatting**

##### **Headers:**
Include these headers in your POST request:
- `Content-Type`: `application/x-www-form-urlencoded`

##### **Body:**
The body must be a **URL-encoded string** containing the following parameters:

| Parameter         | Description                                                                                  |
|--------------------|----------------------------------------------------------------------------------------------|
| `grant_type`       | Set to `password`.                                                                          |
| `client_id`        | The **Consumer Key** from your Salesforce Connected App.                                    |
| `client_secret`    | The **Consumer Secret** from your Salesforce Connected App.                                 |
| `username`         | Your Salesforce username.                                                                   |
| `password`         | Your Salesforce password concatenated with your **security token**.                         |

**Example Body:**
```plaintext
grant_type=password
client_id=YOUR_CLIENT_ID
client_secret=YOUR_CLIENT_SECRET
username=YOUR_USERNAME
password=YOUR_PASSWORD_SECURITY_TOKEN
```

---

#### **3. Correctly Configure Postman**

1. **Set the Endpoint URL**
   - Use the appropriate Salesforce OAuth token URL:
     - For production: `https://login.salesforce.com/services/oauth2/token`
     - For sandbox: `https://test.salesforce.com/services/oauth2/token`

2. **Set Request Method**
   - Use the **POST** method.

3. **Add Headers**
   - Key: `Content-Type`
   - Value: `application/x-www-form-urlencoded`

4. **Set Request Body**
   - In Postman, go to the **Body** tab.
   - Select the **x-www-form-urlencoded** option.
   - Enter the following key-value pairs:
     - `grant_type`: `password`
     - `client_id`: Your connected app's Consumer Key.
     - `client_secret`: Your connected app's Consumer Secret.
     - `username`: Your Salesforce username.
     - `password`: Your Salesforce password concatenated with the security token.

---

#### **4. Example Postman Setup**

| Field          | Value                                                                                       |
|-----------------|---------------------------------------------------------------------------------------------|
| **URL**        | `https://login.salesforce.com/services/oauth2/token`                                        |
| **Method**     | POST                                                                                        |
| **Headers**    | `Content-Type: application/x-www-form-urlencoded`                                           |
| **Body Type**  | x-www-form-urlencoded                                                                       |
| **Body Fields**| - `grant_type`: `password` <br> - `client_id`: `<your_client_id>` <br> - `client_secret`: `<your_client_secret>` <br> - `username`: `<your_username>` <br> - `password`: `<your_password_with_security_token>` |

---

#### **5. Troubleshoot Common Issues**

- **Incorrect `grant_type`:**
  - Ensure you are using the correct value based on your OAuth flow (`password` for this scenario).

- **Missing or Incorrect Security Token:**
  - Salesforce appends the **security token** to your password unless your IP is whitelisted.

- **Incorrect Connected App Configuration:**
  - Ensure the "Username-Password Flow" is enabled in the connected app's OAuth settings.

---

#### **6. Test the Request**

Once everything is correctly configured, the response should contain an access token:

**Example Successful Response:**
```json
{
  "access_token": "00Dxx0000001gP3!AQEAQC...",
  "instance_url": "https://yourInstance.salesforce.com",
  "id": "https://login.salesforce.com/id/00Dxx0000001gP3EAI/005xx000001SvXQAA0",
  "token_type": "Bearer",
  "issued_at": "1609461865000",
  "signature": "signature_value"
}
```

---

#### **7. Update Grant Type if Using Authorization Code Flow**

If you intended to use the **Authorization Code Flow** instead:
1. Set `grant_type` to `authorization_code`.
2. Include an `authorization_code` parameter in your request body.
3. Obtain the authorization code by redirecting the user to the Salesforce authorization URL.

---

This guide should resolve the **unsupported_grant_type** error and ensure your Postman request is configured correctly for the Username-Password OAuth Flow.
