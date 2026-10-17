

# DESCRIPTION

**JADE File Transfer** is an intelligent multi-agent system built with JADE, designed for secure and controlled file transfers between two workstations. The sender user easily selects a file via a simple graphical interface, after which it is chunked and transmitted through a specialized agent (**FileManager**) to the recipient. At the receiver side, after explicit confirmation, the file is saved into a user-selected folder, providing an additional layer of security and control.

The system features automatic agent discovery, real-time visual logging, and an Ollama-based AI assistant that provides plain-language explanations of what is happening at each stage of the transfer in real time. The implemented transfer protocol is asynchronous, based exclusively on FIPA-compliant ACL (Agent Communication Language) messages, and follows these main steps:

* **Agent Registration and Discovery:** Each agent registers with the Directory Facilitator under a specific service:
* `file-sender` (`SenderAgent`)
* `file-receiver` (`ReceiverAgent`)
* `file-manager` (`FileManagerAgent`)


* **Transfer Initiation:** `SenderAgent` sends a `REQUEST` message with the ontology `"file-offer"` to `ReceiverAgent`, containing the filename and file size. `ReceiverAgent` responds with:
* `AGREE` – accepts the transfer
* `REFUSE` – refuses the transfer (missing destination directory or explicit user rejection)


* **Chunked Data Transmission:** Upon acceptance, `SenderAgent` breaks the file into 4096-byte blocks, each encoded in Base64. Each chunk is sent as an `INFORM` message with the ontology `"file-chunk"` to `FileManagerAgent`, accompanied by the following parameters:
* `filename`
* `chunk-index`
* `total-chunks`


* **Intermediate Routing:** `FileManagerAgent` receives each chunk and immediately forwards it to `ReceiverAgent` as an `INFORM` message with the ontology `"file-chunk-to-write"`, keeping the identification parameters unchanged.
* **File Reconstruction and Storage:** `ReceiverAgent` temporarily holds the chunks in a concurrency-safe structure. Once all fragments are received, the file is reconstructed and written to disk in the pre-selected directory. Upon completion, `ReceiverAgent` sends an `INFORM` message with the ontology `"file-write-complete"` to `FileManagerAgent`.
* **Initial State Signaling:** On startup, `ReceiverAgent` notifies `SenderAgent` via an `INFORM` message with the ontology `"receiver-ready"`, ensuring transfer initiation only occurs when the recipient is prepared.

---

## What the System Does

* **SenderAgent** → Selects and sends the file
* **ReceiverAgent** → Receives and saves the file (with manual confirmation)
* **FileManagerAgent** → Mediates the chunked transfer
* **ManagerAgent** → Provides a button to shut down the entire system
* **AI Assistant (Ollama + FastAPI)** → Explains current transfer activity in natural language

---

## Minimum Requirements

### Java / JADE

* **JDK 8+** (11 or 17 recommended)
* **JADE 4.6.0** or a recent compatible version

### Python / AI

* **Python 3.9+**
* **Ollama** installed + `llama3` model (or `llama3.1`, `llama3.2`, etc.)
* **Packages:** `fastapi`, `uvicorn`, `ollama`, `pydantic`

---

## Quick Setup

### 1. JADE

