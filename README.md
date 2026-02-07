# PASS-HASH

A comprehensive guide to password cracking tools and workflows for cybersecurity education and authorized penetration testing.

> ⚠️ **Legal Notice:** Only use these tools on systems you own or have explicit written permission to test. Unauthorized access to computer systems is illegal.


---

## Understanding the Big Picture

Password cracking is like detective work: you start with scrambled data (hashes), use various tools to transform and analyze them, and eventually recover the original passwords.

```
┌───────────────────────────────────────────────────────────────────┐
│                    REAL-WORLD WORKFLOW                            │
├───────────────────────────────────────────────────────────────────┤
│  1. OBTAIN → 2. CONVERT → 3. IDENTIFY → 4. GENERATE → 5. CRACK    │
│     Hashes      Format       Hash Type     Wordlists    Passwords │
│       ↓           ↓             ↓             ↓            ↓      │
│    (Network    (Extract      (Name        (Wordlist    (Hashcat/  │
│    sniffing,   hashes         That         Tools/AI)      John)   │
│    breaches,   from files)    Hash)                               │
│    dumps)                                                         │
└───────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Hash Identification & Conversion

### 1. Name That Hash / Hashes 🔍

**What it does:** Identifies what type of hash you're looking at (MD5, SHA256, NTLM, etc.)

**Installation:**
```bash
pip install name-that-hash
```

Basic Usage:

```bash
# Identify a hash
nth -t "5f4dcc3b5aa765d61d8327deb882cf99"
# Output: MD5 (Most likely)

# Or use the GUI version
hashes
```

Real-world use: You dump a database and see `5f4dcc3b5aa765d61d8327deb882cf99` — is this MD5? SHA1? NTLM? This tool tells you instantly so you know which Hashcat mode (-m) to use.

---

### 2. Conversion Tools 🔄

These extract hashes from files so you can crack them.

Common Converters:

Tool	Purpose	Command	
`zip2john`	ZIP files → hash	`zip2john secret.zip > zip_hash.txt`	
`pdf2john`	PDF files → hash	`pdf2john.pl document.pdf > pdf_hash.txt`	
`keepass2john`	Keepass databases	`keepass2john database.kdbx > keepass_hash.txt`	
`itunes_backup2hashcat`	iTunes backups	`itunes_backup2hashcat Manifest.plist > itunes_hash.txt`	
`7z2hashcat`	7-Zip archives	`7z2hashcat archive.7z > 7z_hash.txt`	
`Rubeus-to-Hashcat`	Kerberoasting output	Convert Rubeus output to Hashcat format	

Example workflow:

```bash
# Extract hash from encrypted ZIP
zip2john secret.zip > zip_hash.txt

# Crack with John
john zip_hash.txt
```

---

## Phase 2: Wordlist Generation & Analysis

### 3. CeWL (Custom Word List Generator) 🕷️

What it does: Spider websites to create custom wordlists based on company-specific terms.

Installation:

```bash
# Usually pre-installed on Kali
sudo apt install cewl
```

Basic Usage:

```bash
# Spider 2 levels deep, minimum 5 characters
cewl -d 2 -m 5 -w company_words.txt https://target-company.com

# Advanced - include numbers and email addresses
cewl -d 3 -m 4 --with-numbers --email -w target.txt https://example.com
```

Real-world workflow:
1. Target is "Stark Industries"
2. Run CeWL on starkindustries.com
3. Gets words: "Stark", "Tony", "ArcReactor", "JARVIS", "Pepper"
4. Use these as base words for password generation

---

### 4. CUPP (Common User Passwords Profiler) 👤

What it does: Generates wordlists based on personal information (birthdays, pets, family names).

Installation:

```bash
git clone https://github.com/Mebus/cupp.git
cd cupp
python3 cupp.py -i
```

Interactive Mode:

```bash
python3 cupp.py -i

# Example inputs:
# First Name: Tony
# Surname: Stark
# Birthdate: 19700529
# Partner's name: Pepper
# Pet's name: Dummy
# Company: Stark Industries

