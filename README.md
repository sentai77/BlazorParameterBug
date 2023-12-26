# Parameters from Static Server component to WebAssembly component fail when parameter type defined in separate project

## Describe the bug:
When using a Blazor application structured into three projects (WebApp, WebApp.Client, and SharedLibrary), an unexpected behavior occurs. The application includes a static server-side rendered page (`/person-page`) in WebApp, embedding an interactive WebAssembly component (`PersonDetails`) from WebApp.Client. This component is supposed to receive a "Person" model (name and age) as a parameter, defined in a SharedLibrary project.

Navigating to `/person-page`, the interactive component fails to render, throwing a JavaScript exception:
`Error: One or more errors occurred. (The parameter 'Person' with type 'SharedLibrary.Person' in assembly 'SharedLibrary' could not be found.)`

**Note that:**
1. The render mode of the interactive component should be `@rendermode @(new InteractiveWebAssemblyRenderMode(prerender: false))`. If prerendered or set to Interactive Auto, it initially renders successfully but fails when switching to Web Assembly.
2. In launch settings, `launchBrowser` should be set to false. The auto-launched browser does not raise the error for unknown reasons.

## Expected Behavior:
The person object should be serialized, passed to the web assembly component, deserialized correctly, and then rendered.

## Steps To Reproduce:
1. Create a new Blazor Web App project with the following settings:

   ![Blazor Web App Project Settings](https://github.com/dotnet/aspnetcore/assets/26582886/3be0ca36-87e0-462e-bac7-e05020e7ab01)

2. Add a new class library project called SharedLibrary. Reference the SharedLibrary in both web projects. The projects should be:
   - WebApp
   - WebApp.Client
   - SharedLibrary

3. In SharedLibrary, create a [Person](https://github.com/sentai77/BlazorParameterBug/blob/main/SharedLibrary/Person.cs) model class

4. In WebApp.Client, create a [PersonDetails](https://github.com/sentai77/BlazorParameterBug/blob/main/WebApp/WebApp.Client/PersonDetails.razor) Razor component taking a `Person` as a parameter. Render mode should be web assembly with prerendering off.

5. In WebApp, create a static [PersonPage](https://github.com/sentai77/BlazorParameterBug/blob/main/WebApp/WebApp/Components/Pages/PersonPage.razor) component. It contains the interactive component and passes a value to the Person parameter.

6. In [launch settings](https://github.com/sentai77/BlazorParameterBug/blob/main/WebApp/WebApp/Properties/launchSettings.json), set `launchBrowser` to false.
7. Open a new browser and navigate to `/person-page`. Observe the failure of the person details to appear.

## Exceptions (if any):
```
blazor.web.js:1  Error: One or more errors occurred. (The parameter 'Person' with type 'SharedLibrary.Person' in assembly 'SharedLibrary' could not be found.)
    at Jn (marshal-to-js.ts:349:18)
    at Ul (marshal-to-js.ts:306:28)
    at dotnet.native.wasm:0x1faca
    at dotnet.native.wasm:0x1bf8b
    at dotnet.native.wasm:0xf172
    at dotnet.native.wasm:0x1e7e4
    at dotnet.native.wasm:0x1efda
    at dotnet.native.wasm:0xcfec
    at dotnet.native.wasm:0x440ad
    at e.<computed> (cwraps.ts:338:24)
callEntryPoint @ blazor.web.js:1
await in callEntryPoint (async)
ei @ blazor.web.js:1
await in ei (async)
Zr @ blazor.web.js:1
startWebAssemblyIfNotStarted @ blazor.web.js:1
resolveRendererIdForDescriptor @ blazor.web.js:1
determinePendingOperation @ blazor.web.js:1
refreshRootComponents @ blazor.web.js:1
(anonymous) @ blazor.web.js:1
setTimeout (async)
rootComponentsMayRequireRefresh @ blazor.web.js:1
onDocumentUpdated @ blazor.web.js:1
Ji @ blazor.web.js:1
```

