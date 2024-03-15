# Kodi debugpy Addon (script.module.debugpy)

A Kodi wrapper for Microsoft [`debugpy`](https://github.com/microsoft/debugpy). The version of this addon will always match the version of `debugpy` included.

## Disclaimer

Psychosaurian makes no warranty that...

- this software will meet your requirements
- this software will work as described and be error-free
- the quality of this software will meet your expectations
- any errors in this software or documentation will be corrected

This software or documentation...

- Could include technical or other mistakes, inaccuracies or typographical errors. Psychosaurian may make changes to this software or documentation without notice.
- May be out of date and Psychosaurian makes no commitment to update these materials.

Psychosaurian assumes no responsibility for errors or ommissions in this software or documentation.

In no event shall Psychosaurian be liable to you or any third parties for any special, punitive, incidental, indirect or consequential damages of any kind, or any damages whatsoever, including, without limitation, those resulting from loss of use, data, or profits, whether or not Psychosaurian has been advised of the possibility of such damages, and on any theory of liability, arising out of or in connection with the use of this software.

The use of this software is done at your own discretion and risk and with agreement that you will be solely responsible for any damage to your computer systems or electronic devices and any loss of data that results from such activities. No advice or information, whether oral or written, obtained by you from Psychosaurian shall create any warranty for the software.

## A Word about platforms

Although versions of `Visual Studio Code` exist for Linux and Mac OSX, only Windows allows Kodi to be installed in a fully [Portable Mode](https://kodi.wiki/view/Portable_mode), which is required to install more than one version of Kodi on your development machine for version compatibility verification. There is a way to use Kodi Portable Mode on Mac OSX but it is only recommended for very experienced developers and makes for a frustrating workflow since every time you make a change the entire bundle is replaced and you will lose any existing portable data. Kodi for Linux does not support Portable Mode at all. For this reason this addon will only install on Kodi for Windows or Mac OSX, with the former the preferred platform since it allows the most seamless intersection of development and debugging in a single editor with no need to push changed code to the target device. If you are a developer testing your code on multiple Kodi versions you need Portable Mode, and only Kodi for Windows does this properly.

This plugin has only been tested on Windows x64 and is not guaranteed to work at all on Mac OSX. If you wish to code and debug on Linux it is recommended you use `Web-PDB` instead of `debugpy` since it works on all platforms.

## debugpy vs. Web-PDB

Prior to Kodi version 19 (Matrix) a wrapper addon for debugpy's predecessor `pydevd` was available in the main Kodi repository to debug Kodi addons and plugins from Visual Studio (`VS`) and Visual Studio Code (`VSCode`). When Kodi migrated to Python 3.x with Matrix that addon was no longer functional, and since `pydevd` was deprecated by then in favour of `debugpy`, it was never updated for Kodi 19+.

Luckily Roman Miroshnychenko stepped in with a browser-based solution called [Web-PDB](https://github.com/romanvm/kodi.web-pdb). This required developers to edit their addon code with one tool but then launch a web browser to catch breakpoints and display code with the current runtime value of variables. `Web-PDB` provides the basic essential set of features for debugging and a host of Kodi addon developers have used it with great success, but there is no link between the code editor and the debug tool and its functionality is limited compared to more robust solutions such as the `VS` and `VSCode` dubuggers. `Web-PDB` lets you insert breakpoints in code, `Step over`, `Step into`, `Step out`, and `Continue`. It also shows global and local variables at each step and allows evaluation of ad-hoc python expressions with access to all imported libraries and in-scope variables. That's enough to be useful when debugging your addon code, but it falls well short of other modern debugger tools.

 `Web-PDB` doesn't have...

* Watch expressions (easily view just the things you are interested in every time execution breaks)
* Call stack (how did execution get to this breakpoint?)
* Persistent breakpoints which can easily be set, disabled, and cleared right in the source code editor
* Conditional breakpoints (break execution when something specific happens without changing your code to check for it)

Those last two limitations are mainly the result of the debugger tool being completely separate from the code editor. Also, with no link between the two tools if your debugging finds a problem you can't just fix the code in the debugger window, you have to go back to your editor and find the relevant line to edit, which might be in any of the python files in your project. With better debuggers you step through your actual source files and when you find something you need to change you are already in your editor on the correct line in the correct file.

With `Web-PDB` the only way to put a persistent breakpoint in one of the python files in your project is a call to `web_pdb.set_trace()`, but this requires each file where a persistent breakpoint is needed to `import web_pdb` and add one or more calls to `set_trace()`. Although `debugpy` has a similar function called `breakpoint()` it is rarely needed or used since you can just turn breakpoints on and off in `VS/VSCode` with a click of the mouse.

`Web-PBD` lists all in-scope variables in one of two relatively small boxes for local and global variables. Once you get deep into addon code before breaking those two boxes can get seriously cluttered and it can be really difficult to find a variable you are interested in, especially if some of those variables contain large entities, such as BeautifulSoup results. The lack of a watch expression display is really frustrating in this situation. Often the only way to see what you want free of all the other variable's clutter is to type in an ad-hoc expression, and you have to do that repeatedly every time you break execution.

The combination of `debugpy` and the powerful `VS` or `VSCode` debuggers overcome all these limitations and aggravations.

## Usage

These instructions assume that you have already set up `VS` or '`VSCode` for Python code development and have any associated Python debugger extensions installed (Installing the Microsoft Python extension in `VSCode` automatically installs the Microsoft Python Debugger extension too).

`debugpy` requires four things to operate...

1. The `debugpy` plugin must be installed in Kodi.
2. The addon you plan to debug needs an import requirement in `addon.xml` for `script.module.debugpy`.
3. A few lines of code must be added at the beginning of the top-level python file of your project.
4. `VS` or `VSCode` must have a properly set up launch configuration.

Also, as stated [earlier](#a-word-about-platforms), it is highly recommended that you install one or more Kodi versions in [Portable Mode](https://kodi.wiki/view/Portable_mode) for full `debugpy` functionality.

Instructions for each of these four steps are detailed below...

### Installation

This plugin is an extension of `xbmc.python.module` which means that it won't appear in any Kodi addon searches. There are two ways to install it...

1. Manually "Install from zip file".
2. Install an addon repository that contains `debugpy` and then do step #2 from the previous section.

If you use the second method `debugpy` will be installed automatically the next time you start Kodi. With either method `debugpy` will not show up in `My add-ons` but will instead appear in Settings under `Manage dependencies`.

### addon.xml
Any addon you want to debug needs the following entry in the `<requires>` section of its `addon.xml`...

```
<addons>
    <addon ...>
        <requires>
            ...
            <import addon="script.module.debugpy" />
            ...
        </requires>
        ...
        ...
    </addon>
</addons>
```

As stated [earlier](#installation), if Kodi finds an installed addon with this requirement when it starts up it will automatically install `debugpy` if it can find it in any of the repositories it knows about. It is strongly recommended that you obtain this plugin by installing the most recent [Psychosaurian Kodi addon repository](https://Psychosaurian.github.io/repo/).

### Python code

Traditionally, when using `debugpy` the following snippet is inserted into your Python code...

```
import debugpy

debugpy.listen(5678)
debugpy.wait_for_client()
debugpy.breakpoint() # OPTIONAL
```

The `listen(5678)` line tells `debugpy` to launch a server on `localhost:5678` to listen for connections from the debugger client (`VS` or `VSCode`).

`listen()` is non-blocking so `wait_for_client()` is needed to halt script execution until the client connects. The call to `breakpoint()` is rarely needed since you can just place a breakpoint on any line you want to stop at directly from the code editor.

In most Python projects which start a main process which runs until the program exits completely (which destroys the `debugpy` listener), this setup works well, but with Kodi addons there are several problems. Kodi python scripts are executed repeatedly with different arguments in a separate process each time. This is what causes problems with using the standard `debugpy` startup code. After the first pass every time the addon's top-level script is executed again it will attempt to set up a listener on port `5678`, but the previous listener was attached to Kodi itself, not the process it ran in, and will still exist so the attempt to start another listener on the same port will fail, generating an Exception. Since you can't reconnect to the existing listener from the new process the `wait_for_client()` call will block forever. Unfortunately there is also no way from Kodi to destroy the existing listener and create a new one, so setting up `debugpy` this way will only work the first time the script is called. On the very next execution of the script `debugpy` will effectively block forever and the only way to recover is to exit Kodi.

The solution to this problem is to flip the connection and make `VS/VSCode` the listener and have `debugpy` connect to that instead of the other way around.

```
import debugpy

try:
    debugpy.connect(5678)
except:
    pass
```

This time we have set up `debugpy` to connect rather than listen. When execution reaches `connect(5678)` `debugpy` will attempt to connect to the `VS/VSCode` debugger listening on `localhost:5678`. If the debugger is listening `debugpy` will connect to it and your top-level script will continue on, stopping at any breakpoints in it or in any child scripts. However, if `VS/VSCode` is not listening then an Exception is raised, which is ignored by the `pass` statement, and the script continues on, bypassing any breakpoints set in it or any child scripts. We don't need the `wait_for_client()` call this time since `debugpy` will either connect or it won't and we want the script to continue executing either way. This gives us the useful capability to let the scripts run without debugging just by not turning on the listener in `VS/VSCode`.

### Debug launch configuration

Since most Kodi addon developers will be using `VSCode` rather than `VS` these instructions will show how to set up a `debugpy` launch configuration for the former, but doing the same for the full `Visual Studio` will be similar.

In `VSCode` with the addon workspace and one of your addon Python files open, click on `Run and Debug` on the left side of the screen or press `Ctrl-Shift-D`. Click the gear icon at the top of the left panel. If you don't see the gear icon that means you don't have a `launch.json` file yet. In this case click the `create a launch.json file` link, choose `addons` as the location for the new `json` file, select `Python Debugger`, and then `Python File`. This will create and open a basic Python `launch.json`.

Add a new launch configuration...

```
{
    "version": "0.2.0",
    "configurations": [        
        {
            ...
            Other configuration
            ...
        },
        {
            "name": "Python: Kodi Remote Listen",
            "type": "debugpy",
            "request": "attach",
            "listen": {
                "host": "localhost",
                "port": 5678
            },
            "pathMappings": [
                {
                    "localRoot": "${workspaceFolder}",
                    "remoteRoot": "${workspaceFolder}"
                }
            ],
            "justMyCode": true
        },
        {
            ...
            Other configuration
            ...
       }
    ]
}
```

Notice the entry for `"listen"`. If we were using the traditional `debugpy` method using `debugpy.listen(5678)` then we would use `"connect"` in the launch configuration instead. You can change `"name"` to anything you like and that will appear in the list of launch configurations. The other entry of note is `"pathMappings"`. This tells the debugger how to find the correct source files to match the ones `debugpy` breaks in. If you are not using a Windows machine for debugging then you may need to change the `"localRoot"` entry. The one in this example assumes that Kodi is installed on Windows, your `VSCode` workspace location is the Kodi addons directory , and you are planning to debug addons under that. If `VSCode` complains that it can't find the source file when debugging this is the value you need to adjust. For more information on `pathMappings` and other launch configuration settings see [Microsoft's `debugpy` documentation](https://code.visualstudio.com/docs/python/debugging).

## Debugging with VSCode and debugpy

To debug your addon with Visual Studio Code start by creating a `VSCode` workspace in the Kodi `addons` directory. If you haven't done so already add the `debugpy` requirement to your addon's `addon.xml` file as [described above](#addonxml). Open your addon's top-level Python script and add this at the very top of the file right after your other imports but before any executable code...

```
import debugpy as dbg

try:
   dbg.connect(5678)
except:
   pass
```

In the `VSCode` editor set any breakpoints you want by clicking on the left end of the line or use the `Run >> New Breakpoint` menu option to set more sophisticated breakpoints such as `Conditional`, `Triggered`, or `Function`.

Start Kodi and select your new listener launch configuration but don't activate it yet. Navigate in your addon until you get to a point where you want to debug the next thing you select, then in `VS/VSCode` hit `F5` to start listening. Tell your addon to proceed and when `debugpy` reaches any breakpoints you have set you can add expressions to the watch list, edit variable values, and do many more things that just aren't possible using `Web-PDB`.

**IMPORTANT:** One quirk is that you must restart the listener every time your addon finishes executing a pass through its code. Even though your addon process has ended the `VSCode` listener holds the connection open and prevents any new connections on that port. If you don't stop and restart it between passes then `debugpy` will hang the next time `debugpy.connect()` is called until you stop the listener with `Shift-F5` or the `Disconnect` button <img src="Disconnect.png" width="15" height="15" /> on the debugger toolbar. As soon as you kill the listener your script will pick up execution again but won't respect any breakpoints during this pass through since `debugpy` won't be connected. Make sure to turn the listener back on before trying again.

It's easy to get into the habit of pressing `F5` to start the listener just before you tell your addon to do something you want to debug, responding to any breakpoints, and then pressing `F5` to let your script finish the current pass through before pressing `Shift-F5` to turn off listening or `Ctrl-Shift-F5` (or click the `Restart` button <img src="Restart.png" width="15" height="15" /> on the debugger toolbar) to stop and immediately restart listening ready for the next pass through your addon script. Remember that if the listener is not running when you start some action in your addon then the script will run through that pass as if the debugger and breakpoints aren't there.