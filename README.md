# exploration_tool-GUI-react

This is the GUI for the Exloration tool of Flocktracker!


## Setting Up React Native Environment and Integrating React Native Fragments into Existing Android App

1. [Software Installation](#installation)
2. [Project Directory Structure](#project-directory-structure)
3. [Modifying Dependencies](#modifying-dependencies)
4. [Building and Compiling](#building-and-compiling)
5. [Switching Between React Native and Android](#switching-between-react-native-and-android)
6. [Common Bugs and Solutions](#common-bugs-and-solutions)

## Software Installation

Go to the following [link](https://facebook.github.io/react-native/docs/getting-started.html), which is the Getting Started page for React Native by Facebook. 
Select the option 'Building Projects with Native Code,' and click the appropriate Development OS and Target OS (I chose macOS and Android).
Follow the appropriate instructions as specified. Since there is a pre-existing android app already, you can skip everything starting at the section 'Creating a new application'.

Note: The project structure was set up while using macOS X and Terminal, so some instructions may be different for Windows/non-Linux users.

## Project Directory Structure

![Directory Structure](http://i.imgur.com/7VPqqVW.png)

The image above is an example of my project directory structure. My project directory is the 'ReactIntegration' folder. I then created a directory within my project directory called 'android' and copy-pasted my existing Android code into that directory as shown. It is important that the index.android.js, package.json, and node_modules are within the ReactIntegration folder and not the android folder. THe ReactIntegration.iml and pacakge-lock.json should be generated when you run the code so no need to worry about those. Furthermore, if you do not see a 'node modules' directory you should cd into the project directory and execute the command:

```bash
npm install
```

## Modifying Dependencies

There are several files that need to be modified in order to set up the correct dependecies to allow React Native to be run. Additionally, please copy over the package.json file from the github over to your project directory. The following files I will specify are:

1. build.gradle (in ../android directory)
2. build.gradle (in ../app directory)
3. gradle.properties (in ../android directory)

The changes I made are shown in the figures and outlined by red boxes.

#### build.gradle (in ../android directory)

![build android](http://i.imgur.com/VZxoriK.png)

#### build.gradle (in ../app directory)

![build app](http://i.imgur.com/NfGIXGn.png)

#### gradle.properties (in ../android directory)

![properties android](http://i.imgur.com/OuQLExt.png)

## Building and Compiling

### Development

When you are running the integrated react native code for the first time, remember to first cd into the project directory and type in the command line:

```bash
npm start
```

This will start a package manager that will bundle the index.android.js file into an index.android.bundle file so that the phone can interpret the React Native script. If the phone loads a blank screen, it is likely because the package manager is not running and it cannot find the index.android.bundle file.

There are two ways you can run/compile the project code. One is through Android Studio, and the other is through Terminal.

On Android Studio: Open the ../android directory as your project directory in Android Studio and run it with a connected Android Device or using a Virtual Device.

On Terminal: Make sure a Virtual Device is running on your computer first (you can use Android Studio for this) and then cd into your project directory and type into Terminal:

```bash
react-native run-android
```

### Unsigned apk Without Development Server

The above method works with Android Studio, but it requires a packaging server to be run when the app is initially launched - otherwise the code will crash. When the application is packaged for final distribution, a packager must be used that will compile and bundle the react code into the apk. The following method works nicely (source https://stackoverflow.com/questions/35283959/build-and-install-unsigned-apk-on-device-without-the-development-server/36961021#36961021):

(In the ReactIntegration folder)

Bundle debug build:
```bash
react-native bundle --dev false --platform android --entry-file index.android.js --bundle-output ./android/explorationfragment/build/intermediates/assets/debug/index.android.bundle --assets-dest ./android/explorationfragment/build/intermediates/res/merged/debug
```
Create debug build:
```bash
cd android
./gradlew assembleDebug
```

Generated apk will be located at android/explorationfragment/build/outputs/apk

If your code is not updating properly, make sure the index.android.bundle file is being modified properly. If not, a gradle clean may be necessary. For a clean build you may need to create the folder ./android/explorationfragment/build/intermediates/assets/debug/ if it does not exist already.

## Switching Between React Native and Android

This stackoverflow question was my reference: [Switching Fragments](https://stackoverflow.com/questions/36005461/how-to-switch-react-native-js-file-to-android-activity)

The first solution that wanted to create a module seemed excessive, so I followed the second solution which used 'linking' and it worked.

However, this seems to be switching from a JS file to an android activity rather than between fragments. One note is that if you follow the way that they set up the intent filter in the AndroidManifest.xml file, then the url should be 'Linking.openURL('my.special.scheme://')'. This URL set-up worked when trying to switch to the Android activity after a Button press.

```javascript
onPress={this.redirectFragment}

redirectFragment() {
    Linking.openURL('my.special.scheme://').catch(err => console.error('An error occurred', err));
}
```

## Common Bugs and Solutions

This website solves some of the bugs I had encountered: [Debugging Website](https://guillermoorellana.es/react-native/2016/07/05/integrating-react-native-in-an-existing-application.html)

The following are a few bugs that I encountered and noted while I was trying to integrate the existing app and React Native.

Problem 1: After running ‘react-native run-android’, would get stuck at ‘addingDebugger 98%’. 

Solution: Need to have an android emulator running or android device connected

Problem 2: When running the android app on Android Studio, would get ‘apk-slice-4/7’ errors.

Solution: Go to the settings of Android Studio and turn off the Instant Run in the “Build, Execution, Tasks” section

Problem 3: Error saying that the methods “onHostPause” and “onHostResume” did not exist in the ReactActivity.java file

Solution: Make sure that the path for the react-native repository is correct in the maven dependences of the build.gradle file for the project (not the app build.gradle file). 

Problem 4: Getting a problem with failure to detecting some library. It was an ‘UnsatisfiedLinkError.’ The cause is apparently because React Native does not support 64 bit.

Solution: Go to gradle.properties and add ‘android.useDeprecatedNdk=true.’ Exclude the 64 bit third party packages in your build.gradle

Problem 5: When you register the React Native component in the Activity.java file you need to make sure the names match. See the diagram to see how the names should match in the index.android.js file and the Activity.java file.

Solution: Blue matches with blue, so the class name and the second argument of ‘registerComponent’ need to be the same. Red matches red, so the first argument of registerComponent is the actual name of the component which will be referenced in the ReactActivity.java file in the ‘startReactApplication’

![component](http://i.imgur.com/d76MXsP.png)

Problem 6: App gets stuck on the loading screen with the swirly circle. No error visible.

Solution: Check to make sure the app has location permissions. Make sure we are targeting version 22 of android, otherwise permissions will need to be enabled manually in the phone settings.

Problem 7: Sometimes the project won't compile and gradle will not build. 

Solution: Make sure to check the 'dependencies' in your package.json file and that they match the ones in the github repo. Should be: "dependencies": {
    "react": "^16.0.0-alpha.12",
    "react-native": "^0.46.2"
  }













