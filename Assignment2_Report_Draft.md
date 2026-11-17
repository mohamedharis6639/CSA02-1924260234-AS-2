# C Programming Assignment 2 – CO2
## Working solution pack and report draft

**Student:** T R Rithvik  
**Register number:** 1912260168  
**GitHub repository:** `CSA02-1912260168-Assignment2` (paste the actual URL after creating it)  
**Compiler:** GCC; enter the exact version shown by `gcc --version`.

> Important: This pack provides code and sample analytical guidance. Execute every program on your own computer, capture genuine screenshots, record your installed compiler version, and replace any illustrative measurements with your own. Do not submit fabricated screenshots or claim unperformed experiments.

---

## Q1 – Precedence, associativity and discount bug (6 marks)

### Problem analysis
The intended sequence is gross amount = price × quantity; senior discount = 15% of gross only for eligible customers; GST = 5% of gross; total = gross − discount + GST; then subtract Rs 50 only when total is strictly greater than Rs 1000.

The supplied expression is parsed using C precedence: multiplication, division and remainder before addition/subtraction; operators at the same precedence level associate left-to-right. The `?:` conditional operator has lower precedence than relational `>`. Integer operands in `/ 100` cause integer division, which can discard fractional parts. The corrected program uses `double` values and explicit intermediate variables.

### Example predictions (assuming GST is on gross)
| Price | Qty | Senior? | Gross | Discount | GST | Total | Net |
|---:|---:|---|---:|---:|---:|---:|---:|
| 100 | 2 | No | 200 | 0 | 10 | 210 | 210 |
| 200 | 5 | Yes | 1000 | 150 | 50 | 900 | 900 |
| 250 | 5 | No | 1250 | 0 | 62.50 | 1312.50 | 1262.50 |

### Undefined behavior
`count = count++ + 1;` modifies `count` through the post-increment and also writes to it through assignment without a sequencing guarantee between the side effect and the other access. The behavior is undefined. `-Wall` may issue a warning, but a warning is not guaranteed to diagnose every undefined expression and does not make the expression safe. Do not run it as a valid calculation.

### Alternative: intermediate variables
The submitted Q1 code uses `gross`, `discount`, `tax`, `total`, and `net`. The arithmetic work remains constant, so time is O(1). A fixed number of temporary variables uses O(1) extra space. The main advantage is auditability and fewer precedence/integer-division mistakes.

---

## Q2 – Short-circuit evaluation (4 marks)

### Logic
For `A && B`, B is evaluated only when A is true. For `A || B`, B is evaluated only when A is false. Bitwise `&` and `|` evaluate both operands; they are not substitutes when the second operand has a cost or side effect.

The Q2 program uses a counter inside `verify_card()` so the call count is observable. It is a local simulation, not a real server connection.

### Experiment design for 1,000 passengers
Use a reproducible test set with columns `senior`, `eligible`, and `disability`. Count calls with the counter. For a concrete comparison, let:
- 300 passengers satisfy `senior && eligible`;
- among the remaining 700, let 100 have a disability card;
- assume the card verification is needed only for the disability path.

Then an order that checks the senior-and-eligible path first can avoid the card verification for those 300 passengers. The actual number of calls depends on the precise condition and test data; include the generated 1,000-row dataset or a summary table in the final report.

### Ordering and audit-log change
Place a cheap condition first when it can prevent an expensive call without changing required side effects. If the verification function must audit every passenger, moving it behind a short-circuit condition would omit required audit entries. In that changed requirement, call the audit function unconditionally, store its result, and then evaluate the concession rule. For n passengers, one audit call per passenger is O(n); a short-circuited version may make between 0 and n calls, but is incorrect if every passenger must be logged.

### Alternative concepts
Nested `if` statements can reproduce short-circuit behavior when written carefully. Replacing logical operators with bitwise operators on integer conditions evaluates both sides and can increase calls to O(n) in a loop over n passengers. Both approaches use O(1) extra space per passenger.

---

## Q3 – Bitwise device register (6 marks)

