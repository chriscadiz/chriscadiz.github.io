---
layout: post
title: Shorcut it out - How to use Android App Shortcuts and Improve Usability
---
Introduced in Android 7.1 Nougat (API level 25), App Shortcuts allow users a new way to interact with your apps from the launcher. You can define static shortcuts in xml and dynamic shorcuts in code using the ShortcutManager API.

Longpressing your app icon from the launcher will bring up a list of up to five shortcuts. Users can then either click to open the shortcuts or pin shortcuts to the home screen. What this means is that users can get to activities they care about with fewer taps.

![Android Nougat Shortcuts]({{ site.baseurl }}/assets/images/android-shortcut.gif "Android Nougat Shortcuts")

### Static Shortcuts

Good static shortcuts are core actions that users are likely to use in your app. For example in our sample app, NI (aka Not Instagram), we use "New Post" and "New Message" as static shortcuts.

These shortcuts are defined in xml. In your `AndroidManifest.xml` define a new `<meta-data>` element under the main activity (the activity with `android.intent.action.MAIN` and `android.intent.category.LAUNCHER` as its action and category Intent filters, respectively). Here we refer to the `shortcuts.xml` resource we are about to create.

{% highlight xml %}
...
<activity
    android:name=".MainActivity"
    android:label="@string/app_name"
    android:theme="@style/AppTheme.NoActionBar">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

    <meta-data
        android:name="android.app.shortcuts"
        android:resource="@xml/shortcuts" />
</activity>
...
{% endhighlight %}

Create the `shortcuts.xml` file and add the shortcut elements. Each element defines basic properties like icon, id, and labels. 

Of note are the `<intent>` elements, which define the Intents the shortcuts will open. You can also add multiple `<intent>` tags to add Intents onto the backstack.

> Sidenote: Follow the Android [App Shorcuts Design Guidelines][design-guidelines] to maintain consistent design.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:enabled="true"
        android:icon="@drawable/new_post"
        android:shortcutDisabledMessage="@string/new_post"
        android:shortcutId="post"
        android:shortcutLongLabel="@string/new_post"
        android:shortcutShortLabel="@string/new_post">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetClass="com.nimblinteractive.demo.PostActivity"
            android:targetPackage="com.nimblinteractive.demo" />
    </shortcut>
    <shortcut
        android:enabled="true"
        android:icon="@drawable/new_message"
        android:shortcutDisabledMessage="@string/new_message"
        android:shortcutId="message"
        android:shortcutLongLabel="@string/new_message"
        android:shortcutShortLabel="@string/new_message">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetClass="com.nimblinteractive.demo.NewMessageActivity"
            android:targetPackage="com.nimblinteractive.demo" />
    </shortcut>
</shortcuts>
{% endhighlight %}

![Static Shortcuts]({{ site.baseurl }}/assets/images/static-shortcut.jpg "Android Static Shortcuts")

_Static shorcuts in the launcher_

### Dynamic Shortcuts

Some shortcutable actions may appear depending on context within your app. In our sample app, let's say a user just followed another user with the tag @kimk. Let's present them with a shorcut to view @kimk's profile and a shortcut to message @kimk.

Creating dynamic shortcuts is almost just as easy with the `ShortcutManager` API. Just like in xml, the `ShortcutInfo.Builder` shortcuts can have multiple Intents representing the backstack.

We use `setDynamicShortcuts()` to set the entire list of dynamic shortcuts. 

In addition, the `ShortcutManager` API provides

* addDynamicShortcuts()
* updateShortcuts()
* removeDynamicShortcuts()
* removeAllDynamicShortcuts()

{% highlight java %}
ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);
Intent profileIntent = new Intent("com.nimblinteractive.demo.action.PROFILE");
Intent messageIntent = new Intent("com.nimblinteractive.demo.action.MESSAGE");

ShortcutInfo messageShortcut = new ShortcutInfo.Builder(context, "kim_message")
        .setShortLabel("@kimk")
        .setIcon(Icon.createWithResource(context, R.drawable.new_message))
        .setLongLabel("Message @kimk")
        .setIntents(new Intent[]{profileIntent, messageIntent})
        .build();

ShortcutInfo profileShortcut = new ShortcutInfo.Builder(context, "kim_profile")
        .setShortLabel("@kimk")
        .setLongLabel("@kimk's Profile")
        .setIntents(new Intent[]{profileIntent})
        .build();

shortcutManager.setDynamicShortcuts(Arrays.asList(messageShortcut, profileShortcut));
{% endhighlight %}

![Dynamic Shortcuts]({{ site.baseurl }}/assets/images/dynamic-shortcut.jpg "Android Dynamic Shortcuts")

_Dynamically added shortcuts in the launcher_

### Best Practices

Whenever a user uses a shortcut or does the equivalent by navigating through your app, call `reportShortcutUsed()`. Launchers will use this information to predict and promote the most likely shortcuts.

If your app allows backup, dynamic shortcuts aren't preserved on restore. The Android team recommends using `getDynamicShortcuts` on launch to check if shortcuts need to be restored.

### Further Reading

* [Android Developers App Shortcuts Documentation][docs]

[design-guidelines]:	//commondatastorage.googleapis.com/androiddevelopers/shareables/design/app-shortcuts-design-guidelines.pdf
[docs]:	//developer.android.com/guide/topics/ui/shortcuts.html

