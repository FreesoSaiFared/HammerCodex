# **HammerCodex (from AutoTeam)**

**Account Rotation and Authentication Sync Tool for ChatGPT Team**

Automatically registers accounts, obtains Codex authentication, rotates seats based on quota limits, and performs two-way synchronization of auth files with [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI).

**Disclaimer**: This project is for educational and research purposes only. Using this tool may violate OpenAI's Terms of Service, including but not limited to automated operations and multi-account management. Users assume all risks, including account bans, IP restrictions, and other potential consequences. The author bears no responsibility for any losses incurred through the use of this tool.

## **Features**

|  | Feature | Description |
| :---- | :---- | :---- |
| 📧 | **Automated Registration** | CloudMail temporary email \+ Playwright automated registration. |
| 🔐 | **Codex OAuth** | Automatically logs into Codex; falls back to email verification codes if no password is set. |
| 🔑 | **Manual OAuth Import** | Supports automatic localhost callbacks, and allows manual pasting of callback URLs. |
| 🔄 | **Smart Rotation** | Automatically removes accounts with insufficient quota and prioritizes reusing old accounts once their quota is reset. |
| ☁️ | **CPA Two-way Sync** | Uploads local active accounts to CPA, and supports reverse importing from CPA. |
| 🖥️ | **Web Dashboard** | Dashboard, Sync Center, OAuth Login, Task History, Logs, Settings. |
| 🔍 | **Background Monitoring** | Scheduled background health checks to monitor quotas and trigger rotation. |
| 📤 | **Export Auth** | One-click export of Codex CLI formatted auth.json to connect directly to OpenAI (bypassing proxies). |
| 🐳 | **Docker Ready** | Supports containerized deployment and data persistence. |

