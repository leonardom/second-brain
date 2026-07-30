# 🚀 SaaS Concept

## “ComplyTrack IoM” (working name)
A lightweight compliance + client tracking platform for:
- Corporate Service Providers (CSPs)
- Accountants
- Small fiduciary firms on the Isle of Man


## 🎯 Core Problem You’re Solving
These firms currently:

- Track compliance in Excel spreadsheets
- Use email/calendar reminders
- Store documents in shared drives
- Struggle with audit trails & deadlines

👉 This creates real risk: missed filings, weak AML evidence, audit failures

## ✅ MVP (What You Build First)
Goal: Sell ASAP, not overbuild

### 1. 👥 Client Management

- Add/edit clients (individuals or companies)
- Store:
  - Name, type, jurisdiction
  - Risk level (Low / Medium / High)
  - Assigned staff member


### 2. 📋 Compliance Checklist System

Each client has:

- Custom checklist (KYC items, due diligence)
- Status tracking:
  - ✅ Complete
  - ⏳ Pending
  - ⚠️ Overdue

**Example checklist:**
- Proof of ID
- Proof of Address
- Source of Funds
- Risk Assessment


### 3. ⏰ Deadline Tracking

- Compliance events per client:
  - Annual review
  - Filing deadlines
- Dashboard:
  - “Due this week”
  - “Overdue”

👉 This alone is extremely valuable

### 4. 📂 Document Storage + Audit Log

- Upload documents (PDF, etc.)
- Automatically log:
  - Upload date
  - User
  - Changes

### 5. 🔔 Alerts / Notifications

Email reminders:
- Upcoming deadlines
- Missing documents


### 6. 📊 Simple Dashboard

- Clients by risk level
- Overdue compliance items
- Upcoming deadlines

### 🧠 What Makes This Irresistible

The key is positioning, not features:
> “Built specifically for Isle of Man compliance workflows”

Add subtle localisation:
- “Annual AML review”
- “Beneficial ownership tracking”
- “GSC-ready audit logs”


## 🏗️ Suggested Tech Stack (fast to build)
Since you’re a developer, go pragmatic:

#### Backend

- Node.js (NestJS or Express)
- PostgreSQL

#### Frontend

- React (Next.js)
- Tailwind UI

#### Auth

- Clerk / Auth0 / custom JWT

#### Storage

- AWS S3 (documents)

#### Email

- Resend / SendGrid

#### Hosting

- Vercel (frontend) + Railway/Fly.io (backend)


### 🗃️ Basic Database Design
users
```
id
name
email
role
```

clients
```
id
name
type (individual/company)
risk_level
assigned_user_id
created_at
```

compliance_items
```
id
client_id
name
status
due_date
completed_at
```

documents
```
id
client_id
file_url
document_type
uploaded_by
created_at
expiry_date
```

audit_logs
```
id
action
user_id
client_id
timestamp
```

Types of documents:
- passport
- proof_of_address
- certificate_of_incorporation
- memorandum_articles
- source_of_funds
- bank_statement
- other

### 💰 Pricing Strategy (IMPORTANT)
Keep it simple:

#### Starter
- £39/month
- Up to 50 clients

#### Professional

- £79/month
- Up to 200 clients

#### Premium

- £149/month
- Unlimited + reporting

👉 Isle of Man firms can afford this easily

### 🧲 How You Get Customers (THIS is the real win)

You don’t need ads. Do this:
#### 1. Direct outreach (super effective locally)
  - Find 20–30 CSPs / small accountants
  - Email or LinkedIn:
    Example:
    > “Hi — I’m building a simple compliance tracking tool specifically for Isle of Man firms currently using spreadsheets. Would you be open to a quick demo and giving feedback?”

##### 2. Sell BEFORE fully building
Offer:
- Early access at £20/month
- In exchange for feedback


#### 3. Demo-driven sales
Show them:
- Add client → assign checklist → see deadlines

That’s enough to sell.

### ⚡ Quick 30-Day Build Plan
Week 1
- Auth
- Client CRUD
- Basic UI

Week 2
- Compliance checklist system
- Deadlines + dashboard

Week 3
- Document upload
- Email reminders

Week 4
- Polish + deploy
- Start selling


