# **Reworked API Documentation**

## Base URL
`/api`

## **Generate Token**
### Endpoint
`/generate-token`

### Method
`POST`

### Request Body
```json
{
    "api_key": "api-key",
    "email": "abc@gmail.com"
}
```
- `api_key` will be provided by Reworked.
- `email` is the ID from which you created a Reworked account.

### Response
Status Code: `200`
```json
{
    "msg": "Token generated successfully",
    "data": {
        "token": "Auth Token",
        "expires_at": "Timestamp"
    }
}
```
- The token should be sent as an authorization header in the `process-leads` API.
- The token will be valid for 24 hours.

---

## Process Leads
### Endpoint
`/process-leads`

### Method
`POST`

### Headers
```json
{
    "Authorization": "token from generate-token API response"
}
```

### Request Body
```json
{
    "file_url": "file upload file url",
    "email": "email",
    "callback_url": "callback_url"
}
```
- `file_url` - The URL of the file to process.
- `callback_url` - URL to be called once file processing is completed.

### Required Columns in File
```json
[
    "owner_full_name",
    "mail_street_address",
    "mail_city",
    "mail_state",
    "mail_zip_code",
    "lot_area_sqft",    
    "apn",             
    "property_zip_code"
]
```
- `lot_area_sqft` and `apn` are applicable for **LAND** type.
### Response
Status Code: `200`
```json
{
    "msg": "File processing started",
    "data": {
        "token": "auth token",
        "file_upload_identifier": "file upload identifier"
    }
}
```
- `token` is to be used in the `file-status` API.
- The token will be valid for 24 hours.

### Error Cases
Status Code: `500`
```json
{
    "msg": "Error 103: Requisite columns required to generate a Betty Score is not present.",
    "data": {
        "error_detail": "Error 103: Requisite columns required to generate a Betty Score is not present.",
        "status": "ERROR"
    }
}
```
Status Code: `500`
```json
{
    "msg": "Error 101: Please sign up for a plan at reworked.ai before proceeding.",
    "data": {
        "error_detail": "Error 101: Please sign up for a plan at reworked.ai before proceeding.",
        "status": "ERROR"
    }
}
```
Status Code: `500`
```json
{
    "msg": "Error 102: Payment method failed, please update payment method at reworked.ai before proceeding.",
    "data": {
        "error_detail": "Error 102: Payment method failed, please update payment method at reworked.ai before proceeding.",
        "status": "ERROR"
    }
}
```

---

## File Status
### Endpoint
`/file-status`

### Method
`POST`

### Headers
```json
{
    "Authorization": "token from process-leads API response"
}
```

### Request Body
```json
{
    "file_upload_identifier": "file upload identifier",
    "email": "email"
}
```
- `file_upload_identifier` is obtained from the process-leads response.

### Response
Status Code: `200`
```json
{
    "msg": "File Processed Successfully",
    "data": {
        "processed_file_url": "Processed file URL",
        "status": "PROCESSING"
    }
}
```

### Error Cases
Status Code: `500`
```json
{
    "msg": "Error 104: File url is not accessible, please try again with correct url.",
    "data": {
        "error_detail": "Error 104: File url is not accessible, please try again with correct url.",
        "status": "ERROR"
    }
}
```
Status Code: `200`
```json
{
    "msg": "Error 105: File is under review",
    "data": {
        "error_detail": "Error 105: File is under review",
        "status": "UnderReview"
    }
}
```

---

## Authenticate User
### Endpoint
`/user-authenticate`

### Method
`POST`

### Request Body
```json
{
    "email": "email",
    "password": "password"
}
```

### Success Response
```json
{
    "msg": "User Authenticated Successfully",
    "data": {
        "api_secret_key": "api_secret_key"
    }
}
```

### Error Responses
#### Invalid Email ID
Status Code: `400`
```json
{
    "msg": "Invalid Email ID",
    "data": {}
}
```

#### Wrong Password
Status Code: `403`
```json
{
    "msg": "Password is not correct",
    "data": {
        "error_detail": {
            "message": "Request failed with status code 403"
        }
    }
}
```

---

## Get User Details
### Curl Request
```sh
curl --location --request POST 'https://api.example.com/user-details' \
--header 'Authorization: Bearer <TOKEN>' \
--data-raw ''
```

---

## API Flowchart
```mermaid
flowchart TD
    A[Start] --> B[Generate Token \n /generate-token API]
    B --> C{Token Generation Successful?}
    C -->|Yes| D[Process Leads \n /process-leads API]
    C -->|No| Z[End]
    D --> E{File Processing Started}
    E -->|Yes| F[Check File Status \n /file-status API]
    E -->|No| Z
    F --> G{File Processed Successfully?}
    G -->|Yes| H[Return File URL]
    G -->|No| I[Handle Error]
    H --> Z
    I --> Z[End]
```

