# Stop Salesforce AI Hallucinations: 5 Ways to Build Trustworthy AI

Welcome to the official repository for the **Buckeye Dreamin' 2026** session presented by **Fnu Avi**, Senior Application & AI Architect at the University of Southern California (USC). 

This repository contains the complete configuration blueprints, prompt templates, declarative formula logic, and native audit metrics required to implement a **5-Layer Shield** framework around Salesforce Generative AI. 

## 📋 Session Details
* **Speaker:** Fnu Avi, Senior Application & AI Architect, USC
* **Event:** Buckeye Dreamin' 2026
* **Core Scenario:** USC President's Scholarship Eligibility Checker (Edge case mitigation where a high GPA student has a disqualifying incomplete course grade).

---

## 🛡️ The 5-Layer Shield Architecture

+-----------------------------------+
              |      User Input (The Query)       |
              +-------------------+---------------+
                                  |
+-------------------------------------v-------------------------------------+
| LAYER 1: GROUNDING (Salesforce CRM Context)                               |
| - Resolves Contact: GPA: 3.85, Incomplete: TRUE, SSN: XXX-XX-1234         |
+-------------------------------------+-------------------------------------+
|
+-------------------------------------v-------------------------------------+
| LAYER 2: CONSTRAIN THE MODEL (Einstein Trust Layer)                       |
| - Masks PII (SSN) -> [Filtered_SSN_1]                                     |
+-------------------------------------+-------------------------------------+
|
+-------------------------------------v-------------------------------------+
| LAYER 3: STRONGER PROMPTS (Einstein Prompt Builder)                       |
| - System Instructions: "Rule 1: GPA must be >=3.5. Rule 2: Incompletes = NO"|
+-------------------------------------+-------------------------------------+
|
+-------------------------------------v-------------------------------------+
| LAYER 4: VALIDATION LAYER (Flow Builder Rule Matcher)                     |
| - If LLM says "Eligible" but CRM Incomplete = True -> Force "Pending"     |
+-------------------------------------+-------------------------------------+
|
+-------------------------------------v-------------------------------------+
| LAYER 5: GOVERNANCE (CRM Native Audit Trail & Reporting)                  |
| - Log prompt metadata, evaluate system performance & track overrides     |
+-------------------------------------+-------------------------------------+
|
+-------------------v---------------+
|     Verified Output to User       |
+-----------------------------------+


---

## 🛠️ Repository File Structure

* `/prompt-templates/USC_Scholarship_Eligibility_Evaluator.md` — Layer 3 Prompt Builder source layout using Role-Rules-Context-Output structure.
* `/flows/USC_Scholarship_Evaluation_Flow.md` — Layer 4 Autolaunched Flow components and pattern-matching JSON formulas.
* `/dashboards/Defensive_Save_Rate_Report.md` — Layer 5 CRM analytics configuration parameters.

---

## 🚀 Layer-by-Layer Configuration Blueprints

### 1. Prerequisites (Target CRM Fields)
Create the following custom fields on the standard `Contact` object:
* **GPA** (`GPA__c`): Number (16, 2)
* **Active Incomplete Grades** (`Active_Incomplete_Grades__c`): Checkbox
* **SSN** (`SSN__c`): Text (11)
* **Scholarship Status** (`Scholarship_Status__c`): Picklist (`Applied`, `Eligible`, `Ineligible`, `Pending Review`)
* **AI Validation Code** (`AI_Validation_Code__c`): Text (255)

### 2. Layer 1 & 3: Einstein Prompt Builder Configuration
Create a **Flex** prompt template named `USC Scholarship Eligibility Evaluator`. Use the configuration below:

```text
You are an elite, highly precise Academic Eligibility Assistant for the University of Southern California (USC).
Your sole task is to evaluate whether a student is eligible for the USC President's Scholarship based strictly on the structured CRM context parameters provided below.

Strict Academic Bylaw Rules:
1. Eligibility GPA Threshold: The student's Cumulative GPA (G) must be G >= 3.5.
2. Disqualification Criteria: If the student has any active incomplete course grades (Active_Incomplete_Grades__c is True), they are strictly INELIGIBLE. This rule overrides a high GPA.

Student Record Parameters (Grounded from Salesforce CRM):
- First Name: {!$Input:Contact.FirstName}
- Cumulative GPA: {!$Input:Contact.GPA__c}
- Active Incomplete Grades Exist?: {!$Input:Contact.Active_Incomplete_Grades__c}
- Student Security Verification Token: {!$Input:Contact.SSN__c}

Strict JSON Output Format:
You must respond only in a valid, parsable JSON block. Do not include any pre-text, post-text, or markdown block wrappers.
{
  "studentName": "{!$Input:Contact.FirstName}",
  "gpa": {!$Input:Contact.GPA__c},
  "isIncompleteChecked": {!$Input:Contact.Active_Incomplete_Grades__c},
  "rawEligibilityResult": "Eligible" | "Ineligible",
  "architectReasoning": "Provide a concise explanation of your decision citing the rules applied."
}
