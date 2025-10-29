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

**Base URL:** `http://localhost:8080/api`

**Production URL:** `https://api.urproject.com/api`

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
Retry-After: 60
```

---

## Error Handling

### Standard Error Response

```json
{
  "success": false,
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
| 405  | Method Not Allowed                     |
| 422  | Validation Error                       |
| 429  | Too Many Requests (Rate Limited)       |
| 500  | Server Error                           |

---

## Endpoints

### Health Checks

#### 1. Basic Health Check

**GET** `/health`

**Description:** Check if API is responsive

**Headers:**

```
Accept: application/json
```

**Response:**

```json
{
  "status": "healthy",
  "timestamp": "2025-10-15T19:00:00Z"
}
```

**Status Code:** `200 OK`

---

#### 2. Detailed Health Check

**GET** `/health/detailed`

**Description:** Get detailed system health information

**Response:**

```json
{
  "status": "healthy",
  "checks": {
    "database": "ok",
    "redis": "ok",
    "storage": "ok"
  },
  "timestamp": "2025-10-15T19:00:00Z"
}
```

**Status Code:** `200 OK`

---

### Authentication - Registration

#### 3. Register Tradesperson

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
  "email": "john.smith@example.com",
  "phone": "+447123456789",
  "password": "Password123!",
  "password_confirmation": "Password123!",
  "bio": "Experienced plumber with 15 years in the industry",
  "skills": ["Plumbing", "Heating", "Gas Safety"],
  "experience_years": 15,
  "city": "London",
  "postcode": "SW1A 1AA"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Registration successful",
  "user": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone": "+447123456789",
    "user_type": "tradesperson",
    "created_at": "2025-10-15T19:00:00Z"
  }
}
```

**Status Code:** `201 Created`

**Validation Rules:**

- `first_name` - required, string, max 255 chars
- `last_name` - required, string, max 255 chars
- `email` - required, unique, valid email
- `phone` - required, unique, valid UK phone number
- `password` - required, min 8 chars, must contain uppercase, lowercase,
  numbers, and symbols
- `password_confirmation` - required, must match password
- `bio` - optional, string
- `skills` - optional, array of strings
- `experience_years` - optional, integer, 0-70
- `city` - optional, string
- `postcode` - optional, valid UK postcode

**Error Response (422):**

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "email": ["The email has already been taken."],
    "password": ["The password confirmation does not match."]
  }
}
```

---

#### 4. Register Company

**POST** `/register/company`

**Description:** Register a new company account

**Request Body:**

```json
{
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane.doe@company.com",
  "phone": "+447987654321",
  "password": "Password123!",
  "password_confirmation": "Password123!",
  "company_name": "ABC Construction Ltd",
  "company_registration": "12345678",
  "company_address": "123 Business Street, London",
  "company_postcode": "EC1A 1BB",
  "company_website": "https://abc-construction.com"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Registration successful",
  "user": {
    "uuid": "550e8400-e29b-41d4-a716-446655440001",
    "first_name": "Jane",
    "last_name": "Doe",
    "email": "jane.doe@company.com",
    "phone": "+447987654321",
    "user_type": "company",
    "company": {
      "company_name": "ABC Construction Ltd",
      "company_registration": "12345678",
      "company_address": "123 Business Street, London",
      "company_postcode": "EC1A 1BB",
      "company_website": "https://abc-construction.com"
    },
    "created_at": "2025-10-15T19:00:00Z"
  }
}
```

**Status Code:** `201 Created`

---

### Phone Verification (OTP)

#### 5. Send OTP

**POST** `/otp/send`

**Description:** Send OTP verification code to phone number

**Headers:**

```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "phone": "+447123456789"
}
```

**Response:**

```json
{
  "success": true,
  "message": "OTP sent successfully to +447123456789",
  "expires_in_minutes": 10
}
```

**Status Code:** `200 OK`

**Rate Limit:** 3 requests per 10 minutes per phone number

**Error Response (429 - Rate Limited):**

```json
{
  "success": false,
  "message": "Too many OTP requests. Please try again later.",
  "retry_after": 600
}
```

