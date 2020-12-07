<!--- @file
  Understanding the Trusted Boot Chain Implementation

  Copyright (c) 2020, Intel Corporation. All rights reserved.<BR>

  Redistribution and use in source (original document form) and 'compiled'
  forms (converted to PDF, epub, HTML and other formats) with or without
  modification, are permitted provided that the following conditions are met:

  1) Redistributions of source code (original document form) must retain the
     above copyright notice, this list of conditions and the following
     disclaimer as the first lines of this file unmodified.

  2) Redistributions in compiled form (transformed to other DTDs, converted to
     PDF, epub, HTML and other formats) must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in the
     documentation and/or other materials provided with the distribution.

  THIS DOCUMENTATION IS PROVIDED BY TIANOCORE PROJECT "AS IS" AND ANY EXPRESS OR
  IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
  MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
  EVENT SHALL TIANOCORE PROJECT  BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
  SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
  OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
  WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
  OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF
  ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

-->

Checklist for Platform Developers
===================================

PCR, Measurement and Attestation
--------------------------------

### General Guideline

1.1.1. Use an **even PCR for code** measurement in general.

1.1.2. Use an **odd PCR for data** measurement in general.

1.1.3. Do **NOT** record data that **that are dynamic and changeable
across the boot**, such as system clock, fan speed, boot count, system
reset reason, battery power, a nonce value, a pointer, etc.

1.1.4. Do **NOT** record the **instance of specific information that may
be used to unique identify a system**, such as an asset tag, a serial
number, etc.

1.1.5. Do **NOT** record **any privacy sensitive information**.

### PCR 0

