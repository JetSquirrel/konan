---
name: reverse-engineering
description: Binary analysis, decompilation, and reverse engineering. Use when you need to (1) Analyze binary executables for malware or CTF challenges, (2) Decompile binaries to understand functionality, (3) Identify vulnerabilities through control flow analysis, (4) Extract algorithms or keys from compiled code, or (5) Patch binaries for analysis or exploitation.
metadata:
  type: security
  category: binary-analysis
---

# Reverse Engineering Skill

Systematic binary analysis using static and dynamic techniques to understand program behavior.

## Prerequisites

### Binary Analysis Tools

```bash
# Ghidra - NSA's reverse engineering suite
# Download from https://ghidra-sre.org/
# Extract and add to PATH

# radare2 - Open source reversing framework
sudo apt-get install radare2

# IDA Free - Industry standard (limited free version)
# Download from https://hex-rays.com/ida-free/

# Binary Ninja Cloud - Modern disassembler
# Access at https://cloud.binary.ninja/

# objdump - Basic disassembly
sudo apt-get install binutils

# gdb with pwndbg/gef - Enhanced debuggers
sudo apt-get install gdb
pip install pwndbg
# or: bash -c "$(curl -fsSL https://gef.blah.cat/sh)"

# ltrace/strace - Library and system call tracers
sudo apt-get install ltrace strace

# Hopper - Disassembler for macOS/Linux
# Download from https://www.hopperapp.com/
```

### Supporting Tools

```bash
# angr - Binary analysis framework
pip install angr

# capstone - Disassembly framework
pip install capstone

# unicorn - CPU emulator
pip install unicorn

# keystone - Assembler engine
pip install keystone-engine

# pwntools - Exploit development
pip install pwntools

# LIEF - Library for Instrumentation of Executable Formats
pip install lief

# pyelftools - ELF parsing
pip install pyelftools

# pefile - PE parsing
pip install pefile
```

## Static Analysis Workflow

### 1. Initial Reconnaissance

Gather basic information about the binary:

```bash
# File type and architecture
file binary
readelf -h binary  # For ELF
objdump -f binary  # Cross-platform

# Check for packing/obfuscation
upx -t binary 2>/dev/null && echo "UPX packed"
detect-it-easy binary  # Comprehensive packer detection

# Extract strings
strings binary | less
strings -e l binary  # UTF-16LE
strings -e b binary  # UTF-16BE

# Check security features
checksec --file=binary  # NX, PIE, RELRO, Canary
rabin2 -I binary       # radare2 info

# Symbol information
nm binary              # List symbols
readelf -s binary      # ELF symbols
objdump -t binary      # Object file symbols

# Section analysis
readelf -S binary      # ELF sections
objdump -h binary      # Section headers

# Dependency analysis
ldd binary             # Shared libraries
readelf -d binary      # Dynamic section
```

### 2. Disassembly with radare2

```bash
# Start radare2
r2 binary

# Inside r2:
> aaa              # Analyze all (functions, xrefs, etc.)
> afl              # List all functions
> pdf @main        # Print disassembly of main
> VV               # Visual graph mode
> / flag           # Search for "flag" string
> /R flag{.*}      # Regex search
> axt @str.flag    # Cross-references to flag string
> s sym.check_password  # Seek to function
> pdf              # Print current function
> afvd             # Show local variables
> pdc @main        # Decompile main (if available)

# Graph generation
> ag               # ASCII graph
> agf              # Full graph

# Binary patching
> oo+              # Reopen in write mode
> wa nop @0x1234   # Write nop at address
> quit
```

### 3. Decompilation with Ghidra

**Automated Ghidra Workflow:**

```python
# Ghidra headless analysis
analyzeHeadless /path/to/project MyProject \
  -import binary \
  -scriptPath /path/to/scripts \
  -postScript DecompileAll.py \
  -log /tmp/ghidra.log

# Python script for Ghidra automation
from ghidra.app.decompiler import DecompInterface
from ghidra.util.task import ConsoleTaskMonitor

def decompile_function(function):
    decompiler = DecompInterface()
    decompiler.openProgram(currentProgram)

    results = decompiler.decompileFunction(function, 0, ConsoleTaskMonitor())
    if results.decompileCompleted():
        return results.getDecompiledFunction().getC()
    return None

# Extract all function decompilations
for func in currentProgram.getFunctionManager().getFunctions(True):
    code = decompile_function(func)
    print(f"// Function: {func.getName()}")
    print(code)
    print("\n" + "="*80 + "\n")
```

