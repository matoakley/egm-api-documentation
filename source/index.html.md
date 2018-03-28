---
title: EGM Netbox API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl

toc_footers:
  - <a href='mailto:info@rangeball.co.uk'>Support</a>
  - <a href='https://www.startupheroes.co.uk/services/embedded-software-development/'>Embedded Development by Startup Heroes</a>

search: true
---

# Introduction

This documentation details and describes how a third party can integrate with the EGM Netbox device, and subsequently control up to 8 EGM golf ball dispensing machines.

The Netbox operates a RESTful API for communication over HTTP.

The Netbox is a physical device that is connected by serial port to EGM dispensers, and secondly, to a hard-wired LAN connection (Ethernet). The device automatically detects on-line ball dispensers, polling them for their status, and makes this information available via it’s HTTP-driven interface on the LAN connection.

It is the responsibility of the third party (software provider) to check the status of the EGM ball dispensers by using the endpoints below; in particular, [Controller Status](#controller-status) and [Authorise Operation](#authorise-operation).

The status of dispensers will usually be `idle`, indicating that the machine is online and awaiting input. When this status changes to `scan`, for example, then the third party will be required to process the received card data, and take decisive action by sending an [authorise](#authorise-operation) message for the transaction.

To summarize: EGM Netbox will take care of polling ball dispensers, and display any received information, as indicated by this documentation. The third party wishing to integrate shall view this data, extract any relevant parts, and control the ball dispenser by sending appropriate messages to the Netbox.

Further help can be obtained by [contacting EGM](mailto:info@rangeball.co.uk).


# HOSTNAME Discovery

On most local networks, the Netbox device is discoverable by its hostname `netbox.local`.

If you are using a network that does not support DNS then you may opt to discover the device via UDP. The UDP service runs constantly on the device and can be contacted at any time.

To request the IP address of the Netbox via UDP, you should send a UDP broadcast on `port 4040`. The packet data should contain a single string `EGM_NETBOX_DISCOVERY`.

Upon sending the broadcast, you should listen for a response from the Netbox which will contain the IP address of the device.

There are a number of popular libraries for most frameworks that may assist in sending and receiving UDP packets.

# API Versioning

This documentation represents `v1.0.0` of the API. By specifying `/api/v1` as part of your URL, you will ensure that your integration continues to function regardless of any future upgrades to the interface.

Your URL should be constructed as follows: `http://<HOSTNAME>/api/v1/<ENDPOINT>`.

See [HOSTNAME Discovery](#hostname-discovery) and [Endpoints](#endpoints) for further information.

# Addressing Controllers

When building endpoint URLs you will notice a variable `<ADDRESS>` being used. This represents the hostname of IP address of the controller with which you wish to communicate.

Please note that you must include the port number of the Netbox service within the URL of your requests. For most devices the default port will be `8080`.

The address is a value between `1` and `8`. In some cases you can specify `all` to broadcast to every controller, this option is highlighted when available.

# Endpoints

## Controller Status

```shell
curl "http://192.168.0.1:8080/api/v1/controllers"
```

> The above command returns JSON structured like this:

```json
[
  {
    "status": "scan",
    "eprom_version": "03.21",
    "data": {
      "special_rate": true,
      "card_type": "magnetic",
      "cycles": 75,
      "cost": 480,
      "cash_credit": 0,
      "card_data": "BAW02004",
      "basket_size": 3
    "controller_type": "single-ball"
    "address": 1
  },
  {
    "status": "dispensing",
    "eprom_version": "03.21",
    "data": {},
    "controller_type": "single-ball"
    "address": 2
  },
  {
    "status": "idle",
    "eprom_version": "03.21",
    "data": {},
    "controller_type": "single-ball"
    "address": 3
  },
  {
    "status": "idle",
    "eprom_version": "03.21",
    "data": {},
    "controller_type": "tipper"
    "address": 4
  }
]
```

Retrieve a list of all online controllers and their current status.

### HTTP Request

`GET /controllers`

### Controller Properties

Parameter | Type | Description
--------- | ----- | -----------
address | `Integer` | This indicates the address of the controller and is expected to be between `1` and `8`.
data | `Object` | Populated during `scan`, see [Controller Data](#controller-data).
controller_type | `String` | Specifies the type of controller either `single-ball` or `tipper`.
eprom_version | `String` | Firmware version of controller chipset.
status | `String` | `idle` - The controller is online and awaiting user input.
 | | `scan` - A user has scanned a card and controller is awaiting a response.
 | | `dispensing` -Controller is currently dispensing balls.
 | | `setup_mode` - Controller is in SETUP mode.
 | | `engineer_mode` - Controller is in ENGINEER mode.
 | | `log_updated` - Activity logs have been updated since last read.
 | | `test_mode` - Controller is in TEST mode.
 | | `out_of_order` - Error present on controller.
 | | `change_hopper_empty` - Controller's change hopper is empty.
 | | `keypad_entry` - TODO
 | | `ball_hopper_low` - Controller's ball hopper is low.

### Controller Data

Parameter | Type | Description
----------|-------|------------
special_rate | `Boolean` | Whether special rate is selected at controller.
card_type | `String` | The type of card that has been read, either `magnetic` or `barcode`.
cycles | `String` | Delivery cycles for selected size.
cost | `String` | Cost of selected size.
cash_credit | `String` | Amount of cash credit at controller.
card_data | `String` | Data read from card scan, this represents the card identifier.
basket_size | `String` | Selected basket size at controller.




## Authorise Operation

> To dispense balls and display details of the transaction on controller:

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/authorise"
  -X POST
  -H "Content-Type: application/json"
  -d '{
        "authorise": {
          "cycles": 50,
          "name": "Joe Bloggs",
          "cost": "3.50",
          "balance: "16.50",
          "transfer_credit": 0,
          "log_voucher": 0
        }
      }'
```

> To tell user they need to top up with custom message:

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/authorise"
  -X POST
  -H "Content-Type: application/json"
  -d '{
        "authorise": {
          "cycles": 0,
          "message": "There is not enough credit available\nPlease top up!",
          "transfer_credit": 0,
          "log_voucher": 0
        }
      }'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Acknowledge card/voucher operation and authorise action at the controller.

<aside class="warning">If an authrorise response is not received within 60 seconds of a card read, the controller will assume the server is offline and return to Home Screen.</aside>

### HTTP Request

`POST /controllers/<ADDRESS>/authorise`


### POST Data

Parameter | Type | Description
--------- | ---- | -----------
cycles | `Integer` | Number of delivery cycles. Send `0` if no balls should be dispensed.
transfer_credit | `Boolean` | Set `true` to deduct cash top-up value from cash-credit store.
log_voucher | `Boolean` | Set `true` to log delivery as voucher.
message | `String` | (Optional) ASCII message to display at controller, use `\n` for new line.
name | `String` | (Optional) The name of the card owner.
cost | `String` | (Optional) Cost of the transaction.
balance | `String` | (Optional) Remaining balance on card owner's account.

<aside class="notice">You are expected to pass <strong>either</strong> a <code>message</code> to display at the machine <strong>or alternatively</strong> each of <code>name</code>, <code>cost</code> & <code>balance</code> and a default message will be displayed.</aside>



## Set Offline

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1"
  -X DELETE
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the controller status to "NOT IN USE", displaying a temporary message on the controller.

### HTTP Request

`DELETE /controllers/<ADDRESS>`

<aside class="success">ADDRESS can be set to <code>all</code> to affect all online controllers.</aside>




## Set Online

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1"
  -X POST
  -H "Content-Type: application/json"
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the controller status to "IN USE".

### HTTP Request

`POST /controllers/<ADDRESS>`

<aside class="success">ADDRESS can be set to <code>all</code> to affect all online controllers.</aside>




## Read Calendar

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/calendar"
  -H "Content-Type: application/json"
```
> The above command returns JSON structured like this:

```json
{
  "year": "17",
  "month": "12",
  "day_of_month": "02",
  "hour": "12",
  "minute": "00",
  "second": "00",
  "day_of_week": "02"
}
```

Read the current date/time set on the controller's calendar.

### HTTP Request

`GET /controllers/<ADDRESS>/calendar`

### Day of Week

Value | Day of Week
--------- | -----------
`01` | Sunday
`02` | Monday
`03` | Tuesday
`04` | Wednesday
`05` | Thursday
`06` | Friday
`07` | Saturday



## Set Calendar

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/calendar"
  -X POST
  -H "Content-Type: application/json"
  -d '{
        "calendar": {
          "year": "17",
          "month": "12",
          "day_of_month": "02",
          "hour": "12",
          "minute": "00",
          "second": "00",
          "day_of_week": "02"
        }
      }'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the date/time set on the controller's calendar.

### HTTP Request

`POST /controllers/<ADDRESS>/calendar`

<aside class="success">ADDRESS can be set to <code>all</code> to affect all online controllers.</aside>

### POST Data

Parameter | Type | Description
--------- | ---- | -----------
year | `String` | 2 character format, e.g. `18` for 2018
month | `String` | 2 character format, e.g. `02` for February
day_of_month | `String` | 2 character format, e.g. `15`
hour | `String` | 2 character format for 24 hour clock, e.g. `13`
minute | `String` | 2 character format, e.g. `30`
second | `String` | 2 character format, e.g. `00`
day_of_week | `String` | `01` - Sunday
 | | `02` - Monday
 | | `03` - Tuesday
 | | `04` - Wednesday
 | | `05` - Thursday
 | | `06` - Friday
 | | `07` - Saturday

<aside class="notice">Note that after setting the calendar, the controller will temporarily go offline to recalibrate its logs.</aside>



## Read Rates

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/rates"
  -H "Content-Type: application/json"
```
> The above command returns JSON structured like this:

```json
{
  "sizes_available": 4,
  "rates": {
    "special": {
      "token_1": {
        "cycles": 50
      },
      "token_2": {
        "cycles": 100
      },
      "token_3": {
        "cycles": 150
      },
      "token_4": {
        "cycles": 200
      },
      "ccr": {
        "cycles": 20
      },
      "baskets": {
        "basket_1": {
          "cycles": 25,
          "price": 150
        }
        "basket_2": {
          "cycles": 50,
          "price": 240
        },
        "basket_3": {
          "cycles": 75,
          "price": 310
        },
        "basket_4": {
          "cycles": 100,
          "price": 360
        },
      },
      "aux": {
        "cycles": 50
      }
    },
    "normal": {
      "token_1": {
        "cycles": 25
      },
      "token_2": {
        "cycles": 50
      },
      "token_3": {
        "cycles": 75
      },
      "token_4": {
        "cycles": 100
      },
      "ccr": {
        "cycles": 10
      },
      "baskets": {
        "basket_1": {
          "cycles": 25,
          "price": 300
        }
        "basket_2": {
          "cycles": 50,
          "price": 480
        },
        "basket_3": {
          "cycles": 75,
          "price": 620
        },
        "basket_4": {
          "cycles": 100,
          "price": 720
        },
      },
      "aux": {
        "cycles": 25
      }
    }
  }
}
```

Read the current rates set on the controller.

### HTTP Request

`GET /controllers/<ADDRESS>/rates`

### Response Data

Parameter | Type | Description
--------- | ---- | -----------
sizes_available | `Integer` | Number of basket sizes available on controller.
cycles | `Integer` | Size of currently described rate.
price | `Integer` | Price in pence of currently described rate.


## Set Rates

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/rates"
  -X POST
  -H "Content-Type: application/json"
  -d '{
        "rates": {
          "sizes_available": 4,
          "rates": {
            "special": {
              "token_1": {
                "cycles": 50
              },
              "token_2": {
                "cycles": 100
              },
              "token_3": {
                "cycles": 150
              },
              "token_4": {
                "cycles": 200
              },
              "ccr": {
                "cycles": 20
              },
              "baskets": {
                "basket_1": {
                  "cycles": 25,
                  "price": 150
                }
                "basket_2": {
                  "cycles": 50,
                  "price": 240
                },
                "basket_3": {
                  "cycles": 75,
                  "price": 310
                },
                "basket_4": {
                  "cycles": 100,
                  "price": 360
                },
              },
              "aux": {
                "cycles": 50
              }
            },
            "normal": {
              "token_1": {
                "cycles": 25
              },
              "token_2": {
                "cycles": 50
              },
              "token_3": {
                "cycles": 75
              },
              "token_4": {
                "cycles": 100
              },
              "ccr": {
                "cycles": 10
              },
              "baskets": {
                "basket_1": {
                  "cycles": 25,
                  "price": 300
                }
                "basket_2": {
                  "cycles": 50,
                  "price": 480
                },
                "basket_3": {
                  "cycles": 75,
                  "price": 620
                },
                "basket_4": {
                  "cycles": 100,
                  "price": 720
                },
              },
              "aux": {
                "cycles": 25
              }
            }
          }
        }
      }'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the rates on the controller.

### HTTP Request

`POST /controllers/<ADDRESS>/rates`

<aside class="success">ADDRESS can be set to <code>all</code> to affect all online controllers.</aside>

### POST Data

Parameter | Type | Description
--------- | ---- | -----------
sizes_available | `Integer` | Number of basket sizes available on controller.
cycles | `Integer` | Size of currently described rate.
price | `Integer` | Price in pence of currently described rate.




## Set Rate Mode

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/rate-mode"
  -X POST
  -H "Content-Type: application/json"
  -d '{
        "rate_mode": {
          "mode": "schedule",
          "sunday": {
            start: "1000",
            end: "1400"
          },
          "monday": {
            start: "1200",
            end: "1400"
          },
          "tuesday": {
            start: "1200",
            end: "1400"
          },
          "wednesday": {
            start: "1200",
            end: "1400"
          },
          "thursday": {
            start: "1200",
            end: "1400"
          },
          "friday": {
            start: "1200",
            end: "1400"
          },
          "saturday": {
            start: "1200",
            end: "1400"
          }
        }
      }'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the rate mode on the controller.

### HTTP Request

`POST /controllers/<ADDRESS>/rate-mode`

<aside class="success">ADDRESS can be set to <code>all</code> to affect all online controllers.</aside>

### POST Data

Parameter | Type | Description
--------- | ---- | -----------
mode | `String` | `schedule` - Set rate mode automatically according to schedule.
 | | `manual` - Override schedule and enable special rate mode.
start | `String` | Time to start special rate mode in 24-hour format.
end | `String` | Time to end special rate mode in 24-hour format.

<aside class="warning">Start and end times for each day must always be specified regardless of <code>mode</code>.</aside>



## Display Message

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/display"
  -X POST
  -H "Content-Type: application/json"
  -d '{
        "display": {
          "duration": 60,
          "text": "Special rates available until 12:00"
        }
      }'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set a message to be displayed on the controller for a given period.

### HTTP Request

`POST /controllers/<ADDRESS>/display`

<aside class="success">ADDRESS can be set to <code>all</code> to affect all online controllers.</aside>

### POST Data

Parameter | Type | Description
--------- | ---- | -----------
duration | `Integer` | Number of minutes to display message, must be between `1` and `60`
 | | Set to `99` to display message indefinitely.
 | | Set to `0` to clear previously set message.
text | `String` | ASCII message to be displayed, use `\n` for new line




## Read Activity Logs

```shell
curl "http://192.168.0.1:8080/api/v1/controllers/1/activity?day_of_month=4&month=11&year=17&hour=12"
  -H "Content-Type: application/json"
```

> The above command returns JSON structured like this:

```json
{
  "day": 20,
  "month": 12,
  "year": 17,
  "hour": 12,
  "top_up_cash": 0,
  "token_1_collected": 1,
  "token_2_collected": 0,
  "token_3_collected": 0,
  "token_4_collected": 0,
  "test_mode_cycles": 0,
  "test_mode_change": 0,
  "test_mode_cash": 0,
  "slave_mode_cycles": 0,
  "keypad_cycles": 0,
  "change_given": 0,
  "ccr_2_operations": 1,
  "cash_collected": 1650,
  "card_cycles": 0,
  "basket_1": {
    "dispensed": 1,
    "cycles": 25,
    "cash": 100
  },
  "basket_2": {
    "dispensed": 1,
    "cycles": 50,
    "cash": 200
  },
  "basket_3": {
    "dispensed": 1,
    "cycles": 75,
    "cash": 300
  },
  "basket_4": {
    "dispensed": 1,
    "cycles": 100,
    "cash": 600
  },
  "aux_operations": 1
}
```

Read activity logs from controller for a given hour.

### HTTP Request

`GET /controllers/<ADDRESS>/activity`

### QUERY Parameters

Parameter | Type | Description
--------- | ---- | -----------
day | `Integer` | Defaults to current day.
month | `Integer` | Defaults to current month.
year | `Integer` | Defaults to current year.
hour | `Integer` | 24-hour format, defaults to current hour.
