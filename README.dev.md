# **AutoTeam: Under the Hood**

**Developer & Architecture Guide for the AutoTeam Subsystem**

This document serves as the technical reference for AutoTeam. It breaks down the internal architecture, implementation specifics, and integration pathways (including CLI and MCP/Agent wrappers) for developers looking to fork, extend, or deeply integrate this tool into broader automation pipelines.

## **🏗 System Architecture Overview**

AutoTeam operates as a hybrid CLI and Web Service. It manages state via local JSON datastores (data/accounts.json, data/auth.json) to maintain portability and avoid database overhead.

The system is divided into three primary layers:

1. **The Automation Engine (src/autoteam/account\_ops.py, codex\_auth.py)**: Asynchronous Playwright scripts that interact with the OpenAI DOM and API endpoints.  
2. **The State Manager (src/autoteam/manager.py)**: A state machine that evaluates account quotas, manages cooldowns, and orchestrates pool replenishment.  
3. **The API & Presentation Layer (src/autoteam/api.py, web/)**: A FastAPI backend serving REST endpoints and a Vue 3 Single Page Application (SPA) compiled via Vite.

## **⚙️ Core Feature Implementations**

### **1\. Automated Registration Engine**

* **How it works**: The registration flow is entirely asynchronous, driven by playwright.async\_api.  
* **Implementation Details**:  
  * The system utilizes src/autoteam/cloudmail.py to interface with a temporary email provider's REST API. It requests a new inbox and polls for incoming OTP (One Time Password) emails.  
  * In account\_ops.py (register\_account), Playwright spawns a headless Chromium context. It navigates the OpenAI signup flow, injecting the generated CloudMail address.  
  * When OpenAI triggers an email verification, the Playwright context pauses while the cloudmail module polls for the regex-matched OTP code, which is then injected back into the DOM.

### **2\. Codex OAuth & Token Interception**

* **How it works**: AutoTeam bypasses standard UI interactions where possible by intercepting network traffic to capture OAuth tokens.  
* **Implementation Details**:  
  * codex\_auth.py initiates a login sequence. If an account relies on passwordless login, it triggers the email code flow, similar to registration.  
  * Instead of parsing the final DOM state, Playwright attaches listeners to the browser context's routing/network events. It specifically watches for the auth0 callback URL.  
  * Once the callback is intercepted, the underlying state and authorization codes are extracted directly from the query parameters, bypassing the need to render the final success page.

### **3\. Smart Rotation & Quota Management**

* **How it works**: A background worker monitors token usage and rotates accounts in/out of the primary Team workspace based on hard limits.  
* **Implementation Details**:  
  * **Quota Checking**: The chatgpt\_api.py module uses the captured session tokens (specifically the \_\_Secure-next-auth.session-token) to query the OpenAI internal /api/auth/session and workspace endpoints.  
  * **The State Machine (manager.py)**: The Manager.rotate() function acts as the orchestrator. It evaluates the current active accounts. If an account is flagged as depleted, it is moved to a "cooldown" state and removed from the workspace via an API call.  
  * **Replenishment**: The manager queries the local pool (accounts.py). It sorts available accounts by their cooldown\_until timestamp, prioritizing accounts that have passed their cooldown window. If the pool is empty, it invokes the Registration Engine to synthesize new accounts.

### **4\. CLIProxyAPI (CPA) Two-Way Sync**

* **How it works**: Ensures the local auth.json state perfectly mirrors a remote proxy gateway.  
* **Implementation Details**:  
  * Handled by src/autoteam/cpa\_sync.py. This module translates the internal account representations into the specific schema required by the Codex CLI format.  
  * **Push**: Iterates through active local accounts, formulates the required payload (combining access tokens, refresh tokens, and session keys), and issues a POST request to the CPA endpoint.  
  * **Pull**: Issues a GET to the CPA endpoint, parses the remote auth.json, and reconciles it against the local accounts.json, adding or updating entries where the remote source is considered the source of truth.

### **5\. Web Dashboard & Task Concurrency**

* **How it works**: A reactive frontend communicating with an async Python backend, designed to prevent race conditions during browser automation.  
* **Implementation Details**:  
  * **FastAPI Backend**: api.py exposes RESTful routes. Because Playwright operations are heavily stateful and state-locking, the backend utilizes a custom task queue/lock mechanism to ensure only one automation task (registration/rotation) occurs concurrently.  
  * **Vue 3 Frontend**: Built with the Composition API and Vite (web/src/). It frequently polls the FastAPI /api/status and /api/tasks endpoints to render real-time state changes and stream log outputs generated by Python's logging module (redirected to an in-memory buffer or file accessible by the API).

## **🤖 Integration with Local Coding Agents & MCP**

For engineers running local coding agents or utilizing Model Context Protocol (MCP) architectures, AutoTeam is designed to be easily wrapped and invoked programmatically.

### **Wrapping AutoTeam as an MCP Server**

Because AutoTeam already exposes a comprehensive set of functions via autoteam.\_\_main\_\_ and autoteam.api, you can easily build an MCP interface over it using tools like integuru or native agent protocols:

1. **Expose CLI Tools**: The autoteam rotate, autoteam check, and autoteam sync commands return standard exit codes. Your coding agent can invoke these as standard shell tools.  
2. **REST API as Tool Call**: Start the API (uv run autoteam api) on a designated port. Your coding agent can issue POST /api/rotate or GET /api/status requests natively, parsing the structured JSON responses to make autonomous decisions about account availability.  
3. **Headless Python Import**:  
   You can directly import AutoTeam's manager into a custom Python script or MCP server wrapper:  
   from autoteam.manager import Manager  
   import asyncio

   async def custom\_agent\_rotation():  
       manager \= Manager()  
       \# Agentistically evaluate if rotation is needed  
       status \= await manager.status()  
       if status\['active\_count'\] \< status\['target'\]:  
           await manager.rotate(target=status\['target'\])  
           await manager.sync\_cpa()  
           return "Rotation complete and synced."  
       return "Quota healthy."

## **🛠 Local Development Setup**

To modify the frontend or core automation logic, set up your dev environment as follows:

**1\. Python Backend Setup**

AutoTeam utilizes uv for ultra-fast dependency resolution.

\# Sync dependencies and create venv  
uv sync

\# Install Playwright browser binaries  
uv run playwright install chromium

\# Run the FastAPI server in dev mode (hot reloading)  
uv run uvicorn src.autoteam.api:app \--reload \--port 8787

**2\. Vue 3 Frontend Setup**

The frontend is located in the web/ directory.

cd web/  
npm install

\# Start Vite dev server with Hot Module Replacement (HMR)  
npm run dev

*Note: In development, Vite is configured (web/vite.config.js) to proxy /api requests to http://localhost:8787 to bypass CORS issues.*

**3\. Running Tests**

Tests are located in tests/unit/.

uv run pytest tests/  
