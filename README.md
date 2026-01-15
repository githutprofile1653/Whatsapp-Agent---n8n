# Doctor Workflow – WhatsApp AI Assistant Documentation

## 1. Overview
<img width="938" height="368" alt="doctor whatsapp workflow" src="https://github.com/user-attachments/assets/e6345e34-813a-4eae-beab-0e3df96044b1" />


The **Doctor Workflow** is an automated **WhatsApp-based healthcare assistant** built using **n8n**, **OpenAI (LangChain Agent)**, and **Airtable**.
It enables patients to interact with a doctor’s practice through WhatsApp for:

* Viewing previous diagnosis and prescriptions
* Asking general health-related questions
* Raising follow-up queries or tickets for doctor review
* Getting guided escalation for critical or unsafe medical concerns

The workflow ensures **safe medical AI usage**, **data-driven responses**, and **human doctor escalation when required**.

---

## 2. Architecture Summary

**Core Technologies Used**

* **n8n** – Workflow orchestration
* **WhatsApp Cloud API** – Patient communication channel
* **OpenAI (GPT-4.1-mini)** – Natural language understanding & response generation
* **LangChain Agent** – Tool-based reasoning & decision-making
* **Airtable** – Patient records & ticket management
* **Memory Buffer** – Conversation continuity per patient

---

## 3. High-Level Workflow Flow

1. Patient sends a WhatsApp message
2. WhatsApp Trigger receives the message
3. AI Agent processes intent and context
4. Agent decides whether to:

   * Query patient records
   * Answer general health FAQ
   * Create a ticket for doctor follow-up
5. Response is sent back to the patient on WhatsApp
6. Conversation state is preserved using memory

---

## 4. Node-by-Node Breakdown

---

### 4.1 WhatsApp Trigger Node

**Node Name:** WhatsApp Trigger
**Type:** `n8n-nodes-base.whatsAppTrigger`

#### Purpose

* Acts as the **entry point** of the workflow
* Listens for **incoming WhatsApp messages**
* Captures patient message, metadata, and contact details

#### Key Data Captured

* Patient WhatsApp ID (`wa_id`)
* Message body text
* Patient name (WhatsApp profile)
* Phone number metadata (used as Patient ID)

---

### 4.2 AI Agent Node (LangChain Agent)

**Node Name:** AI Agent
**Type:** `@n8n/n8n-nodes-langchain.agent`

#### Purpose

This is the **core brain** of the workflow.
It performs **intent detection**, **tool selection**, and **response generation**.

#### Input

* Patient’s WhatsApp message:

```
$json.messages[0].text.body
```

#### System Persona

* Identity: *Helper AI – Digital Assistant for Dr. R Mehta*
* Tone: Polite, empathetic, professional, concise
* Role: Hospital helpdesk-style assistant

#### Supported Intents

1. **Past Diagnosis / Prescription Request**
2. **General Health Questions**
3. **Critical or Unsafe Medical Queries**
4. **Doctor Follow-up / Ticket Creation**

---

### 4.3 Conversational Logic (AI Reasoning Rules)

#### 1. Past Diagnosis / Prescription

* AI searches **Patients table in Airtable**
* Lookup key: Patient phone number (without country code)
* Returns:

  * Last diagnosis summary
  * Prescribed medicines
* Adds disclaimer:

  > “Based on your last hospital record. For new or urgent symptoms, please consult your doctor.”

---

#### 2. General Health FAQs

Examples:

* Diet for diabetes
* Headache remedies
* Vaccination schedules

Rules:

* Short, safe, verified medical advice
* No diagnosis
* Mandatory disclaimer:

  > “This is general guidance only. Please consult your doctor for personalized advice.”

---

#### 3. Critical / Unsafe Queries

Examples:

* Severe symptoms
* Emergency situations
* Complex medication combinations

Response:

* AI refuses to give advice
* Encourages doctor consultation
* Offers help scheduling or contacting doctor

---

#### 4. Doctor Follow-up (Ticket Creation)

Triggered when the patient wants doctor intervention.

AI collects:

* Full Name
* Age
* Mobile Number (if different)
* Concern description

Then:

* Creates a ticket in Airtable
* Confirms submission to patient

---

### 4.4 OpenAI Chat Model Node

**Node Name:** OpenAI Chat Model
**Model:** `gpt-4.1-mini`

#### Purpose

* Provides language understanding and response generation
* Connected to the AI Agent as the **LLM backend**

#### Why GPT-4.1-mini?

* Cost-efficient
* Fast response time
* Strong reasoning for conversational workflows

---

### 4.5 Airtable – Patient Table Tool

**Node Name:** Patient Table
**Operation:** Search

#### Purpose

* Fetch patient medical records
* Used when patient asks about:

  * Diagnosis
  * Prescriptions
  * Medical history

#### Data Source

* Airtable Base: **Patient Records**
* Table: **Patients**

---

### 4.6 Airtable – Create Ticket Tool

**Node Name:** Create a Ticket
**Operation:** Create

#### Purpose

* Stores patient follow-up requests for doctors

#### Fields Stored

* Name
* QueryDate
* Concern
* AssignedDoctor
* Status → `Open`

#### Workflow Behavior

* TicketID derived from patient phone number
* Used for tracking and follow-up

---

### 4.7 Simple Memory Node

**Node Name:** Simple Memory
**Type:** `memoryBufferWindow`

#### Purpose

* Maintains conversation context per patient
* Prevents loss of flow during multi-turn chats

#### Session Key

```
metadata.phone_number_id
```

This ensures:

* Each patient has an isolated conversation memory
* No cross-user data leakage

---

### 4.8 Send Message Node

**Node Name:** Send message
**Type:** `n8n-nodes-base.whatsApp`

#### Purpose

* Sends AI-generated replies back to the patient
* Uses:

```
$json.output
```

---

## 5. Data Security & Safety Considerations

* No self-diagnosis or emergency treatment advice
* AI responses limited to:

  * Airtable data
  * General medical knowledge
* Mandatory doctor escalation rules
* No hallucinated patient data
* Plain-text responses only (no markdown)

---

## 6. Use Cases

* Hospital WhatsApp helpdesk
* Doctor clinic automation
* Telemedicine pre-screening
* Patient engagement systems
* Medical CRM integration

---

## 7. Future Enhancements (Optional)

* Appointment scheduling integration
* Prescription PDF delivery
* Doctor notification via email/SMS
* Multilingual patient support
* Medical report uploads

---

## 8. Conclusion

The **Doctor Workflow** demonstrates a **production-ready AI healthcare assistant** that balances:

* Automation
* Patient safety
* Data accuracy
* Human doctor involvement

It is a strong example of **responsible AI in healthcare communication** using no-code/low-code tools.

---

If you want, I can:

* Convert this into **Markdown**
* Create a **system architecture diagram**
* Rewrite it for **academic project submission**
* Simplify it for **client presentation**
