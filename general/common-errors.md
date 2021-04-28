# General common errors

* [App installation failed](#app-installation-failed)

## App installation failed
On:
Install app to physical iPhoneXs

Error:
```
This application's application-identifier entitlement does not match that of the installed application. These values must match for an upgrade to be allowed.
```

Solution:
* With device open and Xcode open, select `Window > Devices`
* In the left tab, select your problem device.
* In the detail panel on the right, remove offending app from "Installed Apps" list
* Relaunch app to device

Source:
https://stackoverflow.com/questions/32677133/app-installation-failed-due-to-application-identifier-entitlement 