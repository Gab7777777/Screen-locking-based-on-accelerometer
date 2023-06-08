This script is an Android application that locks the screen based on accelerometer data. 
This script sets up an Android application that utilizes the accelerometer sensor to detect a fall (based on a threshold) and locks the screen if a fall is detected. It also handles requesting device admin permissions if necessary.

NOTE:
Ensure that the AdminReceiver class is properly implemented and registered in the manifest file.

Add checks to verify if the accelerometer sensor is available before registering it. 
You can use the getDefaultSensor() method to check if it returns null before registering.

Consider displaying a Toast message when the screen is locked to indicate to the user that the action has been successfully performed.

# Screen-locking-based-on-accelerometer











import android.app.admin.DevicePolicyManager; 
import android.content.ComponentName; 
import android.content.Context; 
import android.content.Intent; 
import android.hardware.Sensor; 
import android.hardware.SensorEvent; 
import android.hardware.SensorEventListener; 
import android.hardware.SensorManager; 
import android.os.Bundle; 
import android.widget.Toast; 

import androidx.appcompat.app.AppCompatActivity; 

public class MainActivity extends AppCompatActivity implements SensorEventListener {

    private SensorManager sensorManager;
    private Sensor accelerometer;
    private DevicePolicyManager devicePolicyManager;
    private ComponentName adminComponent;

    private static final float FALL_THRESHOLD = 9.8f; // Valor de aceleración para detectar caída

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        sensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
        accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);

        devicePolicyManager = (DevicePolicyManager) getSystemService(Context.DEVICE_POLICY_SERVICE);
        adminComponent = new ComponentName(this, AdminReceiver.class);
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

        // Si la aceleración supera el umbral de caída, bloquea la pantalla
        if (acceleration > FALL_THRESHOLD) {
            lockScreen();
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

    private void showToast(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
}



Ensure that the AdminReceiver class is properly implemented and registered in the manifest file.

Add checks to verify if the accelerometer sensor is available before registering it. You can use the getDefaultSensor() method to check if it returns null before registering.

Consider displaying a Toast message when the screen is locked to indicate to the user that the action has been successfully performed.
