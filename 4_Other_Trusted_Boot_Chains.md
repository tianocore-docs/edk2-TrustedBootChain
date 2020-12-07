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

Other Trusted Boot chains
=========================

Besides EDK II, other firmware or firmware related boot loaders also
include the trusted boot chain.

coreboot
--------

[coreboot](https://github.com/coreboot) includes the measured boot flow.
It is a simplified version. Table 3 shows the usage in coreboot.

####### Table 3 Coreboot TPM PCR Usage 

(Source: [coreboot measured boot](https://doc.coreboot.org/security/vboot/measured_boot.html))

  **PCR Index**   **PCR Usage**
  --------------- --------------------------------------------
  0               Google vboot GBB flags
  1               Google vboot GBB HWID
  2               Core Root of Trust for Measurement (CRTM)
  3               Runtime data like hwinfo.hex or MRC cache.
  4               N/A
  5               N/A
  6               N/A
  7               N/A

[vboot](https://github.com/coreboot/vboot)
[2api.h](https://github.com/coreboot/vboot/blob/master/firmware/2lib/include/2api.h)
defines two PCRs:

-   **BOOT\_MODE\_PCR**(0) -- It is to record the digest based on the
    current developer and recovery mode flags in the Google Binary Blob
    (GBB)

-   **HWID\_DIGEST\_PCR**(1) -- It is to record the digest of the
    hardware ID (HWID) from the GBB.

In verstage,
[vboot\_logic.c](https://github.com/coreboot/coreboot/blob/master/src/security/vboot/verstage.c)
**verstage\_main**() calls **extend\_pcrs**() to extend two PCRs.
[tpm\_common.c](https://github.com/coreboot/coreboot/blob/master/src/security/vboot/tpm_common.c)
**vboot\_extend\_pcr**() calls
[2api.c](https://github.com/coreboot/vboot/blob/master/firmware/2lib/2api.c)
**vb2api\_get\_pcr\_digest**() to get the corresponding flags and HWID
digest from the GBB and extends them to the TPM.

Later, coreboot
[crtm.h](https://github.com/coreboot/coreboot/blob/master/src/security/tpm/tspi/crtm.h)
defines two PCRs:

-   **TPM\_CRTM\_PCR**(2) -- It is for Core Root of Trust for
    Measurement (CRTM) modules, including all stages, data and blobs.
    These include COREBOOT CBFS (bootblock, fallback/verstage), FW\_MAIN
    CBFS (fallback/romstage, fspm, fallback/postcar, fallback/ramstage,
    cpu\_microcode\_blob, fsps, vbt, fallback/dsdt.aml,
    fallback/payload), RO\_VPD, GBB, SI\_DESC, SI\_GBE.

-   **TPM\_RUNTIME\_DATA\_PCR**(3) -- It is for runtime changeable data.
    Such as CMOS, SI\_ME, RW\_NVRAM.

[crtm.c](https://github.com/coreboot/coreboot/blob/master/src/security/tpm/tspi/crtm.c)
**tspi\_measure\_cbfs\_hook**() is the hook function to measure
different components in the coreboot file system (CBFS) data. The
CBFS\_TYPE definition can be found at
[cbfs\_serialized.h](https://github.com/coreboot/coreboot/blob/master/src/commonlib/bsd/include/commonlib/bsd/cbfs_serialized.h).
For example:

-   CBFS\_TYPE\_MRC -- PCR2

-   CBFS\_TYPE\_STAGE -- PCR2

-   CBFS\_TYPE\_SELF -- PCR2

-   CBFS\_TYPE\_FIT -- PCR2

-   CBFS\_TYPE\_MRC\_CACHE -- PCR3

-   Other -- runtime data go to PCR3, non-runtime data go to PCR2.

After that,
[log.c](https://github.com/coreboot/coreboot/blob/master/src/security/tpm/tspi/log.c)
**tcpa\_log\_add\_table\_entry**() appends the log to a tcpa table.

Grub2
-----

[Grub2](https://github.com/rhboot/grub2) extends the trusted boot chain
from platform firmware into the OS. Table 4 shows the PCR usage in Grub.

####### Table 4 GRUB TPM PCR Usage 

(Source: [Grub2 Measured Boot](https://www.gnu.org/software/grub/manual/grub/html_node/Measured-Boot.html))

+---------------+-----------------------------------------------------+
| **PCR Index** | **PCR Usage**                                       |
+---------------+-----------------------------------------------------+
| 8             | Grub command line:                                  |
|               |                                                     |
|               | All executed commands (including those from         |
|               | configuration files) will be logged and measured as |
|               | entered with a prefix of "grub\_cmd: "              |
+---------------+-----------------------------------------------------+
|               | Kernel command line:                                |
|               |                                                     |
|               | Any command line passed to a kernel will be logged  |
|               | and measured as entered with a prefix of            |
|               | "kernel\_cmdline: "                                 |
+---------------+-----------------------------------------------------+
|               | Module command line:                                |
|               |                                                     |
|               | Any command line passed to a kernel module will be  |
|               | logged and measured as entered with a prefix of     |
|               | "module\_cmdline: "                                 |
+---------------+-----------------------------------------------------+
| 9             | Files:                                              |
|               |                                                     |
|               | Any file read by GRUB will be logged and measured   |
|               | with a descriptive text corresponding to the        |
|               | filename.                                           |
+---------------+-----------------------------------------------------+

Grub2
[tpm.h](https://github.com/rhboot/grub2/blob/master/include/grub/tpm.h)
defines two PCR index:

-   **GRUB\_STRING\_PCR**(8) -- It is for the command line string.

-   **GRUB\_BINARY\_PCR**(9) -- It is for a file binary.

[tpm.c](https://github.com/rhboot/grub2/blob/master/grub-core/commands/tpm.c)
registers **grub\_tpm\_verify\_string**() and
**grub\_tpm\_verify\_write**() to a grub\_file\_verifier structure. They
will be called by **grub\_verify\_string**() and
**grub\_verifiers\_open**() in
[verifiers.c](https://github.com/rhboot/grub2/blob/master/grub-core/commands/verifiers.c).

when grub2 executes a command line such as
GRUB\_VERIFY\_MODULE\_CMDLINE, GRUB\_VERIFY\_KERNEL\_CMDLINE,
GRUB\_VERIFY\_COMMAND or grub\_create\_loader\_cmdline() in
[cmdline.c](https://github.com/rhboot/grub2/blob/master/grub-core/lib/cmdline.c),
**grub\_verify\_string**() is used. Finally,
**grub\_tpm\_verify\_string**() measures the string to PCR8.

**grub\_verifiers\_open**() is registered as one of grub\_file\_filters
in
[file.h](https://github.com/rhboot/grub2/blob/master/include/grub/file.h).
Whenever grub uses
[file.c](https://github.com/rhboot/grub2/blob/master/grub-core/kern/file.c)
**grub\_file\_open**() this filter is invoked. Finally,
**grub\_tpm\_verify\_write**() measures the file binary to PCR9.

Linux Secure Boot Shim
----------------------

[Shim](https://github.com/rhboot/shim) is used to extend the UEFI secure
boot concept to Linux. Table 5 shows the PCR usage in Shim.

####### Table 5 Shim TPM PCR Usage

+---------------+-----------------------------------------------------+
| **PCR Index** | **PCR Usage**                                       |
+===============+=====================================================+
| 4             | UEFI application, such as second\_stage, FALLBACK,  |
|               | MOK\_MANAGER.                                       |
+---------------+-----------------------------------------------------+
| 7             | UEFI variable, such as "MokSBState".                |
|               |                                                     |
|               | Verification policy authority, such as "Shim",      |
|               | "db", "MokList".                                    |
+---------------+-----------------------------------------------------+
| 14            | UEFI variable, such as "MokList", "MokListX",       |
|               | "MokSBState".                                       |
+---------------+-----------------------------------------------------+

[shim.c](https://github.com/rhboot/shim/blob/main/shim.c)
**start\_image**() supports to execute a UEFI application image, such as
second\_stage, FALLBACK, MOK\_MANAGER. It calls **tpm\_log\_pe**() to
measure it to PCR4.

[Shim](https://github.com/rhboot/shim) defines a set of UEFI variable to
store the shim variable.
[mok.c](https://github.com/rhboot/shim/blob/main/mok.c)
**import\_mok\_state**() checks the mok\_state\_variables, such as
"MokList", "MokListX", "MokSBState", "MokDBState". In which, the
"MokSBState" variable is measured to PCR7 via
**tpm\_measure\_variable**(). The "MokList", "MokListX", "MokSBState"
variables are measured to PCR14 via **tpm\_log\_event**().

To follow the UEFI secure boot protocol,
[shim.c](https://github.com/rhboot/shim/blob/main/shim.c)
**verify\_one\_signature**() will record "Shim" as the authority via
**tpm\_measure\_variable**(), if the **AuthenticodeVerify**() succeeds.
Also **check\_db\_cert**()/**check\_db\_hash**() will record "db",
"MokList" as authority if verification succeeds.

Windows BitLocker
-----------------

Microsoft Windows BitLocker uses the below-listed PCRs. The legacy-boot
version of Windows used PCR\[8, 9, 10, 11\], while UEFI-based Windows
uses PCR\[11, 12, 13, 14\] for the BitLocker policies. Table 6 shows the
PCR usage in Windows BitLocker.

####### Table 6 Windows BitLocker PCR Usage 

(Source: [Windows BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings))

  --------------- -------------------------- ----------------------------------------
  **PCR Index**   **PCR Usage (Legacy)**     **PCR Usage (UEFI)**
  8               NTFS Boot Sector           Reserved
  9               NTFS Boot Block            Reserved
  10              Boot Manager               Reserved
  11              BitLocker access control   BitLocker access control
  12              Reserved                   Data events and highly volatile events
  13              Reserved                   Boot Module Details
  14              Reserved                   Boot Authorities
  --------------- -------------------------- ----------------------------------------
