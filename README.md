[**Project Page**](https://meteor.ame.asu.edu/projects/lenscap/)  |  [**Video**](https://www.youtube.com/watch?v=wzC0DOEbuj8&t=2s)  | [**Paper**](https://dl.acm.org/doi/abs/10.1145/3458864.3467676)

# LensCap_MobiSys2021
Source Code for compiling LensCap plugin for Unreal Engine projects

Table of Contents
=================

* [Overview of LensCap](#overview)
* [System Requirements](#system-requirements)
* [Configuring Unreal Engine for LensCap](#configure-unreal)
  * 1.1 [Setting up LensCap source files in Unreal Engine Source](#11-setup-lenscap-source-files-in-unreal)
  * 1.2 [Creating a Unreal Project with LensCap plugins](12-creating-unreal-project-with-lenscap)
* [Configuring LensCap for stand-alone Android project in Unreal Engine](#configure-android)
  * 1.1 [Opening the Android project](#11-opening-android-project)
  * 1.2 [Enabling LensCap in Android](#12-enabling-lenscap-in-android)
  * 1.3 [App usage in Android](#13-app-usage-in-android)
* [Revision History](#revision-history)

## Overview
This Project contains all the necessary source code to create a split-process (LensCap) application using Unreal Engine, via plugins, and how to enable the LensCap functionality in stand-alone Android projects
## System Requirements
* General Requirements:
  * Unreal Engine v4.24.3 built from source
  * Microsoft Visual Studio 2019 v16.6.0
* Android Requirements:
  * Android Studio v3.6.1
  * SDK Android 9.0 (Pie)
  * NDK r14b (it can be installed via CodeWorks, instructions on how to install it are availavble [here](https://docs.unrealengine.com/4.26/en-US/SharingAndReleasing/Mobile/Android/InstallingAndroidCodeWorksAndroid/))
## Configure Unreal
In order to get the LensCap plugin to correctly build in Unreal Engine, some modifications and additions must be made to the source code of the Engine itself. Creating a new project with the LensCap plugins also requires additional steps.
### 1.1 Setup LensCap source files in Unreal
Make a blank folder named *LensCap* the following path of your Unreal Engine source code: UnrealEngine\Engine\Source\ThirdParty
Copy the four libraries found under [engineComponents](/engineComponents) into the newly created *LensCap* folder.
### 1.2 Creating Unreal project with LensCap
There are a few additional steps in building an Unreal Engine project with LensCap plugins correctly:
1. Open Up the new project creator in Unreal and select new project
2. Use the Handheld AR Template
3. In the options switch build for Desktop/Console to Mobile/Tablet and create the project
4. Close the project, and navigate to its source code, in its main folder copy the [Plugins](/Plugins) folder found in this repo.
5. Open the project in Unreal Engine again, under settings->plugins->Project(at very bottom of left-hand-side) enable the two plugins *LensCap Network Transceiver module* and *Lenscap VisualTransceiver module*, **Do Not Press Restart afterwards**, just close the settings window when done.
6. Under Settings-> Project Settings-> Platforms -> Android SDK, ensure that the *Location of Android NDK* is pointing to the r14b NDK required and that the *NDK API Level* is set as *android-19*.
7. Create an empty C++ class, File->New C++ class-> none. The name you give it does not matter.
8. Also under file, click on *Refresh Visual Studio Project* or *Generate Visual Studio Project* if the latter is not available
9. Close Unreal Engine and open up the project's visual studio solution
10. Then under solution explorer right-click your project select Project Only->Build only [Your Project Name]
11. Once it has built, open up the project in Unreal Engine and launch it to the desired device.
12. The application will then launch on the device
## Configure Android
In order to enable LensCap functionality the Android source code needs some changes to it, and therfore must be edited, the easiest way to do this is through Android Studio and the following instructions assume that Android Studiois being used and that the Unreal Project has already been created and built.
### 1.1 Opening Android project
The Android source code project can be opened by opening the following file path in Android Studio: [Your Project]\Intermediate\Android\APK\Gradle.
It is also recommended that the view of the file explorer is switch to *Project*, for the sake of making the files easier to access.
### 1.2 Enabling LensCap in Android
There are a few additional steps in enabling LensCap in Android Studio, due to the limitations of automation in Unreal Engine:
1. Comment out ndk version line in local.properties 
2. In gradle.properties change the SDK versions to:
```
MIN_SDK_VERSION=26
TARGET_SDK_VERSION=28
```
3. In settings.gradle add the following anywhere in the file:
```
include ':app-networks',':VisualTransceiver',':NetworkTransceiver'
include ':SharedMemLib'
```
4. The project build.gradle should be changed to look accordingly:
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext.kotlin_version = '1.3.61'
    ext.kotlin_version = '1.3.21'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
    apply from: 'buildscriptAdditions.gradle', to: buildscript
}

apply from: 'baseBuildAdditions.gradle'

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
5. In the **apps** build.gradle the following dependency should be added:
```
implementation project(path: ':VisualTransceiver')
```
6. In the apps gameactivity.java (found under gradle->app->src->main->java->com->epicgames.ue4) add the following to its imports:
```
import edu.ame.asu.meteor.lenscap.visualtransceiver.VisualLensCapTransceiver;
```
7. Uncomment the following lines for the LensCap variables and functions:
   * private VisualLensCapTransceiver lensCap; (line: 565)
   * StartLensCap() (line: 637-657)
8. In line 540 the **networks** gameactivity (found under gradle->app-network->src->main->java->com->epicgames.ue4) the old package name has to be replaced with the package name of your new project:
```
val launchIntent = packageManager.getLaunchIntentForPackage("com.meteor.ARtest_2")
```
*com.meteor.ARtest_2* should be replaced with own package name
9. In line 88 of the NetworkTransceiver's transmitter service (found under gradle->NetworkTransciever->main->java.edu.ame.asu.meteor.lenscap->networktransceiver) the old package name has to be replaced with the package name of your new project:
```
ioServiceIntent.setPackage("com.meteor.ARtest_2")
```
*com.meteor.ARtest_2* should be replaced with own package name

### App Usage in Android
Launching the application with LensCap also requires some additional steps:
1. First the visual process (app-app) has to be launched, and the Start AR button must be pressed prior to launching the network.
2. Then the network process (app-networks) can be launched, and the start AR button can be pressed on it to begin your LensCap experience
3. The overlay of the network process can be dismissed at any time by pressing the back key on the device.

## Citation
```
@inproceedings{10.1145/3458864.3467676,
  author = {Hu, Jinhan and Iosifescu, Andrei and LiKamWa, Robert},
  title = {LensCap: Split-Process Framework for Fine-Grained Visual Privacy Control for Augmented Reality Apps},
  year = {2021},
  isbn = {9781450384438},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://doi.org/10.1145/3458864.3467676},
  doi = {10.1145/3458864.3467676},
  booktitle = {Proceedings of the 19th Annual International Conference on Mobile Systems, Applications, and Services},
  pages = {14–27},
  numpages = {14},
  keywords = {AR application development, unreal engine, visual privacy, augmented reality security, split-process control},
  location = {Virtual Event, Wisconsin},
  series = {MobiSys '21}
}
```

## Revision History
* 6/6/2021- Added Initial source code
* 6/11/2021- Added additional NDK information and cleaned up naming conventions
* 10/5/2021- Created test branch with updated version of project
