# Businesses

Author: [Dekode1859](https://github.com/Dekode1859).

This standalone repository contains only the Businesses phone app mod.
The desktop manager is maintained separately at
[schedule-one-mods/schedule-one-manager](https://github.com/schedule-one-mods/schedule-one-manager).

For distribution, run `./package.ps1`. Release ZIPs are written to
`artifacts/releases`; validate one with `python verify_package.py <zip>`.
See RELEASING.md for publishing and maintaining game-version compatibility.

A Schedule I phone app that reuses the game's laundering computer screen to
show live business deposits, amounts, and time remaining. This project targets
the installed IL2CPP build, game 0.4.7f5 / Unity 2022.3.62f2, with MelonLoader
0.7.3. Other game builds and co-op clients need separate verification.

## Use

Open the phone and select **Businesses**. The header shows the selected
business; **Next** cycles through your businesses. The native timeline marker
and amount travel together. The lower table shows every active deposit across
your businesses. Time left uses in-game hours and minutes.

Use **Back**, the phone Home button, Escape, or right-click to return to the phone home screen.
Putting the phone away and reopening it retains the app, like native apps.
The physical Home control uses the native app's phone-model collider hit test.
Its input hook runs immediately before GameInput clears its per-frame click
list, so short clicks are handled in the same frame as native apps.
Header labels do not receive pointer events; the visible button backgrounds
define their click areas. An opaque background fills the entire phone screen.

This is a status viewer. It does not start deposits, change laundering limits,
shorten processing time, or alter bank balances. Completed operations disappear
from the live list, as on the computer; there is no historical payment ledger.

## Build and install

Install .NET SDK **8.0.426**, the pinned compiler in
`src/LaunderingStatus/global.json`. The build uses `tools/dotnet/dotnet.exe`
if present, otherwise `dotnet` on PATH. PowerShell and Python are used for
packaging and package validation. MelonLoader **0.7.3** must already have
generated its `Il2CppAssemblies` for the selected IL2CPP game installation.

```powershell
.\build.ps1
# Close the game, then build and install:
.\build.ps1 -Deploy
# For another game location:
.\build.ps1 -GameDir 'C:\path\to\Schedule I'
```

The only mod DLL to install is `artifacts/LaunderingStatus/LaunderingStatus.dll`,
copied into the game's `Mods` directory. No S1API or Sideload installation is
required. The phone integration is adapted from Sideload's MIT-licensed host;
see THIRD-PARTY-NOTICES.md. Include that notice when distributing the DLL.

## Isolated playtest

```powershell
.\playtest.ps1 -SourceSave 'C:\path\to\SaveGame_5'
```

This creates a new copy under `artifacts/playtest`, builds/deploys the mod, and
loads the copy. The game can save into that test copy during testing. The source
save is left untouched. Close the game before rerunning. `--quitfix` is used for
the playtest because this installation can otherwise hang on shutdown.

Diagnostics are enabled only with `--laundry-diagnostics`. They write
`UserData/LaunderingStatus-inspection.txt` and, when the app opens,
`UserData/LaunderingStatus-snapshot.json` beneath the game directory. The latter
records both UI text and the underlying live operation values for comparison.
Mod and exception logs are in `MelonLoader/Latest.log`.

## Implementation

The mod creates one local phone icon and a landscape panel after the save loads.
It clones the native `LaunderingInterface/Container` presentation subtree into
the phone, removing the world-screen state components and deposit dialog before
activation. Serialized button events are replaced, so cloned controls cannot
invoke the original station's deposit action.

The game owns operation state. The app reads `Business.OwnedBusinesses` and
`LaunderingOperations`, refreshes once per real second while visible, and uses
each operation's actual completion duration. Rows are rebuilt only when the set
of operations changes. It does not clone a native phone App component or replace
its singleton. Exit and phone-close listeners are removed on scene unload.

If the player owns no business and there is no computer UI available to clone,
the app displays an empty-state message and checks again on a later open.

## Files and rollback

SDKs, references, generated assemblies, local builds and saves are excluded
from version control. Earlier save backups and installation research remain
in the original manager workspace; they are not part of this mod repository.
Known-good 0.1.3 release files are available locally under `artifacts/releases`.

To remove just this app, close the game and remove `Mods/LaunderingStatus.dll`.
To disable MelonLoader as well, rename its added `version.dll` to
`version.dll.disabled`. No original game DLL was replaced by this setup.

## Automated local releases

Edit the version once in `Directory.Build.props`, update the changelog, then use `publish.ps1` to build, push and publish. An optional post-commit hook publishes version increases on `main`. See [PUBLISHING.md](PUBLISHING.md).
