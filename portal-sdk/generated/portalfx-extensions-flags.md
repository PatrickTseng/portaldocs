<a name="portal-query-string-flags"></a>
# Portal Query String Flags


<a name="portal-query-string-flags-extension-flags-and-feature-flags"></a>
## Extension flags and Feature Flags
    
There are three types of query string flags that are used with extensions to modify run-time behavior. The purpose of each type of flag is similar but not identical. The Azure Portal makes use of flag values by making modifications to the server or browser, to the shell, and to the extension at runtime. Query string flags can be differentiated by the naming convention that is used to invoke them.
* Some feature flags have their own names, like  `https://portal.azure.com/?<featureName>=<value>`
* Most feature flags are invoked with the  syntax: `https://portal.azure.com/?feature.<featureName>=true`
* Otherwise, flags are directives to the extension, in which case the syntax is `<extensionName>_<extensionFlag>=<value>`

**NOTE**: Features that are invoked through `extensiondefinition` are outside of the scope of this document. For more information about using `extensiondefinition`, see `Microsoft.Portal.Framework.ExtensionDefinition` class.

The following table specifies the types of query string flags that are used with the Azure Portal.

| Flag               | Purpose | Document | 
| ------------------ | ------- | -------- |
| Trace mode         | Temporarily set server characteristics, toggle a behavior, or enable event logging. For the most part, trace mode does   not require changes to extension code. The exception is certain types of logging, as specified in [portalfx-logging-from-typescript-and-dotnet.md](portalfx-logging-from-typescript-and-dotnet.md). <br> Invoked with  `https://portal.azure.com/?trace=<settingName>`.   | [portalfx-extensions-flags-trace.md](portalfx-extensions-flags-trace.md) |
| Extension Flags | Allow developers to specify features that they maintain. <br>Invoked with `https://portal.azure.com/?<extensionName>_<extensionFlag>=<value>`.   |  [portalfx-extensions-flags-extension.md](portalfx-extensions-flags-extension.md)  |
| Shell flags        | Connect the developer's extension to features that are maintained by the Azure Portal team. Shell features do not require changes to the code in the developer's extension.<br> Invoked with  `https://portal.azure.com/?feature.<featureName>=<value>`.   |  [portalfx-extensions-flags-shell.md](portalfx-extensions-flags-shell.md) |
  
<!-- The following sentence is from portalfx-domain-based-configuration-pattern.md. -->
  Changing the default feature flags that are sent to the extension requires Shell configuration changes and redeployment.


<a name="portal-query-string-flags-trace-mode-flags"></a>
## Trace Mode Flags
   
