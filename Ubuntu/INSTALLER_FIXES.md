#List of Issues encountered while installing eSim on the latest version of Ubuntu that is 25.04
#The following list contains of issues listed orderwise (the order of encountering the issues)
#The difficulty of the issues has also been listed alongside them

## I used Oracle VM to install Ubuntu image for 25.04 version on an existing Windows System which in any case does not differ from latter Ubuntu OS used primarily.


#Issue 1
Issue 1 – eSim installer did not support Ubuntu 25.04

Difficulty: Easy
Impact: High (installer could not run on new Ubuntu release)

The existing eSim installer script was not designed to work on Ubuntu 25.04. When I tried to run the installer on Ubuntu 25.04, it failed because there was no specific handling for this version in the script, even though the environment was otherwise compatible. This meant that users on Ubuntu 25.04 could not install eSim using the official script, which is a high‑impact problem because it completely blocks installation on that distribution.

Fix for Issue 1 – Reusing Ubuntu 24.04 installer logic for Ubuntu 25.04

To quickly enable support for Ubuntu 25.04, I updated the installer so that it treats Ubuntu 25.04 in the same way as Ubuntu 24.04. In practice, this means reusing the existing, tested installation logic for Ubuntu 24.04 when the script detects Ubuntu 25.04. This is a simple change (low implementation difficulty), but it has high impact because it immediately allows users on Ubuntu 25.04 to install and use eSim with the same steps that already work on Ubuntu 24.04.






#Issue 2
Issue 2 – KiCad installation step failing or unreliable

Difficulty: Medium–High
Impact: High for GUI + PCB workflow

The main eSim installer is responsible for installing KiCad, which is required for schematic and PCB design workflows in eSim. 
On the tested Ubuntu setup, the KiCad installation step in the installer script was not working reliably: the commands or package names used did not match the currently supported Ubuntu versions, so KiCad was either not installed at all or an unusable version was installed. 
This meant that even if eSim itself was installed, the user could not properly use the GUI + PCB design flow without manually fixing KiCad. 
Because KiCad is one of the core external tools that eSim depends on, this issue has high impact on usability, and medium–high difficulty since it requires understanding distribution‑specific package availability and aligning the installer with the officially supported Ubuntu versions.

Fix for Issue 2 – KiCad installation (Flatpak instead of apt)

To fix the KiCad installation problem, I switched the installer to use KiCad from Flatpak instead of the Ubuntu apt repositories. The apt‑based installation was pulling a KiCad build that depended on libgit2-1.8, which is not available (or not in the expected version) on the tested Ubuntu release, causing installation or runtime failures. By installing KiCad via Flatpak, the installer now gets a self‑contained KiCad package with the correct dependencies bundled, so it no longer depends on the system’s libgit2-1.8 package. This makes the KiCad installation step more reliable across supported Ubuntu versions and ensures that eSim’s GUI and PCB design workflows work out‑of‑the‑box.




#Issue 3
Issue 3 – KiCad library copy step failing

Difficulty: Medium
Impact: Medium (KiCad integration / components not available)

The eSim installer includes a step to copy KiCad libraries into the eSim environment so that symbols, footprints, and example projects work correctly. On the tested Ubuntu setup, this step was failing because the script assumed fixed KiCad installation paths and existing destination directories. When the actual KiCad paths did not match these assumptions, or when the target directories did not exist, the copy operation either failed with errors or silently skipped some files. This left the installation in a partially configured state: KiCad might be installed, but some libraries or resources required by eSim were missing, which degraded the user experience and could break some example designs.

Fix for Issue 3 – KiCad library copy step

To fix the KiCad library copy problem, I updated the installer logic so that it no longer relies on hard‑coded paths or pre‑existing directories. First, I aligned the source paths with the actual KiCad installation location used in the new setup (the same location from which KiCad is now installed). Then, before copying, the script explicitly creates the required target directories so that the copy command cannot fail due to missing folders. With these changes, the KiCad libraries and related files are consistently copied into the correct eSim locations, and the KiCad integration (symbols, footprints, and examples) works as expected after installation.




#Issue 4
Issue 4 – GTK3 related failure in installer

Difficulty: Medium
Impact: Medium–High (affects GUI components)

One more problem I observed was related to GTK3 support during the installation. A GTK3‑related check or configuration step in the installer was failing on the target Ubuntu system, even though the system was capable of running GTK3 applications. Because this check was treated as a hard requirement, the failure caused the installation or configuration of the GUI components to stop, instead of either installing the missing pieces or proceeding safely. As a result, the eSim graphical interface could not be used reliably until this GTK3 issue was manually worked around, which significantly hurt the out‑of‑the‑box experience for users.

Fix for Issue 4 – GTK3 related failure

To resolve the GTK3‑related failure, I adjusted the installer so that this check no longer blocks a valid installation. Instead of letting a failing GTK3 check stop the process, the installer now ensures that the required GTK3 packages are present (or that the correct GTK3 option is enabled) and then continues normally. Where appropriate, the check was relaxed or corrected so that systems which already have usable GTK3 support are not treated as failures. As a result, the GUI components of eSim can be installed and used without manual workarounds, and the installer behaves more reliably across supported Ubuntu versions.



#Issue 5
Issue 5 – NGHDL installer did not support Ubuntu 25.04

Difficulty: Easy
Impact: High (NGHDL / mixed‑signal flow blocked)

The NGHDL installation script shipped with eSim was also not handling Ubuntu 25.04. The script only recognised older Ubuntu versions, so on Ubuntu 25.04 it did not select any valid installation path and effectively failed to install NGHDL. Since NGHDL is required for mixed‑signal simulations in eSim, this meant that users on Ubuntu 25.04 could not use that part of the tool at all, which is a high‑impact limitation even though the underlying system was capable of running NGHDL.


Fix for Issue 2 – Reusing Ubuntu 24.04 logic for NGHDL on Ubuntu 25.04

To fix this, I extended the NGHDL installer so that Ubuntu 25.04 is treated the same way as Ubuntu 24.04. In practice, the script now detects Ubuntu 25.04 and applies the same installation steps and package configuration that are already known to work on Ubuntu 24.04. This is a straightforward change, but it immediately unblocks NGHDL installation on Ubuntu 25.04 and restores mixed‑signal simulation support on that distribution.




#Issue 6
Issue 6– NGHDL / GHDL LLVM backend fails to build on newer Ubuntu versions

Difficulty: High
Impact: High (core mixed‑signal simulation break)

On newer Ubuntu versions, the NGHDL part of the eSim installer was failing while building the GHDL LLVM backend because the new system uses llvm-config-20.
The installer script assumed that LLVM 15 was installed and that llvm-config-15 was available at the fixed path /usr/bin/llvm-config-15.
On systems where this exact binary did not exist (for example, a different LLVM version was installed, or llvm-config-15 was not in that path), the ./configure step for GHDL failed, and the NGHDL installation stopped.
This issue is high impact because it blocks mixed‑signal simulation in eSim, and high difficulty because it involves understanding the LLVM toolchain, version compatibility, and GHDL’s build configuration.


Fix : Made the script to temporarily use llvm-config 15 for stability and check for it and if not checked then install llvm-config-15 and point to its path in the installation script install-nghdl-24.04.sh



##The above fixes made were fixed using the resources on Ubuntu 25.04 and Bash Scripting and were used to successfully install eSim on Ubuntu 25.04
##The changes made in the nghdl scripts cannot be described here as the forked repo does not contains those scripts in this repo and is used as external dependencies

