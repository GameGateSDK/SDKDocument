# PrivateSDK Document for iOS platform

## Table of contents

1. [Download](#download)
2. [Configuration](#configuration)
    1. [Get config info](#get-config-info)
    2. [Add files to your project](#add-files-to-your-project)
    3. [Modify project files](#modify-project-files)
4. [Implementing](#implementing)
    1. [Calling initialization and user authentication function](#calling-initialization-and-user-authentication-function)
        1. [Calling the login function](#calling-the-login-function)
        2. [Calling the logout function](#calling-the-logout-function)
        3. [Calling the delete account function](#calling-the-delete-account-function)
    2. [In App Purchase](#in-app-purchase)
        1. [Calling function to show payment center](#calling-function-to-show-payment-center)
        2. [Receiving payment notification on backend](#receiving-payment-notification-on-backend)
    3. [Push notification](#push-notification)
    4. [Hanlde application status states](#hanlde-application-status-states)
5. [Build file for testing or store submission](#build-file-for-testing-or-store-submission)
   1. [Release on Google Play](#release-on-google-play)
      1. [Package name](#package-name)
      2. [Version code](#version-code)
      3. [KeyStore](#keystore)
   2. [Release on different countries playstore](#release-on-different-countries-playstore)

## Download the required SDK and libraries.

All resource includes: the latest PrivateSDK, libraries and Demo App can be downloaded at [Release section](https://github.com/GameGateSDK/SDKDocument/releases) or https://github.com/GameGateSDK/SDKDocument/releases

## Configuration
### Get config info

All the necessary information will be provided such as: appId, packagename, API list, ... all can be found in the `document` that game operators provide for you.

If you have any questions, please contact us, customer care, key operations or translation personnel are always ready to help.

### Add files to your project

>Note : the highlight files are required!

![PrivateSDK](/images/8.png)


### Modify project files

![PrivateSDK](/images/9.png)

The highlight values should be updated like the value in the `document`

Add the following code to the `build.gradle` at level project 
```
buildscript {

    dependencies {
        ...
        classpath 'com.google.gms:google-services:4.3.13'
    }
}

allprojects {
    repositories {
       ...
        flatDir{
            dirs 'libs'
        }

    }
}
```

- Add the following code to the `build.gradle` at level app

```
dependencies {
    ....
    implementation project(':PrivateSDK')

    implementation 'androidx.multidex:multidex:2.0.1'
    implementation platform('com.google.firebase:firebase-bom:30.3.2')
    implementation 'com.google.firebase:firebase-analytics'
    implementation 'com.google.firebase:firebase-messaging'

    implementation 'com.facebook.android:facebook-login:15.1.0'
    implementation 'com.google.android.gms:play-services-auth:20.3.0'

    implementation "com.android.billingclient:billing:5.1.0"
    implementation 'com.google.guava:guava:31.1-android'
    implementation 'org.apache.httpcomponents:httpcore:4.4.15'

}
```

- Multidex configuration (For games that need to use multidex)
>Note: This declares is optional.
```
android {

    defaultConfig {
       ....
        multiDexEnabled true
        ...
    }
    dexOptions {
        javaMaxHeapSize "4g" //specify the heap size for the dex process
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

- Turn on AndroidX for the project in file `gradle.properties`
```
org.gradle.daemon=true
org.gradle.jvmargs=-Xms1024m -Xmx4096m
android.useAndroidX=true
android.enableJetifier=true
android.enableR8=false
```
- Add `fb_login_protocol_scheme` string to `strings.xml` file
```
<?xml version="1.0" encoding="utf-8"?>
<resources>

   ...

    <string name="fb_login_protocol_scheme">fb{FacebookAppId}</string>
    ...

</resources>
```




### Configure the AndroidMainifest.xml file

- Import necessary permission
```
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="com.android.vending.BILLING" />
```

- Add android: `name = "com.teamae.sdk.PSApp"` field in the application tag
```
<application
        android:name="com.teamae.sdk.PSApp"
        android:allowBackup="true">
        ...
 </application>
```
- Add required declare for `Facebook` & `Push Notification` implement

```
 <application
        ......
        <activity
            android:name="com.facebook.FacebookActivity"
            android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"
            tools:replace="android:theme" />
            
         <activity
            android:name="com.facebook.CustomTabActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data android:scheme="@string/fb_login_protocol_scheme" />
            </intent-filter>
        </activity>

        <service
            android:name="com.teamae.sdk.service.PSMessagingService"    
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
        
         <receiver
            android:name="com.teamae.sdk.service.InstallReceiver"
            android:exported="true"
            android:permission="android.permission.INSTALL_PACKAGES">
            <intent-filter>
                <action android:name="com.android.vending.INSTALL_REFERRER" />
            </intent-filter>
        </receiver>

        .....


    </application>
```



[//]: # (Note: except  FacebookDisplayName, all above values must be configured in the info.plist file.)

## Implementing

After importing all resources and configuring the `info.plist` file and xcode settings, we start coding:


### Handle Event Listener

>Note: These functions must be implemented to handle sdk event.

```
public interface SDKEventListener {
    interface OnLogoutListener {
        void onLogoutSuccessful();
    }

    interface OnLoginListener {
        void onLoginSuccessful(String userName, String userId, String accessToken);

        void onLoginFailed(String reason);
    }

    interface OnPaymentListener {
        void onSuccessful();
        void onDismiss();
    }

    interface OnDelAccListener {
        void onSuccessful();
    }
}

```



### Calling initialization and user functions

```

import com.teamae.sdk.PrivateSDK;
import com.teamae.sdk.listeners.SDKEventListener;
.....


public class MainActivity....{

....

 private PrivateSDK mPrivateSDK;
 private Activity mActivity = this;
 
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        .....

        // initialize SDK
        mPrivateSDK = new PrivateSDK(mActivity, mLoginListener, mLogoutListener);
}

// login callback
    private SDKEventListener.OnLoginListener mLoginListener = new SDKEventListener.OnLoginListener() {

        @Override
        public void onLoginSuccessful(String userName, String userId, String accessToken) {
            //Handle your logic game
        }

        @Override
        public void onLoginFailed(String reason) {
            //Handle your logic game
        }
    };
    // --

    // logout callback
    private SDKEventListener.OnLogoutListener mLogoutListener = new SDKEventListener.OnLogoutListener() {

        @Override
        public void onLogoutSuccessful() {
            //Handle your logic game before call logout function to force user to re-login
            PrivateSDK.callLogin();
        }
    };
   
....
}
```


### Calling the login function
>Note : This function must be implemented.


`mPrivateSDK.callLogin();`

The SDK will handle the login, the result will be returned in listener at SDK init.


### Calling the logout function

>Note : This function must be implemented.


``` mPrivateSDK.callLogout();```

The SDK will handle the logout, the result will be returned in listener at SDK init.

### Calling the delete account function

``` 
PrivateSDK.callDelAcc(new SDKEventListener.OnDelAccListener() {
                    @Override
                    public void onSuccessful() {
                    //Handle game logic and call logout to force user re-login
                        mPrivateSDK.callLogout();
                    }
                });
```


### In App Purchase

>This is the only payment method with PlayStore, it is **strictly prohibited** in code to have other forms of payment. Completely remove the hidden mechanisms, the mechanism of converting other forms of payment other than the form of the SDK. For example: Code switch must be deleted according to the status: type_pay,  ...

### Calling function to show payment center


`PrivateSDK.callPayment("server_id", "char_id", mOnPayListener);`

`int server_id`: ID of server that user is logging

`int char_id`: ID of character of user

The payment result will be returned in `mOnPayListener`:

```
private SDKEventListener.OnPaymentListener mOnPayListener = new SDKEventListener.OnPaymentListener() {

        @Override
        public void onSuccessful() {

        }

        @Override
        public void onDismiss() {


        }
    };

```

### Receiving payment notification on backend


### Push Notification

SDK will receive and handle Push Notification automatically. Please ensure that you have already declared the following tag in `<application>` tag in `AndroidMinifest.xml` file:

```
  <application....>
  
  <service
            android:name="com.teamae.sdk.service.PSMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
  </applivation
```

### Hanlde application status states

>Note: These functions must be called.

```
@Override
    protected void onPause() {

        if (mPrivateSDK != null) {
            mPrivateSDK.onPause();
        }
        super.onPause();
    }

    @Override
    protected void onResume() {

        if (mPrivateSDK != null) {
            mPrivateSDK.onResume();
        }
        super.onResume();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (mPrivateSDK != null) {
            mPrivateSDK.onActivityResult(requestCode, resultCode, data);
        }
        super.onActivityResult(requestCode, resultCode, data);
    }

    @Override
    protected void onDestroy() {
        if (mPrivateSDK != null) {
            mPrivateSDK.onDestroy();
        }
        super.onDestroy();
    }

```


## Build file for testing or store submission

### Release on Google Play

#### Package name

Package name for files is unique and permanent, so please set package name carefully. Package names cannot be deleted or reused in the future.

#### Version code

When building a new AAB(APK) file, you need to set the `versionCode` of the new APK file higher than the `versionCode` of the old AAB(APK) file to be able to update on Google Play. So, don't forget to change the `versionCode` before exporting new AAB(APK) files.



#### KeyStore

We will send you a `keystore`, and use it to sign file.



### Release on different countries playstore

- If the APK file size is less than 100MB or not using OBB file, we can use the APK file released on Google Play for different countries playstore release.

- If the APK file size larger than 100MB or not using OBB files, we need to build one big APK file with every additional resources (images, audio, ...) inside, without downloading from server.


>✨Remarks: just one more thing. You should test the builds carefully before sending them to us. That way we will save a lot of time to communicate between the parties when an error occurs.

