# Albo delle Eccellenze (Writeup)

This challenge was tagged as Reverse, but honestly, it was a pretty straightforward Pwn.

### What's going on?
I tossed the binary into Ghidra and checked out the main logic in `FUN_004024e4`. It asks for some personal info to generate a "Codice Fiscale." The program then checks your input against a hardcoded string: `Al1D612LPSCBLS37`.

The funny part? The logic is flipped. If the check fails (meaning `cVar1` is NOT 1), it triggers the flag function. So, we basically just need to break the comparison.

### The Vulnerability
The input function `FUN_004159c0` is broken. It reads way more than it should (up to 100 bytes) into buffers that are much smaller. Since the control variable `cVar1` is sitting right there on the stack after our input, it's a classic Buffer Overflow.

### The Lazy Solve
Why bother reversing the algorithm when you can just smash the stack? I sent 600 "A"s to overflow everything, overwrite the control variable, and force the program to jump to the flag.

**The Exploit:**
```bash
python3 -c 'print("A"*600)' | nc albo.ctf.pascalctf.it 7004
