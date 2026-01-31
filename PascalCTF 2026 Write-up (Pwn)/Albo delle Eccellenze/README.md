# Writeup: Albo delle Eccellenze (PascalCTF 2026)
### Category: Reverse Engineering / Pwn

## 📝 Challenge Overview
The challenge presents a binary that validates the personal data of a specific CTF player (Name, Surname, DOB, etc.) to grant access to a flag. While it is categorized as a **Reverse Engineering** task, the solution was achieved by identifying and exploiting a memory corruption vulnerability, typical of the **Pwn** category.

* **Target**: `nc albo.ctf.pascalctf.it 7004`
* **Tools Used**: Ghidra, Python, Netcat.

---

## 🔍 Phase 1: Reverse Engineering (Static Analysis)
Initial analysis in **Ghidra** revealed the following:

1.  **Entry Point**: The `entry` function calls the main logic handler at `FUN_004024e4`.
2.  **Logic Flow**: The program prompts for multiple strings including Name, Surname, Date of Birth, and Place of Birth.
3.  **The Validator**: It processes these inputs to generate a "Codice Fiscale" and compares it against the hardcoded string `"Al1D612LPSCBLS37"` in function `FUN_004023eb`.
4.  **The Gatekeeper**: A variable `cVar1` stores the result of this check. If `cVar1` is NOT `1`, the program calls `FUN_00402311`, which reads the flag from the server.

---

## 🛠 Phase 2: Finding the Shortcut (The Pwn Approach)
While analyzing the stack layout in Ghidra, I identified a classic **Buffer Overflow** opportunity.

* **Vulnerability**: The input function `FUN_004159c0` reads up to 100 bytes into buffers that are adjacent to critical control variables on the stack.
* **Memory Layout**: The critical variable `cVar1` (the conditional flag) is located at a higher memory address than the input buffers `local_208` and `local_1a4`.
* **Strategy**: By providing an excessively long input, I could overwrite `cVar1` with arbitrary data, successfully bypassing the "Codice Fiscale" check regardless of the actual data entered.



---

## 🚀 Phase 3: Exploitation
I crafted a payload consisting of 300 "A" characters to ensure the stack was sufficiently smashed to reach and overwrite `cVar1`.

### Execution:
```bash
python3 -c 'print("A"*300)' | nc albo.ctf.pascalctf.it 7004

