# modiceptor

**Challenge name:** modiceptor  
**Category:** OT-security (ICS)  
**Difficulty:** Easy  
**Goal:** Intercept Modbus traffic and extract the hidden flag from register values.

---

## ⚠️ Disclaimer

This repository is a Capture The Flag (CTF) challenge designed to run in a controlled **Haaukins** lab environment.

Do **not** run these techniques against systems or networks you do not own or do not have explicit permission to test. The author(s) are not responsible for misuse or damage caused by running this outside an authorized environment.

---

## What the challenge teaches

This challenge is a beginner-friendly introduction to Industrial Control System (ICS) traffic analysis:

- How to capture and inspect live network traffic
- How Modbus TCP communication works (Master ↔ Slave)
- How to extract data stored in Modbus registers
- Converting raw register values into meaningful text (ASCII)

---

## Prerequisites

You’ll want:

- **Wireshark**
- Basic understanding of **Modbus TCP**
- Familiarity with **TCP ports**

Modbus TCP normally runs on **port 502**.

---

## Step 1 — Open Wireshark and find the Modbus traffic

Start Wireshark and capture traffic on the relevant interface (for example, `eth0`).

To isolate Modbus TCP traffic, use this display filter:

```txt
tcp.port == 502
```

You can also try:

```txt
modbus
```

(Depending on Wireshark’s protocol recognition.)

---

## Step 2 — Inspect Modbus packets for register values

Once filtered, click through the captured Modbus packets.

Look for packets that contain **register data**, typically visible under something like:

- `Modbus/TCP`
- `Read Holding Registers Response`
- `Register value`

In this challenge, the registers contain the following values:

```txt
[108, 97, 118, 101, 95, 109, 95, 115, 116, 101]
```

---

## Step 3 — Convert register values to ASCII

Those numbers are ASCII codes.

Convert each value:

| Decimal | ASCII |
|--------:|:-----|
| 108 | l |
| 97  | a |
| 118 | v |
| 101 | e |
| 95  | _ |
| 109 | m |
| 95  | _ |
| 115 | s |
| 116 | t |
| 101 | e |

That produces the text:

```txt
lave_m_ste
```

---

## Step 4 — Format the flag

The flag format is `HKN{...}`, so the final flag becomes:

```txt
HKN{lave_maste}
```

---

## Final Flag

```txt
HKN{lave_maste}
```

---

## Notes / Tips

- Modbus is a **master/slave** protocol:
  - The **master** requests data
  - The **slave** responds with values (like register contents)
- Flags hidden as **ASCII in registers** are common in beginner ICS challenges.
- Filtering by port `502` is the fastest way to find Modbus TCP traffic.
