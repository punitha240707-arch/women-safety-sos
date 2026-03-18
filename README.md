// Advanced Women Safety SOS App (Kotlin)
// Includes: SOS, Shake Detection, Voice Trigger (basic), Alarm, Fake Call

package com.example.womensafety

import android.Manifest
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.hardware.Sensor
import android.hardware.SensorEvent
import android.hardware.SensorEventListener
import android.hardware.SensorManager
import android.location.Location
import android.media.MediaPlayer
import android.net.Uri
import android.os.Bundle
import android.speech.RecognizerIntent
import android.telephony.SmsManager
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import com.google.android.gms.location.FusedLocationProviderClient
import com.google.android.gms.location.LocationServices

class MainActivity : AppCompatActivity(), SensorEventListener {

    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var sensorManager: SensorManager
    private var lastShakeTime: Long = 0

    private val contacts = listOf("+911234567890", "+919876543210")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)

        // Shake detection setup
        sensorManager = getSystemService(Context.SENSOR_SERVICE) as SensorManager
        val accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)
        sensorManager.registerListener(this, accelerometer, SensorManager.SENSOR_DELAY_NORMAL)

        val sosButton = findViewById<Button>(R.id.sosButton)
        val alarmButton = findViewById<Button>(R.id.alarmButton)
        val fakeCallButton = findViewById<Button>(R.id.fakeCallButton)
        val voiceButton = findViewById<Button>(R.id.voiceButton)

        sosButton.setOnClickListener { sendSOS() }
        alarmButton.setOnClickListener { playAlarm() }
        fakeCallButton.setOnClickListener { fakeCall() }
        voiceButton.setOnClickListener { startVoiceRecognition() }
    }

    // ---------------- SOS FUNCTION ----------------
    private fun sendSOS() {
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), 1)
            return
        }

        fusedLocationClient.lastLocation.addOnSuccessListener { location: Location? ->
            val message = if (location != null) {
                "HELP! I am in danger. My location: https://maps.google.com/?q=${location.latitude},${location.longitude}"
            } else {
                "HELP! I am in danger. Location not found"
            }

            sendSMS(message)
            makeCall()
        }
    }

    private fun sendSMS(message: String) {
        val smsManager = SmsManager.getDefault()
        for (number in contacts) {
            smsManager.sendTextMessage(number, null, message, null, null)
        }
        Toast.makeText(this, "SOS Sent!", Toast.LENGTH_SHORT).show()
    }

    private fun makeCall() {
        val intent = Intent(Intent.ACTION_CALL)
        intent.data = Uri.parse("tel:${contacts[0]}")

        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.CALL_PHONE), 2)
            return
        }
        startActivity(intent)
    }

    // ---------------- SHAKE DETECTION ----------------
    override fun onSensorChanged(event: SensorEvent?) {
        val x = event!!.values[0]
        val y = event.values[1]
        val z = event.values[2]

        val acceleration = Math.sqrt((x * x + y * y + z * z).toDouble())

        if (acceleration > 15) {
            val currentTime = System.currentTimeMillis()
            if (currentTime - lastShakeTime > 2000) {
                lastShakeTime = currentTime
                Toast.makeText(this, "Shake detected! Sending SOS", Toast.LENGTH_SHORT).show()
                sendSOS()
            }
        }
    }

    override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {}

    // ---------------- VOICE RECOGNITION ----------------
    private fun startVoiceRecognition() {
        val intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH)
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
        intent.putExtra(RecognizerIntent.EXTRA_PROMPT, "Say HELP to trigger SOS")
        startActivityForResult(intent, 100)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)

        if (requestCode == 100 && resultCode == RESULT_OK) {
            val result = data?.getStringArrayListExtra(RecognizerIntent.EXTRA_RESULTS)
            if (result != null && result[0].contains("help", true)) {
                Toast.makeText(this, "Voice SOS Triggered!", Toast.LENGTH_SHORT).show()
                sendSOS()
            }
        }
    }

    // ---------------- ALARM ----------------
    private fun playAlarm() {
        val mediaPlayer = MediaPlayer.create(this, R.raw.alarm)
        mediaPlayer.start()
    }

    // ---------------- FAKE CALL ----------------
    private fun fakeCall() {
        val intent = Intent(Intent.ACTION_DIAL)
        intent.data = Uri.parse("tel:100")
        startActivity(intent)
    }
}

// ---------------- UI XML ----------------
/*
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:background="#000000">

    <Button
        android:id="@+id/sosButton"
        android:layout_width="220dp"
        android:layout_height="220dp"
        android:text="SOS"
        android:textSize="32sp"
        android:backgroundTint="#FF0000"
        android:textColor="#FFFFFF" />

    <Button
        android:id="@+id/alarmButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Alarm" />

    <Button
        android:id="@+id/fakeCallButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Fake Call" />

    <Button
        android:id="@+id/voiceButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Voice SOS" />

</LinearLayout>
*/

// ---------------- PERMISSIONS ----------------
/*
<uses-permission android:name="android.permission.SEND_SMS" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.CALL_PHONE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
*/

