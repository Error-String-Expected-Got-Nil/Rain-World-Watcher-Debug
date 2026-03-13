# Attaching a Debugger to Rain World

For those unfamiliar: A debugger, in programming, is a program that "attaches" to another running process and allows you to pause it in place, allowing you to observe the state of all variables accessible at that point, step through the code line-by-line, and even run arbitrary code to test things with few limitations. Such a tool is vital for fixing obscure bugs or errors in complex logic, and being able to use one will save you a substantial amount of headache.

The folder named `DebugBinaries` in this GitHub repository contains everything necessary to convert Rain World into a Unity development build, which you are able to attach a managed .NET debugger to once the game has been launched.

It should be noted, these binaries are specifically for the Windows/Steam release of the game updated to the most recent patch at the time of writing (March 13th, 2026). Outside major updates to the game, this should probably work on future patches. I am unsure if this would work for playing the game through Proton on Linux, but I strongly suspect the answer is 'no'.

## Applying and Unapplying the Patch

The following is a list of all files contained in `DebugBinaries`, showing where they go relative to Rain World's root directory. To find Rain World's root directory from Steam, right click on it in your library, mouse over 'Manage', then hit 'Browse local files' in the menu that appears. You'll know you're in the right place if you see `RainWorld.exe`.
- `/BepInEx/config/BepInEx.cfg`
- `/MonoBleedingEdge/EmbedRuntime/mono-2.0-bdwgc.dll`
- `/RainWorld_Data/boot.config`
- `/RainWorld.exe`
- `/UnityPlayer.dll`
- `/WinPixEventRuntime.dll`

Before continuing, I recommend creating a new folder in the root directory (I titled mine `ReleaseBinaries`), locating these files, and placing them inside, with the exception of `WinPixEventRuntime.dll`, as this does not come with the game. Doing this will make it easier to revert your changes later. Note that your own `ReleaseBinaries` folder should have the same folder structure as shown here: For example, `BepInEx.cfg` should be in a `config` folder inside a `BepInEx` folder inside `ReleaseBinaries`.

To apply the patch, copy all of the contents of `DebugBinaries`, attempt to paste them into Rain World's root directory, and confirm to *replace* all files. Once done, you have now successfully converted Rain World into a development build. The results of this are detailed in the next section. Note that the `RainWorld.exe` in `DebugBinaries` has the Unity logo for an icon, while the game's has Rain World's usual slugcat icon. If the icon doesn't update immediately, you can ignore it, the file should still be replaced correctly.

To unapply the patch, you have two options: If you created the `ReleaseBinaries` folder, you can copy its contents and paste them in the root directory in the same way. Otherwise, you'll need to go find all files mentioned in the list above, delete them, then go to Rain World in your Steam library, right click, hit 'Properties', go to the 'Installed files' tab, and hit 'Verify integrity of the game files' button, then wait for the process to complete. This is slower, but works.

An addition not included in `DebugBinaries` are debugging symbols for the game's own assembly. In the game's root directory, go to `/BepInEx/utils` and find `Assembly-CSharp.pdb`. Drop this into `/RainWorld_Data/Managed` and ignore it when applying/unapplying the patch. Doing this will allow you to use your debugger on vanilla code as well. It only isn't included in `DebugBinaries` because it technically isn't freely available to those who do not own the game.

## What the Patch Does

If the patch was applied successfully, you will see the following differences when launching the game:
- A terminal containing console log output will appear when launching. Closing this will also close the game, so leave it be!
- A logging window with a few exceptions will appear in the game itself. I'm not sure what these are, but they are benign, so you can just ignore them and close this window when you get your cursor.
- The text "Development Build" will be shown in the bottom-right corner of the game window.

Additionally, the game will permit a debugger to attach to it. Both JetBrains Rider and Visual Studio should have an option for this. In Rider: Hit the vertical ellipses (three dots) next to build button, then hit 'Attach to Unity process...' in the drop down, then the process for Rain World should be the only option in the following list if it is running. I am not sure where this option is in Visual Studio (as I only use Rider), but it's likely similar.

Once the debugger is attached, you will be able to use it to debug your code. Note that you will have to build your mod with debugging symbols, in particular a `.pdb` file. How you do this depends on your IDE, but shouldn't be hard to figure out. Search `<your ide> add pdb to build` and you should find it. Ensure the `.pdb` is in the same folder as your mod's `.dll`.

If you want to debug initialization code that runs right when the game starts, you can go into `boot.config` and add the line `wait-for-managed-debugger=1` to the end of the file. This will cause the game to have a popup on launch telling you to attach a debugger, and to pause while this popup is waiting. You can either attach your debugger or close it to continue.

Note that, in development builds like this, most runtime optimizations are disabled, which may significantly reduce the game's performance in some situations. Additionally, while rare, it is possible for certain obscure bugs to appear in release builds, only to disappear in release builds. Mostly this is due to things like method inlining (make sure you don't apply hooks to virtual base methods with empty bodies!). 

## Where do These Files Come From

This section is purely for those curious, and is not necessary for those who simply wish to use the patch.

The modified `BepInEx.cfg` originates from the original `DebugWorld.zip` file that was posted to the Rain World discord server's `#modding-resources` channel a couple years ago. Frankly. I am not sure why it's required, but using the default config results in the game getting stuck in an exception loop after launching. The modified config prevents this.

`boot.config` is simply modified from the game's default to have the line `player-connection-debug=1`, which makes the game allow a debugger to connect.

All other files are from the Unity editor itself. On Windows, if you install the Unity hub, and then get the same Unity editor version Rain World uses (`2020.3.45f1`), you can find these files at `C:\Program Files\Unity\Hub\Editor\2020.3.45f1\Editor\Data\PlaybackEngines\windowsstandalonesupport\Variations\win64_development_mono`. The `RainWorld.exe` included in `DebugBinaries` is actually just `WindowsPlayer.exe` renamed.