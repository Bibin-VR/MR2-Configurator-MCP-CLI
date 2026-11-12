# MR2-Configurator-MCP-CLI

Mitsubishi MR Configurator2 for AI agents and the command line.

MR2-Configurator-MCP-CLI connects Claude, or any other agent that speaks the Model Context
Protocol (MCP), to MR Configurator2, Mitsubishi Electric's setup software for MELSERVO servo
amplifiers. The agent reads and writes drive parameters, checks alarms and status, tests the
I/O and builds `.mrc2` project files. It works through the engines MR Configurator2 itself
uses, without opening the program's window. People can run the same commands in a terminal
with `mrc2`.

<p>
  <a href="https://bibinvr.tech/contact"><img alt="Contact" src="https://img.shields.io/badge/Contact-bibinvr.tech-1f6feb?style=for-the-badge"></a>
</p>

> The source lives in a private repository. This repo is a public overview.

## What you can ask for

> *"Read the drive on USB and tell me why it's in alarm."*

> *"Save everything on this drive to a project file before I change anything."*

> *"Set PA09 to 20 on this drive and read it back."*

> *"Make an MR-JET-G project with PA06 = 3 and PB07 = 23.5."*

> *"Switch output CN3-13 on and off so I can check the wiring."*

> *"Why won't the motor turn?"*

Anything that changes the drive or can move the motor waits for your confirmation.

## Use cases

| Situation | What the agent does |
|---|---|
| Commissioning a new machine | Reads each drive, sets the parameters from your sheet and reads every value back |
| Before changing a running machine | Saves the drive's settings to a `.mrc2` file you can open in MR Configurator2 |
| Many axes or series production | Builds a project file per axis, or writes the same settings to every drive, with the same checks each time |
| A machine that stopped | Reads the alarm, the alarm history, the status monitor, the I/O and the drive's list of reasons it won't rotate, then explains them |
| Wiring checks | Forces outputs on and off and reads the inputs while you check each wire at the panel |
| Remote support | Works on the customer's PC over SSH. Nobody has to be logged in, and MR Configurator2 doesn't have to be running |
| Reviewing a project | Lists only the parameters that differ from the factory setting, with names and units |
| Records and audits | Each command and its result is plain text that can be kept with the machine's documents and run again later |
| Training and demos without a motor | Builds project files for any of the 63 models, and runs a drive in motor-less operation |

## Compared with using MR Configurator2 by hand

| Task | MR Configurator2 by hand | With MR2-Configurator-MCP-CLI |
|---|---|---|
| Read a drive's parameters | Connect, open the parameter window, read, scroll through each group | One request; all 320 parameters of an MR-JE-B as data in about 20 seconds |
| Find what differs from factory | Compare value by value with the manual or a default project | Listed directly, against MR Configurator2's factory values |
| Write parameters | Type each value, write, check that it took | Checked against MR Configurator2's own ranges and digit options before anything is written, then each value is read back. If one value in a batch is wrong, nothing is written |
| New project | New project dialog, pick the model, then edit parameter by parameter | One command for any of the 63 models, edits included. The file opens normally in MR Configurator2 |
| The same settings on 20 drives | The same clicks 20 times | One script, with the same checks on every drive |
| A machine that stopped | Alarm display, alarm history, monitor, I/O and the no-rotation check, one window at a time | One request, and the agent reads them all |
| Remote support | Remote desktop, with someone logged in at the PC | SSH only, with no desktop session needed |
| Record of what was done | Clicks leave no trace | Each command and result can be logged and reviewed |
| AI agents | Can't use a GUI reliably | An MCP server with two tools |

## Tools

### MCP tools

| Tool | What it does |
|---|---|
| `mrc2` | Runs one command from the list below and returns JSON |
| `mrc2_run` | Runs a short Python script against the engines in one session, so several calls share one connection. Only the script's result comes back |

### Commands

