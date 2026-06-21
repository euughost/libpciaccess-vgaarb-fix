libpciaccess-vgaarb-fix
 
Fix null pointer segmentation fault of libpciaccess 0.17-3+b3 for Debian Trixie.
Specially optimized for Intel G31 + ICH7 old motherboards.
 
Problem Background
 
On Debian 13 (trixie) with Intel G31 integrated graphics + ICH7 southbridge,
running screen-off command will crash Xorg.
Root cause: All vgaarb_* functions in common_vgaarb.c access global pci_sys pointer without null check, trigger SIGSEGV when pointer is empty.
 
Patch Principle
 
Add safety null pointer judgment at the entry of every VGA arbitration function:
if (!pci_sys)
return -1;
No modification to original PCI read/write and GPU arbitration logic, only add defensive check.
 
Repository Structure
 
libpciaccess-vgaarb-fix/
├── src/libpciaccess-0.17          Official Debian source with manual null-pointer patch
├── patch/vgaarb-null-fix.patch    Minimal diff patch for source code
├── prebuilt-deb/                  Precompiled amd64 deb packages
│   ├── libpciaccess0_0.17-3+b3_amd64.deb
│   ├── libpciaccess-dev_0.17-3+b3_amd64.deb
│   └── libpciaccess0-dbgsym_0.17-3+b3_amd64.deb
├── build.sh                       One-click auto compile script
└── README.md                      Project documentation
 
Release Binary Download
 
Precompiled installation packages are stored in GitHub Releases (v0.1).
Open Releases page to download the archive file.
 
Quick Usage 1: Install prebuilt deb (Recommended)
 
Installation steps
sudo dpkg -i *.deb
sudo apt -f install -y
sudo ldconfig
 
Lock version to avoid overwritten by official broken package
 
sudo apt-mark hold libpciaccess0 libpciaccess-dev
 
Verify crash fix
 
Test screen-off function to confirm the patch
 
DISPLAY=:0 xset dpms force off
 
- Normal screen shutdown without Xorg crash = Patch works correctly
- Session freeze/disconnect = Fix not applied successfully
 
Quick Usage 2: Rebuild from source
 
chmod +x build.sh
./build.sh
 
Compiled deb files will output to ./output directory
 
sudo dpkg -i output/*.deb
 
Unlock Version Lock (Restore Official Original Package)
 
sudo apt-mark unhold libpciaccess0 libpciaccess-dev
sudo apt full-upgrade
 
Compatibility Note
 
1. Only fit Debian trixie libpciaccess version 0.17-3+b3;
2. Patch only validated on Intel G31 + ICH7 hardware platform;
3. Not guaranteed to apply directly to other Linux distributions or libpciaccess versions.
