# 🧬 MedIntel — Agentic AI Clinical Trial Recommender

**Automated AI-powered workflow built using n8n and integrated with OpenAI for intelligent patient–trial matching**


## 📘 Overview

**MedIntel** is an agentic AI project designed to intelligently match patient data with relevant clinical trials using real-time information from global trial databases.

The system automatically reads patient details from Google Sheets, fetches available trials from *ClinicalTrials.gov*, analyzes suitability using an AI model, and sends summarized trial match alerts to doctors via email.

This solution demonstrates how **agentic workflows** can combine automation, reasoning, and communication — turning data pipelines into autonomous decision-making agents.


## 🎯 Problem Statement

Healthcare professionals often struggle to identify suitable clinical trials for their patients.
Manually searching through thousands of trials across multiple databases is time-consuming, error-prone, and rarely customized for each patient’s medical history.

**MedIntel** addresses this challenge by:

* Automating the retrieval of relevant clinical trials.
* Using AI reasoning to evaluate trial suitability for each patient.
* Sending automated alerts to doctors when promising matches are found.


## 💡 Objective

The goal of **MedIntel** is to act as an **AI assistant for medical researchers and doctors**, helping them find the right clinical opportunities for patients faster and with higher accuracy.


## ⚙️ Technology Stack

| Category            | Tool / Platform                                         |
| ------------------- | ------------------------------------------------------- |
| Workflow Automation | **n8n (Docker deployment)**                             |
| Data Source         | **Google Sheets**                                       |
| Trial Data API      | **ClinicalTrials.gov API**                              |
| AI Engine           | **OpenAI / OpenRouter LLM (Gemini / GPT-based models)** |
| Communication       | **Email (SMTP / Gmail)**                                |
| Hosting             | **Docker Container Environment**                        |


## 🔁 Workflow Explanation

The workflow consists of several connected nodes that automate the entire pipeline from reading data to delivering insights. Below is the step-by-step explanation of each node and its function.


### **1️ start workflow**

This is the **manual trigger** node used to start the execution of the entire workflow.
It allows the user to manually begin the automation by clicking *Execute Workflow* in n8n.


### **2️ fetch patient data**

This node reads patient records from a **Google Sheet** that contains structured information such as:
Patient ID, Name, Age, Sex, Condition, Stage, Biomarkers, Prior Treatments, ECOG, City, Country, Doctor details, and other relevant notes.
It acts as the entry point for patient data into the workflow.


### **3️ check active patients**

This is a **filtering (IF)** node that ensures only valid or active patients are processed.
For example, it checks whether a Patient ID or condition is available.
Patients with incomplete or invalid data are routed to a “No Operation” node for a clean exit.


### **4️ edit fields**

This node organizes and formats the patient details retrieved from the sheet.
It standardizes the data structure so that the next nodes can process it consistently when making API calls or preparing AI inputs.


### **5️ loop through patients**

This **Loop Over Items** node processes each patient individually, ensuring that all operations — from fetching trials to sending emails — are executed one at a time per patient.
It prevents data overlap and maintains a logical flow of execution.


### **6️ Edit Fields1**

This node maintains continuity by carrying the patient’s information forward through the loop.
It ensures that even after multiple operations, the patient’s data is available for reference during trial processing.


### **7️ fetch trials**

This node makes an HTTP request to **ClinicalTrials.gov**, a global database of ongoing and completed clinical studies.
It fetches trials related to the patient’s specific condition, disease, or biomarkers.
For example, if a patient has “HER2-positive breast cancer,” the node retrieves all trials that mention “HER2” or “breast cancer.”


### **8️ split trials**

The **Split Out** node takes the list of trials fetched from the API and separates them into individual items.
This allows each trial to be analyzed independently in the following steps, enabling the AI to assess compatibility per trial.


### **9️ edit fields2**

This node extracts important information from each trial’s dataset such as:

* Trial title and description
* Study phase (I, II, III, etc.)
* Location
* Status
* Conditions being studied
* NCT ID (unique trial identifier)

The extracted details are used to prepare a summarized version for the AI comparison process.


### **10 Basic LLM Chain**

This is the **AI reasoning core** of the workflow.
The node connects to an **OpenAI-compatible model** (via OpenRouter) to evaluate the compatibility between the patient’s condition and each clinical trial.
The model receives structured details about the patient and the trial and outputs a result in JSON format that includes:

