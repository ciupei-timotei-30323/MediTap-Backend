## Overview

The MediTap API is a RESTful backend service built with ASP.NET Core. It provides endpoints to manage medical records, including patients, medics, appointments, medications, symptoms, and affections.

## Base URL

All API requests should be prefixed with the base URL of the host. For local development, this is typically:
`http://localhost:5142` or `https://localhost:7116`

## Authentication

The API uses **JSON Web Tokens (JWT)** for authentication and authorization.
Clients must include the token in the `Authorization` HTTP header using the Bearer schema:
`Authorization: Bearer <token>`

### Roles

The system utilizes role-based access control (RBAC). The primary roles are:

* **Medic**: Healthcare providers. Note that Medic with `Id = 1` functions as the System Administrator (capable of creating other Medics and Patients).
* **Patient**: Standard users accessing their own medical data.

---

## Endpoint Summary

| HTTP Method | Endpoint | Authorization Role | Description |
| --- | --- | --- | --- |
| **Authentication** |  |  |  |
| POST | `/api/Auth/login` | Public | Authenticates a user (Medic or Patient) and returns a JWT token. |
| **Medic** |  |  |  |
| GET | `/api/Medic/me` | Medic | Retrieves the profile summary of the currently authenticated Medic. |
| POST | `/api/Medic` | Admin (Medic ID=1) | Creates a new Medic profile. |
| POST | `/api/Medic/me/scan` | Medic | Scans a patient's card to link the Patient to the Medic and returns a profile summary. |
| **Patient** |  |  |  |
| GET | `/api/Patient/me` | Patient | Retrieves the complete profile of the currently authenticated Patient. |
| POST | `/api/Patient` | Admin (Medic ID=1) | Creates a new Patient profile. |
| GET | `/api/Patient/{id}/profile` | Medic | Retrieves the profile summary of a specific patient. |
| GET | `/api/Patient/{id}/appointment` | Medic | Retrieves a list of appointments for a specific patient. |
| GET | `/api/Patient/{id}/symptom` | Medic | Retrieves a list of symptoms recorded by a specific patient. |
| GET | `/api/Patient/{id}/affection` | Medic | Retrieves a list of affections diagnosed for a specific patient. |
| GET | `/api/Patient/{id}/medication` | Medic | Retrieves a list of medications prescribed to a specific patient. |
| POST | `/api/Patient/me/symptom` | Patient | Adds a new symptom to the currently authenticated patient's profile. |
| **Appointment** |  |  |  |
| GET | `/api/Appointment/{id}` | Medic, Patient | Retrieves a specific appointment by its ID. |
| POST | `/api/Appointment` | Medic, Patient | Creates a new appointment. |
| DELETE | `/api/Appointment/{id}` | Medic, Patient | Deletes a specific appointment. |
| **Affection** |  |  |  |
| GET | `/api/Affection/{id}` | Medic | Retrieves a specific affection record by its ID. |
| POST | `/api/Affection` | Medic | Creates a new affection record for a linked patient. |
| DELETE | `/api/Affection/{id}` | Medic | Deletes a specific affection record. |
| **Medication** |  |  |  |
| GET | `/api/Medication/{id}` | Medic | Retrieves a specific medication record by its ID. |
| POST | `/api/Medication` | Medic | Prescribes a new medication to a linked patient. |
| PUT | `/api/Medication/{id}` | Medic | Updates an existing medication record. |
| DELETE | `/api/Medication/{id}` | Medic | Deletes a specific medication record. |
| **Symptom** |  |  |  |
| PUT | `/api/Symptom/{id}` | Patient | Updates an existing symptom record (e.g., toggles active status). |
| DELETE | `/api/Symptom/{id}` | Patient | Deletes a specific symptom record. |

---

## Standard HTTP Status Codes

The API returns standard HTTP status codes to indicate the success or failure of a request:

* **200 OK**: The request was successful, and the response body contains the requested data.
* **201 Created**: The resource was successfully created. The response includes a `Location` header pointing to the new resource.
* **204 No Content**: The request was successful, but there is no data to return (commonly used for successful DELETE and PUT requests).
* **400 Bad Request**: The request was invalid or could not be understood by the server (e.g., validation errors, invalid CNP, duplicate email).
* **401 Unauthorized**: The request requires user authentication, and the token is missing, invalid, or expired.
* **403 Forbidden**: The authenticated user does not have the necessary permissions (roles) to access the resource.
* **404 Not Found**: The requested resource could not be found, or the user does not have authorization to view that specific entity.
* **500 Internal Server Error**: An unexpected condition was encountered by the server.

