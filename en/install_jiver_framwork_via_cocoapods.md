# Install JIVER Framework via CocoaPods

The latest version of JIVER iOS framework is already included in the project, so you don’t have to install it again. If you’d like to install the framework for your own project, follow the steps below:

1. You must have CocoaPods installed. Refer to https://cocoapods.org/
1. If you don’t have ```Podfile``` in your project, create it.
1. Insert below line into ```Podfile```.
```
pod 'JiverSDK'
```
1. Install JIVER framework. Run following command in terminal.
```
$ pod install
```
1. After the installation, you have to open ```<YOUR_PROJECT>.xcworkspace``` instead of ```<YOUR_PROJECT>.xcodeproj```.
```
$ open <YOUR_PROJECT>.xcworkspace
```

See this [link](https://jiver.gitbooks.io/ios-sdk/content/en/download_sdk.html) for detail.



