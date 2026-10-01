# Problem 5.9 — T Flip-Flop with Asynchronous Clear in Behavioral VHDL

Problem 5.9 asks you to **describe the behavior of a T flip-flop that also has an asynchronous clear**, using behavioral VHDL rather than building it from gates.

A clean solution is:

```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.all;

ENTITY tff IS
    PORT (
        T     : IN  STD_LOGIC;
        Clock : IN  STD_LOGIC;
        Clear : IN  STD_LOGIC;
        Q     : OUT STD_LOGIC
    );
END tff;

ARCHITECTURE Behavior OF tff IS
    SIGNAL Q_int : STD_LOGIC := '0';
BEGIN

    PROCESS (Clock, Clear)
    BEGIN
        IF Clear = '1' THEN
            Q_int <= '0';

        ELSIF rising_edge(Clock) THEN
            IF T = '1' THEN
                Q_int <= NOT Q_int;
            END IF;
        END IF;
    END PROCESS;

    Q <= Q_int;

END Behavior;
```

I used an internal signal `Q_int` because a T flip-flop needs to know its **current state** in order to toggle it.

Conceptually:

\[
T=0 \Rightarrow Q_{next}=Q
\]

\[
T=1 \Rightarrow Q_{next}=\overline{Q}
\]

So if `Q_int = '0'` and `T = '1'`, it becomes `'1'`; if `Q_int = '1'`, it becomes `'0'`.

---

## How to reason your way to the code

Start from the actual behavior of the hardware.

A T flip-flop has this characteristic table:

| Clear | Clock event | T | Next Q |
|---|---|---:|---:|
| 1 | anything | X | 0 |
| 0 | rising edge | 0 | same Q |
| 0 | rising edge | 1 | NOT Q |
| 0 | no rising edge | X | same Q |

The most important thing is that **Clear has priority over the clock**.

That is exactly why the code is arranged as:

```vhdl
IF Clear = '1' THEN
    ...
ELSIF rising_edge(Clock) THEN
    ...
END IF;
```

The asynchronous clear check comes first.

---

## 1. Library section

```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.all;
```

This gives you `STD_LOGIC` and logical operations such as `NOT`.

Your VHDL notes explain that the IEEE package is needed when using `std_logic` and `std_logic_vector`.

---

## 2. The ENTITY

```vhdl
ENTITY tff IS
    PORT (
        T     : IN  STD_LOGIC;
        Clock : IN  STD_LOGIC;
        Clear : IN  STD_LOGIC;
        Q     : OUT STD_LOGIC
    );
END tff;
```

Think of the `ENTITY` as the **outside of the chip**.

It defines the pins:

```text
        ┌────────────┐
T ─────►│            │
Clock ─►│   T Flip   │────► Q
Clear ─►│    Flop    │
        └────────────┘
```

Your VHDL 1 notes describe this exact division: the `ENTITY` defines ports, while the `ARCHITECTURE` defines how the circuit operates.

They also explain that `IN` ports can be read and `OUT` ports are outputs.

---

## 3. The ARCHITECTURE

```vhdl
ARCHITECTURE Behavior OF tff IS
```

This says:

> Now describe what `tff` actually does.

Because the problem specifically says **behavioral code**, we're describing the rules of operation instead of connecting NAND gates, muxes, or another flip-flop component.

Your VHDL 1 material describes the architecture as the part containing the circuit's implementation details.

---

## 4. Why `Q_int` exists

```vhdl
SIGNAL Q_int : STD_LOGIC := '0';
```

A T flip-flop has memory.

Suppose:

```text
Q = 0
T = 1
```

The next state must be:

```text
Q = 1
```

Next clock:

```text
Q = 0
```

Then:

```text
Q = 1
```

So the circuit has to remember its current value.

That's what:

```vhdl
Q_int
```

does.

Your VHDL notes describe internal `SIGNAL` declarations as signals used inside an entity.

---

## 5. The PROCESS

This is the central part:

```vhdl
PROCESS (Clock, Clear)
```

A process is useful here because flip-flops are **sequential circuits**.

Your VHDL 4 notes explain that although a `PROCESS` itself exists concurrently with the rest of the architecture, statements **inside the process execute sequentially from top to bottom**.

That is exactly what we need because we want to establish priority:

```text
1. Check Clear
2. Otherwise check clock
3. Then check T
```

---

## 6. Why are `Clock` and `Clear` in the sensitivity list?

```vhdl
PROCESS (Clock, Clear)
```

These are the signals capable of causing the flip-flop's state to change.

Your notes explain that signals in the process sensitivity list cause the process to be reevaluated when an event occurs on them.

Notice something important:

```vhdl
T
```

is **not required to be in the sensitivity list**.

Why?

Changing `T` alone should not change the flip-flop.

For example:

```text
Q = 0

T changes:
0 → 1

Q should remain:
0
```

Only a clock edge should actually make the T input affect Q.

---

## 7. The asynchronous clear

This is the most important new concept in the problem:

```vhdl
IF Clear = '1' THEN
    Q_int <= '0';
```

"Asynchronous" means:

> Clearing does not wait for the clock.

Suppose the clock is doing nothing:

```text
Clock ─────────────
```

and suddenly:

```text
Clear  0 ────┌──── 1
             │
```

