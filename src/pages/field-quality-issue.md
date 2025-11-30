---
title: "fieldQualityIssue"
description: "Documentation for the fieldQualityIssue JSON field structure and its properties."
icon: "📋"
layout: ../layouts/MarkdownLayout.astro
---

# FieldQualityIssue

## Structure

This object contains all the details of the issue report.

| Field                   | Data Type | Description                                                                                 |
| :---------------------- | :-------- | :------------------------------------------------------------------------------------------ |
| `title`                 | String    | The title of the issue report. (e.g., "test test")                                          |
| `engineer`              | Object    | The name of the associated engineer                                                         |
| `customers`             | Array     | An array containing customer details.                                                       |
| `creationTimestamp`     | String    | The date and time the report was created. An ISO 8601 format (e.g., "2024-02-16T07:00:00Z") |
| `installationTimestamp` | String    | The date the product was installed. May be an empty string `""` if not provided.            |
| `product`               | Object    | Project object containing name, serialNumber, id, etc.                                      |
| `symptom`               | Object    | symptom object                                                                              |
| `repair`                | Object    | repair object                                                                               |
| `attachments`           | Array     | An array of attachment objects. Will be an empty array `[]` if no files are attached.       |

## Example

```json
"fieldQualityIssue": {
  "title": "test test",
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
```

## Context

The `fieldQualityIssue` field will be part of the top-level JSON blob, alongside other fields such as:

- `jobNo`: Job number identifier
- `companyId`: Company identifier
- `customers`: Array of customer information
- `engineers`: Array of engineer information
- `product`: Product details
- `services`: Service records
- `boilerReadings`: Technical readings
- And other job-related fields

## Complete JSON Structure Example

Here's an example showing how `fieldQualityIssue` fits into the complete JSON structure:

```json
{
  "jobNo": "2025112800050",
  "companyId": "0003500747",
  "companyName": null,
  "creationDateTime": "2025-11-28T10:26:05.182Z",
  "customerComment": "Rate: Standard ?80\\r\\nFault :Screen blank at boiler and boiler not working. No heating or hot water\\r\\nSA: Multi-Heat Services Ltd\\r\\nDate: 11/28 PM\\r\\n\\r\\nCalled HO and inform them about the visit.",
  "customers": [
    {
      "address": {
        "country": "UK",
        "postcode": "CF00 0AA",
        "line2": "Cowbridge South Gxxxxxxx",
        "line1": "Cxxxxxxxx"
      },
      "phone": "+447818075000",
      "name": "Ian Lxxxx",
      "title": ""
    },
    {
      "address": {
        "country": "UK",
        "postcode": "CF00 0AA",
        "line2": "Cowbridge South Gxxxxxxx",
        "line1": "Cxxxxxxxx"
      },
      "phone": null,
      "name": null
    }
  ],
  "engineers": [
    {
      "companyId": "0003500747",
      "name": "Gareth Hxxx",
      "email": "admin@xxxxxservices.com"
    }
  ],
  "estimatedSymptom": "A0100178",
  "estimatedSymptomDetail": "A0100191",
  "installationDate": "2021-09-02",
  "jobStatus": "COMPLETED",
  "photos": [],
  "product": {
    "serialNumber": "1709W2112909017",
    "name": "LCB;0036,C,X,GB,CE",
    "id": "PLCB0036SD002",
    "modelName": "LCB700 Combi External 36kW",
    "fuel": "Kerosene"
  },
  "serviceRequestDate": "2025-11-28",
  "services": [
    {
      "symptom": {
        "code": "A0100235",
        "name": "No display"
      },
      "repair": {
        "code": "B0100182",
        "name": "Part Required"
      },
      "status": "ORDERED"
    }
  ],
  "warrantyExpiryDate": "",
  "reportDateTime": "2025-11-30T10:44:45.479Z",
  "visit": {
    "date": "2025-11-28",
    "time": "PM"
  },
  "boilerReadings": {
    "logBook": true,
    "serviceLog": true,
    "tightnessTest": false,
    "warningNotice": false,
    "warningMessage": "",
    "flue": "Pass",
    "ventilation": "Pass",
    "deviceSatisfactory": "Pass",
    "oilPressure": {
      "high": 999,
      "low": 999
    },
    "oilEmission": {
      "O2High": 999,
      "O2Low": 999,
      "CO2High": 999,
      "CO2Low": 999,
      "COHigh": 999,
      "COLow": 999,
      "NOXHigh": 999,
      "NOXLow": 999
    }
  },
  "engineer": {
    "companyId": "0003500000",
    "certificate": "C103649",
    "name": "Daniel Hxxxxxx"
  },
  "jobNotes": "Fuse blown caused by defective 3way valve\\nPlease raise new job for replacement of 3 way valve",
  "fieldQualityIssue": {
    "title": "test test",
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

> **Note**: The `fieldQualityIssue` object appears at the same level as other top-level fields like `jobNo`, `customers`, `engineers`, etc.