### 🔥 Expansion (after first customers)
Once you get traction, add:
- ✅ Risk scoring automation
- ✅ CSV import from Excel
- ✅ Role-based access (teams)
- ✅ Reporting export (PDF for auditors)
- ✅ API integrations


### ⚠️ Avoid These Mistakes
- Don’t overbuild compliance rules engine
- Don’t try to compete with enterprise systems
- Don’t delay talking to real users


### 💡 Reality Check (why this works)
This is a “boring SaaS” = GOOD:
- Recurring revenue ✅
- Low churn ✅
- High trust ✅
- Local niche moat ✅

## 🧠 Product Mental Model (keep this simple)
Your app is essentially:
> Clients → Compliance Requirements → Deadlines → Evidence (documents)

Everything revolves around those 4 things.

### 🖥️ Core App Structure
```
[ Dashboard ]
[ Clients ]
    └── Client Profile
          ├── Overview
          ├── Compliance Checklist
          ├── Documents
          ├── Timeline / Audit Log
[ Tasks & Deadlines ]
[ Reports ]
[ Settings ]
```

### 🧭 FULL USER FLOW (End-to-End)
#### 🟢 FLOW 1: First-time user (onboarding)
##### Screen: “Welcome Setup”
```
---------------------------------
 Welcome to ComplyTrack IoM
---------------------------------
[ Create your workspace ]

Company Name: [________]
Your Role:    [Director ▼]

[ Continue ]
```
→ Next:

##### Screen: “Quick Setup Wizard”
```
Step 1: Add your first client
[ + Add Client ]

Step 2: Choose checklist template
( ) Standard AML/KYC
( ) Custom

Step 3: Invite team (optional)
[ email input ]

[ Finish Setup ]
```

👉 Goal: get them to first value in 2–3 minutes

#### 🟠 FLOW 2: Dashboard (daily view)
##### Screen: Dashboard
```
---------------------------------
 Good morning, Leo 👋
---------------------------------

⚠️ Overdue Items (5)
 - ABC Ltd → Annual Review (3 days overdue)
 - John Smith → Missing Proof of Address

📅 Due This Week (8)
 - XYZ Ltd → KYC refresh
 - Client A → Filing deadline

📊 Snapshot:
 Clients: 42
 High Risk: 6
 Pending Items: 18

[ View All Clients ]  [ Add Client ]
```

👉 This is what they check every morning

#### 🔵 FLOW 3: Add a Client
##### Screen: Add Client
```
---------------------------------
 Add New Client
---------------------------------

Type:
( ) Individual
( ) Company

Name: [________]

Risk Level:
( ) Low
( ) Medium
( ) High

Assigned To:
[ Leo ▼ ]

[ Create Client ]
```
→ After submit → goes to Client Profile

#### 🟣 FLOW 4: Client Profile (CORE SCREEN)
This is your most important screen.
##### Screen: Client Profile → Overview tab
```
---------------------------------
 ABC Ltd
---------------------------------
Risk: High 🔴
Assigned: Leo
Created: Jan 2026

Next Review: 12 Jun 2026

---------------------------------
 QUICK ACTIONS
---------------------------------
[ + Add Compliance Item ]
[ Upload Document ]
[ Complete Review ]
```

#### Tab 1: ✅ Compliance Checklist
```
---------------------------------
 Compliance Checklist
---------------------------------

[ + Add Item ]

✔ Proof of ID               ✅ Complete
⚠ Proof of Address         ⏳ Pending
⚠ Source of Funds          ⏳ Overdue
✔ Risk Assessment          ✅ Complete

---------------------------------
 Filters: [All | Pending | Overdue]
```

👉 Clicking item opens:
Modal: Compliance Item Detail
```
Name: Proof of Address
Status: Pending
Due Date: [ 20 May 2026 ]

Notes:
[______________________]

[ Upload Document ]
[ Mark Complete ]
```

#### Tab 2: 📂 Documents
```
---------------------------------
 Documents
---------------------------------

[ Upload File ]

- passport.pdf      (Uploaded May 1)
- utility_bill.pdf  (Uploaded May 3)

[ Download ] [ Delete ]
```

#### Tab 3: 🕒 Audit Log / Timeline
```
---------------------------------
 Activity
---------------------------------

May 5 → Leo uploaded passport.pdf
May 4 → Compliance item created
May 1 → Client created
```
👉 This is HUGE for audits

