## Application

Implementation of a GATT client on [esp32](https://www.espressif.com/en/products/socs/esp32#:~:text=ESP32%20is%20highly%2Dintegrated%20with,Hybrid%20Wi%2DFi%20%26%20Bluetooth%20Chip) using the BLE library of the ESP-IDF SDK.
I followed the [walkthrough](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/ble/gatt_client/tutorial/Gatt_Client_Example_Walkthrough.md) given by Espressif.

The code here is intended to create a BLE client that connect to the [BLE server](https://github.com/TIT8/BLE) given by the Arduino Nano 33 BLE sense which notify for humidity change from HTS sensor.
So the UUIDs given are for a general Environmental Sensing Service. Check the esp32 [configurations](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/ble/gatt_client/sdkconfig.defaults) when compiling via CMAKE (already handled by PlatformIO). 

You have to connect to the serial the esp32 in order to see the data received (obviously), but also to connect to bluetooth (it won't work in background, it wasn't intended).
