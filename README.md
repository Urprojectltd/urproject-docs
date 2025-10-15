# UrProject API - Postman Collection

Complete Postman collection for testing the UrProject Construction Recruitment API.

## 📦 Files Included

- `UrProject-API.postman_collection.json` - Complete API collection with all endpoints
- `UrProject-Local.postman_environment.json` - Local development environment (localhost:8080)
- `UrProject-Staging.postman_environment.json` - Staging environment
- `UrProject-Production.postman_environment.json` - Production environment

---

## 🚀 Quick Start

### 1. Import Collection

1. Open Postman
2. Click **Import** button (top left)
3. Drag and drop `UrProject-API.postman_collection.json`
4. Collection will appear in left sidebar

### 2. Import Environment

1. Click **Import** again
2. Drag and drop the environment file(s) you need:
   - `UrProject-Local.postman_environment.json` (for local development)
   - `UrProject-Staging.postman_environment.json` (for staging)
   - `UrProject-Production.postman_environment.json` (for production)

### 3. Select Environment

1. Click environment dropdown (top right)
2. Select **UrProject - Local** (or Staging/Production)

---

## 🔐 Authentication Flow

### Step 1: Register a User

Choose one:

- **Authentication → Register Tradesperson**
- **Authentication → Register Company**

Click **Send**. You should get `204 No Content` response.

### Step 2: Send OTP

1. Go to **Authentication → Send OTP**
2. Update the `phone` field to match your registration
3. Click **Send**
4. Check your phone for OTP code (or check logs in local dev)

### Step 3: Verify OTP

1. Go to **Authentication → Verify OTP**
2. Update the `phone` and `code` fields
3. Click **Send**

### Step 4: Login

1. Go to **Authentication → Login**
2. Update `email` and `password` to match your registration
3. Click **Send**
4. **Token is automatically saved!** ✅

The test script automatically saves the Bearer token to `{{access_token}}` environment variable.

### Step 5: Use Protected Endpoints

All endpoints under:

- 👤 Profile Management
- 📄 Document Management
- 🔍 Search

...now use the saved token automatically! Just click **Send** on any request.

---

## 📝 Collection Structure

### 🔐 Authentication (6 requests)

- Register Tradesperson
- Register Company
- Send OTP
- Verify OTP
- Login (auto-saves token)
- Logout

### 👤 Profile Management (5 requests)

- Get Profile
- Update Tradesperson Profile
- Update Company Profile
- Upload Profile Picture
- Delete Profile Picture

### 📄 Document Management (4 requests)

- List Documents
- Upload Document
- Download Document
- Delete Document

### 🔍 Search (2 requests)

- Search Tradespersons (by skills, location, experience)
- Search Companies (by services, location, size)

**Total:** 17 API requests

---

## 🧪 Test Scripts

Every request includes test scripts that:

- ✅ Verify HTTP status codes
- ✅ Check response structure
- ✅ Validate data types
- ✅ Auto-save authentication tokens

View test results in the **Test Results** tab after sending a request.

---

## 🌍 Environment Variables

### Variables Used

| Variable       | Description  | Auto-populated?          |
| -------------- | ------------ | ------------------------ |
| `base_url`     | API base URL | No - set per environment |
| `access_token` | Bearer token | Yes - saved on login     |

### Switching Environments

**Local Development:**

```
base_url = http://localhost:8080
```

**Staging:**

```
base_url = https://staging-api.urproject.com
```

**Production:**

```
base_url = https://api.urproject.com
```

Just select the environment from the dropdown!

---

## 📋 Complete Testing Workflow

### Test Flow for Tradesperson

```
1. Register Tradesperson
2. Send OTP → Verify OTP
3. Login (token saved)
4. Get Profile
5. Update Tradesperson Profile
6. Upload Profile Picture
7. Upload Document (ID, Certifications)
8. Search Companies (find projects)
```

### Test Flow for Company

```
1. Register Company
2. Send OTP → Verify OTP
3. Login (token saved)
4. Get Profile
5. Update Company Profile
6. Upload Profile Picture (company logo)
7. Upload Document (Business registration, Insurance)
8. Search Tradespersons (find workers)
```