#### 🟡 FLOW 5: Deadlines & Tasks View
##### Screen: Tasks & Deadlines
```
---------------------------------
 Tasks & Deadlines
---------------------------------

[ Filter: All | Overdue | Upcoming ]

⚠ Overdue
- ABC Ltd → Source of Funds (3 days)

📅 Upcoming
- XYZ Ltd → Annual Review (2 days)
- John Smith → KYC Check (5 days)
```

👉 Optional:

Calendar view toggle (later feature)


#### 🔴 FLOW 6: Reports (simple first)
##### Screen: Reports
```
---------------------------------
 Reports
---------------------------------

[ Generate Compliance Report ]

Client: [ ABC Ltd ▼ ]

Options:
☑ Include checklist
☑ Include documents list
☑ Include audit history

[ Generate PDF ]
```
👉 Outputs something they can show regulators

#### ⚙️ FLOW 7: Settings
##### Screen: Settings
```
---------------------------------
 Settings
---------------------------------

Company Info
Users & Roles
Checklist Templates
Email Notifications
```

#### 🧩 UX PATTERNS (important)
##### ✅ Use colour meaningfully:
- 🔴 Red = Overdue / high risk
- 🟠 Orange = Pending
- 🟢 Green = Complete


##### ✅ Keep it SIMPLE

Your target users:
- Are not super technical
- Want clarity, not complexity


##### 🧠 “Aha Moment” You Must Deliver

Within 5 minutes, the user should feel:

> “I can see exactly what’s overdue and what I need to fix today”

That’s what they’ll pay for.

##### 📱 Optional Mobile Experience (later)

Not needed day 1, but:
- View dashboard
- Get deadline alerts
- Quick document upload (photo)


##### 🚀 What You Actually Build First (UI Priority)

If you want fast results:

Must-have screens:
- ✅ Dashboard
- ✅ Client list
- ✅ Client profile (checklist + docs)
- ✅ Add client modal

👉 That alone is sellable

##### 🎯 Visual Layout (mental wireframe)

Think:
- Left sidebar navigation
- Main content panel
- Top header with actions
```
[Sidebar]       [ Main Content                 ]
Dashboard       -------------------------------
Clients         Header + Actions
Tasks           Cards / Table / Lists
Reports
Settings
```

##### 🧠 Final Insight
This isn’t about fancy UI.

It’s about:
- Removing Excel
- Reducing compliance anxiety
- Making audits easy

## Checklist examples

#### 🧩 1. Standard AML / KYC Checklist (Core One)
This will be your default template used 80% of the time.
##### ✅ “Individual Client – KYC”
```
Identity Verification
☐ Passport obtained
☐ Passport verified (certified copy)
☐ Selfie / liveness check (optional)

Address Verification
☐ Proof of address obtained (utility bill/bank statement)
☐ Address verified (within last 3 months)

Risk Assessment
☐ Risk level assigned (Low/Medium/High)
☐ PEP (Politically Exposed Person) check completed
☐ Sanctions screening completed

Source of Funds
☐ Source of funds declared
☐ Supporting evidence obtained
☐ Reasonableness reviewed

Ongoing Monitoring
☐ Annual review scheduled
☐ Trigger-based review rules set
```

#### 🏢 2. Company / Corporate Client Checklist
Much more complex → high value for your SaaS
##### ✅ “Corporate Client – Due Diligence”
```
Company Information
☐ Certificate of Incorporation obtained
☐ Memorandum & Articles collected
☐ Registered address verified

Directors
☐ Director list obtained
☐ ID verification for each director
☐ PEP & sanctions checks completed

Beneficial Ownership
☐ UBOs identified (>25%)
☐ Ownership structure mapped
☐ ID & address verified for UBOs

Business Activity
☐ Nature of business documented
☐ Expected transaction activity defined
☐ Risk profile assigned

Source of Wealth / Funds
☐ Source of wealth explanation documented
☐ Supporting documents obtained

Compliance Review
☐ Initial risk assessment completed
☐ Annual review scheduled
```

👉 This checklist alone can justify your SaaS pricing.

