!REDIRECT "https://sendbird.gitbooks.io/how-to-build-an-ios-messaging-app/content/en/install_sendbird_framwork_via_cocoapods.html"

# Install Inteage Framework via CocoaPods

The latest version of Inteage iOS framework is already included in the project, so you don’t have to install it again. If you’d like to install the framework for your own project, follow the steps below:

1. You must have CocoaPods installed. Check out https://cocoapods.org/
1. If you don’t have ```Podfile``` in your project, create it.
1. Insert the following line into ```Podfile```.
```
pod 'InteageSDK'
```
1. Install Inteage framework. Run the following command in the terminal.
```
$ pod install
```
1. After the installation, open ```<YOUR_PROJECT>.xcworkspace```. Don't confuse it with ```<YOUR_PROJECT>.xcodeproj```.
```
$ open <YOUR_PROJECT>.xcworkspace
```

See this [link](https://inteage.gitbooks.io/inteage-ios-sdk/content/en/download_sdk.html) for details.



