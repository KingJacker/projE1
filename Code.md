## Sender
### main.py
```python
from time import sleep
from read_sensor import setup_sensor, read_sensor
from send import send_message

# setup
print("Setup")

occupancy, movement = setup_sensor()

ip = "192.168.1.6"
port = 1234


# loop

while True:
	sensor_data = read_sensor(occupancy, movement)
	b_sensor_data = bytearray(sensor_data) # converting to byte array
	try:
		send_message(ip, port, b_sensor_data)
		print(f"Sending {sensor_data} to {ip} at {port}")
	except Exception as e:
		print(e)
	sleep(5)
```
### read_sensor.py
```python
from gpiozero import DigitalInputDevice

#funcs

## define pins
def setup_sensor():
	occupancy = DigitalInputDevice(24)
	movement = DigitalInputDevice(23)
	print("Sensor Pins set")
	return occupancy, movement

def read_sensor(occ, mov):
	print("Reading Sensor")
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

## Empfänger
### listener.py
```python
from socket import socket, SOL_SOCKET, SO_REUSEADDR
from time import sleep
from gpiozero import LED




# funcs

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



# setup


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
	occupied = bool(receive(s)[0])
	movement = bool(receive(s)[1])
	print(f"Occupied: {bool(receive(s)[0])} \nMoving: {bool(receive(s)[1])}")
	if occupied:
		occupied_led.on()
	else: occupied_led.off()
	if movement:
		movement_led.on()
	else:
		movement_led.off()
	sleep(0.5)



```
