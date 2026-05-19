# Introduction-to-Computer-Security-Coursework
# SEED Security Labs — README

This repository documents three hands-on security labs from the SEED project (by Wenliang Du, Syracuse University). Each lab is performed on the pre-built SEED Ubuntu 20.04 VM (downloadable from the SEED website, or runnable on the cloud). Each section below describes what the lab covers, what environment it requires, the key tasks involved, and what must be submitted.

A note that applies to all three labs: the **deliverable is always a detailed written lab report with screenshots**, describing what you did and what you observed, explaining anything interesting or surprising, and including important code snippets *with explanation* — code attached without explanation receives no credit.

---

## Lab 1 — Secret-Key Encryption (Crypto Encryption)

**Topic:** Hands-on experience with secret-key encryption algorithms, modes, padding, and initial vectors (IVs), plus common developer mistakes and the attacks they enable.

### Environment required
- SEED Ubuntu 20.04 VM.
- A Docker container running an **encryption oracle** — only needed for Task 6.3. Set it up from `Labsetup.zip` using the provided `docker-compose.yml` (aliases: `dcbuild`, `dcup`, `dcdown`; container shell via `dockps` / `docksh`).
- Tools used: `openssl enc`, `tr`, `bless` hex editor, `eog` image viewer, `hexdump` / `xxd`, Python.

### Tasks
- **Task 1 — Frequency analysis:** Given a ciphertext encrypted with a monoalphabetic substitution cipher, recover the original English plaintext using single-letter, bigram, and trigram frequency analysis. A helper script `freq.py` produces the n-gram statistics; use `tr` to progressively substitute letters (use capitals for recovered plaintext).
- **Task 2 — Ciphers and modes:** Use `openssl enc` to encrypt/decrypt a file with **at least 3 different ciphers** (e.g., `-aes-128-cbc`, `-bf-cbc`, `-aes-128-cfb`).
- **Task 3 — ECB vs. CBC:** Encrypt a `.bmp` image in both ECB and CBC modes, restore the original 54-byte header so it renders, view the result with `eog`, and explain what information leaks (repeat with a picture of your own choice).
- **Task 4 — Padding:** Determine which of ECB/CBC/CFB/OFB use PKCS#5 padding and which don't (and why). Create 5-, 10-, and 16-byte files, encrypt with AES-128-CBC, report the resulting sizes, and use `-nopad` decryption plus a hex tool to inspect exactly what padding bytes are added.
- **Task 5 — Error propagation:** Create a ≥1000-byte file, encrypt with AES-128, corrupt a single bit of the 55th byte (`bless`), then decrypt. **Predict first**, then verify how much information is recoverable under ECB, CBC, CFB, and OFB, with justification.
- **Task 6 — IV and common mistakes:**
  - **6.1:** Encrypt the same plaintext with two different IVs and with the same IV; explain why IVs must be unique.
  - **6.2:** Known-plaintext attack on OFB with a reused IV — recover P2 from C1, P1, and C2 (a `sample_code.py` XOR helper is provided). Also answer how much of P2 is revealed if CFB is used instead.
  - **6.3:** Predictable-IV / chosen-plaintext attack against an encryption oracle (`nc 10.9.0.80 3000`). Construct a plaintext to determine whether Bob's secret message is "Yes" or "No".
- **Task 7 — Programming with the crypto library:** *(For CS/Engineering students — check with your instructor whether it's required.)* Given a plaintext, ciphertext, and IV (AES-128-CBC), brute-force the key, which is a dictionary English word under 16 characters padded with `#` (`0x23`) to 128 bits. You must write your own program calling the crypto library (compile with `-lcrypto`); using the `openssl` command for this task earns no credit.

### Submission
Detailed lab report with screenshots, explanations of interesting/surprising observations, and key code snippets with explanation.

---

## Lab 2 — Packet Sniffing and Spoofing

**Topic:** Understanding and implementing packet sniffing and spoofing — both using tools (Scapy) and by writing low-level C programs (pcap library and raw sockets).

### Environment required
- SEED Ubuntu 20.04 VM.
- Three machines on the same LAN (`10.9.0.0/24`) via the `Labsetup.zip` Docker Compose setup: an **attacker** plus **Host A (10.9.0.5)** and **Host B (10.9.0.6)**.
- The attacker container uses a **shared `./volumes` folder** (edit code on the VM, run in the container) and **`network_mode: host`** so it can see all LAN traffic. Find the relevant interface (prefix `br-`, IP `10.9.0.1`) via `ifconfig` or `docker network ls`.
- Two independent task sets — Set 1 (Scapy, minimal Python) and Set 2 (C programming) — either or both may be assigned depending on programming background.

### Task Set 1 — Using Scapy
- **Task 1.1 — Sniffing:** Use Scapy's `sniff()` to capture packets. **1.1A:** demonstrate capture with root privilege, then run without root and explain the difference. **1.1B:** apply BPF filters separately — ICMP only; TCP from a specific IP with destination port 23; and traffic to/from a chosen subnet (not your VM's subnet).
- **Task 1.2 — Spoofing ICMP:** Use Scapy to spoof an ICMP echo request with an arbitrary source IP and verify via Wireshark.
- **Task 1.3 — Traceroute:** Estimate the number of routers to a destination by incrementing the IP TTL field and observing ICMP time-exceeded replies (manual or automated).
- **Task 1.4 — Sniff-and-then-spoof:** Monitor the LAN; whenever an ICMP echo request is seen, immediately spoof an echo reply so the ping always appears successful. Test against `1.2.3.4`, `10.9.0.99`, and `8.8.8.8`, and explain the results (requires understanding ARP and routing).

