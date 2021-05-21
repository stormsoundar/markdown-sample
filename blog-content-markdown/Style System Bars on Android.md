When building mobile User Interfaces (UIs), it is common to only think of the UIs in the App you are building. However, the App UIs are not standalone on the mobile screen, they are integrated with the system UI. As you can see from the above screenshots, you can feel UIs in the second and fourth screenshots are linked, and blend together, as opposed to a break between the notch and the UI, which also placing emphasis on an app’s content instead of its structure.

In this post, I will describe what I have learnt while styling the system bars with our UIs and describe how can you apply it with your App UIs. If you want to know how to style the action bar on Android, you can find my previous post [here](https://medium.com/asos-techblog/style-actionmode-on-android-5e613fa77c32).

## System Bars

What are the system bars? According to [this](https://developer.android.com/design/handhelds#system-bars), they are the status bar and the navigation bar, which are displayed concurrently with your App UI. The app bar or the action bar aren’t included here as it is part of your UI and it should be included in your UI design.

The system bars will be adapted based on your styles setting if you have not set it specifically. The default value will be the `colorPrimaryDark`, which was introduced in the material design theme from Android 5.0 Lollipop (Android 21+).

![](https://miro.medium.com/max/576/1*acozVeni8BbmxIKkXCIGUQ.png)

Except for the theme changes, most of the time, your UIs will not have any connection or overlap with the system bars. If you want to change the status bar and navigation bar colour separately, there are two way you can change them apart from the default `colorPrimaryDark` . One is set in the app style as below

```
<item name="android:statusBarColor">@color/statusBarColour</item>
<item name="android:navigationBarColor">@color/navigationBarColor</item>
```

Or you can change it in the code (support from Android version 19+)

```
android.view.Window#setStatusBarColor(R.color.statusBarColour)
android.view.Window#setNavigationBarColor(R.color.navigationBarColor)
```

It will be better to use a colour reference here, which can make it easier to switch between dark and light mode. But if you have a very light background colour, you will find the icons in the system bar are not clearly visible as the default icon colour is also in a light colour.

![](https://miro.medium.com/max/576/1*_bHElBB03D3puzxaLvIsEQ.png)

There is no way you can customise the status bar or navigation bar icon colours without hacking. But from Android API 23, you can make the status bar and navigation icons dark by setting the style item.

```
<item name="android:windowLightStatusBar">true</item>
<item name="android:windowLightNavigationBar">true</item>
```

Since this API only support from API 23+, we can only enable these flags to have clearly visible status bar icons before lollipop devices and keep the status bar and navigation bar colour with our main background colour.

## Navigation Drawer

Most of the time, your UI will not overlap the status bar or navigation bar. But some UI component can, such as [Navigation Drawer](https://developer.android.com/guide/navigation/navigation-ui#add_a_navigation_drawer), which provides an easy way to shows your app’s main navigation menu.

When you open your drawer, it can overlap with the status bar according to your designer requirement. You can find our current implementation below.

![](https://miro.medium.com/max/288/1*AfBiubqXON2eOooLc1Re3g.gif)

To achieve this, the tricky part is to switch between the _translucent_ status bar _and light_ status bar when drawer open and close. There are two flags in the host window to control this status.

The [_FLAG_TRANSLUCENT_STATUS_](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#FLAG_TRANSLUCENT_STATUS) flag will request a translucent status bar with a minimal system-provided background. So you can see through from the status bar and use your drawer UIs as background.

The [_SYSTEM_UI_FLAG_LIGHT_STATUS_BAR_](https://developer.android.com/reference/android/view/View#SYSTEM_UI_FLAG_LIGHT_STATUS_BAR) flag will request the status bar to draw in a mode that is compatible with light status bar backgrounds. For this flag to take effect, the window must clear the translucent flag and request the [_FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS_](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS).

[SimpleDrawerListener](https://developer.android.com/reference/androidx/drawerlayout/widget/DrawerLayout.SimpleDrawerListener) is used for monitoring events about drawers. Once the drawer is starting to open, the translucent status needs to be set and the light status bar flag needs to be removed if the app is in the light model. The status update need be applied to the drawer layout host [window](https://developer.android.com/reference/android/view/Window).

And you can use Window [_setFlags_](<https://developer.android.com/reference/android/view/Window#setFlags(int,%20int)>) and [_clearFlags_](<https://developer.android.com/reference/android/view/Window#clearFlags(int)>) methods call to set and clear the flags.

```
window.setFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS,
WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS)

window.clearFlags(View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR)
```

The lessons learned when styling the system bars:

- Adapt the system UI to provide a more immerse user experience.

- Try to use the system default option when setting the colour scheme.

- Be careful with different API version support and provide different version resource folder for better

- When you set a system flag effect, be aware of different states and conflicts.

- Handle the light and dark model, provide a great UX!
