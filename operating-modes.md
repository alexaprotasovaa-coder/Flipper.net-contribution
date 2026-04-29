# Overview

:::hint{type="info"}
Some details on this page are pending confirmation. Items marked with a question link (e.g., *(Q1)* reference the [Open questions](./#open-questions) section, where each item lists its source, status, and impact.
:::

This page describes the four operating modes of Flipper One: **Transport**, **Low-Power**, **Boot Menu**, **Linux Mode**. Use it to identify the current mode and understand what each mode does.

---

# Modes at a glance

## Compute state

| Component | Transport Mode | Low-Power Mode | Boot Menu | Linux Mode |
| --------- | -------------- | -------------- | --------- | ---------- |
| MCU       | OFF            | ON             | ON        | ON         |
| CPU       | OFF            | OFF            | ON        | ON         |
| Linux     | OFF            | OFF            | OFF       | ON         |

## User interface

| Component        | Transport Mode | Low-Power Mode | Boot Menu       | Linux Mode  |
| ---------------- | -------------- | -------------- | --------------- | ----------- |
| Display          | OFF            | Lightweight UI | U-Boot UI       | Full UI     |
| Power button LED | OFF            | OFF            | Blinking yellow | Solid green |

## Ports and I/O

Flipper One includes multiple I/O interfaces. The behavior of these interfaces across operating modes is documented separately in [Tech Specs](https://docs.flipper.net/one/general/tech-specs).

Notable mode-specific behavior: 

- USB-C, supports Power Delivery input and Output across all modes, excluding Transport Mode. Connecting USB-C to the charging port switches the device from Transport Mode to Low-Power Mode. [*(Q1, Q5)*](./#open-questions)
- Two GPIO pins (M40, M41) are available in the Low-Power Mode. [*(Q10)*](./#open-questions)
- **All other ports** , active only in Linux Mode unless otherwise specified.

---

# Mode transitions

An authoritative source for mode transitions has not yet been established. The information below summarizes confirmed hardware-defined transitions only.

## Confirmed transitions 

````mermaid
flowchart TD
    Start([Initial state: factory / shipping]) --> T[Transport]
    T -- "Power ON: Power button hold 1s OR USB-C connect<br/>(target TBD)" --> L[Low-Power]
    L -- "'Start' softkey<br/>(target TBD)" --> B[Boot Menu]
    B -- "Boot profile selected" --> X[Linux]
    X -- "Power overlay → 'Turn OFF'" --> L
````
**Legend:** Edges marked `(TBD)` in their label are not confirmed. [*(Q1, Q3, Q4, Q7, Q8)*](./#open-questions)

---

# Hardware controls

Hardware-defined behaviors that affect operating modes (Power button long press, full power cycle, hardware reset) are documented in [Buttons and shortcuts](https://docs.flipper.net/one/hardware/about#buttons-shortcuts).

---

# Per-mode details 

## Transport Mode

The device is completely powered down, the display is not active, only the charger IC and RTC remaining active for maximum power saving. 

**Use this mode for:** Storing the device unused, or shipping it to another location. [*(Q7, Q8)*](./#open-questions)

## Low-Power Mode

The default mode for turned off device. The Low-Power MCU displays battery level and quick controls. Acts as a power bank for charging connected USB-C devices. 

**Use this mode for:**
- Charging USB-C devices as a passive power bank.
- Quickly checking battery level without booting the full system.
- Default off-state for everyday use, when the user turns the device off from Linux Mode.
 
## Boot Menu

Device running U-Boot Menu with available Flipper OS boot profiles. U-Boot has full control over the display and input, operating system is not running yet. [*(Q2, Q3, Q9)*](./#open-questions)

**Use this mode for:** Selecting which OS profile to boot, alternate boot targets, or entering a recovery target.

## Linux Mode

The primary operating mode. The full Flipper OS is running, all hardware is active, and the FlipCTL service controls the display and user interface. [*(Q8, Q9, Q4)*](./#open-questions)

**Use this mode for:** Everyday use.

# Naming reference

The following names appear in design mockups, GitHub discussions, and WIKI files. This section is provided as a search aid. [*(Q6)*](./#open-questions)

| Used in this doc | Also known as                                                                                   |
| ---------------- | ----------------------------------------------------------------------------------------------- |
| Transport Mode   | Power OFF, Deep Power OFF, Full Power OFF, Shipping Mode, Shut down                             |
| Low-Power Mode   | MCU Mode, Standby, Power Bank Mode, Sleep, Always-On MCU, Battery Mode, Passive Mode, Rest Mode |
| Boot Menu        | U-Boot Menu, U-Boot Mode, Bootloader Mode, Boot Selection, Boot stage                           |
| Linux Mode       | Active Mode, Full Mode, OS Mode, Main Operating Mode, Full System Mode                          |

# Related topics

- [Buttons shortcuts](https://docs.flipper.net/one/hardware/about#buttons-shortcuts), full reference for Power button, D-pad, softkeys, Back+Left reset.
- [Power management](https://docs.flipper.net/one/testing/power), battery, charging, and sleep behavior.
- [About MCU firmware Architecture](https://docs.flipper.net/one/mcu-firmware/architecture)
- [Built-in OS profiles](https://docs.flipper.net/one/cpu-software/flipper-os#os-profiles). 
- [FlipCTL](https://docs.flipper.net/one/cpu-software/flipctl). 

---

# Open questions

:::hint{type="info"}
Some behaviors below are not yet documented in production materials. This page will be updated as confirmation arrives.
:::

1. What activates the device from Transport Mode, and where does it boot to?
   - Trigger options (per source):
	    - Hold Power button for 1 second , [docs.flipper.net](https://docs.flipper.net/one/hardware/about#controls-description)
		- Connect USB-C , [docs.flipper.net](https://docs.flipper.net/one/hardware/about#buttons-shortcuts)
	- Unclear: Does the device boot into Low-Power, Boot Menu, or directly into Linux Mode?
	- Impact: Determines first edge in state transition diagram

2. What is the trigger from Low-Power to Boot Menu?
	- **Source:** Miro [v002 mockup](https://miro.com/app/board/uXjVJ6y839o=/) shows a 'Start' softkey
	- **Unclear:** Is this confirmed in production? What is the exact button mapping?

3. What happens after Boot Menu inactivity timeout?
	- **Source:** Not documented in [The original issue text](https://github.com/flipperdevices/flipperone-ui/issues/1#issue-3975606491)  or production docs
	- **Options:** Auto-boot default profile / return to Low-Power / shutdown to Transport

4. Where does Linux Mode 'Turn OFF' lead?
	- **Source:** [Power overlay shown in Linux Mode mockup](https://docs.flipper.net/one/hardware/about#controls-description)
	- **Unclear:** Returns to Low-Power or to Transport?

5. Is USB-C fully OFF in Transport Mode, or only data-disabled?
	- **Source:** Not documented
	- **Impact:** Affects whether USB connect can wake the device (see Q1)

6. Canonical mode naming: Original Issue text vs Miro v002?
	- **Conflict:**
		- **Source:**  [The original issue text](https://github.com/flipperdevices/flipperone-ui/issues/1#issue-3975606491), Power Off, `MCU Mode`, `U-Boot Mode`, `Main Operating Mode`
		- Miro  [v002 mockup](https://miro.com/app/board/uXjVJ6y839o=/): `Transport Mode`, `Low Power` Mode, `U-Boot Menu`, `Linux Mode`
	- **Decision needed:** Which naming is final for user-facing docs?

7. Is Transport Mode user-accessible?
	- **Source:**  [The original issue text](https://github.com/flipperdevices/flipperone-ui/issues/1#issue-3975606491), describes Transport Mode as the factory/shipping state
	- **Unclear:**
		- Can a user enter Transport Mode intentionally (e.g., for travel or long-term storage)?
		- Or is it set only at the factory before shipping, and unreachable after first boot?
	- **Impact:** Determines whether Transport Mode appears in user-facing flows or only in manufacturing docs

8. Can the user transition from Linux Mode directly to Transport Mode?
	- **Source:** Not documented in  [the original issue text](https://github.com/flipperdevices/flipperone-ui/issues/1#issue-3975606491), mockups, or production docs
	- **Options:**
		- Direct shutdown to Transport via a UI action (e.g., 'Power off' overlay)
		- Only via Low-Power as an intermediate step
		- Only via manufacturing or service tools, not user-accessible
	- **Impact:** Affects state diagram (whether Linux → Transport edge exists)

9. Can the user enter Boot Menu from Linux Mode without a full power cycle?
	- **Source:** Not documented
	- **Options:**
		- Reboot menu option in Linux Mode UI that boots into Boot Menu
		- Reset key combo (Back + Left) that lands in Boot Menu
		- Only via full power cycle: Linux → Low-Power → Boot Menu
	- **Impact:** Affects state diagram (whether Linux → Boot Menu edge exists)

10. Are GPIO pins M40 and M41 available in all modes, or only in Low-Power?
	- **Source:**  [the original issue text](https://github.com/flipperdevices/flipperone-ui/issues/1#issue-3975606491) lists M40/M41 GPIO ports availability for Low-Power Mode
	- **Unclear:**
		- Same pins active in Boot Menu?
		- Same pins active in Linux Mode?
		- Different pin sets per mode (e.g., MCU-driven in Low-Power, CPU-driven in Linux)?
	- **Impact:** Affects Ports and I/O table accuracy across all modes
