---
layout: post
title:  "Building a mock location app for Android."
date:   2016-09-02 15:30:00 +0800
categories: android
tags: [android, gps, location, spoofing]
---
<p>Recently I've been asked to test the android app I'm developing for my client.
The app is called SurveyBods and owned by the company called Researchbods and it is only available in the UK.
A notification shows up whenever the user enters a place which is represented by a geofence.
The notification would prompt the user with a survey or a quick poll.
My problem with this is that I'm not allowed to get out of my office and using DDMS is a very tedious task so I decided to make a mock location app to make it more exciting for me.</p>

<p>In this article I will show you how to build a simple mock location app.
First we will create the layout for our main activity.
The layout will contain the required input for latitude,longitude and accuracy.</p>

<h4>activity_main.xml</h4>

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.emmanuelcorrales.locationspoofer.MainActivity">
    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <EditText
            android:id="@+id/latitude"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:ems="10"
            android:hint="@string/latitude"
            android:inputType="numberDecimal|numberSigned" />
    </android.support.design.widget.TextInputLayout>
    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <EditText
            android:id="@+id/longitude"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:ems="10"
            android:hint="@string/longitude"
            android:inputType="numberDecimal|numberSigned" />
    </android.support.design.widget.TextInputLayout>
    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <EditText
            android:id="@+id/accuracy"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:ems="10"
            android:hint="@string/accuracy"
            android:inputType="numberDecimal" />
    </android.support.design.widget.TextInputLayout>
    <Button
        android:id="@+id/mock_location"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="OK" />
</LinearLayout>
{% endhighlight %}

<p>Next we create our MainActivity.</p>

<h4>MainActivity.java</h4>
{% highlight java linenos %}
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private LocationManager mLocationManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button mockLocationBtn = (Button) findViewById(R.id.mock_location);
        mockLocationBtn.setOnClickListener(this);

        mLocationManager = (LocationManager) getSystemService(LOCATION_SERVICE);
        mLocationManager.addTestProvider(
                LocationManager.GPS_PROVIDER,
                false,
                false,
                false,
                false,
                true,
                true,
                true,
                0,
                5
        );
        mLocationManager.setTestProviderEnabled(LocationManager.GPS_PROVIDER, true);
    }

    private void mockLocation(double latitude, double longitude, float accuracy) {
        Location mockLocation = new Location(LocationManager.GPS_PROVIDER);
        mockLocation.setLatitude(latitude);
        mockLocation.setLongitude(longitude);
        mockLocation.setAccuracy(accuracy);
        mockLocation.setTime(System.currentTimeMillis());
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN) {
            mockLocation.setElapsedRealtimeNanos(SystemClock.elapsedRealtimeNanos());
        }
        mLocationManager.setTestProviderLocation(LocationManager.GPS_PROVIDER, mockLocation);
    }

    private LocationManager getLocationManager() {
        return (LocationManager) getSystemService(LOCATION_SERVICE);
    }

    @Override
    public void onClick(View v) {
        EditText latitudeEt = (EditText) findViewById(R.id.latitude);
        EditText longitudeEt = (EditText) findViewById(R.id.longitude);
        EditText accuracyEt = (EditText) findViewById(R.id.accuracy);
        String message;
        if (validateEditText(latitudeEt) | validateEditText(longitudeEt)) {
            double latitude = Double.valueOf(latitudeEt.getText().toString());
            double longitude = Double.valueOf(longitudeEt.getText().toString());
            float accuracy = Float.valueOf(accuracyEt.getText().toString());
            mockLocation(latitude, longitude, accuracy);
            message = "The location has been changed to (" + latitude + "," + longitude + ")";
        } else {
            message = "Spoofing the location has failed.";
        }
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }

    private boolean validateEditText(EditText editText) {
        if (editText == null) {
            throw new IllegalArgumentException("Argument 'editText' cannot be null.");
        }

        if (editText.getText().toString().isEmpty()) {
            editText.setError(getString(R.string.validation_required));
            return false;
        }
        return true;
    }
}
{% endhighlight %}

<p>Then on the manifest we add the mock location permission.</p>

{% highlight xml %}
<uses-permission android:name="android.permission.ACCESS_MOCK_LOCATION" />
{% endhighlight %}

<p>Finally, you will need to enable mock locations for the device using Settings > Developer options > Allow mock locations. After building the app and trying it out, you can check if your location has changed using the Google Maps app.</p>
