# EzLoRaWAN
**EzLoRaWAN** is an evolution of the [TheThingsNetwork_esp32](https://github.com/rgot-org/TheThingsNetwork_esp32) library. Indeed TheThingsNetwork_esp32 does not support SX126X type chips. This library uses the **[BasicMAC](https://github.com/lacunaspace/basicmac)** library as a replacement for [mcci-catena/arduino-lmic](https://github.com/mcci-catena/arduino-lmic/tree/master). The objective of this library is to simplify as much as possible the configuration, the connection, the sending and receiving of frames to a LoRaWan network.
 
# Configuration
The library is tested for EU868 frequency plan (SX1262 & SX1276 chips). TODO enhance and verify for other frenquency plans

Pin assignment and initialization is automatic for some boards (see list below). In this case an `AUTOPIN` constant is declared.
the following boards are tested with `AUTOPIN` :
 - HELTEC WIRELESS STICK (SX1276)
 - HELTEC WIRELESS STICK V3 (SX1262)
 - HELTEC WIFI LORA 32 V1 (SX1276)
 - HELTEC WIFI LORA 32 V2 (SX1276)
 - HELTEC WIRELESS PAPER (SX1262)
 - HELTEC WIRELESS TRACKER V1.1 (SX1262)
 - TTGO TBEAM 1 (SX1276)

To add new board edit the file [**target-config.h**](https://github.com/rgot-org/EzLoRaWAN/blob/main/src/target-config.h)

Without AUTOPIN (for example for generic cards) you must define the board type, the SPI and  LoRa chip pins as follow :
- for **SX1262** 
```c 
#define BRD_sx1262_radio 1
#define RADIO_SCLK_PIN              CHANGE_ME // depends on the pinout of the card
#define RADIO_MISO_PIN              CHANGE_ME
#define RADIO_MOSI_PIN              CHANGE_ME
#define RADIO_TX_PIN				LMIC_CONTROLLED_BY_DIO2
#define RADIO_RX_PIN				LMIC_UNUSED_PIN
#define RADIO_CS_PIN                CHANGE_ME
#define RADIO_DIO0_PIN				LMIC_UNUSED_PIN
#define RADIO_DIO1_PIN              CHANGE_ME
#define RADIO_DIO2_PIN				LMIC_UNUSED_PIN
#define RADIO_BUSY_PIN              CHANGE_ME
#define RADIO_RST_PIN               CHANGE_ME
#define RADIO_TCXO_PIN				LMIC_CONTROLLED_BY_DIO3
```
You can see an example for LILYGO T3S3 in [board_config.h](https://github.com/rgot-org/EzLoRaWAN/blob/main/examples/ttn-otaa/board_config.h) in the ttn-otaa example
- for **SX1276**
```c 
#define BRD_sx1276_radio 1
#define RADIO_SCLK_PIN              CHANGE_ME // depends on the pinout of the card
#define RADIO_MISO_PIN              CHANGE_ME
#define RADIO_MOSI_PIN              CHANGE_ME
#define RADIO_CS_PIN                CHANGE_ME
#define RADIO_DIO0_PIN				CHANGE_ME
#define RADIO_DIO1_PIN              CHANGE_ME      
#define RADIO_DIO2_PIN				CHANGE_ME
#define RADIO_RST_PIN               CHANGE_ME
#define RADIO_BUSY_PIN              LMIC_UNUSED_PIN      
#define RADIO_TCXO_PIN				LMIC_UNUSED_PIN
#define RADIO_TX_PIN				LMIC_UNUSED_PIN
#define RADIO_RX_PIN				LMIC_UNUSED_PIN
```

In the sketch declare a `const` as follow:
```c
const lmic_pinmap lmic_pins = {
	.nss = RADIO_CS_PIN,
	.tx = RADIO_TX_PIN,
	.rx = RADIO_RX_PIN,
	.rst = RADIO_RST_PIN,
	.dio = {RADIO_DIO0_PIN , RADIO_DIO1_PIN, RADIO_DIO2_PIN},
	.busy = RADIO_BUSY_PIN,
	.tcxo = RADIO_TCXO_PIN,
};
```
In setup function add :
```c
SPI.begin(RADIO_SCLK_PIN, RADIO_MISO_PIN, RADIO_MOSI_PIN);
```

then call de `begin()` method. (See `ttn-otaa.ino` with `board-config.h` in  [ttn-otaa example](https://github.com/rgot-org/EzLoRaWAN/blob/main/examples/ttn-otaa) for more details)
# API Reference
## Class: EzLoRaWAN

Include and instantiate the EzLoRaWAN  class. The constructor initialize the library with the Streams it should communicate with. 

```c
#include <EzLoRaWAN.h>
EzLoRaWAN  ttn; 
```
# API methods
## Method: `begin`
Start the **LoRaWAN** stack.
The LoRa chip type and the pinout (LoRa & SPI) must be declared before call this method. (see the above and the [target-config.h](https://github.com/rgot-org/EzLoRaWAN/blob/main/src/target-config.h) file for more details)
```c
void begin(); 
```
## Method: `getAppEui`

Gets the provisioned AppEUI. The AppEUI is set using `provision()` or `join()`.

```c
size_t getAppEui(char *buffer, size_t size);
```
return AppEui as an array of char
```c
String getAppEui();
```
return AppEui as String
## Method: `getDevEui`
Gets the provisioned DevEUI. The DevEUI is set using `provision()` or `join()`.
```c
size_t getDevEui(char *buffer, size_t size, bool hardwareEUI=false);
```
return DevEUI as array of char
```c
String getDevEui(bool hardwareEui=false);
```
return DevEUI as String

- `bool hardwareEui=false`: if true get DevEUI from Mac Address.

## Method: `isJoined`
Check whether we have joined TTN
```c
 bool isJoined();
```
return `true` if joined to TTN, `false` if not.
## Method: `showStatus`

Writes information about the device and LoRa module to `Serial` .

```c
void showStatus();
```

Will write something like:

```bash
---------------Status--------------
Device EUI: 004D22F44E5DXXXX
Application EUI: 70B3D57ED001XXXX
netid: 13
devaddr: 2601XXXX
NwkSKey: 6A2D3C24AD3C0D17614D7325BCXXXX
AppSKey: 9E68DCBEBF8AE9D891252FB7EXXXX
data rate: 5
tx power: 14dB
freq: 867100000Hz
-----------------------------------
```

## Method: `onMessage`

Sets a function which will be called to process incoming messages. You'll want to do this in your `setup()` function and then define a `void (*cb)(const byte* payload, size_t length, port_t port)` function somewhere else in your sketch.

```c
void onMessage(void (*cb)(const uint8_t *payload, size_t size, int rssi));
```

- `const uint8_t* payload`: Bytes received.
- `size_t size`: Number of bytes.
- `int rssi`: the rssi in dB.

See the [ttn-otaa]https://github.com/rgot-org/EzLoRaWAN/blob/main/examples/ttn-otaa/ttn-otaa.ino) example.

## Method: `join`

Activate the device via OTAA (default).

```c
bool join();
bool join(const char *app_eui, const char *app_key, int8_t retries = -1, uint32_t retryDelay = 10000);
bool join(const char *dev_eui, const char *app_eui, const char *app_key, int8_t retries = -1, uint32_t retryDelay = 10000);
```

- `const char *app_eui`: Application EUI the device is registered to.
- `const char *app_key`: Application Key assigned to the device.
- `const char *dev_eui`: Device EUI 
- `int8_t retries = -1`: Number of times to retry after failed or unconfirmed join. Defaults to `-1` which means infinite.
- `uint32_t retryDelay = 10000`: Delay in ms between attempts. Defaults to 10 seconds.

Returns `true` or `false` depending on whether it received confirmation that the activation was successful before the maximum number of attempts.

Call the method without the first two arguments if the device's LoRa module is provisioned or comes with NVS stored values. See `provision`, `saveKeys` and `restoreKeys`

## Method: `personalize`

Activate the device via ABP.

```c
bool personalize(const char *devAddr, const char *nwkSKey, const char *appSKey);
bool personalize();
```

- `const char *devAddr`: Device Address assigned to the device.
- `const char *nwkSKey`: Network Session Key assigned to the device for identification.
- `const char *appSKey`: Application Session Key assigned to the device for encryption.

Returns `true` or `false` depending on whether the activation was successful.

Call the method with no arguments if the device's LoRa module is provisioned or comes with NVS stored values. See `provisionABP`, `saveKeys` and `restoreKeys`

See the [ttn_abp](https://github.com/rgot-org/TheThingsNetwork_esp32/tree/master/examples/ttn_abp) example.

## Method: `sendBytes`

Send a message to the application using raw bytes.

```c
bool sendBytes(uint8_t *payload, size_t length, uint8_t port = 1, uint8_t confirm = 0);
```

- `uint8_t *payload `: Bytes to send.
- `size_t length`: The number of bytes. Use `sizeof(payload)` to get it.
- `uint8_t port = 1`: The port to address. Defaults to `1`.
- `uint8_t confirm = false`: Whether to ask for confirmation. Defaults to `false`

## Method: `poll`

Calls `sendBytes()` with `{ 0x00 }` as payload to poll for incoming messages.

```c
int8_t poll(uint8_t port = 1, uint8_t confirm = 0);
```

- `uint8_t  port = 1`: The port to address. Defaults to `1`.
- `uint8_t confirm = 0`: Whether to ask for confirmation.

Returns the result of `sendBytes()`.


## Method: `provision`

Sets the informations needed to activate the device via OTAA, without actually activating. Call join() without the first 2 arguments to activate.

```c
bool provision(const char *appEui, const char *appKey);
bool provision(const char *devEui, const char *appEui, const char *appKey);
```

- `const char *appEui`: Application Identifier for the device.
- `const char *appKey`: Application Key assigned to the device.
- `const char *devEui`: Device EUI.
## Method: `provisionABP`

Sets the informations needed to activate the device via ABP, without actually activating. call `personalize()` without arguments to activate.
```c
bool provisionABP(const char *devAddr, const char *nwkSKey, const char *appSKey);
```
- `const char *devAddr`: Device Address.
- `const char *nwkSKey`: Network Session Key.
- `const char *appSKey`: Application Session Key.
## Method: `saveKeys`
Save the provisioning keys (OTAA and ABP) in Non Volatile Storage (NVS). 
```c
bool saveKeys();
```
## Method: `restoreKeys`
Restore the keys from NVS and provisioning the informations for OTAA or ABP connection. Call `join()` or `Personalize()` after this method to activate the device.
```c
boobool restoreKeys(bool silent=true);
```
- `bool silent=true`: silent mode (no log)



