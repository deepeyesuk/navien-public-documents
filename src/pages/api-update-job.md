---
title: "Update Job API"
description: "API documentation for the Update Job endpoint in Service Agent application"
icon: "🔄"
layout: ../layouts/MarkdownLayout.astro
---

## Service Agent - Update Job

BaseUrl

- Prod: `https://egb17p3twf.execute-api.eu-west-1.amazonaws.com/dev`
- Test: `https://pawwf1i0m7.execute-api.eu-west-1.amazonaws.com/test`


**Endpoint**: `POST /job/{jobNo}/update`

#### Path Parameters
| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `jobNo` | string | ✔ | Job number to update. Trimmed. Missing triggers 400. |

#### Payload Overview
The update job endpoint accepts a partial Job object (`UpdateJobRequest`), allowing you to update any subset of job fields. All fields are optional unless otherwise noted.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `jobStatus` | string | ✖ | Job status (PENDING, OPEN, ACCEPTED, BOOKED, ON-ROUTE, ARRIVED, REVISIT, COMPLETED, CANCELLED). Special validation for COMPLETED status. |
| `product` | object | ✖ | Product details. |
| `product.serialNumber` | string | ✔* | *Required when `jobStatus` is COMPLETED. Cannot be temporary serial number. |
| `customers` | array | ✖ | Array of customer objects. |
| `engineers` | array | ✖ | Array of engineer objects. Explicitly set (replaces existing). |
| `services` | array | ✖ | Array of service records with symptom, repair, and parts. |
| `photos` | Photo[] | ✖ | Array of photo objects. Photos are copied to permanent storage. |
| `visit` | object | ✖ | Visit date and AM/PM information. |
| `boilerReadings` | object | ✖ | Boiler readings including gas/oil pressure, emissions, etc. |
| `jobNotes` | string | ✖ | Notes about the job. |
| `installationDate` | string | ✖ | ISO date string for product installation. |
| `warrantyExpiryDate` | string | ✖ | ISO date string for warranty expiration. |
| `estimatedSymptom` | string | ✖ | Estimated symptom description. |
| `estimatedSymptomDetail` | string | ✖ | Detailed estimated symptom. |
| `customerComment` | string | ✖ | Customer's comments. |
| `reportDateTime` | string | ✖ | ISO date-time string. Defaults to current timestamp if not provided. |
| `actionAdviceComplete` | object | ✖ | Action advice completion details (see below). |
| `designRelatedBehaviour` | boolean | ✖ | Flag for design-related behavior issues. |
| `similarIssuesObserved` | boolean | ✖ | Flag for similar issues observed. |

#### Action Advice Complete Structure
| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `productFault` | string | ✔ | Product fault description. |
| `actionAdviceNote` | string | ✔ | Action advice notes (max 2000 characters). |
| `photos` | Photo[] | ✔ | Array of photo objects. Photos are copied to permanent storage. |

#### Photo Structure
| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `uri` | string | ✔ | Full URI of the photo. |
| `filename` | string | ✔ | Original filename. |
| `shortFilename` | string | ✔ | S3 key/short filename. |

#### Validation Flow
1. Path parameter `jobNo` must be present; missing → 400 `jobNo is required`.
2. JSON body must parse; else 400 `Invalid JSON body`.
3. If `jobStatus` is `COMPLETED`:
   - `product.serialNumber` must exist; missing → 400 `Cannot compolete the job without serial number`.
   - `product.serialNumber` cannot be temporary serial number; invalid → 400 `Cannot compolete the job with temporary serial number`.
4. Job must exist; not found → error response with appropriate status code.

#### Job Merging Logic
- Uses lodash `mergeWith` to merge existing job with update request.
- `null`, `undefined`, and empty arrays in update request are ignored (existing values preserved).
- `engineers` array is explicitly replaced (not merged) when provided in update request.
- `reportDateTime` defaults to current timestamp if not provided in update request.
- `actionAdviceNote` is truncated to 2000 characters if present.
- **Field Quality Issue**: Only created/recreated when either `designRelatedBehaviour` or `similarIssuesObserved` is true in update request. Otherwise, existing `fieldQualityIssue` is preserved.

#### Photo Handling
- Photos in `photos` array and `actionAdviceComplete.photos` are copied to permanent S3 storage.
- Copy operation updates metadata and sets content type to `image/jpeg`.
- Failed photo copies are logged but do not fail the update.

#### Post-Update Processing
1. Job is persisted to database via `saveJob` service.
2. Job update message is sent to CIC system (SQS).
3. **If job status is BOOKED**:
   - Triggers visit calendar notification.
   - Sends SMS to customer.
4. **If job status is COMPLETED**:
   - Sends SMS to customer.

