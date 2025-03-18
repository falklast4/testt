
## Installation of Scancentral Controller
#### Installation of Java
1. Download Java JDK 11 for Linux (.tar)
2. Extract file to /opt folder
	``` 
	# tar xzvf jdk-11.0.xx_linux-x64_bin.tar.gz -C /opt/jdk-11/ 
	# nano /etc/profile
	```
3. Add `$JAVA_HOME` to `$PATH`
   ```
	export JAVA_HOME=/opt/jdk-11
	export PATH=$JAVA_HOME/bin:$PATH
	```
4.  Source the profile file
   ```
	# source /etc/profile
```

#### Installation of Scancentral Controller
5.  Extract the contents of the `Fortify_ScanCentral_Controller__x64.zip`
   ```
   # unzip Fortify_ScanCentral_Controller__x64.zip -d /opt/
   # chown tomcat:tomcat -R /opt/tomcat/
```
6. Configure the Controller service by creating a systemd unit file `tomcat.service` in the `/etc/systemd/system` directory with the following content.
   ```
   [Unit]
   Description=ScanCentral SAST Controller Service
   After=syslogtarget network.target
   
   [Service]
   Type=forking
   
   Environment=JAVA_HOME=/opt/jdk-11
   Environemnt=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
   Environment=CATALINA_HOME=/opt/tomcat
   Environment=CATALINA_BASE=/opt/tomcat
   
   ExecStart=/opt/tomcat/bin/startup.sh
   ExecStop=/bin/kill -15 $MAINPID
   
   [Install]
   WantedBy=multi-user.target
      ```
7.  Reload the daemon to discover and load the new service file
   ```
   systemctl daemon-reload
```
8. Enable the service to start on startup by running the following command
   ```
   systemctl enable tomcat.service
	```
#### Configuration of Scancentral Controller
1. Configure the values below parameters in `/opt/tomcat/webapps/scancentral-ctrl/WEB-INF/classes/config.properties`
2. Open `config.properties` in a text editor.
   ```
	client_auth_token=***CLIENT
	...
	ssc_scancentral_ctrl_secret=***SCANCENTRAL
	...
	ssc_url=https://ssc-url/ssc
	...
	this_url=https://sc-controller/scancentral-ctrl
	...
	worker_auth_token=***WORKER
	```
3. Save and close `config.properties` file
4. Restart the `tomcat.service`
```
	# systemctl restart tomcat.service
```
#### Integrating Controller with Fortify SSC
1. Login in to Fortify SSC with Administrator Privileges.
2. Navigate to `Administration > Configuration > Scancentral SAST`
3. Tick the `Enable ScanCentral SAST` check box
4. In the `ScanCentral Controller URL`  box type the URL for the Controller.
5. In the `SSC and ScanCentral Controller shared secret` box type the exact `ssc_scancentral_ctrl_secret` which was set `config.properties`
6. Restart the tomcat service for `SSC Server` 
```
	# systemctl restart tomcat9.0.xx.service
```
7. When you next login Fortify SSC, the header will be included with `SCANCENTRAL` tab.
####  Importing Certificates in trusted keystore.
1. Download the certificates from the browser for Fortify SSC and Fortify Scancentral SAST Controller in **`.CRT`** format.
2. Move the certificates to the SSC Server and Controller Server at mentioned path **`/etc/pki/ca-trust/source/anchors`**
3. Run `update-ca-trust enable` 
4. Run `update-ca-trust extract`
5. Open the `.CRT` file with `cat SSC.CRT`  & `cat SSCONTROLLER.CRT`
6. Copy the Contents of each certificate and append it to the mentioned ca-bundle.crt `etc/pki/tls/certs/ca-bundle.crt` 
```
# mkdir /opt/BackUP
# cp /etc/pki/tls/certs/ca-bundle.crt /opt/BackUP/
# cat "# SSC Certificate" >> /etc/pki/tls/certs/ca-bundle.crt
# cat "SSC.CRT" >> /etc/pki/tls/certs/ca-bundle.crt
# cat "# Controller Certificate" >> /etc/pki/tls/certs/ca-bundle.crt
# cat "SSCONTROLLER.CRT" >> /etc/pki/tls/certs/ca-bundle.crt
```


   ![1](images/1.png)
