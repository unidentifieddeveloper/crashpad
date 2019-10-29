# Crashpad for non-Googlers

Are you writing cool apps? Want to crash report like the big G but don't want
to bother setting up a build environment just for Crashpad?

*Unidentified Developer* got you covered!

This repo contains everything you need to build Crashpad in a CMake
environment, making integration in your own project much easier.

We also provide pre-built native binaries on NuGet so you can get started even
easier!

## Getting started

Install the NuGet package.

```powershell
λ nuget install UnidentifiedDeveloper.Crashpad.vc142
```

Add the include directories and libraries to your project - this will differ
between development environments.

Next up, initialize Crashpad using any available example. Here's one. Remember
to add error handling as well - refer to [the Crashpad documentation](https://crashpad.chromium.org/doxygen/index.html).

```c++
#include <client/crash_report_database.h>
#include <client/settings.h>
#include <client/crashpad_client.h>

void InitializeCrashpad()
{
    crashpad::CrashpadClient client;

    client.StartHandler(
        base::FilePath("path/to/crashpad_handler.exe"),
        base::FilePath("Crashpad/db"),
        base::FilePath("Crashpad/db"),
        "https://error-handling-api",
        { },
        { "--no-rate-limit" },
        true,
        true);

    client.WaitForHandlerStart(INFINITE);
}
```


## Building the project from source

Initialize Git submodules (*crashpad*, *mini_chromium* and *zlib*).

```powershell
λ git submodule update --init
```

Generate the build.

```powershell
λ cmake -A [Win32|x64] -S . -B build/[x86|x64]
```

Run the build.

```powershell
λ cmake --build build/[x86|x64] --config [Debug|Release]
```

## Create a NuGet package

Make sure you have built all four configurations (Debug-x86, Debug-x86_64,
Release-x86, Release-x86_64).

```powershell
λ nuget pack package.nuspec -Version <Version>
```