---

## Core Data Transfer Objects (DTOs) Reference

### `LoginRequestDTO`

Used for authentication.

* `Email` (string): The user's email address.
* `Password` (string): The user's password.
* `Role` (string): "Medic" or "Patient".

### `PatientCreationDTO`

Used when an Administrator creates a new patient.

* `DateOfBirth` (DateOnly)
* `CNP` (string): Romanian National Identification Number.
* `FirstName` (string)
* `LastName` (string)
* `Password` (string)
* `Email` (string, optional)
* `PhoneNumber` (string, optional)
* `Address` (string, optional)

### `AppointmentCreationDTO`

Used when scheduling an appointment.

* `Date` (DateTime): The scheduled time of the appointment.
* `IssueDate` (DateTime): When the appointment was requested.
* `Title` (string, optional)
* `Description` (string, optional)
* `OtherUserId` (integer): If the requester is a Medic, this is the PatientId. If the requester is a Patient, this is the MedicId.

### `MedicCreationDTO`

Used when an Administrator creates a new Medic profile.

* `FirstName` (string)
* `LastName` (string)
* `Specialty` (string)
* `MedicStatus` (string): Must be one of `Active`, `Inactive`, `FormerMember`, `Unconfirmed`.
* `Password` (string)
* `Email` (string, optional)
* `PhoneNumber` (string, optional)

### `MedicationCreationDTO`

Used when a Medic prescribes medication to a Patient.

* `PatientId` (integer): The ID of the patient receiving the prescription.
* `Name` (string): The name of the medication.
* `Brand` (string, optional)
* `Quantity` (float)
* `UnitOfMeasure` (string)
* `PercentReimbursed` (integer)
* `IssueDate` (DateOnly): Must not be a future date.
* `StartDate` (DateOnly, optional): Must be present or future date.
* `EndDate` (DateOnly, optional): Must be present or future date.

### `AffectionCreationDTO`

Used to record a medical condition or diagnosis.

* `PatientId` (integer)
* `Name` (string): The name of the affection/condition.
* `DiagnoseDate` (DateOnly): Must not be a future date.
* `Description` (string, optional)

### `SymptomCreationDTO`

Used by Patients to log a new symptom.

* `Name` (string)
* `AddedTime` (DateTime): The time the symptom is logged. Must not be a future date.
* `StartOfSymptom` (DateOnly, optional): Must not be a future date.
* `Description` (string, optional)

### `SymptomUpdateDTO`

Used by Patients to update the status of an existing symptom.

* `isPresent` (boolean): Set to `false` if the symptom has stopped.
* `StartOfSymptom` (DateOnly, optional)
* `Description` (string, optional)

---

## Payload & Response Examples

To ensure predictable integration, below are standardized examples of request payloads and expected responses for critical endpoints.

### 1. Authenticate User (`POST /api/Auth/login`)

**Request Body:**

```json
{
  "email": "medic@example.com",
  "password": "securepassword123",
  "role": "Medic"
}

```

**Success Response (200 OK):**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "role": "Medic",
  "id": 2
}

```

### 2. Create Patient (`POST /api/Patient`)

**Headers:** `Authorization: Bearer <Admin_Token>`

**Request Body:**

```json
{
  "dateOfBirth": "1990-05-15",
  "cnp": "1900515123456",
  "firstName": "John",
  "lastName": "Doe",
  "password": "patientpassword",
  "email": "johndoe@example.com",
  "phoneNumber": "+40712345678",
  "address": "123 Main St, Bucharest"
}

```

**Success Response (200 OK):**
*(Returns the created Patient entity, omitting the plain-text password)*

```json
{
  "id": 4,
  "dateOfBirth": "1990-05-15",
  "cnp": {
    "codNumericPersonal": "1900515123456"
  },
  "firstName": "John",
  "lastName": "Doe",
  "email": {
    "emailAddress": "johndoe@example.com",
    "isValid": true
  },
  "uname": "P-John-1900-a1b2c3d4",
  "phoneNumber": {
    "number": "+40712345678",
    "isValid": true
  },
  "address": "123 Main St, Bucharest"
}

```

### 3. Link Patient via Scan (`POST /api/Medic/me/scan`)

**Headers:** `Authorization: Bearer <Medic_Token>`

**Request Body:**

```json
{
  "id": 4,
  "uname": "P-John-1900-a1b2c3d4",
  "passwordHashed": "$2a$11$e.ExampleHashedPasswordString..."
}

