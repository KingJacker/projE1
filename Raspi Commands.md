## Ethernet connection [stackexchange](https://raspberrypi.stackexchange.com/questions/55762/connecting-two-raspberry-pis-via-ethernet)
The simplest way to hook up the two Pi's is to

- have a cat5e/cat6 **cross over** cable of sufficient length
- configure each Pi to have a unique static IP within the same network by editing the `/etc/network/interfaces`

e.g on PI 1

```bash
auto eth0
iface eth0 inet static
address 192.168.1.5
netmask 255.255.255.0
gateway 192.168.1.6
```

on PI 2

```bash
auto eth0
iface eth0 inet static
address 192.168.1.6
netmask 255.255.255.0
gateway 192.168.1.5
```


## Communication

pi 2: (empfänger)
```bash
nc -l 1234 # netcat opens listener on port 1234
```

pi 1: (sender)
```bash
echo "Hello World" | nc 192.168.1.6 1234
```

--> 
to python program [stackoverflow: socket](https://stackoverflow.com/questions/17664143/python-port-listener-like-nc)
### in python ([python socket](https://ioflood.com/blog/python-socket/#:~:text=A%20Python%20socket%20is%20a,two%20nodes%20on%20a%20network.))
sender:
```python
import socket
s = socket.socket()
s.connect(("192.168.1.6", 1234))
s.send(b"nachricht")
```
empfänger:
```python
import socket
s = socket.socket()
s.bind(("192.168.1.6", 1234))
s.listen(1)

connection, address = s.accept()
message = connection.recv(1024)
out = message.decode()
```


## Raspi GPIO Ansteuerung
[GPIOZero](https://gpiozero.readthedocs.io/en/latest/)
- Beispiel code: blink
```python 
from gpiozero import LED
from time import sleep

led = LED(17)

while True:
    led.on()
    sleep(1)
    led.off()
    sleep(1)
```

