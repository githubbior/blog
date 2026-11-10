# Local AI Security & Hardening Guide (Ollama)

> **Overview:** Running local Large Language Model (LLM) runners like Ollama brings great privacy and control, but default configurations and unpatched versions can introduce severe security risks. This guide summarizes common vulnerabilities and provides a step-by-step audit and hardening workflow.

---

## ⚠️ Key Security Risks in Local AI

1. **Exposed Network Binding (Unauthenticated Access)**
   * **Issue:** By default or due to misconfiguration, local AI services may bind to `0.0.0.0` or `::` (IPv6), exposing port `11434` to any device on the local network or public Wi-Fi.
   * **Impact:** Attackers can query your models, consume GPU/CPU resources, or exploit unauthenticated endpoints.

2. **Memory Leak Vulnerabilities ("Bleeding Llama" - CVE-2026-7482)**
   * **Issue:** Malicious model files or crafted prompts can trick unpatched servers into reading unallocated memory.
   * **Impact:** Leakage of sensitive system memory, active API keys, environment variables, and chat histories.

3. **Remote Code Execution / Path Traversal ("Problem Llama" - CVE-2024-37032)**
   * **Issue:** Path traversal bugs in early versions allowed unauthorized file writes to the host filesystem.
   * **Impact:** Arbitrary code execution (RCE) and full machine compromise.

4. **Shadow AI**
   * **Issue:** Unmonitored or unauthorized installations of local LLM tools on personal or corporate machines create blind spots for security scanners.

---

## 🔍 Audit & Remediation Step-by-Step

### Step 1: Check Network Binding

Verify if your service is exposed to external interfaces:

**PowerShell (Windows):**
```powershell
Get-NetTCPConnection -LocalPort 11434
```

**Terminal (macOS / Linux):**
```bash
netstat -an | grep 11434
# OR
ss -tuln | grep 11434
```

* ❌ **Vulnerable:** `LocalAddress` shows `0.0.0.0` or `::`.
* ✅ **Secure:** `LocalAddress` shows strictly `127.0.0.1` or `::1`.

---

### Step 2: Enforce Loopback Binding (`127.0.0.1`)

Restrict Ollama to respond only to requests originating from the local machine.

#### On Windows:
1. Quit Ollama from the System Tray (near the clock).
2. Open **Environment Variables** settings.
3. Ensure `OLLAMA_HOST` is set to `127.0.0.1:11434` in both **User** and **System** variables (remove any `0.0.0.0` entries).
4. Restart your terminal and launch Ollama again.

#### On macOS / Linux:
Add the following line to your `~/.zshrc` or `~/.bashrc`:
```bash
export OLLAMA_HOST="127.0.0.1:11434"
```
Then reload the configuration:
```bash
source ~/.zshrc
```

---

### Step 3: Verify Model Integrity and Provenance

Always verify that installed models originate from official or trusted repositories.

1. **List installed models:**
   ```bash
   ollama list
   ```

2. **Inspect model metadata and manifest:**
   ```bash
   ollama show <model_name>
   ```

* **Official Library Models:** Names like `llama3`, `qwen2.5-coder`, `deepseek-r1`, `nomic-embed-text` pull directly from the verified [Ollama Library](https://ollama.com/library).
* **Hugging Face Models:** Ensure the prefix matches verified user/organization accounts (e.g., `hf.co/Meta-Llama/...`).

---

### Step 4: Keep Software Updated

Regularly re-install or update Ollama from the official website ([ollama.com](https://ollama.com)) to ensure all critical patches for CVEs and memory leaks are applied.

---

## 🛡️ Quick Security Checklist

- [x] Service bound strictly to `127.0.0.1:11434`.
- [x] Application updated to the latest stable release.
- [x] All local models verified against official library manifests.
- [x] Sensitive cloud API keys rotated if port was previously exposed on public Wi-Fi.

### Sources:

<a href="https://www.youtube.com/watch?v=teybjHUZ-gk" target="_blank">
  <img src="https://img.youtube.com/vi/teybjHUZ-gk/0.jpg" alt="Your Local AI Is Actually Not Safe">
</a>
