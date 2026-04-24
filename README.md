<div align="center">

<br/>

```
 ⬡  N E S T S H A R E
```

### Encrypted P2P File Sharing for Local Networks

[![Python](https://img.shields.io/badge/Python-3.8%2B-3776ab?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Flask](https://img.shields.io/badge/Flask-3.0-000000?style=flat-square&logo=flask&logoColor=white)](https://flask.palletsprojects.com)
[![SocketIO](https://img.shields.io/badge/Socket.IO-4.x-010101?style=flat-square&logo=socket.io&logoColor=white)](https://socket.io)
[![License](https://img.shields.io/badge/License-MIT-22c97a?style=flat-square)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey?style=flat-square)]()

*Share files securely across your LAN — with real-time presence, transfer consent, and AES-128 encryption at rest.*

<br/>

</div>

---

## Overview

**NestShare** is a self-hosted, browser-based file sharing application designed for local networks. Every file is encrypted before it touches disk. Users can upload files to their personal encrypted vault, share files directly with online peers who must explicitly accept the transfer, and monitor all activity through a real-time transfer history log.

It runs entirely on your machine — no cloud, no third-party servers, no data leaving your network.

<br/>

## Screenshots

| Login | Dashboard — My Files | Send File | Incoming Transfer |
|-------|---------------------|-----------|------------------|
| Animated auth page with bcrypt login | File grid with Download · Share · Delete | Online user picker with real-time presence | Push notification modal with Accept / Decline |

> **Responsive:** Full bottom-navigation layout on mobile, sidebar layout on desktop.

<br/>

## Features

### 🔐 Security
- **bcrypt password hashing** — 12 rounds with per-password random salt; plaintext passwords are never stored or logged
- **AES-128 Fernet encryption at rest** — every file is encrypted before being written to disk; decryption only happens in memory at download time
- **Per-user vault isolation** — each user's files are stored in a separate directory; server enforces ownership on every request
- **Transfer holding area** — shared files are held encrypted in a temporary area until the recipient explicitly accepts; rejected transfers are permanently deleted
- **Server-side sessions** — Flask sessions backed by a cryptographically random 32-byte secret key

### 📤 File Operations
- Drag & drop or click-to-browse multi-file upload
- Per-file XHR upload with live progress bars
- One-click authenticated download (decrypted on the fly, served in memory)
- Instant delete from personal vault
- **Share button on every file** — send any uploaded file directly to an online peer without re-uploading

### 👥 Real-Time Presence
- Live online/offline status for all users via WebSocket
- Presence updates instantly broadcast on connect and disconnect
- Online user count badge in sidebar navigation

### 📬 Peer-to-Peer Transfer Flow
- Sender picks a file and selects a recipient from the online users list
- Recipient receives an **instant push notification** with sender name, filename, and file size
- Recipient clicks **Accept** → file moves encrypted into their vault; **Decline** → file is permanently deleted
- Sender receives real-time feedback on the outcome
- Full transfer history with status tracking (pending / accepted / rejected)

### 📱 Responsive UI
- Desktop: fixed sidebar navigation
- Mobile: sticky top header + bottom tab bar with floating Send action button
- Consistent experience across all screen sizes

<br/>

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend framework | [Flask 3.x](https://flask.palletsprojects.com/) |
| Real-time communication | [Flask-SocketIO 5.x](https://flask-socketio.readthedocs.io/) + [Socket.IO 4.x](https://socket.io/) |
| Async server | [eventlet](https://eventlet.net/) |
| File encryption | [cryptography — Fernet (AES-128-CBC + HMAC-SHA256)](https://cryptography.io/) |
| Password hashing | [bcrypt](https://pypi.org/project/bcrypt/) |
| Frontend fonts | [Outfit](https://fonts.google.com/specimen/Outfit) + [Fira Code](https://fonts.google.com/specimen/Fira+Code) |
| Templating | Jinja2 |

<br/>

## Getting Started

### Prerequisites

- **Python 3.8+**
- **pip** (comes with Python)
- A Linux machine on your local network (tested on Kali Linux; works on Ubuntu, Debian, macOS)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/nestshare.git
cd nestshare

# 2. Make the launch script executable
chmod +x start.sh

# 3. Start the server
./start.sh
```

The script will automatically:
- Create a Python virtual environment (`.venv/`)
- Install all dependencies from `requirements.txt`
- Generate cryptographic keys on first run (`.secret_key`, `.flask_secret`)
- Print your local network URL

### Accessing the App

```
Local machine:  http://127.0.0.1:5000
Other devices:  http://<your-machine-ip>:5000
```

> **Find your IP on Linux:** `hostname -I | awk '{print $1}'`

Open the URL in any browser on any device connected to the same Wi-Fi or LAN. Register an account and start sharing.

<br/>

## Project Structure

```
nestshare/
│
├── app.py                  # Flask application — routes, SocketIO events, encryption logic
├── requirements.txt        # Python dependencies
├── start.sh                # One-command setup and launch script
│
├── templates/
│   ├── base.html           # Shared HTML head, fonts, viewport meta
│   ├── login.html          # Authentication page
│   ├── register.html       # Account creation page
│   └── dashboard.html      # Main app — all views (Upload, Files, Send, Users, History)
│
├── static/
│   ├── css/
│   │   └── style.css       # Full responsive stylesheet (mobile-first)
│   └── js/
│       └── dashboard.js    # All client-side logic — upload, socket events, modals
│
├── uploads/                # Encrypted user vaults (auto-created)
│   ├── <username>/         # One directory per user, all files AES-encrypted
│   └── ...
│
├── temp_transfers/         # Encrypted holding area for pending transfers (auto-created)
├── inbox/                  # Reserved (auto-created)
│
├── .secret_key             # AES master key — auto-generated, chmod 600, NEVER commit
├── .flask_secret           # Flask session key — auto-generated, chmod 600, NEVER commit
├── users.json              # User accounts with bcrypt-hashed passwords
└── transfers.json          # Transfer audit log
```

<br/>

## How It Works

### Encryption Model

```
Upload flow:
  Raw file bytes  →  Fernet.encrypt(MASTER_KEY)  →  Written to disk

Download flow:
  Read from disk  →  Fernet.decrypt(MASTER_KEY)  →  Served in memory  →  Browser
```

The master key lives in `.secret_key` (permissions: `600`). It is never transmitted over the network. Without this file, encrypted files on disk are unreadable.

### P2P Transfer Flow

```
  Sender                        Server                      Receiver
    │                             │                             │
    ├── Select file + recipient ──▶                             │
    │                             │ Copy encrypted blob         │
    │                             │ to temp_transfers/<tid>     │
    │                             │                             │
    │                             ├── socket: transfer_request ─▶
    │                             │                        [Modal popup]
    │                             │                             │
    │                             │          Accept ────────────┤
    │                             │ Move blob to receiver vault │
    │◀── socket: transfer_response┤                             │
    │         (accepted/declined) │          Decline ───────────┤
    │                             │ Delete blob from temp/      │
```

> Files are **never decrypted** during a transfer. The encrypted blob is moved directly between directories — the server acts as a router, not a reader.

### Authentication Flow

1. User registers → password hashed with `bcrypt.hashpw(pw, bcrypt.gensalt(rounds=12))`
2. User logs in → `bcrypt.checkpw(pw, stored_hash)` — timing-safe comparison
3. Session cookie set server-side with a random 32-byte Flask secret
4. All API endpoints and file downloads require an active session

<br/>

## API Reference

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `POST` | `/login` | Authenticate user | ✗ |
| `POST` | `/register` | Create new account | ✗ |
| `GET` | `/logout` | Clear session | ✓ |
| `GET` | `/api/files` | List own vault files | ✓ |
| `POST` | `/api/upload` | Upload files to vault | ✓ |
| `DELETE` | `/api/delete/<filename>` | Delete file from vault | ✓ |
| `GET` | `/download/<filename>` | Download + decrypt file | ✓ |
| `GET` | `/api/online-users` | List currently online users | ✓ |
| `GET` | `/api/all-users` | List all users with online status | ✓ |
| `POST` | `/api/transfer/initiate` | Send uploaded file to user | ✓ |
| `POST` | `/api/transfer/initiate-from-vault` | Share vault file with user | ✓ |
| `POST` | `/api/transfer/respond` | Accept or decline transfer | ✓ |
| `GET` | `/api/transfer/history` | Get transfer audit log | ✓ |

### SocketIO Events

| Direction | Event | Payload |
|-----------|-------|---------|
| Server → Client | `presence_update` | `[{username, display}, ...]` — current online users |
| Server → Client | `transfer_request` | `{tid, sender, sender_display, filename, size_hr}` |
| Server → Client | `transfer_response` | `{tid, filename, receiver, receiver_display, accepted}` |

<br/>

## Configuration

All configuration is at the top of `app.py`:

```python
MAX_MB        = 500     # Maximum file upload size in megabytes
BCRYPT_ROUNDS = 12      # bcrypt work factor — increase for more security (slower login)
```

To allow only specific file types, set `ALLOWED_EXT` in `app.py`:

```python
ALLOWED_EXT = {'pdf', 'png', 'jpg', 'zip', 'docx'}  # None = allow all
```

<br/>

## Security Considerations

> NestShare is designed for **trusted local networks** (home, lab, office LAN). It is not hardened for exposure to the public internet without additional measures.

**What is protected:**
- ✅ Passwords never stored or logged in plaintext
- ✅ Files unreadable on disk without the master key
- ✅ Users can only access their own files
- ✅ File transfers require explicit recipient consent
- ✅ Filenames sanitized via `werkzeug.utils.secure_filename`
- ✅ Sessions invalidated on logout

**Recommendations for hardened deployments:**
- Put behind **nginx** as a reverse proxy with **HTTPS** (Let's Encrypt or self-signed cert)
- Restrict access with firewall rules (`ufw allow from 192.168.x.0/24`)
- Back up `.secret_key` securely — losing it means losing access to all encrypted files
- Run as a non-root user with minimal filesystem permissions
- Do **not** commit `.secret_key`, `.flask_secret`, or `users.json` to version control (already in `.gitignore`)

<br/>

## .gitignore

Make sure your `.gitignore` includes:

```gitignore
# Secret keys — NEVER commit these
.secret_key
.flask_secret

# User data
users.json
transfers.json
uploads/
inbox/
temp_transfers/

# Python
.venv/
__pycache__/
*.pyc
*.pyo
*.egg-info/

# OS
.DS_Store
Thumbs.db
```

<br/>

## Dependencies

```
flask>=3.0.0
flask-socketio>=5.3.6
cryptography>=42.0.0
bcrypt>=4.1.0
python-engineio>=4.9.0
python-socketio>=5.11.0
eventlet>=0.35.0
```

Install manually:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

<br/>

## Roadmap

- [ ] HTTPS support via built-in self-signed certificate generation
- [ ] Admin panel — manage users, view all transfers
- [ ] File expiry — auto-delete files after configurable number of days
- [ ] Transfer size limit per user (quota system)
- [ ] End-to-end encryption (client-side key exchange)
- [ ] Dark/light theme toggle
- [ ] File preview for images and PDFs
- [ ] Docker support

<br/>

## Contributing

Contributions are welcome. To contribute:

1. Fork the repository
2. Create a feature branch — `git checkout -b feature/your-feature`
3. Commit your changes — `git commit -m 'Add: your feature description'`
4. Push to your branch — `git push origin feature/your-feature`
5. Open a Pull Request

Please ensure any new routes are protected with the `@login_required` decorator, and that any file operations use `secure_filename()`.

<br/>

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

```
MIT License — Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software to use, copy, modify, merge, publish, and distribute it,
subject to the conditions in the LICENSE file.
```

<br/>

## Author

Built with Flask, SocketIO, and AES-128 encryption.  
Designed for secure, consent-based local file sharing — no cloud required.

---

<div align="center">

**[⬆ Back to top](#)**

*If this project helped you, consider giving it a ⭐ on GitHub.*

</div>
