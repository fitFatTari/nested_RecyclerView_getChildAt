# nested_RecyclerView_getChildAt
This fixes scrolling issues with a RecyclerView inside a ScrollView and also the GetChildAt(position) returning null issue - MvvmCross -> Android 13.0
Hope this helps other people as it took me weeks of ChatGpting and StackOverflowing, mix all solutions to get here.

First create a custom control that inherits MvxRecyclerView:

```c#
using Android.Content;
using Android.Util;
using Android.Views;
using MvvmCross.DroidX.RecyclerView;

namespace MyApp.Droid.CustomControls
{
    public class MyNestedRecyclerView : MvxRecyclerView
    {
        public MyNestedRecyclerView(Context context, IAttributeSet attrs)
        : base(context, attrs)
        {
        }

        public override bool OnInterceptTouchEvent(MotionEvent ev)
        {
            Parent.RequestDisallowInterceptTouchEvent(true);
            return base.OnInterceptTouchEvent(ev);
        }
    }
}
```

Then set your axml designer as below:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:local="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">

    <ScrollView
        android:id="@+id/myScrollView"
        android:minWidth="25px"
        android:minHeight="25px"
        android:padding="10dp"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:fillViewport="true"
        android:layout_weight="1">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <TextView
                android:id="@+id/myTopScrollView"
                android:text="@string/someTextView"
                android:textAppearance="?android:attr/textAppearanceLarge"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1" />

            <DateEdit
                android:id="@+id/someDateEdit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                local:MvxBind="When Start; Enabled !ReadOnly;" />

            <androidx.core.widget.NestedScrollView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:fillViewport="true"
                android:layout_weight="1">

                <MyNestedRecyclerView
                    android:id="@+id/listone"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"    
                    local:MvxItemTemplate="@layout/listone"
                    local:MvxGroupItemTemplate="@layout/listone"
                    local:MvxBind="ItemsSource MyViewModel.ListOne; Enabled !ReadOnly;"/>
            </androidx.core.widget.NestedScrollView>

            <androidx.core.widget.NestedScrollView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:fillViewport="true"
                android:layout_weight="1">

                <MyNestedRecyclerView
                    android:id="@+id/listtwo"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    local:MvxItemTemplate="@layout/listtwo"
                    local:MvxGroupItemTemplate="@layout/listtwo"
                    local:MvxBind="ItemsSource MyViewModel.ListOne; Enabled !ReadOnly;"/>
            </androidx.core.widget.NestedScrollView>

        </LinearLayout>

    </ScrollView>

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnDelete"
            android:text="@string/btnDelete"
            android:layout_alignParentTop="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:includeFontPadding="false"
            local:MvxBind="Click DeleteCommand; Visibility Visibility(!ReadOnly);" />

        <Button
            android:id="@+id/btnSave"
            android:text="@string/btnSave"
            android:layout_alignParentTop="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:includeFontPadding="false"
            local:MvxBind="Click SaveCommand; Visibility Visibility(!ReadOnly);" />

        <Button
            android:id="@+id/btnCancel"
            android:text="@string/btnCancel"
            android:layout_alignParentTop="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:includeFontPadding="false"
            local:MvxBind="Click CancelCommand;" />

    </LinearLayout>

</LinearLayout >

```


Then your view can look like this to retrieve any value from the recycler view:

```c#
using Android.App;
using Android.Content;
using Android.Graphics;
using Android.OS;
using Android.Views;
using Android.Widget;
using AndroidX.Preference;
using MvvmCross;
using MvvmCross.DroidX.RecyclerView;
using MvvmCross.Navigation;
using Newtonsoft.Json;
using Shared.ViewModels;
using System;
using System.Collections.Generic;
using System.IO;
using Xamarin.Controls;

namespace Pirana.Droid.Views
{
    [Activity()]
    public class MyView : MvxActivity<MyViewModel>
    {

        protected override void OnViewModelSet()
        {
            base.OnViewModelSet();
        }


        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            SetContentView(Resource.Layout.topViewId);

            Button btnSave = FindViewById<Button>(Resource.Id.btnSave);
            btnSave.Click += async delegate {

                var myList = FindViewById<MvxRecyclerView>(Resource.Id.listone);

                int i = 0;
                foreach (MyItem item in myList.ItemsSource)
                {
                    try
                    {
                        View myView = myList.GetChildAt(i);

                        signaturePad = myList.FindViewById<SignaturePadView>(Resource.Id.signatureImage);

                        var isBlank = signaturePad.IsBlank;
                        if (!isBlank)
                        {
                            var bitmapImage = signaturePad.GetImage();

                            var stream = new MemoryStream();
                            bitmapImage.Compress(Android.Graphics.Bitmap.CompressFormat.Png, 100, stream);

                            signatory.SignatureImageByteArray = stream.ToArray();
                            signatory.SigningDate = DateTime.Now;
                        }
                    }
                    catch (Exception e)
                    {
                        string error = e.Message;
                    }

                    i++;
                }
            };
        }

```
