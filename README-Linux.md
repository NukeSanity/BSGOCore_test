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

## Local setup on Ubuntu 24.04 LTS

### Requirements for RetardSanity
- Ubuntu 24.04 LTS and JDK 21. Ubuntu provides the [`openjdk-21-jdk`](https://packages.ubuntu.com/noble/openjdk-21-jdk) package.
- OpenSSL to create the local HTTPS certificate; Bash is included with Ubuntu. Ubuntu provides the [`openssl`](https://packages.ubuntu.com/noble/openssl) package.
- A copy of the BSGO client if you want to connect to the server. Game files are not included.

The Maven wrapper is included, so a separate Maven installation is not needed. On first use it downloads Maven 3.9.9 and the project dependencies; internet access is required unless they are already cached. Install the prerequisites and confirm Java is available:

```bash
sudo apt update
sudo apt install openjdk-21-jdk openssl
java -version
```

### Prepare the project

Run these commands from the `BSGOCore_test-main` project root:

```bash
cp .env.example .env
mkdir -p sqlite observability/logs
chmod +x mvnw
```

Edit `.env` and set `GAMESERVER_IGNORE_HASHES=true` for local testing. If no chat server is running, set `GAMESERVER_CHAT_SERVER_PORT=0` to skip the chat connection. `CLIENT_PATH` is only used by the Windows `runclient.bat` launcher; it does not need to point to a client on the Ubuntu server.

Generate the HTTPS certificate and key from the `certs` directory:

```bash
(cd certs && bash ./generate-certs.sh)
```

This creates `certs/cert.pem` and `certs/key.pem`, matching the paths in `.env.example`. The supplied certificate configuration covers `localhost` and `127.0.0.1`; if clients connect using another IP or hostname, add it to `certs/san.cnf` and regenerate the certificate. The certificate is for local development.

The required game data is included under `ServerConfigurationUtils/global`, including `AugmentTemplates`, `ColliderTemplates`, `JsonCards`, `LootTemplates`, `MissionTemplateConfiguration`, `SectorTemplates`, `ShipConfigTemplates`, and `ZoneTemplates`.

### Start the server

From the project root, start the server:

```bash
./mvnw clean quarkus:dev
```

Flyway applies the database migrations at startup. Wait for the log message `Server started!`. The game and legacy login listeners run on ports `27050` and `27051`; Quarkus HTTPS uses port `8443`. If remote clients connect, allow those TCP ports through the Ubuntu host firewall and any network firewall. For UFW, for example:

```bash
sudo ufw allow 27050/tcp
sudo ufw allow 27051/tcp
sudo ufw allow 8443/tcp
```

### Connect a client

The repository includes `runclient.bat`, which is a Windows launcher and cannot run directly on Ubuntu. To use it, launch the client from a Windows computer with its own BSGO client files, set `CLIENT_PATH` in that Windows checkout's `.env`, and pass the Ubuntu server's reachable IP address:

```powershell
.\runclient.bat <UBUNTU_SERVER_IP> en
```

The language is optional and defaults to `en`. The current launcher does not quote the client path inside the `+cdn` argument, so keep the Windows client path free of spaces. No Linux client launcher is included.

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
