When I travel, I frequently use my phone or tablet to read content, and having a mobile app for this purpose can make a
significant difference in my experience.

I have been studying on [educative.io](https://www.educative.io/) for a few weeks and was surprised to find no
official mobile application for this amazing platform.

So I decided to create my own using an Android WebView.

_(I am not sponsored by or have any kind of partnership with Educative)._

## What is a WebView?

In a nutshell, a WebView is a component used in mobile app development that displays web pages within an app's
interface.
It is useful for displaying content that does not require integrations with your device but needs to be displayed on it.

In addition, it can be customized to your needs (style properties, screen size...) and interfaces
can be added to call Android APIs from within the app using JavaScript.

## Android App Architecture Overview

Detailed app architecture and best practices can be found [here](https://developer.android.com/topic/architecture).

## Creating a project

Requirements:

- Android Studio
- Java 11+
- Gradle 7+

Let's get our hands dirty and create the WebView.

### Adding the WebView component

In Android Studio:

- New Project > Empty Activity
- Enter the app name
- Enter the package name
- Select the language (we will use Kotlin for this post)
- Select the minimum API version
- Click on Create.

#### Minimal Setup

Add the following code to the `activity_main.yml` (in `app/res/layout`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                tools:context=".MainActivity">

    <WebView
            android:id="@+id/webview"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

</RelativeLayout>
```

_RelativeLayout documentation [here](https://developer.android.com/develop/ui/views/layout/relative)._

Replace the existing code by the following in `MainActivity.java`:

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var myWebView: WebView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        myWebView = findViewById(R.id.webview)
        myWebView.webViewClient = WebViewClient()

        myWebView.loadUrl(URL)
    }
}
```

This code will load the given url when the app starts.

Last but not the least, you need to authorize the app to access to the Internet by adding the permission to
the `AndroidManifest.xml` file (in `app/manifests`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET"/>
    ...
```

After these steps, you may be able to launch your app, but may need to add some settings to improve your experience.

#### Optimizing the application

If JavaScript is required, you need to enable the `myWebView.settings.javaScriptEnabled` property.
Moreover, I recommend to [bind](https://developer.android.com/develop/ui/views/layout/webapps/webview#BindingJavaScript)
the JavaScript code of the website to your app.

Here is the `MainActivity` class after the update:

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var myWebView: MyWebView

    @SuppressLint("SetJavaScriptEnabled")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        myWebView = findViewById(R.id.webview)
        myWebView.webViewClient = WebViewClient()

        setSettings()

        myWebView.loadUrl(URL)
    }

    private fun setSettings() {
        myWebView.settings.javaScriptEnabled = true
        myWebView.addJavascriptInterface(WebAppInterface(this), "Android")
    }

    class WebAppInterface(private val mContext: Context) {

        @JavascriptInterface
        fun showToast(toast: String) {
            Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show()
        }
    }
}
```

You might want to go back to your history using the back button.

You can override the `onKeyDown` method to add this feature.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var myWebView: MyWebView

    @SuppressLint("SetJavaScriptEnabled")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        myWebView = findViewById(R.id.webview)
        myWebView.webViewClient = WebViewClient()

        setSettings()

        myWebView.loadUrl(URL)
    }

    private fun setSettings() {
        myWebView.settings.javaScriptEnabled = true
        myWebView.addJavascriptInterface(WebAppInterface(this), "Android")
    }

    private class WebAppInterface(private val mContext: Context) {

        @JavascriptInterface
        fun showToast(toast: String) {
            Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show()
        }
    }

    override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
        if (keyCode == KeyEvent.KEYCODE_BACK && myWebView.canGoBack()) {
            myWebView.goBack()
            return true
        }
        return super.onKeyDown(keyCode, event)
    }
}
```

#### Solutions for rendering a web page properly

##### Dom storage

It may be necessary to enable the dom storage to properly render the desired website.
It is possible to enable it with the `myWebView.settings.domStorageEnabled` property.

##### Render once ready

To ensure that a web page is displayed once it is available, you can create a `WebApp` subclass and override
the `invalidate` method.

##### Cache settings

The cache mode can be updated as needed. The default value is `LOAD_DEFAULT` (use cached resources when they are
available and not expired, otherwise load resources from the network).

```kotlin
package com.ths83.educative

import android.annotation.SuppressLint
import android.content.Context
import android.os.Bundle
import android.util.AttributeSet
import android.view.KeyEvent
import android.webkit.JavascriptInterface
import android.webkit.WebView
import android.webkit.WebViewClient
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

private const val URL = "https://www.educative.io"

class MainActivity : AppCompatActivity() {

    private lateinit var myWebView: MyWebView

    @SuppressLint("SetJavaScriptEnabled")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContentView(R.layout.activity_main)

        myWebView = findViewById(R.id.webview)
        myWebView.webViewClient = WebViewClient()

        setSettings()

        myWebView.loadUrl(URL)
    }

    private fun setSettings() {
        myWebView.settings.javaScriptEnabled = true
        myWebView.addJavascriptInterface(WebAppInterface(this), "Android")
        myWebView.settings.builtInZoomControls = false
        myWebView.settings.domStorageEnabled = true
        myWebView.settings.cacheMode = WebSettings.LOAD_CACHE_ELSE_NETWORK
    }

    override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
        if (keyCode == KeyEvent.KEYCODE_BACK && myWebView.canGoBack()) {
            myWebView.goBack()
            return true
        }
        return super.onKeyDown(keyCode, event)
    }

    private class WebAppInterface(private val mContext: Context) {

        @JavascriptInterface
        fun showToast(toast: String) {
            Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show()
        }
    }

    class MyWebView(context: Context, attrs: AttributeSet?) : WebView(context, attrs) {
        override fun invalidate() {
            super.invalidate()
            // Display if fully downloaded and ready to render
            if (progress == 100 && contentHeight > 0) {
                // WebView has displayed some content and is scrollable.  
            }
        }
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                tools:context=".MainActivity">

    <view class="com.ths83.educative.MainActivity$MyWebView"
          android:id="@+id/webview"
          android:layout_width="match_parent"
          android:layout_height="match_parent"/>

</RelativeLayout>
```

##### Interesting settings to look at

In `AndroidManifest.xml`:

Opt out from Google's metrics:

```xml

<meta-data
        android:name="android.webkit.WebView.MetricsOptOut"
        android:value="true"/>
```

Disable safe browsing warnings:

```xml

<meta-data
        android:name="android.webkit.WebView.EnableSafeBrowsing"
        android:value="false"/>
```

## Consider those drawbacks

- Storage: Limited storage space can restrict the amount of data that an app can store locally, which can impact its
  functionality and user experience. Apps should be designed to use storage space efficiently and allow users to manage
  stored data.

- Network: The reliability and speed of mobile networks can vary widely, which can impact the performance and usability
  of apps that rely on network connectivity. Apps should be designed to gracefully handle network connectivity issues
  and provide a good user experience even when network conditions are poor.

- Battery life: Mobile devices have limited battery life, and power-hungry apps can quickly drain a device's battery.
  Apps should be designed to use power efficiently and minimize battery drain, especially for background processes and
  features that are not essential to the app's core functionality.

- Rendering: Mobile devices have different screen sizes, resolutions, and aspect ratios, which can affect the way apps
  are displayed. Apps should be designed to use responsive layouts and scalable graphics to ensure that they are
  displayed correctly on different devices. Additionally, apps should be optimized for performance and rendering speed
  to provide a smooth user experience.

## Outro

You can take a look at the GitHub's repo for this post [here](https://github.com/ths83/educative-android-webapp).

<iframe width="400" height="400" src="https://user-images.githubusercontent.com/11136821/222994706-3dea7fb6-3dfa-415b-8e3d-ed90a7e9b0ac.mp4" controls="controls">
</iframe>

And this is it for the post! Thanks for reading and stay tuned!

## Useful links

- What is web-based content?
    - [WebApps](https://developer.android.com/develop/ui/views/layout/webapps)
    - [WebView](https://developer.android.com/develop/ui/views/layout/webapps/webview)
- [Best practices](https://developer.android.com/develop/ui/views/layout/webapps/best-practices)
- [Optimize page
  rendering](https://stackoverflow.com/questions/4065134/is-there-a-listener-for-when-the-webview-displays-its-content/14678910#14678910)
- [Safe Browsing](https://developer.android.com/develop/ui/views/layout/webapps/managing-webview#safe-browsing)
- [Android Metrics](https://developer.android.com/develop/ui/views/layout/webapps/managing-webview#metrics)
- [Wasm](https://developer.android.com/develop/ui/views/layout/webapps/jsengine)
- [Enable dark theme](https://developer.android.com/develop/ui/views/layout/webapps/dark-theme)
- [Android tutorials](https://developer.android.com/training/basics/firstapp)
