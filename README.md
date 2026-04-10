# Teams Meeting Summarizer

A full-stack project for capturing, transcribing, and summarizing **Microsoft Teams** meetings in real-time, with strong **GDPR health data filtering**.  
Built with **Spring Boot 4.x**, **Azure Communication Services (ACS)**, **local Whisper STT**, and **Ollama LLM**, with an **Angular frontend**.

---

## ⭐️ Features

- **Join Teams meetings programmatically** (ACS Call Automation, Teams federation)
- **Capture real-time audio** and push to your backend via WebSocket
- **Local speech-to-text** using [Whisper](https://github.com/openai/whisper) for cost, speed, and privacy
- **Strict GDPR Article 9 health data filter** (regex + LLM pass)
- **Rolling meeting summary with local LLM (Ollama)**
- **Angular 18+ UI:** paste meeting link, view live summaries, GDPR audit
- **No Microsoft Copilot dependency:** runs on your hardware, fully in your control

---

## 🏗️ Architecture Overview

```
          ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
          │ Angular 18+ │────▶│ Spring Boot │────▶│ Ollama LLM  │
          └─────────────┘     │  4.x, Web   │     │ (local API) │
             UI + SSE         │  WebSocket  │     └─────────────┘
                              │  REST,      │
      ┌─────────────┐         │  ACS SDK    │     ┌─────────────┐
      │ MS Teams    │────────▶│             │────▶│ Whisper STT │
      │ Meeting     │  (audio)└─────────────┘     │ (local API) │
      └─────────────┘                             └─────────────┘
```

---

## 🦾 Stack

- **Java 21+ / Spring Boot 4.x / Spring Framework 7 / Jakarta EE 11**
- **Azure Communication Services (ACS)**: Real-time audio
- **Whisper.cpp or faster-whisper** (local deployment): Speech-to-text
- **Ollama** (`llama3` or similar): Summarization
- **Angular 18+** frontend
- Security: All credentials from env vars (never hardcoded), GDPR-by-design

---

## 🚀 Development Phases

1. **Scaffold project:** Maven, Java 21, all dependencies, base config
2. **Configure ACS resource + Teams federation** (see below)
3. **Implement ACS meeting join + audio WebSocket handler**
4. **Buffer/process/convert audio for STT**
5. **Transcribe speech (Whisper) and filter GDPR Article 9 health data**
6. **Ollama Summarization + structured meeting summary**
7. **Angular UI + SSE live summary stream**
8. **GDPR audit trail, error handling, tests**

Full skill breakdown in [SKILLS.md](./SKILLS.md).

---

## 📦 Setup

### Prerequisites

- Azure subscription & admin access
- Azure Communication Services resource (Europe region for GDPR, recommended)
- Microsoft Teams admin access (to enable federation with ACS)
- Whisper (local) — [Install guide](https://github.com/openai/whisper)
- Ollama (`ollama pull llama3`)
- Java 21+, Maven 4+, Node.js 20+ (for Angular UI)
- ngrok or Azure Dev Tunnels for local backend development (<https> access for ACS audio)

---

### 1. Azure Communication Services + Teams Federation

- **Create ACS Resource:** Azure Portal → Create Resource → Communication Services.  
  Save the `connection string` and `resource ID`.
- **Register Azure AD (Entra ID) App:** with `Calls.AccessMedia.All`, `OnlineMeetings.Read.All` perms  
  (see [dev guide](https://learn.microsoft.com/en-us/azure/communication-services/concepts/identity/teams-interop))
- **Configure Teams:**  
  Use PowerShell:
  ```ps
  Install-Module -Name MicrosoftTeams -Force
  Connect-MicrosoftTeams
  $acsResourceId = "<your ACS Resource Immutable ID>"
  Set-CsTeamsAcsFederationConfiguration -EnableAcsUsers $True -AllowedAcsResources @($acsResourceId)
  Set-CsExternalAccessPolicy -Identity Global -EnableAcsFederationAccess $True
  ```
- **Wait up to 24h** for federation changes to take effect

---

### 2. Local Environment

```properties
# Example .env (never commit real values!)
ACS_CONNECTION_STRING=endpoint=https://...;accesskey=...
AZURE_TENANT_ID=...
AZURE_CLIENT_ID=...
AZURE_CLIENT_SECRET=...
WHISPER_ENDPOINT=http://localhost:8080
OLLAMA_BASE_URL=http://localhost:11434
```

---

## ⚙️ Running Locally (Backend)

1. Start ngrok for WebSocket endpoint:
   ```sh
   ngrok http 8080
   ```
   Update your app config to use the new public URL (for ACS callbacks/audio).
2. Run Whisper server locally (`whisper.cpp` or `faster-whisper`).
3. Start Ollama (`ollama serve`).
4. Run Spring Boot backend:
   ```sh
   ./mvnw spring-boot:run
   ```

---

## 🎤 Joining a Meeting

- From Angular UI, paste a Teams join link. The backend joins the meeting as a bot.
- Audio will stream to your backend, transcribed, GDPR-filtered, summarized.
- UI displays real-time rolling summary, action items, and GDPR audit stats.

---

## ⚖️ GDPR & Security

- Health data phrases and codes are **strictly filtered before LLM processing**
- All connection strings and secrets from env variables only
- All sample data is synthetic by default (no real user information should be committed)

---

## 📑 Additional Docs

- [SKILLS.md](./SKILLS.md) — all required capabilities and acceptance tests
- [DEVELOPMENT_PHASES.md](./DEVELOPMENT_PHASES.md) — detailed phased checklist
- [.github/copilot-instructions.md](./.github/copilot-instructions.md) — full coding style and structure rules
- [Prompts for Copilot agent](./.github/prompts/) for each dev phase

---

## 🤝 Contributing

Open as a private/personal research project — not for production GDPR use.  
Feel free to fork, but review [SKILLS.md](./SKILLS.md) and contribute in modular PRs.

---

## 📄 License

MIT License.

---

## 🙋 FAQ

**Q:** Can I use this with other LLMs or STT services?  
**A:** Yes, swap out the HTTP integration.

**Q:** Will this work with other meeting platforms?  
**A:** The back-end pipeline is modular, but ACS integration is strongly Teams-specific.

**Q:** Does it require Microsoft Copilot?  
**A:** No, all intelligence runs locally — on your hardware.
