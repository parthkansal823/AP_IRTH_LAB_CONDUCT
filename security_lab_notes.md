# Security Lab Notes (Exp 1, 2, 4, 5, 6)

---

## Exp 1: Email Authentication (SPF, DKIM, DMARC)

| Protocol | Full Form | Kaam |
|---|---|---|
| **SPF** | Sender Policy Framework | Kaun email bhej sakta hai. Check karta hai ki mail server authorized hai ya nahi |
| **DKIM** | DomainKeys Identified Mail | Sender ke paas **private key** hoti hai, usse email sign karta hai |
| **DMARC** | Domain-based Message Authentication | SPF + DKIM ke saath kaam karta hai, aur policy decide karta hai |

**SPF**
- Mail server authorized hai to → `PASS`
- Nahi hai to → `FAIL`

**DKIM**
- Valid domain check karta hai
- Check karta hai ki signed document **altered** to nahi hua
- Result → `PASS` / `FAIL`

**DMARC policies**

| Policy | Matlab |
|---|---|
| `none` | Koi action nahi, sirf monitor + report |
| `quarantine` | Suspicious mail ko monitor / spam mein daalo |
| `reject` | Mail ko reject kar do |

> Teeno milke email **authentication** karte hain.

---

## Exp 2: Trivy → Docker Containerization

**Installation**
```bash
sudo apt install trivy -y
trivy --version
```

**Use karna**

1. Image scan karna:
   ```bash
   trivy image <name>:tag
   ```

2. Agar image Docker mein nahi hai:
   ```bash
   trivy image --input <path>
   ```
   Ya pehle Docker mein load karo:
   ```bash
   docker load
   ```

3. Sirf HIGH aur CRITICAL vulnerabilities, JSON output ke saath:
   ```bash
   trivy image --severity HIGH,CRITICAL <name> --format json
   ```

---

## Exp 4: Dataset Analysis (Linux Commands)

**Setup**
```bash
unzip data.zip
ls -lh          # detail info (size human-readable)
```

**Column / Row count**

| Kaam | Command |
|---|---|
| Pehli line (header) | `head -1 data.csv` |
| Columns ka count | `head -1 data.csv \| awk -F',' '{for(i=0;i<NF;i++) print($i)}'` |
| Rows ka count | `wc -l` |

> `awk -F','` → separator `,` hai, line by line padhta hai.

**Head / Tail tricks**

| Kaam | Command | Note |
|---|---|---|
| Header ke baad ki rows count | `tail -n +2 data.csv \| wc -l` | Header hata deta hai |
| Last 2 lines hata ke baaki | `head -n -2 data.csv` | Last 2 lines hat jaati hain |
| Second row (first data) | `head -2` | |
| Last row | `tail -1` | |

**grep**

| Kaam | Command |
|---|---|
| "COVID-19" wali pehli record | `grep "COVID-19" data.csv \| head -1 \| awk ...` |
| Kitni lines mein word hai | `grep -c "COVID-19" data.csv` |
| Word kitni baar aaya (times seen) | `grep -o "COVID-19" data.csv` |
| Case insensitive | `grep -i` |
| Multiple patterns (regex) | `grep -E "abc\|xyz"` |

> `awk` yahan output ko **format / print** karne ke liye hai.

**Unique values ka count**
```bash
tail -n +2 data.csv | awk -F',' '{print $2}' | sort | uniq | wc -l
```
- `sort` pehle zaroori hai, tabhi `uniq` sahi kaam karta hai
- `uniq` → duplicate hatata hai
- `wc -l` → count deta hai

**Top 10 most frequent (jaise language = "en")**
```bash
tail -n +2 data.csv | awk -F',' '$22=="en" {print $2}' | sort | uniq -c | sort -nr | head -10 | awk '{print $2}'
```
- `uniq -c` → occurrence count
- `sort -nr` → descending order
- `head -10` → top 10

---

## Exp 5: iptables (Firewall)

**Setup**
- Client → `128`
- Server → `129`
- Apache check: `sudo systemctl status apache2` (restart / start bhi kar sakte ho)

**Basic syntax**
```bash
sudo iptables -A INPUT -p icmp -j DROP
```

| Part | Options |
|---|---|
| `-A` | Add rule |
| `-D` | Delete rule |
| `INPUT` / `OUTPUT` | Chain (direction) |
| `-p` | Protocol: `tcp`, `udp`, `icmp`, `all` |
| `-j` | Action: `ACCEPT`, `REJECT`, `DROP` |

> **DROP** = packet chupchap gira do (koi reply nahi). **REJECT** = reject karke sender ko batao.

**Useful commands**

| Kaam | Command |
|---|---|
| Saare rules list karo | `sudo iptables -L -n -v` |
| Saare rules flush karo | `sudo iptables -F` |

- `-L` → list all rules
- `-n` → IP ko numeric mein dikhao
- `-v` → verbose mode

**Examples**

| Kaam | Command |
|---|---|
| Incoming TELNET reject | `iptables -A INPUT -p tcp --dport 23 -j REJECT` |
| Outgoing TELNET accept | `iptables -A OUTPUT -p tcp --dport 23 -j ACCEPT` |
| VM ke IP par saara incoming block | `iptables -A INPUT -d <IP> -j DROP` |
| Sab kuch drop | `iptables -A INPUT -j DROP` |

**dport vs sport**
- `--dport` → destination port (jaise `80` HTTP, `443` HTTPS, `23` Telnet)
- `--sport` → source port

---

## Exp 6: RAM Forensics (Volatility)

**Run karne ke tareeke**
- Volatility 2 → `volatility -f <RAM file> <cmd>`
- Volatility 3 → `python3 vol.py -f <RAM file> <cmd>`

| Task | Volatility 3 Plugin |
|---|---|
| Image info | `windows.info` |
| Running processes | `windows.pslist` |
| Scan processes (hidden processes) | `windows.psscan` |
| Process tree | `windows.pstree` |
| Command history | `windows.cmdline` |
| Network connections | `windows.netscan` |
| Consoles info | `windows.console` |
| Registry hives (available hives dhundna) | `windows.registry.hivelist` |
| Password hashes | `windows.hashdump` |
| Malware / injected code | `windows.malfind` |

> `pslist` sirf normal processes dikhata hai, `psscan` **hidden** processes bhi pakadta hai.
