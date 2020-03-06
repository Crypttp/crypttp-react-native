# Crypttp-react-native

## 1. Install crypttp-parser
```
npm i -s crypttp-parser
```

Make sure you already support official `Linking`. [Check here](https://reactnative.dev/docs/linking.html)

## 2. Handle incoming url
```JavaScript
// Example

import CrypttpParser from 'crypttp-parser';
import { Platform, Linking } from 'react-native';

class Home extends React.Component {
    static navigationOptions = { // A
        title: 'SendPage',
    };

    componentDidMount() { // B
        if (Platform.OS === 'android') {
            Linking.getInitialURL().then(url => {
                this.navigate(url);
            });
        } else {
            Linking.addEventListener('url', this.handleOpenURL);
        }
    }
    
    componentWillUnmount() { // C
        Linking.removeEventListener('url', this.handleOpenURL);
    }

    handleOpenURL = (event) => { // D
        this.navigate(event.url);
    }
    
    navigate = (url) => { // E
        const { navigate } = this.props.navigation;
        
        const routeName = url.split("://");

        if (routeName === "crypttp" || routeName === "yourappname") {
            const decoded = CrypttpParser(url)
            
            // this variable will be used in point 6
            applicationOpenedByCrypttp = true

            console.log(decoded)
            /*{
                "id":"merchantId",
                "successUrl": "successUrl",
                "errorUrl": "errorUrl",
                "params":[
                    [
                        "Currency1",
                        "Amount of Currency1 to send",
                        "Address where to send",
                        "custom data to include in transaction",
                        "memo if required",
                    ]
                ]
            }*/

            navigate('YourSendPage', decoded)
        };
    }
}
```

## 3. Configuring for iOS
### Step 1. Add URL type to info.plist

1. Open info.plist and at the top of the file, create a new property called URL types
2. Expand item 0 (zero) and choose URL Schemes.
3. Give item 0 the name you would like your app to be linkable as which is 1 - yourappname, 2 - crypttp

### Step 2. Add the following code to AppDelegate.m

Below last existing import add this import:
```
#import <React/RCTLinkingManager.h>
```

Directly below `@implementation AppDelegate` add this code:
```Swift
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  return [RCTLinkingManager application:application openURL:url
                      sourceApplication:sourceApplication annotation:annotation];
}

// Only if your app is using [Universal Links](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html).
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity
 restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler
{
 return [RCTLinkingManager application:application
                  continueUserActivity:userActivity
                    restorationHandler:restorationHandler];
}
```

## 4. Configuring Android

First, you need to open our Manifest file and add the app name, in our case yourappname and crypttp.

In `android/app/src/main`, open `AndroidManifest.xml` and add the following intent filter below the last intent filter already listed, within the `<activity>` tag:

```XML
<intent-filter android:label="filter_react_native">
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />

    <data
        android:host="crypttp"
        android:pathPrefix=""
        android:scheme="crypttp"
        tools:ignore="AppLinkUrlError" />
    <data
        android:host="crypttp.com"
        android:pathPrefix="/crypttp"
        android:scheme="http" />
    <data
        android:host="crypttp.com"
        android:pathPrefix="/crypttp"
        android:scheme="https" />

    <!-- register your app deeplink -->
    <data
        android:host="crypttp"
        android:pathPrefix=""
        android:scheme="yourappname" />
</intent-filter>
```

## 5. Register first install handler

Get your merchant id in dashboard

<img src="https://i.imgur.com/fyW4P6s.png" height="100%" />

Add the following line to the code where you have first launch handler

If you don't handle first launch (to show welcome screen for instant) you can add it, [example](https://stackoverflow.com/questions/40715266/how-to-detect-first-launch-in-react-native)

```JavaScript
// make sure Linking class imported 
import { Linking } from 'react-native';

// ... your first laucnh handler
if (firstLaunch) {
    // ... your logic
    Linking.openURL("https://api.crypttp.com/track/installation?id=<your merchant id>")
}
else {
    // ... your logic
}
```

## 6. Upgrade your successfull transaction handler
```JavaScript
// ... some code
.on('confirmation', (receipt) => {
    // ... your logic

    if (applicationOpenedByCrypttp) {
        // * description 1
        axios.post('https://api.crypttp.com/aqr/saveConfirmedHash', {
            id: '<id>', // it can be found in passed params in url
            transactionHash: receipt.hash // send hash of confirmed transaction
        })
        // * description 2
        Linking.openUrl(decoded.successUrl)
    }
})
.catch('error', () => {
    // ... your logic

    // * description 2
    if (applicationOpenedByCrypttp) {
        Linking.openUrl(decoded.errorUrl)
    }
})
```

* Description 1

This method help us track paid transactions and provide statistics to our merchants. 

Which in turn helps attract new merchants.

* description 2

If transaction successfully confirmed you should open merchant app / website, to return user to page that purchase made successfully

or to the page that payment failed.

## 7. Register your app in Crypttp system

Signup at [Dashboard](https://crypttp.com/dashboard)

Navigate to Settings/Wallet App

Set:

* Name

* Deeplink

* Discription

* Icon

* Available currencies

* Urls to AppStore

In addition this configuration will help us promote your wallet app. 

Every user that has no wallet installed while paying at Crypttp merchants will be redirected to a special page where user can find featured wallets