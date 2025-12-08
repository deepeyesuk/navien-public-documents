---
title: "Create Job API"
description: "API documentation for the Create Job endpoint in Service Agent application"
icon: "🔧"
layout: ../layouts/MarkdownLayout.astro
---

## Service Agent - Create Job

BaseUrl

- Prod: `https://egb17p3twf.execute-api.eu-west-1.amazonaws.com/dev`
- Test: `https://pawwf1i0m7.execute-api.eu-west-1.amazonaws.com/test`


**Endpoint**: `POST /create-job`

#### Payload Overview
| Field | Type | Required | Validation |
| --- | --- | --- | --- |
| `jobNo` | string | ✔ | Trimmed. Missing triggers 400. |
| `product.id` | string | ✔ | Missing triggers 400. |
| `product.serialNumber` | string | ✖ | Trimmed if present. Validated against serial number format. Defaults to temp serial if missing. |
| `product.name` | string | ✖ | Product name. |
| `product.fuel` | string | ✖ | Fuel type. |
| `product.modelName` | string | ✖ | Model name. |
| `serviceRequestDate` | string | ✔ | ISO date string. Missing triggers 400. |
| `companyId` | string | ✖ | Trimmed if present. |
| `companyName` | string | ✖ | Trimmed if present. |
| `customers` | array | ✔ | Array of customer objects. Missing or empty triggers 400. |
| `customers[].name` | string | ✔ | Trimmed. |
| `customers[].phone` | string | ✔ | Normalized to remove spaces and special characters. |
| `customers[].address` | object | ✖ | Customer address. |
| `customers[].type` | string | ✖ | Customer type (e.g., "Homeowner", "Landlord", "Tenant"). |
| `engineers` | array | ✖ | Array of engineer objects. Affects job status. |
| `engineers[].name` | string | ✔ | Engineer name. |
| `engineers[].email` | string | ✔ | Engineer email for notifications. |
| `engineers[].companyId` | string | ✖ | Engineer's company ID. |
| `engineers[].certificate` | string | ✖ | Engineer's certificate number. |
| `estimatedSymptom` | string | ✖ | Trimmed if present. |
| `estimatedSymptomDetail` | string | ✖ | Trimmed if present. |
| `customerComment` | string | ✖ | Trimmed if present. |
| `installationDate` | string | ✖ | ISO date string. Warning logged if missing. Validated as valid date. |
| `warrantyExpiryDate` | string | ✖ | ISO date string. Warning logged if missing. |
| `fieldQualityIssue` | object | ✖ | Field quality issue details (see below). |

#### Field Quality Issue Structure
| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `fieldQualityIssue.title` | string | ✔ | Issue title. |
| `fieldQualityIssue.engineer` | object | ✔ | Engineer details (name, email, companyId, certificate). |
| `fieldQualityIssue.customers` | array | ✔ | Array of customer objects. |
| `fieldQualityIssue.creationTimestamp` | string | ✔ | ISO date-time string. |
| `fieldQualityIssue.installationTimestamp` | string | ✔ | ISO date-time string. |
| `fieldQualityIssue.product` | object | ✔ | Product details (id, name, serialNumber, modelName, fuel). |
| `fieldQualityIssue.symptom` | object | ✔ | Symptom with main and detail (code and name). |
| `fieldQualityIssue.repair` | object | ✔ | Repair with main and detail (code and name). |
| `fieldQualityIssue.attachments` | array | ✔ | Array of photo objects (uri, filename, shortFilename). |

#### Validation Flow
1. JSON body must parse; else 400 `Invalid JSON body`.
2. Checks for missing required fields (`jobNo`, `product.id`, `serviceRequestDate`); missing → 400 with combined message.
3. Logs warning if `installationDate` or `warrantyExpiryDate` is missing.
4. Serial number validated if present; invalid → 400 with serial number validation error.
5. Installation date validated as valid date; invalid → 400 `Invalid installation date: {date}`.
6. Customers array must exist and not be empty; else → 400 `There is no customer details`.
7. All validation errors combined and returned with 400 status. Failed job recorded in database.

