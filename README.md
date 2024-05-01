## Application

Implementation of a GATT client on [esp32](https://www.espressif.com/en/products/socs/esp32#:~:text=ESP32%20is%20highly%2Dintegrated%20with,Hybrid%20Wi%2DFi%20%26%20Bluetooth%20Chip) using the BLE library of the ESP-IDF SDK.
I followed the [walkthrough](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/ble/gatt_client/tutorial/Gatt_Client_Example_Walkthrough.md) given by Espressif. Thanks to the developers of ESP-IDF examples and documentation, it's quite easy to start and practice 😍.

The code here is intended to create a BLE client that connects to the [BLE server](https://github.com/TIT8/BLE-sensor_PDM-microphone) provided by the Arduino Nano 33 BLE sense, which notifies for humidity changes from the HTS sensor.  
So the UUIDs provided are for a general Environmental Sensing Service (see [here](https://www.bluetooth.com/wp-content/uploads/Files/Specification/HTML/Assigned_Numbers/out/en/Assigned_Numbers.pdf?v=1713791642302) for more). Check the esp32 [configurations](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/ble/gatt_client/sdkconfig.defaults) when compiling via CMAKE (already handled by PlatformIO).

You have to connect the esp32 to the serial in order to see the received data (obviously), but also to connect to Bluetooth (it won't work in the background; it wasn't intended).

## TADAA! :rocket:

![Screenshot (108)](https://github.com/TIT8/BLE_esp32/assets/68781644/b80ea7e6-e04d-4160-b54d-cd3a7a8dd3b4)

## Float values from sensor

See this [reference](https://forum.arduino.cc/t/passing-a-floating-point-number-via-ble/1155922), it explains why I divide for 100 in the code below, while [sending uint32_t data](https://github.com/TIT8/BLE-sensor_PDM-microphone/blob/cad7776612e74c846272bd7182108c19a3b8fe7a/src/main.cpp#L37):

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
        uint32_t k = buff[0] | buff[1] << 8 | buff[2] << 16 | buff[3] << 24;
        /*
            Then, if you really need a float you can use union (if the software is compliled with
            a conforming ISO C compiler, like xtensa-gcc tools):
                union{
            		uint8_t i[4];     // Or use a uint32_t and fill it like "k" above
            		float f;
            	} u;
            	for(int o = 0; o < 4; o++)
            	{
            		u.i[o] = buff[o];
            	}
                printf("float union %.3f\n", u.f);

                Otherwise send a uint32_t multiplied by 100 and, when received, divide by 100
                keeping the float value.
                The humidity is a percentage with not so great and so low value,
                otherwise you won't be capable of read this. So I think this method is enough
                for the application.
        */
        printf("Value in decimal: %.2f%%\n\n", (float)k / 100);
        break;
```

And remember this in the future:

```c
// Transmitting float https://github.com/adafruit/Adafruit_BluefruitLE_nRF51/blob/master/examples/healththermometer/IEEE11073float.cpp

  //Receiving float
  float IEEE11073_2float(uint8_t *dat)
  {
      int32_t Mantissa = (dat[2] << 16 | dat[1] << 8 | dat[0]);
      uint8_t Neg = bitRead(dat[2],7);
      int8_t fExp = dat[3];
      if (Neg) Mantissa |= 255 << 24;
      return (float(Mantissa) * pow(10, fExp));
  }

/*
#define bitRead(value, bit) (((value) >> (bit)) & 0x01)
#define bitSet(value, bit) ((value) |= (1UL << (bit)))
#define bitClear(value, bit) ((value) &= ~(1UL << (bit)))
#define bitWrite(value, bit, bitvalue) (bitvalue ? bitSet(value, bit) : bitClear(value, bit))
*/

```
