title: "The solution that the menu item of Navigation View can't change its background When it is selected/checked."
date: 2016-07-18 10:08:03
tags: NavigationView
categories: Android
---
Today, when I created a left navigation bar in android app, I met a strange issue. That was I couldn't appoint a background color for the navigation item when it was selected. Like this:

{%asset_img no-background.gif %}

 It was annoying, I found many answers on the internet, but they all couldn't fix my issue.

Most of answers said I should created a selector in drawable folder, like:

```XML
<selector xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:drawable="@color/primary" android:state_checked="true" />
        <item android:drawable="@android:color/transparent" />
</selector>
```
and set NavigationView like:

```XML
app:itemBackground="@drawable/nav_view_item_background"
```
For me, it is not the correct answer. Though I set the `state_checked` is true and give it a drawable color, it seems the item is never checked.

For more search, I found the issue was happened with menu item.
The original menu file is:
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:title="主题" android:icon="@drawable/ic_dashboard_grey" android:id="@+id/theme" android:checked="true"/>
        <item android:title="节点" android:icon="@drawable/ic_view_agenda_grey" android:id="@+id/node"/>
        <item android:title="其他" android:id="@+id/other">
            <menu>
                <item android:title="关于" android:icon="@drawable/ic_info_grey" android:id="@+id/about"/>
            </menu>
        </item>
</menu>
```
Then I add a tag `android:checkable=true` to it:
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:title="主题" android:icon="@drawable/ic_dashboard_grey" android:id="@+id/theme" android:checked="true"
            android:checkable="true"/>
        <item android:title="节点" android:icon="@drawable/ic_view_agenda_grey" android:id="@+id/node"
            android:checkable="true"/>
        <item android:title="其他" android:id="@+id/other">
            <menu>
                <item android:title="关于" android:icon="@drawable/ic_info_grey" android:id="@+id/about"
                    android:checkable="true"/>
            </menu>
        </item>
</menu>
```
Now, the effect is normal.

{% asset_img background.gif %}
