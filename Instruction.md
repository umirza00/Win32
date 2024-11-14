# Zscaler Client Deployment via Microsoft Intune

## Overview
This guide provides a comprehensive approach for deploying the Zscaler Client via Microsoft Intune. The deployment process includes preparation, configuration, and steps for automated installation and uninstallation of the Zscaler client.

## Prerequisites
1. **Windows SDK**  
   Download the Windows SDK from [Microsoft’s website](https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/).
   
2. **Windows SDK Installation**  
   Install Windows SDK on a Windows device with the necessary modules selected. After installation, navigate to `C:\Program Files (x86)\Windows Kits\10\bin\10.0.26100.0\x86` and install "Orca-x86_en-us."

3. **Orca Tool Configuration**  
   Launch Orca, open the Zscaler `.msi` file, and add the following properties under the `Property` table:
   
   - Property: `CLOUDNAME`
   - Value(example): `zscaler`
   #
   - Property: `UNINSTALLPASSWORD`
   - Value(example): `examplepass123`
   #
   - Property: `USERDOMAIN`
   - Value(example): `test.com`
   #
**Generate Transform**: In the **Transform** menu, click on **Generate Transform** to save the changes as a transform (`.mst`) file.
## Directory Structure and File Content
1. **Folder Structure**
   - **Zscaler**
     - **input** (folder)
       - install.bat
       - install.ps1
       - uninstall.bat
       - uninstall.ps1
       - Zscaler-windows-3.8.0.100-installer-x64.msi
       - zscaler-client-connector-msi.mst
     - **Output** (folder)
       - install.intunewin
     - **Detection** (folder)
       - ZscalerDetection.ps1

2. **Script Files**
   - **Install.bat**
     ```batch
     @echo off
     powershell.exe -NoProfile -NonInteractive -WindowStyle hidden -executionpolicy bypass -file .\install.ps1
     ```
   - **Install.ps1**
     ```powershell
     msiexec /i "Zscaler-windows-3.8.0.100-installer-x64.msi" /quiet TRANSFORMS="zscaler-client-connector-msi.mst"
     ```
   - **Uninstall.bat**
     ```batch
     @echo off
     powershell.exe -NoProfile -NonInteractive -WindowStyle hidden -executionpolicy bypass -file .\uninstall.ps1
     ```
   - **Uninstall.ps1**
     ```powershell
     msiexec /x "Zscaler-windows-3.8.0.100-installer-x64.msi" /quiet TRANSFORMS="zscaler-client-connector-msi.mst"
     ```
   - **Detection Script (ZscalerDetection.ps1)**
     ```powershell
     $Detect=$(Get-WmiObject Win32_Product | where { $_.name -match "zscaler" } | Select -ExpandProperty IdentifyingNumber)
     if($Detect.count -gt 0){
         Write-Output "Succeeded..."
     }else{
         Write-Error "Installation Failed!"
     }
     ```

## Preparing the Package with Intune Win32 Content Prep Tool
1. **Download**  
   Get the [Win32 Content Prep Tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool/archive/refs/heads/master.zip).

2. **Extract and Run**  
   Extract the tool, navigate to the folder, and run `IntuneWinAppUtil`. Follow the prompts:
   - Specify the path to `Zscaler > Input` folder.
   - Enter `install.bat` as the setup file.
   - Specify the path to `Zscaler > Output` folder.
   - Answer "N" for the catalog folder prompt.

## Deploying the Application in Intune
1. **Intune Console**  
   Access Intune at [Microsoft Intune](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/AppsWindowsMenu/~/windowsApps).

2. **Application Configuration**
   - Click **Add**, select **Windows app (Win32)**, and upload the `.intunewin` package.
   - Configure **App Information**, **Program**, **Requirements**, and upload the **Detection Script**.
   - Complete setup by defining dependencies, supersedence, and group assignment. Finally, create the deployment.

3. **Monitoring Deployment**  
   After setup, allow some time for package upload and monitor the deployment status.
