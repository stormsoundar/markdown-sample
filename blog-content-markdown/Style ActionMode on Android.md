Last week, we released a new feature to our customers in our Android App: **Boards**, which lets our customers group their favourite items. Since users can save up to 500 items for each board, we wanted them to be able to edit the items in batches. In our design, when the user clicks the ‘edit’ button or long presses one Saved Item, the screen transitions into the edit mode. In this mode, the user can select multiple products to add/remove in one batch operation. During development, we found that ActionMode is a perfect fit for this purpose. Here is an animation showing the transition to edit mode and selecting multiple products. If you want to play with it and check more details, please download our Android app [here](https://play.google.com/store/apps/details?id=com.asos.app).

![](https://miro.medium.com/max/602/1*nGNA9YwcLENDXHry9nfV-A.gif)

## ActionMode

So, what’s [ActionMode](https://developer.android.com/reference/android/view/ActionMode)? ActionMode provides alternative interaction modes and replaces parts of the normal UI until finished. It was introduced back from Android HoneyComb with [ActionBar](https://developer.android.com/training/appbar/). There is also an implementation in the support library (_appcompat-v7_), so you can even use it from version 7. To use ActionMode, just make sure you are using the one from support lib, which is `android.support.v7.view.ActionMode`, not the one `android.view.ActionMode`. Btw, you may try to use the classes from support library, which can be updated and bug fixed sooner when the support library gets updated, rather than waiting for the fix from the standard Framework API.

There are two types of ActionMode: Primary mode and Floating mode. The Primary mode is the one we are using in the **Boards** implementation, which replaces the [Toolbar](https://developer.android.com/reference/android/support/v7/widget/Toolbar) UI and contextual menu. The Floating mode provides a floating toolbar, which was introduced from version 23 and you can find in the text edit widget in Gmail or Google Doc.

## Start ActionMode

Starting ActionMode is very simple. Just call the following method and it will return `ActionMode`

`AppCompatActivity. startSupportActionMode(callback):ActionMode`

You can use the callback to prepare the view or menu and release the resource after the ActionMode finishes.

![](https://miro.medium.com/max/576/1*cZeA_3iJjZJSrkw2dCWu5g.png)

## Stop ActionMode

There are several ways to stop the ActionMode:

- User presses back button
- User clicks the close icon you define in your ActionMode
- Call `ActionMode.finish` method in the code

No matter which way the ActionMode is stopped, the `OnDestroyActionMode` method in the `ActionMode.Callbackwill` be called.

![](https://miro.medium.com/max/576/1*beQnodzXbSM4nH3O-tjL9w.png)

Here is a more detailed view of the ActionMode UI. It looks very similar to the Toolbar UI, where you can set the background colour, title style, subTitle style, close icon.

Most of the time you will create a theme to customise the ActionMode UI, so you can set the theme for each activity. You can create a new one if the ActionMode UI style is totally different from other parts. Or, you can extend your app theme, so you can have the same style across the app. A typical theme can be defined like this:

```<style name="ActionModeTheme" parent="AppTheme">
    <item name="actionModeStyle">@style/YourActionModeStyle</item>
    <item name="windowActionModeOverlay">true</item>
    <item name="actionModeCloseDrawable">@drawable/ic_action_close</item>
    <item name="actionMenuTextColor">@color/white</item>
</style>
```

- The `ActionModeStyle` is in the App theme level, which can be set for each activity (make sure you are using `AppCompatActivity` from support library).

- The `WindowActionModeOverlay` flag will let you choose if want to replace the Toolbar you have (overlay) or put your ActionMode view under the Toolbar UI (most of the time you will set true here to only show one Toolbar). When you set the overlay true, you may be aware the toolbar is not hidden, just under the ActionMode UI. There will be some accessibility issues, as the toolbar menu item can still be triggered with voice over.

- The `ActionModeCloseDrawable` is the icon at the start, which you want the user to click and close the ActionMode. If you leave it as default, it will be the black back arrow **_(abc_ic_ab_back_material)_**.

- The `ActionMenuTextColor` is the text colour for the contextual menu you create when you start the ActionModel

![](https://miro.medium.com/max/576/1*UKHaPdhmFS0HG1E1N14stg.png)

Your ActionMode style should extend the `Widget.AppCompat.ActionMode` and it extends from "Base.Widget.AppCompat.ActionMode", which has six properties you can customise.

```<style name="Base.Widget.AppCompat.ActionMode" parent="">
    <item name="background">?attr/actionModeBackground</item>
    <item name="backgroundSplit">?attr/actionModeSplitBackground</item>
    <item name="height">?attr/actionBarSize</item>
    <item name="titleTextStyle">@style/TextAppearance.AppCompat.Widget.ActionMode.Title</item>
    <item name="subtitleTextStyle">@style/TextAppearance.AppCompat.Widget.ActionMode.Subtitle</item>
    <item name="closeItemLayout">@layout/abc_action_mode_close_item_material</item>
</style>
```

- Background: can be a pure colour or drawable

- Title style: any text view, such as text colour, size, font

- SubTitle Style: same as the title style. If you don’t set a subtitle, the title will be vertically centred in a `LinearLayout` . Also, the font size is limited by the ActionMode `height`.

- CloseItemLayout: the layout which contains the close button icon. If you want to customise this one, you need to make sure you have the same resource ID **_(action_mode_close_button)_**

![](https://miro.medium.com/max/576/1*vbMmJxOmWDgMdjH59tff-g.png)

If you want to know more customisable properties, you can check the ActionMode section in the support lib (_appcompat-v7_) style.xml

```
<item name="actionModeStyle">@style/Widget.AppCompat.ActionMode</item>
<item name="actionModeBackground">@drawable/abc_cab_background_top_material</item>
<item name="actionModeSplitBackground">?attr/colorPrimaryDark</item>
<item name="actionModeCloseDrawable">@drawable/abc_ic_ab_back_material</item>
<item name="actionModeCloseButtonStyle">@style/Widget.AppCompat.ActionButton.CloseMode</item>
<item name="actionModeCutDrawable">@drawable/abc_ic_menu_cut_mtrl_alpha</item>
<item name="actionModeCopyDrawable">@drawable/abc_ic_menu_copy_mtrl_am_alpha</item>
<item name="actionModePasteDrawable">@drawable/abc_ic_menu_paste_mtrl_am_alpha</item>
<item name="actionModeSelectAllDrawable">@drawable/abc_ic_menu_selectall_mtrl_alpha</item>
<item name="actionModeShareDrawable">@drawable/abc_ic_menu_share_mtrl_alpha</item>
```

I hope this post gives you some ideas to style your ActionMode UI. If you have any questions, thoughts or suggestions, feel free to leave comments or let me know if you’d like to hear more.

Want to know how to style the system bars on Android, you can check my post [here](https://medium.com/@peng.jiang/style-system-bars-on-android-476ed0b64d02)

Hi! My name is Peng Jiang. I’m always passionate about simple and well-crafted code. I currently work as a Lead Android Engineer at ASOS. In my spare time, I also play with Swift and Flutter. At ASOS, we’re always looking for strong, friendly and talented developers. You will only need to write Kotlin here. If that sounds interesting to you, do [get in touch](https://www.linkedin.com/in/pengj1/)!
