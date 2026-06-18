# Agent Skills Collection 🧠

> **Repo:** [github.com/akbarhlubis/agent-skills](https://github.com/akbarhlubis/agent-skills)

Kumpulan skills untuk AI coding agent (Zed, Codex, Cursor, dll). Setiap skill berisi instruksi terstruktur yang membantu agent menyelesaikan tugas spesifik dengan lebih akurat.

## 📁 Daftar Skills

| Skill | Deskripsi |
|-------|-----------|
| `ekinerja` | Automasi absensi & task E-Kinerja PaperlessHospital via Chrome DevTools MCP |
| `clinical-workflow-ux-audit` | Audit UX untuk workflow klinis, SIMRS, farmasi, billing |
| `code-review-and-quality` | Multi-axis code review sebelum merge |
| `context-engineering` | Optimasi agent context untuk sesi baru |
| `debugging-and-error-recovery` | Systematic root-cause debugging |
| `frontend-ui-engineering` | Build production-quality UI components |
| `planning-and-task-breakdown` | Breakdown pekerjaan menjadi task terurut |
| `security-and-hardening` | Hardening keamanan (input, auth, storage) |
| `hypothesis-driven-testing` | Scientific method for debugging — design test, verify, analyze root cause |
| `simrs-reference-auditor` | Audit implementasi SIMRS vs referensi upstream |
| `laporan-bulanan-ekinerja` | Generate laporan bulanan kinerja dari E-Kinerja ke Google Docs |

## 🚀 Cara Pakai

### Clone repo

```bash
git clone https://github.com/akbarhlubis/agent-skills.git ~/.agents/skills
```

### Zed

Skills otomatis terbaca dari `~/.agents/skills/`. Tambahkan juga MCP servers di Zed config (`settings.json`):

```json
{
  "agent": {
    "skills_directory": "~/.agents/skills"
  },
  "mcp_servers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest"]
    },
    "github": {
      "url": "https://api.githubcopilot.com/mcp/",
      "bearer_token_env_var": "GITHUB_PAT"
    }
  }
}
```

### Codex

Copy atau symlink skill ke `~/.codex/skills/`. Tambahkan MCP servers di `~/.codex/config.toml`:

```toml
[mcp_servers.chrome-devtools]
command = "npx"
args = ["chrome-devtools-mcp@latest"]

[mcp_servers.github]
url = "https://api.githubcopilot.com/mcp/"
bearer_token_env_var = "GITHUB_PAT"

[mcp_servers.dbhub]
command = "npx"
args = ["-y", "@bytebase/dbhub@latest", "--transport=stdio", "--config=/path/to/dbhub.toml"]
```

### Symlink Codex ↔ Zed (Windows PowerShell)

```powershell
New-Item -ItemType Junction -Path 'C:\Users\{USER}\.codex\skills\ekinerja' -Target 'C:\Users\{USER}\.agents\skills\ekinerja'
```

### Cursor
Copy ke `.cursorrules` atau project rules.

## 🔧 MCP Servers yang Direkomendasikan

| MCP Server | Command | Kegunaan |
|------------|---------|----------|
| **chrome-devtools** | `npx chrome-devtools-mcp@latest` | `navigate`, `evaluate`, `screenshot` — automasi browser |
| **github** | via GitHub Copilot API | `create_repo`, `push_files`, PR management |
| **gitlab** | `bunx @modelcontextprotocol/server-gitlab` | GitLab API — commits, MR, issues |
| **dbhub** | `npx @bytebase/dbhub@latest` | MySQL/PostgreSQL query (read-only recommended) |

> ⚠️ **Jangan commit token!** Simpan di environment variable atau file config lokal. Gunakan placeholder di contoh publik.

## 🏗️ Struktur Skill

```
namaskill/
├── SKILL.md          # Instruksi utama (wajib)
├── .env.example      # Template environment variables (opsional)
├── .gitignore        # Ignore .env & secrets
└── agents/           # Sub-agent instructions (opsional)
```

## ⚠️ Keamanan

- File `.env` **TIDAK PERNAH** di-commit
- Gunakan `.env.example` sebagai template
- Credentials disimpan lokal saja