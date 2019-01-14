---
title: Android 7.0 进度条显示 
categories: [blog, android]
description: 记录的是随心所欲
keywords: android, work
---

# Android 7.0 进度条显示 

Android 7.0 去掉了很多 ProgressDialog 的方法，导致网上的定制教程无法使用，所以最好的办法就是最原始的办法：

## 继承 ProgressDialog 自定义自己的 ProgressDialog 

```java
 public class XmlProgressDialog extends ProgressDialog {

        TextView mTextView;
        TextView mTitleView;
        ProgressBar mProgressBar;

        public XmlProgressDialog(Context context) {
            super(context);
        }

        public XmlProgressDialog(Context context, int theme) {
            super(context, theme);
        }

        @Override
        protected void onCreate(Bundle savedInstanceState)
        {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.progress_dialog_layout);

            mTitleView = (TextView) findViewById(R.id.dialog_title);
            mTextView = (TextView) findViewById(R.id.progress_label);
            mProgressBar = (ProgressBar) findViewById(R.id.progressBar);
        }

        @Override
        public void setProgress(int value) {
            super.setProgress(value);
            mTextView.setText(value + "%");
            mProgressBar.setProgress(value);
        }

        @Override
        public void setTitle(CharSequence title) {
            super.setTitle(title);
            mTitleView.setText(title);
        }
    }
```

## 后台显示进度使用 AsyncTask 

```
 class xmlProgress extends AsyncTask<String, String, String> {

        String title;
        int i = 0;

        ProgressDialog mXmlProgressDialog = new XmlProgressDialog(MainActivity.this, ProgressDialog.STYLE_HORIZONTAL);

        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            mXmlProgressDialog.show();
        }


        @Override
        protected String doInBackground(String... aurl) {
            title = aurl[0];

            while (i++ < 100)
            {
                try {
                    Thread.sleep(200);
                    publishProgress("" + i);
                } catch (Exception e) {
                    Log.d("log", "e:" + e.toString());
                }

            }
            return null;
        }

        protected void onProgressUpdate(String... progress) {
            Log.d("log",progress[0]);
            mXmlProgressDialog.setProgress(i);
            mXmlProgressDialog.setTitle(title);
        }

        @Override
        protected void onPostExecute(String unused) {
            //dismiss the dialog after the file was downloaded
            mXmlProgressDialog.dismiss();
        }
    }
```

## 调用的时候，直接调用 exec 接口：

```java
        new xmlProgress().execute("xml 解析进度");
```

## xml 配置为：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:minWidth="250dp"
    android:orientation="vertical" >

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:orientation="vertical" >

        <TextView
            android:id="@+id/dialog_title"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:gravity="center_horizontal|center"
            android:text="进度"
            android:textSize="20sp"
            tools:ignore="HardcodedText" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:orientation="horizontal">

        <ProgressBar
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/progressBar"
            android:layout_weight="5"/>

        <TextView
            android:id="@+id/progress_label"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            />
    </LinearLayout>



</LinearLayout>
```

最后显示如下：

![启动完成](/images/blog/device_show_xml_progress.png)