---

#### 6. Verify OTP

**POST** `/otp/verify`

**Description:** Verify OTP code

**Headers:**

```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "phone": "+447123456789",
  "otp": "123456"
}
```

**Response (Success):**

```json
{
  "success": true,
  "message": "Phone number verified successfully"
}
```

**Status Code:** `200 OK`

**Error Response (422 - Invalid OTP):**

```json
{
  "success": false,
  "message": "Invalid OTP code",
  "errors": {
    "otp": ["The provided OTP code is incorrect"]
  }
}
```

**Error Response (422 - Expired OTP):**

```json
{
  "success": false,
  "message": "OTP code has expired. Please request a new one."
}
```

**Max Attempts:** 3 attempts per OTP (after which a new OTP must be requested)

---

### Authentication - Login

#### 7. Login

**POST** `/login`

**Description:** Login with email and password

**Headers:**

```
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "email": "john.smith@example.com",
  "password": "Password123!"
}
```

**Response (Success):**

```json
{
  "success": true,
  "message": "Login successful",
  "user": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "user_type": "tradesperson",
    "phone_verified": true
  },
  "token": "1|abcdefghijklmnopqrstuvwxyz123456789"
}
```

**Status Code:** `200 OK`

**Store the token** and include it in all subsequent authenticated requests:

```
Authorization: Bearer 1|abcdefghijklmnopqrstuvwxyz123456789
```

**Error Response (422 - Invalid Credentials):**

```json
{
  "success": false,
  "message": "The provided credentials are incorrect",
  "errors": {
    "email": ["These credentials do not match our records"]
  }
}
```

**Error Response (422 - Validation Error):**

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "email": ["The email field is required."],
    "password": ["The password field is required."]
  }
}
```

---

### Authenticated User

#### 8. Get Current User

**GET** `/user`

**Description:** Get authenticated user's profile information

**Headers:**

```
Authorization: Bearer {token}
Accept: application/json
```

**Response (Tradesperson):**

```json
{
  "user": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone": "+447123456789",
    "user_type": "tradesperson",
    "phone_verified": true,
    "bio": "Experienced plumber with 15 years in the industry",
    "skills": ["Plumbing", "Heating", "Gas Safety"],
    "experience_years": 15,
    "city": "London",
    "postcode": "SW1A 1AA",
    "created_at": "2025-10-15T19:00:00Z"
  }
}
```

**Response (Company):**

```json
{
  "user": {
    "uuid": "550e8400-e29b-41d4-a716-446655440001",
    "first_name": "Jane",
    "last_name": "Doe",
    "email": "jane.doe@company.com",
    "phone": "+447987654321",
    "user_type": "company",
    "phone_verified": true,
    "company": {
      "company_name": "ABC Construction Ltd",
      "company_registration": "12345678",
      "company_address": "123 Business Street, London",
      "company_postcode": "EC1A 1BB",
      "company_website": "https://abc-construction.com"
    },
    "created_at": "2025-10-15T19:00:00Z"
  }
}
```

**Status Code:** `200 OK`

**Error Response (401 - Not Authenticated):**

```json
{
  "success": false,
  "message": "Unauthenticated"
}
```

---

### Profile Management

#### 9. Update Tradesperson Profile

**PUT** `/profile/tradesperson`

**Description:** Update tradesperson profile information

**Headers:**

```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "bio": "Experienced plumber with 20 years in the trade. Specializing in residential and commercial installations.",
  "skills": ["Plumbing", "Heating", "Gas Safety", "Bathroom Fitting"],
  "experience_years": 20,
  "city": "Manchester",
  "postcode": "M1 1AA",
  "hourly_rate": 45.0,
  "availability": "full-time"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Profile updated successfully",
  "user": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "first_name": "John",
    "last_name": "Smith",
    "bio": "Experienced plumber with 20 years in the trade. Specializing in residential and commercial installations.",
    "skills": ["Plumbing", "Heating", "Gas Safety", "Bathroom Fitting"],
    "experience_years": 20,
    "city": "Manchester",
    "postcode": "M1 1AA",
    "hourly_rate": 45.0,
    "availability": "full-time"
  }
}
```

**Status Code:** `200 OK`

**Note:** Only users with `user_type: tradesperson` can use this endpoint.
Companies will receive `403 Forbidden`.

**Error Response (422 - Validation Error):**

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "experience_years": ["The experience years must be between 0 and 70."],
    "hourly_rate": ["The hourly rate must be a number."]
  }
}
```

