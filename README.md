# Agent Skills Collection 🧠

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
| `simrs-reference-auditor` | Audit implementasi SIMRS vs referensi upstream |

## 🚀 Cara Pakai

### Zed / Codex
Copy folder skill ke `~/.agents/skills/`

### Cursor
Copy ke `.cursorrules` atau project rules

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
