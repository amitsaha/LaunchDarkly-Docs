---
title: ".NET SDK Reference"
excerpt: ""
---
This reference guide documents all of the methods available in our .NET SDK, and explains in detail how these methods work. If you want to dig even deeper, our SDKs are open source-- head to our [.NET SDK GitHub repository](https://github.com/launchdarkly/dotnet-server-sdk) to look under the hood. The online [API docs](https://launchdarkly.github.io/dotnet-server-sdk/) contain the programmatic definitions of every type and method. Additionally you can clone and run a [sample application](https://github.com/launchdarkly/hello-dotnet) using this SDK.

[block:callout]
{
  "type": "info",
  "title": "Requirements",
  "body": "The .NET SDK requires version 4.5 or later of the .NET framework or .NETCoreApp 1.0 or later."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Getting started"
}
[/block]
Building on top of our [Quickstart](doc:getting-started) guide, the following steps will get you started with using the LaunchDarkly SDK in your .NET application.

The first step is to install the LaunchDarkly SDK as a dependency in your application using your application's dependency manager. 
[block:code]
{
  "codes": [
    {
      "code": "Install-Package LaunchDarkly.ServerSdk\n\n# Note that in earlier versions, the package name was LaunchDarkly.Client",
      "language": "shell"
    }
  ]
}
[/block]
Next you should import the LaunchDarkly client in your application code.
[block:code]
{
  "codes": [
    {
      "code": "using LaunchDarkly.Client;\n\n// Note that this namespace is no longer the same as the package name",
      "language": "csharp"
    }
  ]
}
[/block]
Once the SDK is installed and imported, you'll want to create a single, shared instance of `LdClient`. You should specify your SDK key here so that your application will be authorized to connect to LaunchDarkly and for your application and environment.
[block:code]
{
  "codes": [
    {
      "code": "LdClient ldClient = new LdClient(\"YOUR_SDK_KEY\");",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "It's important to make this a singleton-- internally, the client instance maintains internal state that allows us to serve feature flags without making any remote requests. **Be sure that you're not instantiating a new client with every request.**",
  "title": "LDClient must be a singleton"
}
[/block]
Using `ldClient`, you can check which variation a particular user should receive for a given feature flag.
[block:code]
{
  "codes": [
    {
      "code": "User user = User.WithKey(username);\nbool showFeature = ldClient.BoolVariation(\"your.feature.key\", user, false);\nif (showFeature) {\n  // application code to show the feature \n}\nelse {\n  // the code to run if the feature is off\n}",
      "language": "csharp"
    }
  ]
}
[/block]
Lastly, when your application is about to terminate, shut down `ldClient`. This ensures that the client releases any resources it is using, and that any pending analytics events are delivered to LaunchDarkly. If your application quits without this shutdown step, you may not see your requests and users on the dashboard, because they are derived from analytics events. **This is something you only need to do once**.
[block:code]
{
  "codes": [
    {
      "code": "// shut down the client, since we're about to quit\nldClient.Dispose();",
      "language": "csharp"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Customizing your client"
}
[/block]
You can also pass custom parameters to the client by creating a custom configuration object:
[block:code]
{
  "codes": [
    {
      "code": "Configuration config = LaunchDarkly.Client.Configuration.Builder(\"YOUR_SDK_KEY\")\n          .EventFlushInterval(TimeSpan.FromSeconds(2))\n          .Build();\nLdClient ldClient = new LdClient(config);",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]
Here, we've customized the event flush interval. The complete list of customizable parameters is as follows:
[block:parameters]
{
  "data": {
    "h-0": "Builder Method",
    "h-1": "Parameter Type",
    "0-0": "`Uri`",
    "1-0": "`StreamUri`",
    "2-0": "`EventsUri`",
    "3-0": "`IsStreamingEnabled`",
    "4-0": "`EventCapacity`",
    "5-0": "`EventFlushInterval`",
    "6-0": "`PollingInterval`",
    "7-0": "`StartWaitTime`",
    "8-0": "`ReadTimeout`",
    "9-0": "`ReconnectTime`",
    "10-0": "`ConnectionTimeout`",
    "11-0": "`HttpMessageHandler`",
    "12-0": "`Offline`",
    "h-2": "Description",
    "h-3": "Default Value",
    "0-1": "`Uri` or `string`",
    "1-1": "`Uri` or `string`",
    "2-1": "`Uri` or `string`",
    "4-1": "`int`",
    "5-1": "`TimeSpan`",
    "6-1": "`TimeSpan`",
    "7-1": "`TimeSpan`",
    "8-1": "`TimeSpan`",
    "9-1": "`TimeSpan`",
    "10-1": "`TimeSpan`",
    "11-1": "`HttpMessageHandler`",
    "12-1": "`bool`",
    "3-1": "`bool`",
    "0-2": "Set the base URL of the LaunchDarkly server for this configuration",
    "4-2": "Set the capacity of the events buffer.",
    "10-2": "Set the connection timeout for the configuration.",
    "2-2": "Set the events URL of the LaunchDarkly server for this configuration.",
    "5-2": "Set the number of seconds between flushes of the event buffer.",
    "12-2": "Set whether this client is offline.",
    "6-2": "Set the polling interval (when streaming is disabled). Values less than the default of 30 seconds will be set to 30 seconds.",
    "3-2": "Boolean value which enables streaming.",
    "1-2": "LaunchDarkly stream URI to be used.",
    "7-2": "The timeout when reading data from the EventSource API.",
    "1-3": "https://stream.launchdarkly.com",
    "2-3": "https://events.launchdarkly.com",
    "0-3": "https://app.launchdarkly.com",
    "3-3": "`true`",
    "6-3": "30 seconds",
    "4-3": "10000 events",
    "5-3": "5 seconds",
    "7-3": "10 seconds",
    "8-3": "5 minutes",
    "9-3": "1 second",
    "10-3": "10 seconds",
    "11-3": "A default [HttpClientHandler](https://msdn.microsoft.com/en-us/library/system.net.http.httpclienthandler.aspx)",
    "12-3": "`false`",
    "8-2": "The timeout when reading data from the stream.",
    "9-2": "The stream connection timeout.",
    "11-2": "Sets the handler for http requests. Can be used to configure proxy authentication.",
    "13-0": "`AllAttributesPrivate`",
    "13-1": "`bool`",
    "13-2": "Whether all user attributes (except the user key) should be marked as Private user attributes, and not sent to LaunchDarkly.",
    "13-3": "`false`",
    "14-3": "No attributes are private by default.",
    "14-2": "Adds the name of a user attribute that should be marked as [private](doc:private-user-attributes) to a internally managed list. May be called multiple times to mark additional attributes as private.",
    "14-1": "`string`",
    "14-0": "`PrivateAttributeName`"
  },
  "cols": 4,
  "rows": 15
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "LaunchDarkly servers operate in a load-balancing framework which may cause their IP addresses to change. This could result in the SDK failing to connect to LaunchDarkly if an old IP address is still in your system's DNS cache.\n\nIn .NET, the DNS cache retains IP addresses for two minutes by default. If you are noticing intermittent connection failures that always resolve in two minutes, you may wish to change this setting to a lower value as described [here](https://docs.microsoft.com/en-us/dotnet/api/system.net.servicepointmanager.dnsrefreshtimeout?view=netframework-4.7.2).",
  "title": "DNS caching issues"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Users"
}
[/block]
Feature flag targeting and rollouts are all determined by the *user* you pass to your `variation` calls. In our .NET SDK, the `User` class has a `WithKey` method for creating a simple user with only a key, and a `Builder` method for building a user with other properties. Here's an example:
[block:code]
{
  "codes": [
    {
      "code": " LDUser user = User.Builder(\"aa0ceb\")\n      .FirstName(\"Ernestina\")\n      .LastName(\"Evans\")\n      .Email(\"ernestina@example.com\")\n      .Custom(\"groups\", new List<String>(){\"Google\", \"Microsoft\"})\n      .Build();\n   ",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]
Let's walk through this snippet. The argument to `Builder` is the user's key-- in this case we've used the hash `"aa0ceb"`. **The user key is the only mandatory user attribute**. The key should also uniquely identify each user. You can use a primary key, an e-mail address, or a hash, as long as the same user always has the same key. We recommend using a hash if possible.

All of the other attributes (set via calls to `FirstName`, `LastName`, `Email`, and `Custom` attributes) are optional. The attributes you specify will automatically appear on our dashboard, meaning that you can start segmenting and targeting users with these attributes. 

In addition to the built-in attributes defined in the `User` class, you can pass us any of your own user data by passing `custom` attributes, like the `groups` attribute in the example above. 
[block:callout]
{
  "type": "info",
  "title": "A note on types",
  "body": "Most of our built-in attributes (like names and e-mail addresses) expect string values. Custom attributes values can be strings, booleans (like true or false), numbers, or lists of strings, booleans or numbers. \n\nIf you enter a custom value on our dashboard that looks like a number or a boolean, it'll be interpreted that way. The .NET SDK is strongly typed, so be aware of this distinction."
}
[/block]
Custom attributes are one of the most powerful features of LaunchDarkly. They let you target users according to any data that you want to send to us-- organizations, groups, account plans-- anything you pass to us becomes available instantly on our dashboard.
[block:api-header]
{
  "title": "Private user attributes"
}
[/block]
You can optionally configure the .NET SDK to treat some or all user attributes as [private user attributes](doc:private-user-attributes) . Private user attributes can be used for targeting purposes, but are removed from the user data sent back to LaunchDarkly.

In the .NET SDK there are two ways to define private attributes for the entire LaunchDarkly client:

* When creating the LaunchDarkly `Configuration` object, you can call the `AllAttributesPrivate` method, which takes in a boolean parameter. If `true`, all user attributes (except the key) for *all users* are removed before the user is sent to LaunchDarkly.
* When creating the LaunchDarkly `Configuration` object, you can call the `PrivateAttributeName` method, which takes in an attribute name (string) as a parameter and adds it to an internally managed list of private attributes. This method may be called multiple times to mark additional attributes as private. If any user has a custom or built-in attribute named in the private attributes list, it will be removed before the user is sent to LaunchDarkly.

You can also mark attributes as private when building the user object itself by calling `AsPrivateAttribute()` after setting the attribute on the user builder. For example:
[block:code]
{
  "codes": [
    {
      "code": "var user = User.Builder(\"aa0ceb\")\n  .Email(\"test@example.com\").AsPrivateAttribute()\n  .Build();",
      "language": "csharp"
    }
  ]
}
[/block]
When this user is sent back to LaunchDarkly, the `email` attribute will be omitted.
[block:api-header]
{
  "type": "basic",
  "title": "Anonymous users"
}
[/block]
You can also distinguish logged-in users from anonymous users in the SDK, as follows:
[block:code]
{
  "codes": [
    {
      "code": " LDUser user = User.Builder(\"aa0ceb\")\n      .Anonymous(true)\n      .Build();\n   ",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]
You will still need to generate a unique key for anonymous users-- session IDs or UUIDs work best for this. 

Anonymous users work just like regular users, except that they won't appear on your Users page in LaunchDarkly. You also can't search for anonymous users on your Features page, and you can't search or autocomplete by anonymous user keys. This is actually a good thing-- it keeps anonymous users from polluting your Users page!
[block:api-header]
{
  "type": "basic",
  "title": "Variation"
}
[/block]
The `Variation` method determines whether a flag is enabled or not for a specific user.   In .NET, there is  a `variation` method for each type (e.g. `BoolVariation`, `StringVariation`):
[block:code]
{
  "codes": [
    {
      "code": "var value = ldClient.BoolVariation(\"your.feature.key\", user, false);\n",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]
`variation` calls take the feature flag key, an `LDUser`, and a default value. 

The default value will only be returned if an error is encountered-- for example, if the feature flag key doesn't exist or the user doesn't have a key specified. 

The `variation` call will automatically create a user in LaunchDarkly if a user with that user key doesn't exist already. There's no need to create users ahead of time (but if you do need to, take a look at Identify). 
[block:api-header]
{
  "type": "basic",
  "title": "Track"
}
[/block]
The `Track` method allows you to record actions your users take on your site. This lets you record events that take place on your server. In LaunchDarkly, you can tie these events to goals in A/B tests. Here's a simple example:
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Track(\"your-goal-key\", user);",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]
You can also attach custom data to your event by passing an extra parameter to `Track`, using the `LdValue` type which can contain any kind of data supported by JSON: 
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Track(\"Completed purchase\", user, LdValue.Of(\"sku132\"));\n",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Identify"
}
[/block]
`Identify` creates or updates users on LaunchDarkly, making them available for targeting and autocomplete on the dashboard. In most cases, you won't need to call `Identify`-- the `Variation` call will automatically create users on the dashboard for you. `Identify` can be useful if you want to pre-populate your dashboard before launching any features. 
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Identify(user);",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "All Flags"
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Note that unlike `Variation` and `Identify` calls, `AllFlagsState` does not send events to LaunchDarkly. Thus, users are not created or updated in the LaunchDarkly dashboard."
}
[/block]
The `AllFlagsState` method captures the state of all feature flag keys with regard to a specific user. This includes their values, as well as other metadata.

This method can be useful for passing feature flags to your front-end. In particular, it can be used to provide bootstrap flag settings for our [JavaScript SDK](doc:js-sdk-reference).
[block:code]
{
  "codes": [
    {
      "code": "var state = ldClient.AllFlagsState(user);",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Fallback value and offline mode"
}
[/block]
The default (fallback) values are defined in your code. The default value will only be returned if an error is encountered or if LaunchDarkly is unreachable -- for example, if the feature flag key doesn't exist or the user doesn't have a key specified.

In some situations, you might want avoid remote calls to LaunchDarkly and fall back to default values for your feature flags. For example, if your software is both cloud-hosted and distributed to customers to run on premise, it might make sense to fall back to defaults when running on premise. You can do this by setting offline mode in the client's Config. When the client is in offline mode, no network requests will be made, so it is suitable for unit-testing.
[block:code]
{
  "codes": [
    {
      "code": "Configuration config = Configuration.Builder(\"SDK_KEY\")\n        .Offline(true)\n        .Build();\nLdClient client = new LdClient(config);\n",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Secure mode hash"
}
[/block]
The `SecureModeHash` method computes an HMAC signature of a user signed with the client's SDK key. If you're using our [JavaScript SDK](doc:js-sdk-reference) for client-side flags, this method generates the signature you need for secure mode.
[block:code]
{
  "codes": [
    {
      "code": "var hash = ldClient.SecureModeHash(user);",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Flush"
}
[/block]
Internally, the LaunchDarkly SDK keeps an event buffer for `Track` and `Identify` calls. These are flushed periodically in a background thread. In some situations (for example, if you're testing out the SDK in a REPL), you may want to manually call `Flush` to process events immediately. 
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Flush();",
      "language": "csharp",
      "name": ".NET"
    }
  ]
}
[/block]
Note that the flush interval is configurable-- if you need to change the interval, you can do so via the `Configuration` class.
[block:api-header]
{
  "type": "basic",
  "title": "Dispose"
}
[/block]
Dispose safely shuts down the client instance and releases all resources associated with the client. In most long-running applications, you should not have to call dispose.
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Dispose();",
      "language": "csharp"
    }
  ]
}
[/block]

[block:api-header]
{
  "title": "Logging"
}
[/block]
The .NET SDK uses the `Common.Logging` framework. For an example configuration check out the [Common.Logging readme](https://github.com/net-commons/common-logging#2-register-and-configure-commonlogging).

Be aware of two considerations when enabling the DEBUG log level:

 1. Debug-level logs can be very verbose. It is not recommended that you turn on debug logging in high-volume environments.
 2. Potentially sensitive information is logged including LaunchDarkly users created by you in your usage of this SDK.
[block:api-header]
{
  "title": "Database integrations"
}
[/block]
Separate packages allow feature flag data to be cached with [Redis](https://github.com/launchdarkly/dotnet-server-sdk-redis), [DynamoDB](https://github.com/launchdarkly/dotnet-server-sdk-dynamodb), or [Consul](https://github.com/launchdarkly/dotnet-server-sdk-consul).

See ["Using a persistent feature store"](doc:using-a-persistent-feature-store) for more information.