#### 🔁 3. Annual Review Checklist (VERY IMPORTANT)
This is where recurring usage comes from.
##### ✅ “AML Annual Review”
```
Client Information
☐ Details still accurate
☐ Contact info updated

Risk Review
☐ Risk rating reassessed
☐ Any changes in business activity

Screening
☐ PEP check re-run
☐ Sanctions screening updated

Documents
☐ ID still valid
☐ Proof of address within acceptable timeframe

Transactions
☐ Activity matches expected profile
☐ Any unusual activity flagged

Outcome
☐ Review approved
☐ Escalation required (if needed)
```

#### ⚠️ 4. Enhanced Due Diligence (High-Risk Clients)
👉 High-margin feature later
##### ✅ “High Risk / EDD”
```
Client Risk Factors
☐ High-risk jurisdiction identified
☐ Complex ownership structure

Additional Checks
☐ Additional identity verification
☐ Independent source verification
☐ Adverse media search completed

Wealth Analysis
☐ Detailed source of wealth established
☐ Supporting evidence verified

Approval
☐ Senior management approval obtained
☐ Compliance sign-off recorded
```

#### 🧾 5. Onboarding Checklist (Simplified UX version)
You might create a “friendly” version:
##### ✅ “Client Onboarding – Quick”
```
☐ Client created in system
☐ KYC checklist assigned
☐ Initial documents uploaded
☐ Risk level assigned
☐ Review date scheduled
```
👉 This is useful for less experienced staff

#### 🧠 How to Structure Checklists in Your SaaS
Instead of hardcoding, design them like:
##### Checklist Template Model
```
Template:
- Name
- Applies To (Individual / Company)
- List of Items

Item:
- Title
- Category (KYC, Risk, Documents…)
- Required (true/false)
- Default due days (e.g. 7 days)
```

#### 🎯 UX Idea (Very Important)
Show checklist visually as progress:
```
Completion: ███████░░░ 70%

✅ 14 Completed
⏳ 4 Pending
⚠ 2 Overdue
```

This gives users a sense of control instantly

#### 🚀 Advanced Feature (Later = HUGE value)
Smart Rules
Auto-assign checklist based on:
```
IF client_type = "Company"
→ Apply Corporate Checklist

IF risk_level = "High"
→ Add EDD Checklist

IF jurisdiction = "High Risk Country"
→ Require additional verification
```
👉 This is where your product becomes very powerful

#### 💡 Key Insight
You’re not just building:
> “a checklist tool”

You’re building:
> “A system that ensures nothing gets missed under regulatory pressure”

#### 🔥 What You Should Implement FIRST
Start with only 2 templates:
1. Individual KYC ✅
2. Corporate Client ✅

That’s enough to:
- Demo
- Sell
- Get feedback

### Checklist Templates

#### 🧠 Mental Model for the UI
Think of templates like:
> 📋 A structured form builder, but simpler than something like Typeform

Users should feel:
- “I can create a checklist in under 2 minutes”
- Not: “I’m configuring a system…”


#### 🧭 TEMPLATE BUILDER: FULL UI FLOW

##### 🟢 FLOW 1: Template List Page
###### Screen: “Checklist Templates”
```
---------------------------------
 Checklist Templates
---------------------------------

[ + Create Template ]

STANDARD TEMPLATES
- Individual KYC            (Default)
- Corporate Client          (Default)

CUSTOM TEMPLATES
- High Risk EDD             (Custom)
- Crypto Client Review      (Custom)

---------------------------------
[ Edit ] [ Duplicate ] [ Delete ]
```

👉 Important:
- Separate Default vs Custom
- Prevent accidental editing of system templates (or force duplication)


#### 🟠 FLOW 2: Create Template (Entry Point)
##### Screen: “Create Template”
```
---------------------------------
 Create New Template
---------------------------------

Template Name:
[ __________________ ]

Applies To:
( ) Individual
( ) Company
( ) Both

Category:
[ AML / KYC ▼ ]

[ Create Template ]

→ Redirect to builder
```

#### 🔵 FLOW 3: TEMPLATE BUILDER (CORE UX)
This is your money screen.
Layout:
```
[ Left Panel ]         [ Main Builder Area ]
-------------------------------------------
Template Info          Checklist Items
Settings               -------------------
                       [ + Add Item ]

                       ┌───────────────┐
                       │ Item Card     │
                       └───────────────┘

```