# Output: tony.txt with thousands of combinations
```

Real-world use: Targeted attacks where you know something about the person. Much more effective than generic wordlists.

---

### 5. PACK (Password Analysis and Cracking Kit) 📊

What it does: Analyzes existing password leaks to create masks and rules.

Installation:

```bash
git clone https://github.com/iphelix/pack.git
cd pack
```

Basic Commands:

```bash
# Analyze a password list to find patterns
python3 statsgen.py passwords.txt -o masks.txt

# Generate masks for Hashcat
python3 maskgen.py masks.txt --target-time 3600 --optindex -o hashcat_masks.hcmask

# Analyze character sets
python3 policygen.py --minlength=8 --maxlength=12 -o policy_masks.hcmask
```

Real-world use: You have a dump of 1000 cracked passwords from a company. PACK analyzes them and finds 80% follow `FirstnameYear!` pattern. You generate custom masks to crack the remaining hashes faster.

---

### 6. Mentalist 🧠

What it does: GUI tool for creating wordlists based on mental patterns people use.

Installation:

```bash
git clone https://github.com/sc0tfree/mentalist.git
cd mentalist
python3 mentalist.py
```

Workflow in GUI:
1. Base Words: Add "CompanyName", "ProjectX"
2. Common Substitutions: Check "a→@", "s→", "o→0"
3. Append/Prepend: Add "2024!", "123"
4. Output: Generate wordlist or Hashcat rules

---

### 7. Other Wordlist Tools

Tool	Purpose	Example	
`Crunch`	Generate all combinations	`crunch 8 8 abc123 -o wordlist.txt`	
`princeprocessor`	PRINCE algorithm	`pp64 < wordlist.txt`	
`maskprocessor`	High-performance generator	`mp64 ?u?l?l?d`	
`duplicut`	Remove duplicates	`duplicut wordlist.txt -o clean.txt`	
`anew`	Append new lines only	`cat new.txt \| anew old.txt`	

---

## Phase 3: Hashcat & John the Ripper

### 8. Hashcat ⚡ (The GPU Powerhouse)

What it does: World's fastest password cracker using your graphics card.

Installation:

```bash
# Kali/Debian
sudo apt install hashcat

# Verify installation
hashcat -I  # List devices
```

Attack Modes:

Mode	Flag	Description	When to Use	
Dictionary	`-a 0`	Tries every word in a list	Fastest, try first	
Combination	`-a 1`	Combines two wordlists	Passwords like "word1word2"	
Brute-force	`-a 3`	Tries every possible character	Short passwords, last resort	
Hybrid (Wordlist + Mask)	`-a 6`	Wordlist + mask appended	"Password123" patterns	
Hybrid (Mask + Wordlist)	`-a 7`	Mask + wordlist appended	"123Password" patterns	

Character Sets (Masks):
- `?l` = lowercase (a-z)
- `?u` = uppercase (A-Z)
- `?d` = digits (0-9)
- `?s` = special characters
- `?a` = all of the above

Basic Examples:

```bash
# Simple dictionary attack (MD5 hash)
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Dictionary with rules (transform words)
hashcat -m 0 -a 0 hash.txt wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# Brute-force 8 characters (upper, lower, digits)
hashcat -m 0 -a 3 hash.txt ?u?l?l?l?l?d?d?d

# NTLM hash (Windows)
hashcat -m 1000 -a 0 ntlm_hash.txt rockyou.txt

# WPA2 WiFi handshake
hashcat -m 2500 -a 0 handshake.hccapx rockyou.txt
```

Essential Rules:

```bash
# Best64 - Good starting point
-r /usr/share/hashcat/rules/best64.rule

# OneRuleToRuleThemAll - Comprehensive
-r OneRuleToRuleThemAll.rule

# NSA Rules - Aggressive
-r nsa-rules.rule
```

---

### 9. John the Ripper 🗡️ (The Flexible Veteran)

What it does: CPU-based cracker, excellent for many formats, easier for beginners.

Installation:

```bash
sudo apt install john
```

Basic Usage:

```bash
# Auto-detect and crack
john hash.txt

# Show cracked passwords
john --show hash.txt

