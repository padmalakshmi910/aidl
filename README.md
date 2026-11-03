# Ex.No:2 Develop an android application to implement the AIDL server and client app. The server app hosts a Bound Service and contains the logic to return random colours to its client.The client app calls the service and changes the button's colour within the Main activity.



## AIM:

To Develop an android application to implement the AIDL server and client app. The server app hosts a Bound Service and contains the logic to return random colours to its client.
The client app calls the service and changes the button's colour within the Main activity using AIDL interface in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Griaffe )

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as CSAIDL and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display message give in MainActivity file(client/server).

Step 7: Save and run the application.

## PROGRAM:
```
/*
Program to print the client/server services using AIDL”.
Developed by:G.Padma Lakshmi
Registeration Number :212225230206
*/
```
### SERVER:

## AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>

<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:label="Smart Parking System"
        android:supportsRtl="true"
        android:theme="@style/Theme.SmartParking">

        <service
            android:name="com.example.aidlserver.SmartParkingService"
            android:enabled="true"
            android:exported="true">

            <intent-filter>
                <action
                    android:name="com.example.aidlserver.ISmartParkingService" />
            </intent-filter>

        </service>

    </application>

</manifest>
```

## SmartParkingService.java

```
package com.example.aidlserver;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;

import java.util.ArrayList;

public class SmartParkingService extends Service {

    private static final int TOTAL_SLOTS = 5;

    private final ArrayList<String> parkedVehicles =
            new ArrayList<>();

    private final ISmartParkingService.Stub binder =
            new ISmartParkingService.Stub() {

                @Override
                public void basicTypes(
                        int anInt,
                        long aLong,
                        boolean aBoolean,
                        float aFloat,
                        double aDouble,
                        String aString) {
                }

                @Override
                public synchronized int getAvailableSlots() {
                    return TOTAL_SLOTS - parkedVehicles.size();
                }

                @Override
                public synchronized boolean parkVehicle(String vehicleNumber) {

                    if (vehicleNumber == null ||
                            vehicleNumber.trim().isEmpty()) {
                        return false;
                    }

                    String number =
                            vehicleNumber.trim().toUpperCase();

                    if (parkedVehicles.contains(number) ||
                            parkedVehicles.size() >= TOTAL_SLOTS) {
                        return false;
                    }

                    parkedVehicles.add(number);
                    return true;
                }

                @Override
                public synchronized boolean removeVehicle(String vehicleNumber) {

                    if (vehicleNumber == null) {
                        return false;
                    }

                    return parkedVehicles.remove(
                            vehicleNumber.trim().toUpperCase());
                }

                @Override
                public synchronized String getParkedVehicles() {

                    if (parkedVehicles.isEmpty()) {
                        return "No vehicles parked.";
                    }

                    return String.join("\n", parkedVehicles);
                }
            };

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
}
```

## ISmartParkingService.aidl

```
package com.example.aidlserver;

interface ISmartParkingService {

    void basicTypes(
        int anInt,
        long aLong,
        boolean aBoolean,
        float aFloat,
        double aDouble,
        String aString
    );

    int getAvailableSlots();

    boolean parkVehicle(String vehicleNumber);

    boolean removeVehicle(String vehicleNumber);

    String getParkedVehicles();
}
```

### CLIENT:

## AndroidManifest.xml

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <queries>
        <package android:name="com.example.aidlserver" />
    </queries>
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="Smart Parking Client"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Aidlclient"
        tools:targetApi="31">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>


    </application>

</manifest>
```

## MainActivity.java

