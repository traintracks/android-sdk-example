# android-sdk-example

This is an example project on how to use the Android SDK.

The application is based on https://github.com/tvbarthel/ChaseWhisplyProject and records events as one uses the application and sends them to Traintracks.

The SDK is located at: `traintracks-android-sdk-1.0/traintracks-android-sdk-1.0.aar`

# How to use the SDK #

4.  In every file that uses analytics, import traintracks.android_sdk.Traintracks at the top:

    ```java
    import traintracks.android_sdk.Traintracks;
    ```

5. In the `onCreate()` of your main activity, initialize the SDK:

    ```java
    Traintracks.getInstance().initialize(this, "YOUR_API_ENDPOINT_HERE", "YOUR_API_KEY_HERE", "YOUR_API_SECRET_HERE", "USER_ID_HERE").enableForegroundTracking(getApplication());
    ```

6. To track an event anywhere in the app, call:

    ```java
    Traintracks.getInstance().logEvent("EVENT_IDENTIFIER_HERE");
    ```

7. If you want to use Google Advertising IDs, make sure to add [Google Play Services](https://developer.android.com/google/play-services/setup.html) to your project. _This is required for integrating with third party attribution services_

8. If you are using Proguard, add these exceptions to ```proguard.pro``` for Google Play Advertising IDs and Traintracks dependencies:

    ```yaml
        -keep class com.google.android.gms.ads.** { *; }
        -dontwarn okio.**
    ```

9. Events are saved locally. Uploads are batched to occur every 30 events and every 30 seconds. After calling `logEvent()` in your app, you will immediately see data appear on the Traintracks website.

# Tracking Events #

It's important to think about what types of events you care about as a developer. You should aim to track between 20 and 100 types of events within your app. Common event types are different screens within the app, actions a user initiates (such as pressing a button), and events you want a user to complete (such as filling out a form, completing a level, or making a payment). Contact us if you want assistance determining what would be best for you to track.

# Tracking Sessions #

A session is a period of time that a user has the app in the foreground. Events that are logged within the same session will have the same `session_id`. Sessions are handled automatically now; you no longer have to manually call `startSession()` or `endSession()`.

* For Android API level 14+, a new session is created when the app comes back into the foreground after being out of the foreground for 5 minutes or more. (Note you can define your own session experiation time by calling `setMinTimeBetweenSessionsMillis(timeout)`, where the timeout input is in milliseconds.)

* For Android API level 13 and below, foreground tracking is not available, so a new session is automatically started when an event is logged 30 minutes or more after the last logged event. If another event is logged within 30 minutes, it will extend the current session. (Note you can define your own session expiration time by calling `setSessionTimeoutMillis(timeout)`, where the timeout input is in milliseconds. Also note, `enableForegroundTracking(getApplication)` is still safe to call for Android API level 13 and below, even though it is not available.)

Other Session Options:

1.  By default start and end session events are no longer sent. To renable add this line after initializing the SDK:

    ```java
    Traintracks.getInstance().trackSessionEvents(true);
    ```

2. You can also log events as out of session. Out of session events have a `session_id` of `-1` and are not considered part of the current session, meaning they do not extend the current session (useful for things like push notifications). You can log events as out of session by setting input parameter `outOfSession` to `true` when calling `logEvent()`:

    ```java
    Traintracks.getInstance().logEvent("EVENT", null, true);
    ```

# Setting Event Properties #

You can attach additional data to any event by passing a JSONObject as the second argument to `logEvent()`:

```java
JSONObject eventProperties = new JSONObject();
try {
    eventProperties.put("KEY_GOES_HERE", "VALUE_GOES_HERE");
} catch (JSONException exception) {
}
Traintracks.getInstance().logEvent("Sent Message", eventProperties);
```

You will need to add two JSONObject imports to the code:

```java
import org.json.JSONException;
import org.json.JSONObject;
```

# Setting User Properties #

To add properties that are associated with a user, you can set user properties:

```java
JSONObject userProperties = new JSONObject();
try {
    userProperties.put("KEY_GOES_HERE", "VALUE_GOES_HERE");
} catch (JSONException exception) {
}
Traintracks.getInstance().setUserProperties(userProperties);
```

# User Property Operations #

The SDK supports the operations set, setOnce, unset, and add on individual user properties. The operations are declared via a provided `Identify` interface. Multiple operations can be chained together in a single `Identify` object. The `Identify` object is then passed to the Traintracks client to send to the server. The results of the operations will be visible immediately in the dashboard, and take effect for events logged after.

First you need to import the Identify class by adding this import statement at the top:

```java
import traintracks.android_sdk.Identify;
```

1. `set`: this sets the value of a user property.

    ```java
    Identify identify = new Identify().set('gender', 'female').set('age', 20);
    Traintracks.getInstance().identify(identify);
    ```

2. `setOnce`: this sets the value of a user property only once. Subsequent `setOnce` operations on that user property will be ignored. In the following example, `signupDate` will be set once to `08/24/2015`, and the following setOnce to `09/14/2015` will be ignored:

    ```java
    Identify identify1 = new Identify().setOnce('signupDate', '08/24/2015');
    Traintracks.getInstance().identify(identify1);

    Identify identify2 = new Identify().setOnce('signupDate', '09/14/2015');
    Traintracks.identify(identify2);
    ```

3. `unset`: this will unset and remove a user property.

    ```java
    Identify identify = new Identify().unset('gender').unset('age');
    Traintracks.getInstance().identify(identify);
    ```

4. `add`: this will increment a user property by some numerical value. If the user property does not have a value set yet, it will be initialized to 0 before being incremented.

    ```java
    Identify identify = new Identify().add('karma', 1).add('friends', 1);
    Traintracks.getInstance().identify(identify);
    ```

Note: if a user property is used in multiple operations on the same `Identify` object, only the first operation will be saved, and the rest will be ignored. In this example, only the set operation will be saved, and the add and unset will be ignored:

```java
Identify identify = new Identify().set('karma', 10).add('karma', 1).unset('karma');
Traintracks.getInstance().identify(identify);
```


# Fine-grained location tracking #

Traintracks access the Android location service (if possible) to add the specific coordinates (longitude and latitude)
where an event is logged.

This behaviour is enabled by default, but can be adjusted calling the following methods *after* initializing:

```java
Traintracks.getInstance().enableLocationListening();
Traintracks.getInstance().disableLocationListening();
```

Even disabling the location listening, the events will have the "country" property filled. That property
is retrieved from other sources (i.e. network or device locale).

# Allowing Users to Opt Out

To stop all event and session logging for a user, call setOptOut:

```java
Traintracks.getInstance().setOptOut(true);
```

Logging can be restarted by calling setOptOut again with enabled set to false.
No events will be logged during any period opt out is enabled.
