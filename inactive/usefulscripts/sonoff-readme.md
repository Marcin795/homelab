# Sonoff miniR2

1. connect to ITEAD-XXXXXXXXXX wifi password: 12345678
2. 10.10.7.1
3. connect device to wifi
4. flash tasmota - https://github.com/njh/sonoff-ota-flash-cli - PR for version 3.7.6
5. connect to tasmota wifi
6. connect device to wifi
7. set template configuration -> configure other -> {"NAME":"Sonoff MINIR2","GPIO":[17,0,0,0,9,0,0,0,21,157,0,0,0],"FLAG":0,"BASE":1}
8. set mqtt connection
