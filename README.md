<div align="center">

# 🫀 NeuroCardia

### AI-Assisted Cardiology Information System (CIS)

**DICOM-native ECG acquisition · Deep-learning rhythm classification · RAG-grounded report drafting · Physician-in-the-loop sign-off**

<img src="ecg_machine.jpeg" alt="NeuroCardia ECG System" width="100%" style="border-radius: 12px; max-height: 440px; object-fit: cover;" />

<br/>

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-5.0%2B-092E20?style=for-the-badge&logo=django&logoColor=white)](https://www.djangoproject.com/)
[![DICOM](https://img.shields.io/badge/DICOM-PS3.0-005B94?style=for-the-badge)](https://www.dicomstandard.org/)
[![Channels](https://img.shields.io/badge/Real--Time-Django_Channels_(ASGI)-008080?style=for-the-badge)](https://channels.readthedocs.io/)
[![SDG 3](https://img.shields.io/badge/UN_SDG-3_Good_Health-4C9F38?style=for-the-badge)](https://sdgs.un.org/goals/goal3)
[![Status](https://img.shields.io/badge/Status-Final_Year_Project-brightgreen?style=for-the-badge)](#-14-project-status--limitations)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)

</div>

---

## 📑 Table of Contents

1. [The Problem](#-1-the-problem)
2. [The Solution](#-2-the-solution)
3. [Technology Stack](#-3-technology-stack)
4. [System Architecture](#-4-system-architecture)
5. [Database Design](#-5-database-design)
6. [Deep Learning Model](#-6-deep-learning-model)
7. [RAG Report Generator](#-7-rag-report-generator)
8. [Signal Processing Pipeline](#-8-signal-processing-pipeline)
9. [DICOM Networking Flow](#-9-dicom-networking-flow)
10. [Installation & Quickstart](#-10-installation--quickstart)
11. [Role-Based Access Control](#-11-role-based-access-control)
12. [Security & Privacy](#-12-security--privacy)
13. [Roadmap](#-13-roadmap)
14. [Project Status & Limitations](#-14-project-status--limitations)
15. [License & Author](#-15-license--author)

---

## 📌 1. The Problem

In high-volume public hospitals and cardiology centers in developing regions such as Pakistan, ECG management often remains manual and paper-based.

```mermaid
flowchart LR
    A["Patient queue"] --> B["Paper slip"]
    B --> C["Thermal ECG strip"]
    C --> D["Handwritten note"]
    D --> E["Paper folder / archive"]
    E -. "Patient returns months later" .-> F{"Prior tracing found?"}
    F -- "Often no" --> G["Repeat test, delayed diagnosis"]
    F -- "Sometimes" --> H["Manual comparison"]
```

| Pain point | Impact |
| :--- | :--- |
| **Fragile paper traces** | Thermal strips fade, tear, or get misplaced. |
| **Lost longitudinal context** | In OPDs seeing 150–200+ patients a day, finding a prior tracing is impractical, which leads to redundant tests and higher costs. |
| **Inconsistent free-text reports** | Hurried, handwritten notes vary in vocabulary and structure, increasing ambiguity and error risk. |
| **No urgency triage** | Routine scans share one queue with critical cases (ACS/STEMI). |
| **Repeated data entry** | Technicians retype demographics at the cart, causing typos, mislabeled records, and duplicate IDs. |

---

## 💡 2. The Solution

**NeuroCardia** replaces the paper workflow with a browser-accessible, closed-loop Cardiology Information System designed for hospital networks.

```mermaid
flowchart LR
    A["Bedside ECG cart"] -->|"DICOM C-STORE"| B["Ingestion services"]
    B --> C["Signal processing + AI analysis"]
    C --> D["RAG-grounded draft report"]
    D --> E["Cardiologist review and sign-off"]
    E --> F["Locked report: PDF + database"]
    F --> G["Instant history retrieval"]
```

- **Zero-retype connectivity**: Works with DICOM-capable ECG carts (e.g., GE, Schiller, Edan, Mindray) over LAN/Wi-Fi using Modality Worklist (`C-FIND`) and `C-STORE`.
- **Automated pre-reading**: Cleans waveforms, delineates wave boundaries, measures intervals (PR, QRS, QT, QTc), and flags arrhythmias, conduction blocks, and STEMI territory.
- **Grounded report drafting**: Retrieval-Augmented Generation (RAG) over clinical guidelines (AHA/ACC/ESC) and local report templates produces structured drafts and reduces hallucination risk.
- **Physician-in-the-loop**: AI output is advisory. Every report stays in `Draft` until an authorized cardiologist reviews, edits, and signs it.
- **Longitudinal retrieval**: Waveforms and verified reports are linked to the patient master record for fast side-by-side comparison.
- **On-premise first**: Designed to run inside the hospital network. Local LLM inference is the default; any cloud LLM is strictly opt-in.

---

## 🛠️ 3. Technology Stack

| Layer | Technology | Responsibility |
| :--- | :--- | :--- |
| **Backend** | Django 5.0+, Python 3.10+ | Web framework, ORM, RBAC, sessions |
| **Real-time** | Django Channels (ASGI / WebSockets) | Live notifications and clinical assistant chat |
| **Medical standards** | `pydicom`, `pynetdicom` | DICOM parsing, C-STORE SCP, Modality Worklist SCP |
| **Signal processing** | `neurokit2`, `scipy`, `numpy`, `matplotlib` | Filtering, baseline removal, R-peak detection, delineation |
| **Deep learning** | PyTorch / TensorFlow | 1D-CNN + BiGRU + attention classifier |
| **Generative AI / RAG** | Local LLM (default), Google Generative AI (optional) | Vector retrieval and structured report drafting |
| **Database** | PostgreSQL (production), SQLite3 (development) | ACID-compliant clinical records and visits |
| **Documents** | `Pillow`, `pdf2image`, `python-barcode` | 12-lead rendering and PDF report generation |
| **Frontend** | HTML5, CSS3, Bootstrap 5, FontAwesome | Responsive UI for clinical staff |

---

## 🏗️ 4. System Architecture

NeuroCardia uses a layered architecture with background services for DICOM ingestion.

```mermaid
flowchart TD
    subgraph L1["1 · Hospital hardware"]
        Cart["Bedside ECG cart<br/>(DICOM-capable)"]
    end

    subgraph L2["2 · Acquisition and ingestion"]
        MWL["Modality Worklist SCP<br/>(C-FIND)"]
        Store["Storage SCP<br/>(C-STORE)"]
        Watcher["File watcher daemon<br/>(dicom_watcher.py)"]
    end

    subgraph L3["3 · NeuroCardia core (Django / ASGI)"]
        Router["URL and API router"]
        Accounts["accounts<br/>RBAC, auth, audit log"]
        Patients["patients<br/>registry, visits"]
        ECG["ecg<br/>waveform pipeline"]
        Reports["reports<br/>RAG synthesis, PDF export"]
        Doctors["doctors<br/>worklist, review"]
    end

    subgraph L4["4 · AI inference"]
        Pre["Denoising and calibration"]
        DL["1D-CNN + BiGRU + attention"]
        RAG["Vector knowledge store<br/>(AHA/ESC, templates)"]
    end

    subgraph L5["5 · Persistence"]
        DB[("PostgreSQL / SQLite")]
        Vault[("Encrypted file vault<br/>.dcm, plots, PDFs")]
    end

    Cart -->|"1. Query worklist"| MWL
    MWL -->|"Scheduled visits (from DB)"| Cart
    Cart -->|"2. Send ECG"| Store
    Store -->|"Save .dcm"| Vault
    Watcher -->|"Poll for new studies"| Vault
    Watcher -->|"HTTP POST ingest"| Router

    Router --> Patients
    Router --> ECG
    Router --> Accounts
    Patients -->|"Link study to visit"| DB
    ECG --> Pre --> DL
    DL -->|"Findings + metrics"| Reports
    RAG -->|"Top-K context"| Reports
    Reports -->|"Save draft"| DB
    Doctors -->|"Review and sign-off"| Reports
    Reports -->|"Generate PDF"| Vault
```

---

## 🗄️ 5. Database Design

Relational metadata is stored in the database, while large binary files (`.dcm`, plots, PDFs) live in the file vault and are referenced by path.

```mermaid
erDiagram
    USER ||--o| DOCTOR : "has profile"
    USER ||--o| TECHNICIAN : "has profile"
    USER ||--o| RECEPTIONIST : "has profile"
    USER ||--o| ADMIN : "has profile"

    PATIENT ||--o{ VISIT : attends
    DOCTOR  ||--o{ VISIT : "assigned to"
    VISIT   ||--o| REPORT : generates

    USER {
        int user_id PK
        string username
        string password_hash
        string email
        string role "Doctor, Technician, Receptionist, Admin"
    }
    DOCTOR {
        int doctor_id PK
        int user_id FK
        string specialization
        string license_number
    }
    TECHNICIAN {
        int technician_id PK
        int user_id FK
    }
    RECEPTIONIST {
        int receptionist_id PK
        int user_id FK
    }
    ADMIN {
        int admin_id PK
        int user_id FK
    }
    PATIENT {
        int patient_id PK
        string given_name
        string family_name
        string gender
        date date_of_birth
        string national_id "CNIC"
        string contact
        string address
    }
    VISIT {
        int visit_id PK
        int patient_id FK
        int doctor_id FK
        date visit_date
        time visit_time
        decimal fee
        string modality "ECG, ETT, Holter"
        string status "Scheduled, Completed, Cancelled"
        string location
    }
    REPORT {
        int report_id PK
        int visit_id FK
        string file_path "path to .dcm"
        string ai_classification "AI-predicted findings"
        text report_text "clinician-approved narrative"
        string status "Draft, Final"
        datetime approved_at
    }
```

**Data dictionary highlights**

- **`User` and profiles**: A custom `AbstractUser` with a `role` field separates clinical rights (signing reports) from administrative rights (scheduling, intake).
- **`Patient` and `Visit`**: Anchor every diagnostic encounter. Duplicate registration is prevented by multi-field validation (name + date of birth + national ID).
- **`Report`**: Links the raw waveform reference to metrics, AI findings, and physician edits. Once `status = Final`, the report is locked and cannot be modified.

---

## 🧠 6. Deep Learning Model

The classifier follows a hybrid CNN–RNN–attention design informed by published ECG-classification literature (Xu et al., Singh et al.).

```mermaid
flowchart TD
    A["12-lead input<br/>12 x N, normalized, 500 Hz"] --> B
    B["1D-CNN block<br/>Conv1D + BatchNorm + ReLU + MaxPool<br/>Learns wave morphology, QRS width, ST shifts"] --> C
    C["Bidirectional GRU<br/>Multi-layer, forward and backward context<br/>Learns inter-beat rhythm and conduction timing"] --> D
    D["Lead-wise multi-head attention<br/>Weights each lead's contribution<br/>Produces pooled representation"] --> E
    E["Classification head<br/>Dense + Sigmoid, calibrated thresholds<br/>Multi-label probabilities"]
```

**Explainability.** The attention weights give a per-lead indication of which leads contributed most to a prediction (for example, II, III, and aVF for an inferior STEMI). This is a review aid for clinicians, not a causal explanation.

**Diagnostic scope**

| Category | Labels |
| :--- | :--- |
| **Rhythm** | Normal sinus rhythm, sinus tachycardia, sinus bradycardia, atrial fibrillation |
| **Conduction** | First-degree AV block (PR > 200 ms), LBBB, RBBB |
| **Ischemia / infarction** | STEMI localization: anterior, inferior, lateral |

---

## 📄 7. RAG Report Generator

Free-form text generation by a general-purpose LLM can hallucinate, which is unacceptable in cardiology. NeuroCardia constrains generation by retrieving relevant guidelines and templates and grounding the prompt in measured values.

```mermaid
flowchart TD
    M["Extracted metrics<br/>HR, PR, QRS, QTc, classifier output"] --> P
    K["Local vector knowledge store<br/>AHA/ESC guidelines, hospital phrasing,<br/>verified reference cases"] -->|"Top-K contexts"| P
    P["Context-grounded prompt builder"] --> S
    S["Clinical language synthesizer<br/>Structured 5-section output"] --> D["Draft report<br/>status = Draft"]
    D --> R["Physician review and sign-off"]
```

**Standardized five-section structure**

1. **Rhythm analysis**: Underlying mechanism, rate, and regularity.
2. **Conduction intervals and axis**: PR, QRS, QT/QTc, and electrical axis.
3. **Chamber morphology**: Voltage criteria for LVH/RVH and atrial enlargement.
4. **Ischemia and infarction**: Lead-grouped ST-T changes, reciprocal depression, pathological Q waves.
5. **Clinical impression**: Summary categorized as routine, abnormal, or critical.

---

## ⚡ 8. Signal Processing Pipeline

Raw waveforms pass through a deterministic DSP sequence before reaching the AI or visualization components.

```mermaid
flowchart LR
    A["DICOM ingestion<br/>(pydicom)"] --> B["Amplitude calibration<br/>ADC counts to mV"]
    B --> C["Resample to 500 Hz"]
    C --> D["Bandpass filter<br/>0.5-45 Hz Butterworth, zero-phase"]
    D --> E["DWT delineation<br/>P, QRS, T boundaries"]
    E --> F["Metrics<br/>HR, PR, QRS, QT, QTc, ST"]
```

| Stage | Details |
| :--- | :--- |
| **Ingestion and calibration** | Parse DICOM waveform sequences; convert ADC counts to millivolts using each channel's sensitivity and baseline tags. |
| **Resampling** | Interpolate every lead to a uniform **500 Hz**, so recordings from different machine brands are consistent. |
| **Filtering** | **0.5–45 Hz** zero-phase Butterworth bandpass removes respiratory baseline wander and high-frequency EMG noise. |
| **Delineation** | Discrete Wavelet Transform locates R-peaks and the onset/peak/offset of P, QRS, and T waves. |
| **Metrics** | HR, RR variability, PR, QRS duration, QT/QTc, and ST elevation/depression. |

QTc is computed with Bazett's formula, with `RR` in seconds:

$$
QTc = \frac{QT}{\sqrt{RR}}
$$

---

## 📡 9. DICOM Networking Flow

A dedicated asynchronous DICOM layer connects hospital hardware to the application.

```mermaid
sequenceDiagram
    autonumber
    actor Tech as ECG technician
    participant Cart as ECG cart (SCU)
    participant MWL as MWL SCP
    participant Store as Storage SCP
    participant Watch as File watcher
    participant App as Django backend

    Tech->>Cart: Open worklist
    Cart->>MWL: C-FIND (scheduled procedures)
    MWL-->>Cart: Matching scheduled visits
    Tech->>Cart: Select patient, record 12-lead ECG
    Cart->>Store: C-STORE (12-lead ECG waveform object)
    Store->>Store: Save .dcm to date-partitioned folder
    Store-->>Cart: C-STORE response (success)
    Watch->>Store: Detect new file in media/dicom/
    Watch->>App: Ingest notification (parsed header tags)
    App->>App: Link study to open visit
    App->>App: Generate plots and AI draft report
```

- **Modality Worklist (MWL)**: A DICOM SCP answering `C-FIND` requests. The technician selects the scheduled patient on the cart itself, which removes manual entry.
- **Storage SCP (C-STORE)**: Accepts 12-lead ECG waveform objects (SOP Class UID `1.2.840.10008.5.1.4.1.1.9.1.1`, *12-lead ECG Waveform Storage*).
- **Watcher daemon (`dicom_watcher.py`)**: Runs continuously. When a file lands in `media/dicom/`, it parses header tags, links the study to the patient's open visit, and triggers plot generation and AI drafting.

> **Note:** DICOM's default port, `104`, is privileged on Linux/macOS. Use a non-privileged port such as `11112` in development, and configure the cart's AE title, host, and port to match.

---

## 🚀 10. Installation & Quickstart

### Prerequisites

| Requirement | Notes |
| :--- | :--- |
| **Python** | 3.10 or higher |
| **Git** | Available on `PATH` |
| **Poppler** | Required for PDF rendering. Windows: download the binaries and add `bin/` to `PATH`. Ubuntu/Debian: `sudo apt-get install poppler-utils`. macOS: `brew install poppler`. |

### Setup

```bash
# 1. Clone the repository
git clone https://github.com/RanaHashir0/NeuroCardia.git
cd NeuroCardia

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate          # Linux / macOS
# .\venv\Scripts\Activate.ps1     # Windows (PowerShell)

# 3. Install dependencies
pip install -r requirements.txt

# 4. Initialize the database and create an admin account
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

### Running the services

NeuroCardia needs four processes running together: the web app, the MWL server, the C-STORE receiver, and the file watcher.

**Option A: Windows all-in-one launcher (recommended)**

```cmd
run_all.bat
```

This opens four terminal windows:

1. Django web application (`0.0.0.0:8000`)
2. DICOM Modality Worklist (MWL) server
3. DICOM C-STORE storage receiver
4. DICOM file watcher daemon

**Option B: Manual (any OS)**

Run each command in its own terminal. Adjust the script names to match your repository layout.

```bash
# Terminal 1: Web application
python manage.py runserver 0.0.0.0:8000

# Terminal 2: Modality Worklist server
python dicom/scripts/mwl_server.py

# Terminal 3: C-STORE storage receiver
python dicom/scripts/store_scp.py

# Terminal 4: File watcher daemon
python dicom/scripts/dicom_watcher.py
```

Open the web portal at **http://localhost:8000/**.

---

## 👥 11. Role-Based Access Control

| Role | Permissions |
| :--- | :--- |
| **Receptionist / Admin** | Register patients, book visits and procedures, verify payments, update demographics, manage hospital accounts. |
| **ECG Technician** | Query the Modality Worklist, verify cart–patient pairing, confirm waveform quality, trigger the AI analysis pipeline. |
| **Doctor / Cardiologist** | View the clinical worklist, inspect calibrated 12-lead waveforms, review and edit AI drafts, sign and lock final PDF reports. |

---

## 🔐 12. Security & Privacy

- **Network isolation**: Intended to run on the hospital LAN. No external cloud service is required when using the local LLM.
- **Optional cloud LLM**: If Google Generative AI is enabled, de-identify inputs first and obtain institutional approval. This mode is off by default.
- **Access control**: Role-based permissions enforced server-side; report signing is restricted to authorized clinicians.
- **Auditability**: Authentication events and report state changes are recorded in an audit log.
- **Data at rest**: Studies, plots, and PDFs are stored in a restricted file vault. Enable disk or volume encryption in production.
- **Immutability**: Reports with `status = Final` are locked against modification.

---

## 🗺️ 13. Roadmap

- [x] DICOM Modality Worklist and C-STORE integration
- [x] Signal preprocessing and interval extraction
- [x] Patient, visit, and report management with RBAC
- [ ] Model training, validation, and benchmarking on public ECG datasets
- [ ] RAG knowledge-base expansion and evaluation
- [ ] Side-by-side historical ECG comparison view
- [ ] Urgency-based worklist triage (STEMI-first ordering)
- [ ] Prospective validation with cardiologists

---

## ⚠️ 14. Project Status & Limitations

> **NeuroCardia is an academic final-year project and research prototype. It is not a certified medical device and must not be used for real clinical decision-making without independent validation and the required regulatory approval.**

- AI predictions are assistive only; a qualified cardiologist must review every report.
- Model performance metrics (sensitivity, specificity, F1, calibration) will be published once validation is complete.
- Compatibility with specific ECG cart models depends on each vendor's DICOM conformance statement.

---

## 📄 15. License & Author

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.

**Author:** [Rana Hashir](https://github.com/RanaHashir0)