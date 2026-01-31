# Malta Nightlife - PascalCTF 2026 Write-up (Pwn)

A detailed walkthrough of the **Malta Nightlife** challenge from PascalCTF. This challenge involves an **Integer Overflow** vulnerability that allows a user to bypass financial logic and retrieve a hidden flag.

## 1. Challenge Overview
* **Category:** Pwn
* **Difficulty:** Medium/Easy
* **Description:** A simple drink ordering system where you start with 100€. The goal is to buy the "Flag" drink which costs 1,000,000,000€.

## 2. Analysis

### Static Analysis (Ghidra)
Upon decompiling the binary, we find two key functions:

1.  **`init()` Function:**
    * It fetches the flag from the system environment variable `FLAG`.
    * It stores the flag pointer at an offset of `0x48` (72 bytes) within the descriptions array.
    * In a 64-bit binary, this points exactly to the description of the **10th drink** (index 9).

2.  **`main()` Function:**
    * The program stores drink prices in an array where the 10th item is set to `1,000,000,000`.
    * **The Vulnerability:** The budget check uses a signed integer cast during comparison:
      ```c
      if ((int)balance < (int)(quantity * price))
      ```

### The Vulnerability: Integer Overflow
The flaw is a **Signedness Bug**. The variables are stored as unsigned integers, but the check casts them to signed integers.
* The maximum value for a 32-bit signed integer is `2,147,483,647`.
* If we order **3 units** of the Flag drink: $3 \times 1,000,000,000 = 3,000,000,000$.
* In 32-bit signed representation, `3,000,000,000` overflows and becomes a **negative number** (~ -1.29 billion).
* Since `100 < [Negative Number]` is **False**, the check passes, and the program subtracts a negative cost from your balance, effectively giving you billions in credits.



## 3. Exploitation Steps
1.  **Trigger Overflow:** Order 3 units of drink #10. This overflows the cost calculation and increases your balance.
2.  **Purchase Flag:** Now that you have a massive balance, order 1 unit of drink #10.
3.  **Retrieve Flag:** The program prints the "secret recipe" for drink #10, which the `init()` function previously replaced with the actual flag.

## 4. Exploit Script
The exploit was written using the `pwntools` library in Python.

heck out the full exploit script here: [exploit.py](./exploit.py)

---

## 5.Result 

![PoC](./screenshot.jpg)
*Payload delivered. Shell popped. Mission accomplished.*
