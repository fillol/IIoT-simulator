

# Industrial IoT Simulator: Smart Factory 4.0

![Industry 4.0](https://img.shields.io/badge/-Industry%204.0-4CAF50?logo=industry&logoColor=FFFF00)
![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=ffd343)
![MQTT](https://img.shields.io/badge/-MQTT-660099?logo=mosquitto&logoColor=white)
![Docker](https://img.shields.io/badge/-Docker-2496ED?logo=docker&logoColor=white)
![GitHub License](https://img.shields.io/badge/License-MIT-ff69b4)

## 📜 How it Works
An advanced industrial IoT simulator that emulates three production lines (pressing, welding, painting) with sensors compliant with Industry 4.0 standards.
It generates realistic data on vibration, temperature, and quality, transmitted via MQTT to a control center with predictive alert logic.

### 1. System Architecture
- Production lines publish data with different QoS levels
- The Control Center processes alerts and can integrate with external systems (ERP/SCADA)

### 2. Data Flow from Sensor to Alert
- Continuous cycle of generation-transmission-analysis
- Alert logic based on industrial standards

### 3. Dynamic Sensor Configuration
- The JSON file controls operational parameters
- Python classes implement the industrial logic
- Clear separation between configuration and implementation

## 🚀 Usage

### Prerequisites
- Docker and Docker Compose
- Python 3.10+ (for local development only)

### Quick Start
```bash
# 1. Clone the repository
git clone [https://github.com/fillol/industrial-iot-simulator.git](https://github.com/fillol/industrial-iot-simulator.git)

# 2. Start the infrastructure (and build it)
docker-compose up --build

# 3. Monitor the logs
docker-compose logs -f production-line-1 control-center

# 4. Monitor performance
docker-compose stats
```

---

## 🏗️ Project Structure
```
.
├── compose.yml             # Defines 5 services: 3 lines + control center + mosquitto
├── control-center/         # Central monitoring system
│   ├── Dockerfile          # Image based on Python slim
│   ├── main.py             # Real-time analysis logic
│   └── requirements.txt
├── mosquitto/              # High-performance MQTT broker
│   ├── Dockerfile          # Optimized configuration
│   └── mosquitto.conf      # Supports 10K+ msg/sec
└── publisher/              # Core of the production lines
    ├── config/             # Customizable JSON profiles
    │   ├── line1.json      # Pressing (high frequency)
    │   ├── line2.json      # Welding (mixed QoS)
    │   └── line3.json      # Painting (large payload)
    ├── sensors/            # Industry 4.0 sensor library
    │   ├── base_sensor.py  # Abstract class with payload generation
    │   ├── VibrationSensor.py # Machine health monitoring (ISO 10816)
    │   ├── TemperatureSensor.py # Thermal management (ISO 13732)
    │   └── QualitySensor.py  # Quality control (AI-driven)
    └── requirements.txt
```

## 🔍 Composition:

### 🐳 Key Actors:

#### 1. Production Lines
- **Implementation**: The business logic of each sensor is implemented in a separate Python class.
- **Configuration**: Each line is defined in a JSON file (`line1.json`, etc.) with:
  ```json
  {
    "line_id": "PRESS-LINE-1",
    "sensors": [
      {
        "type": "vibration",
        "interval": 0.5,  // Frequency in seconds
        "payload": "small", // Payload size (small/medium/large)
        "qos": 2          // MQTT Quality of Service level
      }
    ]
  }
  ```

- **Data Generation**:
  - **VibrationSensor**: 3D accelerations (x,y,z), FFT data, spectral metadata.
  - **TemperatureSensor**: Motor/bearing/coolant temperatures with time trends.
  - **QualitySensor**: Defect count, defect coordinates, simulated image hash.

#### 2. Control Center
- **Subscription**: Monitors all `factory/#` topics.
- **Alert Logic**:
  ```python
  if payload.get('x', 0) > 8.0:  # Vibration above 8 mm/s
    logger.warning(f"High vibration! {payload['x']} mm/s")

  if payload.get('defect_count', 0) > 3:  # More than 3 defects
    logger.error("Quality alert!")
  ```

#### 3. MQTT Broker (Mosquitto)
- Optimized configuration for high traffic:
  ```conf
  allow_anonymous true
  max_connections 1000
  message_size_limit 0  # Payloads up to 256MB (configurable)
  ```

### 📦 Payload Strategy

| Category | Size            | Frequency     | Use Case                 | Example Line      |
|-----------|-----------------|---------------|--------------------------|-------------------|
| **Small** | 1-10 KB         | 0.5-2 sec     | Real-time monitoring     | PRESS-LINE-1      |
| **Medium**| 10-100 KB       | 2-5 sec       | Historical trends        | WELDING-LINE-2    |
| **Large** | 100 KB - 1 MB   | 5-10 sec      | Big Data/Analytics       | PAINT-LINE-3      |

**Configuration Example** (line2.json & line1.json):
```json
{
  "sensors": [
    {
      "interval": 10.0,     // Aligned with inspection cycles
      "payload": "large",   // High-resolution image details
      "qos": 2              // Guaranteed for traceability
    },
    {
      "interval": 0.5,      // Aligned with fast production cycles
      "payload": "small",   // Optimized for latency <100ms
      "qos": 2              // Guaranteed for critical data
    }
  ]
}
```

### 🔬 Realistic Industrial Metrics

#### 1. **VibrationSensor** (Predictive Maintenance)
- **Reference Standard**: [ISO 10816-3](https://www.iso.org/standard/45674.html) (Vibrations in industrial machinery)
- **Key Parameters**:
  ```python
  {
    "x": random.uniform(2.0, 15.0),  # Typical range for CNC [mm/s²]
    "fft": [random.random() for _ in range(100)],  # Frequencies 0-1kHz
    "metadata": {
      "samples": size//1000  # Simulates 24-bit acquisition
    }
  }
  ```
  - **Alarm Thresholds**:
    - Warning: >8 mm/s² (imminent damage)
    - Critical: >15 mm/s² (failure within 24h)

#### 2. **TemperatureSensor** (Thermal Management)
- **Reference Standard**: [ISO 13732-1](https://www.iso.org/standard/51567.html) (Thermal contact)
- **Technical Data**:
  ```python
  {
    "motor_temp": random.uniform(30.0, 90.0),  # Class F insulation
    "bearing_temp": random.normalvariate(60.0, 5.0),  # Gaussian distribution
    "trend": [...]  # Data for thermographic analysis
  }
  ```
  - **Critical Thresholds**:
    - Motor: >85°C (efficiency loss)
    - Bearings: >70°C (accelerated wear)

#### 3. **QualitySensor** (Quality Control 4.0)
- **AI-driven Metrics**:
  ```python
  {
    "defect_count": random.randint(0, 5),  # Defect density/m²
    "image_meta": {
      "size_kb": target_size//1024,  # Simulates 1-5MP images
      "defect_coordinates": [...]  # XY system 1000x1000
    }
  }
  ```
  - **Alert**: >3 defects/batch (automotive production limit)

---

## 🛠️ Customization

### 🔧 **Customizing an Existing Line**
Modify the line's JSON file (e.g., `line1.json`) **without touching the Python code**:
1. Add a new sensor to the `sensors` array:
   ```json
   {
     "type": "temperature",       // Type corresponding to an existing Python class
     "interval": 3.0,             // New frequency (seconds)
     "payload": "medium",         // Data size (small/medium/large)
     "qos": 1                     // MQTT priority (0/1/2)
   }
   ```
2. Restart the container:
   `docker-compose up --build production-line-1`

### 🏭 **Adding a New Production Line**
1. Create a new JSON file in `publisher/config/` (e.g., `line4.json`):
   ```json
   {
     "line_id": "ASSEMBLY-LINE-4",  // Unique ID
     "sensors": [
       {
         "type": "vibration",       // Must correspond to an implemented sensor (VibrationSensor.py)
         "interval": 1.5,
         "payload": "medium",
         "qos": 2
       },
       {
         "type": "quality",
         "interval": 5.0,
         "payload": "large",
         "qos": 2
       }
     ]
   }
   ```
2. Add a new service in `compose.yml`:
   ```yaml
   production-line-4:
     build: ./publisher
     volumes:
       - ./publisher/config/line4.json:/app/config/line4.json  # Mount the new file
     environment:
       - CONFIG_FILE=line4.json  # Specify the configuration
     networks:
       - industry-net
     depends_on:
       - mosquitto
   ```
3. Start the new line:
   `docker-compose up --build production-line-4`

#### ✅ **Why Does This Work?**
- **Dynamic Binding**: The Python code looks up sensor types in the JSON and maps them to the corresponding classes (e.g., `"type": "vibration"` → `VibrationSensor.py`).
- **Docker Volumes**: JSON files are mounted as volumes, allowing "hot" modifications without rebuilding images.
- **Separation of Concerns**: Logic (Python) and configuration (JSON) are decoupled, following the [ISA-95](https://www.isa.org/standards-and-publications/isa-standards/isa-standards-standards/isa95) standard for industrial control systems.

📌 **Note**: To add a **completely new sensor type** (e.g., `PressureSensor`), you will need to:
1. Create a new Python class in `sensors/` (e.g., `PressureSensor.py`).
2. Register it in `sensors/__init__.py`.
3. Reference it in the JSON with `"type": "pressure"`.

### Other Common Modifications

1. **Custom Alert Rules**
   Modify `control-center/main.py`:
   ```python
   # Example: Alert for motor temperature >85°C
   if payload.get('motor_temp', 0) > 85.0:
       logger.critical(f"Motor overheating: {payload['motor_temp']}°C")
   ```

2. **Payload Sizes**
   Modify the ranges in `base_sensor.py`:
   ```python
   {
       "small": (1024, 10240),      # 1KB-10KB
       "medium": (10240, 102400),   # 10KB-100KB
       "large": (102400, 1048576)   # 100KB-1MB
   }
   ```

### For Real-World Scenarios

1. **Simulating specific environments**:
   ```python
   # In TemperatureSensor.py
   def _generate_specific_data(self, size):
       return {
           "motor_temp": random.uniform(40.0, 110.0),  # Simulate overheating
           "bearing_temp": random.normalvariate(60.0, 5.0)  # Gaussian distribution
       }
   ```

2. **Simulating scheduled failures**:
   ```python
   # In TemperatureSensor.py
   def _generate_specific_data(self, size):
       if random.random() < 0.01:  # 1% failure probability
           return {"motor_temp": 95.0, "status": "CRITICAL"}
   ```

3. **Optimizing MQTT Performance**:
   ```conf
   # mosquitto.conf
   max_packet_size 268435456     # 256MB for image payloads
   message_size_limit 0          # Disable single message limit
   ```

4. **Adapting to ISO specifications**:
   ```python
   # In VibrationSensor.py
   ISO_10816_THRESHOLDS = {
       "class_II": {"warning": 8.0, "critical": 12.0}  # Medium-large machines
   }
   ```

---

## 🏭 Conclusion: Towards Factory 4.0

This simulator provides a comprehensive framework for:
- Testing scalable IIoT architectures
- Validating QoS policies in real-world scenarios
- Developing predictive maintenance algorithms

## 🗺️ Future Roadmap
- [ ] Integrate a Grafana dashboard to visualize data in real-time
- [ ] Support TLS authentication for MQTT
- [ ] Implement a centralized logging system (ELK Stack)
