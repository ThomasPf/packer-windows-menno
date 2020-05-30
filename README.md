### Packer windows 10

TO-DO:

	-	Authenticate windows version against authentication servers.
	-	Write script to give the ability of specifying chocopacks to install when initializing. script.bat <jsonfile> <chocopack list>. 
	-	Look at Mech for .box files.
	-	If more issues come up with the PinTo10.exe try to understand what happens to it.

#### json files 
The json files are the basis from which packer does its magic.

windows_10.json: The original file from Stefan Scherer, contains:

	-	Builders for qemu, hyperv, vmware and virtualbox. 
	-	Long list of provisioners:
		-  vm-guest-tools.bat
		-  enable-rdp.bat
		-  debloat-windows.ps1
		-  set-powerplan.ps1
		-  docker/disable-windows-defender.ps1
		-  pin-powershell.bat
		-  compile-dotnet-assemblies.bat
		-  set-winrm-automatic.bat
		-  uac-enable.bat
		-  dis-updates.bat
		-  compact.bat
		-  chocolatey.bat
		-  chocopacks.bat 
	-	Has a vagrant post-processor to create a .box file.
		For vmware to use .box files you need to buy some license. (https://github.com/mechboxes/mech seems to be a workaround, haven't tested it yet).

For my adaptation I removed the non vmware builders and the post-processor to just get a .vmx file.

List of Json files:

	-	windows_10.json: base files with all options
	-	windows_10_vmware_all_pp.json: Vmware only and use all original post processors
	-	windows_10_vmware_choco.json: Vmware only, only post processors for chocolatey


#### Autounattend.xml
I re-use the Autonunattend.xml file which has the following options:

	-	Line 89-100: Option to put in a valid ProductKey.
	-	Line 256-283: Options to (not) install windows updates (turned this off for faster running).

#### Scripts folder
Contains scripts to be ran during build or post processor. The only one 1 adjusted is chocopacks.bat, this installs Visual studio 2019 community, 7zip (also installed by vm-guest-tools.bat I think, maybe remove), microsoft-office-deployment, rsat.

In this file you can specify which packages you want chocolatey to install.

#### floppy folder
files used to set initial installation options. The PinTo10.exe file is a bit weird and sometimes gets hidden or locked by windows. Restarting computer solves it. Packer build doesn't seem to mind if the file is hidden and still builds fine.

#### packer_cache folder
Created by the packer build script, contains the iso from which to build the vm

#### output-vmware-iso
Location of the final vm

#### Vmware provider for .box file (special license needed)
install plugin, add box, init box, up box with provider	

	vagrant plugin install vagrant-vmware-desktop
	vagrant box add windows_10 windows_10.box
	vagrant init windows_10
	vagrant up --provider vmware_desktop