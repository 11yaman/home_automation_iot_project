# Home Automation System
Created by: Yaman Alkurdi (ya222gh)

In this comprehensive tutorial you will be provided with a step-by-step guide to create your own home automation application and be able to gain some hands-on experience in IoT-development. Focusing on the practical moments of the application and combining hardware and software foundations you will be provided with a simple home automation project that you can replicate within just 5 hours.
## Objective
To improve the functionality and comfort of the home environment, it was decided to construct a home automation system employing a motion PIR sensor and DHT11 sensor. This system's main goal is to automate multiple processes and provide more control over the home’s environment, which will ultimately increase comfort and energy efficiency.

By integrating the motion PIR sensor, the system can trigger human presence and make specific actions. For instance, to save energy, a room's lights may be programmed to turn on when a person enters and off when they leave. Daily tasks are made easier and require less manual intervention thanks to automation.

The DHT11 sensor, which measures temperature and humidity, gives the home automation system additional data. The system can optimize climate control and modify cooling settings based on current conditions by tracking the temperature. This guarantees a cozy living space while reducing energy use.

In general, this home automation system provides mainly insights about different home environment parameters that can be utilized in automating home devices. But also increases the efficiency of the energy usage and creates a more comfortable mood. Node-RED handles the automation triggers, while the TIG Stack enables real-time monitoring of temperature, humidity, and motion. The combination of Node-RED and the TIG Stack provides a comprehensive solution for home automation and data-monitoring.