**Manual Ghidra Workflow:**

1. Import binary (File → Import File)
2. Analyze (Analysis → Auto Analyze)
3. Review Symbol Tree and Function Graph
4. Examine decompiled code in Decompiler window
5. Add comments and rename variables
6. Export analysis (File → Export Program)

### 4. Control Flow Analysis

Understand program flow and identify key logic:

```python
import angr

def analyze_control_flow(binary_path):
    project = angr.Project(binary_path, auto_load_libs=False)

    # Generate CFG
    cfg = project.analyses.CFGFast()

    # Find interesting functions
    main_func = cfg.kb.functions['main']

    # Analyze basic blocks
    for block in main_func.blocks:
        print(f"Block at {hex(block.addr)}")
        print(f"Size: {block.size}")
        print(f"Instructions: {block.instructions}")

        # Check for comparison operations
        if any(insn.mnemonic.startswith('cmp') for insn in block.capstone.insns):
            print(f"Found comparison at {hex(block.addr)}")

    # Identify critical paths
    graph = main_func.graph
    print(f"Function has {len(graph.nodes())} basic blocks")
    print(f"Function has {len(graph.edges())} edges")

    return cfg
```

### 5. Data Flow Analysis

Track how data flows through the program:

```python
import angr

def track_variable(binary_path, func_name, var_name):
    project = angr.Project(binary_path)

    # Find function
    cfg = project.analyses.CFGFast()
    func = cfg.kb.functions[func_name]

    # Perform reaching definitions analysis
    rd = project.analyses.ReachingDefinitions(func)

    # Track variable uses
    for node in rd.observed_results.keys():
        defs = rd.observed_results[node]
        for reg, definitions in defs.register_definitions.items():
            print(f"Register {reg} at {hex(node.addr)}: {definitions}")

    return rd
```

## Dynamic Analysis

### 1. Debugging with GDB + pwndbg

```bash
# Start debugging
gdb binary

# Inside gdb:
(gdb) break main
(gdb) run arg1 arg2
(gdb) info registers
(gdb) x/20x $rsp         # Examine stack
(gdb) x/s 0x555555554008 # Examine string
(gdb) disassemble main
(gdb) ni                 # Next instruction
(gdb) si                 # Step into
(gdb) continue
(gdb) finish             # Run until return

# pwndbg specific:
(gdb) checksec           # Security features
(gdb) vmmap              # Memory mappings
(gdb) heap               # Heap chunks
(gdb) search flag        # Search memory
(gdb) telescope $rsp 20  # Smart stack view
```

### 2. System Call Tracing

```bash
# Trace system calls
strace binary
strace -e open,read,write binary  # Specific syscalls
strace -o trace.log binary        # Save to file

# Trace library calls
ltrace binary
ltrace -e malloc,free binary      # Specific functions
ltrace -o trace.log binary        # Save to file

# Advanced tracing with eBPF
bpftrace -e 'tracepoint:syscalls:sys_enter_open { printf("%s\n", str(args->filename)); }'
```

### 3. Dynamic Instrumentation with Frida

```javascript
// Frida script: hook_function.js
Interceptor.attach(Module.findExportByName(null, "strcmp"), {
    onEnter: function(args) {
        console.log("strcmp called:");
        console.log("  arg1:", Memory.readUtf8String(args[0]));
        console.log("  arg2:", Memory.readUtf8String(args[1]));
    },
    onLeave: function(retval) {
        console.log("  result:", retval.toInt32());
    }
});

// Hook custom function by address
Interceptor.attach(ptr("0x401234"), {
    onEnter: function(args) {
        console.log("Custom function called");
        console.log("  arg1:", args[0]);
        console.log("  arg2:", args[1]);
    }
});
```

```bash
# Run Frida script
frida -l hook_function.js binary
frida -U -l hook_function.js -f com.app.name  # Android
```

## Advanced Techniques

### 1. Symbolic Execution with angr

Automatically find inputs that reach specific code paths:

