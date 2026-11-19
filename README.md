# ksm-if-onboardai-rag
AI-powered Retrieval-Augmented Generation (RAG) onboarding assistant for new members of Kelompok Studi Mahasiswa Informatika (KSM IF), Universitas Surabaya — built with Langflow, Google Gemini, and DataStax Astra DB.
# 🤖 KSM IF OnboardAI — Intelligent RAG Onboarding Assistant

**KSM IF OnboardAI** is an AI-powered onboarding and organizational knowledge assistant built for **Kelompok Studi Mahasiswa Informatika (KSM IF), Universitas Surabaya**. Powered by a **Retrieval-Augmented Generation (RAG)** pipeline in **Langflow**, **Google Gemini**, and **DataStax Astra DB**, the system centralizes organizational documentation into an interactive conversational buddy (*"KSM IF Onboarding Buddy"*) to help new members adapt faster and reduce repetitive administrative Q&A.

<p align="center">
  <img src="images/main-flow-canvas.png" alt="KSM IF OnboardAI Langflow Canvas" width="900"/>
  <br/>
  <sub><b>System Overview:</b> End-to-end RAG ingestion and conversational agent pipeline in Langflow.</sub>
</p>

---

## 📋 Table of Contents
- [Problem Statement & Impact](#-problem-statement--impact)
- [Target Users](#-target-users)
- [Key Benefits](#-key-benefits)
- [System Architecture (RAG Pipeline)](#-system-architecture-rag-pipeline)
- [Knowledge Base & Prompt Engineering](#-knowledge-base--prompt-engineering)
- [Playground Demo](#-playground-demo)
- [Setup & Installation Guide](#-setup--installation-guide)
- [Future Development Roadmap](#-future-development-roadmap)
- [Author](#-author)

---

## 🎯 Problem Statement & Impact

### The Challenge
Onboarding new members at **KSM IF** often faces bottlenecks because essential information—such as organizational structure, division responsibilities, work programs (*program kerja*), administrative SOPs, and event guidelines—is scattered across multiple documents, chat groups, and digital platforms. As a result, new members struggle to locate accurate references and understand internal workflows, while senior members and board officers repeatedly spend time answering the same foundational questions.

### Impacted Stakeholders
| Stakeholder Group | Core Pain Points |
| :--- | :--- |
| **New KSM IF Members** | Difficulty understanding organizational culture (*"We Not Me"*), leadership structure, division roles, and event execution procedures. |
| **Board & Senior Members** | Repetitive workload from answering identical basic questions and manually guiding every incoming batch. |
| **Division Coordinators** | Extra time spent re-explaining division workflows, task delegation, and operational standards to new recruits. |

---

## 👥 Target Users

1. **New KSM IF Members *(Primary Users)***
   * Newly recruited Informatics students at Universitas Surabaya who need clear onboarding guidance, centralized knowledge access, and instant answers regarding the organization.
2. **Board Members & Division Coordinators *(Supporting Users)***
   * Coordinators and vice coordinators across **IRD, PRD, HRDD, and CDD** responsible for mentoring new members, delivering consistent onboarding materials, and tracking member comprehension.
3. **Executive Board / BPH *(Core Administrators)***
   * Chairperson, Vice Chairperson, Secretary, and designated officers responsible for maintaining up-to-date documentation, managing access permissions, and sustaining onboarding quality across leadership terms.

---

## ✨ Key Benefits

* **Accelerates New Member Adaptation:** Enables new members to independently explore organizational structure, division duties, and SOPs at their own pace.
* **Centralizes Organizational Knowledge:** Consolidates handbooks, FAQs, IT tool guides, and work programs into a single conversational interface.
* **Reduces Administrative Burden:** Automates answers to recurring onboarding questions so coordinators and BPH can focus on mentorship and program execution.
* **Ensures Information Consistency:** Grounded strictly in verified official PDF documents to prevent conflicting or outdated answers.
* **Improves Onboarding Engagement:** Guides newcomers through structured onboarding milestones and actionable next steps.
* **Preserves Institutional Knowledge:** Establishes a reusable knowledge base that carries forward seamlessly to future leadership periods.

---

## 🏗️ System Architecture (RAG Pipeline)

The workflow (`flow/KSM IF OnboardAI.json`) is built in **Langflow (`v1.12.2`)** and consists of two decoupled sub-flows:

### Flow 1: Document Ingestion & Vector Indexing
1. **Read File (`File`):** Loads the 7 official KSM IF onboarding PDF handbooks.
2. **Split Text (`SplitText`):** Splits raw document text into overlapping semantic chunks (`chunk_size = 800`, `chunk_overlap = 120`, recursive separators `["\n\n", "\n", ". ", " ", ""]`).
3. **Google Generative AI Embeddings (`models/gemini-embedding-001`):** Converts text chunks into high-dimensional vector representations.
4. **Astra DB Vector Store (`KSMIF_Database` / `KSMIF_collection`):** Stores text chunks and embeddings inside DataStax Astra DB (`default_keyspace`).

### Flow 2: RAG Conversational Buddy Agent
1. **Chat Input:** Captures incoming questions from users in the Playground or integrated chat interface.
2. **Query Embedding (`models/gemini-embedding-001`):** Embeds the user's question into the same vector space.
3. **Astra DB Vector Search:** Retrieves the top `k = 4` most semantically similar document chunks using cosine similarity search.
4. **Parser (`Stringify` mode):** Formats retrieved JSON/Data chunks into clean plain-text context (`{context}`).
5. **Prompt Template:** Combines `{context}` and `{question}` with strict grounding rules, document-routing instructions, and KSM IF persona guidelines.
6. **Language Model (`gemini-3.5-flash-lite`):** Generates accurate, friendly, Indonesian-language responses at low temperature (`temperature = 0.1`) to minimize hallucination.
7. **Chat Output:** Delivers the structured response back to the user.

---

## 📚 Knowledge Base & Prompt Engineering

### Indexed Reference Documents
The system routes user inquiries across seven specialized reference documents:

| Document | Scope & Topics Covered |
| :--- | :--- |
| `01_Organization_Profile.pdf` | General profile, history, vision, mission, goals, and the *"We Not Me"* motto. |
| `02_Department_Handbook.pdf` | Organizational hierarchy and responsibilities of **BPH, IRD, PRD, HRDD, and CDD**. |
| `03_Work_Programs.pdf` | Annual work programs (*program kerja*), objectives, schedules, venues, and PIC departments. |
| `04_New_Member_FAQ.pdf` | Frequently asked questions from prospective and newly admitted members. |
| `05_IT_Tools_Guide.pdf` | Digital ecosystem guide: KSM IF Website, WhatsApp Community, Google Drive/Sheets, GitHub, Instagram, *Gallery Proker*, and *Bursa Soal*. |
| `06_Onboarding_SOP.pdf` | Step-by-step onboarding stages, department introductions, learning milestones, and evaluations. |
| `07_Member_Guidelines.pdf` | Code of conduct, internal communication etiquette, collaboration, documentation, and data privacy. |

### Prompt Guardrails & Persona Design
<p align="center">
  <img src="images/prompt-template.png" alt="Prompt Template Configuration" width="850"/>
  <br/>
  <sub><b>Prompt Engineering:</b> Structured instruction template enforcing context grounding and friendly onboarding tone.</sub>
</p>

* **Strict Context Grounding:** Instructs the LLM to answer exclusively from retrieved Astra DB context and transparently redirect users to **BPH** or relevant **Division Coordinators** when information is unavailable.
* **Department Role Isolation:** Prevents mixing responsibilities across **BPH** (leadership, admin, finance), **IRD** (internal bonding), **PRD** (external relations & benchmarking), **HRDD** (member skill development, *Bursa Soal*, website), and **CDD** (visual identity, design, social media).
* **Supportive Onboarding Tone:** Uses warm, approachable Indonesian aligned with KSM IF's collaborative culture (*"We Not Me"*).

---

## 💬 Playground Demo

<p align="center">
  <img src="images/playground-demo.png" alt="KSM IF OnboardAI Playground Conversation" width="850"/>
  <br/>
  <sub><b>Live Interaction:</b> New member asking where to begin their KSM IF onboarding journey.</sub>
</p>

---

## 🚀 Setup & Installation Guide

### Prerequisites
* **Langflow** (`v1.12.2` or newer — local Docker/Python installation or DataStax Langflow Cloud)
* **Google AI Studio API Key** (with access to `models/gemini-embedding-001` and `gemini-3.5-flash-lite`)
* **DataStax Astra DB** account with a Vector Database (`KSMIF_Database`) and Collection (`KSMIF_collection`)

### Quick Start
1. **Clone this repository:**
   ```bash
   git clone [https://github.com/valenciatheodora07-tech/ksm-if-onboardai-rag.git](https://github.com/valenciatheodora07-tech/ksm-if-onboardai-rag.git)
   cd ksm-if-onboardai-rag
   ```
2. **Import the Flow into Langflow:**
   * Open your Langflow workspace and click **Import** (or upload `flow/KSM IF OnboardAI.json`).
3. **Configure API Credentials:**
   * Paste your **Google Generative AI API Key** into both `Google Generative AI Embeddings` nodes and the `Language Model` node.
   * Paste your **Astra DB Application Token** into both `Astra DB` vector store nodes and verify `Database` (`KSMIF_Database`) and `Collection` (`KSMIF_collection`).
4. **Ingest Knowledge Base Documents (Flow 1):**
   * Upload the 7 PDF handbooks into the **Read File** component and run the ingestion flow to populate Astra DB.
5. **Launch the Playground (Flow 2):**
   * Click **Playground** in the top-right corner and start chatting with **KSM IF Onboarding Buddy**.

---

## 🗺️ Future Development Roadmap

KSM IF OnboardAI is designed to scale incrementally into a comprehensive organizational learning and knowledge management platform:

1. **Personalized Onboarding Path:** Tailored learning tracks customized by division (**IRD, PRD, HRDD, CDD**), role, and prior experience.
2. **AI Quiz & Learning Assessment:** Automated quiz generation from official handbooks with instant explanations for incorrect answers to help coordinators identify knowledge gaps.
3. **Onboarding Analytics Dashboard:** Visual metrics tracking onboarding completion rates, quiz performance, top recurring questions, and documentation gaps.
4. **Organizational Platform Integrations:** Direct sync with **Google Drive**, **Google Calendar**, and internal messaging platforms for automated reminders and live document updates.
5. **Multimodal AI Assistant:** Support for parsing slide decks, infographics, and voice-based Q&A for faster mobile accessibility.

---
