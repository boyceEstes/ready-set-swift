# Crashing

* [Why do crashes occur](#why-do-crashes-occur)
* [Access crash logs](#access-crash-logs)
* [Enable crash logging](#enable-crash-logging)
* [Misc crash logging](#misc-crash-logging)
* [Best practices for symbolication](#best-practices-for-symbolication)
* [Symbolicate a misc crash log](#symbolicate-a-misc-crash-log)
	* [Acquire dSYM files from App Store](#acquire-dsym-files-from-app-store)
	* [Locate the existing dSYM files on your device](#locate-the-existing-dsym-files-on-your-device)
	* [Performing the symbolication process](#performing-the-symbolication-process)



## Why do crashes occur
* Impossible for CPU to execute code
* Operating system is enforcing a policy
* Programming language is prevent behavior
* The developer is causing the behavior


Whenever you are in the debugger in Xcode, you will see what is called a **symbolicated** crash log whenever there is a crash. You can go through the *back trace* and look at the order in which events occurred so that you can see what led to the crash. You can even click items in the backtrace to take you directly to the line of code that it is referring to.

However, whenever an app crashes that is not attached to Xcode, it will not be so easy to look through. The symbols that happen in the backtrace are represented in binary names and addresses. This is an **unsymbolicated** crash log.

## Access crash logs
(Specifically form TestFlight and App Store apps)

Download the crash logs using the `Organizer` in the `Crashes` tab on the left side panel.

* You can get to the Crashes Organizer by going to Window > Organizer.

### Brief look around
* \*You can click on the button `Open in Project` (in the righthand panel) to open the Crash report in your Xcode project like a normal debugging session crash.
* The crashes are grouped by those that are similar.
* When you click on a crash, you get the **symbolicated** crash log that will show you the fully symbollicated crash log to see how your code acted when it began to go awry.

* When you work on a crash you might have a little checkmark to the left of the Crash in the Crash report list.

### Is that it?
Yes! After you have gotten to this point with the Symbolicated crash logs - you are home free. It will be much easier to debug now that you can see the process leading to the crash. Open the project in Xcode to get the best view of everything in action.
<br />


## Enable crash logging
In order to have crash logs come in symbolicated to your Crash Organizer view whenever they happen. 

* App Store customers that opt in to having their crash logs reported will send their crashes directly to your Crash Organizer and everything "just works".
* Sign in to Xcode with your apple id.
* When you upload your app to the store, make sure you include "symbols" so that you can have server-side symbolization.
* Open Crashes tab of the Organizer window.


## Misc crash logging
When the crash is not coming from TestFlight or App Store what do ya do? How to see crash logs not from there?

* Devices Window
* XCTests - any crash logs resulting from your test run are shown (symbolicated)
* Console app - Mac console app to view logs from your mac or the simulator
* Sharing from device - Settings > Privacy > Analytics > Analytics Data > All your fun logs - these can be shared directly

### Devices Window
Use the Devices Window - symbolicated using the local symbol information on your mac.

* Open Window > Devices and Simulators.
* Go to a device that is plugged in and tap **View Device logs**


## Best practices for symbolication
* Upload Symbols with App - server-side symbolication
* Save app archives - archives contains copies of debug symbols (dsyms) for local symbolization
* Download debug symbols for bitcode apps - use the `Archive` Window's 'Download Debug Symbols' button to download any debug symbols that come from a bitcode compilation

## Symbolicate a misc crash log
So it seems like most times these logs should just symbolicate whenever you begin working with them in Xcode. However, it seems like sometimes, they just don't. All good, we'll just need to get the necessary information to symbolicate them ourselves.

* Make sure that your crash report is a `.crash` file. (Not `.ips`). You can just rename the extension.
* Open the [device logs](#view-device-logs)
* Drag and drop the crash report into the logs. Then scroll around to find it. Cross your fingers, hopefully it appears symbolicated.
	* If it doesn't, before moving on, right click the crash log and select 'Re-symbolicate Log'


If the crash-report didn't symbolicate or only partially symbolicated, Xcode can't locate the matching symbol information, and you'll need to acquire the system information. This probably isn't the problem but better to check than not.

* You need device symbol information matching the operating system version recorded in the crash report.
	* For iOS - this should be as easy as connecting a device that has the same operating system as the device that the crash log is for. Xcode should automatically copy operating system symbols from each device you connect to your Mac.
		* **Note**: iOS 13.1.0 has different symbols than iOS 13.1.2


Here is the make or break time. You need to locate the debug symbols for your app, app extension, or frameworks that aren't symbolicated.

There are two ways to do this:
1. Acquire dSYM files from App Store
2. Locate the existing dSYM files using Spotlight

If you have third-party frameworks, you might need to explicitly ask them for their dSYM file

### Acquire dSYM files from App Store
This process only works whenever you upload to the App Store with *bitcode* enabled. Otherwise the dSYM files should be on the Mac that archived the build, proceed to [Locate the existing dSYM files using Spotlight](#locate-the-existing-dysm-files-using-spotlight)

When you submit a build with *bitcode* to the App Store, the App Store converts that bitcode to machine code. Thus, your mac won't contain dSYM files that match the final build, because final compilation occurs in the App Store. This App Store created build has different UUID and different dSYM files than the one created by original Xcode archive.

* **Note**: You can find out if your app distributes using bitcode by going to your main target in Xcode and looking at the `Enable Bitcode` property in the 'Build Settings' tab.


### Locate the existing dSYM files on your device
This process is all about finding the dSYM file that was used to archive the build on your machine, and using it to symbolicate the crash log.

* **Note**: Do not do this step if you are uploading to the App Store with *bitcode* enabled. Doing that produces a different dSYM than you will have on your machine when you archive. Follow these [steps](#acquire-dsym-files-from-app-store)

1. Find a frame in the backtrace that isn't symbolicated. A frame, in this case, is just a single line in the backline that ideally comes from our project. Ex:
```
17  Doc Halo                      	0x000000010430d03c 0x1042a4000 + 430140
```
2. Note the name in the second column, in this case it is `Doc Halo`. This is the binary image of the symbol's name.
3. Locate the UUID of the binary image at the bottom of the crash log, or type in this handy lil command in terminal:
```
% grep --after-context=1000 "Binary Images:" <Path to Crash Report> | grep <Binary Name>
```
4. Note the binary information, it should look like this:
```
0x1042a4000 - 0x104677fff Doc Halo arm64  <58183f56bfba392cb2fa78f6cf8059a2> /var/containers/Bundle/Application/26BF098A-A67D-40B6-823E-227EBEF587F7/Doc Halo.app/Doc Halo
```

The part in the 5th(ish) column between the arrow brackets is the binary's UUID: `58183f56bfba392cb2fa78f6cf8059a2`.
5. Separate the binary UUID into groups of 8-4-4-4-12 - All letters **Must** be uppercase `58183F56-BFBA-392C-B2FA-78F6CF8059A2`
6. **Important** step - Search for this binary's dSYM file in terminal: 
```
mdfind "com_apple_xcode_dsym_uuids == 58183F56-BFBA-392C-B2FA-78F6CF8059A2"

// You can use a more general command to find all of the binaries on the system:

mdfind -name .dSYM | while read -r line; do dwarfdump -u "$line"; done
```

If nothing shows up:
* Verify that you don't have bitcode enabled, shown in this [section](#acquire-dsym-files-from-app-store) - If you do, you need to be downloading the dSYM from the App Store as shown there.
* Verify that your Xcode archive for the version that crashed is still on your machine.
	* Try to retain archives for releases if you ever want to retrieve the dSYM file for a crash.
* Verify that the Xcode archive is located somewhere Spotlight can find it, such as your macOS home directory.
* Verify that your build includes debug information (I think that you can do this by going to your target's Build Settings and searching 'Debug Information Format')


### Performing the symbolication process

#### Step 1

Gather the materials:
* The debug symbols file (`.dSYM`). As described above, this must match the archived build that reported the crash.
* The app file - This could be described as a `.app` but a `.ipa` will work as well.
* The unsymbolicated crash log (must be `.crash` extension)

Place copy files to an easily accessible folder. `halo-crash-Jun-23-2021`

#### Step 2
Find the symbolicatecrash script file.

It should be in the following directory:
```
/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
```

If the file is not found there, use terminal to find it using the following command:
```
$ find /Applications/Xcode.app -name symbolicatecrash
```
You can do this in termianl if you can't find it

#### Step 3
Copy and paste the symbolicatecrash file into the folder created in [Step 1](#step-1)

#### Step 4
Navigate to the folder created in [Step 1](#step-1) in terminal.
```
$ cd Desktop/halo-crash-Jun-23-2021
```

Switch enviornment
```
$ export DEVELOPER_DIR=$(xcode-select --print-path)
```

Symbolicate the crash
```
$ ./symbolicatecrash -v MyApp-Crash-log.crash MyApp.dSYM
```

That's it!



### Sources: 
* [StackOverflow - how to symbolicate crash log Xcode](https://stackoverflow.com/questions/25855389/how-to-symbolicate-crash-log-xcode)
* [Apple documentation - symbolicating crash log](https://developer.apple.com/documentation/xcode/adding-identifiable-symbol-names-to-a-crash-report#Symbolicate-the-Crash-Report-with-the-Command-Line)