### Installation of Fortify SCA 23.1
### Windows & RHEL
## Installing Fortify Static Code Analyzer

To install Fortify Static Code Analyzer:

1. Run the installer file for your operating system to start the Fortify Static Code Analyzer Setup Wizard:    
    - Windows: `Fortify_SCA__<version>_``_windows_x64.exe`
    - Linux: `Fortify_SCA__<version>_``_linux_x64.run`
2. Review and accept the license agreement, and then click **Next**.
3. Select the components to install, and then click **Next**.
4. If the installer detects that the system does not include the minimum software required to analyze some types of projects, a System Requirements page displays any missing requirements and which projects require them.
5. Choose where to install Fortify Static Code Analyzer, and then click **Next**.
6. Specify the path to the `fortify.license` file, and then click **Next**.
7. On the LIM License page, select **No** to use the Fortify License and Infrastructure Manager (LIM) for managing your concurrent licenses, and then click **Next**.
8. Specify the settings required to update your Fortify security content. click **Next** 
9. Specify if you want to migrate from a previous installation of Fortify Static Code Analyzer on your system. (Migrating from a previous Fortify Static Code Analyzer installation preserves Fortify Static Code Analyzer artifact files. To skip migration of artifacts from a previous release, leave the **Static Code Analyzer Migration** selection set to **No**, and then click **Next**.)
10. Click **Next** on the Ready to Install page to install Fortify Static Code Analyzer, any selected components, and Fortify security content.
11. Click **Finish** to close the Fortify Static Code Analyzer Setup Wizard.
12. Import the Rulepacks with the latest rulepack **`(.ZIP)`** file.
```
fortifyupdate -import Rulepack.zip
```
13. After Installation Import the necessary certificates into the SCA keystore using the below command
	- Fortify SSC Certificate
	- Fortify Scancentral SAST Controller Certificate
```
To add a trusted certificate to the Fortify Static Code Analyzer keystore:
<sca_install_dir>_/jre/bin/keytool -importcert -alias _<alias_name>_ -cacerts -file _<cert_file>

Enter the keystore password.
    
    **Note:** The default password is `changeit`.
    
 When prompted to trust this certificate, select **yes**.
```


## Installing a ScanCentral SAST Sensor as a Service


To install the sensor as a Windows service:

