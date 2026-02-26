# SSH Server in Python
> M1 Logiciels Sûrs — Projet Programmation Réseau  
> Sovanna Tan — Janvier 2026  
> Defense: **27 March 2026**

---

## 👥 Team

| Member | Role | Responsibilities |
|--------|------|-----------------|
| Arslan LARBI | Network Layer | TCP server, epoll event loop (`server.py`, `config.py`) |
| Chaker YOUSFI | SSH Protocol | Paramiko handshake, cryptographic keys (`ssh_handler.py`, `keys/`) |
| Malek RAIS | Session Layer | Authentication, shell session (`auth.py`, `shell.py`) |

---

## 🎯 Project Goal

Build an SSH server in Python that:
- Uses **Paramiko** for the SSH protocol (encryption, key exchange, authentication)
- Uses **epoll** for non-blocking I/O to handle multiple clients simultaneously
- Works with standard SSH clients like `ssh` (Linux/macOS) and **PuTTY** (Windows)

---

## 🗂️ Project Structure

```
ssh_server/
│
├── server.py          # Entry point — TCP socket + epoll event loop
├── ssh_handler.py     # Paramiko SSH negotiation per client connection
├── auth.py            # Authentication logic (password and/or public key)
├── shell.py           # Shell session handler (spawns PTY, runs commands)
├── config.py          # Configuration: port, users, host key path, settings
├── keys/
│   └── server_rsa     # RSA host key (generated once, never committed to git)
├── requirements.txt   # Python dependencies
└── README.md          # This file
```

---

## ⚙️ Architecture Overview

```
[Client: ssh / PuTTY]
        |
        | TCP connection (port 2222)
        v
[server.py — epoll loop]
        |
        |-- new connection  --> create SSHHandler(client_socket)
        |-- data ready      --> handler.handle_data()
        |-- disconnect      --> cleanup handler
        v
[ssh_handler.py — Paramiko Transport]
        |
        |-- SSH handshake (key exchange, encryption negotiation)
        |-- Authentication --> auth.py
        |-- Channel open   --> shell.py (PTY + subprocess)
```

**Key design principle:**  
`epoll` watches the raw TCP sockets at the OS level. Paramiko handles everything inside each connection (encryption, channels, protocol messages). This allows multiple concurrent clients without threads.

---

## 🔀 Git Workflow

```
main              ← stable, demo-ready code only
└── dev           ← integration branch (merge here when a feature works)
    ├── feature/epoll-server        (Arslan)
    ├── feature/ssh-handshake       (Chaker)
    └── feature/auth-shell          (Malek)
```

**Rules:**
- Never push directly to `main`
- Open a Pull Request from your feature branch → `dev`
- Merge `dev` → `main` only at milestones
- If you use code found online, **keep the copyright notice** in the file

---

## 📅 Development Timeline

| Week | Dates | Goal | Owner |
|------|-------|------|-------|
| 1 | Feb 26 | Repo setup, environment, generate host key, basic TCP socket | All |
| 2 | Feb 26– | epoll loop handles multiple connections | Arslan |
| 3 | Feb 26– | Paramiko SSH handshake working | Chaker |
| 4 | Feb 26 –  | Password auth + shell session | Malek |
| 5 | Mar 20 | Integration — all parts working together | All |
| 6 | Mar 20 | Multi-client tests on local network | All |
| 7 | Mar 20 | Polish, README, prepare demo & slides | All |
| **8** | **Mar 27** | **🎤 Defense** | **All** |

---

## 🛠️ Setup & Installation

### Prerequisites
- Python 3.10+
- Linux (epoll is Linux-only)
- pip

### Install dependencies

```bash
pip install -r requirements.txt
```

### Generate the server host key (do this once)

```bash
ssh-keygen -t rsa -b 2048 -f keys/server_rsa -N ""
```

> ⚠️ Never commit `keys/server_rsa` (private key) to git. Add it to `.gitignore`.

### Run the server

```bash
python server.py
```

Server listens on port **2222** by default.

### Connect with a standard client

```bash
ssh -p 2222 user@localhost
```

---

## 📦 Dependencies (`requirements.txt`)

```
paramiko>=3.0.0
```

---

## 🔐 Cryptography & Keys

- The server uses an **RSA host key** to identify itself to clients
- On first connection, the client will be asked to trust the server's fingerprint
- Authentication supports:
  - **Password** — checked against `config.py` user list
  - **Public key** *(optional extension)* — checked against `~/.ssh/authorized_keys`

---

## 🧪 Testing

### Single client test
```bash
ssh -p 2222 -o StrictHostKeyChecking=no testuser@localhost
```

### Multi-client test (local network)
Run the server on one machine, connect from 2+ machines on the same LAN:
```bash
ssh -p 2222 user@<server-ip>
```

---

## 🎤 Defense Checklist (27 March 2026)

- [ ] **Member 1** — Present epoll architecture, show how multiple clients are handled simultaneously
- [ ] **Member 2** — Present Paramiko, explain RSA host key, key exchange, encryption negotiation
- [ ] **Member 3** — Present authentication flow, explain PTY and shell spawning
- [ ] **Demo** — Connect from 2+ machines on a local Ethernet network using standard `ssh` clients
- [ ] **Code review ready** — clean code, comments, copyright notices on borrowed code

---

## 📚 References

- [Paramiko Documentation](https://docs.paramiko.org)
- [Paramiko Demo Server (official example)](https://github.com/paramiko/paramiko/blob/main/demos/demo_server.py)
- *Python Network Programming Cookbook, 2nd Edition* — Kathiravelu & Sarker, Packt, 2017
- [Python `select.epoll` docs](https://docs.python.org/3/library/select.html#epoll-objects)
- [RFC 4251 — SSH Protocol Architecture](https://www.rfc-editor.org/rfc/rfc4251)

---

## 📝 Notes

- `epoll` is **Linux-only**. Development on macOS/Windows requires a Linux VM or WSL.
- The server runs on port **2222** (not 22) to avoid needing root privileges.
- All code borrowed from the internet must retain its original copyright header.