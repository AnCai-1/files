

##  1. Objective

Build a wearable application on Galaxy Watch using **Samsung Health Sensor SDK** to:

- Collect real-time physiological data 
- Save data as structured JSON
- Upload it securely to AWS S3 using **Amplify Storage**

---

##  2. Supported Sensors

| Sensor Name     | Tracker Constant                         | Raw / Processed | Description                                 | frequency |
|-----------------|--------------------------------------------|---------------------------------------------|---------------------------------------------|---------------------------------------------|
| Heart Rate including IBI      | `HEART_RATE_CONTINUOUS`          |   Processed      | Measures BPM (beats per minute) and IBI (inter-beat interval)  | frequency of 1 Hz |
| Accelerometer             |  `ACCELEROMETER_CONTINUOUS`    |   Raw   | Measures movement and orientation.	Detects physical activity, restlessness, or repetitive motion. Raw X, Y, and Z axis data.| frequency of 25 Hz|
| BIA (Body Composition)             |  `BIA_ON_DEMAND`     |   Processed  | Measures fat, muscle, water mass... Bioelectrical impedance analysis data.|
| ECG (Electrocardiogram)             |  `ECG_ON_DEMAND`    |   Raw   | Captures electrical heart activity. Raw electrocardiogram data.| frequency of 500 Hz. |
| PPG (Green, IR, Red)             |  `PPG_CONTINUOUS`     |   Raw  | Optical sensor for HR, SpO2, blood flow.	Raw PPG green, infrared, and red data.|frequency of 25 Hz |
| SpO2 (Oxygen Saturation)             |  `SPO2_ON_DEMAND`    |   Processed   | % of oxygenated hemoglobin in blood. Blood oxygen level.| 
| Skin Temperature             |  `SKIN_TEMPERATURE_CONTINUOUS`    |   Processed   | Measures wrist skin temp and around temp.|

EDA (Electrodermal activity), MF-BIA (Multi frequency BIA) and Sweat loss are available..

---

##  3. System Architecture

```
[Galaxy Watch App]
  ↳ Samsung Health Sensor SDK
      ↳ HealthTrackerType.HEART_RATE_CONTINUOUS
      ↳ ValueKey.HeartRateSet.*
  ↳ Writes JSON locally
  ↳ Uploads to AWS S3 via Amplify

[AWS S3 Bucket]
  ↳ Stores session JSON files
  ↳ Enables remote access & data analysis
```

---

##  3. Sensor Behavior Observations

| Sensor | status | Notes |
|-------------|-------------|-------------|
| Accelerometer | stable | High-frequency data | 
| BIA (Body Composition)  | Conditional |user's profile(height,weight,gender and age) initiation required. processing| 
| ECG (Electrocardiogram)  | Conditional | frequency of 500 Hz. should do short time |
| Heart Rate including IBI | stable | warm-up time is needed. |
| PPG (Green, IR, Red)  | stable | Green, IR, Red are optional | 
| SpO2 (Oxygen Saturation)  | stable | takes 20-30 seconds |
| Skin Temperature    | unstable | Works inconsistently |

conditional means specific posture is required. [measurement Guide](https://developer.samsung.com/health/sensor/guide/measurement-guide.html)

using multiple sensors could conflict, especially when using both on-demand and continuous sensors(e.g. BIA and HeartRate )

---

---

##  4. Sample Collected Data

# 1. Accelerometer

```json
{
  "timestamp": 1753937134939,
  "datetime": "2025-07-30 23:45:34",
  "RawXYZ": {
    "x": -366,
    "y": 663,
    "z": 4029
  },
  "CalculatedXYZ": {
    "x": -0.876204,
    "y": 1.587222,
    "z": 9.645426
  }
}
```

- `timestamp`: Milliseconds since epoch
- `datetime`: Human-readable timestamp
- `RawXYZ`: Raw accelerometer values in hardware units 
- `CalculatedXYZ`: calculated value with : 9.81 / (16383.75 / 4.0)) * value

 <img width="1400" height="600" alt="image" src="/png/Accelerometer.png" />
 Larger Dataset
 <img width="1400" height="600" alt="image" src="https://github.com/user-attachments/assets/58f827c7-db11-48da-b87b-1a4787d7418c" />





- # 2. BIA (Body Composition)

```json
{
  "timestamp": 1753976390425,
  "datetime": "2025-07-31 10:39:50",
  "metabolicRate_kcal": 1442.1187,
  "bodyFatMass_kg": 20.364878,
  "bodyFatRatio_%": 29.092682,
  "impedanceDegree_°": -11.11647,
  "bodyImpedanceMagnitude_Ω": 682.51276,
  "fatFreeMass_kg": 49.635124,
  "fatFreeRatio_%": 70.90732,
  "progress": 0.06666667,
  "muscleMass_kg": 26.436842,
  "muscleRatio_%": 37.76692,
  "water_L": 36.333103,
  "status": 0
}
```

