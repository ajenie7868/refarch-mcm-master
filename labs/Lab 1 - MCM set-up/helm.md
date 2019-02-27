
# helm - Installing HELM CLI

## Lab objectives
<br>

Install helm (CLI).  This CLI is used to create, access, and manage Helm packages.

<b>NOTE:</b> The following instructions utilize the 'curl' command. If curl is not installed on your laptop it must first be installed to complete this lab. 

Scroll to the appropriate operating system and follow the installation instructions. <br>

----

<img align="left" width="63" height="81" src="mac_logo.png">&nbsp;
### &nbsp;&nbsp;macOS 

<br>
1 - Download the macOS CLI to install:

	curl -kLo helm-darwin-amd64-v2.9.1.tar.gz https://master1.ibmserviceengage.com:8443/api/cli/helm-darwin-amd64.tar.gz

2 - Untar the file.  

	tar -xzvf helm-darwin-amd64.tar.gz
	
3 - Export the Helm path:

	export PATH=/root/darwin-amd64:$PATH


<br><br>

----

<img align="left" width="63" height="81" src="linux.png">&nbsp;
### &nbsp;&nbsp;Linux 

<br>
1 - Download the Linux CLI to install:

	curl -kLo helm-linux-amd64-v2.9.1.tar.gz https://master1.ibmserviceengage.com:8443/api/cli/helm-linux-amd64.tar.gz
	
2 - Untar the file.  

	tar -xzvf helm-linux-amd64.tar.gz
	
3 - Export the Helm path:

	export PATH=/root/linux-amd64:$PATH

<br><br>

----

<img align="left" width="81" height="63" src="windows10_logo.png">&nbsp;
### Windows 

<br>

1 - Download the Windows CLI to install. Open a browser and paste the URL below

https://master1.ibmserviceengage.com:8443/api/cli/helm-win-amd64.tar.gz

2 - Once the download is complete, use an unzip tool to extract the files. 

3 - Add the binary in to your PATH.
   
   export PATH=%PATH%:<yourdirectory where you extracted>

---

<br><br>

## Verify the CLI is installed

From a terminal or command prompt enter:

	helm version
	
----