# Specific format
john --format=raw-md5 hash.txt

# With wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# With rules
john --wordlist=words.txt --rules hash.txt
```

John vs Hashcat Decision Tree:

```
Is it a standard hash (MD5, SHA, NTLM)?
├── Yes → Do you have a good GPU?
│   ├── Yes → Use Hashcat (much faster)
│   └── No → Use John
└── No (weird format, proprietary)
    → Use John (supports more formats)
```

---

## Phase 4: Advanced & AI Tools

### 10. PassGPT 🤖

What it does: Uses AI to generate likely passwords based on learned patterns.

Usage:

```bash
# Generate passwords using trained model
python passgpt.py --model passgpt_model --num_passwords 1000000 > ai_wordlist.txt

# Use generated wordlist with Hashcat
hashcat -m 0 -a 0 hash.txt ai_wordlist.txt
```

When to use: Traditional wordlists fail, but you have training data from previous breaches of similar organizations.

---

### 11. RulesFinder 🎯

What it does: Automatically discovers effective rules from cracked passwords.

Usage:

```bash
# Learn rules from known passwords
python rulesfinder.py -d dictionary.txt -p cracked_passwords.txt -o discovered.rules

# Use discovered rules
hashcat -m 0 -a 0 hash.txt dictionary.txt -r discovered.rules
```

---

### 12. Distributed Cracking

Tool	Purpose	Use Case	
`Hashtopolis`	Multi-platform client-server	Enterprise environments	
`CrackLord`	Queue and resource system	Managing multiple jobs	
`fitcrack`	BOINC-based distributed system	Academic/research environments	
`NPK`	AWS serverless cracking	Cloud-based cracking	

---

## Complete Real-World Workflows

### Scenario 1: Penetration Test - Corporate Network

```bash
# STEP 1: Obtain hashes
# (Using tools like Mimikatz, Responder, or hashdump)

# STEP 2: Identify what you have
nth -t "aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0"
# Result: NTLM

# STEP 3: Create targeted wordlist
cewl -d 2 -m 5 -w company.txt https://target-company.com
cupp -i  # Add specific employee info

# STEP 4: Expand with rules
hashcat --stdout company.txt -r /usr/share/hashcat/rules/best64.rule > expanded.txt
cat cupp_output.txt >> expanded.txt

# STEP 5: Crack
hashcat -m 1000 -a 0 ntlm_hashes.txt expanded.txt -o cracked.txt

# STEP 6: If no luck, try masks based on company policy
# (Found policy: 12 chars, uppercase, lowercase, number, special)
hashcat -m 1000 -a 3 ntlm_hashes.txt ?u?l?l?l?l?l?l?d?s?l?l?l
```

---

### Scenario 2: CTF Challenge - Encrypted ZIP

```bash
# STEP 1: Extract hash from ZIP
zip2john challenge.zip > zip_hash.txt

# STEP 2: Check if John can handle it
john --list=formats | grep zip
# Yes, it supports it

# STEP 3: Try quick win with rockyou
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# STEP 4: If slow, convert to Hashcat format
# (Use zip2hashcat or similar)

# STEP 5: GPU acceleration with Hashcat
hashcat -m 17220 -a 0 zip_hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# STEP 6: Still no luck? Brute force with masks
hashcat -m 17220 -a 3 zip_hash.txt ?l?l?l?l?d?d?d?d  # 4 letters + 4 digits
```

---

### Scenario 3: Security Audit - Password Policy Testing

```bash
# STEP 1: Extract domain hashes
secretsdump.py domain/administrator:password@dc01.domain.com > domain_hashes.txt

# STEP 2: Analyze with PACK for patterns
python3 statsgen.py previously_cracked.txt -o masks.txt

# STEP 3: Generate optimized masks
python3 maskgen.py masks.txt --optindex -o audit_masks.hcmask

# STEP 4: Run comprehensive audit

# First: Dictionary attack
hashcat -m 1000 -a 0 domain_hashes.txt rockyou.txt -o cracked_round1.txt

# Second: Rules-based
hashcat -m 1000 -a 0 domain_hashes.txt rockyou.txt -r OneRuleToRuleThemAll.rule -o cracked_round2.txt

