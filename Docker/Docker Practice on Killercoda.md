# Docker Practice on Killercoda

Use a free, browser-based Ubuntu machine from [Killercoda](https://killercoda.com/playgrounds/scenario/ubuntu) to practice Docker. You don't install anything locally.

---

## 1. What is Killercoda?

Killercoda gives you disposable Linux environments that run in the browser. The **Ubuntu playground** gives you:

- Root access to an Ubuntu VM
- Docker already installed
- Internet access (you can pull images from Docker Hub)
- A web terminal, plus an option to open exposed ports in the browser

> **Note:** Sessions are temporary. They last about 1 hour on the free tier (longer on paid plans). When the session ends, everything is wiped. Keep your Dockerfiles and notes in a GitHub repo or on your local machine.

---

## 2. Start the Playground

1. Open https://killercoda.com/playgrounds/scenario/ubuntu
2. Sign in (GitHub, Google, or email). You need an account to start a playground.
3. Click **Start**. Wait for the terminal to load.
4. You are now `root` on an Ubuntu machine.

---

## 3. Verify Docker

```bash
docker --version
docker info
systemctl status docker --no-pager
```