Trace mode flags are associated with code that exists inside the Portal. They can be invoked with the syntax: `https://portal.azure.com/?trace=<settingName>`, or they can be configured externally through the `.config` file.  For example, [https://portal.azure.com/?trace=diagnostics](https://portal.azure.com/?trace=diagnostics) will enable verbose debugging information in the console. The trace mode allows the developer to enable, disable, and filter tracking output.

The information that is displayed in trace mode is associated with debugging, moreso than with regular operation of the extension. This information is an addition to standard console errors, and it can be used to monitor application execution and performance in a deployed environment.  The errors that are presented in the console assist in fixing extension issues. For more information about trace modes, see [https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/how-to-create-initialize-and-configure-trace-switches](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/how-to-create-initialize-and-configure-trace-switches).

Often the errors that are displayed in the console can lead you down the path of fixing your issues. In addition to viewing the standard errors displayed in the console, it's often useful to load the Portal in trace mode.

<a name="portal-query-string-flags-the-clientoptimizations-flag"></a>
## The clientoptimizations flag

The **clientoptimizations** flag behaves somewhat like a trace mode flag, in that it is neither a Shell feature flag nor a flag for an extension.  Instead, it turns off bundling and minification of JavaScript to make debugging easier.   A value of `true` turns on bundling and minification,  and a value of `false` turns them both off. A value of `bundle` turns off JavaScript minification but retains bundling so the Portal still loads fairly quickly.  The value `bundle` is the suggested value for Portal extension debugging.
<!--TODO: Both bundle and false are the recommended values for debugging.  Determine which one  is the better one.  -->

To set the optimizations mode for the portal and all extensions:

    https://portal.azure.com/?clientoptimizations=<value>

To set the optimization mode for a specific extension only:

    https://portal.azure.com/?<YourExtension>_clientoptimizations=<value>

**Recommendation**:  When debugging an extension, the developer should supply `false` for this flag to disable script minification and to turn on additional diagnostics.

**NOTE**:  This applies to both the Portal and extensions source. If testing extensions that are already deployed to production, use the **clientOptimizations** flag instead of the **IsDevelopmentMode** appSetting. If working in a development environment instead, use the **IsDevelopmentMode** appSetting instead of the **clientOptimizations** flag to turn off bundling and minification for this extension only. This will speed up Portal load during development and testing.  To change the **IsDevelopmentMode** appSetting, locate the appropriate `web.config` file and change the value of the **IsDevelopmentMode** appSetting to `true`. 

 <!--TODO:  Verify whether this section contains all trace modes for the Azure Portal. -->

<a name="portal-query-string-flags-trace-mode-flag-list"></a>
## Trace mode flag list
Trace modes are enabled by appending them to the query string, as in the following example: `https://portal.azure.com/?trace=<settingName>`, where ```settingName```, without angle brackets, is the type of trace to run on the extension. The Azure trace mode flags are in the following list.

**desktop**: Log all shell desktop operations. Useful for reporting errors to the alias.

**diagnostics**: Display the debug hub, and add verbose tracing. Also used to enable callstack capturing for all communication that occurs between iframes.

**inputsset.debug.viewModelOrPdlName**: Break into debugger when `onInputsSet` is about to be called on extension side. This trace can use the `viewmodel` name or the blade or part name to filter trace.

**inputsset.log.CommandViewModel**:  Add refinement to the selection of which view model to trace.

<!--TODO:  Validate how this works if onInputSet is being replaced. -->
**inputsset.log.viewModelOrPdlName**: Log in console when `onInputsSet` is about to be called on extension side. This trace can use the `viewmodel` name or the blade or part name to filter trace.

**inputsset.log.WebsiteBlade**: Further refine which view model to trace.

**inputsset.verbose.viewModelOrPdlName**: Log in the console every time the shell evaluates whether `onInputsSet` should be called. This trace can use the viewmodel name or the part or blade name to filter trace. 

**partsettings.viewModelOrPartName**: Shows writes and reads of settings for the specified part. This trace can use the `viewmodel` name or the part name to filter the trace. 

**usersettings**:  Show all writes and reads to the user settings service. Great for debugging issues that require a 'reset desktop'. 




<a name="portal-query-string-flags-extension-flags"></a>
## Extension Flags

Extension flags are specially-formatted query string parameters that are sent through the Portal to extensions and their controller methods.  They are often used while testing to enable and disable features that are maintained in the source code. Flags can only be used on items like form elements or HTML template components; they cannot be used to hide blades, parts, or commands. 

Flags are only accessible by the extension in which they are defined, and therefore are not shared across extensions. Typically, the flag is boolean and has a descriptive name. Most flags are set to a value of `true` or `false`, which respectively enables or disables the feature. However, some  flags send non-boolean values to the extension when more than two options are appropriate to test a specific feature.

Extension features are enabled by appending a flag to the query string, as in the following example: `https://portal.azure.com/?<extensionName>_<flagName>=<value>`, where ```flagName```, without angle brackets, is the feature to enable for the extension. The extension name and the underscore are used by the Portal to determine the extension for which the flag applies.

The only limitation on developer-designed flag names is that they cannot contain underscores. Flags are named according to the following rules.
* Are all lower case
* Must be formatted as `<extensionName>_<flagName>` (e.g. `azurecompute_someflag`)
* Can contain any non-empty value (e.g. `azurecompute_someflag=true`)
* Cannot include an underscore in the flag portion of the name (i.e., `azure_compute_some_flag` does not work, but `azure_compute_someflag` works)
 
<a name="portal-query-string-flags-extension-flags-feature-flag-api-contract"></a>
### Feature flag API contract

Developers can create flags for extensions, and plan to manage them as a part of the software maintenance process.  Typically, the flag is boolean and has a descriptive name. A value of `true` turns on the feature, and a value of `false` turns it off. 

The following code examples demonstrate how to turn extension  flags on and off inside the code. 

<details>
<summary>Reading flags in TypeScript </summary>

* Detecting whether a flag is set

    Use the ```MsPortalFx.isFeatureEnabled``` and  ```MsPortalFx.getFeatureValue``` APIs to access feature values in TypeScript, as in the following code.

    Query string with parameters: `https://portal.azure.com?azure_compute_someflag=true`

   ```ts
    if (MsPortalFx.isFeatureEnabled("someflag"))
    {
        // turn on new feature
    }
   ```

*  Detecting the value of a flag

    Query string with parameters: `https://portal.azure.com?azure_compute_someotherflag=value1`

   ```ts
    switch (MsPortalFx.getFeatureValue("someotherflag"))
    {
        case "value1":
            // behavior 1
            break;
        case "value2":
            // behavior 2
            break;
        default:
            // default behavior
            break;
    }
   ```

</details>
<details>
<summary>Programming default values for flags in C#</summary>

Flags can be enabled for all users in one or more deployments by using an extension configuration, as in the following code. 

1. In the `ApplicationConfiguration` class, add the following property.

     ```cs
    [ConfigurationSetting]
    public CaseInsensitiveReadOnlyDictionary<IReadOnlyDictionary<string, string>> DefaultQueryString
    {
        get;
        private set;
    }
    ```

1. In the derived app context class (probably named `CustomApplicationContext`),  override the `GetDefaultQueryString` method.

     ```cs
    public override IReadOnlyDictionary<string, string> GetDefaultQueryString(string host)
    {
        return this.configuration.DefaultQueryString.GetValueOrDefault(host);
    }
    ```

1. Finally, in the config files (`web.config` or `cscfg`), add the following entry
  ```json
<Setting name="Microsoft.StbPortal.Website.Configuration.ApplicationConfiguration.DefaultQueryString" value="{
  '*': {
      'websitesextension_supportsettingsenabled':'true',
      'websitesextension_troubleshootsettingsenabled':'true'
  },
  'prod.websitesextension.com': {
      'websitesextension_requestsettingsenabled':'true'
  },
  'dogfood.websitesextension.com': {
        'websitesextension_healthsettingsenabled':'true'
  }
}" />
 ```

* The asterisk specifies the default case. The flag will be set to the specified value for all requests to the extension, regardless of the domain.

* Flags that are associated with the domain name in an environment, i.e. the domain name of the incoming extension requests, will take precedence over the flags that are in the default case.

</details>
<details>
<summary>Reading flags in the context of an AJAX call in C#</summary> 

*  Using the RequestFlags dictionary

    When using `MsPortalFx.Base.Net.ajax()`, extension flags are also sent to the controller methods. The controller should extend the  `Microsoft.Portal.Framework.ApiControllerBase` method to receive the flag value. The extension can use the `RequestFlags` dictionary to check whether a flag is being sent to the controller, as in the following code.

   ```cs
    if (RequestFlags.TryGetValue("microsoft_azure_compute_someflag", out value) && value == "true")
    {
        // turn on feature here
    }
   ```

<!-- TODO:  determine whether browsecuration affects the Browse menu, the More Services menu, or both -->

</details>

<br>

For more information about extension flags, see [https://docs.microsoft.com/en-us/vsts/articles/phase-features-with-feature-flags](https://docs.microsoft.com/en-us/vsts/articles/phase-features-with-feature-flags).

You can ask questions on Stackoverflow with the tag [ibiza](https://stackoverflow.microsoft.com/questions/tagged/ibiza).



<a name="portal-query-string-flags-shell-feature-flags"></a>
## Shell feature flags

The Ibiza Fx team supports the following Shell feature flags. These flags are only available to the Shell, unless they are configured to be shared. Unless otherwise noted, a value of `true` enables the feature, and a value of `false` disables it. 

<!--TODO: can the canmodifystamps flag ever be required or included when it is false?  -->
<!--TODO: is the canmodifystamps always required when extensionName is used? -->

Shell feature flags are invoked with the syntax: `https://portal.azure.com/?feature.<featureName>=true`.

The keyboard shortcut CTRL+ALT+D toggles the visibility of the debug tool, as specified in [portalfx-extensions-debugging-overview.md](portalfx-extensions-debugging-overview.md). The yellow sticky that is located at the bottom on the right side of the window can be used to toggle trace mode flags and shell feature flags.

<a name="portal-query-string-flags-shell-feature-flags-the-extensionname-flag"></a>
### The extensionName flag

The name of the extension can be used as a feature flag. The extension name can contain any character in the ranges between [a-z] or [0-9]. 
<!--TODO: Determine whether all uses of the extensionName flag require the 'canmodifystamps` flag. -->
 This flag is used to enable or disable an extension, use different configuration stamps, and provide other run-time functionality.  A value of `true` will temporarily enable a disabled extension, and allows the use of other flags. A value of `false` will temporarily disable the extension and leave it in hidden mode. The syntax for the extensionName flag is `https://portal.azure.com?Microsoft_Azure_DevTestLab=true`. It requires the `canmodifystamps` flag to contain a value of `true` in order to be in effect.  For more information, see [portalfx-extensions-configuration-overview.md](portalfx-extensions-configuration-overview.md).

The name of the extension can be used in the query string to access various Shell flags. These are flags that are independent of the **canmodifystamps** flag.
`   &<extensionName>=true,[<otherShellFlags>]`.  How to use the flags that are within the extension is specified in [portalfx-extensions-feature-flags-developer.md](portalfx-extensions-feature-flags-developer.md);  The Shell flags that require `&<extensionName>=true` are in the following table.
    
  <!--TODO:  Validate that the parameters are used correctly.  -->

  | Directive | Use | 
  | --------- | --- |
  | nocdn | A value of `true` will bypass the CDN when loading resources for the Portal only. A value of `force` will bypass the CDN when loading resources for the Portal and all extensions.|
| testExtensions | Contains the name of the extension, and the environment in which the extension is located. For example, `?testExtensions={"HelloWorld":"https://localhost:44300/"}` specifies the intent to load the `HelloWorld` extension from the localhost port 44300 into the current session of the Portal. The **name** and **uri**  parameters are as specified in [portalfx-extensions-configuration-overview.md#understanding-the-extension-configuration-in-portal](portalfx-extensions-configuration-overview.md#understanding-the-extension-configuration-in-portal).|

    
<a name="portal-query-string-flags-shell-feature-flags-the-canmodifystamps-flag"></a>
### The canmodifystamps flag

The **canmodifystamps** flag is used in conjunction with the **extensionName** parameter to pass developer-specified values to the extension, to specify which stage or build number to use, and other purposes. When the **canmodifystamps** flag is set to true, the following run options can be enabled.

* Create custom deployment environments, as specified in [portalfx-deployment.md](portalfx-deployment.md).

* Use a secondary test stamp, as specified in [portalfx-extensions-configuration-overview.md#extension-stamps](portalfx-extensions-configuration-overview.md#extension-stamps). If the developer is using a secondary test stamp, enter one of the following into the query string immediately after the **canmodifystamps** flag.
 ```js
   &<extensionName>=<stageName>
   &<extensionName>=<buildNumber>
   &<extensionName>=<uriFormatPrefix> 
 ```
  * **extensionName**: The name of the extension.

    * **stageName**:  The stage that represents a datacenter, as specified in [portalfx-extensions-hosting-service-scenarios.md#sideloading](portalfx-extensions-hosting-service-scenarios.md#sideloading). Use the name that is deployed to a specific stage, for example, stage1.
    * **buildNumber**:  The build number as specified in [portalfx-extensions-hosting-service-scenarios.md#sideloading](portalfx-extensions-hosting-service-scenarios.md#sideloading). Replace the dots in the build number with the letter 'd', so that build number 1.0.8.31 is represented by 1d0d8d31.
    * **uriFormatPrefix**: The value to use when building the **uriFormat** string. For example, when **uriFormat** is `//{0}.devtest.ext.azure.com`, the query string `https://portal.azure.com?feature.canmodifystamps=true&Microsoft_Azure_DevTestLab=perf` would cause  the `{0}` in the **uriFormat** string to be replaced with `perf` and attempt to load the extension from `https://perf.devtest.ext.azure.com`.
    
<a name="portal-query-string-flags-shell-feature-flags-shell-flags"></a>
### Shell flags

The following are the feature flags that are invoked with the syntax: `feature.<featureName>=true` unless otherwise noted.

**feature.abovethefold**:  A value of `true` means that the item is above the fold, or is initially displayed on the Web page without scrolling.  A value of `false` means that the item is not above the fold.

**feature.autoenablemove**:  Shows or hides the **Change** (resource move) link in the **Essentials** menu for all resource types.  A value of `true` shows the links, and a value of `false` hides them.  The links are the edit buttons next to the resource group, and the subscription name in the **Essentials** menu.

**feature.browsecategories**: Enables categories in the **Browse** menu.

<!--TODO: Validate what the default value is. -->
**feature.browsecuration={fileName}**:  Switches the curation file used for the **More Services** menu, default favorites, and search results.  The default is `available`.  To submit a partner request for custom curation to support a specific scenario, see [https:\\aka.ms\portalfx\uservoice](https:\\aka.ms\portalfx\uservoice).

**feature.browsecurationflags={a,b}**:  Comma-delimited list, in brackets, of additional flags used while rendering browse curation.  The list is in the following table.

| Flag        | Purpose |
| ----------- | --- | 
| (available: appservice, vms)  | |
| analytics   | Merge **Intelligence** and **Analytics** categories |
| aws         | Model **Compute** category after AWS  |
| cdn         | Remove the **Media/CDN** category |
| databases   | Rename the **Data** category to **Databases** |
| integration | Add all integration services to the **Enterprise Integration** category |
| management  | Rename **Management + Security** to **Monitoring + Management** |
| oms         | Group all OMS services together in the **Management** category |
| security    | Rename **Identity + Access Management** to **Security + Identity** |
| sql         | Add SQL Data Warehouses and SQL Server Stretch DBs kinds |
| storage     | Add all storage services to the **Storage** category  
| vm          | Rename **Compute** to **VMs**, and rename **Web + Mobile** to **App Services** |

**feature.canmodifyextensions**:  Required to support loading untrusted extensions for security purposes. For more information, see [portalfx-extensions-testing-in-production-overview.md#loading-customized-extensions](portalfx-extensions-testing-in-production-overview.md#loading-customized-extensions).

<!--TODO: Determine whether the following flag is associated with msportalfx-test.md#msportalfx-test-running-ci-->

**feature.ci**:  Sets up  the Portal up to work better with tests by disabling NPS popups and other items.

**feature.consoletelemetry**:  Logs most telemetry events to the browser console.

**feature.customportal**:  Overrides the `ms.portal` redirect when signing in to portal.azure.com. A value of `false` turns off automatic redirection for users in the microsoft.com tenant, and a value of `true`   .

**feature.disableextensions**:  Disables all extensions. A value of `true` disables the extension, and a value of `false` enables it. **NOTE**: Extensions must be enabled explicitly with this flag, including Hubs. For example, the following command disables all extensions, enables hubs, and enables the specific extension to be tested. `?feature.DisableExtensions=true&HubsExtension=true&MyOtherExtension=true `

**feature.eagerlyrevealassetpart**:  Asset part will infer resource names from ARM resource IDs, and will auto-reveal its content as soon as it has an icon and name.

**feature.extloadtimeout**:  Extension loading timeout (in seconds???).  Can contain any characters in the ranges [0-9]. The maximum value is .

**feature.fakepreviewassets**: Reserved for team use. <!-- `=true  | Forces Clocks and Hosts (from samples - see Client\V1\Blades\DynamicBlade\ViewModels) to be in preview (for testing only). A value of `true` forces the preview mode,  and a value of `false`       it. -->

<!--TODO: Does the following feature go away with relex? -->
**feature.fastmanifest**:  Enables relex-based manifest caching.   A value of `true` enables  the caching, and a value of `false`  disables it.

**feature.feedback**:  Disables the feedback pane. A value of `true` enables the feedback pane, and a value of `false`  disables it.

**feature.forceiris**:  Forces the Portal to query IRIS even if the time limit  has not expired.  The  time limit is 24 hours.

**feature.fullwidth**:  Forces all resource menu blades to be opened maximized.

**feature.hideavatartenant**: Reserved for internal Azure Portal use. <!-- Hides the directory name under the user's name in the avatar menu. -->

**feature.inmemorysettings**:  Uses in-memory user settings.

**feature.internalonly**:  Shows or hides the **Preview** banner in the top-left.  A value of `true` shows the banner,  and a value of `false` hides it.  

**feature.irissurfacename**:  Defines the surface name to use when querying IRIS.

**feature.multicloud**: Enables multicloud mode. Reserved for team use.  

**feature.nodirectory**:  Opens the avatar menu after the user signs in.

**feature.pinnedextensions**:  Comma-delimited list, in brackets, of extensions that are never unloaded.  Pins the specified extensions so that they exist during the lifetime of the Portal.

**feature.pov2**:  Use ProxiedObservables version 2 (POv2), as specified in .

**feature.pov2compatibilitymode**:  Disable compatibility mode for POv2, when enabled.  A value of `true` enables compatibility mode, and a value of `false`  disables it, so that the extension can run faster. 

**feature.prefetch**: Enables or disables a script prefetch feature in the Portal for diagnostics purposes. Reserved for team use. 

**feature.rapidtoasts**: Enables or disables toast notifications, and sets the default time that a toast is displayed, in milliseconds.  A value of zero disables toasts. 

<!--TODO:  verify whether these relex settings have ceased to exist or are just not used -->
**feature.relex**:  Not used.  Comma-delimited list of extensions to load in Relex. **NOTE**: An asterisk ("*") will load all extensions in Relex.

**feature.relexsinglecomm=true**:  Not used. When run in conjunction with the `feature.relex` flag, runs the corresponding extensions in relex with a single web socket connection to relex for all extensions.

<!--TODO: Determine whether ?feature.resourcemenu=true belongs here.-->

**feature.seetemplate**: "See template" link in Create blades. A value of `true`         ,  and a value of `false`      . 

**feature.settingsprofile**: Loads a Portal with the specified, isolated set of settings. It is used by the CI infrastructure to enable the loading of multiple Portals in parallel without overwriting each other’s settings. Reserved for team use.

**feature.ShowAffinityGroups**:  Set to allow users to add and select affinity groups in the Portal.  A value of `true`         ,  and a value of `false`      .   

**feature.showauditlogsaseventlogs**:  Changes audit log strings.

**feature.showbugreportlink**:  Shows or hides the **Report bug** link in the top bar. A value of `true` shows the link,  and a value of `false` hides it.

**feature.showpreviewtags**: Hides the preview ribbon on the startboard and shows labels in blade headers when extensions are marked as in preview.   **NOTE**: Does not hide the preview ribbon.

<!--TODO:  verify whether these relex settings have ceased to exist or are just not used -->
**feature.showrelexdialog**:  Not used. Hides the relex dialog that is displayed if performance is slow.

**feature.testhubsurl**:  Reserved for team use.

**feature.testremoteurl**: Reserved for team use.

**feature.testsamplesurl**: Reserved for team use.

**feature.tilemgmt**: Reserved for future use.

**feature.UserType**: A value of "test" excludes test traffic from Azure reports.

**feature.verbosediagnostics**: Valid values are in the following table.

| Value | Meaning |
| ----- | ------- |
| all   | Includes default diagnostics and all filtered diagnostics in the Ctrl+Alt+A popup window | 
| permissions |  Includes default diagnostics and the permissions entry in the Ctrl+Alt+A popup window | 
| assets | Includes default diagnostics and the assets/assetManagement entries in the Ctrl+Alt+A popup window | 
| desktop | Includes default diagnostics and the desktop entry in the Ctrl+Alt+A popup window | 

**feature.waitforpendingchanges**: Reserved for future use.

**feature.webworker**: Loads the extension in a separate webworker thread, which makes the extension more performant.  The flag is useful for testing extensions previous to enabling them in production. In a webworker thread, objects such as [DOM](portalfx-extensions-flags-glossary.md), cookies, and other items are not available. The query string parameter works in conjunction with the **supportsWebWorkers** parameter in the `extensions.json` file. The  `extensions.json` file must contain the value  `supportsWebWorkers: "true"`, in order for the feature flag to invoke behavior.  When the `supportsWebWorkers` parameter is absent or set to false, the feature flag cannot invoke any behavior. The query string is as follows: `https://portal.azure.com?extName=<webWorkerId>,<extensionName1>=true,<extensionName2>=true, <extensionName3>=true,feature.webworker=<value>`, where

  * **webWorkerId**: Identifies the webworker thread.

  * **extensionName\<number>**: Required field. Matches the name of the extension, without the angle brackets, as specified in the `<Extension>` element in the  `extension.pdl` file.  There is no default that will turn on all extensions in an environment on the basis of the `supportsWebWorkers: "true"` parameter in the json file.
  
  * **feature.webworker**: A value of `true` will allow the extensions to run in the specified webworker. A value of `false` will not run the extensions in a webworker.  In the previous query string, the **feature.webworker** flag allows the webworker whose id is specified in `webWorkerId` to use the extensions named `<extensionName1>`, `<extensionName2>,` and `<extensionName3>`.

    For more information about webworkers, see [https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers).

<a name="portal-query-string-flags-shell-feature-flags-marketplace-feature-flags"></a>
### Marketplace feature flags

<!--TODO:  Determine whether microsoft_azure_marketplace is an extension name or an example extension name.  If such is the case, then the following 4 flags should be documented as only the suffix name. -->

**microsoft_azure_marketplace_Curation**:  Associated with Marketplace extensions. Uses a named curation  . For more information about curations, see   .

**microsoft_azure_marketplace_curation**: Associated with Marketplace extensions. Uses a named curation . For more information about curations, see   .

<!--TODO:  Determine whether microsoft_azure_marketplace_ItemHideKey is different from microsoft_azure_marketplace_itemhidekey. -->

**microsoft_azure_marketplace_itemhidekey**: Associated with Marketplace extensions. Shows specified Marketplace items that are otherwise hidden, or shows unpublished items in the gallery. A value of `true` shows the item,  and a value of `false` hides it. Used in conjunction with **extensionName** when the extension is enabled. For example, if the hidekey is `DevTestLabHidden`,the following will display the extension in the "Create New" experience while it is enabled.
`https://portal.azure.com?<extensionName>=true&microsoft_azure_marketplace_ItemHideKey=DevTestLabHidden`.

**microsoft_azure_marketplace_quotaIdOverride**:  Associated with Marketplace extensions and the Create Launcher. A value of `true` ,  and a value of `false`. 


<a name="portal-query-string-flags-glossary"></a>
## Glossary

This section contains a glossary of terms and acronyms that are used in this document. For common computing terms, see [https://techterms.com/](https://techterms.com/). For common acronyms, see [https://www.acronymfinder.com](https://www.acronymfinder.com).

| Term                | Meaning |
| ------------------- | --- |
| API contract | An agreement between two pieces of code as to how to pass parameters between them, and how those parameters are processed. |
| CDN                 | Content delivery network | 
| CI infrastructure   | | 
| configuration stamp | | 
| curation            | The process of categorizing content around a specific topic or area of interest. Curated items often formed into various types of collections and are displayed together according to theme or purpose. | 
| diagnostic switch  | |
| DOM                | Document Object Model |
| feature flag       | A coding technique that allows the enabling or disabling of new code within a solution. It allows the testing and development-level viewing of new features, before they are complete and ready for private preview.  |
| IRIS               | | 
| manifest caching | | 
| marketplace | | 
| MVC                | Model-View-Controller, a methodology of software organization that separates the view from the data storage model in a way that allows the processor or a controller to multitask or switch between applications or orientations without losing data or damaging the view.|
| NPS popups         | Net Promoter Score | 
| ProxiedObservables | | 
| query string       | The part of a uniform resource locator (URL) that contains data. Query strings are generated by form submission, or by being entered into the address bar of the browser after the URL. The  query string is specified by the values following the question mark (?). The values are used in Web processing, along with the path component of the URL. Query strings should not be used to transfer large amounts of data.  | 
| stage | | 
| trace mode         | |
| web worker | A way to for extensions to run scripts in background threads. |