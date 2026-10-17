

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
