---

 

copyright:

  years: 2016

 

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Using the {{site.data.keyword.openwhisk_short}} mobile SDK
{: #openwhisk_mobile_sdk}
*Last updated: 22 February 2016*

{{site.data.keyword.openwhisk}} provides a mobile SDK for iOS and watchOS 2 devices that enables mobile apps to easily fire remote triggers and invoke remote actions. A version for Android is currently not available; Android developers can use the {{site.data.keyword.openwhisk}} REST API directly.
{: shortdesc}

The mobile SDK is written in Swift 2.0 and supports iOS 9 and later releases.

## Adding the SDK to your app
{: #openwhisk_add_sdk}
You can install the mobile SDK by using CocoaPods, or from the source directory.

### Installing by using CocoaPods 

The {{site.data.keyword.openwhisk_short}} SDK for mobile is available for public distribution through CocoaPods. The following lines in a Podfile install the SDK for an iOS app with a watchOS 2 extension:

```
source 'https://github.com/openwhisk/openwhisk-podspecs.git'

use_frameworks!

target 'MyApp' do
     platform :ios, '9.0'
     pod 'OpenWhisk'
end

target 'MyApp WatchKit Extension' do
     platform :watchos, '2.0'
     pod 'OpenWhisk-Watch'
end
```
{: codeblock}

### Installing from source code

Source code is available at https://github.com/openwhisk/openwhisk. The SDK is in mobile/iOS/SDK.

## Installing the starter app example
{: #openwhisk_install_sdkstart}

You can use the {{site.data.keyword.openwhisk_short}} CLI to download example code that embeds the {{site.data.keyword.openwhisk_short}} SDK framework.  

To install the starter app example, enter the following command:
```
wsk sdk install ios
```
{: pre}

## Getting started with the SDK
{: #openwhisk_sdk_getstart}

To get up and running quickly, create a WhiskCredentials object with your {{site.data.keyword.openwhisk_short}} API credentials and create an {{site.data.keyword.openwhisk_short}} instance from that.

For example, in Swift 2.1, use the following example code to create a credentials object:

```
let credentialsConfiguration = WhiskCredentials(accessKey: "myKey", accessToken: "myToken")

let whisk = Whisk(credentials: credentialsConfiguration!)
```
{: codeblock}

In previous example, you pass in the `myKey` and `myToken` you get from {{site.data.keyword.openwhisk_short}}.

## Invoking an {{site.data.keyword.openwhisk_short}} action
{: #openwhisk_sdk_invoke}


To invoke a remote action, you can call `invokeAction` with the action name. You can specify the namespace the action belongs to, or just leave it blank to accept the default namespace.  Use a dictionary to pass parameters to the action as required.

For example:

```
// In this example, we are invoking an action to print a message to the OpenWhisk Console
var params = Dictionary<String, String>()
params["payload"] = "Hi from mobile"

do {
    try whisk.invokeAction(name: "helloConsole", package: "mypackage", namespace: "mynamespace", parameters: params, hasResult: false, callback: {(reply, error) -> Void in
        if let error = error {
            //do something
            print("Error invoking action \(error.localizedDescription)")
        } else {
            print("Action invoked!")
        }

    })
} catch {
    print("Error \(error)")
}
```
{: codeblock}

In the previous example, you invoke the `helloConsole` action by using the default namespace.

## Firing an {{site.data.keyword.openwhisk_short}} trigger
{: #openwhisk_sdk_fire}

To fire a remote trigger, you can call the `fireTrigger` method. Pass in parameters as required by using a dictionary.

```
// In this example we are firing a trigger when our location has changed by a certain amount

var locationParams = Dictionary<String, String>()
locationParams["payload"] = "{\"lat\":41.27093, \"lon\":-73.77763}"

do {
    try whisk.fireTrigger(name: "locationChanged", package: "mypackage", namespace: "mynamespace", parameters: locationParams, callback: {(reply, error) -> Void in

        if let error = error {
            print("Error firing trigger \(error.localizedDescription)")
        } else {
            print("Trigger fired!")
        }
    })
} catch {
    print("Error \(error)")
}
```
{: codeblock}

In the previous example, you are firing a trigger called `locationChanged`.

## Using actions that return a result
{: #openwhisk_sdk_actionresult}

If the action returns a result, set hasResult to true in the invokeAction call. The result of the action is returned in the reply dictionary, for example:

```
do {
    try whisk.invokeAction(name: "actionWithResult", package: "mypackage", namespace: "mynamespace", parameters: params, hasResult: true, callback: {(reply, error) -> Void in

        if let error = error {
            //do something
            print("Error invoking action \(error.localizedDescription)")

        } else {
            var result = reply["result"]
            print("Got result \(result)")
        }


    })
} catch {
    print("Error \(error)")
}
```
{: codeblock}

By default, the SDK returns only the activation ID and any result produced by the invoked action. To get metadata of the entire response object, which includes the HTTP response status code, use the following setting:

```
whisk.verboseReplies = true
```
{: codeblock}

## Configuring the SDK
{: #openwhisk_sdk_configure}

You can configure the SDK to work with different installations of {{site.data.keyword.openwhisk_short}} by using the baseURL parameter. For instance:

```
whisk.baseURL = "http://localhost:8080"
```
{: codeblock}

In this example, you use an installation running at localhost:8080.  If you do not specify the baseUrl, the mobile SDK uses the instance running at https://openwhisk.ng.bluemix.net.

You can pass in a custom NSURLSession in case you require special network handling. For example, you might have your own {{site.data.keyword.openwhisk_short}} installation that uses self-signed certificates:

```
// create a network delegate that trusts everything
class NetworkUtilsDelegate: NSObject, NSURLSessionDelegate {
    func URLSession(session: NSURLSession, didReceiveChallenge challenge: NSURLAuthenticationChallenge, completionHandler: (NSURLSessionAuthChallengeDisposition, NSURLCredential?) -> Void) {
        completionHandler(NSURLSessionAuthChallengeDisposition.UseCredential, NSURLCredential(forTrust: challenge.protectionSpace.serverTrust!))
    }
}

// create an NSURLSession that uses the trusting delegate
let session = NSURLSession(configuration: NSURLSessionConfiguration.defaultSessionConfiguration(), delegate: NetworkUtilsDelegate(), delegateQueue:NSOperationQueue.mainQueue())

// set the SDK to use this urlSession instead of the default shared one
whisk.urlSession = session
```
{: codeblock}

### Support for qualified names

All actions and triggers have a fully qualified name that is made up of a namespace, a package, and an action or trigger name. The SDK can accept these as parameters when invoking an action or firing a trigger. The SDK also provides a function that accepts a fully qualified name that looks like `/mynamespace/mypackage/nameOfActionOrTrigger`. The qualified name string supports unnamed default values for namespaces and packages that all {{site.data.keyword.openwhisk_short}} users have, so the following parsing rules apply:

- qName = "foo" results in namespace = default, package = default, action/trrigger = "foo"
- qName = "mypackage/foo" results in namespace = default, package = mypackage, action/trigger = "foo"
- qName = "/mynamespace/foo" results in namespace = mynamespace, package = default, action/trigger = "foo"
- qName = "/mynamespace/mypackage/foo results in namespace = mynamespace, package = mypackage, action/trigger = "foo"

All other combinations issue a WhiskError.QualifiedName error. Therefore, when using qualified names, you must wrap the call in a "`do/try/catch`" construct.

### SDK button

For convenience, the SDK includes a `WhiskButton`, which extends the `UIButton` to allow it to invoke actions.  To use the `WhiskButton`, follow this example:

```
var whiskButton = WhiskButton(frame: CGRectMake(0,0,20,20))

whiskButton.setupWhiskAction("helloConsole", package: "mypackage", namespace: "_", credentials: credentialsConfiguration!, hasResult: false, parameters: nil, urlSession: nil)

let myParams = ["name":"value"]

// Call this when you detect a press event, e.g. in an IBAction, to invoke the action
whiskButton.invokeAction(parameters: myParams, callback: { reply, error in
    if let error = error {
        print("Oh no, error: \(error)")
    } else {
        print("Success: \(reply)")
    }
})

// or alternatively you can set up a "self contained" button that listens for press events on itself and invokes an action

var whiskButtonSelfContained = WhiskButton(frame: CGRectMake(0,0,20,20))
whiskButtonSelfContained.listenForPressEvents = true
do {

   // use qualified name API which requires do/try/catch
   try whiskButtonSelfContained.setupWhiskAction("mypackage/helloConsole", credentials: credentialsConfiguration!, hasResult: false, parameters: nil, urlSession: nil)
   whiskButtonSelfContained.actionButtonCallback = { reply, error in

       if let error = error {
           print("Oh no, error: \(error)")
       } else {
           print("Success: \(reply)")
       }
   }
} catch {
   print("Error setting up button \(error)")
}
```
{: codeblock}
