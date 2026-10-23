# BSGOCore
- This is a personal project
- This repository is neither sponsored nor endorsed by Bigpoint
- The owner of this repository is not responsible for any damages caused by this software
- The owner of this repository is not responsible for other people's actions (such as forks)
- The server was created for educational purposes only
- **The server is a version of the original BSGO 2.0 Resurrection project with minor changes**

### Contributing
**Contributions are welcome!  
Please feel free to open issues for bugs and feature requests, or submit pull requests with improvements.**

## Repository Notes
- The commit history has been reset to provide a clean and maintainable starting point.
- Earlier iterations of this project contained non-essential artifacts, so the history was pruned to avoid confusion.


---
## Introduction
BSGOCore is an open-source game server emulator designed for hosting massively multiplayer online games (MMOPGs).
It is based on the discontinued game Battlestar Galactica Online (BSGO) and seeks to recreate the gameplay experience of the original game as well as provide additional features or change the existing behaviour.

---

The server is developed in Java (JDK 21) with Quarkus and built using Maven. SQLite is used as the default database, with schemas and migrations managed via Flyway.

This implementation is intentionally kept simple and pragmatic: it works and provides the needed functionality, but it is still in an early stage and not yet a polished solution. The database integration, for example, is rudimentary — data is buffered and written back periodically, resulting in behavior that is closer to eventual consistency rather than strict consistency. The current setup was designed to enable faster development and iteration rather than to represent a final or optimal architecture.

---

## [Youtube Video showcase](https://youtu.be/0n3tcONcx90)

## **moved**: [Features](docs/FEATURES.md)

---

## Local setup on Windows

### Requirements
- Windows with PowerShell and JDK 21. Confirm Java is available with `java -version`.
- Bash and OpenSSL to generate the local HTTPS certificate. Git for Windows provides Bash; make sure `openssl` is also available in that shell.
- A copy of the BSGO client if you want to connect to the server. Game files are not included.

The Maven wrapper is included, so a separate Maven installation is not needed. On first use it downloads Maven 3.9.9 and the project dependencies; internet access is required unless they are already cached. The project starts its own game and legacy login listeners, so a separate login server is not needed for this local test. A chat server is optional; set `GAMESERVER_CHAT_SERVER_PORT=0` in `.env` to skip the chat connection when no chat server is running.

### Prepare the project

Run these commands in PowerShell from the `BSGOCore_test-main` project root:

```powershell
Copy-Item .env.example .env
New-Item -ItemType Directory -Force -Path sqlite, observability/logs
```

Edit `.env` and set `CLIENT_PATH` to the full path of your local `bsgo.exe`. Keep the path free of spaces because the current launcher does not quote it inside the `+cdn` argument. Set `GAMESERVER_IGNORE_HASHES=true` for local testing. If no chat server is available, also set `GAMESERVER_CHAT_SERVER_PORT=0`.

Generate the HTTPS certificate and key from the `certs` directory:

```powershell
Push-Location .\certs
bash ./generate-certs.sh
Pop-Location
```

This creates `certs/cert.pem` and `certs/key.pem`, matching the paths in `.env.example`. The supplied certificate configuration covers `localhost` and `127.0.0.1`; if clients connect using another IP or hostname, add it to `certs/san.cnf` and regenerate the certificate. The certificate is for local development.

The required game data is included under `ServerConfigurationUtils/global`, including `AugmentTemplates`, `ColliderTemplates`, `JsonCards`, `LootTemplates`, `MissionTemplateConfiguration`, `SectorTemplates`, `ShipConfigTemplates`, and `ZoneTemplates`.

### Start the server and client

From the project root, start the server:

```powershell
.\mvnw.cmd clean quarkus:dev
```

Flyway applies the database migrations at startup. Wait for the log message `Server started!`, then launch the client in another PowerShell window:

```powershell
.\runclient.bat 127.0.0.1 en
```

The first argument is the server address; the language is optional and defaults to `en`. Use `127.0.0.1` on the same computer as the server. For clients on another computer, pass the server's reachable IP address and allow TCP ports `27050` (game), `27051` (login), and `8443` (HTTPS) through the Windows host firewall and any network firewall. Port `27052` is only used when connecting to a chat server.

> **Hash validation:** `.env.example` contains `GAMESERVER_ALLOWED_HASHES`, but the current application configuration hard-codes the allowed-hash value. For local testing, keep `GAMESERVER_IGNORE_HASHES=true`; the sample hash list is not currently applied.


## MISC

### - [Project History](docs/HISTORY.md)
### - [Infrastructure](docs/INFRA.md)
### - [Paints](docs/PAINTS.md)

## Credits
- [Christer Ericson’s Real-time CollisionDetection](https://realtimecollisiondetection.net/)
  - build most of the collision detection system using the book as source
- [UnityCsReference](https://github.com/Unity-Technologies/UnityCsReference)
  - to ensure compatibility with calculations in Unity
