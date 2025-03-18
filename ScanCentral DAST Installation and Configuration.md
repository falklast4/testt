##
#### SSL Certificate in .PFX format with Private Key
1. Obtain a Certificate with private key from Certificate Portal. **`.CER`**
2. Install the certificate in Personal Store
3. After installation Export the Certificate in **`.PFX`** format with export private key.

#### Download the MCR and Docker Images for Fortify DAST 23.1
1. Download the script from `https://get.mirantis.com/install.ps1`
2. Allow downloaded script files to run in the current session in `powershell`
``` 
	PS> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Force -Scope Process;
```
3. Run the installation script with the **`-DownloadOnly`** parameter. As per the parameter name, this action only downloads the zip file (no installation is performed).
```
	PS> .\<installation-script> -DownloadOnly
```
4.  Copy the installation script and the installation zip file over to the air-gapped machine (PR Server) and run the install with the **`-Offline`** parameter.
```
	PS> .\<installation-script> -Offline
```
5. Start the Docker Service `Docker Engine` from `services.msc`
6. A restart is required for changes to take place.
   
```
	PS> Restart-Computer
```
7. After restart Download the following images using `docker pull`  login Creds `elfskinnedjoyce:27febrauary@1997`
```
	PS> docker login
```

```
	PS> docker pull fortifydocker/lim:23.1
	PS> docker pull fortifydocker/scancentral-dast-api:23.1
	PS> docker pull fortifydocker/scancentral-dast-globalservice:23.1
	PS> docker pull fortifydocker/webinspect:23.1
   
```

#### Installation of LIM on Docker
1. Create a new folder at **`C:\LIM`** & copy the **`.PFX`** certificate in the folder
2. Open notepad and copy the content from below to notepad.
```
#!-- LIM Docker env file. --!
LimUseSSL=true
LimAdminWebSiteName=LIM.Admin
LimServiceSiteName=LIM.Service
LimApiSiteName=LIM.API
LimDirectory=c:\LIM
certpath=c:\LIM\limcert.pfx
certpassword=changeit
LimAdminUsername=limuser
LimAdminPassword=limAdmin1901
LimAdminEmail=limadmin@limadmin.com
LimAdminFriendlyName=LimAdmin!
LimLogsDirectory=c:\LIM\logs\
```

3. Run Powershell as Administrator and run the following command to Configure and Start LIM Container.
```
docker run -v C:/LIM:c:/lim -d -p 9443:443 --restart always --env-file C:\LIM\LimDocker.env --memory=8g --cpus=2 --name lim fortifydocker/lim:23.1
```
4. Make

5. Extract the contents of `scancentral-dast-config.tar` from the `DASTConfigurationTool` File
6.  Locate the file **`fk*`** Folder
7. Extract the `layer.tar` file from the folder
8. Navigate to **`.\app\`**
9. Copy the **`DefaultData.zip`** file to a Different Location (for Database Seeding)
10. Create a folder in **`C:\Config`** 
11. Copy the **`.YAML`** file from the downloaded file to the Config folder
12. Edit the **`YAML`** File with the proper parameter for URL and Credentials
13. Create the configuration file with following command
```
	PS> DAST.ConfigurationTool.exe configureEnvironment -mode autodeploy -settingFile <config.yaml> -outputDirectory C:\Config\
```
17. The Database Seeding Process will start. **`(Will take around 40 mins)`**
18. Open the output Folder and Extract the content of **`scancentral-dast-windows.zip`**
19. Run the **`start-containers.ps1` script as Administrator in powershell.
20. Wait till all the containers are up and running 
```
PS> docker ps
```
17. Login Fortify SSC with Administrator Credentials 
18. Navigate to **`ADMINSITRATION > Configuration > Scancentral DAST`**
19. Enable the ScanCentral DAST check box
20. Enter the DAST Controller URL and click **`SAVE`**
21. Navigate to **`SCANCENTRAL > DAST`** to Load the SCANCENTRAL DAST Controller

## Installation of WebInspect 23.1

1. Double-click the .exe or .msi file to start the Setup Wizard.
    
    The Welcome to the Micro Focus WebInspect Setup Wizard window appears.
    
    ![01_Welcome_NoVer](images/01_Welcome_NoVer.png)
    
2. Click **Next.**
    
    The End‑User License Agreement window appears.
      
    ![02_EULA_NoVer](images/02_EULA_NoVer.png)
      
3. Review the license agreement. If you accept it, select the check box and click **Next**; otherwise click **Cancel**.
    
    If you accept the license agreement, the Destination Folder window appears.
    
    ![03_DestFolder_NoVer](images/03_DestFolder_NoVer.png)
    
4. In the Destination Folder window, do one of the following:
    
    - If you are installing Fortify WebInspect as a sensor for Micro Focus Fortify WebInspect Enterprise, do _not_ make any changes to the default Destination Folder. Click **Next**.
                
        `C:\Program Files\Fortify\Fortify WebInspect`
        
    - Otherwise, you can accept the default Destination Folder or choose a different folder into which you want to install the software. Click **Next**.
        
        The Sensor Configuration window appears.
        
        ![04_Sensor_NoVer](images/04_Sensor_NoVer.png)
        
5. To install Fortify WebInspect as a sensor:
    
    1. In the **Configure WebInspect as a Sensor for this installation (optional)** area, select **Configure WebInspect as a Sensor**.
        
    2. Enter the **Enterprise Manager URL**, that is, the URL of Fortify WebInspect Enterprise manager.
        
    3. In the Sensor Authentication group, enter the Windows account credentials for this sensor.
        
    
    For important information about installing Fortify WebInspect as a sensor and configuring it to work with Fortify WebInspect Enterprise, see the _Micro Focus Fortify WebInspect Enterprise Installation and Implementation Guide_.
    
6. Click **Next**.
    
    The Ready to install Micro Focus WebInspect window appears.
    ![05_Ready_NoVer](images/05_Ready_NoVer.png)
    
    
    
7. Click **Install**.
    
    When the installation process is complete, the Completed the Micro Focus WebInspect Setup Wizard window appears.
    ![07_Complete_NoVer](images/07_Complete_NoVer.png)
    
8. Select **Launch Fortify WebInspect** and click **Finish**.
    
    The Setup Wizard closes and Fortify WebInspect launches.
    
9. Activation of Webinspect with Fortify LIM
	1. Enter the LIM URL `https://LIM_URL:9443/LIM.API`
	2. Enter the License Pool Credentials for DAST 
	3. Click on Activate



## Configuration of DAST Scanner Service
1. Download the **`ScannerService.zip`** file from the Fortify ScanCentral DAST software download package.
2. Install **`.NET Framework 7 SDK`** from 
3. Create a folder in **`C:\ScannerService`**
4. Extract the **`ScannerService.zip`** file to any directory, such as the following: **`C:\ScannerService`**
5. Place the **`appsettings.json`** file that the ScanCentral DAST Configuration Tool created in the same directory, replacing the existing file.
6. Run **`cmd`** as Administrator and run the following command.
```
> sc create ScannerWorkerService binpath= "C:\ScannerService .\DAST.ScannerWorkerService.exe" start= auto depend= "WebInspect API" displayname= "WebInspect DAST Scanner Worker Service"
```
6. The **`ScannerWorkerService`** is created and automatically starts each time the computer is restarted.