---

#### 10. Update Company Profile

**PUT** `/profile/company`

**Description:** Update company profile information

**Headers:**

```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "company_name": "ABC Construction Ltd",
  "company_registration": "12345678",
  "company_address": "456 New Business Park, London",
  "company_postcode": "EC2A 2BB",
  "company_website": "https://abc-construction.co.uk",
  "company_description": "Leading construction company with 30 years of experience"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Company profile updated successfully",
  "user": {
    "uuid": "550e8400-e29b-41d4-a716-446655440001",
    "first_name": "Jane",
    "last_name": "Doe",
    "company": {
      "company_name": "ABC Construction Ltd",
      "company_registration": "12345678",
      "company_address": "456 New Business Park, London",
      "company_postcode": "EC2A 2BB",
      "company_website": "https://abc-construction.co.uk",
      "company_description": "Leading construction company with 30 years of experience"
    }
  }
}
```

**Status Code:** `200 OK`

**Note:** Only users with `user_type: company` can use this endpoint.
Tradespersons will receive `403 Forbidden`.

---

### Password Management

#### 11. Request Password Reset

**POST** `/forgot-password`

**Description:** Send password reset link to email

**Headers:**

```
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "email": "john.smith@example.com"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Password reset link sent to your email"
}
```

**Status Code:** `200 OK`

**Note:** Check `http://localhost:8025` (Mailhog) in development environment
for the reset email.

---

#### 12. Reset Password with Token

**POST** `/reset-password`

**Description:** Reset password using token from email

**Headers:**

```
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "token": "PASTE_TOKEN_FROM_EMAIL_HERE",
  "email": "john.smith@example.com",
  "password": "NewPassword123!",
  "password_confirmation": "NewPassword123!"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Password reset successfully"
}
```

**Status Code:** `200 OK`

**Error Response (422 - Invalid Token):**

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "token": ["This password reset token is invalid."]
  }
}
```

---

#### 13. Change Password (Authenticated)

**PUT** `/password`

**Description:** Change password for authenticated user

**Headers:**

```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

**Request Body:**

```json
{
  "current_password": "Password123!",
  "password": "NewPassword456!",
  "password_confirmation": "NewPassword456!"
}
```

**Response:**

```json
{
  "success": true,
  "message": "Password changed successfully"
}
```

**Status Code:** `200 OK`

**Error Response (422 - Wrong Current Password):**

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "current_password": ["The current password is incorrect."]
  }
}
```

---

### Search & Discovery

#### 14. Search Tradespeople

**GET** `/search/tradespeople`

**Description:** Search for tradespeople (public endpoint)

**Headers:**

```
Accept: application/json
```

**Query Parameters:**

```
?skill=Plumbing
&city=London
&page=1
&per_page=15
```

**Example:**

```
/search/tradespeople?skill=Plumbing&city=London&page=1
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "uuid": "550e8400-e29b-41d4-a716-446655440000",
      "first_name": "John",
      "last_name": "Smith",
      "bio": "Experienced plumber with 15 years in the industry",
      "skills": ["Plumbing", "Heating", "Gas Safety"],
      "experience_years": 15,
      "city": "London",
      "postcode": "SW1A",
      "hourly_rate": 45.0,
      "availability": "full-time"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 15,
    "total": 45,
    "last_page": 3
  }
}
```

**Status Code:** `200 OK`

---

#### 15. Get Tradesperson Public Profile

**GET** `/tradespeople/{uuid}`

**Description:** Get public profile of a specific tradesperson

**Headers:**

```
Accept: application/json
```

**Response:**

```json
{
  "success": true,
  "tradesperson": {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "first_name": "John",
    "last_name": "Smith",
    "bio": "Experienced plumber with 15 years in the industry",
    "skills": ["Plumbing", "Heating", "Gas Safety"],
    "experience_years": 15,
    "city": "London",
    "postcode": "SW1A",
    "hourly_rate": 45.0,
    "availability": "full-time",
    "phone_verified": true,
    "member_since": "2025-01-15T10:00:00Z"
  }
}
```

**Status Code:** `200 OK`

**Error Response (404 - Not Found):**

```json
{
  "success": false,
  "message": "Tradesperson not found"
}
```

---

### Logout & Token Management

#### 16. Logout (Revoke Current Token)

**POST** `/logout`

**Description:** Logout from current device/session

**Headers:**

```
Authorization: Bearer {token}
Accept: application/json
```

**Response:**

```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

