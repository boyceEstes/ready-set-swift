# General common errors

* [App installation failed](#app-installation-failed)
* [A valid provisioning profile for this executable was not found](#a-valid-provisioning-profile-for-this-executable-was-not-found)

## App installation failed

#### On:
Install app to physical iPhoneXs

#### Error:
```
This application's application-identifier entitlement does not match that of the installed application. These values must match for an upgrade to be allowed.
```

#### Solution:
* With device open and Xcode open, select `Window > Devices`
* In the left tab, select your problem device.
* In the detail panel on the right, remove offending app from "Installed Apps" list
* Relaunch app to device

#### Source:
https://stackoverflow.com/questions/32677133/app-installation-failed-due-to-application-identifier-entitlement 



## A valid provisioning profile for this executable was not found
#### On:
Install app to physical iPhoneXs

#### Error:
```
A valid provisioning profile for this executable was not found.

```

#### Attempts:
* Set to Legacy Build System
	* `File` > `Project Settings` 
	* Select `Legacy Build System` from `Build System` dropdown

#### Solution:
* I made a Unit Test target that was not using the same signing account for `Team` in `Signing & Capabilities`
* Changed the Team to be the same as the main target, in this case it was `Halo Health, Inc`
* Cleaned and rebuilt project

#### Source
https://stackoverflow.com/questions/52424462/xcode-10-a-valid-provisioning-profile-for-this-executable-was-not-found