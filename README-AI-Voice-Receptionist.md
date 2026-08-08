<div align="center">

# 📞 AI Voice Receptionist for Appointment Booking

### A real-world **AI Voice Agent** built with **Retell AI + n8n** for 24/7 dental appointment booking

**Answers callers · Collects patient details · Validates input · Checks live calendar availability · Prevents booking conflicts · Creates appointments · Logs bookings · Sends confirmation emails**

<br>

<img src="https://img.shields.io/badge/Retell_AI-Voice_AI-6C63FF?style=for-the-badge" alt="Retell AI"/>
<img src="https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n"/>
<img src="https://img.shields.io/badge/Google_Calendar-Live_Availability-4285F4?style=for-the-badge&logo=googlecalendar&logoColor=white" alt="Google Calendar"/>
<img src="https://img.shields.io/badge/Google_Sheets-Booking_Log-34A853?style=for-the-badge&logo=googlesheets&logoColor=white" alt="Google Sheets"/>
<img src="https://img.shields.io/badge/Gmail-Confirmation_Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Gmail"/>

<br><br>

> **Not just a voice bot.**  
> This project connects a conversational AI Voice Agent to a real appointment-booking workflow with validation, live availability checks, scheduling, logging, and automated confirmation.

</div>

---

## 👋 Meet the AI Voice Receptionist

Every missed call can become a missed appointment.

This project demonstrates how an **AI Voice Receptionist** can support a dental clinic by handling appointment-booking calls without sending patients to voicemail or requiring a human receptionist to manually check every requested time slot.

The caller speaks naturally with the **Retell AI Voice Agent**. When the conversation reaches the booking stage, Retell sends the patient details and requested appointment time to an **n8n workflow**.

The workflow then performs the operational work behind the conversation:

- validates required patient information,
- validates the email address,
- converts the requested time into a structured appointment window,
- checks **Google Calendar** for real-time availability,
- blocks conflicting bookings,
- creates the appointment when the slot is free,
- records the booking in **Google Sheets**,
- sends a formatted confirmation email through **Gmail**,
- and returns a structured result back to the Voice Agent.

<div align="center">

### Voice AI handles the conversation.  
### n8n handles the business process.

</div>

---

## 💼 The Business Problem

A traditional appointment call often looks like this:

```text
Patient calls
    ↓
Receptionist answers
    ↓
Collect patient details
    ↓
Check calendar
    ↓
Confirm / reject requested time
    ↓
Create appointment
    ↓
Record booking
    ↓
Send confirmation
```

This works — until the clinic is busy, the call arrives after hours, or staff are already helping someone else.

The result can be:

```text
Missed call
    ↓
Voicemail
    ↓
Slow callback
    ↓
Patient calls another clinic
```

The goal of this project is to automate the repetitive parts of that process while keeping appointment data connected to the clinic's existing tools.

---

## 🚀 What the AI Voice Receptionist Can Do

| Capability | Implementation |
|---|---|
| 📞 Handle appointment-booking conversations | Retell AI Voice Agent |
| 👤 Capture patient name | Retell → n8n webhook |
| 📧 Capture & validate patient email | n8n validation logic |
| 🕐 Capture requested appointment time | Retell → n8n |
| ✅ Reject missing / invalid booking data | Required-field validation |
| 📆 Check live calendar availability | Google Calendar |
| 🚫 Prevent double booking | Availability check before event creation |
| 📅 Create a 30-minute appointment | Google Calendar |
| 🧾 Generate a booking ID | n8n |
| 📊 Log confirmed bookings | Google Sheets |
| ✉️ Send confirmation email | Gmail |
| 🔁 Return booking result to the Voice Agent | Webhook response |
| 🗣️ Let the caller choose another time if unavailable | Structured `slot_unavailable` response |

---

## 🏗️ End-to-End Architecture

