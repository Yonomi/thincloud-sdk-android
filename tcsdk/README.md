# Thincloud SDK

##### Table of Contents
1. [Getting Started](#getting-started)
2. [Components](#components)
3. [Javadocs](#javadocs)



## Getting Started

This SDK is used for interacting with Thincloud v1 as well as providing a Virtual Gateway implementation.

Setting up your IDE:

1. Download [Android Studio][androidStudio]
2. [Install Lombok Plugin][lombokForJetbrains]
3. [Enable Annotation Processing][annotationProcessing]
4. Open Project


## Components


There are four primary components to be concerned with:

1. [Firebase Setup](#firebase-setup)
2. [ThincloudConfig](#thincloudconfig)
3. [ThincloudSDK](#thincloudsdk)
4. [CommandQueue](#commandqueue)
5. [ThincloudAPI](#thincloudapi)


### Firebase Setup

In order to make use of Firebase Cloud Messaging, a `google-services.json` file must be placed in the root directory of your project. For example with the demo app provided it would be located in `./app/google-services.json`.

Instructions to acquire the `google-services.json` file can be [found here](https://support.google.com/firebase/answer/7015592?hl=en).


### ThincloudConfig

A configuration POJO including the following properties. Uses [Lombok][lombokLib] with `@Data` and `@Accessors(fluent = true)` for interacting with POJO. See [@Data][lombokData] and [@Accessors][lombokAccessor] for more information.


|	Type	|	Variable	|	Usage	| Required |
| ------- | ---------- | ------- | ------- |
| String | instanceName | Instance name, used for API URL generation | Yes |
| String | appName | Name of the implementing application | Yes |
| String | appVersion | Version of the implementing application | Yes | 
| String | apiKey | Thincloud API Key | Yes |
| String | fcmTopic | Firebase Topic | Yes |
| String | clientId | oAuth Client ID | Yes |
| boolean | useJobScheduler | Default `false`, can be turned on to have Android manage background jobs | No |
| Set\<String\> | commandsToIgnore | Ignores commands by name when pulling down the command list for a device. | No |


### ThincloudSDK

Primary singleton interface for dealing with Thincloud. Ideally, this is all that is needed for a basic Virtual Gateway implementation.


| Return Type | Method | Parameters | Usage |
| ---- | ---- | ----- | ---- |
| ThincloudSDK | *initialize* | Context, ThincloudConfig | Initializes the ThincloudSDK and ThincloudAPI. <br>*Requires Android Context to configure Firebase.* |
| void | *setHandler* | CommandHandler | Provides an event handler to be called for each command that is sent. |
| boolean | *isInitialized* |  | Determines if the SDK has been initialized yet. |
| void | login | String username, String password, TCFuture\<Boolean\> callback | Attempt to login using username and password. |
| void | logout | | Attempts to remove previously registered client and invalidate cache. |


### CommandQueue

The CommandQueue implementation is used to coordinate for the Virtual Gateway. It is implemented as a singleton with it's instance methods listed below.

| Method | Arguments | Usage |
| --- | --- | --- |
| setHandler | GenericCommandHandler | Assigns a handler. Can also be accessed statically via ThincloudSDK.setHandler |

#### GenericCommandHandler\<T>

The interface for handlers to implement. Includes a method `onEventReceived(T)`

#### CommandHandler

A GenericCommandHandler implementation to process a single command.

`onEventReceived(Command)`

#### CommandListHandler

A GenericCommandHandler implementation to process a list of commands.

`onEventReceived(List<Command>)`


### ThincloudAPI

An API initializer and manager singleton. Receives a configuration object when the SDK is initialized. Uses [Retrofit][retrofit] behind the scenes for easy API management. Wraps all authentication handling to make interacting with the API easy. When run in an Android App, all API interaction should be called asynchronously to prevent network on the main thread.

| Return Type | Method | Arguments | Usage |
| ---- | ---- | ---- | ---- |
| void | login | String username, String password, TCFuture\<Boolean\> callback | Attempts to login and register a client. |
| void | logout | | Attempts to remove previously registered client and invalidate cache. |
| APISpec | getSpec | | Grabs the API spec. May not be initialized. |
| String | getClientId | | Grabs the cached clientId. |
| boolean| hasRefreshToken | | Checks to see if API has a refresh token set. |


#### ThincloudRequest\<T>

An API wrapper that passes all results to a single handler.

| Method | Arguments | Usage |
| ---- | ---- | --- |
| new ThincloudRequest\<T> | | Initialize a new ThincloudRequest object |
| create | Call\<T>, ThincloudResponse\<T> | Create a new request using with stored tokens. |
| createWithoutAuth | Call\<T>, ThincloudResponse\<T> | Create a new request without attempting to authenticate first. |

#### ThincloudResponse\<T>

An abstract API response handler class with a single handler method.

| Method | Arguments | Usage |
| ---- | ---- | --- |
| new ThincloudResponse\<T> | | Initialize a new ThincloudResponse object |
| handle | Call\<T>, Response\<T>, Throwable | The all purpose response handler |


#### API Interaction Sample

```java
APISpec apiSpec = ThincloudAPI.getInstance().getSpec();
if(apiSpec != null){
    ThincloudResponse<User> handler = new ThincloudResponse<User>() {
        @Override
        public void handle(Call<User> call, Response<User> response, Throwable error) {
            if(error != null){
            	//Failed to get user, an error was thrown
            } else {
                if(response.code() >= 400){
                    //Failed to get user, bad response
                } else {
                    //Got user object, response.body()
                }
            }
        }
    };
    new ThincloudRequest<User>().create(apiSpec.getSelf(), handler);
} else {
    //Failed due to unavailable APISpec
}
```

## Javadocs

Javadocs can be found [here](../docs/).




[javadocs]: ./javadocs
[androidStudio]: https://developer.android.com/studio/index.html
[lombokForJetbrains]: https://plugins.jetbrains.com/plugin/6317-lombok-plugin
[annotationProcessing]: https://www.jetbrains.com/help/idea/compiler-annotation-processors.html
[lombokLib]: https://projectlombok.org
[lombokAccessor]: https://projectlombok.org/features/experimental/Accessors
[lombokData]: https://projectlombok.org/features/Data
[retrofit]: http://square.github.io/retrofit/