Then Q immediately becomes:

```text
Q      1 ────┐
             └──── 0
```

There does **not** need to be a rising clock edge.

That's why `Clear` appears here:

```vhdl
PROCESS (Clock, Clear)
```

and is tested before the clock:

```vhdl
IF Clear = '1' THEN
```

The textbook discussion of asynchronous clear is under **Section 5.4.3, D Flip-Flops with Clear and Preset** in the edition matching this problem set.

---

## 8. Detecting the rising clock edge

Next:

```vhdl
ELSIF rising_edge(Clock) THEN
```

This means:

```text
Clock: 0 → 1
```

Only then does the T flip-flop consider changing state.

Picture it:

```text
Clock

0 ______┌──────
        ↑
       rising edge
```

That upward transition is when the flip-flop operates.

You may also encounter textbook code written as:

```vhdl
ELSIF Clock'EVENT AND Clock = '1' THEN
```

which is another traditional way of describing a positive edge.

For studying, understand **both forms**, even if you use `rising_edge(Clock)` in your own code.

---

## 9. Implementing the T behavior

Inside the clock test:

```vhdl
IF T = '1' THEN
    Q_int <= NOT Q_int;
END IF;
```

This implements the definition of a T flip-flop.

If:

```text
T = 1
```

then toggle:

```text
0 → 1
1 → 0
```

which is exactly:

```vhdl
NOT Q_int
```

For example:

```text
Q_int = 0

NOT Q_int = 1
```

and:

```text
Q_int = 1

NOT Q_int = 0
```

Notice that there is no explicit:

```vhdl
ELSE
    Q_int <= Q_int;
```

That's intentional.

For a storage element, if no assignment is made, the stored value remains unchanged.

So:

```vhdl
IF T = '1' THEN
    Q_int <= NOT Q_int;
END IF;
```

means:

```text
T = 1 → toggle
T = 0 → hold
```

---

## 10. Driving the output

Finally:

```vhdl
Q <= Q_int;
```

The internal stored state drives the external output pin.

So mentally:

```text
Q_int = memory inside the flip-flop
Q     = pin you see outside the flip-flop
```

---

## The whole behavior in plain English

You can almost translate the process line-by-line:

```vhdl
PROCESS (Clock, Clear)
BEGIN
```

> Watch the clock and clear.

```vhdl
IF Clear = '1' THEN
    Q_int <= '0';
```

> If clear is active, immediately reset Q to zero.

```vhdl
ELSIF rising_edge(Clock) THEN
```

> Otherwise, wait for a positive clock edge.

```vhdl
IF T = '1' THEN
    Q_int <= NOT Q_int;
```

> At that clock edge, if T is 1, toggle Q.

If T is 0:

> Do nothing, which means retain the previous Q.

---

## Example timing sequence

Suppose initially:

```text
Q = 0
Clear = 0
```

Then:

| Event | T | Result |
|---|---:|---:|
| rising clock | 0 | Q stays 0 |
| rising clock | 1 | Q → 1 |
| rising clock | 1 | Q → 0 |
| rising clock | 0 | Q stays 0 |
| rising clock | 1 | Q → 1 |
| Clear becomes 1 | X | Q → 0 immediately |

The last row is what makes the reset **asynchronous**.

---

## Where to study this in your textbook

The edition matching the screenshot organizes Chapter 5 roughly like this:

**Chapter 5 — Flip-Flops, Registers, and Counters**

The sections most relevant to Problem 5.9 are:

1. **§5.4 — Edge-Triggered D Flip-Flops** — learn how a clocked flip-flop behaves.
2. **§5.4.3 — D Flip-Flops with Clear and Preset** — explains asynchronous clear.
3. **§5.5 — T Flip-Flop** — gives the T-flip-flop behavior: hold when `T=0`, toggle when `T=1`.
4. **§5.12.2 — Using VHDL Constructs for Storage Elements** — directly useful for coding storage elements behaviorally.
5. Look for **Example 5.4 / Figure 5.38**, which describes a **D flip-flop with an asynchronous active-low reset**. That is very close to the coding template needed here; substitute T-flip-flop next-state behavior for the D input behavior.

Your uploaded VHDL slides also provide the needed coding background:

- **VHDL 1**: `ENTITY`, `ARCHITECTURE`, signals, and `STD_LOGIC`
- **VHDL 4 pages 5–7**: `PROCESS`, sensitivity lists, and `IF/ELSIF`

---

## Useful exam-memory template

For a clocked flip-flop with **asynchronous reset**, memorize this skeleton:

```vhdl
PROCESS (Clock, Reset)
BEGIN

    IF Reset = '1' THEN
        -- reset state

    ELSIF rising_edge(Clock) THEN
        -- normal flip-flop behavior

    END IF;

END PROCESS;
```

Then the only part that changes is the flip-flop behavior.

For a T flip-flop:

```vhdl
IF T = '1' THEN
    Q_int <= NOT Q_int;
END IF;
```

For a D flip-flop:

```vhdl
Q_int <= D;
```

For a JK flip-flop, the inside becomes a small `IF` or `CASE` structure for the four `J,K` combinations.

So the main idea behind Problem 5.9 is:

**asynchronous clear template + T-flip-flop characteristic behavior = solution.**