- `metabolicRate_kcal`: Basal metabolic rate in kilocalories
- `bodyFatMass_kg`: Body fat mass in kilograms.
- `bodyFatRatio_%`: Body fat ratio in percentage.
- `impedanceDegree_°`: Body impedance degree in degrees.
- `bodyImpedanceMagnitude_Ω`: Body impedance magnitude in ohms.
- `fatFreeMass_kg`: Fat free mass in kilograms.
- `fatFreeRatio_%`: Fat free ratio in percentage.
- `progress`: Measurement progress(0-1).
- `muscleMass_kg`: Skeletal muscle mass in kilograms.
- `muscleRatio_`: Skeletal muscle ratio in percentage.
- `water_L`: Total body water in liters.
- `status`: Status flag for body composition (BIA) measurement.(0:	Successful measurement.)

- # 3. ECG (Electrocardiogram)

```json
{
  "timestamp": "2025-07-31 12:28:50",
  "ECG_values_millivolts": [
    125.069046,
    124.46726,
    123.869225,
    123.53652,
    123.31935,
    123.11078,
    123.21853,
    123.18284,
    123.083755,
    123.049194,
    122.917885,
    122.71608,
    122.59355,
    122.5432,
    122.32393,
    122.08578,
    121.85211
  ]
}
```
- `ECG_values_millivolts`: List of average of sampled ECG values in millivolts
```json
{
  "timestamp": 1753983540196,
  "datetime": "2025-07-31 12:39:00",
  "ECG_value_millivolts": 556.66455078125
}
```
- `ECG_value_millivolts`: ECG value in millivolts.

  <img width="1400" height="600" alt="image" src="/png/ECG.png" />
  Larger Dataset
  <img width="1400" height="600" alt="image" src="https://github.com/user-attachments/assets/fc8d8982-8f23-4a26-bce7-2f400258bcf0" />




- # 4. Heart Rate including IBI

```json
{
  "timestamp": 1753924269344,
  "datetime": "2025-07-30 20:11:09",
  "heart_rate": 109,
  "heart_rate_status": 1,
  "ibi_list": [813, 804],
  "ibi_status_list": [0, 0]
}
```

- `timestamp`: Milliseconds since epoch
- `datetime`: Human-readable timestamp
- `heart_rate`: Heart rate in BPM
- `heart_rate_status`: Status (0 = warming up, 1 = valid)
- `ibi_list`: Inter-beat intervals in ms
- `ibi_status_list`: Status of each IBI (0 = normal, -1 = error)

  <img width="1400" height="500" alt="image" src="/png/HR.png" />
  <img width="1400" height="500" alt="image" src="/png/IBI.png" />
  Larger Dataset
  <img width="1400" height="500" alt="image" src="https://github.com/user-attachments/assets/91918849-e389-425e-b647-bf2cb98d68db" />
  <img width="1400" height="500" alt="image" src="https://github.com/user-attachments/assets/eec01fd0-5735-4ed9-b90c-2e23b2ab8a57" />






- # 5. PPG (Green, IR, Red)

```json
{
  "timestamp": 1753985641774,
  "datetime": "2025-07-31 13:14:01",
  "green": 46424,
  "greenStatus": 0,
  "ir": 7201010,
  "irStatus": 0,
  "red": 5195629,
  "redStatus": 0
}
```

- `green`: PPG green LED's ADC (analog-to-digital) value.
- `IR,red`: PPG IR's, red's raw value.
- `status`: Status of green, IR, red LED (0 = normal, -1 = error )

  <img width="1200" height="600" alt="image" src="/png/PPG.png" />
  Larger Dataset
  <img width="1400" height="600" alt="image" src="https://github.com/user-attachments/assets/3f8b87a7-7408-4fa2-9a96-bc3230f771e0" />



- # 6. SpO2 (Oxygen Saturation)

```json
{
    "timestamp": 1753987358576,
    "datetime": "2025-07-31 13:42:38",
    "status": 0,
    "spo2°": 0,
    "hr": 0
  },
  {
    "timestamp": 1753987374537,
    "datetime": "2025-07-31 13:42:54",
    "status": 2,
    "spo2°": 94,
    "hr": 77
  }
```

- `status`: Status flag for SpO2 measurement.(0: Measuring, 2: Measurement complete)
- `spo2`: Oxygen saturation (SpO2) value as a percentage.
- `hr`: Heart rate value in beats per minute.

- # 7. Skin Temperature 

```json
{
    "timestamp": 1753995373434,
    "datetime": "2025-07-31 15:56:13",
    "status": 0,
    "Ambient_temp_°": 35.816296,
    "object_temp_°": 35.003727
  },
```

- `status`: Status flag for temperature measurement.
- `Ambient_temp_°`: Ambient temperature around the Galaxy Watch.
- `object_temp_°`: Temperature of an object contacted with the Galaxy Watch


---


##  5. References

- [Samsung Health Sensor SDK Documentation](https://developer.samsung.com/health/sensor/overview.html)
- [HealthTrackerType API Reference](https://developer.samsung.com/health/sensor/api-reference/overview-summary.html)