---

## 🔍 Search Examples

### Find Plumbers in London

**Endpoint:** `GET /api/search/tradespersons`

**Query Parameters:**

```
skills[] = Plumbing
postcode = SW1A 1AA
radius_miles = 25
min_experience = 5
sort_by = distance
```

### Find Construction Companies

**Endpoint:** `GET /api/search/companies`

**Query Parameters:**

```
services[] = Construction
services[] = Renovation
postcode = EC1A 1BB
radius_miles = 30
company_size = 51-200
verified_only = true
```

---

## 📤 File Uploads

### Profile Pictures

**Endpoint:** `POST /api/profile/profile-picture`

1. Select request
2. Go to **Body** tab
3. Select `form-data`
4. Click **Select Files** next to `profile_image`
5. Choose JPG/PNG (max 5MB)
6. Click **Send**

### Documents

**Endpoint:** `POST /api/documents`

1. Select request
2. Update `document_type` (dropdown has options)
3. Click **Select Files** next to `file`
4. Choose PDF/JPG/PNG (max 5MB)
5. Optionally add `notes`
6. Click **Send**

---

## 🐛 Troubleshooting

### Token Not Saving

**Problem:** After login, token not saved to environment.

**Solution:**

1. Open **Authentication → Login** request
2. Go to **Tests** tab
3. Verify this line exists:
   ```javascript
   pm.environment.set('access_token', jsonData.token)
   ```
4. Make sure you have an environment selected (top right)

### 401 Unauthorized Errors

**Problem:** Getting 401 errors on protected endpoints.

**Solution:**

1. Check environment dropdown - make sure environment is selected
2. Run **Authentication → Login** again
3. Check `{{access_token}}` has a value:
   - Click environment dropdown
   - Click eye icon (👁️)
   - Look for `access_token` value

### Connection Refused

**Problem:** `Error: connect ECONNREFUSED 127.0.0.1:8080`

**Solution:**

- Make sure Docker containers are running:
  ```bash
  docker-compose ps
  ```
- Make sure you're using the right environment (Local/Staging/Production)
- Check `base_url` is correct in environment

### File Upload Not Working

**Problem:** Document or image upload fails.

**Solution:**

1. Make sure you're using `form-data` in Body tab (not `raw`)
2. File must be under 5MB
3. For documents: Use PDF, JPG, or PNG only
4. For profile pictures: Use JPG or PNG only

---

## 💡 Tips & Tricks

### Save Example Responses

After successful requests:

1. Click **Save Response**
2. Click **Save as example**
3. Now you have example responses in the collection!

### Organize with Folders

- Create folders for different user journeys
- Duplicate requests for different test scenarios
- Use descriptive names like "Login - Tradesperson" vs "Login - Company"

### Use Pre-request Scripts

Add dynamic data:

```javascript
// Generate random email
pm.environment.set('random_email', `user${Date.now()}@example.com`)
```

Then use `{{random_email}}` in request body!

### Bulk Testing

Use **Collection Runner** to run all requests in sequence:

1. Click on collection name
2. Click **Run** button
3. Select requests to run
4. Click **Run UrProject API**

---

## 📞 Support

- **API Documentation:** See `/docs/API.md`
- **Authentication Flow:** See `/docs/AUTH.md`
- **Issues:** Contact dev team

---

## 🔄 Keeping Collection Updated

When API changes:

1. Re-import the updated JSON file
2. Postman will ask: **Replace** or **Merge**
3. Choose **Replace** to get latest version

---

## ✅ Checklist Before Production Testing

- [ ] Switch to Production environment
- [ ] Use real email addresses
- [ ] Use real UK phone numbers for OTP
- [ ] Upload real documents (ID, certifications)
- [ ] Test with real postcodes
- [ ] Verify rate limiting works (try sending OTP 4 times)
- [ ] Test logout invalidates token

---

**Last Updated:** October 14, 2025  
**Collection Version:** 1.0.0  
**Postman Version:** 10.0+ recommended