```text
                       📞 PATIENT CALL
                              │
                              ▼
                    ┌───────────────────┐
                    │    RETELL AI      │
                    │   VOICE AGENT     │
                    │                   │
                    │ Natural voice     │
                    │ conversation      │
                    └─────────┬─────────┘
                              │
                   Booking tool / webhook
                              │
                              ▼
                    ┌───────────────────┐
                    │       n8n         │
                    │     WEBHOOK       │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ NORMALIZE BOOKING │
                    │      DATA         │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ VALIDATE REQUIRED │
                    │      FIELDS       │
                    └─────────┬─────────┘
                              │
                     Invalid? ├────────────► Return error to Voice Agent
                              │
                              ▼
                    ┌───────────────────┐
                    │  VALIDATE EMAIL   │
                    └─────────┬─────────┘
                              │
                     Invalid? ├────────────► Ask caller to repeat email
                              │
                              ▼
                    ┌───────────────────┐
                    │ GOOGLE CALENDAR   │
                    │ CHECK AVAILABILITY│
                    └─────────┬─────────┘
                              │
                ┌─────────────┴─────────────┐
                │                           │
            AVAILABLE                  UNAVAILABLE
                │                           │
                ▼                           ▼
      ┌───────────────────┐       ┌───────────────────┐
      │ CREATE APPOINTMENT│       │ RETURN CONFLICT   │
      │  CALENDAR EVENT   │       │ TO VOICE AGENT    │
      └─────────┬─────────┘       └───────────────────┘
                │
                ▼
      ┌───────────────────┐
      │ GOOGLE SHEETS LOG │
      └─────────┬─────────┘
                │
                ▼
      ┌───────────────────┐
      │ GMAIL CONFIRMATION│
      └─────────┬─────────┘
                │
                ▼
      ┌───────────────────┐
      │ SUCCESS RESPONSE  │
      │  TO RETELL AI     │
      └───────────────────┘
```

---

## 🔄 Real Workflow Logic

### 01 — Receive Booking Data from Retell AI

The workflow begins with an **n8n Webhook**.

Retell AI can call this endpoint when it has collected the information needed to attempt a booking.

Expected input includes:

```json
{
  "name": "John Smith",
  "email": "john@example.com",
  "time": "2026-08-15 14:30"
}
```

The workflow then converts those values into a structured booking object.

---

### 02 — Normalize Appointment Data

The workflow prepares:

```text
patient_name
patient_email
appointment_time_raw
appointment_start
appointment_end
created_at
booking_id
source
```

Appointments are configured as **30-minute bookings**.

The current workflow uses:

```text
Timezone: Asia/Karachi
```

> [!IMPORTANT]
> Change the timezone to match the clinic or business using the workflow.

A unique booking reference is generated automatically:

```text
BSD-YYYYMMDDHHMMSS-XXXXX
```

This helps correlate the calendar event, spreadsheet row, email confirmation, and webhook response.

---

### 03 — Validate Required Fields

Before touching the calendar, the workflow checks that the booking contains:

```text
✓ Patient name
✓ Patient email
✓ Requested date/time
✓ Valid appointment timestamp
✓ Appointment time is in the future
```

If any required value is missing or invalid, the workflow stops and returns:

```json
{
  "success": false,
  "reason": "invalid_input"
}
```

The Voice Agent can then continue the conversation and collect the missing information.

---

### 04 — Validate the Email Address

The workflow separately checks that the patient's email has a valid format.

If validation fails, it returns:

```json
{
  "success": false,
  "reason": "invalid_email",
  "message": "The email address is invalid. Please ask the caller to repeat or spell the email address."
}
```

This allows the Voice Agent to recover during the live call instead of creating a booking with unusable contact information.

---

### 05 — Check Real-Time Google Calendar Availability

Once the booking data is valid, n8n checks **Google Calendar** for the exact requested appointment window.

```text
Requested Start
      +
30 Minutes
      =
Requested Appointment Window
```

The workflow asks Google Calendar whether that period is available.

---

## 🚫 Booking Conflict Prevention

