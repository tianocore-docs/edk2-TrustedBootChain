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

References
==========

Books
-----

\[Building Secure Firmware\] Jiewen Yao, Vincent Zimmer, *Building
Secure Firmware: Armoring the Foundation of the Platform*, 2020, Apress,
[https](https://intel-my.sharepoint.com/personal/vincent_zimmer_intel_com/Documents/Documents/https)[://www.amazon.com/gp/product/1484261054/](https://www.amazon.com/gp/product/1484261054/),

<https://link.springer.com/content/pdf/10.1007%2F978-1-4842-6106-4.pdf>

\[TPM2.0 Book\] Will Arthur, David Challener, *A Practical Guide to
TPM2.0*, 2015, Apress,
<https://www.amazon.com/Practical-Guide-TPM-2-0-Platform-ebook/dp/B0781D8J6W>,
<https://link.springer.com/book/10.1007%2F978-1-4302-6584-9>

Specifications
--------------

\[NIST SP800-155\] BIOS Integrity Measurement Guidelines, 2011,
<https://csrc.nist.gov/publications/detail/sp/800-155/draft>

\[NIST SWID\] Software Identification SWID,
<https://csrc.nist.gov/projects/Software-Identification-SWID>

\[NIST IR8086\] NISTIR.8060 Guidelines for the Creation of Interoperable
SWID Tags, <https://csrc.nist.gov/publications/detail/nistir/8060/final>

\[IETF CoSWID\] Concise Software Identification Tags,
<https://datatracker.ietf.org/doc/draft-ietf-sacm-coswid/>

\[IETF CoSWID-RIM\] Reference Integrity Measurement Extension for
Concise Software Identities,
<https://datatracker.ietf.org/doc/draft-birkholz-rats-coswid-rim/>

\[TCG RIM-IM\] TCG Reference Integrity Manifest Information Model
Specification, 2020,
<https://trustedcomputinggroup.org/resource/tcg-reference-integrity-manifest-rim-information-model/>

\[TCG PC Client RIM\] TCG PC Client Reference Integrity Manifest
Specification, 2020,
<https://trustedcomputinggroup.org/resource/tcg-pc-client-reference-integrity-manifest-specification/>

\[TCG PC Client FIM\] TCG PC Client Platform Firmware Integrity
Measurement Specification, 2020,

\[TCG PC Client PFP\] TCG PC Client Platform Firmware Profile
Specification, 2020,
<https://trustedcomputinggroup.org/resource/pc-client-specific-platform-firmware-profile-specification/>,
<https://trustedcomputinggroup.org/wp-content/uploads/TCG_PCClient_PFP_r1p05_v22_02dec2020.pdf>

\[TCG PC Client PTP\] TCG PC Client Platform TPM Profile Specification,
2020,
<https://trustedcomputinggroup.org/resource/pc-client-platform-tpm-profile-ptp-specification/>,

\[TCG Physical Presence\] TCG PC Client Platform Physical Presence
Interface Specification, 2015,
<https://trustedcomputinggroup.org/resource/tcg-physical-presence-interface-specification/>

\[TCG MOR\] TCG PC Client Platform Reset Attack Mitigation
Specification, 2019,
<https://trustedcomputinggroup.org/resource/pc-client-work-group-platform-reset-attack-mitigation-specification/>

\[TCG TPM ACPI\] "TCG ACPI Specification, 2017,
<https://trustedcomputinggroup.org/resource/tcg-acpi-specification/>

\[TCG UEFI Protocol\] TCG EFI Protocol Specification, 2016,
<https://trustedcomputinggroup.org/resource/tcg-efi-protocol-specification/>

\[TCG Storage\] TCG Storage Architecture Core Specification,
<https://trustedcomputinggroup.org/tcg-storage-architecture-core-specification/>

\[TCG SIIS\] TCG Storage Interface Interactions Specification,
<https://trustedcomputinggroup.org/resource/storage-work-group-storage-interface-interactions-specification/>

\[TCG OPAL\] Storage Work Group Storage Security Subsystem Class: Opal,
<https://trustedcomputinggroup.org/storage-work-group-storage-security-subsystem-class-opal/>

\[TCG Pyrite\] Storage Work Group Storage Security Subsystem Class:
Pyrite,
<https://trustedcomputinggroup.org/resource/tcg-storage-security-subsystem-class-pyrite/>

\[TCG Ruby\] Storage Work Group Storage Security Subsystem Class: Ruby,
<https://trustedcomputinggroup.org/resource/tcg-storage-security-subsystem-class-ruby-specification/>

\[TCG BlockSID\] TCG Storage Feature Set: Block SID Authentication,
2015,
<https://trustedcomputinggroup.org/resource/tcg-storage-feature-set-block-sid-authentication-specification/>,
<https://trustedcomputinggroup.org/wp-content/uploads/TCG_Storage_BlockIDAuth_v1p01_r1p14_13jan2021.pdf>

\[TCG PSID\] TCG Storage Feature Set: PSID,
<https://trustedcomputinggroup.org/resource/tcg-storage-opal-feature-set-psid/>

\[TCG Vendor ID\] TCG TPM Vendor ID Registry,
<https://trustedcomputinggroup.org/resource/vendor-id-registry/>

\[TCG DICE Identity\] Implicit Identity Based Device Attestation, 2018,
<https://trustedcomputinggroup.org/resource/implicit-identity-based-device-attestation/>

\[TCG DICE Symmetric Identity\] Symmetric Identity Based Device
Attestation, 2020,
<https://trustedcomputinggroup.org/resource/symmetric-identity-based-device-attestation/>

\[TCG DICE Layer\] DICE Layering Architecture, 2020,
<https://trustedcomputinggroup.org/resource/dice-layering-architecture/>

\[TCG Server Domain\] TCG Server Management Domain Firmware Profile,
2020,
<https://trustedcomputinggroup.org/wp-content/uploads/TCG_ServerManagementDomainFirmwareProfile_v1p00_11aug2020.pdf>

\[ACPI\] ACPI specification, 2019, <https://www.uefi.org/specifications>

\[ACPI and PnP vendor IDs\] <https://uefi.org/PNP_ACPI_Registry>

\[SMBIOS\] DSP0134 System Management BIOS specification, 2020,
<https://www.dmtf.org/standards/smbios>

\[SPDM\] DSP0274 Security Protocol and Data Model Specification, 2020,
<https://www.dmtf.org/standards/pmci>

\[Cerberus\] Project Cerberus Architecture Overview, 2018,
[https://github.com/opencomputeproject/Project\_Olympus/blob/master/Project\_Cerberus](https://github.com/opencomputeproject/Project_Olympus/blob/master/Project_Cerberus/Project%20Cerberus%20Architecture%20Overview.pdf)

\[AMD\] AMD Architecture Programmer's Manual, 2019,
<https://developer.amd.com/resources/developer-guides-manuals/>

\[Intel SDM\] Intel 64 and IA-32 Architecture Software Developer
Manuals, 2019, <https://software.intel.com/en-us/articles/intel-sdm>

\[Intel FSP\] Intel Firmware Support Package External Architecture
Specification,
<https://software.intel.com/content/www/us/en/develop/articles/intel-firmware-support-package.html>

\[TrustZone\] ARM TrustZone,
<https://developer.arm.com/ip-products/security-ip/trustzone>

Internet Links
--------------

\[TCG\] Trusted Computing Group, <https://trustedcomputinggroup.org/>

\[CCC\] Confidential Computing Consortium,
<https://confidentialcomputing.io/>

\[TEE\] Confidential Computing: Hardware-Based Trusted Execution for
Applications and Data, 2020,
<https://confidentialcomputing.io/white-papers/>

\[EDK II\] <https://github.com/tianocore/edk2>

\[EDK II TPM2\] Jiewen Yao, Vincent Zimmer, "A Tour Beyond BIOS - with
the UEFI TPM2 Support in EDK II", 2014,
<https://software.intel.com/sites/default/files/managed/94/2d/a_tour_beyond_bios_implementing_tpm2_support_in_edkii.pdf>

\[EDK II Tcg2DumpLog\] Tcg2DumpLog tool,
<https://github.com/jyao1/EdkiiShellTool/tree/master/EdkiiShellToolPkg/Tcg2DumpLog>

\[EDK II TPM Emulator\] EDK II TPM Emulator,
<https://github.com/jyao1/edk2/tree/feature_tpm_emulator/EmulatorPkg/Tpm2>

\[UEFI TCTI\] <https://github.com/tpm2-software/tpm2-tcti-uefi>

\[EDK II Tpm2TssPkg\] EDK II Tpm2TssPkg,
<https://github.com/flihp/edk2/tree/tpm2-tss/Tpm2TssPkg>

\[EDK II DeviceSecurityPkg\] EDK II DeviceSecurityPkg,
<https://github.com/jyao1/edk2/tree/DeviceSecurity/DeviceSecurityPkg>

\[EDK II FSP Manifest Tool\] FSP manifest tool,
<https://github.com/jyao1/FSP/tree/FspAttestation/Tools/ManifestTools>

\[TPM2 Software\] <https://github.com/tpm2-software>

\[Remote Attestation\]
<https://tpm2-software.github.io/tpm2-tss/getting-started/2019/12/18/Remote-Attestation.html>

\[Microsoft TPM2 Simulator\]
<https://github.com/microsoft/ms-tpm-20-ref>

\[TCG TPM2 Simulator\] <https://github.com/stwagnr/tpm2simulator>

\[OpenPower Trusted Boot\] OpenPOWER secure and trusted boot , Part 1:
Using trusted boot on IBM OpenPOWER servers, 2017,
<https://developer.ibm.com/articles/trusted-boot-openpower/>

\[coreboot\] <https://review.coreboot.org/>,
<https://github.com/coreboot>

\[coreboot MeasuredBoot\] coreboot Measured Boot,
<https://doc.coreboot.org/security/vboot/measured_boot.html>

\[GRUB\] <http://www.gnu.org/software/grub/>,
<https://github.com/rhboot/shim>

\[Grub Measured Boot\]
<https://www.gnu.org/software/grub/manual/grub/html_node/Measured-Boot.html>

\[openspdm\] <https://github.com/jyao1/openspdm>

\[Windows BitLocker\] Windows BitLocker Group Policy Setting,

<https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings>

\[UEFI TCG\] Vincent Zimmer, Shiva Dasari (IBM), Sean Brogan (IBM),
"Trusted Platforms:  UEFI, PI, and TCG-based firmware," Intel/IBM
whitepaper, September 2009,
<http://www.cs.berkeley.edu/~kubitron/courses/cs194-24-S14/hand-outs/SF09_EFIS001_UEFI_PI_TCG_White_Paper.pdf>

\[TCG-CW\] Ned Smith, A Comparison of the trusted Computing Group
Security Model with Clark-Wilson, 2014,
<https://www.semanticscholar.org/paper/A-Comparison-of-the-trusted-Computing-Group-Model-Smith/fa82426d99b86d1040f80b8bd8e0ac4f785b29a6>

\[Windows DMA Protection\] Kernel DMA Protection (Memory Access
Protection) for OEMs,

<https://docs.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-kernel-dma-protection>

\[Windows Secured-Core PC\] Force firmware code to be measured and
attested by Secure Launch on Windows 10,
<https://www.microsoft.com/security/blog/2020/09/01/force-firmware-code-to-be-measured-and-attested-by-secure-launch-on-windows-10/>

\[KeyStone\] Keystone Enclave, <https://keystone-enclave.org/>

