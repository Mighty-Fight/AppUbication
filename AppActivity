package com.example.ubicacionaplicattion.Ubicatedapp


import android.Manifest
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.location.Location
import android.location.LocationListener
import android.location.LocationManager
import android.os.Build
import android.os.Bundle
import android.os.Handler
import android.provider.Settings
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.cardview.widget.CardView
import androidx.core.app.ActivityCompat
import com.example.ubicacionaplicattion.R
import java.net.DatagramPacket
import java.net.DatagramSocket
import java.net.InetAddress
import java.text.SimpleDateFormat
import java.util.*

class AppActivity : AppCompatActivity() {

    private lateinit var gpsInfoTextView: TextView
    private lateinit var locationManager: LocationManager
    private lateinit var locationListener: LocationListener

    private companion object {
        private const val REQUEST_LOCATION_PERMISSION = 1
        private const val MIN_TIME_BETWEEN_UPDATES = 1000L // 1 segundo
        private const val MIN_DISTANCE_FOR_UPDATES = 1.0f // 1 metro
    }

    private var isSending = false
    private val handler = Handler()
    private val intervalMillis: Long = 5000  // Intervalo de envío en milisegundos

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_app)

        // Obtener referencias a las vistas
        val cardViewTitulo: CardView = findViewById(R.id.Titulo)
        val cardViewGPSInfo: CardView = findViewById(R.id.GPSINFO)
        gpsInfoTextView = findViewById(R.id.Infogps)

        // Inicializar LocationManager y LocationListener
        locationManager = getSystemService(Context.LOCATION_SERVICE) as LocationManager
        locationListener = object : LocationListener {
            override fun onLocationChanged(location: Location) {
                // Actualizar la ubicación en el TextView cuando cambie
                updateLocationTextView(location)
            }

            override fun onStatusChanged(provider: String?, status: Int, extras: Bundle?) {}

            override fun onProviderEnabled(provider: String) {}

            override fun onProviderDisabled(provider: String) {
                // Si el proveedor de ubicación está desactivado, mostrar un mensaje al usuario
                showLocationSettingsAlert()
            }
        }

        // Verificar y solicitar permisos de ubicación
        if (checkLocationPermission()) {
            // Si los permisos ya están concedidos, registrar el escuchador de ubicación
            startLocationUpdates()
        } else {
            // Si no es necesario solicitar permisos, obtener la ubicación y mostrarla
            getLocation()
        }

        // Configurar el botón para enviar el paquete UDP
        val sendButton: Button = findViewById(R.id.packetsender)
        sendButton.setOnClickListener {
            sendpacket(it)
        }
    }

    private fun checkLocationPermission(): Boolean {
        // Verificar permisos de ubicación
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (ActivityCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_FINE_LOCATION
                ) != PackageManager.PERMISSION_GRANTED
            ) {
                // Solicitar permisos si no están concedidos
                requestLocationPermission()
                return false
            }
        }
        return true
    }

    private fun requestLocationPermission() {
        ActivityCompat.requestPermissions(
            this,
            arrayOf(Manifest.permission.ACCESS_FINE_LOCATION),
            REQUEST_LOCATION_PERMISSION
        )
    }

    private fun getLocation() {
        // Obtener la última ubicación conocida y mostrarla en el TextView
        try {
            if (ActivityCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_FINE_LOCATION
                ) == PackageManager.PERMISSION_GRANTED
            ) {
                val lastKnownLocation =
                    locationManager.getLastKnownLocation(LocationManager.GPS_PROVIDER)
                lastKnownLocation?.let {
                    updateLocationTextView(it)
                } ?: run {
                    gpsInfoTextView.text = "No se pudo obtener la ubicación"
                }
            } else {
                gpsInfoTextView.text = "Permiso de ubicación denegado"
            }
        } catch (e: SecurityException) {
            e.printStackTrace()
            gpsInfoTextView.text = "Error al obtener la ubicación"
        }
    }

    private fun startLocationUpdates() {
        // Registrar el escuchador de ubicación para actualizaciones en tiempo real
        try {
            if (ActivityCompat.checkSelfPermission(
                    this,
                    Manifest.permission.ACCESS_FINE_LOCATION
                ) == PackageManager.PERMISSION_GRANTED
            ) {
                locationManager.requestLocationUpdates(
                    LocationManager.GPS_PROVIDER,
                    MIN_TIME_BETWEEN_UPDATES,
                    MIN_DISTANCE_FOR_UPDATES,
                    locationListener
                )
            }
        } catch (e: SecurityException) {
            e.printStackTrace()
            gpsInfoTextView.text = "Error al solicitar actualizaciones de ubicación"
        }
    }

    private fun updateLocationTextView(location: Location) {
        // Actualizar el TextView con la nueva ubicación y la fecha y hora
        val formattedLocation = "Latitud: ${location.latitude}\nLongitud: ${location.longitude}"
        val formattedDateTime = "Fecha y Hora: ${getCurrentDateTime()}"
        gpsInfoTextView.text = "$formattedLocation\n$formattedDateTime"
    }

    private fun showLocationSettingsAlert() {
        // Mostrar un cuadro de diálogo al usuario indicando que el proveedor de ubicación está desactivado
        val alertDialog: AlertDialog.Builder = AlertDialog.Builder(this)
        alertDialog.setTitle("Configuración de ubicación")
        alertDialog.setMessage("El proveedor de ubicación está desactivado. ¿Desea activarlo?")
        alertDialog.setPositiveButton(
            "Configuración"
        ) { _, _ ->
            val intent = Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS)
            startActivity(intent)
        }
        alertDialog.setNegativeButton(
            "Cancelar"
        ) { dialog, _ -> dialog.cancel() }
        alertDialog.show()
    }

    private fun getCurrentDateTime(): String {
        val currentDateTime = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault()).format(Date())
        return currentDateTime
    }

    private fun sendpacket(view: View) {
        // Cambiar el estado del botón
        isSending = !isSending

        if (isSending) {
            // Comenzar a enviar datos periódicamente
            startSendingData()
        } else {
            // Detener el envío periódico
            stopSendingData()
        }
    }

    private fun startSendingData() {
        // Obtener la dirección IP y el puerto desde los EditText
        val ipAddressEditText: EditText = findViewById(R.id.edittext)
        val portEditText: EditText = findViewById(R.id.PORT)
        val ipAddress = ipAddressEditText.text.toString()
        val port = portEditText.text.toString().toInt()

        // Iniciar el envío periódico de datos
        handler.post(object : Runnable {
            override fun run() {
                // Obtener la ubicación y la fecha y hora actual
                val location = gpsInfoTextView.text.toString()
                val dateTime = getCurrentDateTime()

                // Extraer la latitud, longitud, fecha y hora de la ubicación
                val (latitud, longitud) = extractLatLong(location)

                // Construir el mensaje con un espacio entre los datos
                val message = "$latitud $longitud $dateTime"

                // Enviar el mensaje a través de UDP
                Thread {
                    try {
                        val socket = DatagramSocket()
                        val ipAddressBytes = InetAddress.getByName(ipAddress).address
                        val packet = DatagramPacket(
                            message.toByteArray(),
                            message.length,
                            InetAddress.getByAddress(ipAddressBytes),
                            port
                        )
                        socket.send(packet)
                        socket.close()
                    } catch (e: Exception) {
                        e.printStackTrace()
                    }
                }.start()

                // Programar la siguiente ejecución después de 5 segundos
                handler.postDelayed(this, intervalMillis)
            }
        })
    }

    private fun stopSendingData() {
        // Detener el envío periódico de datos
        handler.removeCallbacksAndMessages(null)
    }

    private fun extractLatLong(location: String): Pair<String, String> {
        // Asumiendo que la cadena de ubicación tiene el formato "Latitud: xx.xx\nLongitud: yy.yy"
        val latitudRegex = "Latitud: ([-+]?[0-9]*\\.?[0-9]+)".toRegex()
        val longitudRegex = "Longitud: ([-+]?[0-9]*\\.?[0-9]+)".toRegex()

        val latitudMatch = latitudRegex.find(location)
        val longitudMatch = longitudRegex.find(location)

        val latitud = latitudMatch?.groupValues?.get(1) ?: "Latitud no encontrada"
        val longitud = longitudMatch?.groupValues?.get(1) ?: "Longitud no encontrada"

        return Pair(latitud, longitud)
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)

        if (requestCode == REQUEST_LOCATION_PERMISSION) {
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // Permiso concedido, registrar el escuchador de ubicación
                startLocationUpdates()
            } else {
                gpsInfoTextView.text = "Permiso de ubicación denegado"
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        // Detener las actualizaciones de ubicación cuando la actividad se destruye
        try {
            locationManager.removeUpdates(locationListener)
        } catch (e: SecurityException) {
            e.printStackTrace()
        }

        // Detener el envío periódico de datos
        stopSendingData()
    }
}
