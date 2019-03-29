
# SendBird VideoChat for Android

[![Platform](https://img.shields.io/badge/platform-android-orange.svg)](https://github.com/smilefam/sendbird-videochat-android)
[![Languages](https://img.shields.io/badge/language-java-orange.svg)](https://github.com/smilefam/sendbird-videochat-android)
[![Maven](https://img.shields.io/badge/maven-v0.9.0-green.svg)](https://github.com/smilefam/sendbird-videochat-android/tree/master/com/sendbird/sdk/sendbird-videochat/0.9.0)
[![Commercial License](https://img.shields.io/badge/license-Commercial-brightgreen.svg)](https://github.com/smilefam/sendbird-videochat-android/blob/master/LICENSE.md)

SendBird `VideoChat` is an add-on to your application that enables users to make video and audio calls. SendBird `VideoChat` is available through [WebRTC](https://webrtc.org/). 

Note: This is a beta version and is not yet open to all users. If you would like to try SendBird `VideoChat`, contact our [sales support](https://help.sendbird.com/hc/en-us/requests/new) for more information.

## Contents

- [How to set and initialize `VideoChat`](#how-to-set-and-initialize-videochat)
    - [build.gradle](#build.gradle)
    - [AndroidManifest.xml](#androidmanifest.xml)
    - [`onCreate()` in the application](#onCreate()-in-the-application)
- [Restrictions](#restrictions)
- [Making video and audio calls](#making-video-and-audio-calls)
    - [`startCall()`](#startcall())
    - [Registering/unregistering an event handler](#registering/unregistering-an-event-handler)
    - [Timeout for `startCall()`](#timeout-for-startcall())
    - [Getting a `Call` instance](#getting-a-call-instance)
    - [Identifying the type of a message](#identifying-the-type-of-a-message)
    - [Starting a call from a push notification](#starting-a-call-from-a-push-notification)
- [Classes](#classes)
    - [`SendBirdVideoChat`](#sendbirdvideochat)
    - [`SendBirdVideoView`](#sendbirdvideoview)
    - [`CallOptions`](#calloptions)
    - [`Call`](#call)
    - [`CallUser`](#calluser)
    - [`VideoChatType`](#videochattype)
    - [`VideoChatError`](#Videochaterror)
- [Change Log](https://github.com/smilefam/sendbird-videochat-android/blob/master/CHANGELOG.md)
- [License](https://github.com/smilefam/sendbird-videochat-android/blob/master/LICENSE.md)

## How to set and initialize `VideoChat`

The following shows how to set SendBird `VideoChat` in the `build.gradle` and ` AndroidManifest.xml` files and how to initialize it in your application.   

### `build.gradle`

Add the lines below to your `build.gradle` file at application level (not project level).

```
android {
    defaultConfig {
        minSdkVersion 16
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

repositories {
    maven { url "https://raw.githubusercontent.com/smilefam/sendbird-videochat-android/master/" }
}

dependencies {
    // WebRTC
    implementation 'org.webrtc:google-webrtc:1.0.26885'

    // SendBird
    implementation 'com.sendbird.sdk:sendbird-android-sdk:3.0.91'

    // VideoChat
    implementation 'com.sendbird.sdk:sendbird-videochat:0.9.0'
}
```

> Note: `VideoChat` SDK is predicated upon the [SendBird Android SDK](https://github.com/smilefam/SendBird-SDK-Android) at least version 3.0.91. If you don't have the Android SDK, you should install it first.

### AndroidManifest.xml

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

### `onCreate()` in the application

```java
// 'onCreate()' in the BaseApplication extends Application
SendBird.init(APP_ID, getApplicationContext());
SendBirdVideoChat.init(getApplicationContext());
```

## Restrictions

There are a few restrictions that apply to the beta version of SendBird `VideoChat`.  

- It is only available in a `normal`, `private` group channel settings. (non-super and non-public)
- It only supports video and audio calls between two users.   
- To start video and audio calls, the two users must be connected to SendBird server.  
- In the same channel, you can't start a new, separate video or audio call.  

> Warning: Don't call the `removeAllChannelHandlers()` of SendBird Android SDK. It will not only remove handlers already added, but also remove handlers managed by `SendBirdVideoChat`.  

## Making video and audio calls

You can make video and audio calls between two users using the `SendBirdVideoChat`'s methods.   

### `StartCall()`  

The `startCall()` sends a video or audio call request to SendBird server with specified options. This function requires the `isAudioCall` and `callOptions` parameters.   

- **isAudioCall**: determines whether to enable an audio only in the call or both a video and an audio. 
- **callOptions**: specifies the call settings with a `CallOptions` instance. For more information on the settings, see the [CallOptions](#-CallOptions). 

```java
SendBirdVideoChat.startCall(isAudioCall, callOptions, new SendBirdVideoChat.StartCallHandler() {
    @Override
    public void onResult(SendBirdException e) {
    }
});
```

If the server accepts the request and notify a callee of it, a caller receives a success callback through the `VideoChatHandler`'s `onStartSent()`. And the callee also receives a notification about the request through the `VideoChatHandler`'s `onStartReceived()`.  

### Registering/unregistering an event handler

Like the SendBird SDKs, the `SendBirdVideoChat` has its own event handler. The `VideoChatHandler` supports various methods which receive events callbacks from SendBird server. You can create, register, and implement the handler in your application. Then your application will be notified of events happening in video and audio calls. 

```java
SendBirdVideoChat.addVideoChatHandler("IDENTIFIER", new SendBirdVideoChat.VideoChatHandler() {
    @Override
    public void onStartReceived(Call call) {
    }

    @Override
    public void onAcceptReceived(Call call) {
    }

    @Override
    public void onEndReceived(Call call) {
    }

    @Override
    public void onStartSent(Call call, UserMessage userMessage) {
    }

    @Override
    public void onAcceptSent(Call call) {
    }

    @Override
    public void onEndSent(Call call, UserMessage userMessage) {
    }

    @Override
    public void onConnected(Call call) {
    }

    @Override
    public void onOpponentVideoStateChanged(Call call) {
    }

    @Override
    public void onOpponentAudioStateChanged(Call call) {
    }
});

// If you unregister VideoChatHandler, use as follows.  
// SendBirdVideoChat.removeVideoChatHandler("IDENTIFIER");
// SendBirdVideoChat.removeAllVideoChatHandlers();
```

### Timeout for `startCall()`

If a callee doesn't respond to a call request from a caller during 30 seconds (by default), the `SendBirdVideoChat` automatically closes the call. A timeout value can be set between 30 and 60 seconds.  

```java
SendBirdVideoChat.setStartCallTimeout(40);
```

### Getting a `Call` instance  

The `SendBirdVideoChat` provides a `Call` class which has its own properties and methods that represent a specific video or audio call. For more information on the object, see the [`Call`](#-Call). 

- **getActiveCall()**: retrieves the current `Call` instance in progress.  
- **getCall()**: retrieves the `Call` instance by a passed call ID.   
- **buildCallFromNotification()**: with the `SendBird` payload in a push notificaiton message, creates and returns a new 'Call' instance.  
- **buildCall()**: with a passed `UserMessage` object which requests a video or audio call, creates and returns a new `Call` instance.  

```java
Call activeCall = SendBirdVideoChat.getActiveCall();
Call call = SendBirdVideoChat.getCall("CALL_ID");
Call call = SendBirdVideoChat.buildCallFromNotification("SendBird_Payload");
Call call = SendBirdVideoChat.buildCall(userMessage);
```

### Identifying the type of a message

Using the `getRenderingMessageType()`, you can identify the type of a passed message and determine how to render your chat view based on the message. The method returns one of the following four values:  

- **START**: the call request message.
- **END**: the call close message.
- **CHAT**: returned if a message is one of `UserMessage`, `FileMessage`, and `AdminMessage` as the SendBird SDK's `BaseMessage`.
- **NOT_RENDER**: a message that does not need to be rendered on the chat view.  

```java
VideoChatType.RenderingMessageType renderingType = SendBirdVideoChat.getRenderingMessageType(message);
```

### Starting a call from a push notification

The following shows how to parse a notification message containing the `sendbird` payload and how to enable a video or audio call by using the information of the payload.

```java
// public static boolean handleNotification(final String sendbird);
// public static Call buildCallFromNotification(final String sendbird);
// public static Call getCall(final String callId);

@Override
public void onMessageReceived(RemoteMessage remoteMessage) {
    ...
    String sendbird = remoteMessage.getData().get("sendbird");
    JSONObject sendbirdObj = new JSONObject(sendbird);
    JSONObject channel = (JSONObject) sendbirdObj.get("channel");
    String channelUrl = channel.getString("channel_url");
    if (channelUrl != null) {
        if (SendBirdVideoChat.handleNotification(sendbird)) {
            Call call = SendBirdVideoChat.buildCallFromNotification(sendbird);
            if (call != null) {
                // Start VideoChatActivity with callId. (call.getCallId())
                // Do getCall(callId) to get `Call` instance in VideoChatActivity.
            }
        } else {
            // Normal chat message.
        }
    }
}
```

## Classes

### `SendBirdVideoChat`

As the main class of the SendBird `VideoChat` add-on, the `SendBirdVideoChat` is well-explained with the code snippets in the above. Therefore, in this section we focus on its `VideoChatHandler` which handles various events for managing a video or audio call.

The `VideoChatHandler` supports various methods which receive events callbacks from SendBird server. If the handler is registered and implemented in your application, events happening in video and audio calls are notified to the application through the methods.
The following are provided with the `VideoChatHandler`:

1. **onStartSent(Call call, UserMessage message)**
    - Called when a caller's video or audio call request is successfully accepted by SendBird server from the `startCall()`. (At the caller's application) 
    - **call**: a `Call` instance which contains the current call status.  
    - **message**: a `UserMessage` instance which contains the text sent with the call request.  
2. **onStartReceived(Call call)**
    - Called when a callee receives a video or audio call request. (At the callee's application)  
    - **call**: a `Call` instance which contains the current call status.  
3. **onAcceptSent(Call call)**
    - Called when a callee has accepted a video or audio call request using the `accept()` and SendBird server confirms the acceptance. (At the callee's application)      
    - **call**: a `Call` instance which contains the current call status.  
4. **onAcceptReceived(Call call)**
    - Called when TURN server starts a video or audio call delivery between a caller and callee. (At the caller's application)    
    - **call**: a `Call` instance which contains the current call status.  
5. **onEndSent(Call call, UserMessage message)**
    - Called when a call close request has been sent to SendBird server using the `end()` and the server successfully accepts the request. (At the application which sent the close request)  
    - **call**: a `Call` instance which contains the current call status.  
    - **message**: a `UserMessage` instance which contains the text sent with the call close request.  
6. **onEndReceived(Call call)**
    - Called when a call has been closed from the opponent's request. (At the application which receives the close request)    
    - **call**: a `Call` instance which contains the current call status.  
7. **onConnected(Call call)**
    - Called when caller and callee are connected via SendBird server and can communicate with each other. (at both applications)   
    - **call**: a `Call` instance which contains the current call status.  
8. **onOpponentAudioStateChanged(Call call)**
    - Called when the audio state of either a caller or callee has been changed. (Notifies the opposite application)    
    - **call**: a `Call` instance which contains the current call status.  
9. **onOpponentVideoStateChanged(Call call)**
    - Called when the video state of either a caller or callee has been changed. (Notifies the opposite application) 
    - **call**: a `Call` instance which contains the current call status.  

### `SendBirdVideoView`

The `SendBirdVideoView` supports the methods for local and remote video chat views.

### `CallOptions`  

The `CallOptions` is a class that users use to request calls with the `startCall()` or to accept calls with the `accept()`. The items of `CallOptions` are:

- **CallOptions(Context context, String groupChannelUrl)**: specifies the URL of the channel to send a call request.
- **setLocalVideoView(SendBirdVideoView localVideoView)**: specifies the `SendBirdVideoView` to render the `Caller`'s chat view.  
- **setRemoteVideoView(SendBirdVideoView remoteVideoView)**: specifies the `SendBirdVideoView` to render the `Callee`'s chat view.
- **setVideoEnabled(boolean enable)**: determines whether to use video in the `Call`. This value is only accepted in a video call. (default: true)
- **setAudioEnabled(boolean enable)**: determines whether to use audio in the `Call`. (default: true)
- **setVideoWidth(int videoWidth)**: specifies the width of the video. This value is only effective in a video call. (default: 1280)
- **setVideoHeight(int videoHeight)**: specifies the height of the video. This value is only effective in a video call. (default: 720)
- **setVideoFps(int videoFps)**: specifies the frame rate of the video. This value is only effective in a video call. (default: 30)
- **setDefaultCamera(boolean frontCamera)**: specifies the default camera. Try to set the front camera if `true`. (default: true)

```java
CallOptions callOptions = new CallOptions(context, groupChannel.getUrl())
    .setLocalVideoView(localVideoView)
    .setRemoteVideoView(remoteVideoView)
    .setVideoEnabled(true)
    .setAudioEnabled(true)
    .setVideoWidth(1280)
    .setVideoHeight(720)
    .setVideoFps(30)
    .setDefaultCamera(true);
```

### `Call`

Through a `Call` instance, you can make actions of a video or audio call. It also contains up-to-date information of the call.  

- **getCallId()**: retrieves the unique ID that distinguishes each call.  
- **getGroupChannelUrl()**: retrieves the URL of the channel of your video or audio call.  
- **getMessageId()**: retrieves the ID of a message which containing information about the `Call`.  
- **isAudioCall()**: checks if the `Call` is an audio call.  
- **getCaller()**: retrieves information of the `Caller`, the one who makes the call.  
- **getCallee()**: retrieves information of the `Callee`, the one who receives the call.  
- **getEnder()**: retrieves information of the `Caller` or `Callee`, the one who ends the call.
- **getEndType()**: specifies that the `Call` has ended. This has one of the following values.
    - **NONE**: it's not ended yet.
    - **END**: a `Caller` or `Callee` ended the video or audio call after the connection.
    - **CANCEL**: the `end()`called by the `Caller` before `Callee` accepts the `Call`.
    - **DECLINE**: the `end()` called by the `Callee` without accepting the `Call`.
    - **TIMEOUT**: the `Call` was closed when a `Callee` didn't respond to a call request.
    - **UNKNOWN**: the `Call` was closed for unknown reasons.
- **getPeriod()**: retrieves the length of time in unix timestamp for `Call`.
- **getMyRole()**: retrieves the role on the `Call` as a `Caller` or `Callee`.
- **accept()**: the `Callee` accepts the `Call`.
- **end()**: close the `Call`.
- **startVideo()**: calls the `onOpponentVideoStateChanged()` handler that you registered to start video streaming on the `Call`.
- **stopVideo()**: calls the `onOpponentVideoStateChanged()` handler that you registered to stop video streaming on the `Call`.
- **muteMicrophone()**: calls the `onOpponentAudioStateChanged()` handler that you registered to start audio streaming on the `Call`.
- **unmuteMicrophone()**: calls the `onOpponentAudioStateChanged()` handler that you registered to stop audio streaming on the `Call`.

```java
SendBirdVideoChat.addVideoChatHandler("IDENTIFIER", new SendBirdVideoChat.VideoChatHandler() {
    @Override
    public void onStartReceived(Call call) {
        // Decline a call
        call.end(new Call.EndCallHandler() {
            @Override
            public void onResult(SendBirdException e) {
            }
        });
    }

    @Override
    public void onStartSent(Call call, UserMessage userMessage) {
        // Cancel a call
        call.end(new Call.EndCallHandler() {
            @Override
            public void onResult(SendBirdException e) { 
            }
        });
    }

    @Override
    public void onConnected(Call call) {
        // End a call
        call.end(new Call.EndCallHandler() {
            @Override
            public void onResult(SendBirdException e) {
            }
        });
    }

    // ...
});

Call call = SendBirdVideoChat.getActiveCall();
call.startVideo();
call.stopVideo();
call.muteMicrophone();
call.unmuteMicrophone();
```

### `CallUser`

`CallUser` can be identified as a `Caller` or `Callee`. The `Caller` is the one who requests a call and the `Callee` is the one who receives a call request.

- **getUserId()**: retrieve the user ID of the `Caller` or `Callee`.  
- **getNickname()**: retrieve the nickname of the `Caller` or `Callee`.  
- **getProfileUrl()**: retrieve the profile URL of the `Caller` or `Callee`.
- **isVideoEnabled()**: indicates whether the `Caller` or `Callee` is using video.  
- **isAudioEnabled()**: indicates whether the `Caller` or `Callee` is using audio.  

### `VideoChatType`  

The `VideoChatType` is determined by three different types of `SendBirdVideoChat`.  

```java
public class VideoChatType {
    public enum RenderingMessageType {
        START, END, CHAT, NOT_RENDER
    }

    public enum Role {
        NONE, CALLER, CALLEE
    }

    public enum EndType {
        NONE, END, CANCEL, DECLINE, TIMEOUT, UNKNOWN
    }
}
```

### `VideoChatError`

The `VideoChatError` is returned in the event of an error in `SendBirdVideoChat`.  

```java
public static final int ERR_UNKNOWN                         = 820000;
public static final int ERR_SENDBIRD_CONNECTION_REQUIRED    = 820100;
public static final int ERR_INVALID_ACTION                  = 820200;
public static final int ERR_ACCEPTING_FAIL                  = 820300;
public static final int ERR_INVALID_PARAMETER               = 820400;
public static final int ERR_UNSUITABLE_CHANNEL              = 820401;
public static final int ERR_ALREADY_CAMERA_SWITCHING        = 820510;
public static final int ERR_CALL_ON_GOING                   = 900600;
```
