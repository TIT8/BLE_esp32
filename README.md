## Application

Implementation of a GATT client on [esp32](https://www.espressif.com/en/products/socs/esp32#:~:text=ESP32%20is%20highly%2Dintegrated%20with,Hybrid%20Wi%2DFi%20%26%20Bluetooth%20Chip) using the BLE library of the ESP-IDF SDK.
I followed the [walkthrough](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/ble/gatt_client/tutorial/Gatt_Client_Example_Walkthrough.md) given by Espressif. Thanks to the developers of ESP-IDF examples and documentation, it's quite easy to start and practice 😍.

The code here is intended to create a BLE client that connects to the [BLE server](https://github.com/TIT8/BLE-sensor_PDM-microphone) provided by the Arduino Nano 33 BLE sense, which notifies for humidity changes from the HTS sensor.  
So the UUIDs provided are for a general Environmental Sensing Service (see [here](https://www.bluetooth.com/wp-content/uploads/Files/Specification/HTML/Assigned_Numbers/out/en/Assigned_Numbers.pdf?v=1713791642302) for more). Check the esp32 [configurations](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/ble/gatt_client/sdkconfig.defaults) when compiling via CMAKE (already handled by PlatformIO).

You have to connect the esp32 to the serial in order to see the received data (obviously), but also to connect to Bluetooth (it won't work in the background; it wasn't intended).

## TADAA! :rocket:

![Screenshot (104)](https://github.com/TIT8/BLE_esp32/assets/68781644/b1fac8e4-7b08-4dc1-9ee2-ce851c011d54)

## Float value from sensor

I was not interested in the values coming from the BLE central (Google can also say the humidity of a location), but in the future I can make this better. Remember this [reference](https://forum.arduino.cc/t/passing-a-floating-point-number-via-ble/1155922) and modify this section to "create" a float from the byte stream:
```C
    case ESP_GATTC_NOTIFY_EVT:
          if (p_data->notify.is_notify){
              ESP_LOGI(GATTC_TAG, "ESP_GATTC_NOTIFY_EVT, receive notify value:");
          }else{
              ESP_LOGI(GATTC_TAG, "ESP_GATTC_NOTIFY_EVT, receive indicate value:");
          }
          esp_log_buffer_hex(GATTC_TAG, p_data->notify.value, p_data->notify.value_len);
          /*
              p_data->notify.value is a pointer of uint8_t, so I want a 32 bit integer
              p_data->notify.value_len is always 4. Two way to convert in decimal the 
              bytes received.
  
              FIRST WAY:   
                  Use the little endian memory model of the xtensa inside the
                  esp32. Good, but it's undefined behaviour (though, it works on xtensa LX6)
                  See <https://gist.github.com/shafik/848ae25ee209f698763cffee272a58f8>
                      uint32_t* k = (uint32_t *)p_data->notify.value; 
                  
                  So use memcpy instead:
                      uint32_t k = 0;
                      memcpy(&k, p_data->notify.value, sizeof(uint32_t));
  
              SECOND WAY:
                  This will promote single byte to 32 bit number, arranging its 4 bytes in order
                  and it's valid only for little endian (which xtensa is).
          */
          uint8_t* buff = p_data->notify.value;
          int k = buff[0]|(buff[1]<<8)|(buff[2]<<16)|(buff[3]<<24);
          printf("Value in decimal: %d%%\n\n", k / 100);
          break;
```