### Implementation
Each device maps to one bit. `reg | (1u << dev)` sets a bit; `reg & ~(1u << dev)` clears it; XOR toggles it; AND tests it. `unsigned char` represents the byte register.

### Register trace
| Statement | Register (binary) | Hex |
|---|---|---|
| Initial | 00000000 | 0x00 |
| Switch LIGHT on | 00000001 | 0x01 |
| Switch AC on | 00000101 | 0x05 |
| Toggle FAN | 00000111 | 0x07 |
| Switch LIGHT off | 00000110 | 0x06 |

Expected output:
```text
00000101
00000110
AC is ON, PUMP is OFF
```

`reg & (1u << dev)` tests the selected bit. `(reg & 1) << dev` first isolates bit zero, so it does not test bit `dev`. Use unsigned values for bit operations. Setting the high bit of a signed `char` can lead to implementation-defined conversion or problematic signed shifts; use an unsigned type.

### Alternative: `char state[8]`
Each state occupies one byte, so eight states require 8 bytes per home versus 1 byte for the packed register. For one million homes this is approximately 8 MB versus 1 MB (ignoring container overhead). Accessing or changing one state is O(1) in either representation. Counting ON devices is O(8) = O(1) for a fixed eight devices, but the bit-packed version can use repeated shifts or a bit-count method.

---

## Q4 – Grade classifier (6 marks)

### Grade boundaries
90–100 S; 80–89 A; 70–79 B; 60–69 C; 50–59 D; 0–49 F; outside 0–100 is invalid (`?`).

The switch uses `marks / 10`, so 100 maps to case 10 and 90–99 map to case 9. Cases 10 and 9 share the S grade. Every non-empty case returns, preventing fall-through.

### Boundary test table
| Input | Expected |
|---:|:---:|
| -1 | ? |
| 0 | F |
| 49 | F |
| 50 | D |
| 59 | D |
| 60 | C |
| 69 | C |
| 70 | B |
| 79 | B |
| 80 | A |
| 89 | A |
| 90 | S |
| 99 | S |
| 100 | S |
| 101 | ? |

A switch cannot directly express ranges such as 75–84 because case labels must be integer constant expressions, not relational ranges. A missing `break` or `return` causes fall-through into later cases. The lookup table uses O(1) indexing and 11 characters of storage; an else-if ladder may require up to k comparisons for k boundaries. For frequently edited grade bands, a table is compact, while an if-else ladder is clearest for irregular ranges.

For scholarship eligibility, calculate the grade first, then use a clearly separated income-band decision. Nested `if` or a switch on grade with an inner income check is easier to explain than combining all possibilities into one long condition.

---

## Q5 – Prime checking and loop counts (7 marks)

### Improved condition
A composite number has a factor no larger than its square root. Avoid calling `sqrt()` by using `i <= n / i`, which also avoids overflow from `i * i`. Count one division/remainder test for each candidate divisor actually tested. Stop immediately when a divisor is found.

### Expected counts for the assignment's improved loop
| n | Result | Divisors tested |
|---:|---|---:|
| 97 | Prime | 8 (2 through 9) |
| 100 | Not prime | 1 (divisor 2) |
| 7919 | Prime | 87 (2 through 88) |
| 1000003 | Prime | 999 (2 through 1000) |

For the naive loop from 2 to n−1, 97 requires 1 test before finding 97 prime? It tests 95 candidate divisors (2 through 96); 100 stops after one test at divisor 2. The improved loop is O(√n) time and O(1) extra space; naive trial division is O(n) time and O(1) space.

The Q5 code includes for, while and do-while versions. The do-while form must guard n < 2 before entering and check the loop bound before performing a divisor test. For n = 2 or 3, no divisor needs to be tested. `i * i` can overflow for sufficiently large unsigned integers; `i <= n / i` avoids that multiplication.

### Sieve alternative
For many queries up to N, the Sieve of Eratosthenes takes O(N log log N) preprocessing time and O(N) memory, then answers each query in O(1). It is preferable for many queries in a bounded range; trial division is preferable for a small number of large isolated inputs.

---

## Q6 – Loan repayment schedule and termination (8 marks)