| Command | What it does |
|---|---|
| `models` | The 63 amplifier models in MR Configurator2's catalogue |
| `catalog MODEL [PA04\|monitor\|pins\|graph]` | Parameter definitions with ranges, units, factory values and digit options; monitor items; connector pins |
| `api [REGEX]` | The 218 calls of the servo communication engine, with their signatures |
| `api --vt [REGEX]` | The 627 lower-level engine and project-data methods |
| `env` | MR Configurator2 and engine versions, servo USB devices and COM ports on the PC |
| `amp info` | Amplifier and motor model, option card, encoder and current alarm. The model is detected automatically |
| `amp alarms` | Current alarm and the alarm history |
| `amp monitor` | Status display values with names and units |
| `amp params [--group PA]` | Every parameter on the drive, labelled from MR Configurator2's catalogue |
| `amp set PA09=20 --yes` | Writes parameters by number or symbol, checked first and read back |
| `project new MODEL -o FILE [PA05=...]` | A new one-axis `.mrc2` project with factory settings |
| `project info FILE` | Project name, connection settings and axes |
| `project params FILE [--changed] [--find TEXT]` | Parameter values with number, symbol, name, unit and factory value |
| `project set FILE PA05=12345 -o OUT` | Edits parameters, with range and digit-option checks |
| `project points FILE` | Point table rows |
| `call MEMBER ARGS + MEMBER ARGS` | Any engine call, several in one session |
| `run script.py` | A Python script in one session |

Scripts (`run` or `mrc2_run`) can make any engine call, work on project files and run any
command above. I/O diagnosis, forced outputs, test operation, alarm reset and the drive's
rotation check are done from scripts for now.

## How it was checked

| Run | What was done | Result |
|---|---|---|
| New projects for every model | A project created and re-opened for each of the 63 models in MR Configurator2's catalogue | 63 of 63 passed |
| A new project in MR Configurator2 | An MR-JET-G project with PA06 = 3 and PB07 = 23.5, opened in MR Configurator2 | Opened with no warnings. The two values show as changed and the rest at factory |
| Edit and re-open | Parameters edited in MR-JE-A and MR-JET-G projects, saved and re-opened in a fresh session | The new values read back and the original files were unchanged |
| Reading a drive | An MR-JE-70B on USB with no motor connected: model detection, the current alarm and a 16-entry history, 52 monitor values, 320 parameters | All read. The 320 parameters took about 20 seconds |
| Writing a drive | PA09 changed from 28 to 27 and back to 28 | Each value read back from the drive. A batch with one out-of-range value wrote nothing |
| I/O and outputs | I/O diagnosis, and outputs CN3-13, CN3-15 and CN3-9 forced on and off | Each state confirmed by the drive's own output diagnosis |
| Diagnosis and test mode | The rotation check, data saved at the time of the alarm, test mode in and out, alarm reset, motor-less operation | All worked. Motor-less operation cleared the missing-encoder alarm |
| Backup | Every setting read from the drive into a new `.mrc2` file | The file reopens with the drive's non-factory values |

A command takes about 2 seconds, and a project edit about 10.

## Works with

- Claude Code, Claude Desktop, Cursor and other MCP clients.
- MR Configurator2 version 1.165X, the version it was tested with.
- The agent runs on a Mac, Linux or Windows machine. MR Configurator2 stays on its Windows PC
  and is reached over SSH, or everything runs on that one PC.
- Amplifiers on USB (tested) or RS-232C.

## Safety

- Writing parameters, resetting alarms, test operation, forced outputs and raw protocol frames
  need an explicit confirmation.
- Values are checked against MR Configurator2's limits before anything is written, and read
  back afterwards.
- There is no default PC. Each project names the PC it works on.
- Only one program can hold the servo link at a time, so close MR Configurator2's online
  connection first.

## Limits

- Real motion hasn't been tested yet, because the test drive had no motor.
- Online parameter reads and writes for MR-J5, MR-JET-G and a few other newer series aren't
  supported by the engine version tested. Their project files work.
- These features are built into the MR Configurator2 program, outside the engines it uses,
  and aren't covered yet: one-touch tuning, the graph display, the drive recorder, machine
  diagnosis, safety parameters and converting projects between models.
- Connections through a PLC or motion controller aren't covered yet.
- MR Configurator2 has to be installed and licensed on the Windows PC.

## Contact

For access, licensing or a demo:

<p>
  <a href="https://bibinvr.tech/contact"><img alt="Contact" src="https://img.shields.io/badge/Contact-bibinvr.tech-1f6feb?style=for-the-badge"></a>
</p>

---

<sub>MR2-Configurator-MCP-CLI is proprietary. This repository is an overview; the source is private.
MR Configurator2, MELSERVO and MELSOFT are trademarks of Mitsubishi Electric Corporation. This project
is independent and is not affiliated with or endorsed by Mitsubishi Electric.</sub>
