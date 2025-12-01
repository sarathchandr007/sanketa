## 1. Overview

The Admin pre-populates the database with shadow records via CSV. When a user registers, the system attempts to link their live account to a Shadow Record using a unique identifier.

Success: The account is verified and instantly ACTIVE. ( User creation in the database actually happens here.)

Failure: The account is created but flagged as PENDING for Admin review. ( We would probably just need to maintain this in a separate table)

## 2. The Workflow Logic

### Step A: Data Ingestion (Admin Side)

- Admin uploads a CSV file containing the "Source of Truth" (Student/Parent/Teacher details).
- System validates the CSV and stores these as shadow records in the database.

### Step B: User Registration (Client Side)

- User enters personal details (Name, Phone) and a Unique Identifier (Admission Number OR Class + Roll No). Email id 
- System generates a username programmatically (e.g., firstname.lastname).
- System performs a Lookup & Match Operation.
- These details are entered into a separate table
### Step C: The Decision Engine (Server Side)

- Look up the Unique Identifier in the Unclaimed_Profiles table.

IF Match Found:
- Validate secondary data (e.g., Does the Phone Number match the record? We can just throw an error in such cases asking the user to fix this).
- Action: Create the user entity in the database. To simplify furhter workflows and reduce duplicate data, we can clean up the user entry in shadow records table.
- Status: User account is created
- Result: User gets logged in. 

IF Match NOT Found (or Record already claimed):
- Action: Create User_Account without linking.
- Status: Set to PENDING_APPROVAL.
- Result: User sees: "Account created but pending verification. Please wait for Admin approval."

## 3. Username Generation Strategy

Since users do not choose their usernames, you must handle duplicates (e.g., two students named "Vijay Simha").

Algorithm:
- Sanitize input: firstname + lastname (Lowercase, remove spaces).
- Check DB for existence.
- If exists, append the last 4 digits of their Phone or Admission ID. Admission ID is probably not reliable as it may create confusion if the school is changed later , we can go with phone number 
- Example: vijay.simha -> vijay.simha8821 (Very rare chance that the last numbers would collide but we will figure it out later)

## 4. Recommended Data Fields

### A. The CSV File (Master Data Upload)

Admin uploads this. It acts as the validation key.

Field Header | Required? | Purpose
--- | --- | ---
Admission_Number | Yes | Primary Key for matching. This key would be different from the key in the final users table. (For teachers: Employee ID).
First_Name | Yes | Validation & Profile creation.
Last_Name | Yes | Validation & Profile creation.
Role | Yes | Student, Parent, or Teacher.
Class_Grade | No | Useful for grouping (e.g., "10", "5").
Roll_Number | No | Secondary match if Admission No isn't known.
Registered_Phone | Yes | Crucial for confirming the user is who they say they are.

### A. User registration form 
1) First Name 
2) Last Name 
3) Mail ID 
4) Phone number 
5) Role (Drop down) - Handle collisions for teachers that are parents-> TBD 
6) Admission Number ? Probably not needed as this can be figured out from the csv data 