1. Download from [https://jade.tilab.com/](https://jade.tilab.com/)
2. Extract anywhere (e.g., `C:\jade` or `~/jade`)

### 2. Ollama

1. Download from [https://ollama.com/download](https://ollama.com/download)
2. After installation, pull the model:
```bash
ollama pull llama3

```



### 3. Python Dependencies

```bash
cd python
python -m venv venv

# Windows
venv\Scripts\activate          

# Linux/macOS
source venv/bin/activate       

pip install fastapi uvicorn ollama pydantic

```

---

## How to Run

### Terminal 1 – AI Assistant

```bash
cd python
uvicorn main:app --host 0.0.0.0 --port 8000

# Or with live reload during development:
# uvicorn main:app --reload

```

*Verify at: `http://localhost:8000/health*`

### Terminal 2 – JADE Platform

```bash
# Inside the directory containing the compiled .class files:

# Windows
java -cp "jade.jar;." agents.MainBoot          

# Linux/macOS
java -cp "jade.jar:." agents.MainBoot          

```

*Alternatively, run the `MainBoot` class directly from your IDE.*

---

## Usage

1. The agent windows and GUIs for both Sender and Receiver will open.
2. **On Receiver GUI:** Select a destination folder.
3. **On Sender GUI:** Select a file and click **"Send"** (*Trimite*).
4. The Receiver will prompt for confirmation to accept the incoming file.
5. **If accepted:** The transfer starts in chunks.
6. Click **"Ask AI"** (*Intreaba ai*) in either GUI to get a real-time natural language explanation of the current transfer status.

---

## Shutdown

Click the **"Shutdown System"** button in the `ManagerAgent` window.

# DESCRIERE - RO

**JADE File Transfer** este un sistem inteligent multi-agent dezvoltat cu JADE, conceput pentru transferul sigur și controlat de fișiere între două stații de lucru. Utilizatorul de la sursă selectează ușor fișierul printr-o interfață grafică simplă, după care acesta este fragmentat în bucăți și transmis prin intermediul unui agent specializat (FileManager) către destinatar. La receptor, fișierul se salvează într-un folder ales de utilizator, după o confirmare explicită, oferind astfel un nivel suplimentar de securitate și control. Sistemul beneficiază de descoperire automată a agenților, logging vizual în timp real și  un asistent AI bazat pe Ollama care explică în limbaj ce se întâmplă în fiecare etapă a transferului, in momentul respectiv.
Protocolul de transfer implementat este unul asincron, bazat exclusiv pe mesaje ACL (Agent Communication Language) conform standardelor FIPA, și implică următorii pași principali:

Înregistrarea și descoperirea agenților
Fiecare agent se înregistrează la Directory Facilitator cu un serviciu specific:
file-sender (SenderAgent)
file-receiver (ReceiverAgent)
file-manager (FileManagerAgent)

Inițierea transferului
SenderAgent trimite un mesaj de tip REQUEST cu ontologia „file-offer” către ReceiverAgent, conținând numele fișierului și dimensiunea acestuia.
ReceiverAgent răspunde cu:
AGREE – acceptă transferul
REFUSE – refuză (lipsă director destinație sau refuz explicit al utilizatorului)

Transmiterea datelor în bucăți
După acceptare, SenderAgent fragmentează fișierul în blocuri de 4096 octeți, fiecare bloc fiind codat în Base64.
Fiecare bucată este expediată ca mesaj INFORM cu ontologia „file-chunk” către FileManagerAgent, însoțit de parametrii:
filename
chunk-index
total-chunks

Rutarea intermediară
FileManagerAgent preia fiecare bucată și o retransmite imediat către ReceiverAgent sub forma unui mesaj INFORM cu ontologia „file-chunk-to-write”, păstrând neschimbați parametrii de identificare.
Reconstituirea și salvarea fișierului
ReceiverAgent stochează temporar bucățile într-o structură concurent-sigură.
La recepționarea tuturor fragmentelor, fișierul este reconstruit și scris pe disc în directorul prealabil selectat.
La finalizare, ReceiverAgent trimite un mesaj INFORM cu ontologia „file-write-complete” către FileManagerAgent.
Semnalizarea stării inițiale
La pornire, ReceiverAgent notifică SenderAgent prin mesaj INFORM cu ontologia „receiver-ready”, permițând astfel inițierea transferului doar atunci când destinatarul este pregătit.

### Ce face sistemul

- SenderAgent → selectează și trimite fișierul
- ReceiverAgent → primește și salvează fișierul (cu confirmare manuală)
- FileManagerAgent → intermediază transferul în bucăți (chunks)
- ManagerAgent → buton de oprire a întregului sistem
- Asistent AI (Ollama + FastAPI) → explică ce se întâmplă cu transferul

### Cerințe minime

**Java / JADE**
- JDK 8+ (recomandat 11 sau 17)
- JADE 4.6.0 sau versiune recentă compatibilă

**Python / AI**
- Python 3.9+
- Ollama instalat + model `llama3` (sau `llama3.1`, `llama3.2` etc.)
- pachete: `fastapi uvicorn ollama pydantic`

### Instalare rapidă

1. JADE  
   Descarcă de pe https://jade.tilab.com/  
   Dezarhivează oriunde (ex: `C:\jade` sau `~/jade`)

2. Ollama  
   https://ollama.com/download  
   După instalare:
   ```
   ollama pull llama3
   ```

3. Dependințe Python
   ```
   cd python
   python -m venv venv
   venv\Scripts\activate          # Windows
   # sau
   source venv/bin/activate       # Linux/macOS

   pip install fastapi uvicorn ollama pydantic
   ```

### Cum pornești totul

**Terminal 1 – Asistentul AI**
```bash
cd python
uvicorn main:app --host 0.0.0.0 --port 8000
# sau cu reload în dev:
# uvicorn main:app --reload
```

Verifică: http://localhost:8000/health

**Terminal 2 – Platforma JADE**
```bash
#  în folderul cu .class-urile compilate
java -cp "jade.jar;." agents.MainBoot          # Windows
# sau
java -cp "jade.jar:." agents.MainBoot          # Linux/macOS
```

Sau rulează direct clasa `MainBoot` din IDE.

### Cum folosești

1. Se deschid ferestrele agenților + GUI-urile Sender și Receiver
2. La Receiver → alege un folder de destinație
3. La Sender → alege un fișier → apasă „Trimite”
4. Receiver întreabă dacă vrei să primești fișierul
5. Dacă accepți → transferul pornește în bucăți
6. Poți apăsa „Intreaba ai” în oricare GUI pentru explicație în limbaj natural a stadiului actual al transmiterii.

### Oprire

Apasă butonul **Shutdown System** din fereastra ManagerAgent.