# Third: Mask attack based on policy
hashcat -m 1000 -a 3 domain_hashes.txt audit_masks.hcmask -o cracked_round3.txt

# Fourth: Combinator (users love combining words)
hashcat -m 1000 -a 1 domain_hashes.txt names.txt years.txt -o cracked_round4.txt

# STEP 5: Generate report
python3 password-smelter.py -i cracked_round*.txt -o report.html
```

---

### Scenario 4: Wireless Security Assessment

```bash
# STEP 1: Capture handshake
aircrack-ng -w /dev/null -b 00:11:22:33:44:55 capture.cap

# STEP 2: Convert to Hashcat format
hcxpcapngtool -o handshake.hc22000 -E wordlist capture.cap

# STEP 3: Create location-specific wordlist
cewl -d 1 -m 5 -w location.txt https://coffee-shop-target.com

# STEP 4: Combine with common WiFi patterns
echo "CoffeeShop" > base.txt
echo "Guest" >> base.txt
hashcat --stdout base.txt -r /usr/share/hashcat/rules/best64.rule > wifi_candidates.txt

# STEP 5: Crack
hashcat -m 22000 -a 0 handshake.hc22000 wifi_candidates.txt

# STEP 6: If failed, try common WiFi patterns
hashcat -m 22000 -a 3 handshake.hc22000 ?u?l?l?l?l?l?l?d?d?d?d
```

---

Tool Combinations & Pipelines

The "Kitchen Sink" Pipeline

When you need to crack at any cost:

```bash
# 1. Generate massive custom list
cewl -d 3 -m 4 https://target.com > cewl.txt
cupp -i >> cupp.txt  # Interactive
cat cewl.txt cupp.txt | sort -u > combined.txt

# 2. Expand with rules
hashcat --stdout combined.txt -r best64.rule -r toggles5.rule | sort -u > expanded.txt

# 3. Add AI-generated passwords
python passgpt.py --num 100000 >> expanded.txt

# 4. Remove duplicates efficiently
rling expanded.txt final_wordlist.txt

# 5. Analyze and optimize
pack/statsgen.py final_wordlist.txt -o masks.txt

# 6. Crack with everything
hashcat -m [mode] -a 0 hashes.txt final_wordlist.txt -o cracked.txt
hashcat -m [mode] -a 3 hashes.txt masks.txt -o cracked.txt
```

---

Quick Reference

Tool Categories

Category	Tools	Use When	
Identification	Name That Hash, Hashes	You don't know the hash type	
Conversion	2john tools, 7z2hashcat	Hash is embedded in a file	
Wordlist Gen	CeWL, CUPP, Mentalist	You need targeted words	
Wordlist Analysis	PACK, Pipal, password-smelter	You have password dumps to analyze	
Wordlist Optimization	duplicut, Rling, anew	Lists are too big/have duplicates	
Cracking (GPU)	Hashcat	Speed matters, standard hashes	
Cracking (CPU)	John the Ripper	Weird formats, no GPU available	
Distributed	Hashtopolis, CrackLord	Enterprise scale, many hashes	
AI/ML	PassGPT, neural_network_cracking	Traditional methods fail	

Common Hashcat Modes

Mode	Description	
0	MD5	
100	SHA1	
1000	NTLM	
1800	SHA512crypt (Linux)	
2500	WPA/WPA2	
3200	bcrypt	
5500	NetNTLMv1	
5600	NetNTLMv2	
13100	Kerberos 5 TGS-REP	
22000	WPA-PBKDF2-PMKID+EAPOL	

Essential Resources

- Wordlists: RockYou, SecLists, WeakPass
- Rules: Best64, OneRuleToRuleThemAll, NSA-Rules
- Communities: Hashcat Forum, Hashmob, Hashkiller
- Practice: TryHackMe, Hack The Box, CrackMe

---

Contributing

Feel free to submit issues and enhancement requests!

License

# This guide is for educational purposes only. Use responsibly and legally.

---

# Happy Cracking! 🔓

Remember: With great power comes great responsibility.