#### Job Status Logic
- **ACCEPTED**: When exactly 1 engineer is assigned.
- **PENDING**: When more than 1 engineer is assigned.
- **OPEN**: When no engineers are assigned.

#### Processing
- Builds `Job` object with:
  - Trimmed and normalized values (phone numbers normalized)
  - Initial empty arrays for `services` and `photos`
  - Job status determined by engineer count
  - Creation timestamp (ISO string)
  - Temp serial number if none provided
- Saves job via `saveJob` service; failure returns propagated error with appropriate status code.
- Sends in-app notifications to assigned engineers (if user exists in system).
- Sends new job email to relevant parties.
- Sends SMS notification to customer.
- If job status is `ACCEPTED`, triggers visit calendar notification.

#### Responses
- Success: `201 Created`
- Errors:
  - `400 Bad Request`: Invalid JSON, missing required fields, invalid serial number, invalid installation date, or missing customers
  - `500 Internal Server Error`: Job creation failure (or specific error code from service layer)

#### Example Request
```json
{
  "jobNo": "JOB123456",
  "companyId": "CT99",
  "companyName": "Navien UK",
  "customers": [{
    "name": "John Smith",
    "phone": "+442012345678",
    "address": {
      "line1": "123 Main Street",
      "line2": "Apt 4B",
      "city": "London",
      "postcode": "EC4N 8AF",
      "country": "UK"
    },
    "type": "Homeowner"
  }],
  "product": {
    "id": "PLCB0036SD002",
    "name": "LCB;0036,C,X,GB,CE",
    "serialNumber": "1709W2211109003",
    "modelName": "LCB 700 Combi External 36KW",
    "fuel": "Kerosene"
  },
  "engineers": [{
    "name": "Jane Engineer",
    "email": "jane.engineer@company.com",
    "companyId": "CT99",
    "certificate": "570911"
  }],
  "serviceRequestDate": "2024-12-08T10:00:00Z",
  "estimatedSymptom": "No heat",
  "estimatedSymptomDetail": "Boiler not firing",
  "customerComment": "Urgent repair needed",
  "installationDate": "2023-01-15",
  "warrantyExpiryDate": "2025-01-15",
  "fieldQualityIssue": {
    "title": "Installation Quality Check",
    "engineer": {
      "name": "Soojin Lee",
      "companyId": "CT99",
      "email": "soojinlee@navienuk.com",
      "certificate": "570911"
    },
    "customers": [{
      "name": "Test Customer1",
      "phone": "+442012341111",
      "address": {
        "postcode": "EC4N 8AF",
        "country": "UK",
        "line1": "111",
        "line2": "Queen Victoria Street"
      },
      "type": "Homeowner"
    }],
    "creationTimestamp": "2024-02-16T07:00:00Z",
    "installationTimestamp": "2023-02-16T07:00:00Z",
    "product": {
      "name": "LCB;0036,C,X,GB,CE",
      "modelName": "LCB 700 Combi External 36KW",
      "serialNumber": "1709W2211109003",
      "id": "PLCB0036SD002",
      "fuel": "Kerosene"
    },
    "symptom": {
      "main": {
        "name": "No heat",
        "code": "A0100199"
      },
      "detail": {
        "name": "Transformer",
        "code": "A0100266"
      }
    },
    "repair": {
      "main": {
        "name": "Unit Replacement",
        "code": "B0100006"
      },
      "detail": {
        "name": "Defective Burner",
        "code": "B0100047"
      }
    },
    "attachments": [{
      "uri": "https://navien-service-agent-dev-photo-storage.s3.eu-west-1.amazonaws.com/5ae0cb33-5a85-4c39-a326-7ef84fbb18a4.jpg",
      "shortFilename": "5ae0cb33-5a85-4c39-a326-7ef84fbb18a4.jpg",
      "filename": "file:///data/user/0/com.navien.serviceagent/cache/Camera/5ae0cb33-5a85-4c39-a326-7ef84fbb18a4.jpg"
    }]
  }
}
```
