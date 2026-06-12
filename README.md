##  Project Overview and Goal

This repository provides a robust, macro-based solution for automating print status updates from the Kipper Klipper
platform to Discord.

The primary goal is **reliable remote notification**. Instead of manual checks or relying on persistent connections
(like webhooks in OctoPrint), this system leverages the built-in G-code event hooks of the firmware (`[gcode_macro]`)
to trigger an external Python script whenever a critical print event occurs (Success, Failure, Pause, etc.).

###  Architecture Flow Diagram
The notification flow is strictly layered:

`Klipper Event (G-Code)` $\rightarrow$ `Call Script Command (Shell/Bash)` $\rightarrow$ `Python Webhook Sender`
$\rightarrow$ `Discord Discord API Webhook URL` $\rightarrow$ **Notification Sent**

##  Prerequisites and Requirements

Before setup, ensure the following components are ready:

### 1. Hardware / Software
*   A functional Kipper Klipper-enabled printer (running compatible firmware).
*   A host machine (e.g., Raspberry Pi) connected to run the Python script calls. This host must have **Python 3**
installed and network connectivity to Discord.
*   `requests` library installed on the host computer (`pip install requests`).

### 2. Discord Setup
*   An active Discord account for the desired notification channel.
*   A dedicated Webhook URL generated within that specific Discord channel's settings (The webhook must be read-only
and treated like a password). **(CRITICAL: This is your secret key!)**

##  Installation Guide (Step-by-Step)

### Step 1: Configure the Python Sender Script
1.  Create the primary communication script (`send_discord.py`) in an accessible directory on your host machine.
2.  Paste the following structure, **replacing placeholders** with your actual Webhook URL and required formatting
logic.

*(Include your full `send_discord.py` code block here.)*

### Step 2: Integrate Macros into Klipper Firmware
1.  Place the macro definitions in a dedicated section of your main `printer.cfg` file (e.g.,
`macros/status_macros.cfg`).
2.  Verify that all macros correctly execute the call to the Python script using the host's command executor (`!python
send_discord.py ...`).

*(Include your full G-code macro code block here, e.g., `[gcode_macro DISCORD_FAIL]...`)*

### Step 3: Testing and Validation
1.  **Initial Test:** Manually run a low-impact test command (e.g., `DISCORD_WARN`) directly from your interface to
ensure the host computer can correctly execute the shell call and communicate with Discord.
2.  **Full Cycle Test:** Run a full, short print job. Verify that both the successful and failure/warning macros are
triggered at the expected points in the macro execution flow.

##  Repository Structure

```
kipper-klipper-discord-macros/
├── README.md          <-- This file!
|
├── config/
│   └── printer.cfg    <-- Your main firmware configuration (Includes macros)
│       └── [gcode_macro DISCORD_FAIL] ...
|
├── scripts/
│   └── send_discord.py  <-- The core Python webhook sender script
│
└── docs/
    └── setup_guide.pdf <-- Optional PDF flow diagram/user manual
```

##  Usage and Calling Conventions (Reference)

These are the dedicated macros for triggering notifications from your main `G-code` routines. Always use these wrappers
to ensure standardized formatting.

###  Print Success Macro
Use this macro as the absolute last step in your printing sequence when the job completes successfully.
```gcode
DISCORD_SUCCESS
# Sends: "Job Completed! Model finished successfully."
```

###  Print Failure/Critical Stop Macro
Triggered by error handlers, limit switches, or critical failures (e.g., runaway). This is the most urgent status code.
```gcode
DISCORD_FAIL
# Sends: "🚨 PRINT FAILURE DETECTED! Stopping immediately."
```

###  Warning/Pause Macro
Use this when the print pauses for non-fatal reasons (e.g., temperature drop, material change).
```gcode
DISCORD_WARN
# Sends: "💡 PAUSED/WARNING - Check feed lines or bed leveling."
```

##  Troubleshooting and Known Issues

*   **"No Discord Message Received":** The most common issue is an incorrect Webhook URL or permission lapse on the
Discord side. Double-check the URL!
*   **`ModuleNotFoundError: No module named 'requests'`:** You have not installed the necessary Python library on your
hostç machine (`pip install requests`).
*   **Timing Issues:** If the notification seems delayed, ensure the `gcode_macro` is placed *after* the command that
triggers the failure/success event.
---
