# 📸 CameraX Demo App

A feature-rich Android application demonstrating how to use the [CameraX Jetpack library](https://developer.android.com/training/camerax) to interact with the device camera.

This project serves as a practical example for students and developers learning how to implement camera functionality, handle runtime permissions, and save media using `MediaStore`.

---

## 🚀 Features

* **Live Camera Preview:** Displays a real-time feed from the camera.
* **Photo Capture:** Takes high-quality photos and saves them to the device's public "Pictures" gallery.
* **Video Recording:** Records video (with audio) and saves it to the "Movies" folder.
* **Image Analysis:** Performs real-time frame analysis to calculate the average luminosity (brightness) of the current scene.
* **Permission Handling:** Demonstrates best practices for requesting Camera and Audio permissions at runtime.

---

## 🛠 Tech Stack

* **Language:** Kotlin
* **Camera Library:** Android Jetpack CameraX
* **UI:** XML Layouts with ViewBinding (Note: Project includes Compose theme files, but the main camera UI uses standard Views).
* **Concurrency:** `java.util.concurrent.ExecutorService` for handling camera operations off the main thread.
* **Storage:** `MediaStore` API (Scoped Storage compliant).

---

## 🔐 Permissions Explained

To access hardware features, this app declares the following permissions in `AndroidManifest.xml`:

```xml
<uses-feature android:name="android.hardware.camera.any" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="28" />

```

* **CAMERA:** Required to see the preview and capture content.
* **RECORD_AUDIO:** Required specifically for recording sound with video.
* **WRITE_EXTERNAL_STORAGE:** Required for saving files on older Android versions (SDK < 29). On newer Android versions, we use Scoped Storage via `MediaStore`, so this permission is not needed.

---

##🧑‍💻 How It Works (Code Highlights)###1. Starting the CameraThe core logic resides in `MainActivity.kt`. We use the `ProcessCameraProvider` to bind specific "Use Cases" to the lifecycle of the Activity. This ensures the camera opens when the app starts and closes when the app stops, preventing battery drain.

```kotlin
// From MainActivity.kt
val cameraProvider: ProcessCameraProvider = cameraProviderFuture.get()

// We bind 4 distinct use cases:
cameraProvider.bindToLifecycle(
    this,
    CameraSelector.DEFAULT_BACK_CAMERA,
    preview,       // The live viewfinder
    imageCapture,  // For taking photos
    videoCapture,  // For recording video
    imageAnalyzer  // For calculating luminosity
)

```

###2. Capturing Photos (`ImageCapture`)We create an `ImageCapture` use case. When the user taps the button, we define output options (where to save the file) and call `takePicture`. The result is saved to `MediaStore.Images.Media.EXTERNAL_CONTENT_URI`.

###3. Recording Video (`VideoCapture`)We use the `Recorder` and `Recording` classes.

* **Audio Handling:** The app explicitly checks for `Manifest.permission.RECORD_AUDIO` before enabling audio in the recording.
* **State Management:** The UI updates (Red button vs. Blue button) based on `VideoRecordEvent` (Start/Finalize).

###4. Image Analysis (`LuminosityAnalyzer`)This is a powerful feature of CameraX. We implement a custom `ImageAnalysis.Analyzer` that runs on every frame.

* It accesses the raw buffer of the image.
* It calculates the average pixel intensity.
* It updates a text view with the "Luminosity" score in real-time.

```kotlin
// Our custom analyzer logic
private class LuminosityAnalyzer(private val listener: LumaListener) : ImageAnalysis.Analyzer {
    override fun analyze(image: ImageProxy) {
        // Access raw byte buffer
        val buffer = image.planes[0].buffer
        // ... Calculate average ...
        listener(luma) // Send data back to UI
        image.close() // Important! You must close the image to receive the next frame.
    }
}

```

---

##📱 How to Run
1. **Clone the repository** and open it in Android Studio.
2. **Sync Gradle** to ensure dependencies are downloaded.
3. **Run on a physical device** (Recommended).
* *Note: While the Android Emulator supports camera features, performance is better on a real device.*

4. **Grant Permissions:** When prompted, allow the app to take pictures and record video.
5. **Test:**
* Tap the **"Take Photo"** button.
* Tap the **"Start Capture"** button to record video, and tap again to stop.
* Observe the "Luminosity" text updating as you point the camera at light or dark areas.



---

##⚠️ Troubleshooting* **"Permission Denied":** If you denied permissions permanently, go to the App Info settings on your phone and manually enable Camera and Microphone access.
* **Video Save Error:** Ensure your device has enough storage space. The `MediaStore` output can fail if the disk is full.