**Status Code:** `200 OK`

---

#### 17. Logout from All Devices

**POST** `/logout-all`

**Description:** Revoke all tokens and logout from all devices

**Headers:**

```
Authorization: Bearer {token}
Accept: application/json
```

**Response:**

```json
{
  "success": true,
  "message": "Logged out from all devices successfully"
}
```

**Status Code:** `200 OK`

**Use Case:** When user suspects unauthorized access or wants to force logout
from all sessions.

---

## Error Scenarios

### Rate Limit Exceeded (429)

When making too many requests:

**Response:**

```json
{
  "success": false,
  "message": "Too many requests. Please try again later.",
  "retry_after": 60
}
```

**Headers:**

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
Retry-After: 60
```

---

### Method Not Allowed (405)

When using wrong HTTP method:

**Response:**

```json
{
  "success": false,
  "message": "The DELETE method is not supported for this route."
}
```

---

## Postman Collection

### Import Collection

1. Download files from `postman/` directory:
   - `UrProject-API.postman_collection.json`
   - `UrProject-Local.postman_environment.json`
2. Open Postman
3. Click **Import** → Select both files
4. Select **UrProject - Local** environment from dropdown

### Environment Variables

The collection uses the following variables:

- `base_url` - API base URL (default: `http://localhost:8080/api`)
- `token` - Auth token (auto-populated after login)
- `user_uuid` - Current user's UUID (auto-populated after
  registration/login)
- `company_token` - Company user token (auto-populated after company login)

### Quick Start

1. **Health Checks** → Run "Basic Health Check" to verify API is running
2. **Register** → Use "Register Tradesperson (Success)" or "Register
   Company (Success)"
3. **Login** → Use "Login Tradesperson (Success)" - token auto-saved
4. **Test Protected Routes** → All other requests automatically include
   token

### Test Scripts

Each request includes test scripts that:

- Validate response status codes
- Check response structure
- Auto-save tokens and UUIDs to environment variables
- Verify success/error messages

---

## Testing with cURL

### Health Check

```bash
curl -X GET http://localhost:8080/api/health \
  -H "Accept: application/json"
```

### Register Tradesperson

```bash
curl -X POST http://localhost:8080/api/register/tradesperson \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone": "+447123456789",
    "password": "Password123!",
    "password_confirmation": "Password123!",
    "bio": "Experienced plumber with 15 years in the industry",
    "skills": ["Plumbing", "Heating", "Gas Safety"],
    "experience_years": 15,
    "city": "London",
    "postcode": "SW1A 1AA"
  }'
```

### Send OTP

```bash
curl -X POST http://localhost:8080/api/otp/send \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "phone": "+447123456789"
  }'
```

### Verify OTP

```bash
curl -X POST http://localhost:8080/api/otp/verify \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "phone": "+447123456789",
    "otp": "123456"
  }'
```

### Login

```bash
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "email": "john.smith@example.com",
    "password": "Password123!"
  }'
```

**Save the token from the response!**

### Get Current User

```bash
curl -X GET http://localhost:8080/api/user \
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
    "experience_years": 20,
    "city": "Manchester",
    "postcode": "M1 1AA",
    "hourly_rate": 45.00,
    "availability": "full-time"
  }'
```

### Request Password Reset