1.2.1. Do configure
[PcdTcgPfpMeasurementRevision](https://github.com/tianocore/edk2/blob/master/MdeModulePkg/MdeModulePkg.dec)
to select TCG PFP compliance revision.

1.2.2. Do configure
[PcdFirmwareVersionString](https://github.com/tianocore/edk2/blob/master/MdeModulePkg/MdeModulePkg.dec)
to a valid Unicode string for version, so that it can be measured.

1.2.3. Do report **all FV information** in PEI, so that all of them can
be measured.

1.2.4. Do install
[EFI\_PEI\_FIRMWARE\_VOLUME\_INFO\_MEASUREMENT\_EXCLUDED\_PPI](https://github.com/tianocore/edk2/blob/master/SecurityPkg/Include/Ppi/FirmwareVolumeInfoMeasurementExcluded.h)
for the FV that is ready measured.

1.2.5. Do measure individual non-FV component, if it is loaded from the
platform firmware.

1.2.6. Do NOT update the loaded component before measuring it.

1.2.7. Do measure the non-host platform information using
[EV\_NONHOST\_INFO](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h),
if it exists.

1.2.8. Do measure the non-host platform component using
[EV\_NONHOST\_CODE](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h)
if it can only be updated by the platform firmware.

### PCR 1

1.3.1. Do measure Microcode.

1.3.2. Do measure SMBIOS table after filtering changeable data or
instance data.

1.3.3. Do measure "Entering ROM Based Setup" with
[EV\_ACTION](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h)
for a setup utility.

1.3.4. Do measure security related configuration data from non-volatile
storage, such as UEFI setup variable, or CMOS.

1.3.5. Do measure the hardware device list with
[EV\_TABLE\_OF\_DEVICES](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h).

1.3.6. Do measure the non-host platform configuration using
[EV\_NONHOST\_CONFIG](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h)
if it can only be updated by the platform firmware.

### PCR 2

1.4.1. Do link
[DxeTpm2MeasureBootLib](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/DxeTpm2MeasureBootLib)
as **LAST** NULL instance lib for SecurityStubDxe.inf in a DSC file.

1.4.2. Do measure the non-host platform component using
[EV\_NONHOST\_CODE](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h)
if it can be updated by entities other than the platform firmware.

1.4.3. Do measure SPDM-capable device hardware or firmware use
[EV\_EFI\_SPDM\_FIRMWARE\_BLOB](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h).

### PCR 3

1.5.1. Do measure "Entering ROM Based Setup" with
[EV\_ACTION](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h)
for a UEFI application based setup utility.

1.5.2. Do measure the non-host platform configuration using
[EV\_NONHOST\_CONFIG](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h)
if it can be updated by entities other than the platform firmware.

1.5.3. Do measure SPDM-capable device hardware configuration or firmware
configuration use
[EV\_EFI\_SPDM\_FIRMWARE\_CONFIG](https://github.com/tianocore/edk2/blob/master/MdePkg/Include/IndustryStandard/UefiTcgPlatform.h).

### PCR 4

1.6.1. Do measure the additional pre-OS code loaded by an UEFI
application the using EV\_COMPACT\_HASH.

### PCR 5

1.7.1 Do measure the additional data configuration related to the UEFI
application.

### PCR 6

N/A

### PCR 7

1.9.1. Do measure **security configuration** if it exits. (It means the
whole policy.)

1.9.2. Do measure **security authority** if it exits. (It means the
specific policy which is used to verify the component.)

1.9.3. Do measure the security feature disabling event, such as "UEFI
Debug Mode", "DMA Protection Disabled".

### NO\_ACTION event

1.10.1. Do record startup locality event, if an ACM starts the TPM.

TPM Device Startup
------------------

### Device Selection

2.1.1. Do NOT support TPM1.2.

2.1.2. Do choose a proper Tcg2Config driver for device selection, such
as
[Tcg2Config](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Tcg/Tcg2Config)
or platform specific one.

2.1.3. Do set
[PcdTpmInstanceGuid](https://github.com/tianocore/edk2/blob/master/SecurityPkg/SecurityPkg.dec)
directly after the TPM device selection.

2.1.4. Do set
[PcdTpm2InitializationPolicy](https://github.com/tianocore/edk2/blob/master/SecurityPkg/SecurityPkg.dec)
if the TPM device is started.

### TPM Device Interface

2.2.1. Do link proper
[Tpm2DeviceLib](https://github.com/tianocore/edk2/blob/master/SecurityPkg/Include/Library/Tpm2DeviceLib.h)
to Tcg2Dxe.inf and Tcg2Pei.inf in platform DSC, such as
[Tpm2DeviceLibDTpm](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/Tpm2DeviceLibDTpm)
or other TPM device such as I2C.

### Error Handling

2.3.1. Do register a ReportStatusCode callback handler to process the
TPM error, if the platform wants to reset system, or disable the TPM
hardware on error.

TCG Physical Presence
---------------------

3.1.1. Do call **Tcg2PhysicalPresenceLibNeedUserConfirm**() and
**Tcg2PhysicalPresenceLibProcessRequest**() in the platform BDS before
EndOfDxe.

3.1.2. Do **connect all trusted consoles** if
**Tcg2PhysicalPresenceLibNeedUserConfirm**() is TRUE.

### TPM Bank Selection

3.2.1. Do choose
[HashLibBaseCryptoRouter](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/HashLibBaseCryptoRouter)
if the platform wants to support crypto agile.

3.2.2. Do link proper multiple HashLib instances, such as
[HashInstanceLibSha256](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/HashInstanceLibSha256),
[HashInstanceLibSha384](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/HashInstanceLibSha384),
[HashInstanceLibSha512](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/HashInstanceLibSha512)
and
[HashInstanceLibSm3](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/HashInstanceLibSm3),
to Tcg2Pei.inf and Tcg2Dxe.inf in platform DSC.

3.2.3. Do NOT use
[HashInstanceLibSha1](https://github.com/tianocore/edk2/tree/master/SecurityPkg/Library/HashInstanceLibSha1).

### TPM Hierarchy Management

3.3.1. Do choose a proper Tcg2Platform module to manage the TPM platform
hierarchy, such as
[Tcg2Platform](https://github.com/tianocore/edk2-platforms/tree/master/Platform/Intel/MinPlatformPkg/Tcg)
or platform specific one.

3.3.2. Do randomize TPM platform auth before EndOfDxe.

3.3.3. Do randomize TPM platform auth before EndOfPei in S3 resume, if
TPM error happens.

3.3.4. Do send **Tpm2HierarchyControl**() command to enable or disable
the hierarchy, if it is supported.

TCG Memory Override
-------------------

4.1.1. Do check MOR variable and clear memory in memory initialization
if MOR request is set.

4.1.2. Do treat MOR variable missing as requested.

OS Interface
------------

### ACPI Table

5.1.1. Do configure
[PcdTpm2AcpiTableRev](https://github.com/tianocore/edk2/blob/master/SecurityPkg/SecurityPkg.dec)
to indicate TPM2 ACPI table version.

5.1.2. Do configure
[PcdTpmPlatformClass](https://github.com/tianocore/edk2/blob/master/MdeModulePkg/MdeModulePkg.dec)
for client or server.

5.1.3. Do configure
[PcdTcgPhysicalPresenceInterfaceVer](https://github.com/tianocore/edk2/blob/master/SecurityPkg/SecurityPkg.dec)
to indicate the TCG Physical Presence Interface version.

5.1.4. Do configure
[PcdTpm2CurrentIrqNum](https://github.com/tianocore/edk2/blob/master/SecurityPkg/SecurityPkg.dec)
and
[PcdTpm2PossibleIrqNumBuf](https://github.com/tianocore/edk2/blob/master/SecurityPkg/SecurityPkg.dec)
to indicate the TPM IRQ information.

### TCG2\_PROTOCOL

N/A

TCG Storage
-----------

### OPAL Password

6.1.1. Do **connect trusted storages and trusted consoles** in Platform
BDS before EndOfDxe if there is OPAL password request.

6.1.2. Do include storage disk drivers in PEI for S3 auto-unlock.

### OPAL Feature

6.2.1. Do **connect trusted storages and trusted consoles** in Platform
BDS before EndOfDxe if there is OPAL feature request.

### BlockSid

6.3.1. Do enable BlockSid by default.

### TPer reset

6.4.1. Do **connect trusted storages** in Platform BDS before EndOfDxe
if MOR request is set.

6.4.2. Do treat MOR variable missing as requested.
