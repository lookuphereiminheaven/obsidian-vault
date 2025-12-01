### I. Core Model: Abusing Memory Integrity

Native compiled code, particularly when dealing with memory allocation and data structure, fails to protect against input that exceeds expected boundaries. These flaws, rooted in C/C++ memory handling, violate the integrity of the application's runtime environment.

- **The Trust Failure:** The vulnerability exists because low-level functions assume that user-supplied data, which includes every parameter, cookie, and header, will respect buffer limits and data type ranges. The input is trusted by the compiled component.
- **Attack Objective:** Overwrite memory or manipulate control flow to execute arbitrary code or trigger denial of service (DoS).
- **High Risk:** Probing for these flaws carries a high risk of causing unhandled exceptions and resulting in a denial of service (DoS) to the application.

---

### II. Flaw Taxonomy: Corrupting Runtime State

WAHH Chapter 16 focuses on classic software vulnerabilities that materialize when user input interacts with low-level compiled components used by the web application.

#### A. Buffer Overflows

The most critical native code vulnerability, arising when data written to a memory buffer exceeds its allocated size, corrupting adjacent data or execution pointers.

- **Mechanism:** Functions that rely on fixed-size buffers (e.g., handling lengthy form input) fail to check the length of user-supplied input before copying it into the buffer.
- **Weaponization:** Input is crafted to include arbitrary code (shellcode) that overwrites the instruction pointer on the stack, diverting execution to the attacker's payload, achieving RCE.

#### B. Integer Vulnerabilities

These vulnerabilities arise from the finite size of numerical variables, exploiting how languages handle mathematical operations that exceed maximum or minimum data types.

- **Integer Overflow:** A calculation exceeds the maximum capacity of an integer, wrapping around to a small or negative number.
- **Signedness Errors:** Exploiting confusion between signed and unsigned integers (where a negative number can be interpreted as a large positive one), often leading to buffer overruns when calculating buffer sizes.
- **Weaponization:** Manipulating numerical input (e.g., quantity or length parameters) to bypass validation checks or trigger unexpected memory allocation flaws that lead to privilege escalation or RCE.

#### C. Format String Bugs

These bugs occur when user input is used as the format string parameter in functions like `printf()`, giving the attacker control over stack operations.

- **Mechanism:** Attacker provides format specifiers (e.g., `%x`, `%n`) that leak memory contents or allow the attacker to write arbitrary data to memory locations.
- **Weaponization:** Used to read arbitrary data from the stack or execute code by controlling memory pointers, a complex path to RCE.

---

### III. Attacker Playbook: Probing for Instability

Detecting these flaws from a black-box web perspective requires intentionally submitting oversized, malformed, or numerically extreme inputs to functions likely to be handled by compiled code (e.g., image processing, encryption, or complex API calls).

1. **Input Vector Testing:** Fuzz every conceivable input vector—headers, cookies, URL parameters, and POST data—with excessively long strings (e.g., thousands of identical characters) to trigger buffer overflows.
2. **Integer Fuzzing:** Systematically submit boundary values for numerical input parameters: 0, -1, maximum positive integer, maximum negative integer, and values slightly above or below valid limits. This specifically targets integer vulnerabilities.
3. **Anomalous Behavior Monitoring:** Monitor the target application for signs of instability:
    - **Application Crash:** The target stops functioning or returns generic connection errors (indicative of a detected vulnerability that led to an unhandled exception).
    - **Verbose Errors:** Any stack trace or debug output (Chapter 15) must be harvested immediately, as it may reveal internal memory addresses or library versions needed for exploitation.
4. **Format String Probing:** Test parameters with format string specifiers (e.g., `%p%p%p%p`) and monitor output for repeated values or memory leakage, confirming a format string vulnerability.

---

### IV. Real Exploits and Defense Gaps

While low-level, these flaws are highly prized because they circumvent application-level security, yielding RCE.

- **Exploitation Difficulty:** These flaws require specialized knowledge (reverse engineering, assembly language, memory manipulation) far beyond typical web hacking to exploit fully. However, the initial detection remains a crucial first step for bug hunters.
- **Defense Failure (Trust in Data):** The fundamental defense failure is the language choice itself (C/C++), where manual memory management is required. Failure to check input length (`strcat` or `strcpy` without bounds checking) and relying on assumption-based bounds in functions are the root causes.
- **LLM Context (2025+):** If user input is passed to a native library via an AI wrapper (A01: LLM Prompt Injection), the attacker can inject oversized strings designed to overflow buffers within the native component, leading to RCE, effectively utilizing the AI layer to breach the system boundary.

---

### V. One-Liner

Use Burp Intruder (Sniper) to test for potential **Buffer Overflows** by submitting extremely long, unique, and sequential strings to a single input parameter.

```
# Burp Intruder (Sniper Attack)
# Payload Position: Target a header value or POST parameter
# Payload Type: Custom Iterator - generate strings of length 1000, 2000, 4000, 8000
# Grep Match: Flag responses that return a 500 error or a complete connection reset.
ffuf -w long_strings.txt -u https://target.com/api/process_image?data=FUZZ -X POST -d "param=value" -mc 500,400 -fs 0
```

_Purpose: Fuzzes a parameter with progressively massive input, monitoring for catastrophic failures (500 Internal Server Error) or connection resets that indicate a crash/buffer overflow of a native component._