```python
import angr
import claripy

def symbolic_execution_solver(binary_path, find_addr, avoid_addrs):
    project = angr.Project(binary_path, auto_load_libs=False)

    # Create symbolic input
    flag = claripy.BVS('flag', 8*32)  # 32 bytes
    state = project.factory.entry_state(stdin=flag)

    # Constrain to printable ASCII
    for byte in flag.chop(8):
        state.solver.add(byte >= 0x20)
        state.solver.add(byte <= 0x7e)

    # Create simulation manager
    simgr = project.factory.simulation_manager(state)

    # Explore to find target, avoid failure paths
    simgr.explore(find=find_addr, avoid=avoid_addrs)

    if simgr.found:
        found_state = simgr.found[0]
        solution = found_state.solver.eval(flag, cast_to=bytes)
        print(f"Found solution: {solution}")
        return solution

    print("No solution found")
    return None

# Example usage
find_addr = 0x401234  # Address of success message
avoid_addrs = [0x401250, 0x401260]  # Addresses of failure messages
solution = symbolic_execution_solver("challenge.bin", find_addr, avoid_addrs)
```

### 2. Binary Patching

Modify binary to bypass checks or enable features:

```python
import lief

def patch_binary(input_file, output_file, patches):
    """
    patches: [(address, original_bytes, new_bytes), ...]
    """
    binary = lief.parse(input_file)

    for addr, original, new in patches:
        # Find section containing address
        section = binary.section_from_virtual_address(addr)
        offset = addr - section.virtual_address

        # Verify original bytes
        current = section.content[offset:offset+len(original)]
        if bytes(current) != original:
            print(f"Warning: bytes at {hex(addr)} don't match expected")
            continue

        # Apply patch
        for i, byte in enumerate(new):
            section.content[offset + i] = byte

        print(f"Patched {len(new)} bytes at {hex(addr)}")

    # Write modified binary
    binary.write(output_file)

# Example: NOP out a check
patches = [
    (0x401234, b'\x74\x05', b'\x90\x90'),  # je -> nop nop
    (0x401250, b'\x75\x03', b'\xeb\x03'),  # jne -> jmp
]
patch_binary("challenge.bin", "challenge_patched.bin", patches)
```

### 3. Malware Analysis

Safe analysis of potentially malicious binaries:

```bash
# Static analysis (safe)
strings malware.exe | grep -i http
rabin2 -i malware.exe    # Imports
rabin2 -zz malware.exe   # All strings

# Check for common malware indicators
grep -r "CreateRemoteThread\|VirtualAllocEx\|WriteProcessMemory" imports.txt

# Behavior analysis in sandbox
# Use Cuckoo Sandbox, ANY.RUN, or Joe Sandbox

# Memory dump analysis
volatility -f memory.dump imageinfo
volatility -f memory.dump --profile=Win10x64 psscan
volatility -f memory.dump --profile=Win10x64 malfind
volatility -f memory.dump --profile=Win10x64 yarascan -y malware_rules.yar
```

### 4. Anti-Analysis Detection & Bypass

**Common Anti-Analysis Techniques:**

```python
# Detect anti-debugging checks
def detect_anti_debug(binary_path):
    binary = lief.parse(binary_path)

    suspicious_imports = [
        'IsDebuggerPresent',
        'CheckRemoteDebuggerPresent',
        'NtQueryInformationProcess',
        'OutputDebugString',
        'ptrace'
    ]

    found = []
    for imp in binary.imports:
        if imp.name in suspicious_imports:
            found.append(imp.name)

    return found

# Patch out anti-debug checks
def bypass_anti_debug(binary_path, output_path):
    patches = []

    # Common patterns:
    # call IsDebuggerPresent -> xor eax, eax; nop
    # test eax, eax -> xor eax, eax

    # These need to be found dynamically or with signatures

    patch_binary(binary_path, output_path, patches)
```

## Automated Analysis Scripts

### Comprehensive Binary Analysis

```python
#!/usr/bin/env python3
import angr
import sys
import json

def analyze_binary(binary_path):
    project = angr.Project(binary_path, auto_load_libs=False)

    report = {
        'filename': binary_path,
        'architecture': str(project.arch),
        'entry_point': hex(project.entry),
        'functions': [],
        'strings': [],
        'security': {}
    }

    # CFG analysis
    cfg = project.analyses.CFGFast()

    # Extract functions
    for func_addr in cfg.kb.functions:
        func = cfg.kb.functions[func_addr]
        report['functions'].append({
            'address': hex(func.addr),
            'name': func.name,
            'size': func.size,
            'num_blocks': len(list(func.blocks))
        })

    # Extract strings (from loaded binary)
    for _, string in project.loader.main_object.strings:
        report['strings'].append(string.decode('utf-8', errors='ignore'))

    # Security features
    if hasattr(project.loader.main_object, 'pic'):
        report['security']['PIE'] = project.loader.main_object.pic

    return report

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <binary>")
        sys.exit(1)

    report = analyze_binary(sys.argv[1])
    print(json.dumps(report, indent=2))
```

