---
section: libraries
toc: true
description: How to install, initialize and use Auth0.Swift
url: /libraries/auth0-swift
topics:
  - libraries
  - swift
contentType:
    - how-to
    - index
useCase: enable-mobile-auth
---

# Auth0.swift

Auth0.swift is a client-side library for Auth0.

::: note
Check out the [Auth0.swift repository](https://github.com/auth0/Auth0.swift) on GitHub.
:::

## Requirements

- iOS 9+ / macOS 10.11+ / tvOS 9.0+ / watchOS 2.0+
- Xcode 11.4+ / 12.x
- Swift 4.x / 5.x

## Installation

### Cocoapods

If you are using [Cocoapods](https://cocoapods.org), add this line to your `Podfile`:

```ruby
pod 'Auth0', '~> 1.0'
```

Then run `pod install`.

::: note
For more information on Cocoapods, check [their official documentation](https://guides.cocoapods.org/using/getting-started.html).
:::

### Carthage

If you are using [Carthage](https://github.com/Carthage/Carthage), add the following line to your `Cartfile`:

```ruby
github "auth0/Auth0.swift" ~> 1.0
```

Then run `carthage bootstrap`.

::: note
For more information about Carthage usage, check [their official documentation](https://github.com/Carthage/Carthage#if-youre-building-for-ios-tvos-or-watchos).
:::

### SPM

If you are using the Swift Package Manager, open the following menu item in Xcode:

**File > Swift Packages > Add Package Dependency...**

In the **Choose Package Repository** prompt add this url: 

```text
https://github.com/auth0/Auth0.swift.git
```

Then press **Next** and complete the remaining steps.

::: note
For further reference on SPM, check [its official documentation](https://developer.apple.com/documentation/xcode/adding_package_dependencies_to_your_app).
:::

## Adding Auth0 Credentials

You will need to add an `Auth0.plist` file, containing your Auth0 client id and domain, to your main bundle. Here is an example of the file contents:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
   <key>ClientId</key>
   <string>${account.clientId}</string>
   <key>Domain</key>
   <string>${account.namespace}</string>
</dict>
</plist>
```

### Web-based Auth (iOS / macOS 10.15+)

First go to [Auth0 Dashboard](${manage_url}/#/applications) and go to application's settings. Make sure you have in **Allowed <dfn data-key="callback">Callback URLs</dfn>** a URL with the following format:

```text
{YOUR_BUNDLE_IDENTIFIER}://${account.namespace}/ios/{YOUR_BUNDLE_IDENTIFIER}/callback
```

In your application's `Info.plist` file register your iOS Bundle Identifier as a custom scheme like this:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>None</string>
        <key>CFBundleURLName</key>
        <string>auth0</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>{YOUR_BUNDLE_IDENTIFIER}</string>
        </array>
    </dict>
</array>
```

::: note
If your `Info.plist` is not shown in this format, you can **Right Click** on `Info.plist` in Xcode and then select **Open As / Source Code**.
:::

::: note
Auth0.swift will only handle URLs with your Auth0 domain as host, for example `com.auth0.MyApp://samples.auth0.com/ios/com.auth0.MyApp/callback`
:::

Allow Auth0 to handle authentication callbacks. In your `AppDelegate.swift`, add the following:

##### iOS

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey: Any]) -> Bool {
    return Auth0.resumeAuth(url)
}
```

##### macOS

```swift
func application(_ application: NSApplication, open urls: [URL]) {
    Auth0.resumeAuth(urls)
}
```

#### Authenticate with Universal Login

The first step in adding authentication to your application is to provide a way for your users to log in. The fastest, most secure, and most feature-rich way to do this with Auth0 is to use <dfn data-key="universal-login">Universal Login</dfn>.

::: note
For more information on the two types of login flows, please refer to [Browser-Based vs. Native Login Flows on Mobile Devices](/design/browser-based-vs-native-experience-on-mobile)
:::

```swift
Auth0
    .webAuth()
    .audience("https://${account.namespace}/userinfo")
    .start { result in
        switch result { // Auth0.Result
        case .success(let credentials):
            print("Credentials: \(credentials)")
        case .failure(let error):
            print(error)
        }
    }
```

::: warning
If you're using **Swift 5+**, `Auth0.Result` may shadow Swift's `Result` type. To prevent that, replace it with `Swift.Result` whenever you want to refer to Swift's built-in type. This will be fixed in the next major version of Auth0.swift.
:::

::: note
To ensure a response that complies with <dfn data-key="openid">OpenID Connect (OIDC)</dfn>, you must either request an <dfn data-key="audience">`audience`</dfn> or enable the **OIDC Conformant** switch in your [Auth0 dashboard](${manage_url}), under **Application > Settings > Show Advanced Settings > OAuth**. For more information, refer to [How to use the new flows](/api-auth/tutorials/adoption#how-to-use-the-new-flows).
:::

#### Authenticate with a specific Auth0 connection

The `connection` option allows you to specify a connection that you wish to authenticate with. If no connection is specified here, the browser will show the login page, with all of the connections which are enabled for this application.

```swift
Auth0
    .webAuth()
    .connection("facebook")
    .audience("https://${account.namespace}/userinfo")
    .start { result in
        switch result {
        case .success(let credentials):
            print("Credentials: \(credentials)")
        case .failure(let error):
            print(error)
        }
    }
```

#### Authenticate using a specific scope

Using <dfn data-key="scope">scopes</dfn> can allow you to return specific claims for specific fields in your request. Adding parameters to `scope` will allow you to add more scopes. The default scope is `openid`, and you should read our [documentation on scopes](/scopes/current) for further details about them.

```swift
Auth0
    .webAuth()
    .scope("openid email")
    .connection("google-oauth2")
    .audience("https://${account.namespace}/userinfo")
    .start { result in
        switch result {
        case .success(let credentials):
            print("Credentials: \(credentials)")
        case .failure(let error):
            print(error)
        }
    }
```

### Getting user information

In order to retrieve a user's profile, you call the `userInfo` method and pass it the user's `accessToken`.  Although the call returns a [UserInfo](https://github.com/auth0/Auth0.swift/blob/master/Auth0/UserInfo.swift) instance, this is a basic OIDC conformant profile and the only guaranteed claim is the `sub`, which contains the user's ID. Depending on the requested scope, the claims returned may vary.  You can also use the `sub` value to call the [Management API](https://auth0.com/docs/api/management/v2#!/Users/get_users_by_id) and return a full user profile.

```swift
Auth0
   .authentication()
   .userInfo(withAccessToken: accessToken)
   .start { result in
       switch result {
       case .success(let profile):
           print("User Profile: \(profile)")
       case .failure(let error):
           print("Failed with \(error)")
       }
   }
```

## Next Steps

Take a look at the following resources to see how the Auth0.swift SDK can be customized for your needs:

::: next-steps
* [Auth0.Swift Database Authentication](/libraries/auth0-swift/database-authentication)
* [Auth0.Swift Passwordless Authentication](/libraries/auth0-swift/passwordless)
* [Auth0.Swift Refresh Tokens](/libraries/auth0-swift/save-and-refresh-jwt-tokens)
* [Auth0.Swift User Management](/libraries/auth0-swift/user-management)
* [Auth0.Swift Touch ID / Face ID Authentication](/libraries/auth0-swift/touchid-authentication)
:::