#### Responses
- Success: `200 OK` with updated job object and message `Job updated successfully`.
- Errors:
  - `400 Bad Request`: Missing jobNo, invalid JSON, missing serial number for completed job, or temporary serial number for completed job.
  - `404 Not Found`: Job does not exist.
  - `500 Internal Server Error`: Database or service layer errors.

#### Example Request - Update Job Status to BOOKED
```json
{
  "jobStatus": "BOOKED",
  "visit": {
    "date": "2024-12-22",
    "ampm": "AM"
  }
}
```

#### Example Request - Complete Job
```json
{
  "jobStatus": "COMPLETED",
  "product": {
    "id": "PLCB0036SD002",
    "name": "LCB;0036,C,X,GB,CE",
    "serialNumber": "1709W2211109003",
    "modelName": "LCB 700 Combi External 36KW",
    "fuel": "Kerosene"
  },
  "services": [{
    "symptom": {
      "code": "A0100199",
      "name": "No heat"
    },
    "repair": {
      "code": "B0100047",
      "name": "Defective Burner"
    },
    "part": {
      "code": "P123456",
      "name": "Burner Assembly",
      "quantity": 1
    },
    "status": "COMPLETED"
  }],
  "boilerReadings": {
    "logBook": true,
    "serviceLog": true,
    "tightnessTest": true,
    "gasPressure": {
      "working": 21.5,
      "standing": 22.0
    },
    "gasEmission": {
      "CO2High": 9.2,
      "CO2Low": 9.1
    }
  },
  "photos": [{
    "uri": "https://uri.com/abc123.jpg",
    "shortFilename": "abc123.jpg",
    "filename": "file:///data/user/0/cache/Camera/abc123.jpg"
  }],
  "jobNotes": "Burner replaced and tested. System operating normally."
}
```

#### Example Request - Add Action Advice
```json
{
  "actionAdviceComplete": {
    "productFault": "Design flaw in heat exchanger",
    "actionAdviceNote": "Recommend product design review for heat exchanger assembly. Multiple similar failures observed.",
    "photos": [{
      "uri": "https://uri.com/def456.jpg",
      "shortFilename": "def456.jpg",
      "filename": "file:///data/user/0/com.navien.serviceagent/cache/Camera/def456.jpg"
    }]
  },
  "designRelatedBehaviour": true,
  "similarIssuesObserved": true
}
```

#### Example Request - Create Field Quality Issue
```json
{
  "designRelatedBehaviour": true,
  "similarIssuesObserved": true
}
```

When either `designRelatedBehaviour` or `similarIssuesObserved` is set to `true`, the system automatically generates a `fieldQualityIssue` object from the current job data:
- Title: "Field Quality Issue - {jobNo}"
- Engineer: First engineer from job
- Customers: Current job customers
- Creation timestamp: Current time
- Installation timestamp: From job's installationDate
- Product: Current job product
- Symptom/Repair: From first service record
- Attachments: Current job photos

#### Example Success Response
```json
{
  "origin": "service-agent-api",
  "message": "Job updated successfully",
  "data": {
    "jobNo": "JOB123456",
    "jobStatus": "COMPLETED",
    "companyId": "CT99",
    "companyName": "Navien UK",
    "serviceRequestDate": "2024-12-08T10:00:00Z",
    "reportDateTime": "2024-12-21T22:00:00Z",
    "product": {
      "id": "PLCB0036SD002",
      "name": "LCB;0036,C,X,GB,CE",
      "serialNumber": "1709W2211109003",
      "modelName": "LCB 700 Combi External 36KW",
      "fuel": "Kerosene"
    },
    "customers": [{
      "name": "John Smith",
      "phone": "+442012345678",
      "address": {
        "line1": "123 Main Street",
        "postcode": "EC4N 8AF",
        "country": "UK"
      }
    }],
    "engineers": [{
      "name": "Jane Engineer",
      "email": "jane.engineer@company.com",
      "companyId": "CT99",
      "certificate": "570911"
    }],
    "services": [{
      "symptom": {
        "code": "A0100199",
        "name": "No heat"
      },
      "repair": {
        "code": "B0100047",
        "name": "Defective Burner"
      },
      "part": {
        "code": "P123456",
        "name": "Burner Assembly",
        "quantity": 1
      },
      "status": "COMPLETED"
    }],
    "photos": [{
      "uri": "https://uri.com/abc123.jpg",
      "shortFilename": "abc123.jpg",
      "filename": "file:///data/user/0/com.navien.serviceagent/cache/Camera/abc123.jpg"
    }],
    "boilerReadings": {
      "logBook": true,
      "serviceLog": true,
      "tightnessTest": true,
      "gasPressure": {
        "working": 21.5,
        "standing": 22.0
      }
    },
    "jobNotes": "Burner replaced and tested. System operating normally.",
    "creationDateTime": "2024-12-08T11:00:00Z",
    "installationDate": "2023-01-15",
    "warrantyExpiryDate": "2025-01-15"
  }
}
```