### Vulnerability Scanner

```python
def scan_vulnerabilities(binary_path):
    """Scan for common binary vulnerabilities"""
    project = angr.Project(binary_path, auto_load_libs=False)
    cfg = project.analyses.CFGFast()

    vulnerabilities = []

    for func in cfg.kb.functions.values():
        # Check for dangerous functions
        dangerous_funcs = ['gets', 'strcpy', 'sprintf', 'scanf']

        for block in func.blocks:
            for insn in block.capstone.insns:
                if insn.mnemonic == 'call':
                    target = insn.operands[0]
                    # Check if calling dangerous function
                    for dangerous in dangerous_funcs:
                        if dangerous in str(target):
                            vulnerabilities.append({
                                'type': 'buffer_overflow',
                                'function': func.name,
                                'address': hex(insn.address),
                                'dangerous_call': dangerous,
                                'severity': 'HIGH'
                            })

    return vulnerabilities
```

## Integration with Other Skills

### Link to Vulnerability Scanner

```bash
# Scan binary for vulnerabilities
uv run bub chat
> Can you use vulnerability-scanner to check this binary for known CVEs?

# Then reverse engineer any identified issues
> Now use reverse-engineering to analyze the vulnerable function at 0x401234
```

### Link to Exploit Development

```bash
# After finding vulnerability
> I found a buffer overflow in function check_password.
> Can you use exploit-development to create a working exploit?
```

## CTF Reverse Engineering Workflow

```python
def solve_reverse_ctf(binary_path):
    """Automated CTF reverse engineering"""

    # Step 1: Basic analysis
    print("[*] Step 1: Basic Analysis")
    run_command(f"file {binary_path}")
    run_command(f"strings {binary_path} | grep -i flag")

    # Step 2: Check for easy wins
    print("[*] Step 2: Quick Checks")
    output = run_command(f"strings {binary_path}")
    if 'flag{' in output:
        print("[+] Found flag in strings!")
        return extract_flag(output)

    # Step 3: Disassembly
    print("[*] Step 3: Disassembly")
    run_r2_commands(binary_path, ['aaa', 'afl', 'pdf @main'])

    # Step 4: Symbolic execution
    print("[*] Step 4: Symbolic Execution")
    flag = symbolic_execution_solver(
        binary_path,
        find_addr=find_success_address(binary_path),
        avoid_addrs=find_failure_addresses(binary_path)
    )

    if flag:
        print(f"[+] Found flag: {flag}")
        return flag

    # Step 5: Dynamic analysis
    print("[*] Step 5: Dynamic Analysis")
    # Use gdb or frida

    return None
```

## Best Practices

1. **Start Simple**
   - Run file, strings, readelf first
   - Check for obvious flags
   - Look at imports and exports

2. **Understand the Architecture**
   - Know your target (x86, ARM, MIPS)
   - Understand calling conventions
   - Recognize common patterns

3. **Document Everything**
   - Rename functions and variables
   - Add comments in disassembler
   - Keep notes on findings

4. **Use Multiple Tools**
   - Cross-reference findings
   - Each tool has strengths
   - Combine static and dynamic

5. **Automate When Possible**
   - Script repetitive tasks
   - Use symbolic execution for complex logic
   - Leverage existing frameworks

6. **Stay Safe**
   - Analyze malware in VMs only
   - Never run untrusted binaries on host
   - Use sandboxes for dynamic analysis

## Reference Materials

- [Ghidra](https://ghidra-sre.org/)
- [radare2 Book](https://book.rada.re/)
- [angr Documentation](https://docs.angr.io/)
- [Practical Binary Analysis](https://practicalbinaryanalysis.com/)
- [Malware Analysis Bootcamp](https://malwareunicorn.org/)
- [Reverse Engineering for Beginners](https://beginners.re/)
- [OpenSecurityTraining](https://opensecuritytraining.info/)