### Task Set 2 — Writing C Programs
*(Compile C on the host VM, run inside the container; use `docker cp` to transfer binaries.)*
- **Task 2.1 — Sniffer (pcap):** **2.1A:** write a sniffer that prints source/destination IPs; answer questions on the essential pcap call sequence, why root is required (and where it fails without), and how to demonstrate promiscuous mode on/off. **2.1B:** write filters for ICMP between two hosts, and TCP with destination ports 10–100. **2.1C:** capture a Telnet password off the wire.
- **Task 2.2 — Spoofing (raw sockets):** **2.2A:** write a C spoofing program (provide a Wireshark trace as evidence). **2.2B:** spoof an ICMP echo request on behalf of another machine to a live remote host. Answer questions on arbitrary IP length fields, IP-header checksum calculation, and why root is required.
- **Task 2.3 — Sniff and then spoof (in C):** Implement the sniff-and-spoof behavior of Task 1.4, but in C, with screenshots and well-commented code.
- **Guidelines:** Typecasting buffers into header structs, and converting between network/host byte order (`htonl`/`ntohl`/`htons`/`ntohs`, `inet_addr`, etc.).

### Submission
Detailed lab report with screenshots, explanations, and important code snippets with explanation.

---

## Lab 3 — Buffer Overflow Attack (Set-UID Version)

**Topic:** Exploiting a stack buffer-overflow vulnerability in a root-owned Set-UID program to gain a root shell, then evaluating OS-level countermeasures.

### Environment required
- SEED Ubuntu 20.04 VM.
- **Countermeasures disabled** for the attack phase:
  - Address space randomization: `sudo sysctl -w kernel.randomize_va_space=0`
  - `/bin/sh` relinked to `zsh` (which lacks dash's Set-UID protection): `sudo ln -sf /bin/zsh /bin/sh`
  - StackGuard and non-executable stack disabled at compile time (`-fno-stack-protector`, `-z execstack`)
- Tools: `gcc` (with `-m32` for 32-bit), `gdb` (peda), provided `Makefile`, `exploit.py` skeleton.
- *Note:* the vulnerable buffer size is set via `BUF_SIZE` / Makefile variables `L1`–`L4`, which instructors customize, so prior solutions won't transfer.

### Tasks
- **Task 1 — Shellcode:** Examine the C, 32-bit, and 64-bit shellcode; compile and run `call_shellcode.c` (`make` → `a32.out`, `a64.out`) and describe observations. Compilation requires the `execstack` option.
- **Task 2 — Understand the vulnerable program:** Study `stack.c`, which `strcpy()`s a 517-byte input into an undersized buffer. Compile it as a root-owned Set-UID binary (`chown root` **before** `chmod 4755`).
- **Task 3 — Level 1 (32-bit):** Use `gdb` to find the distance between the buffer and the saved return address (note the gdb-vs-real frame-pointer caveat), complete `exploit.py` to build `badfile`, and obtain a root shell. The report must explain how every value in `exploit.py` was derived.
- **Task 4 — Level 2 (32-bit, unknown buffer size):** Same goal, but you may **not** use the buffer size; assume only that it's 100–200 bytes (frame pointer is a multiple of 4). Build **one** payload that works for any size in the range — brute force loses credit.
- **Task 5 — Level 3 (64-bit):** Attack `stack-L3`. The main added challenge is that 64-bit addresses contain zero bytes, which `strcpy()` stops at — solving this is the core difficulty (frame pointer is `rbp`, not `ebp`).
- **Task 6 — Level 4 (64-bit, tiny buffer):** Same as Level 2/3 but with `BUF_SIZE = 10`; explain how you overcame the very small buffer.
- **Task 7 — Defeating dash's countermeasure:** Relink `/bin/sh` back to `/bin/dash`, prepend `setuid(0)` to the shellcode (32- and 64-bit variants given), test via `make setuid`, then re-run the Level-1 attack with the countermeasure on and prove it (`ls -l /bin/sh /bin/zsh /bin/dash`).
- **Task 8 — Defeating address randomization:** Re-enable ASLR (`kernel.randomize_va_space=2`) and use the provided brute-force loop script against `stack-L1` (only ~19 bits of entropy on 32-bit). Describe observations.
- **Task 9 — Other countermeasures:** **9.a:** recompile *with* StackGuard (drop `-fno-stack-protector`), re-attack, explain. **9.b:** recompile `call_shellcode.c` *without* `-z execstack` (non-executable stack), run, and explain (note: return-to-libc, covered in a separate lab, defeats this).

### Submission
Detailed lab report with screenshots demonstrating investigation and attacks, **with explanation of how the exploit values were chosen** (demonstrating a working attack without explaining why it works earns few points), and key code snippets with explanation.

---

## General notes across all three labs

- All labs run on the **SEED Ubuntu 20.04 VM** (local or cloud); Labs 1 and 2 use Docker containers from a `Labsetup.zip`, with shared `.bashrc` aliases (`dcbuild`/`dcup`/`dcdown`, `dockps`/`docksh`).
- Every lab's deliverable is the same: a thorough report with screenshots, explanation of observations, and commented code snippets — unexplained code is not credited.
