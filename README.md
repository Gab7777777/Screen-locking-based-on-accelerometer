English:
I have some suggestions regarding the project.
1. Sound alarm when falling to immediately know when it fell
2. When try to hack the phone from the computer, it will be reset to factory settings so that the data does not get to the attacker.

Recommendations:
1. Make sure all required dependencies and libraries are properly included in your android project.
2. Verify that all required permissions are specified in your application's manifest file. For example, <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" /> may be required to access the acceleration sensor.
3. Make sure that all resources (sound file, layouts and other files) are available and correctly identified in the appropriate parts of the code.
4. Check that your device is set up for development via USB debugging and has the necessary drivers.
5. Make sure your device is connected to your computer with a USB cable and is correctly detected by the development environment.
6. Check that you have Android Debug Bridge (ADB) installed and configured to run adb shell commands on your device.
7. If necessary, adjust and check the device's system settings to ensure they match the expected values.

Русский:
У меня есть некоторые предложения, касаемо проекта. 
1. Звуковой сигнал при падении, чтобы сразу узнать когда он упал
2. При попытке взлома телефона с компьютера, будут сбрасываться до заводских настроек, чтобы данные не попали к злоумышленнику.

Рекомендации:
1. Убедитесь, что все необходимые зависимости и библиотеки правильно подключены к вашему проекту Android.
2. Проверьте, что все необходимые разрешения указаны в файле манифеста вашего приложения. Например, для доступа к датчику ускорения может потребоваться разрешение <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />.
3. Убедитесь, что все ресурсы (звуковой файл, макеты и другие файлы) доступны и корректно идентифицированы в соответствующих частях кода.
4. Проверьте, что ваше устройство настроено для разработки через USB-отладку и имеет необходимые драйверы.
5. Убедитесь, что ваше устройство подключено к компьютеру с помощью USB-кабеля и правильно обнаружено средой разработки.
6. Проверьте, что у вас установлен и настроен Android Debug Bridge (ADB) для выполнения команд adb shell на вашем устройстве.
7. При необходимости настройте и проверьте системные параметры устройства, чтобы убедиться, что они соответствуют ожидаемым значениям.


import android.content.Context;
import android.content.Intent;
import android.media.MediaPlayer;
import android.net.Uri;
import android.os.Bundle;
import android.os.Vibrator;
import android.support.v7.app.AppCompatActivity;
import android.widget.Toast;
import android.hardware.Sensor;
import android.hardware.SensorEvent;
import android.hardware.SensorEventListener;
import android.hardware.SensorManager;
import android.app.admin.DevicePolicyManager;
import android.content.ComponentName;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class MainActivity extends AppCompatActivity implements SensorEventListener {
    private SensorManager sensorManager;
    private Sensor accelerometer;
    private DevicePolicyManager devicePolicyManager;
    private ComponentName adminComponent;
    private MediaPlayer mediaPlayer;
    private Vibrator vibrator;

    private static final float FALL_THRESHOLD = 9.8f; // Valor de aceleración para detectar caída

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        sensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
        accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);

        devicePolicyManager = (DevicePolicyManager) getSystemService(Context.DEVICE_POLICY_SERVICE);
        adminComponent = new ComponentName(this, AdminReceiver.class);

        mediaPlayer = MediaPlayer.create(this, R.raw.alarm_sound); // Звуковой файл будильника
        vibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);

        checkRootAccess();
        checkCustomFirmware();
        checkUnlockedBootloader();
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (accelerometer != null) {
            sensorManager.registerListener(this, accelerometer, SensorManager.SENSOR_DELAY_NORMAL);
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        sensorManager.unregisterListener(this);
    }

    @Override
    public void onSensorChanged(SensorEvent event) {
        float x = event.values[0];
        float y = event.values[1];
        float z = event.values[2];

        // Calcula la aceleración resultante
        float acceleration = (float) Math.sqrt(x * x + y * y + z * z);

        // Si la aceleración supera el umbral de caída, bloquea la pantalla y reproduce el sonido del alarm
        if (acceleration > FALL_THRESHOLD) {
            lockScreen();
            playAlarmSound();
        }
    }

    @Override
    public void onAccuracyChanged(Sensor sensor, int accuracy) {
        // No se utiliza en este ejemplo
    }

    private void lockScreen() {
        boolean isAdmin = devicePolicyManager.isAdminActive(adminComponent);
        if (isAdmin) {
            devicePolicyManager.lockNow();
            showToast("Pantalla bloqueada");
        } else {
            // Si la aplicación no tiene permisos de administrador de dispositivo,
            // se abre la configuración de administración de dispositivos para solicitarlos
            Intent intent = new Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN);
            intent.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN, adminComponent);
            intent.putExtra(DevicePolicyManager.EXTRA_ADD_EXPLANATION, "Se requieren permisos de administrador de dispositivo");
            startActivity(intent);
        }
    }

    private void playAlarmSound() {
        if (mediaPlayer != null && !mediaPlayer.isPlaying()) {
            mediaPlayer.start();
        }
        if (vibrator != null && vibrator.hasVibrator()) {
            long[] pattern = {0, 1000, 1000

}; // Vibration pattern: stop, vibrate, pause
            vibrator.vibrate(pattern, 0);
        }
    }

    private void stopAlarmSound() {
        if (mediaPlayer != null && mediaPlayer.isPlaying()) {
            mediaPlayer.stop();
            mediaPlayer.reset();
        }
        if (vibrator != null && vibrator.hasVibrator()) {
            vibrator.cancel();
        }
    }

    private void showToast(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mediaPlayer != null) {
            mediaPlayer.release();
        }
    }

    private void checkRootAccess() {
        try {
            Process process = Runtime.getRuntime().exec("adb shell which su");
            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line = reader.readLine();
            if (line != null) {
                System.out.println("Device has root access");
                resetDeviceSettings();
                return;
            }

            process = Runtime.getRuntime().exec("adb shell su -c id");
            reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            line = reader.readLine();
            if (line != null && line.contains("uid=0")) {
                System.out.println("Device has root access");
                resetDeviceSettings();
            } else {
                System.out.println("Root access not confirmed");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void checkCustomFirmware() {
        try {
            Process process = Runtime.getRuntime().exec("adb shell getprop ro.build.display.id");
            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line = reader.readLine();
            if (line != null && line.toLowerCase().contains("custom")) {
                System.out.println("The device runs on custom firmware");
                resetDeviceSettings();
            } else {
                System.out.println("The device runs on official firmware");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void checkUnlockedBootloader() {
        try {
            Process process = Runtime.getRuntime().exec("adb shell getprop ro.bootloader");
            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line = reader.readLine();
            if (line != null && line.toLowerCase().contains("unlocked")) {
                System.out.println("Device bootloader unlocked");
                resetDeviceSettings();
            } else {
                System.out.println("Device bootloader locked");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void resetDeviceSettings() {
        System.out.println("Your phone is being reset for security purposes");
        try {
            Process process = Runtime.getRuntime().exec("adb shell recovery --wipe_data");
            int exitCode = process.waitFor();
            if (exitCode == 0) {
                System.out.println("Reset completed successfully");
            } else {
                System.out.println("Error when performing factory reset");
            }
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
