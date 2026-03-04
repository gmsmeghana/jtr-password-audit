# 📋 Findings Report — Password Security Audit

**Project:** JTR Password Security Audit Lab  
**Tool:** John the Ripper  
**Environment:** Kali Linux (VirtualBox)  
**Date:** March 2026  

---

## Scenario 1 — MD5 Dictionary Attack

**Command:**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes/md5_hashes.txt
```

**Output:**
```
Loaded 3 password hashes with 3 different salts (md5crypt)
Will run 7 OpenMP threads
qwerty       (charlie)
letmein      (bob)
password123  (alice)
3g DONE — Session completed.
```

**Result:** 3/3 cracked  
**Analysis:** All 3 passwords were common dictionary words found within the first few million entries of rockyou.txt. MD5crypt provides minimal resistance against dictionary attacks due to its low computational cost.

---

## Scenario 2 — SHA-512 Dictionary Attack

**Command:**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes/sha512_hashes.txt
```

**Analysis:** SHA-512crypt uses 5000 iterations (cost factor) compared to MD5crypt's single pass. This makes each guess ~5000x slower, but common passwords like `password123` and `hello123` are still found quickly because they appear early in rockyou.txt.

---

## Scenario 3 — Rule-Based Attack

**Command:**
```bash
john --wordlist=hashes/custom_wordlist.txt --rules=best64 hashes/md5_hashes2.txt
```

**Output:**
```
Loaded 1 password hash (md5crypt)
1g DONE
Password123!  (frank)
```

**Result:** 1/1 cracked  
**Analysis:** `Password123!` is not literally in the custom wordlist — John cracked it by applying the `best64` rule set to the base word `password`. Rules perform transformations like capitalizing the first letter, appending numbers, and adding symbols. This demonstrates that simply substituting characters (p@ssword, Password1!) does NOT make passwords secure.

---

## Scenario 4 — Brute Force (Incremental Mode)

**Command:**
```bash
john --incremental=alpha --max-length=4 hashes/brute_hashes.txt
```

**Result:** `abc` cracked  
**Analysis:** Short passwords (≤4 characters) are trivially vulnerable to brute force. Even with only alphabetic characters, the search space for 4-character passwords is just 26⁴ = 456,976 combinations — cracked in milliseconds.

---

## Scenario 5 — Unshadow + Linux Shadow File Attack

**Command:**
```bash
unshadow hashes/fake_passwd.txt hashes/fake_shadow.txt > hashes/unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes/unshadowed.txt
```

**Output:**
```
Loaded 1 password hash (sha512crypt, $6$)
Cost 1 (iteration count) is 5000
monkey    (henry)
1g DONE — Session completed.
henry:monkey:1001:1001::/home/henry:/bin/bash
```

**Result:** 1/1 cracked  
**Analysis:** This scenario simulates the most realistic attack vector — obtaining `/etc/shadow` from a Linux system (via privilege escalation or file exposure) and cracking it offline. `monkey` is ranked in the top 500 most common passwords in rockyou.txt, making it trivially crackable despite SHA-512's cost factor.

---

## Overall Summary

| Scenario | Hash | Method | Cracked | Notes |
|----------|------|--------|---------|-------|
| 1 | MD5crypt | Dictionary | ✅ 3/3 | Instant — weak algorithm |
| 2 | SHA-512crypt | Dictionary | ✅ | Slower but common passwords still fall |
| 3 | MD5crypt | Rule-based | ✅ 1/1 | Symbol substitution is ineffective |
| 4 | MD5crypt | Brute Force | ✅ | Short passwords trivially cracked |
| 5 | SHA-512crypt | Unshadow + Dict | ✅ 1/1 | Realistic Linux attack simulation |

---

## Conclusions

- **Weak passwords fall regardless of hash algorithm.** Even SHA-512 cannot save `monkey` or `password123`.
- **MD5 should never be used for password hashing.** Its speed makes it trivially attackable.
- **Rule-based attacks defeat predictable password patterns.** `Password1!` is not meaningfully stronger than `password`.
- **Password length matters most.** A random 16-character password would resist all these attacks.
- **Recommended hashing algorithms:** bcrypt, scrypt, or Argon2id with appropriate cost factors.