✅ Template Header
```
---------------------------------
 Individual KYC Template
---------------------------------

Applies To: Individual
Category: AML / KYC

[ Save ]   [ Publish ]   [ Back ]

```

🧩 Checklist Items (Card-Based UI)
Each item is a card:
```
┌──────────────────────────────┐
 Proof of Address
 Category: Address Verification
 Required: ✅
 Due: 7 days

 [ Edit ]  [ Delete ]  [ Drag ]
└──────────────────────────────┘
```

👉 Cards are:
- Drag-and-drop reorderable
- Easy to scan visually


##### ➕ Add Item Button
Click:
###### Modal: “Add Checklist Item”
```
---------------------------------
 Add New Item
---------------------------------

Title:
[ Proof of Address ]

Category:
[ Address ▼ ]

Required:
☑ Yes

Default Due Time:
[ 7 ] days after assignment

Notes / Guidance:
[ e.g. must be within last 3 months ]

[ Save ]
```

👉 Keep this SIMPLE—no enterprise complexity

#### 🟣 FLOW 4: Edit Item
Same modal but pre-filled:
```
Edit Item

Title: Proof of Address
Category: Address
Required: ✅
Due: 7 days
Notes: "Must be recent"

[ Save Changes ]
```

#### 🟡 FLOW 5: Categories (Lightweight)
Don’t over-engineer this—just enough structure:
Example Categories:
- Identity
- Address
- Risk
- Source of Funds
- Documents

Optional UI:
```
[ Manage Categories ]
 + Add Category
```

👉 Or just use a dropdown with defaults initially

#### 🔴 FLOW 6: Save vs Publish (Important Concept)
Introduce:
- Draft → still being edited
- Published → usable on clients

UI:
```
Status: Draft

[ Save Draft ]   [ Publish Template ]
```

👉 Prevents breaking active workflows

#### 🟤 FLOW 7: Assign Template to Client
Client Screen:
```
---------------------------------
 Assign Checklist
---------------------------------

Select Template:
[ Individual KYC ▼ ]

[ Assign ]
```

👉 Instantly creates checklist items for that client

#### ⚡ UX Enhancements (HIGH VALUE)

##### ✅ 1. Drag & Drop Ordering
Users can reorder like:
```
☰ Proof of ID
☰ Proof of Address
☰ Source of Funds
```

👉 Super intuitive

##### ✅ 2. “Duplicate Item” Button
Compliance lists often repeat:
```
[ Duplicate ] → edit slightly
```

##### ✅ 3. Prebuilt Templates (Huge selling point)
Include:
- Individual KYC ✅
- Corporate ✅
- Annual Review ✅


👉 This makes onboarding frictionless

##### ✅ 4. Smart Defaults
When adding item:
- Default due: 7 days
- Required: true

👉 Fewer decisions = faster UX

##### ✅ 5. Inline Editing (later upgrade)
Instead of modal:
Click title → edit directly

##### 🧠 Data Model (Flexible but simple)
Here’s how to structure it cleanly:

templates
```
id
name
applies_to (individual/company/both)
category
status (draft/published)
created_by
```

template_items
```
id
template_id
title
category
required (boolean)
due_days (int)
order_index
requires_document
document_type
notes
```

##### 🎯 What Makes This Builder GOOD
Most tools fail because they:
- Overcomplicate
- Add too many fields
- Assume expert users

Your advantage:
- ✅ Fast
- ✅ Visual
- ✅ Dead simple
- ✅ Preloaded with IoM-relevant templates

##### 🔥 MVP Scope (IMPORTANT)
Do NOT build everything above initially.

Build only:
- Create template ✅
- Add/edit/delete items ✅
- Assign template to client ✅

👉 Skip:
- Categories management
- Draft/publish system
- Drag & drop (can be added later)


##### 🧠 Real Insight
Users won’t spend hours building templates.
They will:
- Start with your defaults
- Slightly tweak them

👉 So your biggest feature isn’t the builder…
It’s the pre-built compliance knowledge

## Template to Client checklist

### 🧠 1. Core Concept
A template is a blueprint.
When you assign it to a client, you:
> ✅ COPY the template items into real, trackable client checklist items

