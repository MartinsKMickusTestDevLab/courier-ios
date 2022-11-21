# Courier iOS Overview

```swift
Courier.shared.isDebugging = true

let userId = Courier.shared.userId

await Courier.shared.signIn(
    accessToken: "asdf...",
    userId: 'example_user_id',
)

await Courier.shared.signOut()

let currentPermissionStatus = await Courier.shared.getNotificationPermissionStatus()
let requestPermissionStatus = await Courier.shared.requestNotificationPermission()

await Courier.shared.setFcmToken(token)

let fcmToken = Courier.shared.fcmToken
let apnsToken = Courier.shared.apnsToken

let messageId = await Courier.shared.sendPush(
    authKey: "asdf...",
    userId: "example_user_id",
    title: "Hey! 👋",
    body: "Courier is awesome!!",
    isProduction: false,
    providers: [.apns, .fcm],
)

class AppDelegate: CourierDelegate {

    override func pushNotificationDeliveredInForeground(message: [AnyHashable : Any]) -> UNNotificationPresentationOptions {
        print(message)
        return [.sound, .list, .banner, .badge]
    }
    
    override func pushNotificationClicked(message: [AnyHashable : Any]) {
        print(message)
    }

}
```
&emsp;

# Requirements & Support

| Min SDK | Swift | Obj-C | Firebase Cloud Messaging | Apple Push Notification Service | Expo | OneSignal | Courier Inbox | Courier Toast |
| :-----: | :---: | :---: | :----------------------: | :-----------------------------: | :--: | :-------: | :-----------: | :-----------: |
|  `13`   |  ✅   |  ✅   |            ✅            |               ✅                |  ❌  |    ❌     |      ❌       |      ❌       |

