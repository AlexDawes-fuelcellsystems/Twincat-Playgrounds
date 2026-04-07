# LAUDA Chiller Profinet Commissioning Procedure

## Purpose

This document describes how to commission communications between a TwinCAT PLC and the LAUDA chiller over Profinet.

This procedure assumes the LAUDA side uses the anybus-x profinet gateway.

## System Overview

- Beckhoff PLC / TwinCAT has a profinet port built-in.
- Anybus X-gateway is the Profinet device to interface between the PLCs.
- LAUDA chiller is connected on the gateway's opposite fieldbus/network side.

## Required Files

- `GSDML-V2.3-HMS-ANYBUS_X_GATEWAY_PROFINET_IO-20161110.xml`
- `chiller.xti` 

The Anybus device must be added manually.

Beckhoff reference:
https://infosys.beckhoff.com/english.php?content=../content/1033/tf6271_tc3_profinet_rt_controller/index.html&id=7887317531525096249

## Pre-Commissioning Checks

Before going online, confirm the following:

1. The Beckhoff PLC Ethernet connection is connected to the top port of the Anybus terminal/gateway.
2. The LAUDA / Siemens side is connected to the bottom side of the Anybus terminal, normally through the network switch as used on the machine.
3. The gateway, switch, PLC, and chiller are all powered.
4. Link/activity LEDs are healthy on all relevant ports.
5. The correct GSDML/device description files are available on the engineering PC.
6. You have the correct process data map for the LAUDA chiller signals.

## TwinCAT Configuration Procedure

### 1. Open the TwinCAT project

Open the correct TwinCAT solution and put the target into `Config` mode if changes to I/O configuration are required.

### 2. Scan for I/O devices

In the I/O tree:

1. Scan for hardware.
2. Confirm that a `Profinet RT Controller` device appears in the TwinCAT I/O configuration.

If the Profinet controller does not appear, resolve this before continuing.

### 3. Add the Anybus device

If the Anybus gateway is not available in the TwinCAT device catalogue:

1. Manually paste (with links) the Anybus XTI / device description file onto the Profinet controller. (`Chiller.xti`)
2. Ensure the HMS Anybus GSDML file is installed in  (..\TwinCAT\3.1\Config\Io\Profinet) on the Twincat PLC.
3. Refresh or reopen the project if required.
4. Go online and download project onto the PLC.


### 5. Check device identity and addressing

Confirm:

1. The configured station/device matches the physical Anybus gateway.
2. On the Profinet controller level, under Profinet, Scan PNIO devices, ensure that the abx-prt appears as defined in the device. (The expected device name and IP settings are correct)
3. GO to box states tab under the Profinet controller interface, ensure that Communication is established.
3. The I/O byte lengths match the gateway mapping from INTERFACE LIST DOCUMENT.

### 6. Link process data to PLC variables

Map the Profinet input and output data to PLC variables used by the application.

Pay special attention to data types and byte ordering, especially for `REAL` values.

## Online Commissioning and Verification

### 1. Download configuration

Activate the TwinCAT configuration and place the system online.

### 2. Confirm the Profinet device is healthy

Check that:

- Lights are all green on anybus terminal.
- Link is flickering on each side.
- Customer DCS connection goes green on LAUDA chiller HMI when remote mode is activated by setting remote to true on Twincat.


### 3. Prove the full comms path

Communications should only be considered commissioned once all of the following are true:

- Set the chiller into remote mode.
- Change the setpoint and verify that the actual chiller setpoint updates to follow. (Bit mapping may need to be changed around on the Twincat PLC output)
- Ensure all read values demonstrate reasonable values.


## Data Handling Note for REAL Values

Check byte order carefully when reconstructing `REAL` values in TwinCAT.

Observed note from commissioning:

- bits `0..7` of the TwinCAT variable correspond to byte `4` of the REAL value data packet
- bits `24..31` correspond to the first byte of the packet

This indicates byte ordering must be checked explicitly rather than assumed. If values look incorrect, unstable, or wildly scaled, verify:

1. Byte order
2. Word order
3. Signed/unsigned interpretation
4. Scaling applied in PLC code

## Common Faults to Check

- No link lights on the Anybus ports
- Wrong Ethernet port used on the Anybus gateway/Twincat PLC
- Profinet controller not present in TwinCAT I/O
- GSDML file missing on the PLC config folder
- Wrong .xti file loaded in
- Wrong device name or IP assignment
- Module layout in TwinCAT does not match gateway configuration (Profinet controller type etc)
- Data type mismatch between gateway map and PLC variables
- REAL values decoded with incorrect byte order, typically nonsensical values like 5.766764E-41

## Relevant files needed

- `GSDML-V2.3-HMS-ANYBUS_X_GATEWAY_PROFINET_IO-20161110.xml`
- `chiller.xti` 
- `InterfaceList.pdf`