Important:
- Templates = reusable definitions
- Client checklist = live, mutable data

👉 Never link directly. Always copy.

### 🔄 2. Flow: Template → Client Checklist
Step-by-step:

✅ Step 1: User selects template
```
Client: ABC Ltd
Template: Corporate Client
```

✅ Step 2: User clicks “Assign”

System does:
```
FOR each item in template:
    CREATE a new client checklist item
```

✅ Step 3: Items become “real tasks”

Each item now:
- Has its own status
- Has a due date
- Can be completed independently

✅ Step 4: Client dashboard updates

Now shows:
- Pending tasks
- Overdue items
- Progress %

### 🗃️ 3. Database Structure (CRITICAL)

You need two separate layers:

✅ Templates (Reusable)

templates
```
id: 1
name: "Corporate Client"
applies_to: "company"
```

template_items
```
id: 101
template_id: 1
title: "Certificate of Incorporation"
due_days: 3
order_index: 1

id: 102
title: "UBO Identification"
due_days: 7
```

✅ Client-side (REAL data)

client_checklists
```
id: 5001
client_id: 200
template_id: 1
assigned_at: 2026-05-21
```

client_checklist_items
```
id: 9001
client_id: 200
checklist_id: 5001
title: "Certificate of Incorporation"
status: "pending"
due_date: 2026-05-24
```

### 🧾 4. Real Example (End-to-End)

Template:

```
Corporate Client Template:

1. Certificate of Incorporation (3 days)
2. Director ID verification (5 days)
3. UBO identification (7 days)
```

User assigns template on:
```
Date: 1 June 2026
```

System generates:
```
1. Certificate of Incorporation
   due_date = 4 June 2026

2. Director ID verification
   due_date = 6 June 2026

3. UBO identification
   due_date = 8 June 2026
```

Later:

User marks item complete:
```
status = "completed"
completed_at = timestamp
```

### ⚙️ 5. Backend Logic (This is what you implement)

Here’s the exact logic you need.

✅ Assign Template Function

Pseudo-code:
```javascript
function assignTemplateToClient(clientId, templateId) {
  // 1. Get template
  const template = getTemplate(templateId);

  // 2. Get template items
  const items = getTemplateItems(templateId);

  // 3. Create client checklist
  const checklist = createClientChecklist({
    client_id: clientId,
    template_id: templateId,
    assigned_at: now()
  });

  // 4. Create checklist items
  items.forEach(item => {
    createClientChecklistItem({
      checklist_id: checklist.id,
      client_id: clientId,
      title: item.title,
      status: "pending",
      due_date: addDays(now(), item.due_days),
      order_index: item.order_index
    });
  });

  return checklist;
}
```

✅ SQL Example (simplified)
```sql
INSERT INTO client_checklists (client_id, template_id, assigned_at)
VALUES (200, 1, NOW());
```

Then:
```sql
INSERT INTO client_checklist_items
(client_id, checklist_id, title, due_date, status)
VALUES
(200, 5001, 'Certificate of Incorporation', '2026-06-04', 'pending');
```

### 🧠 6. Why This Design Matters

If you don’t copy templates, you’ll break everything.

❌ BAD approach:
```
Client points directly to template items
```

Problem:
- Changing template breaks existing clients
- No audit consistency
- Historical data corrupted

✅ GOOD approach:
```
Template → copied → client checklist items
```

Benefits:
- Safe updates
- Immutable history
- Audit-friendly ✅

### 🔥 7. Advanced Logic (Next-Level Features)

Once basic flow works, you can add:

✅ Conditional Items
```
IF risk = high
→ add "Enhanced Due Diligence"
```

✅ Multi-template assignment

Client can have:
- KYC checklist
- Annual review checklist
- EDD checklist

✅ Re-assignment (Annual Reviews)
```
assignTemplateToClient(clientId, annualReviewTemplateId)
```

Creates a fresh checklist every year

✅ Progress Calculation
```
progress = completed_items / total_items
```

Used for:
- Dashboard
- Client overview

### 🚨 8. Common Pitfalls (AVOID THESE)

❌ 1. Editing templates after assignment

→ breaks logic

✅ Solution:
- Treat templates as versioned OR immutable


❌ 2. Missing due date logic

→ system becomes useless