If the requested slot is already occupied:

```text
Requested Time
      │
      ▼
Google Calendar
      │
      ▼
❌ NOT AVAILABLE
      │
      ▼
No event created
      │
      ▼
Return result to Retell AI
```

The workflow responds with:

```json
{
  "success": false,
  "available": false,
  "reason": "slot_unavailable",
  "message": "The requested appointment time is unavailable. Please ask the caller for another date or time."
}
```

This is a key feature of the system.

The AI Voice Agent does not blindly promise an appointment.

It checks the real calendar first.

The conversation can then continue and the caller can provide another preferred time.

---

## ✅ Successful Booking Path

When the requested slot is free:

```text
Availability Check
      │
      ▼
✅ AVAILABLE
      │
      ▼
Create Calendar Event
      │
      ▼
Log Booking
      │
      ▼
Send Confirmation Email
      │
      ▼
Return Success to Retell AI
```

---

## 📅 Google Calendar Appointment Creation

The workflow creates the appointment directly in Google Calendar.

The event contains:

```text
Patient Name
Patient Email
Booking ID
Booking Source
Appointment Start
Appointment End
```

The event title follows the pattern:

```text
Patient Name | Dental Appointment
```

This means the clinic receives a normal calendar event that staff can view and manage from Google Calendar.

---

## 📊 Google Sheets Booking Log

After calendar creation, the workflow appends the confirmed appointment to Google Sheets.

The current implementation records:

| Field | Purpose |
|---|---|
| Booking ID | Unique booking reference |
| Patient Name | Caller / patient |
| Patient Email | Contact information |
| Appointment Start | Scheduled start time |
| Appointment End | Scheduled end time |
| Status | Booking status |
| Calendar Event ID | Google Calendar reference |
| Calendar Event Link | Direct calendar event link |
| Source | Retell AI Voice Agent |
| Created At | Booking timestamp |
| Error Message | Reserved for error information |

This provides a simple operational booking ledger outside the calendar.

---

## ✉️ Automatic Patient Confirmation Email

Once the appointment is created, the workflow sends a branded HTML confirmation email through Gmail.

The confirmation includes:

```text
✓ Patient name
✓ Appointment date
✓ Appointment time
✓ 30-minute duration
✓ Booking ID
✓ Link to the calendar appointment
```

The email is sent only after the appointment has been created.

This keeps the voice conversation, calendar, booking log, and patient communication synchronized.

---

## 🔁 Structured Responses Back to Retell AI

The workflow does not simply end after execution.

It returns structured JSON so the Voice Agent knows what happened.

### Booking successful

```json
{
  "success": true,
  "available": true,
  "reason": "appointment_booked",
  "message": "The dental appointment has been booked successfully."
}
```

### Requested slot unavailable

```json
{
  "success": false,
  "available": false,
  "reason": "slot_unavailable"
}
```

### Invalid email

```json
{
  "success": false,
  "reason": "invalid_email"
}
```

### Missing / invalid booking data

```json
{
  "success": false,
  "reason": "invalid_input"
}
```

These machine-readable results allow Retell AI to continue the live conversation appropriately.

---

## 🧠 Voice Intelligence vs Workflow Logic

The project intentionally separates the conversational layer from the operational layer.

<table>
<tr>
<th>Retell AI Voice Agent</th>
<th>n8n Workflow</th>
</tr>
<tr>
<td>Talks naturally with the caller</td>
<td>Validates structured booking data</td>
</tr>
<tr>
<td>Collects name, email, and desired time</td>
<td>Converts time into appointment timestamps</td>
</tr>
<tr>
<td>Explains booking results conversationally</td>
<td>Checks real Google Calendar availability</td>
</tr>
<tr>
<td>Asks for another time after a conflict</td>
<td>Prevents calendar conflicts</td>
</tr>
<tr>
<td>Continues the live conversation</td>
<td>Creates and records confirmed bookings</td>
</tr>
</table>

<div align="center">

