---
title: Integrate the POS with a new hardware device
description: This topic explains how to integrate the point of sale (POS) with a new hardware device.
author: mugunthanm
ms.date: 07/27/2020
ms.topic: article
audience: Developer
ms.reviewer: rhaertle
ms.search.region: Global
ms.author: mumani
ms.search.validFrom: 2019-08-2019
ms.dyn365.ops.version: AX 7.3.0, Retail July 2017 update, AX 10.0.11
---

# Integrate the POS with a new hardware device

[!include [banner](../../includes/banner.md)]

This topic explains how to integrate the point of sale (POS) with a new hardware device. 

To call Hardware station from the POS, you must use a request and a response:

+ **HardwareStationDeviceActionRequest** – The request that is sent from the POS to Hardware station.
+ **HardwareStationDeviceActionResponse** – The response that the POS receives from Hardware station.

The class that you extend depends on the version of the Retail software development kit (SDK) that you're using.

+ For Retail SDK version 10.0.11 or later, you extend the **IController** interface.
+ For Retail SDK versions that are earlier than version 10.0.11, you extend the **HardwareStationController** and **IHardwareStationController** classes.

## HardwareStationDeviceActionRequest

The following code example shows the definition of **HardwareStationDeviceActionRequest**.

```TypeScript
class HardwareStationDeviceActionRequest<TResponse extends HardwareStationDeviceActionResponse> extends Request<TResponse> {
    readonly device: string;
    readonly action: string;
    readonly actionData: any;
    constructor(device: string, action: string, actionData: any, correlationId?: string);
}
```

The following table describes the parameters.

| Parameter  | Data type | Description |
|------------|-----------|-------------|
| device     | String    | The device name that is passed to the Hardware station request should match the **Export** attribute that is added in the Hardware station Device extension controller class. |
| action     | String    | The method that should be called in the Hardware station extension. The method name should be passed as a string value. The core POS Hardware station layer will then call the corresponding method from your Hardware station extension code. The method should exactly match the method name in your Hardware station extension. The Hardware station extension should be passed as a parameter. |
| actionData | any       | A custom parameter for the extension to pass. |

### Sample code

The following code example creates a **HardwareStationDeviceActionRequest** object.

```TypeScript
let hardwareStationDeviceActionRequest: HardwareStationDeviceActionRequest<HardwareStationDeviceActionResponse> =
    new HardwareStationDeviceActionRequest("Export attribute in Hardware station controller class",
        "extension method name in Hardware station", "Custom parameters/you can also pass custom object");
return this.extensionContextRuntime.executeAsync(hardwareStationDeviceActionRequest);
```

## HardwareStationDeviceActionResponse

The following code example shows the definition of **HardwareStationDeviceActionResponse**.

```TypeScript
class HardwareStationDeviceActionResponse extends Response {
    readonly response: any;
    constructor(response: any);
}
```

The following table describes the parameters.

| Parameter  | Data type | Description |
|------------|-----------|-------------|
| response   | any       | The response that is sent from the Hardware station extension code to the POS. |

## End-to-end flow

The follow diagram shows the flow between the POS, Hardware station, and the hardware device.

![Flow diagram.](./media/POSDeviceExtension.png)

## Hardware station extension

To call your new hardware device, you must implement the Hardware station code. You call your hardware device from that code.

To implement the Hardware station extension for Retail SDK version 10.0.11 or later, follow these steps.

1. Create a new C# class library project by using the Microsoft .NET Framework version 4.6.1. Alternatively, use one of the samples in the Retail SDK as a template. (You can find the samples at **...\\RetailSDK\\SampleExtensions\\HardwareStation\\**.) We recommend that you use a sample as a template.
2. In the extension project, use the NuGet package manager to add the **Microsoft.Dynamics.Commerce.Hosting.Contracts** package. You can  find the NuGet packages in the **RetailSDK\\pkgs** folder.
3. Add a new controller class that extends the **IController** interface.
4. Add the **RoutePrefix** attribute to the controller class to expose the controller class to clients.

    ```csharp
    [RoutePrefix("ISVEXTENSIONDEVICE")]
    ```

