
# cloudctl - Installing IBM Cloud Private CLI

## Lab objectives
<br>

Install the IBM® Cloud Private command line interface (CLI) cloudctl.  This CLI is used to create, access, and manage ICP clusters.

<b>NOTE:</b> The following instructions utilize the 'curl' command. If curl is not installed on your laptop it must first be installed to complete this lab. 

Scroll to the appropriate operating system and follow the installation instructions. <br>

----
<img align="left" width="63" height="81" src="mac_logo.png">&nbsp;
### &nbsp;&nbsp;macOS 

<br>
1 - Download the macOS CLI to install:

Replace <ICP IP> with the IP address of the IBM Cloud Private instance from where the CLI will be obtained.

	curl -kLo cloudctl-darwin-amd64-3.1.1-973 https://<ICP IP>:8443/api/cli/cloudctl-darwin-amd64
	
Example file name from the above curl command:

<p align="center">
<img src="cloudctl_file_macos.png">
</p>

2 - Rename the above downloaded file to cloudctl.  

	mv cloudctl-darwin-amd64-3.1.1-973 cloudctl
	

3 - Make the kubectl binary executable.

	chmod +x ./kubectl


4 -  Move the binary in to your PATH.

	sudo mv ./kubectl /usr/local/bin/kubectl


<br><br>

----

<img align="left" width="63" height="81" src="linux.png">&nbsp;
### &nbsp;&nbsp;Linux 

<br>
1 - Download the Linux CLI to install:

Replace <ICP IP> with the IP address of the IBM Cloud Private instance from where the CLI will be obtained.

	curl -kLo cloudctl-linux-amd64-3.1.1-973 https://<ICP IP>:8443/api/cli/cloudctl-linux-amd64
	
Example file name from the above curl command:

<p align="center">
<img src="cloudctl_file_linux.png">
</p>


2 - Rename the above downloaded file to kubectl.  

	mv cloudctl-linux-amd64-3.1.1-973 cloudctl
	

3 - Make the kubectl binary executable.

	chmod +x ./kubectl


4 - Move the binary in to your PATH.


	sudo mv ./kubectl /usr/local/bin/kubectl


<br><br>

----

<img align="left" width="81" height="63" src="windows10_logo.png">&nbsp;
### Windows 

<br>

1 - Download the Windows CLI to install.

Replace <ICP IP> with the IP address of the IBM Cloud Private instance from where the CLI will be obtained.

	curl -kLo cloudctl-win-amd64-3.1.1-973.exe https://<ICP IP>:8443/api/cli/cloudctl-win-amd64.exe
	
Example file name from the above curl command:

<p align="center">
<img src="cloudctl_file_windows.png">
</p>

2 - Run the above file to install the CLI on the Windows system.

3 - Add the binary in to your PATH.

---

<br><br>

## Verify the CLI is installed

From a terminal or command prompt enter:

	cloudctl version
	

Example output:

<p align="center">
<img src="cloudctl_verify.png">
</p>

----