**For first-time users, we highly recommend reading:** [Getting Started Guide](http://docs.google.com/docs/getting-started.md)

## **Quick Start**

### **Installation**

\# Linux  
bash setup.sh  
\# Or manually: uv sync && uv run playwright install chromium

\# Windows / macOS  
uv sync  
uv run playwright install chromium

Supports Linux, Windows, and macOS. Windows/macOS do not require xvfb.

### **Launch**

\# Web Dashboard \+ API (Recommended)  
uv run autoteam api

\# Or run rotation directly  
uv run autoteam rotate

On the first launch, it will automatically guide you to configure CloudMail, CPA, API Keys, and verify connectivity.

### **Docker Deployment**

git clone \[https://github.com/cnitlrt/AutoTeam.git\](https://github.com/cnitlrt/AutoTeam.git) && cd AutoTeam  
mkdir \-p data && cp .env.example data/.env  
\# Edit data/.env to fill in your configuration (or configure it via the Web UI after launch)  
docker compose up \-d

If you are using **Linux \+ Docker** and need the container to access proxies / CloudMail / CPA on the host machine, it is recommended to add the following to your docker-compose.yml:

extra\_hosts:  
  \- "host.docker.internal:host-gateway"

Then, set the host service addresses like this:

PLAYWRIGHT\_PROXY\_URL=socks5://host.docker.internal:3333

For more details, see the [Docker Deployment Documentation](http://docs.google.com/docs/docker.md).

### **CLI Commands**

| Command | Description |
| :---- | :---- |
| api | Starts the Web Dashboard \+ HTTP API (default port 8787). |
| rotate \[N\] | Smart rotation; replenishes the pool to N accounts (default is 5). |
| status | Check account status. |
| check | Check quota usage. |
| add | Add a new account. |
| manual-add | Manual OAuth account addition (opens a link to log in, then paste the callback URL). |
| fill \[N\] | Replenish members up to N. |
| cleanup \[N\] | Clean up excess members. |
| sync | Sync auth files to CPA. |
| pull-cpa | Reverse sync auth files from CPA to local. |
| admin-login | Administrator login. |

For more parameters and endpoint details, see the [API Documentation](http://docs.google.com/docs/api.md).

## **Web Management Dashboard**

After starting with uv run autoteam api, visit http://localhost:8787.

| Page | Functionality |
| :---- | :---- |
| 📊 Dashboard | Account statistics \+ status table \+ login/remove/delete/sync operations. |
| 👥 Team Members | All Team members (including external members). |
| 🔁 Account Pool Operations | Actions that directly alter the account pool state, such as rotation, checking, replenishing, adding, and cleaning up. |
| 🔄 Sync Center | Reconciliation and synchronization actions like syncing accounts to/from CPA. |
| 🔐 OAuth Login | Generates auth links; prioritizes auto-receiving localhost callbacks, with manual URL pasting as a fallback. |
| 📜 Task History | View background task execution status, parameters, execution time, and results. |
| 📋 Logs | Real-time log viewer. |
| ⚙️ Settings | Admin login \+ Main account Codex sync \+ Monitoring configuration. |

## **Documentation**

| Document | Content |
| :---- | :---- |
| [Getting Started](http://docs.google.com/docs/getting-started.md) | Complete guide from scratch, from installation to the first rotation. |
| [Configuration Guide](http://docs.google.com/docs/configuration.md) | .env settings, admin login, and auth file formats. |
| [Docker Deployment](http://docs.google.com/docs/docker.md) | Docker Compose, data persistence, and Web configuration. |
| [API Documentation](http://docs.google.com/docs/api.md) | All HTTP endpoints and usage examples. |
| [Architecture & Workflow](http://docs.google.com/docs/architecture.md) | Rotation workflow, state machine, project structure, and dependencies. |
| [Troubleshooting](http://docs.google.com/docs/troubleshooting.md) | Issues related to installation/login/rotation/Docker/Web Dashboard. |

## **Use Cases**

* Maintaining a fixed number of available Team seats.  
* Synchronizing Codex auth files to CLIProxyAPI.  
* Completing daily rotation, reconciliation, and OAuth imports entirely within a Web dashboard.

## **Known Limitations**

* **IP Risks** — VPS IPs are easily flagged by OpenAI/Cloudflare. Using residential proxies is highly recommended.  
* **Concurrency Limits** — Only one Playwright operation is allowed at a time.  
* **Verification Codes** — OpenAI verification codes have a short lifespan; network latency may cause them to expire.

For more details, see [Troubleshooting](http://docs.google.com/docs/troubleshooting.md).

## **Acknowledgements**

Thanks to the **LinuxDo** community for their support\!

## **Star History**

# **Glossary & Terminology Evaluation**

*This section provides an evaluation and translation rationale for specific niche or technical terms used in the original Chinese text.*

| Chinese Term | English Translation | Evaluation / Explanation |
| :---- | :---- | :---- |
| **账号轮转** | **Account Rotation** | Standard industry term for programmatically switching between different API accounts/keys to bypass rate limits or maintain continuous service. |
| **额度** | **Quota** / **Usage Limit** | In the context of APIs (like OpenAI), "quota" perfectly captures the allocated limits (e.g., token limits, message caps). |
| **席位** | **Seat(s)** | Refers directly to a user slot within a SaaS subscription model (e.g., a "ChatGPT Team" member seat). |
| **双向同步** | **Two-way Sync** | A common software term indicating that data changes (in this case, auth states) flow in both directions between the local tool and the target API (CPA). |
| **Codex 认证** | **Codex Auth / Authentication** | "Auth" or "Authentication" is the correct technical term here, referring to the specific token/cookie extraction format required by the Codex CLI/proxy tool. |
| **自动巡检** | **Background Monitoring** | A literal translation would be "Auto Patrol/Inspection." However, in DevOps/Software, "Background Monitoring" or "Health Checks" natively describes a system periodically checking quotas and triggering actions. |
| **移出** | **Remove (from pool/team)** | Translated as "remove" to signify taking an account out of active rotation or removing a member from the Team seats. |
| **补满** | **Replenish** | A native, accurate term for refilling a pool or list up to a target threshold (N). "Fill" is also acceptable, but "Replenish" carries the exact contextual weight. |
| **主号** | **Main Account** | Refers to the primary or admin account that owns the Team workspace and manages the billing/seats. |
| **对账** | **Reconciliation** | Originally an accounting term, but highly native in system architecture when comparing local states/databases against external services (like CPA) to ensure they match perfectly. |
| **账号池** | **Account Pool** | The collection or inventory of available accounts managed by the tool. |
| **CPA** | **CPA** | Left as an acronym, as the context clearly establishes it as shorthand for CLIProxyAPI. |