5. To implement your custom logic to call the hardware device, in the controller class, add a method that has the **HttpPost** attribute. This method will be passed as the second parameter (action parameter) to the POS **HardwareStationDeviceActionRequest**. From the extension method, the extension can call other requests, such as printing and cash drawer requests. Just include the relevant NuGet packages from the Retail SDK.

    ```C#
    [HttpPost]
    public async Task<bool> IsReady(IEndpointContext context)
    {
    }
    ```

6. Build the project.

To implement the Hardware station extension for Retail SDK versions that are earlier than version 10.0.11, follow these steps.

1. Create a new C# class library project.
2. Add a new controller class that extends **HardwareStationController** and **IHardwareStationController**.
3. Add the **Export** attribute to the controller class. The **Export** attribute must be in all uppercase letters, and you must pass the value as a parameter from the POS extension. The device parameter that is passed from the POS **HardwareStationDeviceActionRequest** must match this value.
4. To implement your custom logic to call the hardware device, add your method in the controller class. This method will be passed as the second parameter (action parameter) to the POS **HardwareStationDeviceActionRequest**.
5. Build the project.

To deploy the Hardware station extension in Modern POS and test it by using the local Hardware station, follow these steps.

1. Copy the output library to the **C:\\Program Files (x86)\\Microsoft Dynamics 365\\70\\Retail Modern POS\\ClientBroker\\ext** folder.
2. Open the **HardwareStation.Extension.config** file.
3. In the **composition** section, add the extension library details.

    ```Xml
    <add source="assembly" value="your extension library name" />
    ```
 
4. Save the file.
5. Close Modern POS if it's running.
6. Open Task Manager, and end the **dllhost.exe** task.
7. Open Modern POS, and configure it to use the local Hardware station.
8. Validate your scenario.

To test by using Cloud POS, deploy the dynamics-link library (DLL) of the Hardware station extension to the shared Hardware station **ext** folder. Then update the **HardwareStation.Extension.config** file with the custom library in the shared Hardware station folder.

## Retail SDK samples

The Retail SDK includes some samples that you can use for reference.

+ **POS:** \\RetailSDK\\POS\\Extensions\\FiscalRegisterSample
+ **Hardware station:** \\RetailSDK\\SampleExtensions\\HardwareStation\\Extension.FiscalRegisterSample

## Sample code for Retail SDK version 10.0.11 or later

```csharp
namespace Contoso
{
    namespace Commerce.HardwareStation.ISVExtensionDevice
    {
        using Microsoft.Dynamics.Commerce.Runtime.Hosting.Contracts;
        using System;
        using System.Threading.Tasks;

        /// <summary>;
        /// Sample hardware station extension
        /// </summary>

        [RoutePrefix("ISVEXTENSIONDEVICE")]
        public class ISVExtensionDeviceController : IController
        {
            /// <summary>
            /// Sample.
            /// </summary>

            /// <param name="request">Custom request.<param>
            /// <returns>Result of Custom response.</returns>

            [HttpPost]
            public async Task<CustomResponse> Sample(CustomRequest request, IEndpointContext context)
            {
                CustomResponse response;
                try
                {
                    response = new CustomResponse();
                }
                catch (Exception ex)
                {
                    throw ex;
                }
                return await Task.FromResult(response);
            }
        }
        public class CustomResponse
        {
            public string sampleProp { get; set; }
            public CustomResponse()
            {
                this.sampleProp = "sampleValue";
            }
        }
    }
}
```

## Sample code for Retail SDK versions before version 10.0.11

