# Sender
### Struktur
```shell
└── pi
    ├── launcher.sh
    ├── main.py
    ├── read_sensor.py
    └── send.py
```

### launcher.sh
```shell
#!/bin/sh
# launcher.sh

cd /
cd home/pi/pro1
sudo python main.py
cd /
```
### main.py
```python
from time import sleep
from read_sensor import setup_sensor, read_sensor
from send import send_message

# setup
OCCUPANCY_PIN = 24
MOVEMENT_PIN = 23

occupancy, movement = setup_sensor(OCCUPANCY_PIN, MOVEMENT_PIN)

IP = "192.168.1.6"
PORT = 1234


# loop

while True:
	sensor_data = read_sensor(occupancy, movement)
	b_sensor_data = bytearray(sensor_data) # converting to byte array
	try:
		send_message(IP, PORT, b_sensor_data)
		print(f"Sending {sensor_data} to {IP} at {PORT}")
	except Exception as e:
		print(e)
	sleep(1)

```
### read_sensor.py
```python
from gpiozero import DigitalInputDevice

#funcs

## define pins
def setup_sensor(occupancy_pin, movement_pin):
	occupancy = DigitalInputDevice(occupancy_pin)
	movement = DigitalInputDevice(movement_pin)
	print(f"SET SENSOR PINS: \n\tOccupancy: {occupancy_pin}\n\tMovement: {movement_pin}")
	return occupancy, movement

def read_sensor(occ, mov):
	return occ.value, mov.value

```
### send.py
```python
from socket import socket

# send message to ip at port
def send_message(ip, port, message):
	s = socket()
	s.connect((ip, port))
	s.send(message)
```

---
## Empfänger
### Struktur
```shell
└── empfänger
    ├── launcher.sh
    └── listener.py
```
### launcher.sh
```shell
#!/bin/sh
# launcher.sh

cd /
cd home/pi/empf
sudo python listener.py
cd /
```

### listener.py
```python
from socket import socket, SOL_SOCKET, SO_REUSEADDR
from time import sleep
from gpiozero import LED

# functions
## open connection to ip on port
def listen(ip, port):
	s = socket()
	s.bind((ip, port))
	s.listen(10)
	print(f"Listening on port {port}")
	return s

# split individual bytes apart
def split_bytes(bytes):
	return [bytes[i:i + 1] for i in range(0, len(bytes), 1)]

## receive data from socket s and return message content
def receive(s):
	conn, addr = s.accept()
	byte_message = conn.recv(1024)
	message = split_bytes(byte_message)
	return message

## define led pin
occupied_led = LED(14)
movement_led = LED(15)

occupied_led.off()
movement_led.off()

## set port and ip
port = 1234
ip = "192.168.1.6"

## setup
s = listen(ip, port)

## loop
while True:
	occupied = receive(s)[0] == b"\x01"
	movement = receive(s)[1] == b"\x01"
	print(f"({occupied}, {movement})")
	if occupied:
		occupied_led.on()
	else: occupied_led.off()
	if movement:
		movement_led.on()
	else:
		movement_led.off()
	sleep(0.5)
```