## Material
| Component | Price and link | Specification |
| --- | --- | --------------------- |
| Start Kit – Applied IoT at Linnaeus University (2023)| [399 SEK](https://www.electrokit.com/produkt/start-kit-applied-iot-at-linnaeus-university-2023/)| Includes a Raspberry Pi Pico WH, a breadborad, wires, and a DHT11 sensor (Temp/humidity sensor)|
| PIR rörelsedetektor HC-SR501| [49 SEK]( https://www.electrokit.com/produkt/pir-rorelsedetektor-hc-sr501/)| Motion sensor that detects the heat from people and animals and has a sensing distance of 3-7 m|
| TP-link Tapo P100| [199 SEK]( https://www.kjell.com/se/produkter/smarta-hem/fjarrstrombrytare/fjarrstrombrytare-wifi/tp-link-tapo-mini-smart-wifi-fjarrstrombrytare-1-pack-p51639/)| Smart Wifi Remote Switch|
| YEELIGHT LED Smart bulb E27| [109 SEK](https://www.maxgaming.se/sv/ljuskallor/led-smart-bulb-e27-8w-900lm-w3-white-dimmable?utm_source=kelkoose&utm_medium=cpc&utm_campaign=kelkooclick&utm_term=Yeelight+LED+Smart+bulb+E27+8W+900Lm+W3+/)| Smart LED Light Bulb from Yeelight |

## Computer setup
### Visual Studio Code and Pymakr
To program your Raspberry Pi Pico WH using MicroPython and Pymakr in Visual Studio Code (VSCode), you can follow these steps:

1. Flashing the firmware on Raspberry Pi Pico WH:
   - Download the latest MicroPython firmware for Raspberry Pi Pico from the official MicroPython [website]( https://micropython.org/download/rp2-pico/)
   - Put the Raspberry Pi Pico WH into bootloader mode by pressing and holding the `BOOTSEL` button while connecting it to your computer using a USB cable.
   - The Raspberry Pi Pico WH will appear as a USB mass storage device on your computer.
   - Drag and drop the downloaded firmware file `(a .uf2 file)` onto the Raspberry Pi Pico WH USB mass storage device.
   - The firmware will be flashed to the device, and the Raspberry Pi Pico WH will restart with MicroPython.
2. Installing Visual Studio Code and Pymakr:
   - Download and install Visual Studio Code from the official website: https://code.visualstudio.com/.
   - After launching VS Code, open the Extensions view by clicking on the square icon on the left sidebar.
   - Search for "Pymakr" in the Extensions view and install it.
3. Setting up Pymakr:
   - Create a new folder on your computer for your Pymakr project.
   - Open the project folder in Visual Studio Code.
   - Connect your Raspberry Pi Pico WH to your computer via USB.
   - In Visual Studio Code, open the Command Palette by pressing `Ctrl+Shift+P`.
   - Search for "Pymakr: Select Serial Port" and choose the serial port corresponding to your Raspberry Pi Pico WH.
   - You can also use the Pymakr toolbar to perform common tasks like uploading files and executing commands on the device.
4. Adding and uploading code:
   - Add the code inside the `src` folder from my Github repository into your project folder in VS Code.
   - Use the Pymakr toolbar or right-click on the file and select `Upload` to upload the code to the Raspberry Pi Pico WH.
   - The code will be uploaded and executed on the device, and you can view the output in REPL terminal.

### Docker, Mosquitto, TIG-stack and Node-RED
The Node-RED platform is used as a user interface of the home automation system and the Mosquitto broker as a MQTT communication protocol between the Pico and the user interface. 

1. Install Docker Desktop:
   - Follow the official documentation to install `Docker`: https://docs.docker.com/get-docker/
2. Create a project directory:
   - Create a new directory for your project and navigate to it in your terminal or command prompt.
3. Docker Compose:
	- Download the `docker-compose.yml` file from my repository to your project directory
4. Configuration files:
   
	Mosquitto:
	- In the same project directory, create first a file with the name `acl.acl` with these configurations:
	     ```
		user admin
		topic read $SYS/#
		topic readwrite devices/#
		
		user myUser
		topic readwrite devices/#
	     ```
    - Create then a password file for mosquitto with your own users and passwords (Must be the same users as in the acl file above)
    - In the same project directory, create a file named `mosquitto.conf` and open it using a text editor
    - Add these custom configurations to the `mosquitto.conf` file:
		
		```
		listener 1883
		allow_anonymous false
		persistent_client_expiration 15d     
		autosave_interval 60
		persistence true
		persistence_file mosquitto.db
		persistence_location /mosquitto/data
		max_keepalive 0    
		password_file /mosquitto/config/passwd
		acl_file /mosquitto/config/acl.acl
		```
    - Save the `mosquitto.conf` file in your project directory.

	Telegraf:
	- In the same project directory, create a file named `telegraf.conf` and open it using a text editor
 	- Edit the file with the mosquitto username and password you chose in previous step
  	- The username and password of influxdb should be the same as these in the the `docker-compose` file
	```
	[agent]
	  flush_interval = "15s"
	  interval = "15s"
	
	[[inputs.mqtt_consumer]]
	  name_override = "mosquitto"
	  servers = ["tcp://127.0.0.1:1883"]
	  qos = 0
	  connection_timeout = "30s"
	  topics = [ "devices/#" ]
	  username = "myUser"
	  password = "myUser"
	  data_format = "json"
	
	
	[[outputs.influxdb]]
	  database = "iot_db"
	  urls = [ "http://influxdb:8086" ]
	  username = "admin"
	  password = "admin"
	```

	
4. Set up Node-RED:
   - Create a directory named `node-red` in your project directory.
   - Open a terminal and navigate to your project directory.
   - Run the following command to create the necessary files and folders for Node-RED:
		```
		docker-compose run --rm node-red npm install node-red-dashboard
		```

5. Start the containers:
   - In the terminal or command prompt, navigate to your project directory.
   - Run the following command to start the containers:
     ```
     docker-compose up -d
     ```

6. Access Node-RED/Grafana:
   - Open a web browser and go to `http://localhost:1880` to access the Node-RED editor.
   - You can now import the `flow.json` from my Github repository file using the Node-RED editor.
   - Grafana (optional): Open a web browser and go to `http://localhost:3000`. Use the credentials `admin/admin` to log in.

7. Configure MQTT in Node-RED:
   - Inside the Node-RED editor, add the MQTT nodes to your flows.
   - Configure the MQTT nodes to connect to the Mosquitto MQTT broker using the hostname `mosquitto` and port `1883`. Then add the username and password from the `mosquitto.conf` file.

8. Import the flow:
   - Inside the Node-RED editor, click on the menu icon in the top-right corner and select `"Import" -> "Clipboard"`.
   - Select a file from you local machine and click the `Import` button to import the flow.

9. Deploy the flow:
   - Once the flow is imported, click the `Deploy` button in the top-right corner of the Node-RED editor to deploy the flow.

Note: Ensure that you have a basic understanding of Docker and Docker Compose concepts. It's always recommended to refer to the official documentation and resources for detailed and up-to-date instructions specific to your operating system and environment.

## Putting everything together
![Wiring](https://github.com/11yaman/home_automation_iot_project/blob/main/Wiring.png)

The circuit diagram showcases the connections between the Pico W, DHT11 sensor, and PIR motion sensor, providing a visual representation of how the components are connected.

The DHT11 sensor operates at 3.3V, and its data pin is connected to a GPIO pin on the Pico W. The PIR motion sensor also operates at 3.3V and is connected to another GPIO pin on the Pico W. Resistors are not specifically required for this setup as the sensors are designed to be directly connected to the microcontroller's GPIO pins. However, it is important to ensure that the current requirements of the sensors are within the limits of the microcontroller's GPIO pins to avoid any damage. 

## Platform
Initially, I experimented with the TIG (Telegraf, InfluxDB, and Grafana) stack and Grafana as the user interface for data visualization. I realized later that Node-RED better suited my home automation requirements. Node-RED is integrated for its ease of use, visual programming approach, and extensive library of pre-built nodes that simplify the integration of various services.

However, I continue to use the TIG stack as it provides a robust and scalable solution for storing and visualizing sensor data. Specifically, I leverage InfluxDB from the TIG stack in my Node-RED application to display the collected data. By utilizing InfluxDB's time-series database capabilities, I can store and query sensor data efficiently.

Considering the project requirements, it is acceptable to have a slight delay in data transmission to the Node-RED platform. Real-time data updates within seconds are not critical for my application, allowing for an acceptable delay. This approach strikes a balance between leveraging the real-time capabilities of the TIG stack for data storage and visualization, and utilizing the flexibility and automation capabilities of Node-RED.

In conclusion, Node-RED is used as the main platform for implementing the automation logic and controlling my devices, while the TIG stack ensures reliable and efficient data storage and visualization. By combining the strengths of both the TIG stack and Node-RED, I have created a comprehensive home automation system.
## The code
All the code required for this project is attached in this repository. Please note that you have to edit the `config.py` file according to your own credentials/IP addresses:

```python
# Mosquitto broker
mosquitto_ip_addr = 'localhost'
mosquitto_user = 'YOUR_MOSQUITTO_USER'
mosquitto_pass = 'YOUR_MOSQUITTO_PASSWORD'

# Yeelight
bulb_ip_addr = "YOUR_BULB_IP_ADDRESS"

# IFTT Webhooks
iftt_webhooks_key = "YOUR_IFTT_KEY"
iftt_ac_on_event = "AC_on" # Example on event name
iftt_ac_off_event = "AC_off" # Example on event name
 ```


## Data Transmission and Connectivity
To transmit the data, the home automation system utilizes the MQTT (Message Queuing Telemetry Transport) protocol over WIFI. MQTT is a lightweight messaging protocol suitable for IoT applications due to its low overhead and efficient data transmission. MQTT enables efficient and reliable communication between devices and the server (MQTT broker).

### Package Format:
The data is published in JSON format using the MQTT protocol. The JSON format allows for structured data representation, making it easy to organize and interpret the temperature, humidity, and motion data. For example:

- Temperature and Humidity Data Format:
  ```json
  {"temperature": 25.5, "humidity": 45.2}
  ```
- Motion Data Format:
  ```json
  {"motion": true}
  ```

### Data Transmission and protocols:
The temperature and humidity data are measured and published at a fixed interval of 15 minutes. This interval determines how often the data is collected and transmitted to the MQTT broker. On the other hand, the motion data is continuously scanned, but it is published only when a motion is detected and at a frequency of once every minute. This approach helps to conserve network bandwidth and optimize resource usage.

WiFi is the wireless protocol used in the home automation system. WiFi provides a reliable and widely available wireless network connection, allowing the devices to connect to the local network and communicate with the MQTT broker over the internet. The MQTT protocol is used as the transport protocol for data transmission. It ensures efficient and reliable message exchange between the device and the MQTT broker.

### Design Choices and Impact:
Regarding data transmission frequency, the chosen intervals strike a balance between collecting data frequently enough to capture meaningful insights and reducing power consumption. Measuring the temperature and humidity every 15 minutes helps capture changes in environmental conditions while minimizing energy usage. Sending motion data every minute strikes a balance between real-time monitoring and resource optimization.

The design choices made in terms of wireless protocols, data transmission intervals, and the use of MQTT help achieve an optimal balance between device range, battery consumption, and data collection frequency. The WiFi protocol ensures reliable connectivity within the home network, while MQTT facilitates efficient and lightweight data transmission. The selected data transmission intervals help strike a balance between capturing timely information and conserving power, optimizing battery life for the connected devices.

## Presenting the data
The main dashboard is built using Node-RED and displays the temperature, humidity, and motion data in a user-friendly and visually appealing manner. The dashboard components include temperature and humidity gauges, line charts for data in the last few hours, and chart indicator for motion detections. These components are dynamically updated as new data is received from the sensors and stored in the database. The dashboard layout and design can be customized to suit specific preferences and requirements.

![Dashboard](https://github.com/11yaman/home_automation_iot_project/blob/main/Dashboard.png)

An alternative dashboard in Grafana:

![Dashboard2](https://github.com/11yaman/home_automation_iot_project/blob/main/Dashboard2.png)


The collected data is stored in an InfluxDB database. InfluxDB is a time-series database that is well-suited for storing and analyzing time-stamped data. It efficiently handles large volumes of data and allows for easy retrieval and querying based on time ranges.

The data is saved in the database every time it is received by the MQTT broker. As previously mentioned, the temperature and humidity data are measured and published at a fixed interval of 15 minutes, while the motion data is published only when a motion is detected and at a frequency of once every minute. Each time the data is published, it is captured by Telegraf, which is responsible for collecting and storing the data into the InfluxDB database.

The choice of InfluxDB as the database for this home automation system is driven by its suitability for time-series data. InfluxDB efficiently handles the continuous influx of sensor data and provides powerful querying capabilities for time-based analysis. It supports the retention of data for the desired duration, allowing for historical comparisons and trend analysis.

The automation and triggers for the home automation system are handled in MicroPython running on the Raspberry Pi Pico W. Node-RED interacts with the Pico W by sending triggers or instructions based on user options.

## Final results

Here are the final results of my home automation project. The system successfully integrates a motion PIR sensor and DHT11 sensor to automate various processes and enhance comfort in my home. The custom dashboard in Node-RED provides real-time insights into temperature, humidity, and motion data, allowing me to make informed decisions. The project could have been further enhanced by integrating additional sensors or implementing voice control functionality. Overall, the project went well and achieved its objectives.

[Video presentation of the final result: ](https://youtu.be/vGo5s0sw_3E)

[![Video](https://img.youtube.com/vi/vGo5s0sw_3E/0.jpg)](https://youtu.be/vGo5s0sw_3E)
