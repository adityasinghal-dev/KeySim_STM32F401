# Keyboard_Int — A Keyboard Built From Logic Gates and Pure Chaos
 
> *"Why use a microcontroller pin for every key when you can use four priority encoders, six logic gates, and one questionable life decision instead?"*
> — probably me, at 2 AM, first year of college
 
## 🎓 What Even Is This
 
This is my very first proper tinker-project with STM32 — built while I was still figuring out what a GPIO pin even was. The idea was simple (the execution, less so): build a **full 36-key keyboard** where every keypress gets squeezed down into a tiny 5-bit code using pure combinational logic, before the STM32 ever gets involved.
 
Instead of wiring 36 keys straight into 36 microcontroller pins like a normal person, I ran them through **four 74LS148 priority encoders**, mixed their "somebody pressed something" flags through a pile of NOT gates and a 4001/4002 NOR-gate combo, and produced a single clean interrupt line into the STM32. The chip wakes up, reads a 5-bit code, and figures out exactly which key you pressed — like a tiny detective with only 5 clues.
 
There's an LCD to greet you, LEDs to show the raw bits flying around, and a virtual terminal that prints out whatever you type, key by key, like a caveman texting.
 
And yes — there is a button labeled **DO NOT DISTURB**. We'll get to that. Don't scroll ahead.
 
## 🧠 How It Actually Works (the short, non-boring version)
 
- **36 keys** (A–Z, Caps, Space, Enter, Backspace, Credits, Interrupt, and the forbidden one) are split across **4 groups of 9**, each group wired into a **74LS148 8-to-3 priority encoder**.
- Each encoder outputs a **3-bit code (A0–A2)** telling you *which* key in its group got pressed, plus a **GS (Group Select)** line that goes low the moment *any* key in that group is touched.
- The four GS lines get funneled through **NOT gates (U3–U6)** and **OR'd together (U2, U7, and the 4002)** into a single master signal: `INTERRUPT:A`.
- That signal hits the STM32F103C8's interrupt pin. The chip goes "someone's typing!", reads the 5 relevant bits across its GPIOs, and cross-references which encoder + which code = which actual key.
- The result gets:
  - Shown live on the **onboard LEDs (BIT1–BIT5)** — literally watch your keypress turn into binary in real time.
  - Printed on the **LCD screen**.
  - Streamed out over **USART to the Proteus Virtual Terminal**, so you can watch your typing appear like it's 1987 and you're hacking a mainframe.
Basically: 4 encoders + a handful of logic gates = a whole keyboard, decoded entirely in hardware before software even wakes up. No keyboard matrix scanning loops, no debouncing 36 pins in software — just interrupts and vibes.
 
## 📦 What's In This Repo
 
| File | What it does |
|---|---|
| `Keyboard_Int.pdsprj` | The Proteus project — schematic + PCB-style keyboard layout |
| `Keyboard_Encoder.hex` | The compiled firmware — this is what actually runs on the virtual STM32 |
| `*.workspace` | Your Proteus workspace file (safe to ignore, Proteus just likes to remember stuff) |
 
## 🛠️ Setting This Up (Step-by-Step, For Humans)
 
### Step 1: Get Proteus
You'll need **Proteus Design Suite** (the VSM simulation environment) to actually run this — it's what simulates the STM32, the logic gates, the keyboard, everything.
 
1. Download it from [Labcenter Electronics' official site](https://www.labcenter.com/) (or grab it from wherever your college handed it to you — no judgment).
2. Install it. Grab a coffee, it takes a bit.
3. Open it up and pray your license still works.
### Step 2: Load the Project
1. Open **Proteus (ISIS)**.
2. `File → Open Project` → select `Keyboard_Int.pdsprj`.
3. You should now be staring at the same beautiful mess of wires shown in the schematic — four encoder ICs, a wall of NOT/NOR gates, an STM32, an LCD, and a keyboard that looks suspiciously hand-drawn (it was).
### Step 3: Flash the Firmware (into the *virtual* chip, relax)
Since we're in simulation, "flashing" just means telling Proteus which `.hex` file the STM32 should run:
 
1. **Double-click** on the STM32F103C8 chip (labeled **U1**) in the schematic.
2. In the component properties window, find the field called **"Program File"**.
3. Browse to and select `Keyboard_Encoder.hex`.
4. Hit **OK**.
That's it — no ST-Link, no drivers, no `dfu-util` fighting you at 3 AM. Proteus just loads it straight into the virtual flash.
 
### Step 4: Open the Terminal
This is where you actually *see* your typing:
 
1. Go to `Debug → Virtual Terminal` (or find the **Virtual Terminal** icon already dropped on the schematic — it's wired to the STM32's RXD/TXD/RTS/CTS pins).
2. It'll pop up as a little black terminal window.
3. Make sure its **baud rate matches the firmware's USART config** (check your code, but it's typically 9600 unless you changed it).
4. Leave this window open before you hit run — you want to catch every keystroke live.
### Step 5: Hit Run and Start Typing
1. Click the **Play ▶** button at the bottom-left of Proteus to start the simulation.
2. Click on any key in the keyboard layout (A–Z, Space, Enter, Backspace...).
3. Watch the magic:
   - The **BIT LEDs** flicker as the 5-bit code races through the logic.
   - The **LCD** updates.
   - The **Virtual Terminal** prints your key.
4. Mash the keyboard like you're trying to break it. You built the encoder — it can take it.
## 🚨 About That One Button
 
You will notice, tucked in the bottom-right corner of the keyboard layout, a button that says:
 
```
   DO NOT DISTURB
```
 
Every fiber of your being will want to click it. This is a trap laid by your past self (me) for your future self (also me, and now also you).
 
**Do not click it.**
 
I'm not saying anything bad happens. I'm not saying nothing happens either. I'm just saying — there's a reason it's not labeled A through Z, and there's a reason it's sitting all alone, away from the rest of the keys, like it's been put in time-out.
 
If you click it anyway (and let's be honest, you will), don't say I didn't warn you when your LCD or terminal suddenly has *opinions*. You'll laugh, then you'll immediately regret it, then you'll probably click it again just to see it happen one more time. That's the cycle. Welcome to it.
 
## 🏁 Credits
 
Designed, wired, debugged, and mildly regretted by **Aditya** — Version 1.0, 2026.
Built while discovering STM32 and electronics for the first time in college, one priority encoder and one blown-up gate simulation at a time.
 
If this README made you smile half as much as building this project frustrated me, mission accomplished. Now go press some keys (but seriously, leave that one button alone).
