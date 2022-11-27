# PrivateSDK Document for iOS platform
## Table of contents

1. [Download](#download)
2. [Install](#install)
3. [Configuration](#configuration)
   1. [Get config info](#get-config-info)
   2. [Unzip the zip file then import the framework and resource bundle files into the your project](#unzip-the-zip-file-then-import-the-framework-and-resource-bundle-files-into-the-your-project)
   3. [Xcode Settings Configuration](#xcode-settings-configuration)
4. [Implement](#implement)
   1. [Calling initialization and user authentication function](#calling-initialization-and-user-authentication-function)
      1. [Calling the login function](#calling-the-login-function)
      2. [Calling the logout function](#calling-the-logout-function)
      3. [Calling the delete account function](#calling-the-delete-account-function)
   2. [In App Purchase](#in-app-purchase)
      1. [Calling function to show payment center](#calling-function-to-show-payment-center)
      2. [Receiving payment notification on backend](#receiving-payment-notification-on-backend)
   3. [Push notification](#push-notification) 
   4. [URL schemes](#url-schemes)
   5. [Hanlde application status states](#hanlde-application-status-states)
5. [Build file for testing or store submission](#build-file-for-testing-or-store-submission)

## Download

All resource includes: the latest PrivateSDK, libraries and Demo App can be downloaded at [Release section](https://github.com/GameGateSDK/SDKDocument/releases) or https://github.com/GameGateSDK/SDKDocument/releases

## Install
### Get config info

All the necessary information will be provided such as: gameId, game bundleId, certificate for distribution build, provisioning files, iap item list , API list , ... all can be found in the `document` that game operators provide for you.

If you have any questions, please contact us, customer care, key operations or translation personnel are always ready to help.

### Unzip the zip file then import the framework and resource bundle files into the game project



### Import all the necessary libraries and frameworks

>Note : These libraries are required!

![PrivateSDK](/images/1.png)

as shown below: Accelerate.framework, ...

![Frameworks](/images/2.png)

>Note: If you are not using AppTrackingTransparency, please remove the AppTrackingTransparency framework from Frameworks, Libaries and Embedded content.

Build phases must include following items:


![Build phases](/images/7.png)

`GoogleService-Info.plist`, `GoogleSignIn.bundle`, `PrivateSDKResources.bundle` should be added into `Copy Bundle Resources`.

### Configure the info.plist file


The picture is for reference only, the values above should be changed for each game:

![Info.plist](/images/3.png)


Replace following info with provided info in `document` that game operators provide for you:

- `SDKAppId`: The id of your app in our system.
- `FacebookAppID` & `FacebookClientToken`: for Facebook Login
  ```
  <dict>
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>fb{FacebookAppID}</string>
            </array>
        </dict>
        ....
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>fbapi</string>
        <string>fb-messenger-share-api</string>
    </array>  
  </dict>
  ```

- `GoogleClientID` & `GoogleReversedClientID`: for Google Login
    ```
    <dict>
    <key>CFBundleURLTypes</key>
    <array>
        ....
        <dict>
            <key>CFBundleTypeRole</key>
            <string>Editor</string>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>{GoogleReversedClientID}</string>
            </array>
        </dict>
    </array>
  </dict>
  ```
- Privacy - Tracking Usage Description
```
<key>NSUserTrackingUsageDescription</key>
<string>Mã nhận dạng này sẽ được sử dụng để phân phối quảng cáo được cá nhân hóa cho bạn</string>

```
- NSPhotoLibraryUsageDescription
```
<key>NSPhotoLibraryUsageDescription</key>
<string>Cho phép ứng dụng truy cập thư viện ảnh của bạn!</string>
```

[//]: # (Note: except  FacebookDisplayName, all above values must be configured in the info.plist file.)

### Configuration

- Certificate configuration: certificate file (signature) will be provided here xcode settings

![Certificate](/images/4.png)

- Configure linker flag: set the value to `-ObjC,  -lc++`

![Configure linker flag](/images/5.png)

- Capability configuration: Push notification, In-App Purchase, Sign in with Apple

![Configure linker flag](/images/6.png)

## Implement

After importing all resources and configuring the `info.plist` file and xcode settings, we start coding:


### Private delegates

>Note: These functions must be implemented to handle sdk event.

```
//Login success
-(void)PSLoginDelegate:(int)user_id withName:(NSString*) user_name accesToken:(NSString *)access_token;

//Logout sucesss
- (void)PSLogoutDelegate;

//Purchase success
- (void)PSDidPurchaseSuccess;

//Purchase fail
- (void)PSDidPurchaseDismiss;

//Delete account success
- (void)PSDidDeleteAccount;
```



### Calling initialization and user functions

```

#import <PrivateSDK/PrivateSDK.h>
#import <PrivateSDK/PSConfigs.h>
.....


@interface AppDelegate () <PSDelegate, UIApplicationDelegate>


@end

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

   ....
    [PSConfigs setMainWindow:self.window];
    [PrivateSDK initSDKWithGameDelegate:self application:application launchOptions:launchOptions loginManually:NO];
   

    ...
    return YES;
}
```


>Note: Make sure you call `[PSConfigs setMainWindow]` after the Unity window has been initialized.


### Calling the login function
>Note : This function must be implemented.


`[PrivateSDK callLogin];`

The SDK will handle the login, the result will be returned via the following delegate:

```
- (void)PSLoginDelegate:(int) user_id withName:(NSString*) user_name accesToken:(NSString *)access_token {
   //Hanlde login result

}
```

### Calling the logout function

>Note : This function must be implemented.


```[PrivateSDK callLogout];```

The SDK will handle the logout, the successful logout will be returned via the following delegate:
```
- (void)PSLogoutDelegate {
  //NSLog(@"Logout successfully!");
  [PrivateSDK callLogin];
  }
```
>Note: It is required to handle ingame logout in this delegate.

### Calling the delete account function
>Note : This function is required to be implemented from Apple.

`[PrivateSDK callDeleteAcc];`

The result will be return via the following delegate:

```
- (void)PSDidDeleteAccount{
    //Handle your game logic and calll logout function to let user re-login;
    [PrivateSDK callLogOut];
}
```

### In App Purchase

>This is the only payment method with AppStore, it is **strictly prohibited** in code to have other forms of payment. Completely remove the hidden mechanisms, the mechanism of converting other forms of payment other than the form of the SDK. For example: Code switch must be deleted according to the status: type_pay, appstore_ pay, ...

### Calling function to show payment center

`[PrivateSDK callPayment:@"server_id" andCharId:@"char_id"];`

`server_id`: ID of server that user is logging

`char_id`: ID of character of user

The payment result will be returned in delegates:

```
- (void)PSDidPurchaseSuccess {
    //NSLog(@"Purchase successfully!");
}

- (void)PSDidPurchaseDismiss {
    //NSLog(@"Purchase cancelled!");
}

```

### Receiving payment notification on backend

### Push notification

>Note : This function must be called.

The SDK has a built-in push notification service subscription. You must implement following functions to make it works:

```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
    [PrivateSDK application:application didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
}

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [PrivateSDK application:application didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}
```
  One delegate is responsible for sending device tokens, another delegate is tasked with receiving notifications of the device.


### URL schemes

>Note: This function must be implemented

```
- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {

    return [PrivateSDK application:app
                           openURL:url
                           options:options];
}

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler{
    
        return [PrivateSDK application:application
                  continueUserActivity:userActivity
                    restorationHandler:restorationHandler];
    
}
```

### Hanlde application status states

>Note: These functions must be called.

```
- (void)applicationDidEnterBackground:(UIApplication *)application {
    // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later.
    // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    [PrivateSDK sdkHandleDidEnterBackground];
}

- (void)applicationWillEnterForeground:(UIApplication *)application {
    // Called as part of the transition from the background to the inactive state; here you can undo many of the changes made on entering the background.
    [PrivateSDK sdkHandleDidEnterForeground];
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
    /*
     Need for facebook
     */
    [PrivateSDK sdkHandleDidBecomeActive];
}

- (void)applicationWillTerminate:(UIApplication *)application {
    // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.

    /*
     Need for facebook
     */
    [PrivateSDK sdkHandleWillTerminate];
}

```


## Build and deploy

Build file for testing or store submission

- Step 1: use the certificate file and provisioning file to build the `xcarchive` file.
- Step 2: send us that `xcarchive` file.

And we're done here.

>✨Remarks: just one more thing:
> - Please run the demo application first and see how it works before contact us for asking.
> - You should test the builds carefully before sending them to us
> 
> That way we will save a lot of time to communicate between the parties when an error occurs.