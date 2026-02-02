# Investigation of eSim 2.5 Installation on Ubuntu 25.10

## Overview

eSim version 2.5 is officially validated only for Ubuntu releases up to 24.04.  
This document presents an independent investigation into installing eSim 2.5 on Ubuntu 25.10, a newer and currently unsupported Ubuntu release.

The focus of this work is to identify installation failures introduced by the operating system upgrade, analyze the installer scripts, and apply minimal, maintainable changes to enable partial functionality.

Rather than forcing a complete installation, the objective is to expose incompatibilities, document their causes, and outline realistic solutions for future support.

---

## Test Environment

- **Operating System:** Ubuntu 25.10 (VirtualBox environment)
- **Application:** eSim 2.5
- **Installer Used:** Official `install-eSim.sh`
- **System Architecture:** x86_64

---

## Observed Problems and Resolutions

### 1. Installer Rejection of Ubuntu 25.10 (Resolved)

#### Observation

The installer terminates early with the following message:

Unsupported Ubuntu version: 25.10 ()


This prevents dependency evaluation and further troubleshooting.

#### Analysis

The installer validates the operating system using a hardcoded `VERSION_ID` check and does not recognize Ubuntu 25.x releases.

#### Modification

Ubuntu 25.10 was temporarily mapped to the Ubuntu 24.04 installer logic to allow controlled execution:

``bash
"24.04"|"25.10")
    SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
    ;;
Outcome
The installation process proceeds beyond the version check

No assumption of full compatibility is introduced

2. KiCad Repository Incompatibility (Resolved)
Observation
The official KiCad PPA fails to add on Ubuntu 25.10, causing the installer to terminate.

Analysis
Third-party PPAs often lag behind development or pre-release Ubuntu versions.

Modification
Skipped addition of the unsupported KiCad PPA

Installed KiCad from Ubuntu’s default repositories

Documented Snap-based installation as an optional alternative

echo "Skipping KiCad PPA for unsupported Ubuntu version"
sudo apt install -y kicad
Outcome
KiCad installs successfully

The installation flow continues without interruption

3. NGHDL and Verilator Setup Failures (Partially Mitigated)
Observation
Multiple issues were encountered:

Missing GTK-related libraries

NGHDL extracted into an unexpected directory

Verilator dependency inconsistencies

Actions Taken
Non-essential GUI dependencies were commented out

NGHDL extraction was corrected manually

tar xvf $ghdl.tar.gz
Rationale
These components are not required for core schematic capture and simulation workflows. Skipping them allows attention to be focused on more critical toolchain failures.

4. GHDL Incompatibility with LLVM 20 (Root Cause Identified)
Observation
The following error occurs during GHDL setup:

Unhandled version llvm 20.1.8
Analysis
Ubuntu 25.10 ships with LLVM 20

GHDL 4.1.0 officially supports LLVM versions up to 15

The installer deploys a GHDL build intended for older Ubuntu releases

This results in a hard failure due to unsupported LLVM versions.

Attempted Workarounds
Installed LLVM 15 and Clang 15 alongside the system LLVM

sudo apt install llvm-15 clang-15
Considered forcing GHDL to link against LLVM 15 using environment configuration or symbolic links

Status
The issue remains unresolved, but the incompatibility has been conclusively identified.

Required Long-Term Fix
Compile GHDL from source with LLVM 20 support
or

Use containerized or version-pinned toolchains

Summary and Conclusions
This investigation demonstrates how major Ubuntu upgrades can disrupt tightly coupled dependency chains in EDA tools such as eSim.

Key Outcomes
Enabled installer execution on Ubuntu 25.10 for diagnostic purposes

Removed KiCad-related installation blockers

Isolated the LLVM–GHDL incompatibility as the primary unresolved issue

Documented changes that are minimal, reversible, and maintainable

Although full functionality on Ubuntu 25.10 depends on upstream updates, this work provides a clear technical baseline for future support and development.