* A **match score** (0–100)
* A **short summary** explaining the reasoning behind the score

This step transforms raw data into intelligent analysis using agentic AI reasoning.


### **11️ Edit Fields3**

This node extracts the AI model’s response and cleans it.
It isolates the **match score** and **summary** fields from the JSON response.
This ensures that subsequent nodes can use numeric match scores for filtering without errors.


### **12️ If1**

This **conditional logic** node filters out trials that have a match score below the defined threshold (for example, 75%).

* If the score is above or equal to the threshold, it is considered a potential match.
* If it’s below, the trial is ignored and routed to a “No Operation” node for a clean exit.

This ensures that only the most promising trials are forwarded to the notification step.


### **13️ Send email**

This node sends an **automated email** to the doctor associated with the patient.
The email includes:

* The patient’s name and condition
* Trial title and details
* Match score and summary of the AI’s reasoning
* A direct link to the ClinicalTrials.gov trial page

This transforms the AI’s analysis into immediate actionable information for the medical team.


### **14️ No Operation, do nothing**

This node is connected to branches that do not need further processing.
It acts as a termination point, ensuring that incomplete or unmatched cases end gracefully without affecting other workflow executions.

### **15 Appen row in sheet**

This node records the final results of successful matches into a separate Google Sheet.
Whenever a patient–trial match has a high score (≥75), it automatically appends a new row containing the patient details, trial title, match score, summary, and timestamp.
This creates a complete log of all matches found by the workflow for future reference or analysis.

## 🔁 Flow Diagram
<img width="812" height="322" alt="image" src="https://github.com/user-attachments/assets/8bf53cbd-3834-4a51-a4f5-a6a8d3a00a74" />


## 🧠 How the AI Makes Decisions

The AI model uses a combination of contextual reasoning and scoring logic to assess compatibility between patient and trial data.
It considers factors such as:

* Disease or condition similarity
* Biomarker alignment
* Trial phase and eligibility
* Prior treatment compatibility
* Age and performance scale (ECOG)

Each of these aspects contributes to a final **match score**, which helps prioritize the most relevant trials.


## 📤 Output and Notifications

When a patient–trial match exceeds the defined threshold:

* The doctor receives an **email notification** summarizing the trial and score.
* Trials with lower scores are ignored.
* Each execution cycle ensures the doctor only receives meaningful updates.
* High-match trials are automatically emailed to the respective doctor.
* The same trial details, including patient info, match score, and AI summary, are appended to a MatchedTrials Google Sheet for record-keeping.
* This sheet acts as a history log that can be used for further analysis or to avoid re-processing already matched patients.


## 🧩 Workflow Summary

| Step             | Function                                 |
| ---------------- | ---------------------------------------- |
| Data Ingestion   | Reads patient records from Google Sheets |
| Data Validation  | Filters valid patients                   |
| Trial Discovery  | Fetches relevant trials via API          |
| AI Analysis      | Scores trials using reasoning-based LLM  |
| Result Filtering | Retains trials above threshold           |
| Communication    | Sends alert emails to doctors            |
|Data Logging      |Apends hight match results to a separate google sheet for tracking|


## 🔒 Error Handling & Stability

* **Invalid Rows:** Handled by IF node to prevent errors.
* **Empty API Response:** The Split Out node handles no-trial cases gracefully.
* **AI Timeout or Invalid Output:** The workflow defaults to a safe “No Match” state.
* **Email Failures:** Logged automatically by n8n execution manager.


## 📈 Future Enhancements

* **Automatic Daily Scheduling:** Add a cron trigger to run the workflow automatically each morning.
* **Multi-Agent Extension:** Add a validation agent to recheck AI decisions.
* **Integration with Hospital EMR:** Directly sync patient data from electronic medical records.
* **WhatsApp Notification:** Parallel message alerts for faster doctor communication.


## 🏁 Conclusion

**MedIntel** successfully demonstrates how Agentic AI can combine automation, reasoning, and communication to solve real healthcare challenges.
The system autonomously reads patient data, discovers global clinical trials, evaluates compatibility using AI, alerts doctors with high-match recommendations, and now also logs every successful match into a separate Google Sheet for record-keeping.
By integrating data processing, intelligent decision-making, and result tracking in a single workflow, MedIntel moves closer to practical, AI-assisted precision medicine — giving doctors and patients faster access to potential life-saving treatments.



