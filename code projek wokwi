#define BLYNK_TEMPLATE_ID "TMPL6WJqHV7Uj"
#define BLYNK_TEMPLATE_NAME "Smart Fan Controller"
#define BLYNK_AUTH_TOKEN "oA7R4uGIqFr-FA-OhSQSvE4G8u3BKKGC"

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>


// WIFI
char ssid[] = "Wokwi-GUEST";
char pass[] = "";

// DHT22
#define DHTPIN 4
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

// PIN ESP32
#define RELAY_PIN      18
#define GREEN_LED_PIN  19
#define RED_LED_PIN    21
#define BUZZER_PIN      23

#define RELAY_ON HIGH
#define RELAY_OFF LOW


const float FAN_ON_TEMP = 30.0;
const float FAN_OFF_TEMP = 29.0;     
const float DANGER_TEMP = 38.0;


// VARIABEL SENSOR
float temperature = 0;
float humidity = 0;
// STATUS SISTEM
bool autoMode = true;
bool fanStatus = false;
bool manualFan = false;
bool alarmStatus = false;
bool notificationSent = false;


// TIMER
BlynkTimer timer;


String systemStatus = "NORMAL";


void readSensor()
{
    humidity = dht.readHumidity();
    temperature = dht.readTemperature();

    if (isnan(temperature) || isnan(humidity))
    {
        Serial.println("ERROR : DHT22 Tidak Terbaca");
        return;
    }
}


// MODE AUTO

void autoControl()
{

    if (temperature >= FAN_ON_TEMP)
    {
        fanStatus = true;
    }

    if (temperature <= FAN_OFF_TEMP)
    {
        fanStatus = false;
    }

}


// MODE MANUAL

void manualControl()
{
    fanStatus = manualFan;
}

// UPDATE OUTPUT


void updateOutput()
{
    if (fanStatus)
        digitalWrite(RELAY_PIN, RELAY_ON);
    else
        digitalWrite(RELAY_PIN, RELAY_OFF);

    digitalWrite(RED_LED_PIN, fanStatus);
    digitalWrite(GREEN_LED_PIN, !fanStatus);
}



// CEK STATUS SISTEM & ALARM

void checkAlarm()
{
    // Status sistem
    if (temperature >= DANGER_TEMP)
    {
        systemStatus = "DANGER";
    }
    else if (temperature >= FAN_ON_TEMP)
    {
        systemStatus = "HOT";
    }
    else
    {
        systemStatus = "NORMAL";
    }

    // Alarm
    if (temperature >= DANGER_TEMP)
    {
        alarmStatus = true;

        digitalWrite(BUZZER_PIN, HIGH);

        if (!notificationSent)
        {
            Blynk.logEvent(
                "high_temp",
                "Warning! Room temperature above 38°C"
            );

            notificationSent = true;
        }
    }
    else
    {
        alarmStatus = false;
        notificationSent = false;

        digitalWrite(BUZZER_PIN, LOW);
    }
}


// UPDATE BLYNK

void updateBlynk()
{
    Blynk.virtualWrite(V0, temperature);
    Blynk.virtualWrite(V1, humidity);
    Blynk.virtualWrite(V2, fanStatus);
    Blynk.virtualWrite(V5, alarmStatus);
}

// SERIAL MONITOR


void printSerial()
{
    Serial.println();
    Serial.println("======================================");

    Serial.print("Temperature : ");
    Serial.print(temperature);
    Serial.println(" °C");

    Serial.print("Humidity    : ");
    Serial.print(humidity);
    Serial.println(" %");

    Serial.print("Mode        : ");
    Serial.println(autoMode ? "AUTO" : "MANUAL");

    Serial.print("Fan         : ");
    Serial.println(fanStatus ? "ON" : "OFF");

    Serial.print("Alarm       : ");
    Serial.println(alarmStatus ? "ON" : "OFF");

    Serial.print("Status      : ");
    Serial.println(systemStatus);

    Serial.println("======================================");
}


// SISTEM UTAMA

void mainSystem()
{
    readSensor();

    if (isnan(temperature) || isnan(humidity))
        return;

    if (autoMode)
        autoControl();
    else
        manualControl();

    updateOutput();

    checkAlarm();

    updateBlynk();

    printSerial();
}

// MODE AUTO / MANUAL

BLYNK_WRITE(V3)
{
    autoMode = param.asInt();

    Serial.print("Mode berubah menjadi : ");
    Serial.println(autoMode ? "AUTO" : "MANUAL");

    // Saat berpindah ke AUTO, langsung sesuaikan kondisi kipas
    if (autoMode)
    {
        autoControl();
        updateOutput();
    }
}



BLYNK_WRITE(V4)
{
    manualFan = param.asInt();

    if (!autoMode)
    {
        manualControl();
        updateOutput();
    }
}


void checkConnection()
{
    if (WiFi.status() != WL_CONNECTED)
    {
        Serial.println("WiFi Disconnect");
        return;
    }

    if (!Blynk.connected())
    {
        Serial.println("Reconnect Blynk...");
        Blynk.connect();
    }
}


void setup()
{
    Serial.begin(115200);

    Serial.println();
    Serial.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~");
    Serial.println(" Kipas Pintar ");
    Serial.println("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~");

    dht.begin();

    pinMode(RELAY_PIN, OUTPUT);
    pinMode(GREEN_LED_PIN, OUTPUT);
    pinMode(RED_LED_PIN, OUTPUT);
    pinMode(BUZZER_PIN, OUTPUT);

    digitalWrite(RELAY_PIN, RELAY_OFF);
    digitalWrite(GREEN_LED_PIN, HIGH);
    digitalWrite(RED_LED_PIN, LOW);
    digitalWrite(BUZZER_PIN, LOW);

    Blynk.begin(
        BLYNK_AUTH_TOKEN,
        ssid,
        pass
    );

    // Jalankan sistem setiap 2 detik
    timer.setInterval(2000L, mainSystem);

    // Cek koneksi setiap 10 detik
    timer.setInterval(10000L, checkConnection);

    Serial.println("System Ready");
}
//loop
void loop()
{
    Blynk.run();
    timer.run();
}