```
package com.example.aidlclient;

import android.app.Activity;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import com.example.aidlserver.ISmartParkingService;

public class MainActivity extends Activity {

    private ISmartParkingService parkingService;

    private TextView txtStatus;
    private TextView txtSlots;
    private TextView txtVehicles;
    private EditText edtVehicle;

    private boolean isBound = false;

    private final ServiceConnection connection =
            new ServiceConnection() {

                @Override
                public void onServiceConnected(
                        ComponentName name, IBinder service) {

                    parkingService =
                            ISmartParkingService.Stub.asInterface(service);

                    isBound = true;

                    txtStatus.setText("Connected to Smart Parking Server");

                    refreshDetails();
                }

                @Override
                public void onServiceDisconnected(ComponentName name) {
                    parkingService = null;
                    isBound = false;
                    txtStatus.setText("Server disconnected");
                }
            };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        txtStatus = findViewById(R.id.txtStatus);
        txtSlots = findViewById(R.id.txtSlots);
        txtVehicles = findViewById(R.id.txtVehicles);
        edtVehicle = findViewById(R.id.edtVehicle);

        Button btnPark = findViewById(R.id.btnPark);
        Button btnRemove = findViewById(R.id.btnRemove);
        Button btnRefresh = findViewById(R.id.btnRefresh);

        btnPark.setOnClickListener(v -> parkVehicle());
        btnRemove.setOnClickListener(v -> removeVehicle());
        btnRefresh.setOnClickListener(v -> refreshDetails());
    }

    @Override
    protected void onStart() {
        super.onStart();

        Intent intent = new Intent(
                "com.example.aidlserver.ISmartParkingService");

        intent.setPackage("com.example.aidlserver");

        boolean bound = bindService(
                intent,
                connection,
                Context.BIND_AUTO_CREATE
        );

        if (!bound) {
            txtStatus.setText("Unable to connect to server");
        }
    }

    private void parkVehicle() {

        if (parkingService == null) {
            showMessage("Server is not connected");
            return;
        }

        String vehicle =
                edtVehicle.getText().toString().trim();

        if (vehicle.isEmpty()) {
            edtVehicle.setError("Enter vehicle number");
            return;
        }

        try {
            boolean success =
                    parkingService.parkVehicle(vehicle);

            if (success) {
                showMessage("Vehicle parked successfully");
                edtVehicle.setText("");
            } else {
                showMessage(
                        "Parking failed: vehicle already parked or no slots available");
            }

            refreshDetails();

        } catch (RemoteException e) {
            showMessage("Server communication error");
        }
    }

    private void removeVehicle() {

        if (parkingService == null) {
            showMessage("Server is not connected");
            return;
        }

        String vehicle =
                edtVehicle.getText().toString().trim();

        if (vehicle.isEmpty()) {
            edtVehicle.setError("Enter vehicle number");
            return;
        }

        try {
            boolean success =
                    parkingService.removeVehicle(vehicle);

            if (success) {
                showMessage("Vehicle removed successfully");
                edtVehicle.setText("");
            } else {
                showMessage("Vehicle not found");
            }

            refreshDetails();

        } catch (RemoteException e) {
            showMessage("Server communication error");
        }
    }

    private void refreshDetails() {

        if (parkingService == null) {
            return;
        }

        try {
            int availableSlots =
                    parkingService.getAvailableSlots();

            txtSlots.setText(
                    "Available Slots: " + availableSlots);

            txtVehicles.setText(
                    parkingService.getParkedVehicles());

        } catch (RemoteException e) {
            showMessage("Unable to refresh parking details");
        }
    }

    private void showMessage(String message) {
        Toast.makeText(
                MainActivity.this,
                message,
                Toast.LENGTH_SHORT
        ).show();
    }

    @Override
    protected void onStop() {
        if (isBound) {
            unbindService(connection);
            isBound = false;
        }

        parkingService = null;

        super.onStop();
    }
}
```

## ISmartParkingService.aidl

```
package com.example.aidlserver;

interface ISmartParkingService {

    void basicTypes(
        int anInt,
        long aLong,
        boolean aBoolean,
        float aFloat,
        double aDouble,
        String aString
    );

    int getAvailableSlots();

    boolean parkVehicle(String vehicleNumber);

    boolean removeVehicle(String vehicleNumber);

    String getParkedVehicles();
}
```

## activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>

<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true"
    android:background="#F2F7F4">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="20dp">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="SMART PARKING"
            android:textSize="28sp"
            android:textStyle="bold"
            android:textColor="#176B45"
            android:gravity="center"
            android:padding="16dp" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Manage your parking easily"
            android:textSize="16sp"
            android:gravity="center"
            android:textColor="#555555"
            android:paddingBottom="20dp" />

        <TextView
            android:id="@+id/txtStatus"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Connecting to server..."
            android:textSize="16sp"
            android:textStyle="bold"
            android:textColor="#444444"
            android:padding="12dp"
            android:background="#FFFFFF" />

        <TextView
            android:id="@+id/txtSlots"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Available Slots: --"
            android:textSize="23sp"
            android:textStyle="bold"
            android:textColor="#176B45"
            android:gravity="center"
            android:padding="20dp"
            android:layout_marginTop="16dp"
            android:background="#FFFFFF" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Vehicle Number"
            android:textSize="18sp"
            android:textStyle="bold"
            android:textColor="#222222"
            android:paddingTop="24dp" />

        <EditText
            android:id="@+id/edtVehicle"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Example: TN01AB1234"
            android:inputType="textCapCharacters"
            android:singleLine="true"
            android:padding="12dp"
            android:backgroundTint="#176B45" />

        <Button
            android:id="@+id/btnPark"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Park Vehicle"
            android:textStyle="bold"
            android:textColor="#FFFFFF"
            android:backgroundTint="#176B45"
            android:layout_marginTop="12dp" />

        <Button
            android:id="@+id/btnRemove"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Remove Vehicle"
            android:textStyle="bold"
            android:textColor="#FFFFFF"
            android:backgroundTint="#C62828"
            android:layout_marginTop="8dp" />

        <Button
            android:id="@+id/btnRefresh"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Refresh Parking Details"
            android:textStyle="bold"
            android:layout_marginTop="8dp" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Parked Vehicles"
            android:textSize="21sp"
            android:textStyle="bold"
            android:textColor="#176B45"
            android:paddingTop="24dp"
            android:paddingBottom="10dp" />

        <TextView
            android:id="@+id/txtVehicles"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="No vehicles parked."
            android:textSize="17sp"
            android:textColor="#333333"
            android:padding="16dp"
            android:background="#FFFFFF"
            android:minHeight="70dp" />

    </LinearLayout>

</ScrollView>
```


## OUTPUT

<img width="317" height="706" alt="Screenshot 2026-11-03 103039" src="https://github.com/user-attachments/assets/b324d30b-ea60-4270-8293-e495ab93aafa" />



## RESULT
Thus a Simple Android Application to create a AIDL interface and communicate the process between client and server using AIDL interface in Android Studio is developed and executed successfully.
