# Exploratory Testing Session Report
*SBTM-Style Session*

## 1. Session Metadata

| Field | Details |
|---|---|
| **Tester** | Sadgi Nayak (independent tester)|
| **Session Duration** | 60 minutes |
| **Date** | 08 September 2026 |
| **Application Under Test** | Orthanc
| **Environments Used** | Postman/Browser direct calls |
| **Session Type** | Session-Based Test Management (SBTM) — single charter, exploratory |

## 2. Charter
Explore the Anonymize API endpoint with sample CT DICOM files to discover how the system handles re-anonymization, empty/null input, UI-vs-API synchronization.

## 3. Oracle

1. After an anonymized record is successfully created, the API should expose the resulting record without an unexpected delay. The UI should eventually reflect the same server state.
4. System should distinguish the original study from anonymized/derived studies.
2. Empty required fields should either be rejected (validation error) or auto-filled with a traceable placeholder. Incase such values are accepted, the record should be identifiable.

## 4. Pre requisites- 
Sample DICOM files are uploaded in the application.
Entries for the uploaded series are  shown.

## 5. Navigation Path Observed
Tool used: Postman / direct browser calls
Endpoint: POST /studies/{id}/anonymize
Get {id} via GET /studies first, then call anonymize with a JSON body.

### Valid inputs (Positive) -
#### Data-
Pateint ID- NewPatient1
, Patient Name - Anonymized NewPatientTest1
, Patient Sex- Female, Study Description- Functional


### Objective

Investigate whether the Orthanc API reflects a newly anonymized study
before the UI displays the updated state.

### Step 1 — Initial API State

**Request**

[GET /studies](http://localhost:8042/studies)

**Timestamp**

20.53.00

**Response**

HTTP 200 OK

Studies count: 1

**Evidence**

Postman response 200
```json
[
    "a2390fab-3be3e31b-268f6c22-4eb2e70f-6e5d1726"
]
```
### Step 2 — Anonymization
Performed anonymization through Orthanc UI.

Input:

- Patient ID:Newpatient1
- Patient Name:Anonymized NewpatientTest1
- Patient sex: Female
- Study Description:FUNCTIONAL
- CLick Anonymize

### Step 3 — Immediate API Check

**Request**

GET /studies

**Timestamp**

20.53.03

**Response**
```json
[
    "9b2ab3fa-54035550-ba9e2fd2-84aca27a-abf469cf",
    "a2390fab-3be3e31b-268f6c22-4eb2e70f-6e5d1726"
]
```
### Step 4 — UI Comparison

UI state immediately after anonymization:

Still shown one record

UI state after refresh:

Refreshed and shown two records

### Observation
API reponse is instant as the request is sent its getting updated,although UI requires Hard refresh to reflect.

### Risk (Medium): 
UI does not reflect server state immediately, which can mislead a user into believing data loss occurred (confirmed firsthand in this session).


## Investigation 2: Re-anonymization

### Objective
Investigate whether an already anonymized study can be anonymized again
and how the API represents the connection.

### Test Data

Original Study:
[a2390fab-3be3e31b-268f6c22-4eb2e70f-6e5d1726](http://localhost:8042/studies/a2390fab-3be3e31b-268f6c22-4eb2e70f-6e5d1726)

First Anonymized Study:
[9b2ab3fa-54035550-ba9e2fd2-84aca27a-abf469cf](http://localhost:8042/studies/9b2ab3fa-54035550-ba9e2fd2-84aca27a-abf469cf)

Re-anonymized Study:
[1c68d5f5-518acf3e-5f535fe4-7a6fb4e6-e17724ba](http://localhost:8042/studies/1c68d5f5-518acf3e-5f535fe4-7a6fb4e6-e17724ba)

### Observation

Orthanc allowed anonymization to be performed on an already anonymized
study.

A new Patient and Study resource were created. The new study has a
different Study ID, StudyInstanceUID, Series ID and ParentPatient.

### Result
AnonymizedFrom records only the immediately preceding source study, not the original study. For a re-anonymized study, tracing the the original requires manually following the chain: re-anonymized → first anonymized → original.

### Potential Risk

If re-anonymization is not an intended workflow, users may unintentionally
create multiple generations of derived data and lose a direct relationship
to the original source study.

### Investigation 3: Empty/null input

### Objective

Investigate how the anonymization API handles empty Patient ID and
Patient Name values.

### Request

POST /studies/a2390fab-3be3e31b-268f6c22-4eb2e70f-6e5d1726/anonymize

### Request Body

```json
{
  "Replace": {
    "PatientID": "",
    "PatientName": ""
  }
}
```
### Expected

Empty values should either be rejected with a validation error or,
if accepted, the resulting record should remain identifiable.

### Observation: 
The API responses requiring "Force": true to replace PatientID, but the Orthanc UI allows editing PatientID freely with no warning or confirmation.

### Risk (Medium): 
Inconsistency between UI and API.

### After adding Force:true

```json
{
  "Replace": {
    "PatientID": "",
    "PatientName": ""
  },
  "Force": true
}
```
Response:

```json
{
      "PatientBirthDate" : "",
      "PatientID" : "",
      "PatientName" : "",
      "PatientSex" : ""
   }
```

### Observation: 
With Force: true, the API accepted empty PatientID and PatientName values and created a new anonymized study. The resulting study contains both PatientID: "" and PatientName: "".

### Result: 
Confirmed. The API permits empty PatientID and PatientName when Force: true is supplied, resulting in an anonymized study with blank patient identification fields.

### Risk (High): 

An anonymized study can be created with blank Patient ID and Patient Name. This may make the resulting study difficult to identify or distinguish from other records through the UI.