1. Navigate to the `` `_<sca_install_dir>_\bin\scancentral-worker-service` `` directory, and then do one of the following:
    
    - To use a clear text password, run `setupworkerservice.bat <_sca_version_> <_full_controller_url_> <_shared_secret_>`
                
        **Examples:**  
        For a worker_auth_token that contains a single caret, such as this^that, run the following command:  
        `setupworkerservice.bat 23.1 http://url.com **WORKER
          
                       
2. Start the service, as follows:
    
    `net start FortifyScanCentralWorkerService`
    
3. After verification, change the startup type setting to **Automatic (Delayed Start)** in Windows Services.
    

The services installer creates the `<_sca_install_dir_>\Core\config\worker.properties` file for you.



## Installing Fortify Static Code Analyzer Applications and Tools

To install Fortify Static Code Analyzer applications and tools:

1. Run the installer file for your operating system to start the Fortify Applications and Tools Setup Wizard:
    
    - Windows: `Fortify_Apps_and_Tools__<version>__windows_x64.exe`
    - Linux: `Fortify_Apps_and_Tools__<version>__linux_x64.run`             
     
2. Click **Next**.
    
3. Review and accept the license agreement, and then click **Next**.
4. Choose where to install Fortify Applications and Tools, and then click **Next**.
    ```
    Important! Do not install Fortify Applications and Tools in the same directory where Fortify Static Code Analyzer is installed.
    ```
5. Select the components to install, and then click **Next**. (Plugins According to the IDEs)    
6. Specify the path to the `fortify.license` file, and then click **Next**.
7. Specify if you want to migrate from a previous installation of Fortify Applications and Tools on your system.
       
    To migrate artifacts from a previous installation:
    
    1. In the Applications and Tools Migration page, select **Yes**, and then click **Next**.
    2. Specify the location of the existing Fortify Applications and Tools installation on your system, and then click **Next**.
    
    To skip migration of artifacts from a previous release, leave the Applications and Tools Migration selection set to **No**, and then click **Next**.
    
8. If you are installing the Fortify Extension for Visual Studio, do the following:
    
    1. Specify whether to install the extensions for the current install user or for all users.
        
        The default is to install the extensions for only the current install user.
        
    2. Click **Next**.
        
9. Click **Next** on the Ready to Install page to install Fortify Applications and Tools.
10. Click **Finish** to close the Fortify Applications and Tools Setup Wizard.

## To install the Fortify Analysis Plugin:

**Note** : Use restrictedfortify.license for Plugin related installations.

1. Run the Fortify Static Code Analyzer Applications and Tools installation and select the **IntelliJ IDEA Analysis** component from the list of plugins.
    
2. Start IntelliJ IDEA
3. Open the Settings dialog box as follows:
    
    - On Windows or Linux, select **File > Settings**.
        
4. On the left, select **Plugins**.
5. Select **Install Plugin from Disk**, browse to the `_<tools_install_dir>_/plugins/IntelliJAnalysis` directory, and then select `Fortify_IntelliJ_Analysis_Plugin__<version>_.zip`.
    
6. Click **OK**.
7. To activate the plugin, restart the IDE.

The menu bar now includes the Fortify menu.


## Configuring Fortify ScanCentral SAST Options


To configure the Fortify ScanCentral SAST options:

1. Select **Fortify > Analysis Settings**.
2. Select the **ScanCentral SAST Configuration** tab.
3. Select **Enable ScanCentral SAST upload**.    
    ![[images/ScanCentralConfig.png]]
    
4. To specify how to connect to Fortify ScanCentral SAST, do one of the following:
    
    - Select **Use Controller URL**, and then in the **Controller URL** box, type the URL for the ScanCentral SAST Controller.
        
        Example: `https://_<controller_host>_:_<port>_/scancentral-ctrl`
        
        **Tip:** Click **Test Connection** to confirm that the URL is valid, and the Controller is accessible.
        
    - Select **Get Controller URL from SSC**, and then in the **Token** box, paste the decoded token value for an authentication token of type ToolsConnectToken.
                 
        **Tip:** Click **Test Connection** to confirm that the URL and token is valid, and the server is accessible.
        
5. To upload the analysis results to Fortify Software Security Center, do the following:
    
    1. Select the **Send scan results to SSC** check box.
        
    2. In the **Token** box, paste the decoded token value for an authentication token of type ToolsConnectToken.
                
6. Under **Default translation type**, specify where to run the translation phase of the analysis by selecting one of the following:
            
    - **Remote**—Run the entire analysis with Fortify ScanCentral SAST.
            
7. (Optional) To specify Micro Focus Fortify Static Code Analyzer command-line options for the translation or scan phase:
    
    1. Select the **Advanced Options** tab.
        
    2. Select the **Use additional SCA options** check box and type Fortify Static Code Analyzer command‑line options for the translation or scan phase.

8. Click **OK** to save the configuration.

### Scanning Projects with Fortify ScanCentral SAST

To scan a project with Fortify ScanCentral SAST:

1. Select **Fortify > Analyze Project with ScanCentral**.
    
2. If prompted, select the application version where you want to upload the analysis results, and then click **OK**.
    ![[images/SelectAppVersion.png]]     

To view the analysis results, you can either:

- Copy the provided job token and use it in the Fortify ScanCentral SAST command-line interface to check the status and retrieve the analysis results