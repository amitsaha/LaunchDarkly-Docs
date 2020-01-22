---
title: "Electron SDK Reference"
excerpt: ""
---
This reference guide documents the LaunchDarkly SDK for the [Electron](https://electronjs.org/) desktop application framework. This is a variant of the [client-side JavaScript SDK](./js-sdk-reference) with additional functionality for Electron.

If you want to dig even deeper, our SDKs are open source-- head to our [Electron SDK GitHub repository](https://github.com/launchdarkly/electron-client-sdk) to look under the hood. It also makes use of code from the [JavaScript SDK repository](https://github.com/launchdarkly/js-client-sdk). The [online API docs](https://launchdarkly.github.io/electron-client-sdk/) contain the programmatic definitions of every type and method. Additionally you can clone and run a [sample application](https://github.com/launchdarkly/hello-electron) using this SDK.
## Why use this instead of the Node SDK?
Since Electron is based on Node.js, it is possible to run the [LaunchDarkly Node SDK](./node-sdk-reference) in it. However, this is strongly discouraged, as the Node SDK is meant for server-side use—not for applications that are distributed to users. There are several reasons why this distinction matters:

* The server-side SDKs include an SDK key that can download the entire definition (including rollout rules and individual user targets) of all of your feature flags. If you embed this SDK key in an application, any user who looks inside the application can then access all of your feature flag definitions—which may include sensitive data such as other users' email addresses. The client-side and mobile SDKs use different credentials that do not allow this.
* The server-side SDKs do in fact download your entire flag data using this key, since they have to be able to evaluate flags quickly for any user. That can be quite a large amount of data. The client-side and mobile SDKs, which normally evaluate flags for just one user at a time, use a much more efficient protocol where they request only the active variation for each flag for that specific user.
* The Electron SDK also includes features that are specific to Electron, such as the ability to access main-process flags from the front end as described below.
## Known issues in this release
* If you are using the normal pattern of configuring your LaunchDarkly client in the main process, and then using `initializeInRenderer()` to get a mirror of the client in a renderer process, the client instance in the renderer process will not allow you to call `identify()` to change the current user. The current user can only be set in the main process. This may or may not change in future versions.
* When the main process receives new feature flag values from LaunchDarkly, they should be sent to the renderer processes automatically; however, there was one report of this not working, i.e. the renderer process continued to have the old flag values. Please let us know if you see any such problems.
## Getting started
Building on top of our [Quickstart](./getting-started) guide, the following steps will get you started with using the LaunchDarkly SDK in your Electron code.

You can install the SDK into your Electron project using `npm`:
[block:code]
{
  "codes": [
    {
      "code": "npm install --save launchdarkly-electron-client-sdk
# Note, in earlier versions, the package name was ldclient-electron",
      "language": "shell",
      "name": "Installing with npm"
    }
  ]
}
[/block]

## Initializing the client
Every Electron application consists of a _main process_, which is essentially a Node.js application, and some number of _renderer processes_, each of which is a Chromium web browser with its own window. These processes have their own independent JavaScript engines and data spaces, although there are ways to communicate between them.

The LaunchDarkly Electron SDK is designed to make it easy to use LaunchDarkly feature flags from within any of these processes. In the normal use case, there is an SDK client running in the main process; the renderer processes can then create client instances that are in effect mirrors of the main one.

To set up the main process client, you need the client-side ID for your LaunchDarkly environment, an object containing user properties (although you can change the user later), and optional configuration properties.
[block:code]
{
  "codes": [
    {
      "code": "const LDElectron = require('launchdarkly-electron-client-sdk');
const user = { key: 'example' };\nconst options = {};\nconst client = LDElectron.initializeInMain('YOUR_CLIENT_SIDE_ID', user, options);",
      "language": "javascript"
    }
  ]
}
[/block]
In a renderer process, to create a client object that uses the same feature flag data, you only need to do this:
[block:code]
{
  "codes": [
    {
      "code": "const LDElectron = require('launchdarkly-electron-client-sdk');
const client = LDElectron.initializeInRenderer();",
      "language": "javascript"
    }
  ]
}
[/block]
This gives you an object with the same interface—so you can evaluate feature flags, listen for flag change events, etc., in exactly the same way in the main process and the renderer process. However, only the main-process client is actually communicating with LaunchDarkly; the renderer-process clients are just delegating to the main one. This means that the overhead per application window is minimal (although it is still a good idea to retain a single client instance per window, rather than creating them ad-hoc when you need to evaluate a flag).

Both types of client are initialized asynchronously, so if you want to determine when the client is ready to evaluate feature flags, use the `ready` event or `waitForInitialization()`:
[block:code]
{
  "codes": [
    {
      "code": "// using an event listener:\nclient.on('ready', function() {\n  // now we can evaluate some feature flags\n});
// or, using a Promise:\nclient.waitForInitialization().then(function() {\n  // now we can evaluate some feature flags\n});",
      "language": "javascript"
    }
  ]
}
[/block]
If you try to evaluate feature flags before the client is ready, it will behave as it would if no flags existed (i.e. `variation` will return a default value).
## Bootstrapping
The `bootstrap` property in the client options allows you to speed up the startup process by providing an initial set of flag values.

If you set `bootstrap` to an object, the client will treat it as a map of flag keys to flag values. These values can be whatever you want (although in the browser environment, there is a [special mechanism](./js-sdk-reference#bootstrapping) that is commonly used). The client will immediately start out in a ready state using these values. It will still make an initial request to LaunchDarkly to get the actual latest values, but that will happen in the background.

If you set `bootstrap` to the string `"localstorage"`, the client will try to get flag values from persistent storage, using a unique key that is based on the user properties. In Electron, persistent storage consists of files in the [`userData`](https://electronjs.org/docs/all?query=getpath#appgetpathname) directory. If the client finds flag values stored for this user, it uses them and starts up immediately in a ready state—but also makes a background request to LaunchDarkly to get the latest values, and stores them as soon as it receives them.
## Feature flags
To evaluate any feature flag for the current user, call `variation`:
[block:code]
{
  "codes": [
    {
      "code": "var showFeature = client.variation(\"YOUR_FEATURE_KEY\", false);
if (showFeature)  {\n  // feature flag is  on\n} else {\n  // feature flag is off\n}",
      "language": "javascript"
    }
  ]
}
[/block]
The return value of `variation` will always be either one of the variations you defined for your flag in the LaunchDarkly dashboard, or the default value. The default value is the second parameter to `variation` (in this case `false`) and it is what the client will use if it's not possible to evaluate the flag (for instance, if the flag key does not exist, or if something about the definition of the flag is invalid).

You can also fetch all flags for the current user:
[block:code]
{
  "codes": [
    {
      "code": "var flags = client.allFlags();\nvar showFeature = flags['YOUR_FEATURE_KEY'];",
      "language": "javascript"
    }
  ]
}
[/block]
This returns a key-value map of all your feature flags. It will contain `null` values for any flags that could not be evaluated.

Note that both of these methods are synchronous. The client always has the last known flag values in memory, so retrieving them does not involve any I/O.
## Changing users
The `identify()` method tells the client to change the current user, and obtain the feature flag values for the new user. For example, on a sign-in page in a single-page app, you may initialize the client with an anonymous user; when the user logs in, you'd want the feature flag settings for the authenticated user. 

If you provide a callback function, it will be called (with a map of flag keys and values) once the flag values for the new user are available; after that point, `variation()` will be using the new values. You can also use a Promise for the same purpose.
[block:code]
{
  "codes": [
    {
      "code": "var newUser = { key: 'someone-else', name: 'John' };
client.identify(newUser, function(newFlags) {\n  console.log('value of flag for this user is: ' + newFlags[\"YOUR_FEATURE_KEY\"]);\n  console.log('this should be the same: ' + client.variation(\"YOUR_FEATURE_KEY\"));\n});
// or:\nclient.identify(newUser).then(function(newFlags) {\n  // as above\n});",
      "language": "javascript"
    }
  ]
}
[/block]
Note that the client always has _one_ current user. The client-side SDKs are not designed for evaluating flags for different users at the same time.
## Analytics events
Evaluating flags, either with `variation()` or with `allFlags()`, produces analytics events which you can observe on your LaunchDarkly Debugger page. Specifying a user with `identify()` (and also the initial user specified in the client constructor) also produces an analytics event, which is how LaunchDarkly receives your user data.

You can also explicitly send an event with any data you like using the `track` function:
[block:code]
{
  "codes": [
    {
      "code": "client.track('my-custom-event-key', { customProperty: someValue });",
      "language": "javascript"
    }
  ]
}
[/block]
You can completely disable event sending by setting `sendEvents` to `false` in the client options, but be aware that this means you will not have user data on your LaunchDarkly dashboard.
## Receiving live updates
By default, the client requests feature flag values only once per user (i.e. once at startup time, and then each time you call `identify()`). You can also use a persistent connection to receive flag updates whenever they occur.

Setting `streaming` to `true` in the client options, or calling `client.setStreaming(true)`, turns on this behavior. LaunchDarkly will push new values to the SDK, which will update the current feature flag state in the background, ensuring that `variation()` will always return the latest values.

If you want to be notified when a flag has changed, you can use an event listener for a specific flag:
[block:code]
{
  "codes": [
    {
      "code": "client.on('change:YOUR_FEATURE_KEY', function(newValue, oldValue) {\n  console.log('The flag was ' + oldValue + ' and now it is ' + newValue);\n});",
      "language": "javascript"
    }
  ]
}
[/block]
Or, you can listen for all feature flag changes:
[block:code]
{
  "codes": [
    {
      "code": "client.on('change', function(allFlagChanges)) {\n  Object.keys(allFlagChanges).forEach(function(key) {\n    console.log('Flag ' + key + ' is now ' + allFlagChanges[key]);\n  });\n});",
      "language": "javascript"
    }
  ]
}
[/block]
Subscribing to `change` events will automatically turn on streaming mode too, unless you have explicitly set `streaming` to `false`.
## Logging
By default, the SDK uses the [`winston`](https://www.npmjs.com/package/winston) package. There are four logging levels: `debug`, `info`, `warn`, and `error`; by default, `debug` and `info` messages are hidden.

To change the logging configuration, you can set `LDOptions.logger` to either another Winston instance or any object that implements the `LDLogger` interface. The `createConsoleLogger` function is a simple way to create a minimal logger.
## Server-side Node SDK compatibility
For developers who were using the server-side Node.js in Electron before the Electron SDK was available, there are differences between the APIs that can be inconvenient. For instance, in the server-side Node SDK, `variation()` is an asynchronous call that takes a callback, whereas in the client-side SDKs it is synchronous.

To make this transition easier, the LaunchDarkly Electron SDK provides an optional wrapper that emulates the Node SDK. When creating the main-process client, after calling `initializeInMain`, pass the client object to `createNodeSdkAdapter`. The resulting object will use the Node-style API.
[block:code]
{
  "codes": [
    {
      "code": "var realClient = LDElectron.initializeInMain('YOUR_CLIENT_SIDE_ID', user, options);
var wrappedClient = LDElectron.createNodeSdkAdapter(realClient);
wrappedClient.waitForInitialization().then(function() {\n    wrappedClient.variation(flagKey, user, defaultValue, function(err, result) {\n        console.log('flag value is ' + result);\n    });\n});\n",
      "language": "javascript"
    }
  ]
}
[/block]
Keep in mind that the underlying implementation is still the client-side SDK, which has a single-current-user model. Therefore, when you call `client.variation(flagKey, user, defaultValue)` it is really calling `client.identify(user)` first, obtaining flag values for that user, and then evaluating the flag. This will perform poorly if you attempt to evaluate flags for a variety of different users in rapid succession.