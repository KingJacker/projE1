## Disable Wi-Fi on Raspberry Pi
```bash
sudo crontab -e
@reboot ifconfig wlan0 down
```

**To bring the Wi-Fi up again (temporarily), use the following:**  
```bash
sudo ifconfig wlan0 up
```
Or remove the line in the crontab to enable it at each boot.


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

pi 2:
```bash
nc -l 1234 # netcat opens listener on port 1234
```

pi 1:
```bash
echo "Hello World" | nc 192.168.1.6 1234
```