```csharp
namespace Contoso
{
    namespace Commerce.HardwareStation.ISVExtensionDevice
    {
        using System;
        using System.Composition;
        using System.Web.Http;
        using Microsoft.Dynamics.Commerce.HardwareStation;

        /// <summary>;
        /// Fiscal register peripheral web API controller class.
        /// </summary>

        [Export("ISVEXTENSIONDEVICE", typeof(IHardwareStationController))]
        [Authorize]
        public class ISVExtensionDeviceController : HardwareStationController, IHardwareStationController
        {
            /// <summary>
            /// Sample.
            /// </summary>
            /// <param name="request">Custom request.<param>
            /// <returns>Result of Custom response.</returns>

            [HttpPost]
            public CustomResponse Sample(CustomRequest request)
            {
                ThrowIf.Null(request, "request");
                try
                {
                    return null;
                }
                catch (Exception ex)
                {
                    throw ex;
                }
            }
        }
    }
}
```

## Sample POS code to call the Hardware station extension

From your POS extension, call the Hardware station by using the following pattern.

```TypeScript
let hardwareStationDeviceActionRequest: HardwareStationDeviceActionRequest<HardwareStationDeviceActionResponse> =
    new HardwareStationDeviceActionRequest("ISVEXTENSIONDEVICE",
        "Sample", "Custom parameters or custom object");
return this.extensionContextRuntime.executeAsync(hardwareStationDeviceActionRequest);
```

## Generate an extension installer for the shared Hardware station for application release 10.0.18 or later

Dynamics 365 Commerce 10.0.18 and later supports sealed installers. You can generate an extension installer separately from the base installer, and the installer can be independently installed and serviced. This will work only if you are using the [sealed self-service installers](enhanced-mass-deployment.md).

To generate the sealed extension installer for shared a Hardware station, follow these steps:

1. Download the [sample Hardware station installer project](https://github.com/microsoft/Dynamics365Commerce.InStore/tree/release/9.31/src/HardwareStationSample/HardwareStation.Installer) from GitHub and open the **HardwareStation.Installer.csproj** project in Visual Studio.
2. Add your Hardware station extension project as a project reference to the **HardwareStation.Installer.csproj** installer project. Remove the existing sample project reference from the **HardwareStation.Installer.csproj** project.

    The installer project consumes the **Microsoft.Dynamics.Commerce.Sdk.Installers.HardwareStation** package to generate the extension installer. The sample installer project class extends the **ExtensionPackageInstallerSetup** class to implement the installation steps.

3. To update the installer name, set the **InstallerName** variable in the sample code.
4. The sample installer project can be updated to perform additional logic during installation. For example, you could ping a server. To add logic, use the **HardwareStationExtensionPackageInstallerSetup.cs** file in the **HardwareStation.Installer.csproj**.
5. You can also add pre- and post-steps to the installer by implementing the **IExtensionInstallerStep** interface. To add steps, use the **TestExtensionInstallerPreInstallStep.cs** file and **TestExtensionInstallerPostInstallStep.cs** in the **HardwareStation.Installer.csproj** sample installer project.
6. Build the extension **HardwareStation.Installer.csproj** project to generate the shared Hardware station extension installer. The output of the project is the extension installer. You can find the generated extension installer path in the Visual Studio Output window after build is finished.
7. Deploy the shared Hardware station extension for Point of Sale (POS) and test it in your scenario.

    > [!NOTE]
    > The sealed shared Hardware server installer must be installed before running the extension installer.

8. Run the extension installer generated by using the command prompt. Open the command prompt in admin mode and run the installer with the **install** parameter. To uninstall, run the extension installer with the **uninstall** parameter. For example:

    ```dos
    C:\HardwareStation.Installer\bin\Debug\net461> .\HardwareStation.Installer.exe install
    ```

9. Close POS if it's running.
10. Open POS and configure it to use the shared Hardware station.
11. Validate the extension hardware station scenario.

## Generate an extension installer for the shared Hardware station for Dynamics 365 Commerce 10.0.17 or earlier

To generate an extension installer for the shared Hardware station for Dynamics 365 Commerce 10.0.17 or earlier, follow the steps in [Create deployable packages](retail-sdk/retail-sdk-packaging.md).

[!INCLUDE[footer-include](../../includes/footer-banner.md)]
