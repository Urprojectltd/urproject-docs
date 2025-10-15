# UrProject API Documentation

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Rate Limiting](#rate-limiting)
4. [Error Handling](#error-handling)
5. [Endpoints](#endpoints)
6. [Postman Collection](#postman-collection)
7. [Testing with cURL](#testing-with-curl)

---

## Overview

**Base URL:** `https://api.urproject.com`

**Staging URL:** `https://staging-api.urproject.com`

**API Version:** v1

**Authentication:** Laravel Sanctum (Bearer Token)

**Content Type:** `application/json`

---

## Authentication

### Authentication Flow

1. **Register** → Create account (tradesperson or company)
2. **Verify Phone** → Send and verify OTP
3. **Login** → Receive Bearer token
4. **Use Token** → Include in `Authorization` header for protected endpoints

### Headers Required

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer {your_token_here}
```

---

## Rate Limiting

| Endpoint Type  | Rate Limit                             |
| -------------- | -------------------------------------- |
| API Endpoints  | 60 requests/minute                     |
| Login/Register | 5 requests/minute                      |
| OTP Send       | 3 requests/10 minutes per phone number |

**Rate Limit Headers:**

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
```

---

## Error Handling

### Standard Error Response

```json
{
  "message": "Error description",
  "errors": {
    "field_name": ["Validation error message"]
  }
}
```

### HTTP Status Codes

| Code | Meaning                                |
| ---- | -------------------------------------- |
| 200  | Success                                |
| 201  | Created                                |
| 204  | No Content (Success, no response body) |
| 400  | Bad Request                            |
| 401  | Unauthorized                           |
| 403  | Forbidden                              |
| 404  | Not Found                              |
| 422  | Validation Error                       |
| 429  | Too Many Requests (Rate Limited)       |
| 500  | Server Error                           |

---

## Endpoints

### Authentication

#### 1. Register Tradesperson

**POST** `/register/tradesperson`

**Description:** Register a new tradesperson account

**Headers:**

```
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "first_name": "John",
  "last_name": "Smith",
  "email": "john@example.com",
  "phone": "+447123456789",
  "password": "Password123!",
  "password_confirmation": "Password123!",
  "postcode": "SW1A 1AA",
  "skills": ["Plumbing", "Heating"],
  "certifications": ["Gas Safe", "CSCS"],
  "experience_years": 10,
  "bio": "Experienced plumber"
}
```

**Response:** `204 No Content`

**Validation Rules:**

- `email` - unique, valid email
- `phone` - unique, valid UK phone number
- `password` - min 8 chars, mixed case, numbers, symbols
- `postcode` - valid UK postcode
- `experience_years` - 0-70

---

#### 2. Register Company

**POST** `/register/company`

**Description:** Register a new company account

**Request Body:**

```json
{
  "first_name": "Sarah",
  "last_name": "Johnson",
  "email": "sarah@company.com",
  "phone": "+447987654321",
  "password": "Password123!",
  "password_confirmation": "Password123!",
  "company_name": "BuildRight Ltd",
  "company_type": "limited",
  "registered_address": "123 Street",
  "city": "London",
  "postcode": "EC1A 1BB",
  "company_email": "info@company.com",
  "company_phone": "+442071234567",
  "description": "Construction company"
}
```

**Response:** `204 No Content`

**Company Types:**

- `sole_trader`
- `partnership`
- `limited`
- `plc`

---

#### 3. Send OTP

**POST** `/otp/send`

**Description:** Send OTP to phone number

**Request Body:**

```json
{
  "phone": "+447123456789",
  "purpose": "phone_verification"
}
```

**Response:**

```json
{
  "message": "OTP sent successfully",
  "expires_in_minutes": 10
}
```

**OTP Purposes:**

- `registration`
- `login`
- `password_reset`
- `phone_verification`

**Rate Limit:** 3 OTPs per 10 minutes per phone number

---

#### 4. Verify OTP

**POST** `/otp/verify`

**Description:** Verify OTP code

**Request Body:**

```json
{
  "phone": "+447123456789",
  "code": "123456",
  "purpose": "phone_verification"
}
```

**Response:**

```json
{
  "message": "OTP verified successfully"
}
```

**Max Attempts:** 3 attempts per OTP

---

#### 5. Login

**POST** `/login`

**Description:** Login and receive Bearer token

**Request Body:**

```json
{
  "email": "john@example.com",
  "password": "Password123!",
  "device_name": "mobile"
}
```

**Response:**

```json
{
  "message": "Login successful",
  "user": {
    "uuid": "...",
    "first_name": "John",
    "last_name": "Smith",
    "email": "john@example.com",
    "user_type": "tradesperson"
  },
  "token": "1|abcdefghijklmnopqrstuvwxyz..."
}
```

**Store the token** and include in all subsequent requests:

```
Authorization: Bearer 1|abcdefghijklmnopqrstuvwxyz...
```

---

#### 6. Logout

**POST** `/logout`

**Description:** Logout (revoke current token)

**Headers:**

```
Authorization: Bearer {token}
```

**Response:** `204 No Content`

---

### Profile Management

#### 7. Get Profile

**GET** `/api/profile`

**Description:** Get authenticated user's profile

**Headers:**

```
Authorization: Bearer {token}
```

**Response:**

```json
{
  "user": {
    "uuid": "...",
    "first_name": "John",
    "last_name": "Smith",
    "email": "john@example.com",
    "phone": "+447123456789",
    "user_type": "tradesperson",
    "bio": "Experienced plumber",
    "skills": ["Plumbing", "Heating"],
    "experience_years": 10,
    "city": "London",
    "postcode": "SW1A 1AA",
    "company": null
  }
}
```

---

#### 8. Update Tradesperson Profile

**PUT** `/api/profile/tradesperson`

**Description:** Update tradesperson profile

**Headers:**

```
Authorization: Bearer {token}
```

**Request Body:**

```json
{
  "bio": "Updated bio",
  "skills": ["Plumbing", "Heating", "Gas Fitting"],
  "experience_years": 15,
  "service_radius_miles": 30
}
```

**Response:**

```json
{
  "message": "Profile updated successfully",
  "user": { ... }
}
```

**Only tradespersons** can use this endpoint (403 for companies)

---

#### 9. Update Company Profile

**PUT** `/api/profile/company`

**Description:** Update company profile

**Request Body:**

```json
{
  "company_name": "BuildRight Ltd",
  "description": "Updated description",
  "services_offered": ["Construction", "Renovation"],
  "company_size": "51-200"
}
```

**Response:**

```json
{
  "message": "Company profile updated successfully",
  "user": { ... }
}
```

**Only companies** can use this endpoint (403 for tradespersons)

---

#### 10. Upload Profile Picture

**POST** `/api/profile/profile-picture`

**Description:** Upload profile picture

**Headers:**

```
Authorization: Bearer {token}
Content-Type: multipart/form-data
```

**Request Body:**

```
profile_image: (file - JPG/PNG, max 5MB)
```

**Response:**

```json
{
  "message": "Profile picture uploaded successfully",
  "profile_image_url": "https://..."
}
```

---

### Document Management

#### 11. List Documents

**GET** `/api/documents`

**Description:** List all verification documents

**Response:**

```json
{
  "documents": [
    {
      "id": 1,
      "document_type": "id_document",
      "file_name": "drivers_license.pdf",
      "file_size": 1024000,
      "status": "approved",
      "uploaded_at": "2025-10-14T10:00:00Z"
    }
  ]
}
```

**Document Types (Tradesperson):**

- `id_document`
- `proof_of_address`
- `qualification`
- `certification`
- `insurance`
- `license`
- `cscs_card`
- `dbs_check`

**Document Types (Company):**

- `business_registration`
- `vat_certificate`
- `public_liability_insurance`
- `employers_liability_insurance`
- `professional_indemnity`

---

#### 12. Upload Document

**POST** `/api/documents`

**Description:** Upload verification document

**Headers:**

```
Authorization: Bearer {token}
Content-Type: multipart/form-data
```

**Request Body:**

```
document_type: id_document
file: (PDF/JPG/PNG, max 5MB)
notes: (optional)
```

**Response:**

```json
{
  "message": "Document uploaded successfully",
  "document": { ... }
}
```

---

#### 13. Download Document

**GET** `/api/documents/{id}/download`

**Description:** Download document file

**Response:** Binary file download

---

### Search

#### 14. Search Tradespersons

**GET** `/api/search/tradespersons`

**Description:** Search for tradespersons

**Query Parameters:**

```
?skills[]=Plumbing
&skills[]=Heating
&postcode=SW1A 1AA
&latitude=51.5074
&longitude=-0.1278
&radius_miles=25
&min_experience=5
&available_only=true
&sort_by=distance
&per_page=15
```

**Response:**

```json
{
  "data": [
    {
      "uuid": "...",
      "first_name": "John",
      "last_name": "Smith",
      "bio": "Experienced plumber",
      "skills": ["Plumbing", "Heating"],
      "experience_years": 10,
      "city": "London",
      "distance": 5.2
    }
  ],
  "current_page": 1,
  "per_page": 15,
  "total": 45
}
```

**Sort Options:**

- `distance` (default)
- `experience`
- `rating`

---

#### 15. Search Companies

**GET** `/api/search/companies`

**Description:** Search for companies

**Query Parameters:**

```
?services[]=Construction
&postcode=SW1A 1AA
&radius_miles=25
&company_size=51-200
&verified_only=true
&per_page=15
```

**Response:**

```json
{
  "data": [
    {
      "uuid": "...",
      "company": {
        "company_name": "BuildRight Ltd",
        "description": "Construction company",
        "city": "London",
        "services_offered": ["Construction", "Renovation"],
        "verification_status": "verified"
      }
    }
  ],
  "current_page": 1,
  "total": 23
}
```

---

## Postman Collection

### Import Collection

1. Download `postman/UrProject-API.postman_collection.json`
2. Open Postman
3. Click **Import** → Select file
4. Import `UrProject-Local.postman_environment.json`
5. Select **UrProject - Local** environment

### Quick Start

1. **Register** → Use "Register Tradesperson" request
2. **Login** → Token auto-saved to environment variable
3. **All other requests** → Token automatically included

### Environment Variables

```
base_url: http://localhost:8080
access_token: (auto-populated after login)
```

---

## Testing with cURL

### Register Tradesperson

```bash
curl -X POST http://localhost:8080/register/tradesperson \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john@example.com",
    "phone": "+447123456789",
    "password": "Password123!",
    "password_confirmation": "Password123!",
    "postcode": "SW1A 1AA",
    "skills": ["Plumbing", "Heating"],
    "experience_years": 10,
    "bio": "Experienced plumber"
  }'
```

### Send OTP

```bash
curl -X POST http://localhost:8080/otp/send \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "phone": "+447123456789",
    "purpose": "phone_verification"
  }'
```

### Verify OTP

```bash
curl -X POST http://localhost:8080/otp/verify \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "phone": "+447123456789",
    "code": "123456",
    "purpose": "phone_verification"
  }'
```

### Login

```bash
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "Password123!",
    "device_name": "mobile"
  }'
```

**Save the token from the response!**

### Get Profile (Authenticated)

```bash
curl -X GET http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Update Tradesperson Profile

```bash
curl -X PUT http://localhost:8080/api/profile/tradesperson \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "bio": "Updated bio text",
    "skills": ["Plumbing", "Heating", "Gas Fitting"],
    "experience_years": 15
  }'
```

### Upload Profile Picture

```bash
curl -X POST http://localhost:8080/api/profile/profile-picture \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "profile_image=@/path/to/photo.jpg"
```

### Upload Document

```bash
curl -X POST http://localhost:8080/api/documents \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -F "document_type=id_document" \
  -F "file=@/path/to/document.pdf" \
  -F "notes=Driver's license copy"
```

### Search Tradespersons

```bash
curl -X GET "http://localhost:8080/api/search/tradespersons?skills[]=Plumbing&postcode=SW1A%201AA&radius_miles=25&min_experience=5" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Search Companies

```bash
curl -X GET "http://localhost:8080/api/search/companies?services[]=Construction&postcode=SW1A%201AA&radius_miles=25&verified_only=true" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Logout

```bash
curl -X POST http://localhost:8080/logout \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

---

## Common Use Cases

### Complete Registration Flow

```bash
# 1. Register
curl -X POST http://localhost:8080/register/tradesperson \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"first_name":"John","last_name":"Smith","email":"john@example.com","phone":"+447123456789","password":"Password123!","password_confirmation":"Password123!","postcode":"SW1A 1AA","skills":["Plumbing"],"experience_years":10}'

# 2. Send OTP
curl -X POST http://localhost:8080/otp/send \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"phone":"+447123456789","purpose":"phone_verification"}'

# 3. Verify OTP (use code from SMS)
curl -X POST http://localhost:8080/otp/verify \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"phone":"+447123456789","code":"123456","purpose":"phone_verification"}'

# 4. Login
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"email":"john@example.com","password":"Password123!","device_name":"mobile"}'
```

---

## API Versioning

Future versions will use URL versioning:

- `https://api.urproject.com/v1/...` (current)
- `https://api.urproject.com/v2/...` (future)

---

## Support

- **API Issues:** dev@urproject.com
- **Documentation:** https://docs.urproject.com
- **Status Page:** https://status.urproject.com

---

## Changelog

### v1.0.0 (2025-10-14)

- Initial API release
- Authentication endpoints (Register, Login, OTP)
- Profile management (Tradesperson & Company)
- Document management
- Search functionality (Tradespersons & Companies)
- Postman collection and environment files

---

**Last Updated:** October 14, 2025  
**API Version:** v1.0.0