```bash
curl -X POST http://localhost:8080/api/password/forgot \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "email": "john.smith@example.com"
  }'
```

### Reset Password

```bash
curl -X POST http://localhost:8080/api/password/reset \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "token": "PASTE_TOKEN_FROM_EMAIL",
    "email": "john.smith@example.com",
    "password": "NewPassword123!",
    "password_confirmation": "NewPassword123!"
  }'
```

### Change Password

```bash
curl -X PUT http://localhost:8080/api/password \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "current_password": "Password123!",
    "password": "NewPassword456!",
    "password_confirmation": "NewPassword456!"
  }'
```

### Search Tradespeople

```bash
curl -X GET "http://localhost:8080/api/search/tradespeople?skill=Plumbing&city=London&page=1" \
  -H "Accept: application/json"
```

### Get Tradesperson Public Profile

```bash
curl -X GET http://localhost:8080/api/tradespeople/550e8400-e29b-41d4-a716-446655440000 \
  -H "Accept: application/json"
```

### Logout

```bash
curl -X POST http://localhost:8080/api/logout \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Logout from All Devices

```bash
curl -X POST http://localhost:8080/api/logout-all \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

---

## Complete Registration & Login Flow

```bash
# 1. Check API Health
curl -X GET http://localhost:8080/api/health \
  -H "Accept: application/json"

# 2. Register Tradesperson
curl -X POST http://localhost:8080/api/register/tradesperson \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john@example.com",
    "phone": "+447123456789",
    "password": "Password123!",
    "password_confirmation": "Password123!",
    "bio": "Experienced plumber",
    "skills": ["Plumbing"],
    "experience_years": 10,
    "city": "London",
    "postcode": "SW1A 1AA"
  }'

# 3. Login (get token)
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "Password123!"
  }'

# 4. Send OTP (using token from step 3)
curl -X POST http://localhost:8080/api/otp/send \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "phone": "+447123456789"
  }'

# 5. Verify OTP (use code from SMS)
curl -X POST http://localhost:8080/api/otp/verify \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "phone": "+447123456789",
    "otp": "123456"
  }'

# 6. Get User Profile
curl -X GET http://localhost:8080/api/user \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

---

## API Endpoint Summary

| Method | Endpoint                 | Auth Required | Description                       |
| ------ | ------------------------ | ------------- | --------------------------------- |
| GET    | `/health`                | No            | Basic health check                |
| GET    | `/health/detailed`       | No            | Detailed health check             |
| POST   | `/register/tradesperson` | No            | Register tradesperson             |
| POST   | `/register/company`      | No            | Register company                  |
| POST   | `/otp/send`              | Yes           | Send OTP code                     |
| POST   | `/otp/verify`            | Yes           | Verify OTP code                   |
| POST   | `/login`                 | No            | Login (get token)                 |
| GET    | `/user`                  | Yes           | Get current user profile          |
| PUT    | `/profile/tradesperson`  | Yes           | Update tradesperson profile       |
| PUT    | `/profile/company`       | Yes           | Update company profile            |
| POST   | `/forgot-password`       | No            | Request password reset            |
| POST   | `/reset-password`        | No            | Reset password with token         |
| PUT    | `/password`              | Yes           | Change password                   |
| GET    | `/search/tradespeople`   | No            | Search tradespeople (public)      |
| GET    | `/tradespeople/{uuid}`   | No            | Get tradesperson profile (public) |
| POST   | `/logout`                | Yes           | Logout (revoke current token)     |
| POST   | `/logout-all`            | Yes           | Logout from all devices           |

---

## Support

- **API Issues:** dev@urproject.com
- **Documentation:** https://docs.urproject.com
- **Status Page:** https://status.urproject.com

---

## Changelog

### v1.0.0 (2025-10-15)

- Initial API release
- Health check endpoints
- Authentication endpoints (Register, Login, OTP)
- Profile management (Tradesperson & Company)
- Password management (Forgot, Reset, Change)
- Search functionality (Tradespeople with public profiles)
- Logout and token management
- Complete Postman collection with test scripts

---

**Last Updated:** October 15, 2025  
**API Version:** v1.0.0