✅ Always compute:
```
due_date = assigned_date + due_days
```

❌ 3. No audit logging

✅ Log:
- "Item marked as complete"
- "Document uploaded"

You never serve the recipe to a customer—you serve the meal 🍽️

### 🧠 1. Are all checklists annual?

👉 No. Not at all.

Think of checklists as different types of workflows:

✅ Types of Checklists

1. One-time (Onboarding)

    Used when a client is first added.

    Example:
    - KYC (Individual)
    - Corporate onboarding

    👉 Happens once (initial setup)
  
2. Recurring (Annual / Periodic)
    
	Used repeatedly over time.

    Example:
    - Annual AML review
    - Quarterly risk check

    👉 These are the ones you re-assign

3. Conditional (Triggered)
    
	Only used when needed.

    Example:
    - High-risk client → Enhanced Due Diligence (EDD)
    - Suspicious activity → investigation checklist

    👉 Not regular—triggered manually or by rules
	
✅ So your app will have:

| Checklist type | Example         | Frequency  |
|----------------|-----------------|------------|
|Onboarding      | KYC / Corporate | Once       |
|Annual Review   | AML Review      | Every Year |	
|Conditional     | EDD             | When needed|

🔁 What “Re-assignment” Actually Means

Let’s simplify:
> Re-assignment = creating a fresh new checklist from the same template later

Example Timeline:

✅ Day 1 (Client created)

You assign:
```
Corporate Client Template
```

✅ 1 year later

You assign:
```
Annual Review Template
```

👉 This creates a NEW checklist, separate from the original.

🧠 Important Insight

A client can have multiple checklists over time:
```
ABC Ltd
├── Checklist #1: Onboarding (Completed ✅)
├── Checklist #2: Annual Review 2026
├── Checklist #3: Annual Review 2027
```

👉 You are not reusing the same checklist

👉 You are creating a new one each time

### 📂 2. How Documents Relate to Checklist Items

This is a very important design decision, and there are 2 approaches.

✅ Recommended Approach (Best for MVP)

> Documents belong to the CLIENT, not directly to checklist items

Structure:
```
Client
 ├── Documents (shared pool)
 ├── Checklist Items
 ```
 
Example:
``` 
Client: ABC Ltd

Documents:
- passport.pdf
- utility_bill.pdf
- incorporation_certificate.pdf
```

Then checklist items:
```
☐ Proof of Address
☐ Certificate of Incorporation
```

How they connect:

👉 When viewing a checklist item:

```
Proof of Address

Documents:
[ utility_bill.pdf ] ✅

[ Upload New Document ]
```

👉 The system simply lets users:
- Upload a doc
- Associate it with an item (optional link)

🧠 Why this is better
- ✅ Documents are reusable
- ✅ Real-world behavior matches
- ✅ Simpler database
- ✅ Less duplication

✅ Recommended Hybrid Model

Best balance:

Database Design

documents
```
id
client_id
file_url
uploaded_at
```

client_checklist_items
```
id
client_id
title
status
```


✅ Linking table (optional but useful)

checklist_item_documents
```
- item_id
- document_id
```

✅ UI Experience
```
In Checklist Item:

Proof of Address

Linked Documents:
✔ utility_bill.pdf

[ Upload Document ]
[ Link Existing Document ]
```

Upload flow:

When user uploads:

```
Upload Document

File: [Choose file]
Type: [Proof of Address ▼]
Link to item: ✅

[ Upload ]
```

#### 🔁 Example End-to-End

Step 1: Assign checklist
```
Proof of Address → Pending
```

Step 2: User uploads file
```
utility_bill.pdf
```

Step 3: User links it
```
Proof of Address → linked to utility_bill.pdf
```

Step 4: User completes item
```
✔ Proof of Address → Complete
```

🧠 Key Insight
A checklist item represents:

> ✅ “We have verified this requirement”

Documents are:

> 📂 “Evidence supporting that decision”

🔥 Simple Rule to Remember
```
Checklist item = status (what’s done)
Document = proof (why it’s done)
```

====

### 📑 MANXVERIFY | AML COMPLIANCE FILE

Generated On: 21 May 2026, 14:27 BST
Regulated Entity: Island Properties Ltd, Castletown, Isle of Man
IOMFSA Registration No: DB-0492-IOM1. 