### Loop design
A `while` loop is suitable because the number of months is not known in advance. Each month:
1. Calculate interest = opening balance × monthly rate.
2. Add interest to obtain amount due.
3. Pay the smaller of fixed repayment and amount due.
4. Update closing balance.
5. Stop when balance is within a small currency tolerance of zero.

Do not test `balance != 0` with floating-point values. Rounding can leave a tiny residual. The last payment must be capped at the exact amount due so the balance does not become negative. If repayment is less than or equal to the interest on the outstanding principal, the balance will not decrease; reject that case or report that the loan will not amortize. A 360-month safety limit prevents an endless run.

### Hand-check example: Rs 10,000, 2% monthly, Rs 2,000 repayment
| Month | Opening | Interest | Payment | Closing |
|---:|---:|---:|---:|---:|
| 1 | 10000.00 | 200.00 | 2000.00 | 8200.00 |
| 2 | 8200.00 | 164.00 | 2000.00 | 6364.00 |
| 3 | 6364.00 | 127.28 | 2000.00 | 4491.28 |
| 4 | 4491.28 | 89.83 | 2000.00 | 2581.11 |
| 5 | 2581.11 | 51.62 | 2000.00 | 632.73 |
| 6 | 632.73 | 12.65 | 645.38 | 0.00 |

Total interest is approximately Rs 645.38; six months are required. Minor differences can occur depending on whether interest and balances are rounded each month. The provided code keeps double precision internally and displays two decimals.

### Direct formula alternative
For principal P, periodic rate r and payment E, the idealized month count is `n = -log(1 - r*P/E) / log(1+r)` when the assumptions of the formula apply. It is O(1) time and O(1) space, but floating-point precision and rounding of the final month must be handled. Monthly simulation is O(n) time and O(1) space and directly produces the schedule, so it is more useful when an auditable month-by-month statement is required.

---

## Q7 – Day-4 design challenge: improve prime checking (13 marks)

### A. Diagnose
The original improved Q5 implementation tests every integer divisor from 2 through √n. For a large prime such as 1,000,003, it performs 999 divisor tests. Most candidate divisors after 2 are even and cannot divide an odd prime.

### B. Measurable objective
Reduce the number of divisor tests for odd prime inputs by approximately half while returning exactly the same prime/not-prime result.

### C. Redesign
Check divisor 2 once. If n is even, return not prime. Then test only odd candidates 3, 5, 7, … while `i <= n / i`.

Pseudocode:
```text
if n < 2: return not prime
if n == 2: return prime
count test of divisor 2
if n is even: return not prime
for i = 3; i <= n/i; i = i + 2:
    count one test
    if n mod i == 0: return not prime
return prime
```

### D. Prediction
For odd primes, the number of trial divisors after 2 should fall from roughly √n to roughly √n/2. Time complexity remains O(√n), but the constant factor is improved. Extra space remains O(1).

### E. Validation
Run `Q7.c` on identical values (97, 100, 7919, 1000003). It prints the original and redesigned division counts side by side. For an odd prime, the redesigned count includes one check for 2 plus only odd divisors thereafter. For even composites, the redesign detects divisibility by 2 immediately.

### F. Trade-offs and conclusion
The redesign reduces modulo operations for odd inputs without needing an array or additional dynamic memory. The asymptotic complexity is unchanged, but the constant factor improves. The original version is simpler and may be sufficient for small inputs; the redesigned version is preferable when many prime checks are performed.

---

## Final verification checklist
- [ ] Compile every source file using `-std=c11 -Wall -Wextra -pedantic`.
- [ ] Run all seven programs and capture genuine screenshots.
- [ ] For Q1, test both senior and non-senior cases and values around Rs 1000.
- [ ] For Q2, create a 1,000-passenger test dataset and record exact function-call counts.
- [ ] For Q3, include the register trace.
- [ ] For Q4, test every grade boundary and invalid values.
- [ ] For Q5 and Q7, compare exact divisor counts for identical inputs.
- [ ] For Q6, verify the schedule and test a repayment that cannot reduce the balance.
- [ ] Add the actual GitHub URL to the cover page and README.
- [ ] Make at least one meaningful commit per question.
