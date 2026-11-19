# ksm-if-onboardai-rag
AI-powered Retrieval-Augmented Generation (RAG) onboarding assistant for new members of Kelompok Studi Mahasiswa Informatika (KSM IF), Universitas Surabaya — built with Langflow, Google Gemini, and DataStax Astra DB.
# 🤖 KSM IF OnboardAI

**AI-Powered Intelligent Onboarding System for New Members of Kelompok Studi Mahasiswa Informatika (KSM IF) — Universitas Surabaya**

**KSM IF OnboardAI** is a Retrieval-Augmented Generation (RAG) conversational assistant built using **Langflow**, **Google Gemini**, and **DataStax Astra DB**. Designed as the **KSM IF Onboarding Buddy**, the system centralizes organizational knowledge and guides new members through KSM IF's culture, department structures, work programs (*program kerja*), digital tools, and onboarding Standard Operating Procedures (SOPs).

---

## 📋 Table of Contents
- [Problem Statement & Impact](#-problem-statement--impact)
- [Target Users](#-target-users)
- [Key Benefits](#-key-benefits)
- [System Architecture](#-system-architecture)
  - [Pipeline Overview](#pipeline-overview)
  - [Component & Hyperparameter Specifications](#component--hyperparameter-specifications)
- [Knowledge Base Structure](#-knowledge-base-structure)
- [Setup & Installation](#-setup--installation)
- [Playground Demo & Verification](#-playground-demo--verification)
- [Future Development Roadmap](#-future-development-roadmap)
- [Author](#-author)

---

## 🎯 Problem Statement & Impact

Onboarding new members into **KSM IF Universitas Surabaya** frequently faces bottlenecks because critical organizational information—ranging from management structures and division responsibilities to work programs, administrative SOPs, and digital tool guides—is scattered across separate documents, chat groups, and platforms.

### Affected Stakeholders
* **New Members (*Anggota Baru KSM IF*):** Experience difficulty understanding organizational culture, management structure, division tasks, and event execution workflows.
* **Committee & Senior Members (*Pengurus & Anggota Lama*):** Must repeatedly answer the same foundational questions and manually guide newcomers through basic organizational info.
* **Division Coordinators (*Koordinator Divisi*):** Spend additional time re-explaining internal workflows, task delegation, and operational standards to incoming members.

### Why It Matters
A structured, AI-assisted onboarding system enables new members to adapt faster, understand their roles clearly, and contribute effectively while maintaining consistent, verified answers across the organization.

---

## 👥 Target Users

| User Group | Role | Primary Needs |
| :--- | :--- | :--- |
| **1. New Members (*Anggota Baru KSM IF*)** | Primary Users | Easy-to-understand onboarding guides, centralized information access, and instant answers to organizational questions. |
| **2. Committee & Division Coordinators (*Pengurus & Koordinator Divisi*)** | Supporting Users | Reduced repetitive Q&A, standardized onboarding materials, and visibility into newcomer comprehension. |
| **3. Core Executive Board (*Pengurus Inti / BPH*)** | Administrative Users | Updating official documentation, managing access permissions, and keeping onboarding content up to date. |

---

## ✨ Key Benefits

* **Accelerates Newcomer Adaptation:** Enables self-paced learning of organizational structure, division duties, and operational workflows.
* **Centralizes Organizational Knowledge:** Unifies handbooks, SOPs, FAQs, IT tool guides, and work program schedules in one conversational interface.
* **Reduces Committee Workload:** Automates responses to repetitive onboarding questions so coordinators and vice-coordinators can focus on mentoring and program execution.
* **Ensures Information Consistency:** Grounds every response strictly in verified official KSM IF documentation.
* **Improves Member Engagement:** Supports structured onboarding stages, checklists, and interactive Q&A aligned with KSM IF's *"We Not Me"* motto.
* **Preserves Institutional Knowledge:** Retains centralized documentation for future leadership periods (*regenerasi kepengurusan*).

---

## 🏗️ System Architecture

The system is implemented in **Langflow (`v1.12.2`)** using a dual-pipeline RAG architecture backed by **DataStax Astra DB** and **Google Generative AI**.

![KSM IF OnboardAI Langflow Canvas](docs/Main_Flow_Canvas.png)

### Pipeline Overview

```mermaid
flowchart TD
    subgraph Flow1["Flow 1: Document Ingestion & Vector Indexing"]
        A[Read File<br/>7 Official KSM IF PDFs] --> B[Split Text<br/>Chunk: 800 | Overlap: 120]
        B --> C[Google Generative AI Embeddings<br/>models/gemini-embedding-001]
        C --> D[(Astra DB Vector Store<br/>KSMIF_Database / KSMIF_collection)]
    end

    subgraph Flow2["Flow 2: Buddy Agent Q&A Retrieval"]
        E[Chat Input<br/>User Question] --> F[Google Generative AI Embeddings<br/>models/gemini-embedding-001]
        F --> G[(Astra DB Vector Store<br/>Vector Similarity Search, k=4)]
        D -.->|Indexed Vectors| G
        G --> H[Parser<br/>Mode: Stringify]
        H -->|context| I[Prompt Template<br/>KSM IF Onboarding Buddy]
        E -->|question| I
        I --> J[Language Model<br/>gemini-3.5-flash-lite]
        J --> K[Chat Output<br/>Grounded Response]
    end