#### 1. CLIENT PROFILE OVERVIEW

Client Name: David Robert Christian

Date of Birth: 14 November 1981

Nationality: Manx / British

Current Residential Address: 12 Mooragh Promenade, Ramsey, Isle of Man, IM8 3AB

Business Relationship Type: One-off High-Value Property Transaction (Buyer)

File Status: 🟢 VERIFIED & APPROVED2. 

#### 2. RISK ASSESSMENT MATRIX

The risk assessment below follows the IOMFSA Sector Specific AML/CFT Guidance Notes.

|Risk Factor|Assessment|System Notes / Justification|
|-----------|----------|----------------------------|
Geographic Risk | Low Risk |Client is a permanent Isle of Man resident utilizing a local Manx bank account.|
Customer Risk|Low Risk|Individual client; not a Politically Exposed Person (PEP) or high-profile figure.|
Delivery Channel|Medium Risk|Initial contact via digital web portal; subsequent identity verification completed face-to-face.|
Transaction Risk|Medium Risk|Property purchase value (£450,000) matches standard local market averages.
OVERALL RISK RATING|LOW RISK|Simplified Due Diligence (SDD) measures applied.

### 3. CUSTOMER DUE DILIGENCE (CDD) VERIFICATION LOG

📥 Document A: Verification of Identity (Photo ID)

Document Type: Isle of Man Driving Licence

Document Number: IOM-811114-DC-391

Expiry Date: 30 September 2031

Verification Method: Physical inspection of original document by staff member.

System Action: High-resolution scan captured and encrypted in secure Manx database cloud storage.

Timestamp / Verified By: 12 May 2026, 09:14 BST | Verified by: Sarah Quayle (Compliance Officer)

📥 Document B: Verification of Residential Address

Document Type: Manx Utilities Electricity Bill

Issue Date: 14 April 2026 (Within the required 3-month validity window)

Address Shown: 12 Mooragh Promenade, Ramsey, IM8 3AB (Matches client profile)

Timestamp / Verified By: 12 May 2026, 09:16 BST | Verified by: Sarah Quayle (Compliance Officer)

### 4. MANDATORY SCREENING CHECKS (PEP & SANCTIONS)

Automated background screening run via integrated global watchlists.

PEP Screening: ❌ NO MATCH FOUND (Client is not a Politically Exposed Person)

Sanctions Screening: ❌ NO MATCH FOUND (Client is not listed on UK/IOM consolidated financial sanctions lists)

Adverse Media Check: ❌ NO MATCH FOUND

Next Scheduled Re-Screening: 12 May 2027 (Or upon trigger event/subsequent transaction)

### 5. SOURCE OF FUNDS / WEALTH (SoF / SoW)

Stated Source of Funds: Local mortgage via Isle of Man Bank + Personal savings equity.

Evidence Secured: Mortgage Offer Letter (Reference: IOMB-99231-X) + 3 months of Isle of Man Bank statements showing consistent salary accumulation.

Compliance Sign-Off: Source of wealth aligns perfectly with the client’s known employment history as a local marine engineer.

### 6. DIGITAL AUDIT TRAIL (IMMUTABLE LOG)

This log cannot be edited or backdated by users, fulfilling IOMFSA record-keeping integrity requirements.

11 May 2026, 16:02 BST: Client profile created by user S_QUAYLE.

12 May 2026, 09:14 BST: Driving Licence uploaded, OCR data extracted, expiry validated.

12 May 2026, 09:17 BST: Automated PEP/Sanction background screening executed. Status: CLEAN.

12 May 2026, 11:30 BST: Risk assessment finalized as "Low Risk" by user S_QUAYLE.

12 May 2026, 11:31 BST: File status shifted to APPROVED. Digital certificate issued.

### Why this satisfies the inspector instantly:

When an inspector visits, they look for gaps, missing dates, or unverified files. By putting this specific look on your SaaS output, the inspector can instantly see who checked it, when it was checked, the exact legal documents used, and an uneditable timeline proving the business didn't just scramble to create the file the night before the audit.If you are planning to build this dashboard, let me know if you would like to explore the database schema needed to generate this log securely or look at the required user privacy rules (GDPR) for storing local Manx customer data.

