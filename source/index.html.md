---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction


# Authentication

# Discovery

# Controllers

## Controller Status

```shell
curl "http://192.168.0.1/controllers"
```

> The above command returns JSON structured like this:

```shell
[
  {
    "status": "scan",
    "eprom_version": "0321",
    "data": {
      "special_rate": true,
      "card_type": "magnetic",
      "cycles": "0000",
      "cost": "00000000",
      "cash_credit": "00000000",
      "card_data": "BAW02004",
      "basket_size": "0000"
    "controller_type": "single-ball"
    "address": 1
  },
  {
    "status": "dispensing",
    "eprom_version": "0321",
    "data": {},
    "controller_type": "single-ball"
    "address": 2
  },
  {
    "status": "idle",
    "eprom_version": "0321",
    "data": {},
    "controller_type": "single-ball"
    "address": 3
  },
  {
    "status": "idle",
    "eprom_version": "0321",
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

Parameter | Value | Description
--------- | ------- | -----------
address | `Integer` | This indicates the address of the controller and is expected to be between `1` and `8`.
data | `Object` | See [Controller Data](#controller-data).
controller_type | `String` | Specifies the type of controller either `single-ball` or `tipper`.
eprom_version | `String` | Firmware version of controller chipset.
status | `String` | `idle` - The controller is online and awaiting user input.
 | | `scan` - A user has scanned a card and controller is awaiting a response.
 | | `dispensing` - TODO
 | | `setup_mode` - TODO
 | | `engineer_mode` - TODO
 | | `log_updated` - TODO
 | | `test_mode` - TODO
 | | `out_of_order` - TODO
 | | `change_hopper_empty` - TODO
 | | `keypad_entry` - TODO
 | | `ball_hopper_low` - TODO

### Controller Data




## Authorise Operation

```shell
curl "http://192.168.0.1/controllers/1/authorise"
  -X POST
  -d '{}'
```
> The above command returns JSON structured like this:

```json
{
}
```

Acknowledge card/keypad operation and authorise action at the controller.

### HTTP Request

`POST /controllers/<ADDRESS>/authorise`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to authorise

### POST Data

Parameter | Description
--------- | -----------
message | foo




## Set Offline

```shell
curl "http://192.168.0.1/controllers/1"
  -X DELETE
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the controller status to "NOT IN USE", displaying a temporary message on the controller.

### HTTP Request

`DELETE /controllers/<ADDRESS>`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to set offline, accepts `all`




## Set Online

```shell
curl "http://192.168.0.1/controllers/1"
  -X POST
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the controller status to "IN USE".

### HTTP Request

`POST /controllers/<ADDRESS>`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to set online, accepts `all`




## Read Calendar

```shell
curl "http://192.168.0.1/controllers/1/calendar"
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

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to query

### Day of Week

Value | Day of Week
--------- | -----------
01 | Sunday
02 | Monday
03 | Tuesday
04 | Wednesday
05 | Thursday
06 | Friday
07 | Saturday



## Set Calendar

```shell
curl "http://192.168.0.1/controllers/1/calendar"
  -X POST
  -d '{
        "year": "17",
        "month": "12",
        "day_of_month": "02",
        "hour": "12",
        "minute": "00",
        "second": "00",
        "day_of_week": "02"
      }'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the date/time set on the controller's calendar.

### HTTP Request

`POST /controllers/<ADDRESS>/calendar`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to write, accepts `all`

### POST Data

Parameter | Description
--------- | -----------
year | 2 digit string format, e.g. "18" for 2018
month | 2 digit string format, e.g. "02" for February
day_of_month | 2 digit string format, e.g. "15"
hour | 2 digit string format for 24 hour clock, e.g. "13"
minute | 2 digit string format, e.g. "30"
second | 2 digit string format, e.g. "00"
day_of_week | 2 digit string format, see table below

### Day of Week

Value | Day of Week
--------- | -----------
"01" | Sunday
"02" | Monday
"03" | Tuesday
"04" | Wednesday
"05" | Thursday
"06" | Friday
"07" | Saturday



## Read Rates

```shell
curl "http://192.168.0.1/controllers/1/rates"
```
> The above command returns JSON structured like this:

```json
{
  "sizes_available": "04",
  "rates": {
    "special": {
      "token_4": {
        "cycles": "0000"
      },
      "token_3": {
        "cycles": "0000"
      },
      "token_2": {
        "cycles": "0000"
      },
      "token_1": {
        "cycles": "0000"
      },
      "ccr": {
        "cycles": "0000"
      },
      "baskets": {
        "basket_4": {
          "size": "0000",
          "price": "000000"
        },
        "basket_3": {
          "size": "0000",
          "price": "000000"
        },
        "basket_2": {
          "size": "0000",
          "price": "000000"
        },
        "basket_1": {
          "size": "0000",
          "price": "000000"
        }
      },
      "aux": {
        "cycles":"0025"
      }
    },
    "normal": {
      "token_4": {
        "cycles": "0095"
      },
      "token_3": {
        "cycles": "0080"
      },
      "token_2": {
        "cycles": "0050"
      },
      "token_1": {
        "cycles": "0025"
      },
      "ccr": {
        "cycles": "0000"
      },
      "baskets": {
        "basket_4": {
          "size": "0100",
          "price": "000720"
        },
        "basket_3": {
          "size": "0075",
          "price": "000620"
        },
        "basket_2": {
          "size": "0050",
          "price": "000480"
        },
        "basket_1": {
          "size": "0025",
          "price": "000300"
        }
      },
      "aux": {
        "cycles": "0025"
      }
    }
  }
}
```

Read the current rates set on the controller.

### HTTP Request

`GET /controllers/<ADDRESS>/rates`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to write




## Set Rates

```shell
curl "http://192.168.0.1/controllers/1/rates"
  -X POST
  -d '{}'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the rates on the controller.

### HTTP Request

`POST /controllers/<ADDRESS>/rates`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to write, accepts `all`

### POST Data

Parameter | Description
--------- | -----------
message | TODO




## Set Rate Mode

```shell
curl "http://192.168.0.1/controllers/1/rate_mode"
  -X POST
  -d '{}'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set the rate mode on the controller.

### HTTP Request

`POST /controllers/<ADDRESS>/rate_mode`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to write, accepts `all`

### POST Data

Parameter | Description
--------- | -----------
message | TODO




## Display Message

```shell
curl "http://192.168.0.1/controllers/1/display"
  -X POST
  -d '{
        "duration": "60",
        "text": "Special rates available until 12:00"
      }'
```
> The above command returns JSON structured like this:

```json
"ok"
```

Set a message to be displayed on the controller for a given period.

### HTTP Request

`POST /controllers/<ADDRESS>/display`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to write, accepts `all`

### POST Data

Parameter | Description
--------- | -----------
duration | Number of minutes to display message in 2 digit string format, e.g. "30"
text | ASCII message to be displayed, use `\n` for new line




## Read Activity Logs

```shell
curl "http://192.168.0.1/controllers/1/activity?day_of_month=04&month=11&year=17&hour=12"
```

> The above command returns JSON structured like this:

```json
TODO
```

Read activity logs from controller for a given hour.

### HTTP Request

`GET /controllers/<ADDRESS>/activity`

### URL Parameters

Parameter | Description
--------- | -----------
ADDRESS | The ADDRESS of the controller to read

### GET Parameters

Parameter | Description
--------- | -----------
day_of_month | TODO
month | TODO
year | TODO
hour | TODO
