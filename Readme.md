# Creating custom Windows Mobile update packages

This guide will detail how to create your own update packages for Windows 10 Mobile, the packages are test signed.

## Requirements:
- Unlocked Bootloader for deploying the Update Package (This is needed to bypass issue with the test certificate being used)
- W10M_Tools.7z I prepared [here](https://github.com/Empyreal96/Updating-WP-FFUs-Guide/raw/main/W10M_Tools.7z) from the [Updating WP FFU Images](https://github.com/Empyreal96/Updating-WP-FFUs-Guide) guide
- XML definition file *(This will be explained)*

## Guide
*I will presume you have extracted the W10M Tools into a folder of your choice, I will choose `C:\WPCT\Tools\bin\i386` for this guide.*

### Creating your XML definition manifest
This file is essential as without it you cannot make your packages, it lists Files and Registry entries that you choose, and adds them to the package. There are other possibilities on what to add but for now I will keep it simple (I am still learning how to add the other options)

- Open a Text Editor of you choice, I recommend Notepad++ for this due to it's colour syntax highlighting
- Add the text below into a new file:
```
<?xml version="1.0" encoding="utf-8"?> 
<Package xmlns="urn:Microsoft.WindowsPhone/PackageSchema.v8.00" 
   Owner="" 
   OwnerType="OEM" 
   ReleaseType="Production" 
   Platform="Generic" 
   Component="" 
   SubComponent=""> 
   <Components> 
      <OSComponent> 
      
      </OSComponent> 
   </Components> 
</Package> 
```
This is the basic structure of the file, you are **required** to fill out the `Owner=""`, `Component=""` and `SubComponent=""` tags. Think of it as `Owner.Component.SubComponent.cab` so for this guide it will be `Empyreal96.MainOS.Tutorial_Package.cab` but you can experiment and call it whatever you want. 

There are other Header options you can add, i.e `Partition=""` is the partition you are targeting the package for so `EFIESP`, `MAINOS` or `DATA`

- Now time to add a file! 
We place objects we want to include between the `<OSComponent> </OSComponent>` tags, again there are other options but for now we will keep it simple.

```
   <OSComponent>  
     <Files> 
      <File Source="C:\Apps\WPDevPortal_1.0.18.0_x86_x64_arm.appxbundle"  
            DestinationDir="$(runtime.CommonFiles)\Xaps"/>
      <File Source="Path to file"  
            DestinationDir="Destination on device"/> 
     </Files> 
   <OSComponent>
```
As you can see it's pretty straightforard: we add the `<Files>  </Files>` tags, and files you wish to add go between them with a `<File />` section, **Source** is where the file is stored on your PC, **DestinationDir** is the desired location on the Phone. You can add multiple files between the `<Files>` tag.

You will notice `$(runtime.CommonFiles)` is used here, this may seem confusing but its a **Macro**, valid Macros can be found in the `C:\WPCT\Tools\bin\i386\pkggen.cfg.xml` file of W10M Tools. The `runtime.CommonFiles` part just tells the Phone to use the `C:\PROGRAMS\CommonFiles\` folder. We need to use these **Macros** when setting the `DestinationDir`

- Now onto Registry entries!
```
...
   </Files> 
   <RegKeys>
    <RegKey KeyName="$(hklm.software)\Microsoft\Windows\CurrentVersion\AppModelUnlock">
     <RegValue Name="AllowAllTrustedApps" Value="00000001" Type="REG_DWORD"/>
     <RegValue Name="AllowDevelopmentWithoutDevLicense" Value="00000001" Type="REG_DWORD"/>
    </RegKey>
    <RegKey KeyName="Path To Registry Key">
     <RegValue Name="Name of value" Value="Data you want to add" Type="Type of value"/>
    </RegKey>
   </RegKeys>
 </OSComponents>
```
Registry entries are a bit more work than Files, You need to know the exact `Key`, `Name`, `Type` and `Value` of the entry you want to add, here I have used the example of enabling Developer Mode to install unsigned apps. We have to add the `<RegKeys> </RegKeys>` tags to imply that we will be adding registry entries, and each Registry entry will be added by using `<RegVaue />` tags between them. You can add a variety of different `Type` values (e.g REG_SZ, REG_BINARY) but to keep it simple i have gone with `REG_DWORD`.

Again we have a **Macro** used (Refer to pkggen.cfg.xml for more), `$(hklm.software)` simply tells us that the registry key is in `HKEY_LOCAL_MACHINE\SOFTWARE` hive, the rest of the **KeyName** is the rest of the location. 


- Now that we have covered that, lets look at it all in a single XML file:
```
<?xml version="1.0" encoding="utf-8"?> 
<Package xmlns="urn:Microsoft.WindowsPhone/PackageSchema.v8.00" 
   Owner="Empyreal96" 
   OwnerType="OEM" 
   ReleaseType="Production" 
   Platform="Generic" 
   Component="MainOS" 
   SubComponent="Tutorial_Package"> 
   <Components> 
      <OSComponent> 
         <RegKeys>
          <RegKey KeyName="$(hklm.software)\Microsoft\Windows\CurrentVersion\AppModelUnlock">
           <RegValue Name="AllowAllTrustedApps" Value="00000001" Type="REG_DWORD"/>
          </RegKey>
          <RegKey KeyName="$(hklm.software)\Microsoft\Windows\CurrentVersion\AppModelUnlock">
           <RegValue Name="AllowDevelopmentWithoutDevLicense" Value="00000001" Type="REG_DWORD"/>
          </RegKey>
         </RegKeys>
         <Files> 
          <File Source="C:\Apps\WPDevPortal_1.0.18.0_x86_x64_arm.appxbundle"  
                DestinationDir="$(runtime.CommonFiles)\Xaps"/>
          <File Source="Path to file"  
                DestinationDir="Destination on device"/> 
         </Files> 
      </OSComponent> 
   </Components> 
</Package> 
```

That is now complete, Save the XML file to somewhere safe, I name my manifests like this: `Owner.Component.SubComponent.pkg.xml` so this will be saved as `Empyreal96.MainOS.Tutorial_Package.pkg.xml` in `C:\WPCT\Tools\bin\i386\XML`

### Creating the Package
Now we have our definition file, we need to create the package, but this has some setup required beforehand.

- Open Command Prompt as Admin to `C:\WPCT\Tools\bin\i386`
- (First time only) Run the following commands
  - `set "WPDKCONTENTROOT=C:\WPCT"`
  - `installoemcerts.cmd`
*This is required first time to install the Test Certificates that will be used for signing*
- Now we need to set the date to 2014 and configure some signing settings, type:
  - `date 01-01-2014` *(note: Windows sometimes resets the time back after running this command, if it does run it again)*
  - `set SIGN_OEM=1`
  - `set SIGN_WITH_TIMESTAMP=0`
- It's time to create the package by typing:
  - `PkgGen.exe "C:\WPCT\Tools\bin\i386\XML\Empyreal96.MainOS.Tutorial_Package.pkg.xml" /version:1.0.0.0 /config:pkggen.cfg.xml /output:C:\WPCT\Tools\bin\i386\spkg_out`
  
  *(You can choose anywhere to save the output file, I just create the folder and use `C:\WPCT\Tools\bin\i386\spkg_out` as my personal preference)*
- If all goes well then your newly created package will be in `C:\WPCT\Tools\bin\i386\spkg_out`, this will create both an `.spkg` and a `.cab` file for you to use, `.spkg` is best for WP8.x devices and the `.cab` is best for W10M devices


### Pushing the package to device
*As stated in the [requirements](#requirements) we need an unlocked bootloader, otherwise you will get a `CERT_E_CHAINING` error when deploying*

- Set your phone date anywhere between 01-01-2014 and 2016.
- Copy/Move your new `.cab` file to a folder, make sure it is the *only* file in the folder.
- Use IUTool to push the new package.
  - `iutool -V -p "Path to folder"`
- The device should restart and apply the update, once booted if you don't get "Update installed successfully" notification, or your changes aren't visible/taken effect then follow the instructions below.

### Diagnosing why an update isn't applying
- Use this command to fetch the logging cab and open it
  - `getdulogs -o logs.cab && logs.cab`
- When a window opens, look for `ImgUpd.log.cbs.log` and look for any errors in the file (File Collisions, Registry Collisions etc)


### Extras

- [Creating a package that provisions an App install](https://github.com/Empyreal96/creating-windows-phone-packages/blob/main/Adding%20an%20application%20to%20package.md)