> Most of this SDK depends on a Courier account: [`Create a Courier account here`](https://app.courier.com/signup)

> Testing push notifications requires a physical device. Simulators will not work.

&emsp;

# **Installation**

>
> Link to [`Example App`](https://github.com/trycourier/courier-ios/tree/master/Example)
>

1. [`Install the package`](#1-install-the-package)
2. [`Setup`](#2-setup)
3. [`Configure Push Provider`](#3-configure-push-provider)
6. [`Managing User State`](#4-managing-user-state)
7. [`Testing Push Notifications`](#5-testing-push-notifications)

&emsp;

## **1. Install the package**
### Using Swift Package Manager

https://user-images.githubusercontent.com/29832989/202578202-32c0ebf7-c11f-46c0-905a-daa8fc3ba8bd.mov

1. Open your iOS project and increase the min SDK target to iOS 13.0+
2. In your Xcode project, go to File > Add Packages
3. Paste the following url in "Search or Enter Package URL"

```
https://github.com/trycourier/courier-ios
```

### Using Cocoapods
1. Update Podfile for ios 13.0+
```ruby
platform :ios, '13.0'
```
2. Add `pod 'Courier-iOS'` in your base target
3. Open terminal in root directory and run
```sh
pod install
```
&emsp;


## **2. Setup**
1. Change your `AppDelegate` to extend the `CourierDelegate` and add `import Courier` to the top of your `AppDelegate` file
    - This automatically syncs APNS tokens to Courier
2. Enable the "Push Notifications" capability
![Entitlement setup](https://github.com/trycourier/courier-ios/blob/master/push-notification-entitlement.gif)
    <ol start="1" type="1">
        <li>Select your Xcode project file</li>
        <li>Click your project Target</li>
        <li>Click "Signing & Capabilities"</li>
        <li>Click the small "+" to add a capability</li>
        <li>Type "Push Notifications"</li>
        <li>Press Enter</li>
    </ol>

&emsp;

### **Add the Notification Service Extension (Recommended)**

To make sure Courier can track when a notification is delivered to the device, you need to add a Notification Service Extension. Here is how to add one.

https://user-images.githubusercontent.com/29832989/202580269-863a9293-4c0b-48c9-8485-c0c43f077e12.mov

1. Download and Unzip the Courier Notification Service Extension: [`CourierNotificationServiceTemplate.zip`](https://github.com/trycourier/courier-notification-service-extension-template/archive/refs/heads/main.zip)
2. Open the folder in terminal and run `sh make_template.sh`
    - This will create the Notification Service Extension on your mac to save you time
3. Open your iOS app in Xcode and go to File > New > Target
4. Select "Courier Service" and click "Next"
5. Give the Notification Service Extension a name (i.e. "CourierService").
6. Link the Courier SDK to your extension

#### If you are using Swift Package Manager
1. Select `Courier` from package dropdown
2. Click Finish
3. Click on your project file
4. Under Targets, click on your new Target
5. Under the General tab > Frameworks and Libraries, click the "+" icon
6. Select the Courier package from the list under Courier Package > Courier

#### If you are using Cocoapods
1. Select `Courier_iOS` from the package dropdown
2. Add the following snippet to the bottom of your Podfile

```ruby 
target 'CourierService' do
    pod 'Courier-iOS'
end
```

3. Run `pod install`

&emsp;

## **3. Configure Push Provider**

> If you don't need push notification support, you can skip this step.

To get push notification to appear in your app, add support for the provider you would like to use:
- [`APNS (Apple Push Notification Service)`](https://www.courier.com/docs/guides/providers/push/apple-push-notification)
- [`FCM (Firebase Cloud Messaging)`](https://www.courier.com/docs/guides/providers/push/firebase-fcm/)

&emsp;

## **4. Managing User State**

Best user experience practice is to synchronize the current user's push notification tokens and the user's state. Courier does most of this for you automatically!

> You can use a Courier Auth Key [`found here`](https://app.courier.com/settings/api-keys) when developing.

> When you are ready for production release, you should be using a JWT as the `accessToken`.
> Here is more info about [`Going to Production`](#going-to-production)

Place these functions where you normally manage your user's state:
```swift
// Saves accessToken and userId to native level local storage
// This will persist between app sessions
await Courier.shared.signIn(
    accessToken: accessToken,
    userId: userId,
);

await Courier.shared.signOut();
```

If you followed the steps above:
- APNS tokens on iOS will automatically be synced to Courier

### **Support FCM (Firebase Cloud Messaging)**

1. Add the Firebase Package

#### If you are using Swift Package Manager
- Add the Firebase Swift Package [`firebase-ios-sdk`](https://github.com/firebase/firebase-ios-sdk)
- Select `firebase-messaging`

#### If you are using Cocoapods
- [`FirebaseCore`](https://cocoapods.org/pods/FirebaseCore)
- [`FirebaseMessaging`](https://cocoapods.org/pods/FirebaseMessaging)

2. Change your `AppDelegate` to also extend `MessagingDelegate`
3. Add `import FirebaseCore`, `import FirebaseMessaging` to the top of your `AppDelegate` file
4. Modify your `AppDelegate` according to the snippet below
    - This will automatically sync FCM tokens to Courier when Firebase detects them
    - If you need more custom integrations, you can call `Courier.shared.setFCMToken(token)` where ever works best for you
```swift
import UIKit
import Courier
import FirebaseCore
import FirebaseMessaging

@main
class AppDelegate: CourierDelegate, MessagingDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        FirebaseApp.configure()
        Messaging.messaging().delegate = self
        return true
    }

    ..
    
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        
        guard let token = fcmToken else { return }
        
        Task {
            try await Courier.shared.setFCMToken(token)    
        }
        
    }

}
```

&emsp;

## **5. Testing Push Notifications**

> If you don't need push notification support, you can skip this step.

Courier allows you to send a push notification directly from the SDK to a user id. No token juggling or backend needed!

```swift
class YourViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        Task {
        
            let notificationPermission = await Courier.shared.getNotificationPermissionStatus()
            print(notificationPermission)

            // Notification permissions must be `.authorized` to receive pushes
            let requestedNotificationPermission = await Courier.shared.requestNotificationPermission()
            print(requestedNotificationPermission)
            
            sendTestPush()
        
        }
        
    }
    
    private func sendTestPush() {
    
        Task {

            let messageId = await Courier.shared.sendPush(
                authKey: 'a_courier_auth_key_that_should_only_be_used_for_testing',
                userId: 'example_user',
                title: 'Chirp Chrip!',
                body: 'Hello from Courier 🐣',
                isProduction: false, // false == sandbox / true == production
                providers: [.apns, .fcm],
            )

        }
    
    }

}

class AppDelegate: CourierDelegate {

    ..

    override func pushNotificationDeliveredInForeground(message: [AnyHashable : Any]) -> UNNotificationPresentationOptions {
        print(message)
        return [.sound, .list, .banner, .badge] // Pass [] to hide any foreground presentation
    }

}
```

&emsp;

## **Going to Production**

For security reasons, you should not keep your `authKey` (which looks like: `pk_prod_ABCD...`) in your production app. The `authKey` is safe to test with, but you will want to use an `accessToken` in production.

To create an `accessToken`, call this: 

```curl
curl --request POST \
     --url https://api.courier.com/auth/issue-token \
     --header 'Accept: application/json' \
     --header 'Authorization: Bearer $YOUR_AUTH_KEY' \
     --header 'Content-Type: application/json' \
     --data
 '{
    "scope": "user_id:$YOUR_USER_ID write:user-tokens",
    "expires_in": "$YOUR_NUMBER days"
  }'
```

Or generate one here:
[`Issue Courier Access Token`](https://www.courier.com/docs/reference/auth/issue-token/)

> This request to issue a token should likely exist in a separate endpoint served on your backend.

&emsp;

## **Share feedback with Courier**

We want to make this the best SDK for managing notifications! Have an idea or feedback about our SDKs? Here are some links to contact us:

- [Courier Feedback](https://feedback.courier.com/)
- [Courier iOS Issues](https://github.com/trycourier/courier-ios/issues)