### Retell AI handles conversation ambiguity.  
### n8n handles deterministic business execution.

</div>

---

## 🛡️ Guardrails Built Into the Booking Flow

This workflow includes several controls before an appointment is considered confirmed:

```text
01. Required fields must exist
02. Appointment timestamp must be valid
03. Appointment must be in the future
04. Email format must be valid
05. Requested calendar window must be available
06. Calendar event must be created before success is returned
```

The design avoids the most dangerous failure mode for a booking Voice Agent:

> **telling a caller that an appointment is booked when the calendar was never updated.**

---

## 🧪 Example Test Scenarios

### Test 01 — Valid Available Slot

```text
Patient provides valid name
        +
Valid email
        +
Future appointment time
        +
Calendar slot available
        =
✅ BOOKING CREATED
```

Expected:

```text
Calendar event created
Google Sheets row added
Confirmation email sent
Retell receives success
```

### Test 02 — Slot Already Booked

```text
Requested time
        +
Existing calendar event
        =
❌ SLOT UNAVAILABLE
```

Expected:

```text
No duplicate appointment
No false confirmation
Retell receives slot_unavailable
Caller can choose another time
```

### Test 03 — Invalid Email

```text
Patient Email:
johnexample.com
```

Expected:

```text
❌ invalid_email
```

The Voice Agent can ask the caller to repeat or spell the email address.

### Test 04 — Missing Patient Information

Missing name, email, time, invalid timestamp, or past appointment:

```text
❌ invalid_input
```

No calendar event is created.

---

## 🛠️ Technology Stack

| Technology | Responsibility |
|---|---|
| **Retell AI** | Conversational AI Voice Agent |
| **n8n** | Workflow orchestration and booking logic |
| **Google Calendar** | Live availability and appointment creation |
| **Google Sheets** | Appointment logging |
| **Gmail** | Automated patient confirmation |
| **Luxon / n8n DateTime** | Date and timezone processing |
| **Webhook / JSON** | Communication between Retell AI and n8n |

---

## 💡 Why This Is More Than a Voice Bot

A basic Voice AI demo can answer questions.

This project connects the conversation to real operational tools.

| Capability | Basic Voice Bot | This AI Voice Receptionist |
|---|:---:|:---:|
| Talk with callers | ✅ | ✅ |
| Collect booking information | ⚠️ | ✅ |
| Validate patient input | ❌ | ✅ |
| Check live calendar availability | ❌ | ✅ |
| Detect booking conflicts | ❌ | ✅ |
| Create calendar appointments | ❌ | ✅ |
| Log confirmed bookings | ❌ | ✅ |
| Send confirmation emails | ❌ | ✅ |
| Return structured booking state | ❌ | ✅ |
| Continue conversation after failure | ⚠️ | ✅ |

> The value is not simply that the AI can talk.  
> The value is that the conversation can trigger a controlled business process.

---

## 🚀 Setup & Quick Start

### 1. Import the n8n Workflow

Import the workflow JSON included in this repository:

```text
n8n → Workflows → Import from File
```

### 2. Connect the Required Services

Configure your own credentials for:

```text
Retell AI
Google Calendar
Google Sheets
Gmail
```

> [!IMPORTANT]
> Do not publish API keys, OAuth tokens, private webhook URLs, calendar IDs, spreadsheet IDs, or customer data.

### 3. Configure Your Calendar

Update both Google Calendar nodes to use the appointment calendar for your business.

Also update the workflow timezone if your business does not operate in:

```text
Asia/Karachi
```

### 4. Create the Appointments Sheet

Create a Google Sheet with these columns:

```text
Booking ID
Patient Name
Patient Email
Appointment Start
Appointment End
Status
Calendar Event ID
Calendar Event Link
Source
Created At
Error Message
```

Connect the **Append row in sheet** node to that spreadsheet.

### 5. Connect Gmail

Configure the Gmail node with the account that should send appointment confirmations.

