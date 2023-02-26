# Adding an application to a package

This guide will go though how to setup and create a package that will install an application to the device, these are provisioned apps so will survive after resetting the device.

# Requirements
- [Windows Configuration Designer](https://apps.microsoft.com/store/detail/windows-configuration-designer/9NBLGGH4TX22?hl=en-us&gl=us&rtc=1) (can also be found in the Windows ADK as "Windows Imaging and Configuration Designer")
- The Appx/AppxBundle/Xap you wish to include

# Create a Provisioning Package

- First open the Configuration Designer app
- Click on the **Provision Windows mobile devices** tile
- Give the new project a name of your choice and click **Finish**
- Click **Switch to advanced editor** and click **Yes**
- On the left of the window, open the **Runtime Settings** option and scroll down until you see **UniversalAppInstall** and open that option.
- Click **UserContextApp** and you will see in the middle of the window has changed, in the box next to **PackageFamilyName** type the **Package Family Name** of your app (for example I will use "WinUniversalTool_eyr0bca9nc39y")
- Click **Add**, You will see some red bold text on the left where **UserContextApp** is, double click that and you will see your added Package in the list, open that so you can see **Application File** in red.
- Click **Application File** and then **Browse** in the middle of the window, choose the Appx/Appxbundle/Xap you are going to use.
- Once you have selected a file, you should include any dependencies it requires, back on the left look for **DependencyAppxFiles** and click it.
- Same as before, click **Browse** and select the dependency you need, you can select multiple files at once. when that is done you are now ready to package
- At the top of the window, click **Export** > **Provisioning Package**, a new window will open.
- Choose a name for the file (Version you will only need to change if you make revisions to the package in the future), then click **Next**, then **Next** again as we *don't* need to sign the package, Then **Next** one final time.
- Now you can review the Name, Type and Output of the package, If all is good click **Build**
- When that completes, keep a note of where you have saved the `.ppkg` file as you will need this next.

# Creating the Update Package
*Here I will presume you have read the guide on making a package already*

- Create a new XML file similar to below, I am using [WUT](https://github.com/basharast/wut) in this example.
```
<?xml version="1.0" encoding="utf-8"?> 
<Package xmlns="urn:Microsoft.WindowsPhone/PackageSchema.v8.00" 
   Owner="Empyreal96" 
   OwnerType="OEM" 
   ReleaseType="Production" 
   Platform="Generic" 
   Component="MainOS" 
   SubComponent="WUT_Prov"> 
   <Components> 
      <OSComponent> 
         <Files>
          <File Source="C:\Users\Empyreal96\Desktop\Empyreal's Cabs\App Provisioning\WUT.Provisioning.ppkg"
	       DestinationDir="$(runtime.windows)\provisioning\packages" />
        </Files>
      </OSComponent> 
   </Components> 
</Package> 
```

- Make sure to have the `Source` pointing to where your created ppkg file is, keep `DestinationDir` as `$(runtime.windows)\provisioning\packages`/
- Now build your package with `PkgGen` and push to device just as you did in the [Creating Packages](https://github.com/Empyreal96/creating-windows-phone-packages) guide.
