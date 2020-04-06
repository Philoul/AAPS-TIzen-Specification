# SDK installation

[TOC]

## Tizen Studio install (only for native version of App and Wear)

[Prerequisites (Java Dev Kit version is very important)](https://developer.tizen.org/development/tizen-studio/download/prerequisites)

[Download Tizen Studio](https://developer.tizen.org/development/tizen-studio/download)

[Preparing the SAP Server Test Environment](https://developer.samsung.com/galaxy-watch-develop/creating-your-first-app/native-companion/galaxy-watch-emulator.html)

[Installing Certificate Extension](https://developer.samsung.com/galaxy-watch-develop/getting-certificates/install.html)

**[Creating Certificates](https://developer.samsung.com/galaxy-watch-develop/getting-certificates/create.html)**

- Don't forget to add the DUID of your watch in distributor certificate. before creating your certificate, connect your watch to device manager.
- Don't add emulator's DUID to your distributor certificate (I spent one week to understand that it was the cause of Error-12 while uploading project to watch!)

**[Extending Certificate Expiry](https://developer.samsung.com/galaxy-watch-develop/getting-certificates/extend.html)**

Certificate duration is only one year...

**[Connect your watch to computer](https://developer.samsung.com/galaxy-watch-design/studio/faq.html#Why_cant_I_connect_to_my_device)** 

After having enable debugger mode

- Turn Off bluetooth
- set display alway active
- enable wifi
- power off your watch near your PC (wifi access point)
- Turn your watch on

don't forget to reboot your watch after setting debugger mode

wait 30-60 s after reboot keeping your eyes on it to enable RSA key

**[Managing Projects](https://developer.tizen.org/development/tizen-studio/native-tools)**



## .net (Visual Studio and Xamarin plugin)

- install Visual Studio community **VS2017** (or VS2019 ?)
  - See [.NET Application Overview](https://docs.tizen.org/application/dotnet/index)
- Install and configure  [xamarin extension](https://developer.tizen.org/development/visual-studio-tools-tizen/installing-visual-studio-tools-tizen) in Visual Studio
- Check you have installed the latest version of Visual Studio Tools for Tizen (3.1.0.0 or above)

**[Update Nuget Package to latest version](https://developer.samsung.com/galaxy-watch-develop/creating-your-first-app/net-companion/use-sap.html):**

1. in AndroidAPS_CompanionXAML Right Clic in dependencies thens elect "Manage Nuget Packages..."
2. Click on Cog wheel (settings) in the top right of the screen (near Package source)
3. Click on green "+" button to add another source for packages
   - Give a name (i.e. nuget2) and enter https://tizen.myget.org/F/dotnet/api/v3/index.json in source field and clic on "Update", then OK to close option screen.
4. Select in Package source "nuget2"
   - Update  Tizen.NET.API6 to latest available version (v6.0.0.15094) 
     - this update to Tizen.NET.API6.6.0.0.15094 and Tizen.NET.API5.5.0.0.14639...
5. Update Nuget package on 

## native (Tizen Studio)

Tizen Studio and native  (c++) is an old environment. I think it's easier to install but not sure it's the best for this kind of project...

- You can not package an UI app with watch app. Go through the **Table: Combinations** in [**this link**](https://developer.tizen.org/ko/development/training/native-application/application-development-process?langredirect=1) to know about possible packaging combinations in detail.
  - UI App can include Service and Widget sub-project
  - Watchface can only include Service sub-project
- To develop multi-packaged project in Tizen, you may follow **Developing Multiple Projects as a Combined Package** section in **[this link](https://developer.tizen.org/development/training/native-application/application-development-process#develop)**.
- To do communication between android app and Tizen app please go through **Build your First Companion type application** section in **[this link](https://developer.samsung.com/galaxy-watch/develop/creating-your-first-app)**.
  - [Setup SDK](https://developer.samsung.com/galaxy-watch-develop/creating-your-first-app/native-companion/setup-sdk.html)
  - [Download SDK for detailled documentation](https://developer.samsung.com/galaxy-accessory/download.html)
- To debug your application, you may try various application debugging methods e.g. debugging with logs as described in **[this link](https://developer.tizen.org/ko/development/training/native-application/application-development-process/debugging-applications?langredirect=1#methods)**.
- You may try creating another app to enter Carbs quantity, Insulin Quantity, etc data and launch it from watchface app. In that case, you may follow **[this link](https://developer.tizen.org/development/guides/native-application/application-management/application-controls)**. 
  - this describe how we can launch (through **Application Control**) UI commands from service (when AAPS request acknowledge from watch) or from a watchface (when we want to launch a command)

