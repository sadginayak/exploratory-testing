# Exploratory Testing Session Report
*SBTM-Style Session*

## 1. Session Metadata

| Field | Details |
|---|---|
| **Tester** | Sadgi Nayak (independent tester)|
| **Session Duration** | 60 minutes |
| **Date** | 08 September 2026 |
| **Application Under Test** | Orthanc
| **Environments Used** | Default browser; Chrome, Microsoft |
| **Session Type** | Session-Based Test Management (SBTM) — single charter, exploratory |

## 2. Charter
Explore Anonymize with sample CT DICOM files to discover anonymization is applied succesfully.

**Areas to Explore**
- Patient Anonymization

## 3. Pre requisites- 
Sample DICOM files are uploaded in the application.
Entries for the uploaded series are  shown.

## 4. Navigation Path Observed
Click on uploaded record->Select Anonymize icon->Enter new Patient ID and Patient Name->Click Anonymize -> Click on Show modified resources

### Valid inputs (Positive) -
#### Data-
Pateint ID- NewPatient1
, Patient Name - Anonymized NewPatientTest1
, Patient Sex- Female, Study Description- Functional


#### Expected- 
Anonymized patient record is created and count of number of patient is updated.

#### Observation-  
After clicking Anonymize button new patient data row is created but previous one is not displayed immediately, there is a lag or required hard Refresh. Same in settings/System Info, Patient, Studies and Series shows stale count as 1 instead of 2 until user performs Hard reset. At first it appears that the original Patient data is lost, which made me redo the whole process.

#### Risk- Severity(Medium) 
User can conclude anonymized data is overwritten on original data, which can lead to effort duplication or false reporting.

### Duplicate data -
#### Steps: 
Anonymize the original source study twice, using identical Patient ID/Name/Description both times, 

Pateint ID- NewPatient1
, Patient Name - Anonymized NewPatientTest1
, Patient Sex- Female, Study Description: Functional

#### Observation: 
Another Study record is created with same params but different Study ID.

#### Risk: 
Medium — User would need Study instance ID in advance to differentiate between the records which is inconvenient and can lead to accessing wrong records. In addition on UI, user can not filter the data based on session ID. 

#### Re-anonymizing an already-anonymized study with short length data
Patient name- A1
Patient ID-P1

#### Observation -
 Created a anonymized record of an existing anonymised study record created above. User can either anonymise original or already anonymized data to create study.

#### Risk- High-  
Anonymised record can have alreation to the original series or images, allowing anonymisation of it can mislead the further studies.


### With Empty values 
Patient ID, name and Study desccription is empty 

#### Observation - 
new study record is created with empty fields  but only Study Instance 

#### Risk- High- 
Due to no separate Field for Fidning study with Study instance data can not be discovered through UI incase of multiple such records exist, they may be difficult to identify or distinguish through the UI.



#### Open Questions:

Is creating multiple anonymized records with identical visible metadata expected behaviour, if yes, how does the user distinguish them?