Update the clinic/business branding inside the HTML email template if needed.

### 6. Connect Retell AI

Configure a Retell AI function / custom tool that calls the n8n production webhook.

The tool should send:

```json
{
  "name": "Patient Name",
  "email": "patient@example.com",
  "time": "yyyy-MM-dd HH:mm"
}
```

Your Retell AI prompt should handle the returned `reason` values so it can respond naturally during the call.

For example:

```text
appointment_booked
→ Confirm the booking to the caller

slot_unavailable
→ Ask for another date or time

invalid_email
→ Ask the caller to repeat or spell the email

invalid_input
→ Collect the missing or corrected information
```

---

## 🔐 Security Before Publishing or Deploying

Before using the workflow publicly or committing it to GitHub:

- remove credential references from exported workflow files where appropriate,
- replace private webhook paths,
- remove Google Calendar IDs,
- remove Google Sheet IDs and URLs,
- remove n8n instance metadata,
- remove real patient information,
- never commit OAuth tokens or API keys,
- use a dedicated production calendar and inbox,
- review how patient data should be stored under the regulations that apply to your business.

> [!CAUTION]
> This project demonstrates the automation architecture. Healthcare organizations may have additional privacy, consent, security, and regulatory requirements that must be addressed before handling real patient data.

---

## 📁 Suggested Repository Structure

```text
n8n-ai-voice-agent-receptionist/
│
├── README.md
├── workflow/
│   └── ai-voice-receptionist.json
├── sample-data/
│   └── appointments-template.csv
├── docs/
│   └── setup-notes.md
├── .gitignore
└── LICENSE
```

---

## 🧩 Extending the Project

The current implementation focuses on **appointment booking**, but the same architecture can be expanded for:

```text
Appointment rescheduling
Appointment cancellation
Appointment reminders
Missed-call follow-up
Lead qualification
Patient FAQs
Clinic hours and location questions
Insurance pre-screening
CRM synchronization
SMS confirmations
Human call transfer
Multi-location scheduling
```

These are possible extensions — they are **not all implemented in the current workflow**.

The reusable pattern remains:

```text
VOICE CONVERSATION
      ↓
STRUCTURED REQUEST
      ↓
BUSINESS VALIDATION
      ↓
REAL SYSTEM ACTION
      ↓
STRUCTURED RESULT
      ↓
VOICE RESPONSE
```

---

## 👨‍💻 About the Developer

Hi, I'm **Ikram** — an Automation Specialist building practical **AI Agents, AI Employees, Voice AI systems, and workflow automations** for real business processes.

My focus is not just on making AI generate responses.

I build systems that connect AI to APIs, business rules, calendars, CRMs, databases, communication tools, and human workflows.

This AI Voice Receptionist is part of a growing portfolio of real-world automation projects built around that idea.

<div align="center">

### AI should not just talk.

### It should help the business get work done.

</div>

---

## ⭐ Found This Project Useful?

If this repository helps you understand how **Retell AI, n8n, Voice AI, and appointment automation** can work together:

- ⭐ Star the repository
- 🍴 Fork it and adapt the workflow
- 🤝 Connect if you're building AI Agents or Voice AI systems

---

## 📬 Let's Connect

[![GitHub](https://img.shields.io/badge/GitHub-Ikram--Ul--Hassan-181717?style=for-the-badge&logo=github)](https://github.com/Ikram-Ul-Hassan/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Ikram_Ul_Hassan-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/ikram-ul-hassan/)
[![Email](https://img.shields.io/badge/Email-ikramautomations%40gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:ikramautomations@gmail.com)

---

### 📄 License

This project can be published under the **MIT License** if that is the license you choose for the repository.

---

<div align="center">

## 📞 n8n AI Voice Agent Receptionist

**Retell AI • n8n • Voice AI • Google Calendar • Google Sheets • Gmail • Appointment Automation**

<br>

**Built to answer. Designed to validate. Connected to real business systems.**

</div>