```

**Success Response (200 OK):**

```json
{
  "id": 4,
  "firstName": "John",
  "lastName": "Doe",
  "cnp": "1900515123456",
  "email": "johndoe@example.com",
  "phoneNumber": "+40712345678"
}

```

---

## Domain Constraints & Business Logic

The API strictly enforces several domain rules at the service layer. Violating these rules will result in a `400 Bad Request` or `500 Internal Server Error` with a descriptive message.

| Validation Domain | Rule Enforced | Exception Raised |
| --- | --- | --- |
| **CNP (Romanian ID)** | Must be exactly 13 characters and pass the official checksum calculation algorithm. Must be unique in the system. | `InvalidCNPException`, `CNPAlreadyExistsException` |
| **Email Address** | Must contain an `@` symbol and a valid domain format. | `InvalidEmailException` |
| **Phone Number** | Verified against the Google `libphonenumber-csharp` library. Assumes Romanian (`RO`) regional formatting by default. | `InvalidPhoneNumberException` |
| **Dates (Birth/Past)** | Date of Birth, Affection Diagnose Dates, and Symptom Start Dates cannot be set in the future. | `FutureDateException` |
| **Dates (Appointments)** | Appointment dates must be in the future. Issue dates must be past or present. | Standard `BadRequest` |
| **Scheduling Conflicts** | Appointments cannot be scheduled if either the Medic or the Patient has another appointment within a +/- 15-minute window. | Standard `BadRequest` |
| **Username Generation** | System-generated upon creation. Format: `[Role_Prefix]-[FirstName]-[Identifier]-[UUID_Hash]`. Must be unique. | `UnameAlreadyExistsException` |

## Authorization Constraints

The API utilizes strict cross-referencing to ensure data privacy:

* **Medic Access:** A Medic cannot query, modify, or add records (Symptoms, Affections, Medications, Appointments) for a Patient unless the two entities are explicitly linked via the `MedicPatient` relationship table (established via the `/scan` endpoint).
* **Patient Access:** A Patient can only view and modify their own records. They cannot access records of other patients.
* **Deletion:** Resources can only be deleted by the user who owns them or the Medic who created them, provided the authorization link still exists.


### Data Architecture & Entity Relationships

Understanding the underlying data model is crucial for integrating with the MediTap API effectively. The system uses Entity Framework Core with a PostgreSQL database, structured around the following core relationships:

* **Patient ↔ Medic (Many-to-Many):** A patient can be treated by multiple medics, and a medic treats multiple patients. This relationship is established when a Medic scans a Patient's card (via the `MedicPatient` linking table). A valid link here is the primary authorization gateway for accessing a patient's medical records.
* **Patient ↔ Appointment (One-to-Many):** A patient can have multiple appointments.
* **Medic ↔ Appointment (One-to-Many):** A medic can oversee multiple appointments.
* **Patient ↔ Medication (One-to-Many):** A patient can have multiple prescribed medications.
* **Medic ↔ Medication (One-to-Many):** A medic can issue multiple prescriptions.
* **Patient ↔ Symptom (One-to-Many):** A patient logs their own symptoms over time. (Note: Medics can view these symptoms and mark them as "checked", establishing a nullable many-to-one relationship with the checking Medic).
* **Patient ↔ Affection (One-to-Many):** A patient can be diagnosed with multiple affections (conditions).
* **Medic ↔ Affection (One-to-Many):** A medic can diagnose multiple affections.

---

### Error Handling & Custom Exceptions

The API utilizes domain-specific exceptions to enforce business rules. When these exceptions are caught by the controllers, they are mapped to standard HTTP responses (typically `400 Bad Request` or `500 Internal Server Error` depending on the severity).

If a request fails validation, the response body will typically contain a plain-text string detailing the specific error, originating from one of the following system exceptions:

* `InvalidCNPException`: "CNP provided is not valid."
* `InvalidEmailException`: "The Email address provided is not valid."
* `InvalidPhoneNumberException`: "The phone number provided is not valid."
* `UnameAlreadyExistsException`: "Username already exists. Please try again."
* `CNPAlreadyExistsException`: "CNP already exists."
* `FutureDateException`: "Selected date is in the future."
* `PatientNotFoundException`: Returns a `404 Not Found` when attempting to query a patient that does not exist or is not linked.
* `NoAppointmentFoundException` / `NoMedicAssignedException`: Internal domain exceptions utilized by the service layer to halt unauthorized processing.

---


