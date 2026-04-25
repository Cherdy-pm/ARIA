# ARIA Technical Architecture

## 1. Design Philosophy
ARIA is built on an **Event-Driven Architecture (EDA)**. The system does not just "wait"; it actively polls and calculates time-deltas to trigger actions at the precise moment they are needed.

## 2. Technical Layers

### A. The Ingestion Layer (Polling Logic)
- **n8n Interval Trigger:** Set to poll the Airtable 'Appointments' table every 60 minutes.
- **Filtering Node:** Uses a JavaScript function to compare `Current_Time` vs `Appointment_Time`.
  - *Logic:* If `Diff == 24hrs`, trigger Reminder Sequence A.
  - *Logic:* If `Diff == 1hr`, trigger Reminder Sequence B.

### B. The Orchestration Layer (Decision Tree)
- **Router Node:** Evaluates the patient’s preferred contact method (stored in their profile).
- **Wait/Resume Logic:** If a reminder is sent, the workflow enters a "Wait for Webhook" state, listening for the patient's button click on Telegram.
- **Update Logic:** Upon receiving a confirmation, an HTTP Request is sent to Airtable to update the `Status` field to `Confirmed`.

### C. The Error & Recovery Layer
- **No-Response Handling:** If no confirmation is received within 4 hours, ARIA triggers a secondary notification to the Clinic Admin.
- **Conflict Resolution:** If a "Reschedule" is requested, ARIA pulls available slots from the 'Clinic_Availability' table and presents them to the user.

## 3. Data Schema
- **Patients:** `ID`, `Name`, `Phone`, `Preferred_Channel`.
- **Appointments:** `Appt_ID`, `Patient_ID`, `Timestamp`, `Status` (Pending/Confirmed/Cancelled).
