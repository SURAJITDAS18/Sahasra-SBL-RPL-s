Slim Bootloader
================

**Welcome to the Slim Bootloader project!**

Slim Bootloader is an open-source boot firmware solution, built from the
ground up to be secure, lightweight, and highly optimized while leveraging
robust tools and libraries from the [EDK II](https://www.tianocore.org/edk2/)
framework.

Sahasra-SlimBoot-Loader
Slim Bootloader (SBL) IFWI Stitching Guide for Intel Raptor Lake-S This guide covers setting up the IFWI stitching environment on Ubuntu for your Intel Raptor Lake-S (RPL-S) motherboard, using the Slim Bootloader (SBL) build output and Intel's stitching tools. Based on your previous troubleshooting, you have already: • Cloned the Slim Bootloader repository into ~/slimbootloader. • Built or prepared the RPL-S SBL components under Outputs/rpls. • Obtained Stitch_Components.zip. • Extracted Intel stitching tools into ~/Stitching. • Fixed execute permissions for mfit, bpmgen2, and meu. The next step is to make sure the environment, input files, configuration, and stitching command are correct.

Understand the stitching process The general workflow is:
Set up the Ubuntu build environment.
Build SBL for the RPL-S platform.
Prepare the Intel FIT, MEU, BPMGen2, and MFIT tools.
Prepare the required firmware inputs.
Configure the SBL stitching script.
Run IFWI stitching.
Verify the resulting IFWI image before flashing it. Important: Building SBL and stitching the IFWI are separate operations. A successful SBL build does not guarantee a successful IFWI stitch. Intel's Raptor Lake documentation confirms that rpls is the platform identifier for Raptor Lake-S DDR5 UDIMM/SODIMM boards. The firmware ingredients must also match the target platform.
Step 1: Download the Latest SBL Source Code Please clone the latest Slim Bootloader source code from the official repository using the command below: git clone https://github.com/slimbootloader/slimbootloader.git slimbootloader cd slimbootloader git submodule update --init --recursive Step 2: Apply the Provided Patch Navigate to the cloned repository directory and apply the patch file provided: cd slimbootloader git apply sbl_uatx_enablement.patch Note: Please ensure that the patch file is placed in the correct directory before running the command above. Error and solution: Finding in slimbootloader: C:\Users\Leena\slimbootloader>git apply sbl_uatx_enablement.patch error: No valid patches in input (allow with "--allow-empty") FINDSTR: Warning - input file ... is in Unicode format. Convert the patch to UTF-8 without changing its patch content: Solution powershell -NoProfile -Command "$p=Get-Content .\sbl_uatx_enablement.patch -Raw; [System.IO.File]::WriteAllText('.\sbl_uatx_enablement_utf8.patch',$p,(New-Object System.Text.UTF8Encoding($false)))" After that git apply sbl_uatx_enablement_utf8.patch

Step 3: Build the UEFI Payload from the SBL Software Package Please refer to the following Intel® Resource & Documentation Center (RDC) entry to obtain the UEFI Payload required for this build: RDC Reference Number: RDC #789633 Release Note #863411 From 863411 follow section 5.6 as mentioned below: To build UEFI payload, follow the steps:

Extract UefiPldPlatform.zip to the UefiPldPlatform folder and navigate it to the directory in the command line window.
Clone EDK II specified version to Edk2Open folder. Example: git clone https://github.com/tianocore/edk2.git Edk2Open cd Edk2Open git checkout edk2-stable202411 git submodule update --init --recursive cd .. If error occur: Do: git config --system core.longpaths true Then run git submodule update --init –recursive cd..
Execute the following command to build UEFI payload: To build DEBUG UEFI payload binary: python BuildPayload.py build Rpls To build RELEASE UEFI payload binary: python BuildPayload.py build Rpls -r On a successful build, the UEFI payload binary should be generated in the folder Build\RplPlatformPkgX64_CLANGDWARF\UniversalPayload.elf. Note: Motive is to get UniversalPayload.elf, location may be vary. Copy built binary to below path copy UniversalPayload.elf to slimbootloarder\PayloadPkg\PayloadBins\UniversalPayload.elf Note: You have to make PayloadBins folder manually in the above path. Step 4: Copy the attached delta file in (CfgDataInt_Rpls_Rvp_Uatx_Ddr5.dlt) Platform\RaptorlakeBoardPkg\CfgData Step 5: Build the SBL Image with UEFI Payload Run the following build command to compile the SBL source code: python BuildLoader.py build rpls  -p "OsLoader.efi:LLDR:Lz4;UniversalPayload.elf:UEFI:Lzma" Error: Sblkeys Solution: Set the SBL_KEY_DIR environment to specify the location for SBL keys. By default, SBL_KEY_DIR is set to SblKeys. See section 5.2 on SBL keys. If there is an error during the SBL build due to these SBL keys, you may need to set the path manually to the SblKeys folder path as per below: set SBL_KEY_DIR= <full_SblKeys_folder_path> Slimbootloader.bin image will be generated in the Outputs\rpls folder.
Check the Ubuntu environment Open a terminal and run: cd ~/slimbootloader
python3 --version git --version openssl version nasm -v iasl -v

Check that the SBL source tree and RPL-S stitching files exist: ls -l Platform/RaptorlakeBoardPkg/Script/StitchIfwi.py ls -l Platform/RaptorlakeBoardPkg/Script/StitchIfwiConfig_rpls.py ls -lh Outputs/rpls/Stitch_Components.zip If any of these files are missing, stop and resolve that issue before proceeding. 3. Build SBL for Raptor Lake-S If you have already built SBL successfully, you can skip this step. From the SBL repository: cd ~/slimbootloader python3 BuildLoader.py build rpls

Check the generated files: ls -lh Outputs/rpls/

You should see the relevant SBL output files, including SlimBootloader.bin and, if generated by your build configuration, Stitch_Components.zip. The official SBL documentation identifies rpls as the Raptor Lake-S build target.

Prepare the Intel stitching workspace Your existing workspace is:
/home/surajit/Stitching/ ├── Fit/ ├── BpmGen2/ ├── Meu/ ├── Input/ ├── Temp/ ├── Intel_CSME_ADL-RPL-S_16.1.40.2765v3_Consumer/ └── ...

Check the required tools: ls -l ~/Stitching/Fit/ ls -l ~/Stitching/BpmGen2/ ls -l ~/Stitching/Meu/

Check their executable permissions:

chmod u+x ~/Stitching/Fit/mfit chmod u+x ~/Stitching/BpmGen2/bpmgen2 chmod u+x ~/Stitching/Meu/meu

Check that each tool starts: ~/Stitching/Fit/mfit ~/Stitching/BpmGen2/bpmgen2 ~/Stitching/Meu/meu Note: Some tools print help text or report invalid command-line options when run without arguments. That does not necessarily indicate a problem; the stitching script normally supplies the required arguments. Also check OpenSSL: which openssl openssl version

The stitching script and Intel tools may have specific version or dependency requirements. Use the versions supplied or approved for your RPL-S firmware package rather than substituting arbitrary versions. 5. Verify the firmware input files Run: ls -lh ~/Stitching/Input/

Your previous workspace contained these files: File General purpose acm0.bin Authenticated Code Module CsePlugin#NPHY.bin CSE-related plugin component EcRegion.bin Embedded Controller firmware region EcRegionPointer.bin EC region pointer information GbeRegion.bin Gigabit Ethernet firmware region IshImage.bin Integrated Sensor Hub firmware MeRegionFile.bin Intel CSME firmware region PchcSubPartitionData.bin CSE/PCH-related firmware data PdtBinary.bin Platform data component PhyBinaryFile.bin PHY firmware component PmcBinary.bin Power Management Controller firmware

These names are from your existing workspace; the precise files required depend on the RPL-S stitching configuration and Intel firmware package. Check for the BPMGen2 parameter file: find ~/Stitching/BpmGen2 -maxdepth 2 -type f -printf '%f\n' If bpmgen2.params is missing, don't create an arbitrary replacement. Check the instructions for your specific Intel BPMGen2 package. Important: Do not download firmware components from unrelated platforms or mix firmware releases simply because their filenames match. Use the correct Intel RPL-S firmware ingredients for your target board. 6. Check the stitching script arguments Before stitching, display the script's help: cd ~/slimbootloader

python3 Platform/RaptorlakeBoardPkg/Script/StitchIfwi.py -h Your current command uses these arguments: Argument Purpose -s SBL stitching components ZIP -c Platform-specific stitching configuration -w Intel stitching workspace -p rpls Select the Raptor Lake-S platform -b legacy Select the Legacy Boot Guard profile -o s14 Platform-specific stitching option, interpreted by the script/configuration

The exact accepted options depend on your checked-out SBL version and the RPL-S configuration script. Before using -b legacy, confirm that this profile is appropriate for your target board and its Boot Guard provisioning. Do not assume that the same setting is suitable for every production board. 7. Run IFWI stitching Once the environment and firmware inputs are verified, run your existing command: cd ~/slimbootloader

python3 Platform/RaptorlakeBoardPkg/Script/StitchIfwi.py
-s Outputs/rpls/Stitch_Components.zip
-c Platform/RaptorlakeBoardPkg/Script/StitchIfwiConfig_rpls.py
-w /home/surajit/Stitching
-p rpls
-b legacy
-o s14
