![TAConnections Logo](./images/ta-connections-logo-small.jpg)

# StormX REST API Documentation 

Version $RELEASE_VERSION$

**Confidential.**

Copyright (c) 2017-2022 TAConnections, inc.



[TOC]



## About this document

This is a technical document that explains how your Airline can integrate with TAConnections's StormX REST API.

Using the StormX REST API, your Airline can automate the process of accommodating distressed passengers with hotels, meal vouchers, and transportation to hotels.


### Intended Audience

This document is for Airline employees or contractors that are implementing the software integration between your Airline's systems and the TAConnections StormX system.

This document assumes that the audience is familiar with HTTP, RESTful APIs, JSON, and programming languages.


## Authentication

The StormX API consists of an Airline API, and a Passenger API.

The Airline API implements Basic Authentication.

For the Passenger API, authentication will be handled through two query string parameters containing GUID values.
`ak1`, and `ak2` (Auth Key 1 and Auth Key 2) will be passed in as query string parameters.

These unique keys are generated from the Passenger Import process.
The keys will be passed back in the passenger import response as `ak1`, and `ak2` for every unique passenger.

**Security Note:** Keep access to `ak1` and `ak2` limited to just the passenger who they belong to.
Anyone who has access to a passenger's `ak1` and `ak2` can access their voucher information.



## General API Response Conventions

All API endpoints follow a similar response structure.

HTTP 2XX status codes indicate success and any other status code indicates failure.
Additionally, clients should check the `error` and `meta.message` fields present in the JSON body of the response.

The API always returns JSON unless otherwise noted. In some exceptional circumstances (5XX responses due to system outages, etc.) JSON bodies may not be available.

**General response format:**<br/>
Here is the general JSON body returned in the responses:
```
{
    "error": <Boolean>,
    "data": <usually Object, Array, or null>,
    "meta": {
        "error_code": <String>,
        "error_description": <String>,
        "error_detail": <list of Objects>,
        "message": <String>,
        "href": <requested URL>,
        "method": <String>,
        "status": <Integer>
    }
}
```

**General response fields:**<br/>
* **error:** Boolean, **NOTE: important field!**  indicates whether or not an error occurred. We recommend always checking this value, even in the case of an HTTP 2XX response code.
* **data:** Usually an Object, an Array, or `null`. The data type and structure depends on the specific endpoint being accessed. For example, `data` could be an Object containing Passenger information, or an Array of hotel objects.
* **meta:** Object. contains some information about the request, error information if available, and a copy of some of the HTTP response information.
* **meta.error_code:** String, a machine-readable enumeration describing the type of problem occurring. Each endpoint has a different set of `error_code` enumerations that are possible.
  * **Note:** This field can be particularly useful if your airline wishes to build logic around certain classes of errors programmatically.
  * Here are a few of the common ones:
    * `INVALID_JSON` - the data POSTed to the API was not valid JSON data.
    * `INVALID_INPUT` - the GET query parameters or POST JSON data failed validation. One or more fields are missing or incorrect somehow. Generally, `error_detail` can show you which fields are invalid.
* **meta.error_message:** String, A human-readable explanation of the problem.
* **meta.error_detail:** List of Objects, explains which fields have incorrect input values and the reason why the field is invalid.
  * Each object in the `error_detail` list has the form:
    ```
    {
      "field": <String>,
      "message": <String>
    }
    ```
  * `field` can be things like `"number_of_nights"` for simple input structures or something like `"[0].ticket_level"` to indicate a particular field in a nested structure of inputs.


**General response echo fields:**<br/>

These fields are useful for developer convenience, tracing requests, and debugging.

* **meta.href:** String, echo of the requested URL.
* **meta.method:** String. echo of the request method (i.e., `"GET"`, `"PUT"`, etc.)
* **meta.status:** Integer. echo of the HTTP response's status code.
* **meta.message:** String, echo of the HTTP reason (i.e., "OK", "Bad Request", etc.)


## Common Response Objects

These blocks of data are common response patterns returned by various API endpoints.
These are referenced by name in the specific endpoints that use them.

### PassengerResponseObject
This section contains all fields returned in a response relating to a Passenger Object used by the Airline API.

**Fields from airline-provided data:**<br/>

See [Passenger Import](#passenger-import) input payload section for detailed explanation of all the fields provided by the airline during import.

**Additional system-generated passenger object fields:**<br/>
* **ak1:** UUID. Authorization Key used by passenger api for authentication.
* **ak2:** UUID. Authorization Key used by passenger api for authentication.
* **airline_id:** Integer. Airline ID for airline associated with passenger.
* **canceled_date:** Datetime. Time in UTC that the passenger's offer was canceled.
* **create_date:** Datetime. Time in UTC that the passenger's offer was inserted into the system.
* **declined_date:** Datetime. Time in UTC that the passenger's offer was declined.
* **expiration_date:** Datetime. Time in UTC that the passenger's offer will expire. expiration_date is 24 hours greater than the passenger's create_date. The passenger has up to this date to act on their offer.
* **modified_date:** Datetime. Time in UTC that the passenger's offer was last acted upon or when their information was last updated.
* **offer_url:** String. URL for the passenger to view their offer via the passenger app. May be `null` if `passenger.life_stage="child"`.
* **voucher_id:** UUID. ID of the voucher. `voucher_id` will be populated once a passenger has meal or hotel vouchers. May be `null` if passenger has no voucher.
* **hotel_accommodation_status:** String. Current status of the hotel offer. Possible values: not_offered, offered, accepted, declined, canceled_offer, or canceled_voucher.
* **meal_accommodation_status:** String. Current status of the meal offer. Possible values: not_offered, offered, accepted, declined, or canceled_offer.
* **transport_accommodation_status:** String. Always returned as null for now (functionality in future release). Current status of the transportation offer.
* **offer_opened_date:** offer_url: DateTime. Initial date that the passenger opened the `offer_url`. May be `null` if passenger has not opened `offer_url`.
* **hotel_allowance_status:** [HotelAllowanceStatusObject](#hotelallowancestatusobject)

**Example value:**<br/>
```json
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": "35a40f0f382a4655922b07fd74a1ab00",
            "ak2": "a109910d069645cc9cbef637ad26b652",
            "canceled_date": null,
            "context_id": "adultCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [
                "famousjj@aol.com"
            ],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "Jet",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "not_offered",
                "breakfast": "not_offered",
                "dinner": "not_offered",
                "lunch": "not_offered"
            },
            "last_name": "Jackson",
            "life_stage": "adult",
            "meal_accommodation": true,
            "meal_accommodation_status": "offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [
                {
                    "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                    "notification_type": "offer",
                    "sent_date": null,
                    "sent_to": "+13125551234",
                    "sent_via": "text"
                },
                {
                    "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                    "notification_type": "offer",
                    "sent_date": null,
                    "sent_to": "famousjj@aol.com",
                    "sent_via": "email"
                }
            ],
            "notify": true,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": "https://sxa018.tvlinc.com/offer?ak1=35a40f0f382a4655922b07fd74a1ab00&ak2=a109910d069645cc9cbef637ad26b652",
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [
                "+13125551234"
            ],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": "a5917c45-8085-4e1f-9d1f-705d13c9d5b8"
        }
```

### HotelResponseObjectForPassenger
Note: this is similar to [HotelResponseObjectForAirline](#hotelresponseobjectforairline), but it contains fewer fields, as some are not appropriate for the Passenger to know about.

### HotelResponseObjectForAirline

**General hotel information fields:**<br/>
* **hotel_id:** String (length up to 128 characters). this is the unique identifier used to book a hotel, etc (NOTE: this string should be considered case-sensitive, and may contain uppercase and lowercase letters).
    * Here are two examples to express the potential range:
      * `"tvl-83845Xz9ff-Njg1MjFlNTk5YjFmOWY5ZTIxMTQzYjQ.2"`
      * `"ean-Yjg2Y2UyM1M2E0Zjg0ZjZlZGFkNTk3NzkyY2RlY2MyYzYyMDUxMjkwYzc3OGJjODBkODUyYjcxZGEwOWQwYTI5YTg1MDg3N2NDYwZDV.OGQ1NmFjNW-A5NTA1Ndm"`
* **hotel_name:** String. the name of the hotel, suitable for display.
* **image_url:** String. A picture of the hotel. May be `null` if image is unavailable.
    * Note: the aspect ratios of these pictures varies.
* **airport_code** String. The airport this hotel services. Examples: `"LAX"`, `"JFK"`, etc.
* **phone_number:** String. The phone number to call the front desk of the hotel. Also the phone number that can be called to request a hotel shuttle.
* **provider:** String (length up to 5 characters). The hotel inventory provider. Current providers are `"tvl"` (TAConnections), and `"ean"` (Expedia), but expect other providers in the future.

**Hotel availability information:**<br/>
* **available:** Integer, total number of rooms available in this hotel. Includes the sum of all soft block rooms and your airline's open hard block rooms.
(NOTE: for `multi-night` availability responses, the `available` field value will be the lowest value compared across all days of the request.)
* **hard_block_count:** Integer, the number of hard block rooms available to your airline (available rooms your airline has already paid for).
(NOTE: for `multi-night` availability responses, the `hard_block_count` field value will be the average value compared across all days of the request.)
* **contract_block_count:** Integer, the number of contract block rooms available to your airline (available rooms your airline has already paid for).
(NOTE: for `multi-night` availability responses, the `contract_block_count` field value will be the average value compared across all days of the request.)
* **proposed_check_in_date:** Date, YYYY-MM-DD. Proposed check in date in local timezone of port of accommodation. NOTE: If hour is between 12:00 AM and 5:00 AM in local timezone of port of accommodation, proposed_check_in_date may be a date in the past.
* **proposed_check_out_date:** Date, YYYY-MM-DD. Proposed check out date in local timezone of port of accommodation.

**Hotel services and amenities:**
* **star_rating:** Integer. 1-5. The quality rating of the hotel.
* **premium:** Boolean. The hotel is premium or not.
* **amenities:** Array of [AmenityResponseObject](#amenityresponseobject)s
* **restaurant_on_property:** Boolean. The hotel has a restaurant on property.
* **pets_allowed:** Boolean.
* **service_pets_allowed:** Boolean.
* **hotel_on_airport:** Boolean. Hotel is physically attached to or inside of the airport. A passenger could potentially walk to the hotel.
* **hotel_on_airport_instructions:** String (length up to 255 characters). Instructions that tell passengers how to go to the hotel from the airport. NOTE: will be an empty string if the hotel is not attached to an airport (`"hotel_on_airport": false`) or no instructions are present.
* **passenger_note:** String (length up to 255 characters). A note from the hotel to the passenger.
The StormX passenger app will use this field to display information to the passenger from the hotel.
example: "Shuttle at door G3"
* **hotel_allowances:** [HotelAllowanceResponseObject](#hotelallowanceresponseobject)

**Hotel pricing fields:**<br/>
* **currency_code:** String. Currency code used for the pricing fields of the hotel. Currently just `"USD"`.
* **rate:** String Decimal. cost per night of the hotel. Note: this does not include `tax`, `pets_fee`, or `service_pets_fee`.
* **tax:** String Decimal.
* **pets_fee:** String Decimal.
* **service_pets_fee:** String Decimal.

**Hotel address fields:**<br/>
* **address1:** String.
* **address2:** String. Can be used to supply additional address lines in some locations. Empty string if not used.
* **address3:** String. Can be used to supply additional address lines in some locations. Empty string if not used.
* **address4:** String. Can be used to supply additional address lines in some locations. Empty string if not used.
* **city:** String.
* **state:** String.
* **post_code:** String.
* **country:** String.

**Hotel positional information:**<br/>
* **latitude:** Float. Units: degrees.
* **longitude:** Float. Units: degrees.
* **distance:** Float. Distance from the airport in miles.
    * Note: this is a direct distance. Actual driving distance will often be more.

**Hotel shuttle information:**<br/>
* **shuttle:** Boolean. Whether or not the hotel offers a free shuttle for passengers passengers to go from the airport to the hotel.
* **shuttle_timing:** String. The shuttle schedule. Can be empty string.
    - Examples:
        * `""` - indicates no shuttle schedule available.
        * `"0:00 23:59"` - indicates 24-hour shuttle. This is common.
        * `"9:30 22:30"` - indicates the shuttle runs from 9:30 AM to 10:30 PM.
        * `"6:00 1:00"` - indicates the shuttle runs from 6:00 AM to 1:00AM the next morning (note the second time can be lower!)
        * `"10:00 23:15"` - indicates the shuttle runs from 10:00 AM to 11:15 PM.
* **shuttle_available_now:** Boolean. Whether or not the shuttle is available at the time of the request. Current time is moved 30 mins forward to account for passenger movement (this results in shuttle not being availabe at 23:30 even if the hotel has shuttle until 0:00).

**Example value:**<br/>
```json
{
    "address1": "690 Newport Center Dr",
    "address2": "",
    "address3": "",
    "address4": "",
    "airport_code": "LAX",
    "amenities": [
        {
            "name": "Breakfast"
        },
        {
            "name": "Lunch"
        },
        {
            "name": "Dinner"
        },
        {
            "name": "Pool"
        },
        {
            "name": "Fitness Center"
        },
        {
            "name": "Cocktail Hour"
        }
    ],
    "available": 60,
    "city": "Los Angeles",
    "country": "USA",
    "currency_code": "USD",
    "distance": 0.5438059704292102,
    "hard_block_count": 0,
    "contract_block_count": 0,
    "hotel_allowances": {
        "amenity": {
           "amount": "50.00"
        },
        "meals": {
            "breakfast": {
                "rate": "10.00"
            },
            "dinner": {
                "rate": "20.00"
            },
            "lunch": {
                "rate": "30.00"
            }
        }
    },
    "hotel_id": "tvl-84399",
    "hotel_name": "Island Hotel Newport Beach",
    "hotel_on_airport": false,
    "hotel_on_airport_instructions": "",
    "image_url": "https://i.travelapi.com/hotels/1000000/20000/14200/14197/14197_183_b.jpg",
    "latitude": 33.630133,
    "longitude": -117.872168,
    "passenger_note": "Shuttle at door G3",
    "pets_allowed": false,
    "pets_fee": "0.00",
    "phone_number": "19497590808",
    "post_code": "92660",
    "proposed_check_in_date": "2018-05-03",
    "proposed_check_out_date": "2018-05-04",
    "premium": true,
    "provider": "tvl",
    "rate": "110.00",
    "service_pets_allowed": false,
    "service_pets_fee": "0.00",
    "shuttle": true,
    "shuttle_timing": "0:00 23:59",
    "shuttle_available_now": true,
    "star_rating": 4,
    "state": "CA",
    "tax": "9.57"
}
```

### VoucherPassengerResponseObjectForPassenger
Note: this is similar to [VoucherPassengerResponseObjectForAirline](#voucherpassengerresponseobjectforairline), but it contains fewer fields, as some are not appropriate for the Passenger to know about.

### VoucherPassengerResponseObjectForAirline
This object is a Passenger object that is a member of the Voucher.
It contains fewer fields than the [PassengerResponseObject](#passengerresponseobject)
but contains all the fields that may be changed during a `Book Hotel`, `Decline Offer`, or `Cancel Voucher` request.
<br><br>
Note: for detailed description of passenger fields, refer to [PassengerResponseObject](#passengerresponseobject).
<br><br>
**General hotel information fields:**<br/>
* **context_id** `passenger.context_id`
* **modified_date** `passenger.modified_date`
* **canceled_date** `passenger.canceled_date`
* **declined_date** `passenger.declined_date`
* **offer_opened_date** `passenger.offer_opened_date`
* **hotel_accommodation_status** `passenger.hotel_accommodation_status`
* **meal_accommodation_status** `passenger.meal_accommodation_status`
* **transport_accommodation_status** `passenger.transport_accommodation_status`
* **meal_vouchers** an array of [MealVoucherResponseObject](#mealvoucherresponseobject)s.
* **notifications** an array of [PassengerNotificationResponseObject](#passengernotificationresponseobject)s.
* **hotel_allowances_voucher:** [PassengerHotelAllowanceResponseObject](#passengerhotelallowanceresponseobject)
* **hotel_allowance_status:** [HotelAllowanceStatusObject](#hotelallowancestatusobject)
* **ride_vouchers:** [RideVoucherObject](#ridevoucherobject)s

**Example:**
```json
{
    "context_id": "adultCON",
    "canceled_date": null,
    "declined_date": null,
    "hotel_accommodation_status": "accepted",
    "hotel_allowances_voucher": {
        "amenity": {
            "amount": "40.00"
        },
        "meals": {
            "breakfast": {
                "count": 1,
                "rate": "10.00",
                "total": "10.00"
            },
            "dinner": {
                "count": 1,
                "rate": "30.00",
                "total": "30.00"
            },
            "lunch": null
        }
    },
    "hotel_allowance_status": {
        "amenity": "accepted",
        "breakfast": "accepted",
        "lunch": "not_offered",
        "dinner": "accepted"
    },
    "meal_accommodation_status": "accepted",
    "meal_vouchers": [
        {
            "active_from": "2018-04-23 15:22",
            "active_to": "2018-04-24 20:59",
            "amount": "15.00",
            "billing_zip_code": "55343",
            "card_number": "0000000001469006",
            "card_type": "MASTERCARD",
            "currency_code": "USD",
            "cvc2": "1234",
            "expiration": "04/2021",
            "id": "43c69ff0-1720-4bfe-ace6-3d48135fb249",
            "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiWWtGdkt2ZGRTU1NoNm8zZlU2eUY3dERvUlFGREdFbndzT1BQdWpPUFd3ayJ9.2JDYOj-nNQocumDTGOtcRAFRQsQ/meal.png",
            "time_zone": "America/Los_Angeles",
            "provider": "tvl",
            "url": null
        }
    ],
    "modified_date": "2018-04-23 22:22:34.082710",
    "notifications": [
        {
            "id": "a6a47266-d8e1-4dd8-b1af-c379188d8d47",
            "notification_type": "confirmation",
            "sent_date": "2018-04-23 22:22:33.504160",
            "sent_to": "famousjj@aol.com",
            "sent_via": "email"
        }
    ],
    "offer_opened_date": null,
    "ride_vouchers": [
        {
            "active_from": "2021-04-13 11:43",
            "active_to": "2021-04-15 03:59",
            "allowed_products": {
                "lyft": [
                    "lyft"
                ]
            },
            "canceled_date": null,
            "create_date": "2021-04-13 11:43:01.864710",
            "destination": null,
            "id": "906a8242-f1ac-4a47-b00f-4063bce16e59",
            "number_of_days": 1,
            "number_of_passengers": 1,
            "number_of_vehicles": 1,
            "origin": null,
            "passenger": "contextid-4ABB9BE57D0749A59806586FE573A29BX",
            "premium": false,
            "ride_type": "airport_to_hotel_roundtrip",
            "rides": [
                {
                    "canceled_by": null,
                    "cell_phone_number": "+14805551796",
                    "create_date": "2021-04-13 12:25:00.774289",
                    "currency_code": null,
                    "destination": {
                        "airport_iata_code": null,
                        "hotel_id": "tvl-85370",
                        "latitude": null,
                        "longitude": null,
                        "name": "Hampton Inn"
                    },
                    "id": "a2cf27ed-7046-4fe8-ab39-542824c00fd6",
                    "external_ride_id": "1945326875197128310",
                    "origin": {
                        "airport_iata_code": "LAX",
                        "hotel_id": null,
                        "latitude": null,
                        "longitude": null,
                        "name": "Los Angeles International Airport"
                    },
                    "product": "lyft",
                    "provider": "lyft",
                    "ride_type": "airport_to_hotel",
                    "status": "accepted",
                    "total_amount": null,
                    "price_data": {}
                }
            ],
            "rule": "available_when_no_shuttle",
            "status": "offered",
            "surge_override": false,
            "time_zone": "America/Los_Angeles"
        }
    ],
    "transport_accommodation_status": null
}
```


### HotelVoucherResponseObjectForPassenger
Note: this is similar to [HotelVoucherResponseObjectForAirline](#hotelvoucherresponseobjectforairline), but it contains fewer fields, as some are not appropriate for the Passenger to know about.

### HotelVoucherResponseObjectForAirline

**General hotel information fields:**<br/>
* **hotel_id:** String (length up to 128 characters). this is the unique identifier used to book a hotel, etc (NOTE: this string should be considered case-sensitive, and may contain uppercase and lowercase letters).
  * Here are two examples to express the potential range:
    * `"tvl-83845Xz9ff-Njg1MjFlNTk5YjFmOWY5ZTIxMTQzYjQ.2"`
    * `"ean-Yjg2Y2UyM1M2E0Zjg0ZjZlZGFkNTk3NzkyY2RlY2MyYzYyMDUxMjkwYzc3OGJjODBkODUyYjcxZGEwOWQwYTI5YTg1MDg3N2NDYwZDV.OGQ1NmFjNW-A5NTA1Ndm"`
* **hotel_name:** String. the name of the hotel, suitable for display.
* **image_url:** String. A picture of the hotel. May be `null` if image is unavailable.
    * Note: the aspect ratios of these pictures varies.
* **airport_code** String. The airport this hotel services. Examples: `"LAX"`, `"JFK"`, etc.
* **phone_number:** String. The phone number to call the front desk of the hotel. Also the phone number that can be called to request a hotel shuttle.
* **provider:** String (length up to 5 characters). The hotel inventory provider. Current providers are `"tvl"` (TAConnections), and `"ean"` (Expedia), but expect other providers in the future.
* **room_type:** String (length up to 3 characters). Type of room booked. Current types are `"roh"` (run of house), and `"ssr"` (Special Service Request).
 
**Hotel availability information:**<br/>
* **available:** Integer, total number of rooms available in this hotel. Includes the sum of all soft block rooms and your airline's open hard block rooms.
* **hard_block_count:** Integer, the number of hard block rooms available to your airline (available rooms your airline has already paid for).
* **contract_block_count:** Integer, the number of contract block rooms available to your airline (available rooms your airline has already paid for).

**Hotel services and amenities:**
* **star_rating:** Integer. 1-5. The quality rating of the hotel.
* **premium:** Boolean. The hotel is premium or not.
* **amenities:** Array of [AmenityResponseObject](#amenityresponseobject)s
* **pets_allowed:** Boolean.
* **service_pets_allowed:** Boolean.
* **hotel_on_airport:** Boolean. Hotel is physically attached to or inside of the airport. A passenger could potentially walk to the hotel.
* **hotel_on_airport_instructions:** String (length up to 255 characters). Instructions that tell passengers how to go to the hotel from the airport. NOTE: will be an empty string if the hotel is not attached to an airport (`"hotel_on_airport": false`) or no instructions are present.
* **hotel_allowances:** [HotelAllowanceResponseObject](#hotelallowanceresponseobject)

* **fees:** array of objects with the following fields:
  - **type:** String. Some of the common fees are `"additional_person"`, `"service_pet"`, and `"pet"`. However other types of fees may be listed.
  - **count**: Integer. The number of fees being charged for this type.
  - **rate**: String Decimal. The rate of the fee per count.
  - **total:** String Decimal. The total charge for this fee type and count.

**Hotel pricing fields:**<br/>
* **currency_code:** String. Currency code used for the pricing fields of the hotel. Currently just `"USD"`.
* **tax:** String Decimal.Currency for the amount is specified by the `currency_code`.
* **pets_fee:** String Decimal. Currency for the amount is specified by the `currency_code`.
* **service_pets_fee:** String Decimal. Currency for the amount is specified by the `currency_code`.
* **total_amount** String Decimal. Currency for the amount is specified by the `currency_code`.
* **hotel_allowances_voucher_total:** String (max length 13, two decimal format). Sum of `PassengerHotelAllowances` on voucher.

* **taxes:** Breakdown of `taxes` that makes up the `tax` field.
New `tax.name` values may be added at any time. array of objects with the following fields:
    - **name**: String. The name of the applied tax to voucher.
    - **amount:** String Decimal (2 decimal places). The tax amount for the corresponding tax name.

**Hotel reservation information:**<br/>
* **check_in_date:** Date, YYYY-MM-DD. Check in date in local timezone of port of accommodation. NOTE: If hour is between 12:00 AM and 5:00 AM in local timezone of port of accommodation, check_in_date may be a date in the past.
* **check_out_date:** Date, YYYY-MM-DD. Check out date in local timezone of port of accommodation.
* **hotel_key:** `null` or String, 5 characters. The passenger must present this code to the hotel to unlock the payment for the room. If this is `null`, then `confirmation_id` is filled in.
* **confirmation_id:** `null` or String, up to 20 characters.  A confirmation ID to show to the hotel when checking in that will assist the hotel clerk in finding your hotel reservation. If this is `null`, then `hotel_key` is filled in.
* **room_vouchers:** array of objects with the following fields:
    * **block_type:** String. The type of inventory block used by the voucher. The current possible values are `contract_block`, `hard_block`, and `soft_block`. NOTE: These values may change in the future. NOTE: Vouchers where the `provider` is not `tvl` will always have a value of `soft_block`.
    * **count:** Integer. The number of rooms booked in this block.
    * **hard_block**: Boolean. `true` means `block_type` is either `contract_block` or `hard_block`. `false` means `block_type` is `soft_block`. NOTE: Vouchers where the `provider` is not `tvl` will always have a value of `false`.
    * **rate**: String Decimal. Currency for this amount comes from from the `currency_code` field found a a higher level in the response structure.
* **id:** UUID.
* **hotel_message** `null` or String, up to 128 characters.
A message from the airline to the hotel that contains information about the reservation.
example: Guest is requesting a roll away bed.

**Hotel address fields:**<br/>
* **address1:** String.
* **address2:** String. Can be used to supply additional address lines in some locations. Empty string if not used.
* **address3:** String. Can be used to supply additional address lines in some locations. Empty string if not used.
* **address4:** String. Can be used to supply additional address lines in some locations. Empty string if not used.
* **city:** String.
* **state:** String.
* **post_code:** String.
* **country:** String.

**Hotel positional information:**<br/>
* **latitude:** Float. Units: degrees.
* **longitude:** Float. Units: degrees.
* **distance:** Float. Distance from the airport in miles.
    * Note: this is a direct distance. Actual driving distance will often be more.

**Hotel shuttle information:**<br/>
* **shuttle:** Boolean. Whether or not the hotel offers a free shuttle for passengers passengers to go from the airport to the hotel.
* **shuttle_timing:** String. The shuttle schedule. Can be empty string.
    - Examples:
        * `""` - indicates no shuttle schedule available.
        * `"0:00 23:59"` - indicates 24-hour shuttle. This is common.
        * `"9:30 22:30"` - indicates the shuttle runs from 9:30 AM to 10:30 PM.
        * `"6:00 1:00"` - indicates the shuttle runs from 6:00 AM to 1:00AM the next morning (note the second time can be lower!)
        * `"10:00 23:15"` - indicates the shuttle runs from 10:00 AM to 11:15 PM.
* **shuttle_tracker_url:** String or `null`, max length 500. `null` when unavailable. This is a link to view hotel's shuttle information and a map that tracks the real-time locations of the hotel's shuttles. The linked page may contain other shuttle information as well.
    * Note: You may be able to embed this map in your own website using an `<iframe>` tag. Ask TAConnections for details.
    * Note: The link may expire over time, generally around 24 hours. A fresh link (if still available) can be retrieved via the various `GET` endpoints.
* **shuttle_available_now:** Boolean. Whether or not the shuttle is available at the time of the request. Current time is moved 30 mins forward to account for passenger movement (this results in shuttle not being availabe at 23:30 even if the hotel has shuttle until 0:00).

**Example:**
```json
{
    "address1": "11250 Santa Monica Blvd",
    "address2": "",
    "address3": "",
    "address4": "",
    "airport_code": "LAX",
    "amenities": [
        {
            "name": "Breakfast"
        },
        {
            "name": "Lunch"
        },
        {
            "name": "Dinner"
        },
        {
            "name": "Pool"
        },
        {
            "name": "Fitness Center"
        },
        {
            "name": "Cocktail Hour"
        }
    ],
    "check_in_date": "2018-04-23",
    "check_out_date": "2018-04-24",
    "city": "Los Angeles",
    "confirmation_id": null,
    "country": "USA",
    "currency_code": "USD",
    "distance": 0.10980656141604725,
    "fees": [
        {
            "count": 1,
            "rate": "25.00",
            "total": "25.00",
            "type": "pet"
        },
        {
            "count": 1,
            "rate": "50.00",
            "total": "50.00",
            "type": "service_pet"
        }
    ],
    "hotel_allowances": {
        "amenity": {
           "amount": "50.00"
        },
        "meals": {
            "breakfast": {
                "rate": "10.00"
            },
            "dinner": {
                "rate": "20.00"
            },
            "lunch": {
                "rate": "30.00"
            }
        }
    },
    "hotel_allowances_voucher_total": "0.00",
    "hotel_id": "tvl-83845",
    "hotel_key": "RBX71",
    "hotel_message": "Guest is requesting a roll away bed.",
    "hotel_name": "Holiday Inn Express West Los Angeles",
    "hotel_on_airport": false,
    "hotel_on_airport_instructions": "",
    "id": "cbd5d9b0-641e-4624-b3be-b1daa6a040e3",
    "image_url": "https://i.travelapi.com/hotels/1000000/20000/19200/19104/401faeec_b.jpg",
    "latitude": 34.046576,
    "longitude": -118.447337,
    "passenger_note": "Shuttle at door G3",
    "pets_allowed": false,
    "pets_fee": "25.00",
    "phone_number": "1 310-494-7890",
    "post_code": "90025",
    "premium": true,
    "provider": "tvl",
    "room_type": "roh",
    "room_vouchers": [
        {
            "block_type": "soft_block",
            "count": 1,
            "hard_block": false,
            "rate": "104.00"
        }
    ],
    "rooms_booked": 1,
    "service_pets_allowed": false,
    "service_pets_fee": "50.00",
    "shuttle": true,
    "shuttle_timing": "0:00 23:59",
    "shuttle_tracker_url": "https://trackmyshuttle.com/x/y/12341234abcdefg12345678/9012345",
    "star_rating": 4,
    "state": "CA",
    "tax": "19.05",
    "taxes": [
        {
            "amount": "13.05",
            "name": "CITY"
        },
        {
            "amount": "6.00",
            "name": "LOCAL"
        }
    ],
    "total_amount": "198.05"
}
```


### MealVoucherResponseObject

**Fields:**<br/>

* **currency_code:** Currency for the meal voucher.
* **amount:** String Decimal. meal amount, in whatever `currency_code` was selected.
* **billing_zip_code:** String (max length 20). The billing zip code of the credit card. Can be null.
* **card_number:** String. 16 digits. The credit card number for the meal voucher. Can be null.
* **card_type:** String (max length 20). Can be null.
* **cvc2:** String. 3-4 digits. Can be null.
* **expiration:** String. Format: `"MM/YYYY"` Can be null.
* **id:** UUID.
* **qr_code_url:**  URL to a *.png image file that contains a QR code of the credit card. This is in a format that can be scanned by vendors like HMS Host. Can be null.
    * **Security note:** take care in who you share this URL with!
* **active_from:** the local date and time time that the meal voucher credit card will start working.
* **active_to:** the local date and time time that the meal voucher credit card will stop working. Note that this will end up being ~midnight eastern standard time.
* **time_zone:** the local time zone where the meal voucher is expected to be spent.
* **provider:** String (max length 10). The provider of the meal voucher. Current possible returned values are `"tvl"`, `"icoupon"`, and `"swiipr"`.
* **delay_code** Possible values: (`"passenger_baggage"`, `"cargo_mail"`, `"handling"`, `"technical"`, `"damage"`, `"operations"`, `"weather"`, `"air_traffic"`, `"late_inbound"`, `null`) Can be null.
*  **event_id** String (max length 50). Associates with the swiipr event. Can be null if not available.
*  **url** String (max length 200). Associates with the swiipr event. Link for the passenger to get the benefit. Can be null if not available.

**Example:**<br/>
```json
{
    "active_from": "2018-04-24 09:43",
    "active_to": "2018-04-25 20:59",
    "amount": "12.99",
    "billing_zip_code": "55343",
    "card_number": "0000000001469089",
    "card_type": "MASTERCARD",
    "currency_code": "USD",
    "cvc2": "1234",
    "expiration": "04/2021",
    "id": "a03a8932-afae-4fce-9a69-de485510fcd9",
    "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiUFQ2bzB3Q2hTSHVlT3hvVHlxYmZ1UEFxbW0tZFNFd2dpVmJEdUNYUU1rUSJ9.U5FfVU4wsvvbIKt19m4mUE-E660/meal.png",
    "time_zone": "America/Los_Angeles",
    "provider": "tvl",
    "delay_code": null,
    "event_id": null,
    "url": "https://swiiprsdemo.page.link/asdw"
}
```

### AmenityResponseObject

**Fields:**<br/>
* **name:** String. the name of the amenity provided by the hotel.

**Example:**<br/>
```json
{
  "name": "Fitness Center"
}
```

**NOTE:** These are strings that can change at any time except for the following amenity example:
```json
{
  "name": "Restaurant on Property"
}
```

### PassengerNotificationResponseObject

**Fields:**<br/>
1. **notification_type:** String. either `"offer"`, `"confirmation"`, or `"passenger_pay"`.
2. **sent_via:** String. Either `"email"` or `"text"`.
3. **sent_to:** String. the email address or phone number used for the notification. You know the type of data by examining `sent_via`.
4. **sent_date:** DateTime. time that notification was sent (UTC)
5. **id:** UUID.

**Passenger offer notification via email example:**<br/>
```json
{
    "id": "f1fd3223-0703-4e7c-94cf-d62355b2c16a",
    "notification_type": "offer",
    "sent_date": "2018-03-13 22:26",
    "sent_to": "famousjj@aol.com",
    "sent_via": "email"
}
```

**Passenger offer notification via text example:**<br/>
```json
{
    "id": "f1fd3223-0703-4e7c-94cf-d62355b2c16a",
    "notification_type": "offer",
    "sent_date": "2018-03-13 22:26",
    "sent_to": "+13125551234",
    "sent_via": "text"
}
```

**Passenger confirmation notification via email example:**<br/>
```json
{
    "id": "f1fd3223-0703-4e7c-94cf-d62355b2c16a",
    "notification_type": "confirmation",
    "sent_date": "2018-03-13 22:26",
    "sent_to": "famousjj@aol.com",
    "sent_via": "email"
}
```

**Passenger confirmation notification via text example:**<br/>
```json
{
    "id": "f1fd3223-0703-4e7c-94cf-d62355b2c16a",
    "notification_type": "confirmation",
    "sent_date": "2018-03-13 22:26",
    "sent_to": "+13125551234",
    "sent_via": "text"
}
```

These objects are found in a list on the `notifications` property of passenger objects. <br>
sent_date should be expected to be returned as null initially and updated once the notification is processed. <br>
Note: in the future, notifications could intentionally be delayed for a longer time based on business intelligence and rules for providing the best passenger experience.

### HotelAllowanceResponseObject
NOTE: The currency of the allowances is the same as the `Hotel.currency_code`.<br/><br/>
**Fields:**<br/>
1. **amenity:** HotelAllowanceAmenityObject (object may be null). Each HotelAllowanceAmenityObject has the following fields:<br>
   - **amount:** String (max length 13, two decimal format). The maximum amount the hotel supports for an amenity allowance.
2. **meals:** HotelAllowanceMealsObject (object may be null). Each HotelAllowanceMealsObject has the following fields:<br>
   - **breakfast:** MealsBreakfastObject (object may be null). Each MealsBreakfastObject has the following fields:<br>
      - **rate:** String (max length 13, two decimal format). The allowance rate offered by the hotel.
   - **lunch:** MealsLunchObject (object may be null). Each MealsLunchObject has the following fields:<br>
      - **rate:** String (max length 13, two decimal format). The allowance rate offered by the hotel.
   - **dinner:** MealsDinnerObject (object may be null). Each MealsDinnerObject has the following fields:<br>
      - **rate:** String (max length 13, two decimal format). The allowance rate offered by the hotel.

**Example:**<br/>
```json
{
    "amenity": {
       "amount": "50.00"
    },
    "meals": {
        "breakfast": {
            "rate": "10.00"
        },
        "lunch": {
            "rate": "20.00"
        },
        "dinner": {
            "rate": "30.00"
        }
    }
}
```

### PassengerHotelAllowanceResponseObject
NOTE: The currency of the allowances is the same as the `HotelVoucher.currency_code`.<br/><br/>
**Fields:**<br/>
1. **amenity:** PassengerHotelAllowanceAmenityObject (object may be null). Each PassengerHotelAllowanceAmenityObject has the following fields:<br>
   - **amount:** String (max length 13, two decimal format). The amenity allowance amount for the passenger.
2. **meals:** PassengerHotelAllowanceMealsObject (object may be null). Each PassengerHotelAllowanceMealsObject has the following fields:<br>
   - **breakfast:** MealsBreakfastObject (object may be null). Each MealsBreakfastObject has the following fields:<br>
      - **count:** Integer. The quantity of breakfast allowances.
      - **rate:** String (max length 13, two decimal format)
      - **total:** String (max length 13, two decimal format)
   - **lunch:** MealsLunchObject (object may be null). Each MealsLunchObject has the following fields:<br>
      - **count:** Integer. The quantity of lunch allowances.
      - **rate:** String (max length 13, two decimal format)
      - **total:** String (max length 13, two decimal format)
   - **dinner:** MealsDinnerObject (object may be null). Each MealsDinnerObject has the following fields:<br>
      - **count:** Integer. The quantity of dinner allowances.
      - **rate:** String (max length 13, two decimal format)
      - **total:** String (max length 13, two decimal format)

**Example:**<br/>
```json
{
    "amenity": {
        "amount": "40.00"
    },
    "meals": {
        "breakfast": {
            "count": 1,
            "rate": "10.00",
            "total": "10.00"
        },
        "lunch": {
            "count": 1,
            "rate": "30.00",
            "total": "30.00"
        },
        "dinner": null
    }
}
```

### HotelAllowanceStatusObject
The status of the [PassengerHotelAllowanceResponseObject](#passengerhotelallowanceresponseobject)
broken down by HotelAllowance type.
**Fields:**<br/>
* **breakfast:** The status of the `breakfast` HotelAllowance
* **lunch:** The status of the `lunch` HotelAllowance
* **dinner:** The status of the `dinner` HotelAllowance
* **amenity:** The status of the `amenity` HotelAllowance

**Possible Values:**<br/>
* **not_offered:** This `HotelAllowance` was not included as a part of a `hotel_voucher` offer/issuance.
* **offered:** This `HotelAllowance` was included as part of a `hotel_voucher` offer to the `passenger`.
* **accepted:** This `HotelAllowance` was included as a part of a `hotel_voucher` offer/issuance and is accepted and available at the `hotel`.
* **declined:** This `HotelAllowance` was included as a part of a `hotel_voucher` offer/issuance which was declined by the `passenger` or the airline agent.
* **canceled_offer:** This `HotelAllowance` was included as a part of a `hotel_voucher` offer/issuance which was canceled by the `passenger` or the airline agent.
* **canceled_voucher:** This `HotelAllowance` was included as a part of a `hotel_voucher` offer/issuance which was canceled by the `passenger` or the airline agent.
* **offered_unavailable:** This `HotelAllowance` was included as a part of a `hotel_voucher` offer/issuance but was unavailable at the `hotel`.

**Example:**<br/>
```json
{
    "amenity": "not_offered",
    "breakfast": "offered",
    "lunch": "offered_unavailable",
    "dinner": "accepted"
}
```


### RideVoucherObject
Ride voucher object.

**Fields:**<br/>

**Example:**<br/>
```json
{
    "active_from": "2021-04-13 11:43",
    "active_to": "2021-04-15 03:59",
    "allowed_products": {
        "lyft": [
            "lyft"
        ],
        "uber": [
            "uberx",
            "uber_green"
        ]
    },
    "canceled_date": null,
    "create_date": "2021-04-13 11:43:01.864710",
    "destination": null,
    "id": "906a8242-f1ac-4a47-b00f-4063bce16e59",
    "number_of_days": 1,
    "number_of_passengers": 1,
    "number_of_vehicles": 1,
    "origin": null,
    "passenger": "contextid-4ABB9BE57D0749A59806586FE573A29BX",
    "premium": false,
    "ride_type": "airport_to_hotel_roundtrip",
    "rides": [
        {
            "canceled_by": null,
            "cell_phone_number": "+14805551796",
            "create_date": "2021-04-13 12:25:00.774289",
            "currency_code": null,
            "destination": {
                "airport_iata_code": null,
                "hotel_id": "tvl-85370",
                "latitude": null,
                "longitude": null,
                "name": "Hampton Inn"
            },
            "id": "a2cf27ed-7046-4fe8-ab39-542824c00fd6",
            "external_ride_id": "1945326875197128310",
            "origin": {
                "airport_iata_code": "LAX",
                "hotel_id": null,
                "latitude": null,
                "longitude": null,
                "name": "Los Angeles International Airport"
            },
            "product": "lyft",
            "provider": "lyft",
            "ride_type": "airport_to_hotel",
            "status": "accepted",
            "total_amount": null,
            "price_data": {}
        }
    ],
    "rule": "available_when_no_shuttle",
    "status": "offered",
    "surge_override": false,
    "time_zone": "America/Los_Angeles"
}
```


### TransportVoucherResponseObject
Transport voucher object.

**Fields:**<br/>

* **id:** UUID.
* **provider:** String. The provider of the transport voucher. Current possible returned value is `"tvl"`.
* **currency_code:** Currency for the transport voucher. Currently just `"USD"`.
* **amount:** String Decimal. transport amount, in whatever `currency_code` was selected.
* **card_type:** String (max length 20).
* **card_number:** String. 16 digits. The credit card number for the transport voucher.
* **cvc2:** String. 3-4 digits.
* **billing_zip_code:** String (max length 20). The billing zip code of the credit card.
* **expiration:** String. Format: `"MM/YYYY"`
* **active_from:** the local date and time time that the transport voucher credit card will start working.
* **active_to:** the local date and time time that the transport voucher credit card will stop working. Note that this will end up being ~midnight eastern standard time.
* **time_zone:** the local time zone where the transport voucher is expected to be spent.
* **status:** String. Current possible returned values are `"Accepted", "Offered"`. 
* **qr_code_url:**  URL to a *.png image file that contains a QR code of the credit card. This is in a format that can be scanned by vendors like HMS Host.
  * **Security note:** take care in who you share this URL with!


**Example:**<br/>
```json
{
    "id": "cb8ae997-6d7a-4263-9a74-0020d3e9bf23",
    "provider": "tvl",
    "amount": "10.00",
    "currency_code": "USD",
    "card_type": "MASTERCARD",
    "card_number": "5100000000000537",
    "cvc2": "1234",
    "billing_zip_code": "60173",
    "expiration": "08/2024",
    "active_from": "2021-08-13 07:51",
    "active_to": "2021-08-14 20:59",
    "time_zone": "America/Los_Angeles",
    "status": "Accepted",
    "qr_code_url": "http://passenger.stormx.test/api/v1/offer/transport/qr/eyJ1IjoiRFdxR3AzSWZUdUtMTVA3aHZxU2dnZjEybWpULS1VTE1sWjFNM0ExQ0gyWSJ9.fMRT7ws7SHLc0sTiwgGesoacp5QBndSAdW6YRw00H483w_3cZA5UAmu4p2ys1FJitTAieF4w_tSplCq_hwifSQ/transport.png"
}
```


## Airline API

The Airline API is a collection of endpoints that are meant for integrating your airline's backend systems with TAConnections.
This helps you automate or streamline the process of accommodating distressed passengers.
You can build an Agent experience by calling these endpoints in your airline's backend systems.

### Ping

The `ping` endpoint can be used to test authentication to the StormX API.
Use this to validate your credentials with the Basic Authentication Header.

**Path:**  /api/v1/ping<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Response example:**<br/>
```json
{
    "data": null,
    "error": false,
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/ping",
        "message": "PONG",
        "method": "GET",
        "status": 200
    }
}
```



### Passenger Import


The Passenger Import Process is the main process for importing passengers from the PNR into the StormX system.

Along with providing the appropriate passenger information, you must also select the types of accommodations to provide the distressed passenger:
* hotel accommodation
* meal accommodation
* transport accommodation (a Lyft taxi) to the hotel.

All fields below are required unless marked "(optional)".

All fields below that are "(optional)" and are strings may also be submitted as `null` or an empty string (`""`).

**Payload fields:**
* **context_id:** String. max length 64. A globally unique ID for the passenger. Can contain letters, numbers, underscore (`_`), and hyphens (`-`).
* **pax_record_locator:** String, max length 50. The PNR ID number.
    * Note: All `pax_record_locator`s must be set to the same value in one passenger import request. This means that a maximum of one PNR can be processed per import request.
* **pax_record_locator_group:**  String, max length 50. Used if you wish to split group passengers on the same PNR into different groups to assign different types accommodations, assign to different hotels, etc.
    * Note: by combining `pax_record_locator_group` and `pnr_create_date`, a group of passengers can be uniquely identified.
    * Note: It is okay to submit passengers under different `pax_record_locator_group`s in the same passenger import request.
    * Suggestion: if all passengers share the same accommodations and you do not need to separate them, you may set `pax_record_locator_group` to `pax_record_locator`. This practice is completely optional, but it might simplify your implementation logic. You can also use this if you do not need the `pax_record_locator_group` feature.
* **pnr_create_date:**  Date String, `Y-m-d`. Should be based on UTC. The date that the PNR was created in your system.
    * Note: by combining `pax_record_locator_group` and `pnr_create_date`, a group of passengers can be uniquely identified.
* **flight_number:** String, max length 10. The flight number that was disrupted.
* **scheduled_depart:** DateTime String **(optional)** `Y-m-d H:M` The timezone is UTC.  This is for the newly-scheduled flight. May also be submitted as `null`.
* **requester:** String, max length 100. For auditing purposes. Your airline sends over information to track which employee and/or system submitted the record. Example values: an employee ID number, an employee first and last name, or the name of an automated system.
* **first_name:**  String, max length 50.  Passenger first name.
* **last_name:** String, max length 50.  Passenger last name.
* **phone_numbers:** List(String). **(optional)** For String in List max length 16, in [E.164](https://support.twilio.com/hc/en-us/articles/223183008-Formatting-International-Phone-Numbers) format AND it must always start with a '+'. May be used to send text message notifications to the passenger. Example United States phone number: '+14805551234'.
* **emails:** List(String). **(optional)** For String in List max length 45.  May be used to send notifications to the passenger.
* **pax_status:** String. **(optional)** max length 50. The passenger's frequent flyer status. For example, your airline could send over "Silver", "Platinum Plus", "Gold", or whatever frequent flyer status names your company uses. As directed by your airline, we may use this field to determine if the passenger qualifies to stay at a premium hotel (i.e., a four- or five-star hotel).      
* **ticket_level:** String. **(optional)** Valid choices: (`"first"`, `"business"`, `"premium_economy"`, `"economy"`). As directed by your airline, we may use this field to determine if the passenger qualifies to stay at a premium hotel (i.e., a four- or five-star hotel).
* **port_origin:** String, max length 3. The first port traveled from the beginning of the PNR. Example, if John flies PHX to ORD to MSP, the port origin is `"PHX"`.
* **port_arrival:** String, max length 3. **(optional)** The last port traveled from the beginning of the PNR. Example, if John flies PHX to ORD to MSP, the port arrival is `"MSP"`.
* **port_accommodation:** String, max length 3 - The port where the passenger needs hotel and/or meal accommodations.
* **hotel_accommodation:** Boolean. `true` means a hotel offer will be extended to the passenger.
* **meal_accommodation:** Boolean. `true` means that meal voucher(s) will be extended to the passenger. If `true`, then the `meals` field is required. If `false`, then the `meals` field is ignored.
* **meals:** List of Meal Objects. **(optional)** Note: requires that `meal_accommodation` be `true` to be accepted. Each Meal Object has the following fields:
    - **meal_amount:**  String Decimal, decimal places 2, max digits 8, min value `"0.01"`
    - **currency_code:** String. Valid choices: (`"USD"`, `"CAD"`, `"AUD"`, `"NZD"`, `"GBP"`, `"EUR"`)
    - **number_of_days:** Integer.  The number of days the meal voucher can be used before it expires. If provider = "icoupon", currently must be set to 1.
    - **provider:** String. **(optional)** Valid choices: (`"tvl"`, `"icoupon"`, `"swiipr"`), defaults to `tvl` if not provided.
    - **delay_code** **(optional)** Valid choices: (`"passenger_baggage"`, `"cargo_mail"`, `"handling"`, `"technical"`, `"damage"`, `"operations"`, `"weather"`, `"air_traffic"`, `"late_inbound"`, `null`, `""`) note: can be empty string; this field is required when provider = "swiipr". 
    - **event_id** **(optional)** String, max length 50 - Associates with the swiipr event. When this event is closed using other endpoints, the Swiipr benefits will expire. Generally, this should be unique per flight, and this id should not be reused for other flights.
* **rides:** List of Ride Objects. **(optional)** Each Ride Object has the following fields:
    - **provider** - String. Valid choices: (`"lyft"`, `"uber"`)
    - **allowed_products** - Optional. `null` to allow intelligent defaults based on information about the passengers and the rest of this voucher. Or an object in the following form {"lyft":["lyft"]}), with the following fields:
      * **provider** - String. Valid choices: (`"lyft"`, `"uber"`)
      * **products** - List, optional. If not specified, assumes a reasonable default based on other known information about the group of passengers. Valid choices for uber provider are: uberx, uber_comfort, uberx_black, uber_green, uberxl, uber_black_suv. Valid choices for lyft provider are: lyft, lyft_lux, lyft_lux_black, lyft_plus, lyft_luxsuv.
    - **premium** - Optional. Boolean `null` if null will auto-match
    - **number_of_passengers** - Optional. Integer `null` 1-40. Defaults to number of passengers in the group.
    - **number_of_vehicles** - Optional. Integer `null` 1-5. Automatically calculated if not provided.
    - **number_of_days** - Optional. Integer, 1-14. Defaults to number of nights in the hotel.
    - **rule** - Required. String. Valid choices: (`"available_when_no_shuttle"`, `"no_restrictions"`)
    - **ride_type** - Required. String. Valid choices: (`"airport_to_hotel_roundtrip"`, `"airport_to_hotel"`, `"hotel_to_airport"`)
    - **origin** - Optional. RideWaypoint object or `null`. Specify only one of `airport_iata_code`, `hotel_id`, or `address` + `latitude` + `longitude` rest should be `null` or not present.
        - **airport_iata_code** String  or `null` 
        - **hotel_id** - String or `null`
        - **address** - RideWaypointAddress or `null`
            - **street_address** - String, max length 100
            - **city** - String, max length 100
            - **state_province_name** - String, max length 256
            - **postal_code** - String, max length 15
            - **country_code** - String, max length 3
        - **latitude** - float - required `address` is submitted.
        - **longitude** - float - required `address` is submitted.
    - **destination** - Optional. RideWaypoint object or `null`
    - **surge_override** - Optional. Boolean. defaults to `false`. Whether or not the airline is willing to pay for surge pricing for this voucher.
* **transport_accommodation:** Boolean. Whether the airline will pay for transport from the airport to the hotel.
* **transport_vouchers:** List of Transport Objects. **(optional)** Each Transport Object has the following fields:
    - **amount** - String Decimal, decimal places 2, max digits 8, min value `"0.01"`
    - **currency_code:** String. Valid choices: (`"USD"`,)
    - **number_of_days:** Integer.  The number of days voucher can be used before it expires. min value `1` max value `99`
    - **provider:** String. **(optional)** Valid choices: (`"tvl"`,), defaults to `tvl` if not provided.
    - **airport:** String. **(optional)** Any 3-letter airport IATA code. This will restrict the ride voucher to a geo fence around the specified airport.
* **life_stage:** String. Valid choices: (`"child"`, `"young_adult"`, `"adult"`). Used to determine the passenger's eligibility for hotel rooms. Adults and young adults are eligible for a hotel room while children are required to share a room with an adult. Children cannot receive notifications and cannot book accommodations by themselves.
* **pet:** Boolean. Used by pax app to determine if passenger needs a hotel that allows pets.
* **service_pet:** Boolean. Used by pax app to determine if passenger needs a hotel that allows service pets.
* **handicap:** Boolean. Used by pax app to determine if passenger needs a handicap room in the hotel.
* **notify:** Boolean. If set to `true`, then send an offer notification to the passenger on the available channels (email and/or text message). The available `emails` field will be used to send an email and the available `phone_numbers` will be used to send a text message. If `notify` is set to `true`, then you must provide the `emails` field, the `phone_numbers` field, or both fields.
* **disrupt_depart:** DateTime String `Y-m-d H:M`.   The timezone is UTC. The original time of the flight that was disrupted.
* **number_of_nights:** Integer. The number of nights to stay in the hotel. Currently required to be `1`, maximum `7`.
* **airline_pay:** Boolean. `true` means airline pay. `false` means passenger pay.
* **hotel_allowances:** HotelAllowances Object. **(optional)** This object may be NULL. The Hotel Allowances feature provides a method to add funds to a hotel stay voucher (hotel virtual credit card) for ancillary items such as amenity fees or meals,
which are contractually agreed upon between the hotel and TAConnections on behalf of the airline. They are requested by providing the amount needed by type of meal in the meal object. A general amenity fund can be added as well in the amenity object.
 The amount for the amenity is provided by the airline as shown below.
 Note: requires that `hotel_accommodation` be `true` to be accepted. This field should be populated if an airline
 wishes to use the HotelAllowances feature. If populated the requested allowances will be added to the voucher upon
 booking a hotel that supports the requested allowances for the passenger. Each HotelAllowances Object has the following fields:
    - **meals:**  Meals Object. **(optional)** This object may be NULL. The meal allowances that the passenger needs. The unit represents the quantity of meals (not the dollar amount).
    Note: at least one of the following properties must be set to greater than `0`. Each Meals object has the following fields:
        - **breakfast:** Integer. **(optional)** min value `0`, max value `14`
        - **lunch:** Integer. **(optional)** min value `0`, max value `14`
        - **dinner:** Integer. **(optional)** min value `0`, max value `14`
    - **amenity:** Amenity Object. **(optional)** This object may be NULL. An additional amenity amount to be spent at the hotel. Each Amenity object has the following fields:
        - **amount:**  String Decimal, decimal places 2, max digits 8, min value `"0.01"`
        - **currency_code:** String. Valid choices: (`"USD"`, `"CAD"`, `"AUD"`, `"NZD"`, `"GBP"`, `"EUR"`)
* **disrupt_type:** String, max length 50. This field tracks the reason/cause of the disruption for your Airline's organizational purposes. Suggested values: `"weather"`, `"mechanical"`, etc.
* **system_id:** String (max length 50) **(optional)** Optional field used to distinguish systems that are submitting the passenger information. The information will be available in the Passenger App (passenger API endpoints) although it is not intended for visual display to the passenger.  **Restrictions:** Please submit a value that is NOT prefixed by `tvl-` or `tac-`. TAConnections reserves the right to submit values such as `"tvl-agentapp"`, `"tac-agentapp"` etc. in the future. **Note:** if this field is not filled in, then it will not exist in API responses, full state queries, or queue messages.
* **custom_field_1:** String (max length 50) **(optional)** Optional field for organizational purposes. This field may be useful for tracking additional information for specific airlines needs. The information may be presented to the passenger in the Passenger App and the information is also available in Airline Queue messages. **Note:** if this field is not filled in, then it will not exist in API responses, full state queries, or queue messages.
* **custom_field_2:** String (max length 50) **(optional)** Optional field for organizational purposes. This field may be useful for tracking additional information for specific airlines needs. The information may be presented to the passenger in the Passenger App and the information is also available in Airline Queue messages. **Note:** if this field is not filled in, then it will not exist in API responses, full state queries, or queue messages.
* **custom_field_3:** String (max length 50) **(optional)** Optional field for organizational purposes. This field may be useful for tracking additional information for specific airlines needs. The information may be presented to the passenger in the Passenger App and the information is also available in Airline Queue messages. **Note:** if this field is not filled in, then it will not exist in API responses, full state queries, or queue messages.
* **custom_field_4:** String (max length 50) **(optional)** Optional field for organizational purposes. This field may be useful for tracking additional information for specific airlines needs. The information may be presented to the passenger in the Passenger App and the information is also available in Airline Queue messages. **Note:** if this field is not filled in, then it will not exist in API responses, full state queries, or queue messages.

**Payload example:**

```json
[
    {
        "context_id": "adultCON",
        "pax_record_locator": "prl5",
        "pax_record_locator_group": "prl5",
        "scheduled_depart": "2018-04-24 09:30",
        "pnr_create_date": "2018-04-20",
        "flight_number": "PR 1111",
        "disrupt_type": "mechanical",
        "requester": "AIRLINE USER",
        "first_name": "Jet",
        "last_name": "Jackson",
        "phone_numbers": ["+13125551234"],
        "emails": ["famousjj@aol.com"],
        "pax_status": "custom status",
        "ticket_level": "premium_economy",
        "port_origin": "JFK",
        "port_arrival": "LAX",
        "port_accommodation": "LAX",
        "hotel_accommodation": true,
        "meal_accommodation": true,
        "meals": [
            {
                "currency_code": "USD",
                "meal_amount": 15,
                "number_of_days": 1

            },
            {
                "currency_code": "USD",
                "meal_amount": "12.99",
                "number_of_days": 1,
                "provider": "tvl"
            }
        ],
        "transport_accommodation": true,
        "transport_vouchers": [
            {
                "amount": "10",
                "currency_code": "USD",
                "number_of_days": 1,
                "provider": "tvl",
                "airport": "LAX"
            }
        ],
        "life_stage": "adult",
        "pet": false,
        "service_pet": false,
        "handicap": false,
        "notify": true,
        "disrupt_depart": "2018-04-23 14:30",
        "airline_pay": true,
        "number_of_nights": 1
   },
   {
        "context_id": "childCON",
        "pax_record_locator": "prl5",
        "pax_record_locator_group": "prl5",
        "scheduled_depart": "2018-04-24 09:30",
        "pnr_create_date": "2018-04-20",
        "flight_number": "PR 1111",
        "disrupt_type": "mechanical",
        "requester": "AIRLINE USER",
        "first_name": "JR",
        "last_name": "Jackson",
        "pax_status": "custom status",
        "ticket_level": "premium_economy",
        "port_origin": "JFK",
        "port_arrival": "LAX",
        "port_accommodation": "LAX",
        "hotel_accommodation": true,
        "meal_accommodation": false,
        "transport_accommodation": false,
        "life_stage": "child",
        "pet": false,
        "service_pet": false,
        "handicap": false,
        "notify": false,
        "disrupt_depart": "2018-04-23 14:30",
        "airline_pay": true,
        "number_of_nights": 1,
        "hotel_allowances": {
            "meals": {
                "breakfast": 1,
                "lunch": 0,
                "dinner": 1
            },
            "amenity": {
                "amount": "40.00",
                "currency_code": "USD"
            }
        }
    }
]
```

**Path:** /api/v1/passenger<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>
**Payload:** Passenger Import Example, (Above)<br/>

**Response data fields:**
* **data:** an array of [PassengerResponseObject](#passengerresponseobject)s.
            This object contains all of the fields described in the payload fields above plus a few more that are generated by the StormX system.

**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger",
        "message": "Created",
        "method": "POST",
        "status": 201
    },
    "error": false,
    "data": [
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": "35a40f0f382a4655922b07fd74a1ab00",
            "ak2": "a109910d069645cc9cbef637ad26b652",
            "canceled_date": null,
            "context_id": "adultCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [
                "famousjj@aol.com"
            ],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "Jet",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "not_offered",
                "breakfast": "not_offered",
                "lunch": "not_offered",
                "dinner": "not_offered"
            },
            "last_name": "Jackson",
            "life_stage": "adult",
            "meal_accommodation": true,
            "meal_accommodation_status": "offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [
                {
                    "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                    "notification_type": "offer",
                    "sent_date": null,
                    "sent_to": "+13125551234",
                    "sent_via": "text"
                },
                {
                    "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                    "notification_type": "offer",
                    "sent_date": null,
                    "sent_to": "famousjj@aol.com",
                    "sent_via": "email"
                }
            ],
            "notify": true,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": "https://sxa018.tvlinc.com/offer?ak1=35a40f0f382a4655922b07fd74a1ab00&ak2=a109910d069645cc9cbef637ad26b652",
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [
                "+13125551234"
            ],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "port_arrival": "LAX",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": null
        },
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": null,
            "ak2": null,
            "canceled_date": null,
            "context_id": "childCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "JR",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "offered",
                "breakfast": "offered",
                "lunch": "not_offered",
                "dinner": "offered"
            },
            "last_name": "Jackson",
            "life_stage": "child",
            "meal_accommodation": false,
            "meal_accommodation_status": "not_offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [],
            "notify": false,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": null,
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "port_arrival": "LAX",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": null
        }
    ]
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments                                                                                                                                                                                                                                                                                                                                                  |
| ---- | ---- |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 201 |               | successful                                                                                                                                                                                                                                                                                                                                                |
| 400 | INVALID_JSON  | the data POSTed to the API was not valid JSON data.                                                                                                                                                                                                                                                                                                       |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the POSTed data.                                                                                                                                                                                                                                                                                  |
| 400 | AGENT_INTERVENTION_REQUIRED | You are attempting to automate a scenario that currently requires an agent to process. For example, you cannot set up self-service for a group of unaccompanied children. In certain cases, an airline agent may need to call TAConnections to properly ensure that a passenger's with a certain set of special accommodation requirements needs are met. |
| 400 | PASSENGER_INFO_MISMATCH | When importing multiple passengers at once, certain fields need to match between passengers.                                                                                                                                                                                                                                                              |
| 400 | INVALID_PNR_PASSENGER | Request contains passengers on different PNRs.                                                                                                                                                                                                                                                                                                            |
| 400 | MAX_PASSENGER_IMPORT_EXCEEDED |                                                                                                                                                                                                                                                                                                                                                           |
| 400 | INVALID_ACCOMMODATION |                                                                                                                                                                                                                                                                                                                                                           |
| 400 | MEAL_CANNOT_BE_PROCESSED |                                                                                                                                                                                                                                                                                                                                                           |
| 400 | AMOUNT_TOO_HIGH | meal card amount too high for the currency                                                                                                                                                                                                                                                                                                                |
| 400 | PASSENGER_ALREADY_EXISTS | The passenger's `context_id` has already been imported.                                                                                                                                                                                                                                                                                                   |
| 400 | FEATURE_NOT_SUPPORTED | you may receive this error if you attempt to use a feature that is still being developed.                                                                                                                                                                                                                                                                 |
| 400 | PNR_ALREADY_FINALIZED | PNR already in a finalized state. Passengers cannot be added to the PNR group.                                                                                                                                                                                                                                                                            |
| 400 | PNR_OFFER_EXPIRED | PNR offer is expired. Passengers cannot be added to an expired PNR offer.                                                                                                                                                                                                                                                                                 |
| 400 | INVALID_PORT | 000 is not a valid choice for port_accommodation                                                                                                                                                                                                                                                                                                          |
| 424 | MEAL_VOUCHER_PROVISIONING_FAILED | The downstream meal provider was unable to process the request. The downstream meal provider may be having intermittent issues or is unexpectedly unavailable. Please try the request again momentarily or without meal vouchers.                                                                                                                         |


### Passenger Search

Passenger Search endpoint retrieving passengers by `flight_number`, `disrupt_depart`, and `port_accommodation`.
This endpoint is geared toward returning all passengers on a flight.

**Path:** /api/v1/passenger?flight_number={flight_number}&disrupt_depart={disrupt_depart}&port_accommodation={port_accommodation}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Response example:** <br/>
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger?flight_number=PR%201111&disrupt_depart=2018-04-23%2014:30&port_accommodation=LAX",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": [
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": "35a40f0f382a4655922b07fd74a1ab00",
            "ak2": "a109910d069645cc9cbef637ad26b652",
            "canceled_date": null,
            "context_id": "adultCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [
                "famousjj@aol.com"
            ],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "Jet",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "not_offered",
                "breakfast": "not_offered",
                "lunch": "not_offered",
                "dinner": "not_offered"
            },
            "last_name": "Jackson",
            "life_stage": "adult",
            "meal_accommodation": true,
            "meal_accommodation_status": "offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [
                {
                    "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                    "notification_type": "offer",
                    "sent_date": "2018-04-23 21:57:00.908564",
                    "sent_to": "+13125551234",
                    "sent_via": "text"
                },
                {
                    "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                    "notification_type": "offer",
                    "sent_date": "2018-04-23 21:57:00.921360",
                    "sent_to": "famousjj@aol.com",
                    "sent_via": "email"
                }
            ],
            "notify": true,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": "https://sxa018.tvlinc.com/offer?ak1=35a40f0f382a4655922b07fd74a1ab00&ak2=a109910d069645cc9cbef637ad26b652",
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [
                "+13125551234"
            ],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": null
        },
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": null,
            "ak2": null,
            "canceled_date": null,
            "context_id": "childCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "JR",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "offered",
                "breakfast": "offered",
                "lunch": "not_offered",
                "dinner": "offered"
            },
            "last_name": "Jackson",
            "life_stage": "child",
            "meal_accommodation": false,
            "meal_accommodation_status": "not_offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [],
            "notify": false,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": null,
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": null
        }
    ]
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 404 | FLIGHT_NOT_FOUND |  |




### GET Passenger

Passenger endpoint to get passenger by `context_id`.

**Path:** /api/v1/passenger/{context_id}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Response example:**<br/>

```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger/adultCON",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": {
        "airline_id": 294,
        "airline_pay": true,
        "ak1": "35a40f0f382a4655922b07fd74a1ab00",
        "ak2": "a109910d069645cc9cbef637ad26b652",
        "canceled_date": null,
        "context_id": "adultCON",
        "create_date": "2018-04-23 21:57:00.888451",
        "declined_date": null,
        "disrupt_depart": "2018-04-23 14:30",
        "disrupt_type": "mechanical",
        "emails": [
            "famousjj@aol.com"
        ],
        "expiration_date": "2018-04-24 21:57:00.888451",
        "first_name": "Jet",
        "flight_number": "PR 1111",
        "handicap": false,
        "hotel_accommodation": true,
        "hotel_accommodation_status": "offered",
        "hotel_allowance_status": {
            "amenity": "not_offered",
            "breakfast": "not_offered",
            "lunch": "not_offered",
            "dinner": "not_offered"
        },
        "last_name": "Jackson",
        "life_stage": "adult",
        "meal_accommodation": true,
        "meal_accommodation_status": "offered",
        "modified_date": "2018-04-23 21:57:00.888451",
        "notifications": [
            {
                "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                "notification_type": "offer",
                "sent_date": "2018-04-23 21:57:00.908564",
                "sent_to": "+13125551234",
                "sent_via": "text"
            },
            {
                "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                "notification_type": "offer",
                "sent_date": "2018-04-23 21:57:00.921360",
                "sent_to": "famousjj@aol.com",
                "sent_via": "email"
            }
        ],
        "notify": true,
        "number_of_nights": 1,
        "offer_opened_date": null,
        "offer_url": "https://sxa018.tvlinc.com/offer?ak1=35a40f0f382a4655922b07fd74a1ab00&ak2=a109910d069645cc9cbef637ad26b652",
        "pax_record_locator": "prl5",
        "pax_record_locator_group": "prl5",
        "pax_status": "custom status",
        "pet": false,
        "phone_numbers": [
            "+13125551234"
        ],
        "pnr_create_date": "2018-04-20",
        "port_accommodation": "LAX",
        "port_origin": "JFK",
        "requester": "AIRLINE USER",
        "scheduled_depart": "2018-04-24 09:30",
        "service_pet": false,
        "ticket_level": "premium_economy",
        "transport_accommodation": false,
        "transport_accommodation_status": null,
        "voucher_id": null
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 404 | PASSENGER_NOT_FOUND |  |



### GET Passengers Related

Passenger endpoint used to get other related passenger objects on the same PNR.

**Path:** /api/v1/passenger/{context_id}/related<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger/adultCON/related",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": [
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": null,
            "ak2": null,
            "canceled_date": null,
            "context_id": "childCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "JR",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "offered",
                "breakfast": "offered",
                "lunch": "not_offered",
                "dinner": "offered"
            },
            "last_name": "Jackson",
            "life_stage": "child",
            "meal_accommodation": false,
            "meal_accommodation_status": "not_offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [],
            "notify": false,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": null,
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": null
        }
    ]
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 404 | PASSENGER_NOT_FOUND |  |


### GET PNR Passengers

Endpoint used to get all passengers on a PNR. Either pax_record_locator or pax_record_locator_group is required.

* `"pnr_create_date"` date string format %Y-%m-%d (optional)
* `"pax_record_locator"` string max length 50
* `"pax_record_locator_group"` string max length 50


**Path:** /api/v1/pnr?pax_record_locator_group={pax_record_locator_group}&pnr_create_date={pnr_create_date}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/pnr?pax_record_locator_group=prl5&pnr_create_date=2018-04-20",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": [
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": "35a40f0f382a4655922b07fd74a1ab00",
            "ak2": "a109910d069645cc9cbef637ad26b652",
            "canceled_date": null,
            "context_id": "adultCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [
                "famousjj@aol.com"
            ],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "Jet",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "not_offered",
                "breakfast": "not_offered",
                "lunch": "not_offered",
                "dinner": "not_offered"
            },
            "last_name": "Jackson",
            "life_stage": "adult",
            "meal_accommodation": true,
            "meal_accommodation_status": "offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [
                {
                    "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                    "notification_type": "offer",
                    "sent_date": "2018-04-23 21:57:00.908564",
                    "sent_to": "+13125551234",
                    "sent_via": "text"
                },
                {
                    "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                    "notification_type": "offer",
                    "sent_date": "2018-04-23 21:57:00.921360",
                    "sent_to": "famousjj@aol.com",
                    "sent_via": "email"
                }
            ],
            "notify": true,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": "https://sxa018.tvlinc.com/offer?ak1=35a40f0f382a4655922b07fd74a1ab00&ak2=a109910d069645cc9cbef637ad26b652",
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [
                "+13125551234"
            ],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": null
        },
        {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": null,
            "ak2": null,
            "canceled_date": null,
            "context_id": "childCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "JR",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "offered",
            "hotel_allowance_status": {
                "amenity": "offered",
                "breakfast": "offered",
                "lunch": "not_offered",
                "dinner": "offered"
            },
            "last_name": "Jackson",
            "life_stage": "child",
            "meal_accommodation": false,
            "meal_accommodation_status": "not_offered",
            "modified_date": "2018-04-23 21:57:00.888451",
            "notifications": [],
            "notify": false,
            "number_of_nights": 1,
            "offer_opened_date": null,
            "offer_url": null,
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": null
        }
    ]
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 404 | PNR_NOT_FOUND |  |


### GET Hotels

Hotel endpoint for getting hotel availability.
Intended use by the agent to retrieve hotel availability for a passenger offer.

Query String Parameters

Mandatory:
* port (port of accommodation)
* room_count (number of desired rooms)

Optional:
* provider (string max length 5, current valid choices: (`"tvl"`, `"ean"`), but expect other values in the future).
  * Use `"tvl"` to search for only TAConnections hotel inventory.
  * Use `"ean"` to search only Expedia hotel inventory.
  * When no provider filter is specified, TAConnections inventory is presented. If TAConnections cannot supply sufficient number of rooms for the query, then we will present the inventory of other partners and OTAs inventory if available.
     * **Recommendation:** TAConnections recommends not specifying a specific provider for most use cases.

* hotel_allowances (URL Encoded JSON). Note: requires that at least one of the following fields be set. The JSON object has the following properties:
    - **meals:** Boolean. **(optional)**
    - **amenity:** Boolean. **(optional)**

* URL Encoded JSON Examples:
    - **meals:** hotel_allowances=%7B%22meals%22%3A+true%7D
    - **amenity:** hotel_allowances=%7B%22amenity%22%3A+true%7D
    - **meals and amenity:** hotel_allowances=%7B%22meals%22%3A+true%2C+%22amenity%22%3A+true%7D

* stay_date_start (date string) format: %Y-%m-%d, min: today's date (port time),
max: currently 7 days in the future (port time)
NOTE: This field may be NULL. This field should ONLY be used for future bookings.
This field should be submitted carefully during times between 12 AM and 5 AM local port time.
Hotel inventory for `tvl` and `ean` during this period is considered yesterday's inventory in StormX.
When this field is omitted StormX will calculate the correct `stay_date_start`
as NOW and the booking will be correct, current, and NOT a future booking.  Best practice would be to
exclude the `stay_date_start` for bookings during this period if it is intended for the passenger to check-in now.

(default to false if not provided)
* is_premium (boolean, if true returns hotels with star rating of 4 or 5)
* service_pet (boolean)
* pet (boolean)
* handicap (boolean)

(default to 1 if not provided):
* number_of_nights (integer) minimum value: 1, maximum_value: 7

**Path:** /api/v1/hotels?port={port}&room_count={room_count}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>

**Response data fields:**
* **data:** an array of  [HotelResponseObjectForAirline](#hotelresponseobjectforairline)s



**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/hotels?port=LAX&room_count=1",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": [
        {
            "address1": "11250 Santa Monica Blvd",
            "address2": "",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "available": 10,
            "city": "Los Angeles",
            "country": "USA",
            "currency_code": "USD",
            "distance": 7.587,
            "hard_block_count": 10,
            "contract_block_count": 2,
            "hotel_allowances": {
                "amenity": null,
                "meals": null
            },
            "hotel_id": "tvl-83845",
            "hotel_name": "Holiday Inn Express West Los Angeles",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "image_url": "https://i.travelapi.com/hotels/1000000/20000/19200/19104/401faeec_b.jpg",
            "latitude": 34.046576,
            "longitude": -118.447337,
            "passenger_note": "Shuttle at door G3",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "310-494-7890",
            "post_code": "90025",
            "premium": true,
            "proposed_check_in_date": "2018-05-03",
            "proposed_check_out_date": "2018-05-04",
            "provider": "tvl",
            "rate": "120.00",
            "restaurant_on_property": true,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_available_now": true,
            "star_rating": 4,
            "state": "CA",
            "tax": "10.44"
        },
        {
            "address1": "285 Bay Street",
            "address2": "",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "available": 60,
            "city": "Long Beach",
            "country": "USA",
            "currency_code": "USD",
            "distance": 17.301,
            "hard_block_count": 60,
            "contract_block_count": 10,
            "hotel_allowances": {
                "amenity": {
                    "amount": "50.00"
                },
                "meals": {
                    "breakfast": {
                        "rate": "15.00"
                    },
                    "lunch": {
                        "rate": "20.00"
                    },
                    "dinner": {
                        "rate": "30.00"
                    }
                }
            },
            "hotel_id": "tvl-81519",
            "hotel_name": "Hyatt the Pike Long Beach",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "image_url": "https://i.travelapi.com/hotels/3000000/2550000/2547300/2547269/20e08eec_b.jpg",
            "latitude": 33.764961,
            "longitude": -118.194829,
            "passenger_note": "",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "310-436-1047",
            "post_code": "90802",
            "premium": true,
            "proposed_check_in_date": "2018-05-03",
            "proposed_check_out_date": "2018-05-04",
            "provider": "tvl",
            "rate": "104.00",
            "restaurant_on_property": true,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "star_rating": 4,
            "state": "CA",
            "tax": "9.05"
        }
    ]
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 400 | INVALID_PORT | 000 is not a valid choice for port |
| 404 | PASSENGER_NOT_FOUND |  |
| 200 | MAX_ROOMS_EXCEEDED | `room_count` is larger than the provider supports. |



### Book Hotels

Hotel endpoint used by the agent for accepting a hotel offer and creating a voucher for all `context_ids` passed into the request.
This endpoint is flexible in that it allows passengers to accept offers outside of other passengers on the PNR if desired or need be.

**Path:** /api/v1/hotels<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>


**Payload (fields):**
* **context_ids:** list of context ids accepting the offer
* **hotel_id:** hotel_id from hotel object returned in the get hotel inventory request
* **room_count:** number of rooms allowed to reserve for voucher

(default to `passenger.number_of_nights` from passenger import if not provided):
* number_of_nights (integer) minimum value: 1, maximum_value: 7
* NOTE: if not provided `number_of_nights` must be the same for all
passengers in the provided `context_ids` list.

(default to `null` if not provided):
* hotel_message (string) maximum length: 128.
field serves as a message from the airline to the hotel
containing information about the reservation.

* stay_date_start (date string) format: %Y-%m-%d, min: today's date (port time),
max: currently 7 days in the future (port time)
NOTE: This field may be NULL. This field should ONLY be used for future bookings.
This field should be submitted carefully during times between 12 AM and 5 AM local port time.
Hotel inventory for `tvl` and `ean` during this period is considered yesterday's inventory in StormX.
When this field is omitted StormX will calculate the correct `stay_date_start`
as NOW and the booking will be correct, current, and NOT a future booking.  Best practice would be to
exclude the `stay_date_start` for bookings during this period if it is intended for the passenger to check-in now.

**Payload example:**
```json
{
    "context_ids": ["adultCON","childCON"],
    "hotel_id": "tvl-83845",
    "room_count": 1
}
```


**Response data fields:**<br/>
* **data.hotel_voucher:** [HotelVoucherResponseObjectForAirline](#hotelvoucherresponseobjectforairline).
* **data.voucher_id** ID of voucher.
* **data.modified_date** `modified_date` of the voucher.
* **data.status** `status` of the voucher. The value is limited to only `"finalized"` or `"canceled"`.
* **room_type:** String (length up to 3 characters). Type of room booked. Current types are `"roh"` (run of house), and `"ssr"` (Special Service Request).
* **shuttle_available_now**: Boolean. Whether or not the shuttle is available at the time of the request. Current time is moved 30 mins forward to account for passenger movement (this results in shuttle not being availabe at 23:30 even if the hotel has shuttle until 0:00).
* **ride_vouchers:** an array of [RideVoucherObject](#ridevoucherobject).
* **data.passengers:** an array of [VoucherPassengerResponseObjectForAirline](#voucherpassengerresponseobjectforairline).

**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/hotels",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": {
        "hotel_voucher": {
            "address1": "11250 Santa Monica Blvd",
            "address2": "",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "check_in_date": "2018-04-23",
            "check_out_date": "2018-04-24",
            "city": "Los Angeles",
            "confirmation_id": null,
            "country": "USA",
            "currency_code": "USD",
            "distance": 7.587,
            "fees": [],
            "hotel_allowances": {
                "amenity": null,
                "meals": null
            },
            "hotel_allowances_voucher_total": "0.00",
            "hotel_id": "tvl-83845",
            "hotel_key": "EASSV",
            "hotel_message": null,
            "hotel_name": "Holiday Inn Express West Los Angeles",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "id": "cbd5d9b0-641e-4624-b3be-b1daa6a040e3",
            "image_url": "https://i.travelapi.com/hotels/1000000/20000/19200/19104/401faeec_b.jpg",
            "latitude": 34.046576,
            "longitude": -118.447337,
            "passenger_note": "Shuttle at door G3",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "1 310-494-7890",
            "post_code": "90025",
            "premium": true,
            "provider": "tvl",
            "restaurant_on_property": true,
            "room_vouchers": [
                {
                    "block_type": "hard_block",
                    "count": 1,
                    "hard_block": true,
                    "rate": "120.00"
                }
            ],
            "rooms_booked": 1,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_tracker_url": "https://trackmyshuttle.com/x/y/12341234abcdefg12345678/9012345",
            "star_rating": 4,
            "state": "CA",
            "tax": "10.44",
            "taxes": [
                {
                    "amount": "10.44",
                    "name": "CITY"
                }
            ],
            "total_amount": "130.44"
        },
        "modified_date": "2018-04-23 22:22:33.356294",
        "passengers": [
            {
                "context_id": "adultCON",
                "canceled_date": null,
                "declined_date": null,
                "hotel_accommodation_status": "accepted",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "accepted",
                "meal_vouchers": [
                    {
                        "active_from": "2018-04-23 15:22",
                        "active_to": "2018-04-24 20:59",
                        "amount": "15.00",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469006",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "43c69ff0-1720-4bfe-ace6-3d48135fb249",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiWWtGdkt2ZGRTU1NoNm8zZlU2eUY3dERvUlFGREdFbndzT1BQdWpPUFd3ayJ9.2JDYOj-nNQocumDTGOtcRAFRQsQ/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": null
                    },
                    {
                        "active_from": "2018-04-23 15:22",
                        "active_to": "2018-04-24 20:59",
                        "amount": "12.99",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469014",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "46256844-84e4-40b8-bb47-7c391c4b3ed9",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoialFWTGNPOVBScmlBMWhDM2lQQUFLQmJkV1VqNVhrd2dyTG1aZkJVa3pPZyJ9.kZs1wUGajhqbXAYTx9M03KO2pO8/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": "https://swiiprsdemo.page.link/asdw"
                    }
                ],
                "modified_date": "2018-04-23 22:22:34.082710",
                "notifications": [
                    {
                        "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                        "notification_type": "offer",
                        "sent_date": "2018-04-23 21:57:00.908564",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                        "notification_type": "offer",
                        "sent_date": "2018-04-23 21:57:00.921360",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    },
                    {
                        "id": "3eead061-2816-45c3-8c2f-eacf0ec026cc",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-23 22:22:33.491365",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "id": "a6a47266-d8e1-4dd8-b1af-c379188d8d47",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-23 22:22:33.504160",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    }
                ],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            },
            {
                "context_id": "childCON",
                "canceled_date": null,
                "declined_date": null,
                "hotel_accommodation_status": "accepted",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "not_offered",
                "meal_vouchers": [],
                "modified_date": "2018-04-23 22:22:34.082710",
                "notifications": [],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            }
        ],
        "status": "finalized",
        "voucher_id": "a5917c45-8085-4e1f-9d1f-705d13c9d5b8"
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_JSON  | the data POSTed to the API was not valid JSON data. |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the POSTed data. |
| 400 | INVALID_ACCOMMODATION | Passengers in the booking request are not elgible for hotel accommodations. |
| 400 | PASSENGER_INFO_MISMATCH | When booking multiple passengers at the same time, certain fields need to match between passengers. |
| 400 | PASSENGER_INVALID_STATUS | Passengers in the booking request are not in a status of offered. |
| 400 | FEATURE_NOT_SUPPORTED | you may receive this error if you attempt to use a feature that is still being developed. |
| 400 | PASSENGER_OFFER_EXPIRED | Passenger's offers in the booking request are expired. |
| 400 | INSUFFICIENT_INVENTORY | Hotel provided for booking request does not have enough inventory. |
| 404 | HOTEL_NOT_FOUND | hotel_id provided was not found in the system as a valid hotel. |
| 404 | PASSENGER_NOT_FOUND | Not all Passengers were found for provided context_ids. |
| 200 | MAX_ROOMS_EXCEEDED | `room_count` is larger than the provider supports. |
| 400 | PREVIOUS_DAY_INVENTORY_UNAVAILABLE | The inventory source you are using does not support bookings between times X and Y local time. |
| 404 | HOTEL_OFFER_EXPIRED_OR_NOT_FOUND | The `hotel_id` passed in may be too stale, especially in external booking scenarios. When this happens, refresh inventory and complete the booking within 20 minutes using a `hotel_id` from the fresh inventory. |
| 200 | BOOKING_FAILED | The booking failed. You can attempt another booking. |
| 200 | BOOKING_STATE_UNKNOWN | In the rare event that StormX cannot confirm a booking with a partner succeeded or failed, this error may be raised. A voucher status update will be pushed the the Airline Queue when resolution of state is determined. |
| 424 | MEAL_VOUCHER_PROVISIONING_FAILED | The downstream meal provider was unable to process the request. The downstream meal provider may be having intermittent issues or is unexpectedly unavailable. Please try the request again momentarily or without meal vouchers. |

### Decline Passenger Offer


Endpoint used by agent to decline an offer on behalf of the passenger.

This endpoint may be used if the passenger has a `status` of `"offered"` only, meaning the passenger has not accepted, declined, or canceled an offer yet.

If a passenger has `meal_accommodation` set to `true` and the request is decline hotel only, then the meal vouchers will be returned in the response.

**Path:** /api/v1/passenger/{context_id}/decline?include_pnr={Boolean}&meals={Boolean}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** PUT<br/>

**Query parameters:**<br/>
Both query string parameters are optional. If a query string parameter is not provided it will default to false.
* **include_pnr:** Used to determine if the request needs to decline the offer for all individuals on the same pax_record_locator_group.
                   This allows the flexibility to decline an offer for a single passenger or for an entire group.
                   Set `include_pnr=true` to decline offer for they group or `include_pnr=false` for the individual passenger only.
* **meals:** Used to determine if the request needs to decline meal offers.

**Response data fields:**<br/>
* **data.hotel_voucher:** always `null`, since the hotel has been declined.
* **data.voucher_id** ID of voucher.
* **data.modified_date** `modified_date` of the voucher.
* **data.status** `status` of the voucher. The value is limited to only `"finalized"` or `"canceled"`.
* **data.passengers:** an array of [VoucherPassengerResponseObjectForAirline](#voucherpassengerresponseobjectforairline).

**Example response:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger/adultCON/decline?include_pnr=true&meals=false",
        "message": "UPDATED",
        "method": "PUT",
        "status": 200
    },
    "error": false,
    "data": {
        "hotel_voucher": null,
        "modified_date": "2018-04-24 16:04:00.021612",
        "passengers": [
            {
                "context_id": "adultCON",
                "canceled_date": null,
                "declined_date": "2018-04-24 16:04:00.021612",
                "hotel_accommodation_status": "declined",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "accepted",
                "meal_vouchers": [
                    {
                        "active_from": "2018-04-24 09:04",
                        "active_to": "2018-04-25 20:59",
                        "amount": "15.00",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469055",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "47fd6122-5084-48db-ae42-f7ec88aae579",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiejB1M283R1RUTXFRTU1vRXdCUngzMW0wejRselRreHVpaFJJLXRjUHBTYyJ9.OqmjLmw3oc8zRV57DBjLYLLV1Jc/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": null
                    },
                    {
                        "active_from": "2018-04-24 09:04",
                        "active_to": "2018-04-25 20:59",
                        "amount": "12.99",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469063",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "8ed73bb1-895b-4376-8abb-6081daabca16",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoicGlJcTExaGJSTHVKelp6Y1Q2dm9laDBab0VPa0IwNEJnRnpWaF9jc2diUSJ9.jMAVU7abdvH933Fwp_WUlk-fzpk/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl"
                    }
                ],
                "modified_date": "2018-04-24 16:04:00.018665",
                "notifications": [
                    {
                        "id": "0d5c72a1-4de7-42c5-b4e4-aa0eca84ed92",
                        "notification_type": "offer",
                        "sent_date": "2018-04-24 16:02:02.376943",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "id": "9e023af3-a1a3-4813-a280-44f048ff9690",
                        "notification_type": "offer",
                        "sent_date": "2018-04-24 16:02:02.399928",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    }
                ],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            },
            {
                "context_id": "childCON",
                "canceled_date": null,
                "declined_date": "2018-04-24 16:04:00.021612",
                "hotel_accommodation_status": "declined",
                "hotel_allowance_status": {
                    "amenity": "declined",
                    "breakfast": "declined",
                    "lunch": "not_offered",
                    "dinner": "declined"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "not_offered",
                "meal_vouchers": [],
                "modified_date": "2018-04-24 16:04:00.018665",
                "notifications": [],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            }
        ],
        "status": "finalized",
        "voucher_id": "a5917c45-8085-4e1f-9d1f-705d13c9d5b8"
    }
}
```


**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 400 | PASSENGER_CANNOT_DECLINE | |
| 404 | PASSENGER_NOT_FOUND |  |
| 424 | MEAL_VOUCHER_PROVISIONING_FAILED | The downstream meal provider was unable to process the request. The downstream meal provider may be having intermittent issues or is unexpectedly unavailable. Please try the request again momentarily or without meal vouchers. |



### GET Passenger Offer Full State


Endpoint used to get full state of a passenger offer by `context_id`.
Data returned includes hotel voucher, meal vouchers including credit card information, and full state of passenger object.

**Path:** /api/v1/passenger/{context_id}/state<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>

**Query parameters:**<br/>
* **send_queue_message:** Boolean, optional, default false, used to determine if the request needs to send a queue message to the
airline message queue for the provided context_id.
* **update_modified_date:** Boolean, optional, default false, used to determine if the request needs to update the `passenger.modified_date` resulting in a queue message being sent to the airline message queue for the provided context_id.

**Example response:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger/adultCON/state",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": {
        "passenger": {
            "airline_id": 294,
            "airline_pay": true,
            "ak1": "35a40f0f382a4655922b07fd74a1ab00",
            "ak2": "a109910d069645cc9cbef637ad26b652",
            "canceled_date": null,
            "context_id": "adultCON",
            "create_date": "2018-04-23 21:57:00.888451",
            "declined_date": null,
            "disrupt_depart": "2018-04-23 14:30",
            "disrupt_type": "mechanical",
            "emails": [
                "famousjj@aol.com"
            ],
            "expiration_date": "2018-04-24 21:57:00.888451",
            "first_name": "Jet",
            "flight_number": "PR 1111",
            "handicap": false,
            "hotel_accommodation": true,
            "hotel_accommodation_status": "accepted",
            "hotel_allowance_status": {
                "amenity": "not_offered",
                "breakfast": "not_offered",
                "lunch": "not_offered",
                "dinner": "not_offered"
            },
            "last_name": "Jackson",
            "life_stage": "adult",
            "meal_accommodation": true,
            "meal_accommodation_status": "accepted",
            "modified_date": "2018-04-23 22:22:34.082710",
            "notifications": [
                {
                    "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                    "notification_type": "offer",
                    "sent_date": "2018-04-23 21:57:00.908564",
                    "sent_to": "+13125551234",
                    "sent_via": "text"
                },
                {
                    "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                    "notification_type": "offer",
                    "sent_date": "2018-04-23 21:57:00.921360",
                    "sent_to": "famousjj@aol.com",
                    "sent_via": "email"
                },
                {
                    "id": "3eead061-2816-45c3-8c2f-eacf0ec026cc",
                    "notification_type": "confirmation",
                    "sent_date": "2018-04-23 22:22:33.491365",
                    "sent_to": "+13125551234",
                    "sent_via": "text"
                },
                {
                    "id": "a6a47266-d8e1-4dd8-b1af-c379188d8d47",
                    "notification_type": "confirmation",
                    "sent_date": "2018-04-23 22:22:33.504160",
                    "sent_to": "famousjj@aol.com",
                    "sent_via": "email"
                }
            ],
            "notify": true,
            "number_of_nights": 1,
            "offer_opened_date": "2018-04-23 20:35:09.005656",
            "offer_url": "https://sxa018.tvlinc.com/offer?ak1=35a40f0f382a4655922b07fd74a1ab00&ak2=a109910d069645cc9cbef637ad26b652",
            "pax_record_locator": "prl5",
            "pax_record_locator_group": "prl5",
            "pax_status": "custom status",
            "pet": false,
            "phone_numbers": [
                "+13125551234"
            ],
            "pnr_create_date": "2018-04-20",
            "port_accommodation": "LAX",
            "port_origin": "JFK",
            "requester": "AIRLINE USER",
            "scheduled_depart": "2018-04-24 09:30",
            "service_pet": false,
            "ticket_level": "premium_economy",
            "transport_accommodation": false,
            "transport_accommodation_status": null,
            "voucher_id": "a5917c45-8085-4e1f-9d1f-705d13c9d5b8"
        },
        "voucher": {
            "hotel_voucher": {
                "address1": "11250 Santa Monica Blvd",
                "address2": "",
                "address3": "",
                "address4": "",
                "airport_code": "LAX",
                "check_in_date": "2018-04-23",
                "check_out_date": "2018-04-24",
                "city": "Los Angeles",
                "confirmation_id": null,
                "country": "USA",
                "currency_code": "USD",
                "distance": 7.587,
                "fees": [],
                "hotel_id": "tvl-83845",
                "hotel_allowances": {
                    "amenity": null,
                    "meals": null
                },
                "hotel_allowances_voucher_total": "0.00",
                "hotel_key": "EASSV",
                "hotel_message": null,
                "hotel_name": "Holiday Inn Express West Los Angeles",
                "hotel_on_airport": false,
                "hotel_on_airport_instructions": "",
                "id": "cbd5d9b0-641e-4624-b3be-b1daa6a040e3",
                "image_url": null,
                "latitude": 34.046576,
                "longitude": -118.447337,
                "passenger_note": "",
                "pets_allowed": false,
                "pets_fee": "0.00",
                "phone_number": "1 310-494-7890",
                "post_code": "90025",
                "premium": true,
                "provider": "tvl",
                "restaurant_on_property": true,
                "room_vouchers": [
                    {
                        "block_type": "contract_block",
                        "count": 1,
                        "hard_block": true,
                        "rate": "120.00"
                    }
                ],
                "rooms_booked": 1,
                "service_pets_allowed": true,
                "service_pets_fee": "0.00",
                "shuttle": true,
                "shuttle_timing": "0:00 23:59",
                "shuttle_tracker_url": "https://trackmyshuttle.com/x/y/12341234abcdefg12345678/9012345",
                "star_rating": 4,
                "state": "CA",
                "tax": "10.44",
                "taxes": [
                    {
                        "amount": "10.44",
                        "name": "CITY"
                    }
                ],
                "total_amount": "130.44"
            },
            "hotel_allowances_voucher": {
                "amenity": null,
                "meals": null
            },
            "meal_vouchers": [
                {
                    "active_from": "2018-04-23 15:22",
                    "active_to": "2018-04-24 20:59",
                    "amount": "15.00",
                    "billing_zip_code": "55343",
                    "card_number": "0000000001469006",
                    "card_type": "MASTERCARD",
                    "currency_code": "USD",
                    "cvc2": "1234",
                    "expiration": "04/2021",
                    "id": "43c69ff0-1720-4bfe-ace6-3d48135fb249",
                    "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiWWtGdkt2ZGRTU1NoNm8zZlU2eUY3dERvUlFGREdFbndzT1BQdWpPUFd3ayJ9.2JDYOj-nNQocumDTGOtcRAFRQsQ/meal.png",
                    "time_zone": "America/Los_Angeles",
                    "provider": "tvl",
                    "url": null
                },
                {
                    "active_from": "2018-04-23 15:22",
                    "active_to": "2018-04-24 20:59",
                    "amount": "12.99",
                    "billing_zip_code": "55343",
                    "card_number": "0000000001469014",
                    "card_type": "MASTERCARD",
                    "currency_code": "USD",
                    "cvc2": "1234",
                    "expiration": "04/2021",
                    "id": "46256844-84e4-40b8-bb47-7c391c4b3ed9",
                    "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoialFWTGNPOVBScmlBMWhDM2lQQUFLQmJkV1VqNVhrd2dyTG1aZkJVa3pPZyJ9.kZs1wUGajhqbXAYTx9M03KO2pO8/meal.png",
                    "time_zone": "America/Los_Angeles",
                    "provider": "tvl"
                }
            ],
            "modified_date": "2018-04-23 22:22:33.356294",
            "ride_vouchers": [
                {
                    "active_from": "2021-04-13 11:43",
                    "active_to": "2021-04-15 03:59",
                    "allowed_products": {
                        "lyft": [
                            "lyft"
                        ],
                        "uber": [
                            "uberx",
                            "uber_green"
                        ]
                    },
                    "canceled_date": null,
                    "create_date": "2021-04-13 11:43:01.864710",
                    "destination": null,
                    "id": "906a8242-f1ac-4a47-b00f-4063bce16e59",
                    "number_of_days": 1,
                    "number_of_passengers": 1,
                    "number_of_vehicles": 1,
                    "origin": null,
                    "passenger": "contextid-4ABB9BE57D0749A59806586FE573A29BX",
                    "premium": false,
                    "ride_type": "airport_to_hotel_roundtrip",
                    "rides": [
                        {
                            "canceled_by": null,
                            "cell_phone_number": "+14805551796",
                            "create_date": "2021-04-13 12:25:00.774289",
                            "currency_code": null,
                            "destination": {
                                "airport_iata_code": null,
                                "hotel_id": "tvl-85370",
                                "latitude": null,
                                "longitude": null,
                                "name": "Hampton Inn"
                            },
                            "id": "a2cf27ed-7046-4fe8-ab39-542824c00fd6",
                            "external_ride_id": "1945326875197128310",
                            "origin": {
                                "airport_iata_code": "LAX",
                                "hotel_id": null,
                                "latitude": null,
                                "longitude": null,
                                "name": "Los Angeles International Airport"
                            },
                            "product": "lyft",
                            "provider": "lyft",
                            "ride_type": "airport_to_hotel",
                            "status": "accepted",
                            "total_amount": null,
                            "price_data": {}
                        }
                    ],
                    "rule": "available_when_no_shuttle",
                    "status": "offered",
                    "surge_override": false,
                    "time_zone": "America/Los_Angeles"
                }
            ],
            "status": "finalized"
        }
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | FEATURE_NOT_SUPPORTED |  |
| 404 | PASSENGER_NOT_FOUND |  |


### GET Voucher

Endpoint used to get voucher object by `voucher_id`.
Data returned includes hotel voucher, passengers and their meal/transport vouchers including credit card information.

**Path:** /api/v1/voucher/{voucher_id}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>


**Response data fields:**<br/>
* **data.hotel_voucher:** [HotelVoucherResponseObjectForAirline](#hotelvoucherresponseobjectforairline).
* **data.voucher_id** ID of voucher.
* **data.modified_date** `modified_date` of the voucher.
* **data.status** `status` of the voucher. The value is limited to only `"finalized"` or `"canceled"`.
* **data.passengers:** an array of [VoucherPassengerResponseObjectForAirline](#voucherpassengerresponseobjectforairline).
* **data.ride_vouchers:** an array of [RideVoucherObject](#ridevoucherobject).
* **data.transport_vouchers:** an array of [TransportVoucherResponseObject](#transportvoucher). #TODO


**Example response:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/voucher/a5917c45-8085-4e1f-9d1f-705d13c9d5b8",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": {
        "hotel_voucher": {
            "address1": "11250 Santa Monica Blvd",
            "address2": "",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {   
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },                
                {
                    "name": "Restaurant on Property"
                }
            ],
            "check_in_date": "2018-04-23",
            "check_out_date": "2018-04-24",
            "city": "Los Angeles",
            "confirmation_id": null,
            "country": "USA",
            "currency_code": "USD",
            "distance": 7.587,
            "fees": [],
            "hotel_allowances": {
                "amenity": {
                   "amount": "50.00"
                },
                "meals": {
                    "breakfast": {
                        "rate": "10.00"
                    },
                    "dinner": {
                        "rate": "20.00"
                    },
                    "lunch": {
                        "rate": "30.00"
                    }
                }
            },
            "hotel_allowances_voucher_total": "80.00",
            "hotel_id": "tvl-83845",
            "hotel_key": "EASSV",
            "hotel_message": null,
            "hotel_name": "Holiday Inn Express West Los Angeles",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "id": "cbd5d9b0-641e-4624-b3be-b1daa6a040e3",
            "image_url": "https://i.travelapi.com/hotels/1000000/20000/19200/19104/401faeec_b.jpg",
            "latitude": 34.046576,
            "longitude": -118.447337,
            "passenger_note": "",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "1 310-494-7890",
            "post_code": "90025",
            "premium": true,
            "provider": "tvl",
            "restaurant_on_property": true,
            "room_vouchers": [
                {
                    "block_type": "contract_block",
                    "count": 1,
                    "hard_block": true,
                    "rate": "120.00"
                }
            ],
            "rooms_booked": 1,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_tracker_url": "https://trackmyshuttle.com/x/y/12341234abcdefg12345678/9012345",
            "star_rating": 4,
            "state": "CA",
            "tax": "10.44",
            "taxes": [
                {
                    "amount": "10.44",
                    "name": "CITY"
                }
            ],
            "total_amount": "210.44"
        },
        "modified_date": "2018-04-23 22:22:33.356294",
        "passengers": [
            {
                "context_id": "adultCON",
                "canceled_date": null,
                "declined_date": null,
                "hotel_accommodation_status": "accepted",
                "hotel_allowance_status": {
                    "amenity": "accepted",
                    "breakfast": "accepted",
                    "lunch": "not_offered",
                    "dinner": "accepted"
                },
                "hotel_allowances_voucher": {
                    "amenity": {
                        "amount": "40.00"
                    },
                    "meals": {
                        "breakfast": {
                            "count": 1,
                            "rate": "10.00",
                            "total": "10.00"
                        },
                        "dinner": {
                            "count": 1,
                            "rate": "30.00",
                            "total": "30.00"
                        },
                        "lunch": null
                    }
                },
                "meal_accommodation_status": "accepted",
                "meal_vouchers": [
                    {
                        "active_from": "2018-04-23 15:22",
                        "active_to": "2018-04-24 20:59",
                        "amount": "15.00",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469006",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "43c69ff0-1720-4bfe-ace6-3d48135fb249",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiWWtGdkt2ZGRTU1NoNm8zZlU2eUY3dERvUlFGREdFbndzT1BQdWpPUFd3ayJ9.2JDYOj-nNQocumDTGOtcRAFRQsQ/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": null
                    },
                    {
                        "active_from": "2018-04-23 15:22",
                        "active_to": "2018-04-24 20:59",
                        "amount": "12.99",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469014",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "46256844-84e4-40b8-bb47-7c391c4b3ed9",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoialFWTGNPOVBScmlBMWhDM2lQQUFLQmJkV1VqNVhrd2dyTG1aZkJVa3pPZyJ9.kZs1wUGajhqbXAYTx9M03KO2pO8/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl"
                    }
                ],
                "modified_date": "2018-04-23 22:22:34.082710",
                "notifications": [
                    {
                        "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                        "notification_type": "offer",
                        "sent_date": "2018-04-23 21:57:00.908564",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                        "notification_type": "offer",
                        "sent_date": "2018-04-23 21:57:00.921360",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    },
                    {
                        "id": "3eead061-2816-45c3-8c2f-eacf0ec026cc",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-23 22:22:33.491365",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "id": "a6a47266-d8e1-4dd8-b1af-c379188d8d47",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-23 22:22:33.504160",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    }
                ],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            },
            {
                "context_id": "childCON",
                "canceled_date": null,
                "declined_date": null,
                "hotel_accommodation_status": "accepted",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "not_offered",
                "meal_vouchers": [],
                "modified_date": "2018-04-23 22:22:34.082710",
                "notifications": [],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            }
        ],
        "ride_vouchers": [
            {
                "active_from": "2021-04-13 12:39",
                "active_to": "2021-04-15 03:59",
                "allowed_products": [{"products": ["lyft"],
                                      "provider": "lyft"}],
                "canceled_date": null,
                "create_date": "2021-04-13 12:39:53.005549",
                "destination": null,
                "id": "4b91d67b-db0a-4d16-aa88-bab18b026886",
                "number_of_days": 1,
                "number_of_passengers": 1,
                "number_of_vehicles": 1,
                "origin": null,
                "passenger": "contextid-854A2838991C4600BBA290056AD7B156X",
                "premium": false,
                "ride_type": "airport_to_hotel_roundtrip",
                "rides": [
                    {
                       "canceled_by": null,
                       "cell_phone_number": "+14805559750",
                       "create_date": "2021-04-13 12:40:00.918773",
                       "currency_code": null,
                       "destination": {
                           "airport_iata_code": null,
                           "hotel_id": "tvl-85370",
                           "latitude": null,
                           "longitude": null,
                           "name": "Hampton Inn NYC - JFK"
                       },
                       "id": "063e5fa5-2892-41e7-a9cb-f1064ccebac9",
                       "external_ride_id": "1945326875197128310",
                       "origin": {
                           "airport_iata_code": "JFK",
                           "hotel_id": null,
                           "latitude": null,
                           "longitude": null,
                           "name": "John F. Kennedy International Airport"
                       },
                       "product": "lyft",
                       "provider": "lyft",
                       "ride_type": "airport_to_hotel",
                       "status": "accepted",
                       "total_amount": null, 
                       "price_data": {}
                    }
                ],
                "rule": "available_when_no_shuttle",
                "status": "accepted",
                "surge_override": false,
                "time_zone": "America/New_York"
            }
        ],
        "transport_vouchers": [
            {
                "id": "cb8ae997-6d7a-4263-9a74-0020d3e9bf23",
                "provider": "tvl",
                "amount": "10.00",
                "currency_code": "USD",
                "card_type": "MASTERCARD",
                "card_number": "5100000000000537",
                "cvc2": "1234",
                "billing_zip_code": "60173",
                "expiration": "08/2024",
                "active_from": "2021-08-13 07:51",
                "active_to": "2021-08-14 20:59",
                "time_zone": "America/Los_Angeles",
                "status": "Accepted",
                "qr_code_url": "http://passenger.stormx.test/api/v1/offer/transport/qr/eyJ1IjoiRFdxR3AzSWZUdUtMTVA3aHZxU2dnZjEybWpULS1VTE1sWjFNM0ExQ0gyWSJ9.fMRT7ws7SHLc0sTiwgGesoacp5QBndSAdW6YRw00H483w_3cZA5UAmu4p2ys1FJitTAieF4w_tSplCq_hwifSQ/transport.png"
            }
        ],
        "status": "finalized",
        "voucher_id": "a5917c45-8085-4e1f-9d1f-705d13c9d5b8"
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 404 | VOUCHER_NOT_FOUND |  |

### Passenger Cancel

This endpoint may be used to cancel an offer or voucher.
<br><br>
Certain conditions must be met for the cancellation to be successful:<br>
* the offer/voucher is not already canceled.
* the hotel credit card is still locked (has not been unlocked by the hotel).

**Path:** /api/v1/passenger/{context_id}/cancel<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** PUT<br/>

**Response data fields:**<br/>
* **data.hotel_voucher:** [HotelVoucherResponseObjectForAirline](#hotelvoucherresponseobjectforairline).
* **data.voucher_id** ID of voucher.
* **data.modified_date** `modified_date` of the voucher.
* **data.status** `status` of the voucher. The value is limited to only `"finalized"` or `"canceled"`.
* **data.passengers:** an array of [VoucherPassengerResponseObjectForAirline](#voucherpassengerresponseobjectforairline).


**Example response:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger/adultCON/cancel",
        "message": "UPDATED",
        "method": "PUT",
        "status": 200
    },
    "error": false,
    "data": {
        "hotel_voucher": {
            "address1": "11250 Santa Monica Blvd",
            "address2": "",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "check_in_date": "2018-04-23",
            "check_out_date": "2018-04-24",
            "city": "Los Angeles",
            "confirmation_id": null,
            "country": "USA",
            "currency_code": "USD",
            "distance": 7.587,
            "fees": [],
            "hotel_allowances": {
                "amenity": null,
                "meals": null
            },
            "hotel_allowances_voucher_total": "0.00",
            "hotel_id": "tvl-83845",
            "hotel_key": "EASSV",
            "hotel_message": null,
            "hotel_name": "Holiday Inn Express West Los Angeles",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "id": "cbd5d9b0-641e-4624-b3be-b1daa6a040e3",
            "image_url": "https://i.travelapi.com/hotels/1000000/20000/19200/19104/401faeec_b.jpg",
            "latitude": 34.046576,
            "longitude": -118.447337,
            "passenger_note": "",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "1 310-494-7890",
            "post_code": "90025",
            "premium": true,
            "provider": "tvl",
            "restaurant_on_property": true,
            "room_vouchers": [
                {
                    "block_type": "soft_block",
                    "count": 1,
                    "hard_block": false,
                    "rate": "120.00"
                }
            ],
            "rooms_booked": 1,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_tracker_url": "https://trackmyshuttle.com/x/y/12341234abcdefg12345678/9012345",
            "star_rating": 4,
            "state": "CA",
            "tax": "10.44",
            "taxes": [
                {
                    "amount": "10.44",
                    "name": "CITY"
                }
            ],
            "total_amount": "130.44"
        },
        "modified_date": "2018-04-24 15:57:44.047891",
        "passengers": [
            {
                "context_id": "adultCON",
                "canceled_date": "2018-04-24 15:57:44.047891",
                "declined_date": null,
                "hotel_accommodation_status": "canceled_voucher",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "accepted",
                "meal_vouchers": [
                    {
                        "active_from": "2018-04-23 15:22",
                        "active_to": "2018-04-24 20:59",
                        "amount": "15.00",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469006",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "43c69ff0-1720-4bfe-ace6-3d48135fb249",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiWWtGdkt2ZGRTU1NoNm8zZlU2eUY3dERvUlFGREdFbndzT1BQdWpPUFd3ayJ9.2JDYOj-nNQocumDTGOtcRAFRQsQ/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": null
                    },
                    {
                        "active_from": "2018-04-23 15:22",
                        "active_to": "2018-04-24 20:59",
                        "amount": "12.99",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001469014",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "id": "46256844-84e4-40b8-bb47-7c391c4b3ed9",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoialFWTGNPOVBScmlBMWhDM2lQQUFLQmJkV1VqNVhrd2dyTG1aZkJVa3pPZyJ9.kZs1wUGajhqbXAYTx9M03KO2pO8/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl"
                    }
                ],
                "modified_date": "2018-04-24 15:57:44.047891",
                "notifications": [
                    {
                        "id": "e8458f0b-c648-427c-ac75-ca1f1fbfaf13",
                        "notification_type": "offer",
                        "sent_date": "2018-04-23 21:57:00.908564",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "id": "1aa2499f-7e15-4571-8030-66aaae21af7d",
                        "notification_type": "offer",
                        "sent_date": "2018-04-23 21:57:00.921360",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    },
                    {
                        "id": "3eead061-2816-45c3-8c2f-eacf0ec026cc",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-23 22:22:33.491365",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "id": "a6a47266-d8e1-4dd8-b1af-c379188d8d47",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-23 22:22:33.504160",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    },
                    {
                        "id": "b809bba3-c29a-415a-b959-86c20095c6ca",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-24 15:51:15.409467",
                        "sent_to": "famousjjBro@aol.com",
                        "sent_via": "email"
                    },
                    {
                        "id": "6a5d2bb2-f4ca-4f83-a57b-0d4fefd464c5",
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-24 15:52:31.825145",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    }
                ],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            },
            {
                "context_id": "childCON",
                "canceled_date": "2018-04-24 15:57:44.047891",
                "declined_date": null,
                "hotel_accommodation_status": "canceled_voucher",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "not_offered",
                "meal_vouchers": [],
                "modified_date": "2018-04-24 15:57:44.047891",
                "notifications": [],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            }
        ],
        "ride_vouchers": [],
        "status": "canceled",
        "voucher_id": "a5917c45-8085-4e1f-9d1f-705d13c9d5b8"
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 400 | PASSENGER_CANNOT_CANCEL | |
| 404 | PASSENGER_NOT_FOUND |  |

### Passenger Notifications

Endpoint used to send this passenger's offer/confirmation details to an email or phone number
depending on the status of the passenger. Send up either text or email but not both.
Note: This endpoint may be used if the passenger has a `hotel_accommodation_status` of `"canceled_voucher"` or `"declined"` only if the passenger has a `"meal_accommodation_status"` of `"accepted"`.
See PassengerNotificationResponseObject for detailed description of response object.

**Path:** /api/v1/passenger/{context_id}/notifications<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>

**Payload fields:**<br/>
**email:** string max_length=45<br>
**text:** string max_length=16

**Email notification payload example:**
```json
{
    "email": "famousjjBro@aol.com"
}
```

**Email notification response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger/adultCON/notifications",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": {
        "notification": {
            "id": "2871468b-09d5-4f72-b4bc-4ba6b60754b9",
            "notification_type": "offer",
            "sent_date": null,
            "sent_to": "famousjjBro@aol.com",
            "sent_via": "email"
        },
        "passenger": {
            "context_id": "adultCON",
            "modified_date": "2018-05-01 18:25:48.296134"
        }
    }
}
```

**Text notification payload example:**
```json
{
    "text": "+13125551234"
}
```

**Text notification response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/passenger/adultCON/notifications",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": {
        "notification": {
            "id": "a6afc2f4-44ab-4649-8b3c-4d8c34266456",
            "notification_type": "offer",
            "sent_date": null,
            "sent_to": "+13125551234",
            "sent_via": "text"
        },
        "passenger": {
            "context_id": "adultCON",
            "modified_date": "2018-05-01 18:26:37.361989"
        }
    }
}
```


**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 400 | CANNOT_NOTIFY | |
| 404 | PASSENGER_NOT_FOUND |  |


### Add Rides to Passenger

Endpoint used to add a ride voucher (uber/lyft) to the passenger.


**Path:** /api/v1/passenger/{context_id}/rides<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>


This shows one Ride Object, you need to send a list of them:

* **provider** - String. Valid choices: (`"lyft"`, `"uber"`)
* **allowed_products** - Optional. `null` to allow intelligent defaults based on information about the passengers and the rest of this voucher. Or an object in the following form {"lyft":["lyft"]}), with the following fields:
  * **provider** - String. Valid choices: (`"lyft"`, `"uber"`)
  * **products** - String, optional. If not specified, assumes a reasonable default based on other known information about the group of passengers. Valid choices for uber provider are: uberx, uber_comfort, uberx_black, uber_green, uberxl, uber_black_suv. Valid choices for lyft provider are: lyft, lyft_lux, lyft_lux_black, lyft_plus, lyft_luxsuv.
* **premium** - Optional. Boolean `null` if null will auto-match
* **number_of_passengers** - Optional. Integer `null` 1-40. Defaults to number of passengers in the group.
* **number_of_vehicles** - Optional. Integer `null` 1-5. Automatically calculated if not provided.
* **number_of_days** - Optional. Integer, 1-14. Defaults to number of nights in the hotel.
* **rule** - Required. String. Valid choices: (`"available_when_no_shuttle"`, `"no_restrictions"`)
* **ride_type** - Required. String. Valid choices: (`"airport_to_hotel_roundtrip"`, `"airport_to_hotel"`, `"hotel_to_airport"`)
* **origin** - Optional. RideWaypoint object or `null`. Specify only one of `airport_iata_code`, `hotel_id`, or `address` + `latitude` + `longitude`.
    - **airport_iata_code** String  or `null`
    - **hotel_id** - String or `null`
    - **address** - RideWaypointAddress or `null`
        - **street_address** - String, max length 100
        - **city** - String, max length 100
        - **state_province_name** - String, max length 256
        - **postal_code** - String, max length 15
        - **country_code** - String, max length 3
    - **latitude** - float - required `address` is submitted.
    - **longitude** - float - required `address` is submitted.
* **destination** - Optional. RideWaypoint object or `null`
* **surge_override** - Optional. Boolean. defaults to `false`. Whether the airline is willing to pay for surge pricing for this voucher.


**Payload example:**

```json
[
    {
        "provider": "lyft",
        "allowed_products": null,
        "premium": null,
        "number_of_passengers": null,
        "number_of_vehicles": null,
        "number_of_days": 1,
        "rule": "available_when_no_shuttle",
        "ride_type": "airport_to_hotel_roundtrip",
        "origin": null,
        "destination": null,
        "surge_override": false
    }
]
```

**Response** 

Same as `GET Passenger /api/v1/passenger/{context_id}


**Path:** /api/v1/passenger/{context_id}/rides<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>


**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 201 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 404 | PASSENGER_NOT_FOUND |  |


### Get Rideshare vehicle types

Endpoint used to get information about rideshare products and their capacity.

**Path:** /api/v1/transport/vehicle-types<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>

**Response data fields:**<br>
**provider:** string, either lyft or uber.<br/>
**code_name:** string, the internal name of the rideshare product.<br/>
**display_name:** string, formatted name of rideshare product suitable for UI presentation.<br/>
**allow_single_passenger:** Boolean, can this product be ordered for a single passenger.<br/>
**group_minimum:** integer, minimum number of passengers for the related rideshare product.<br/>
**group_maximum:** integer, maximum number of passengers for the related rideshare product.<br/>
**note:** string, optional string for additional info on product.<br/>

**Response Example:** 
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": null,
        "href": "http://api.stormx.test/api/v1/transport/lyft/vehicle_types",
        "message": "OK",
        "method": "GET",
        "status": "200"
    },
    "error": false,
    "data": [
        {
            "provider": "lyft",
            "allow_single_passenger": true,
            "code_name": "lyft",
            "display_name": "Lyft",
            "group_maximum": 3,
            "group_minimum": 2,
            "note": ""
        },
        {
            "provider": "lyft",
            "allow_single_passenger": true,
            "code_name": "lyft_premier",
            "display_name": "Lux",
            "group_maximum": 3,
            "group_minimum": 2,
            "note": ""
        },
        {
            "provider": "lyft",
            "allow_single_passenger": true,
            "code_name": "lyft_lux",
            "display_name": "Lux Black",
            "group_maximum": 3,
            "group_minimum": 2,
            "note": ""
        },
        {
            "provider": "lyft",
            "allow_single_passenger": false,
            "code_name": "lyft_plus",
            "display_name": "Lyft",
            "group_maximum": 5,
            "group_minimum": 3,
            "note": ""
        },
        {
            "provider": "lyft",
            "allow_single_passenger": false,
            "code_name": "lyft_luxsuv",
            "display_name": "Lux Black XL",
            "group_maximum": 5,
            "group_minimum": 3,
            "note": ""
        }
    ]
}
```


**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |



### Get Rideshare pickup estimates

Endpoint used to get ride pickup estimates 

**Path:** /api/v1/offer/tvl/ride/pickup-estimates?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** GET<br/>

**Request query fields:**<br>
**provider:** string, either lyft or uber.<br/>
**direction:** string, in which direction does the ride goes. One of: airport_to_hotel, hotel_to_airport, airport_to_airport. Used with `ride_voucher_id`. Not required.<br/>
**pickup_latitude:** float. Not required.<br/>
**pickup_longitude:** float. Not required.<br/>
**dropoff_latitude:** float. Not required.<br/>
**dropoff_longitude:** float. Not required.<br/>
**ride_voucher_id:** uuid. Not required.<br/>

You should either provide all pickup/dropoff points or provide `ride_voucher_id` field. If `ride_voucher_id` is not provided and there
are multiple ride vouchers, then first fit for the `direction` will be used. If no fit is found, first ride voucher will be used. 
If no `direction` is provided then `airport_to_hotel` is assumed.

**Response data fields:**<br>
**products:** list of product objects<br/>
**origin:** object, fields: type,name,latitude,longitude<br/>
**destination:** fields: type,name,latitude,longitude<br/>

**Response Example:** 
```json
{
    "data": {
        "destination": {
            "latitude": 33.7949914,
            "longitude": -118.3375419,
            "name": "Ramada Inn - Torrance",
            "type": "hotel"
        },
        "origin": {
            "latitude": 33.9415889,
            "longitude": -118.40853,
            "name": "LAX",
            "type": "airport"
        },
        "products": [
            {
                "display_name": "UberX",
                "estimated_distance_miles": 3.02,
                "estimated_pickup_seconds": 8,
                "estimated_trip_duration_seconds": 898,
                "image_url": null,
                "product_id": "893cb2a1-c412-4fee-8947-dd2accbc56a1",
                "product_type": "uberx"
            },
            {
                "display_name": "Uberx - Espa\u00f1ol",
                "estimated_distance_miles": 3.02,
                "estimated_pickup_seconds": 16,
                "estimated_trip_duration_seconds": 676,
                "image_url": null,
                "product_id": "08798dd6-0426-44fd-8aab-af7c53bc014c",
                "product_type": "uberx"
            }
        ]
    },
    "error": false,
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": null,
        "href": "http://testserver/api/v1/offer/tvl/ride/pickup-estimates?ak1=1b691a31-88c1-4e6a-9fb0-3cd725cee214&ak2=7ca3b7e9-4050-43e6-864c-bb481e5ba3e1&provider=uber&ride_voucher_id=20929da7-f2ff-47bb-b826-094e54fdc752",
        "message": "OK",
        "method": "GET",
        "status": 200
    }
}

```


### Get Rideshare products info

Endpoint used to find out whether the pickup location is serviced by provider. Currently only works for Uber. Called to avoid errors on estimates call.

**Path:** /api/v1/offer/tvl/ride/products-info?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** GET<br/>

**Request query fields:**<br>
**provider:** string, either lyft or uber.<br/>
**direction:** string, in which direction does the ride goes. One of: airport_to_hotel, hotel_to_airport, airport_to_airport. Used with `ride_voucher_id`. Not required.<br/>
**pickup_latitude:** float. Not required.<br/>
**pickup_longitude:** float. Not required.<br/>
**ride_voucher_id:** uuid. Not required.<br/>

You should either provide pickup_latitude/pickup_longitude point or provide `ride_voucher_id` field. If `ride_voucher_id` is not provided and there are multiple ride vouchers, then first fit for the `direction` will be used. If no fit is found, first ride voucher will be used. 
If no `direction` is provided then `airport_to_hotel` is assumed.


If service is available response will be 200 with empty `data` field.<br>
If service is not available response will be 400 with `error_code` `uber_outside_service_area`.<br>

**200 Response Example:** 
```json
{
    "data": null,
    "error": false,
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": null,
        "href": "...",
        "message": "OK",
        "method": "GET",
        "status": 200
    }
}
```

**400 Response Example:** 
```json
{
    "data": null,
    "error": true,
    "meta": {
        "error_code": "uber_outside_service_area",
        "error_description": "Uber does not offer service at this airport. Please see an airline agent to arrange alternate transportation.",
        "error_detail": null,
        "href": "...",
        "message": "Bad Request",
        "method": "GET",
        "status": 400
    }
}
```


### Get Rideshare airport pickup options

Endpoint used to find out pickup points on airports for uber/lyft. The location.id is then used in ride booking call. `disabled_ride_types` field contains which ride types are not available at this location - only provided by Lyft. 

**Path:** /api/v1/offer/tvl/ride/{provider}/airport-pickup-options?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** GET<br/>

**Request query fields:**<br>
**ride_voucher_id:** uuid. Not required.<br/>

If `ride_voucher_id` is not provided and there are multiple ride vouchers, then first fit for the `direction == airport_to_hotel` will be used. If no fit is found, first ride voucher will be used. 
If no `direction` is provided then `airport_to_hotel` is assumed.


**Response data fields:**<br>
**airport_name:** airport name<br/>
**zones:** list, zones object with fields name,locations(list of objects)<br/>


**Response Example:** 
```json
{
    "data": {
        "airport_name": "Los Angeles International Airport",
        "zones": [
            {
                "locations": [
                    {
                        "disabled_ride_types": null,
                        "id": "e28a9f0d-9f06-4d4f-b3a4-bd281497733d",
                        "latitude": 33.4349961,
                        "longitude": -112.009549,
                        "name": "South Door 2 thru 6 - Outer Curb",
                        "note_to_traveller": "From baggage claim, during construction; exit Door 6 or 8 South, cross the crosswalk, turn right, and look for the rideshare signs on the outer curb by Door 2. Your driver will be waiting for you curbside."
                    }
                ],
                "name": "Terminal 3"
            },
            {
                "locations": [
                    {
                        "disabled_ride_types": null,
                        "id": "a48e0918-1286-48c9-b793-36437128abe3",
                        "latitude": 33.4349986,
                        "longitude": -112.008721,
                        "name": "South Door 8 - Outer Curb",
                        "note_to_traveller": "From baggage claim, exit Door 6 South, cross the crosswalk, and look for the Prearranged signs on the outer curb. Your driver will be waiting for you curbside. "
                    }
                ],
                "name": "Terminal 3 "
            },
            {
                "locations": [
                    {
                        "disabled_ride_types": null,
                        "id": "e295995b-ccea-4c6a-9d1a-301da0f9338f",
                        "latitude": 33.436439,
                        "longitude": -112.01361,
                        "name": "Past Door 8 - Inner Curb",
                        "note_to_traveller": "Exit door 8 to the East and look for the rideshare signs on the inner curb"
                    }
                ],
                "name": "Terminal 2"
            }
        ]
    },
    "error": false,
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": null,
        "href": "http://testserver/api/v1/offer/tvl/ride/uber/airport-pickup-options?ak1=dbb5d496-755b-42e3-a6d6-15c7cc3ef592&ak2=95435b63-c6e7-48c5-ac23-4783c868a66b&ride_voucher_id=a7432eaf-c576-4624-904d-53b4b044f7a5",
        "message": "OK",
        "method": "GET",
        "status": 200
    }
}

```

### Post Rideshare book ride

Endpoint used to book a ride. 

**Path:** /api/v1/offer/tvl/ride/accept?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** POST<br/>

**Request payload fields:**<br>
**provider:** string, uber/lyft<br/>
**product:** string, one of supported products: 'lyft', 'lyft_premier', 'lyft_lux','lyft_lux_black', 'lyft_plus', 'lyft_luxsuv', 'uberx', 'uber_comfort', 'uberx_black', 'uber_green', 'uberxl', 'uber_black_suv'<br/>
**product_id:** string, uber required<br/>
**ride_voucher_id:** uuid<br/>
**pickup_location_id:** String, required for airport_to_hotel, airport_to_airport ride_type. <br/>
**ride_type:** String, One of: airport_to_hotel, hotel_to_airport, airport_to_airport.  <br/>

**Response data fields:**<br>
**ride_status:** Object<br/>
**ride:** Object<br/>


**Response Example:** 
```json
{
    "data": {
        "ride": {
            "cell_phone_number": "+13455688889",
            "create_date": "2025-10-08 13:25:12.120095",
            "destination": {
                "airport_iata_code": null,
                "hotel_id": "tvl-98711",
                "latitude": null,
                "longitude": null,
                "name": "Ramada Inn - Torrance"
            },
            "id": "ca36ae14-ab6e-4280-93d9-32e8c13d8f2c",
            "origin": {
                "airport_iata_code": "LAX",
                "hotel_id": null,
                "latitude": null,
                "longitude": null,
                "name": "Los Angeles International Airport"
            },
            "product": "uberx",
            "provider": "uber",
            "ride_type": "airport_to_hotel",
            "status": "accepted"
        },
        "ride_status": {
            "canceled_by": null,
            "destination": {
                "address": "2888 Pacific Coast Highway, Torrance, CA 90505 USA",
                "eta_seconds": null,
                "latitude": 33.7949914,
                "longitude": -118.3375419
            },
            "driver": null,
            "driver_messages": null,
            "origin": {
                "address": "1 World Way Los Angeles, California 98045 USA",
                "eta_seconds": 7.0,
                "latitude": 33.4337477,
                "longitude": -112.0230657
            },
            "ride_id": "ca36ae14-ab6e-4280-93d9-32e8c13d8f2c",
            "ride_product": "uberx",
            "ride_provider": "uber",
            "route_url": null,
            "status": "pending",
            "vehicle": null
        }
    },
    "error": false,
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": null,
        "href": "http://testserver/api/v1/offer/tvl/ride/accept/?ak1=7c064457-5d87-41b8-a310-cfefd8dde311&ak2=e9d41970-f848-4ec2-8358-5499ffdfd939",
        "message": "OK",
        "method": "POST",
        "status": 200
    }
}

```


### Get Rideshare ride status

Endpoint used to find out status of a ride. Can be polled to see what happens real time.

**Path:** /api/v1/offer/tvl/ride/{ride_uuid}/status?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** GET<br/>

**Response data fields:**<br>
**canceled_by:** String, one of null/passenger/driver/provider <br/>
**destination:** object<br/>
**origin:** object<br/>
**driver:** object,or null, name,phone,rating,image<br/>
**driver_messages:** list of objects, messages from driver<br/>
**ride_id:** uuid<br/>
**ride_product:** String<br/>
**ride_provider:** String, lyft/uber<br/>
**route_url:** String<br/>
**status:** String<br/>
**vehicle:** Object or null, info about car<br/>


**Response Example:** 
```json
{
    "data": {
        "canceled_by": null,
        "destination": {
            "address": "TODO address",
            "eta_seconds": null,
            "latitude": 33.93088,
            "longitude": -118.40072
        }, 
        "driver": {
            "image_url": "https://lyftapi.s3.amazonaws.com/staging/photos/320x200/1531839227711722164_driver.jpg",
            "first_name": "Sandbox LAX 134cd",
            "id": "1531839227711722164",
            "rating": "5.0",
            "phone_number": "+15878483247"
        },
        "driver_messages": [],
        "origin": {
            "address": "LAX-it Pickup Lot, Zone 5A Curb",
            "eta_seconds": null,
            "latitude": 33.94733,
            "longitude": -118.398524
        },
        "ride_id": "ca83890e-c492-4fbb-8789-c81d5993070e",
        "ride_product": "lyft",
        "ride_provider": "lyft",
        "route_url": "https://ride-sandbox.lyft.net/csl/1675647202903546240/GJk7hfRnBp6Rg1fFsVieHA==",
        "status": "pending",
        "vehicle": {
            "make": "GMC",
            "license_plate": "3LYF123",
            "license_plate_state": "CA",
            "compliance_group_name": "PERSONAL",
            "year": 2018,
            "color": "Black",
            "model": "Yukon XL",
            "image_url": "https://s3.amazonaws.com/lyftapi/staging/photos/2018/gmc/yukonxl/black/whitebg/640x400/0121f051786d7fa9f96c98fd439d5dc9.png"
        }
    },
    "error": false,
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": null,
        "href": "http://testserver/api/v1/offer/tvl/ride/.../status?ak1=bdc5f7c3-c414-4ea8-8cf5-6a811a63239e&ak2=dfc30850-75b3-4769-98c4-b844bf3392bf",
        "message": "OK",
        "method": "GET",
        "status": 200
    }
}
```


### POST Rideshare ride cancel

Endpoint used to cancel a ride. No POST payload. 200 response on success, 400 otherwise.

**Path:** /api/v1/offer/tvl/ride/{ride_uuid}/cancel?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** POST<br/>


**Response Example:** 
```json
{
    "data": null,
    "error": false,
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": null,
        "href": "http://testserver/api/v1/offer/tvl/ride/.../cancel?ak1=bdc5f7c3-c414-4ea8-8cf5-6a811a63239e&ak2=dfc30850-75b3-4769-98c4-b844bf3392bf",
        "message": "OK",
        "method": "GET",
        "status": 200
    }
}
```


### GET Transaction
Endpoint used to get `transaction` objects by `transaction_id`. `transaction_id` is the `id` property
of the `transaction` objects returned in the `redemption/transaction` queue.
For more information on object structure see the `stormx_transaction` documentation.
* `"send_queue_message"` boolean (optional), if `true` will re-send the `transaction`
message to the `transaction/redemption` queue

**Path:** /api/v1/transaction/{transaction_id}?send_queue_message={send_queue_message}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "http://api.stormx.test/api/v1/transaction/3701f67c-7f4d-4017-a07e-84e86ec7ea05",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": {
        "billing_amount": 11.22,
        "billing_currency_code": "USD",
        "cc_id": "6b336bcc-3fce-405c-91dc-9be089b75d3e",
        "context_ids": [
            "contextid-34088BCF19994B5CB6CCC1F76247F37CX",
            "contextid-BF0D0EFA7B1641E3B7D8CC61488C2E59X"
        ],
        "conversion_rate": 1.0,
        "id": "3701f67c-7f4d-4017-a07e-84e86ec7ea05",
        "merchant_acceptor_id": "483200924999",
        "merchant_city": "PHOENIX",
        "merchant_country_code": "USA",
        "merchant_description": "HILTON PHOENIX AIRPORT",
        "merchant_state": "AZ",
        "merchant_zip": "85034",
        "original_amount": "112.5700",
        "posting_date": "2020-08-13 18:46:23",
        "sic_mcc_code": "7011",
        "source_amount": 11.22,
        "source_currency_code": "USD",
        "transaction_date": "2020-08-13 18:46:23",
        "transaction_type": "D",
        "type": "hotel",
        "unlock_reason": "default",
        "voucher_id": "7ab0f0f8-9de1-4b0f-870c-e2514721da5a"
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | INVALID_INPUT | There were problems with one or more fields provided in the query parameters. |
| 404 | TRANSACTION_NOT_FOUND |  |

### POST Job
Endpoint used to submit larger requests to StormX which will be run asynchronously. These larger requests are known as `job`s.

**Path:** /api/v1/job<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>

**Payload fields:**<br/>
**name:** string, choices: `"counts"`<br>
**input_data:** JSON, payload for correlating `job.name`, not required for all jobs, can be `null`<br>

**Response data fields:**<br>
**id:** UUID, the unique identifier of the `job`.<br>
**create_date:** datetime, the `datetime` in UTC the `job` was created.<br>
**finish_date:** datetime, the `datetime` in UTC the `job` was finished.<br>
**status:** `status` of the job. possible values: `"submitted"`, `"started"`, `"succeeded"`,
`"failed"`, `"canceled"`.<br>
**start_date:** datetime, the `datetime` in UTC the `job` was created.<br>
**name:** string, the `name` of the `job`.<br>
**input_data:** JSON, input related data for correlating `job.name` (may be null)<br>
**output_data:** JSON, output related data for correlating `job.name` (may be null)<br>

Please find are the details of the job specific data below.

#### `counts` job

**background:** Under normal operation, your airline system will ingest API responses and queue messages to stay synced with StormX. If your system degrades, loses messages, or experiences a data loss, this job can help you identify missing records so that you can recover them.

This `job` is used to request record counts and a large file of UUID lists within a date range. The `type` field is submitted in the job to control which type of record IDs should be returned.

**recommended use:** You may use the returned record count for a simple check. You can download the UUID records from StormX and compare them to the same time frame in your system. If a UUID is not present in your system,
the `voucher_id` is available. Use the `voucher_id` with the `GET` /voucher endpoint
to receive the related `context_id`s, and `voucher` information to proceed as needed, i.e.
requesting full state queue messages and transaction re-pushes via `passenger state` and `transaction` endpoints.

**input_data fields:**<br/>
**type:** string, choices: `"meal_voucher"`, `"hotel_voucher"`, `"meal_transaction"`, `"hotel_transaction"`<br>
  Note: when you select a type, the IDs returned will be related to the following data type fields:
  * **meal_voucher:** `transaction.cc_id`
  * **hotel_voucher:** `transaction.cc_id`
  * **ride_voucher:** `transaction.cc_id`
  * **meal_transaction:** `transaction.id`
  * **hotel_transaction:** `transaction.id`
  * **ride_transaction:** `transaction.id`
  <br>
**report_start_date:** date string, format: %Y-%m-%d<br> NOTE: this correlates to StormX's `voucher`/`transaction` `create_date` in UTC.<br>
**report_end_date:** date string, format: %Y-%m-%d<br> NOTE: this correlates to StormX's `voucher`/`transaction` `create_date` in UTC. <br> NOTE: maximum difference between `report_start_date` and `report_end_date` is 7 days.
`voucher` and `transaction` UUID's returned will be greater than or equal to `report_start_date`
and strictly less than `report_end_date`.<br>
**format:** string, choices: `"json"`, `"csv"`<br>


**Input payload example:**<br>

```json
{
    "name": "counts",
    "input_data": {
        "type": "hotel_voucher",
        "report_start_date": "2020-08-01",
        "report_end_date": "2020-08-08",
        "format": "json"
    }
}
```

**Asynchronous response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/job",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": {
        "create_date": "2020-12-08 19:59:29.340711",
        "finish_date": null,
        "id": "0169d8ca-ecb4-4c4e-917f-133c355d7891",
        "input_data": {
            "format": "json",
            "report_end_date": "2020-08-08",
            "report_start_date": "2020-08-01",
            "type": "hotel_voucher"
        },
        "output_data": null,
        "name": "counts",
        "start_date": null,
        "status": "submitted"
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 400 | JOB_REQUEST_ERROR | There were problems with one or more fields provided in the post data. |

### GET Job
Endpoint used to check on asynchronous `job` status and receive `job` details.

**Path:** /api/v1/job<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>

**Response data fields:**<br>
**id:** UUID, the unique identifier of the `job`.<br>
**create_date:** datetime, the `datetime` in UTC the `job` was created.<br>
**finish_date:** datetime, the `datetime` in UTC the `job` was finished.<br>
**status:** `status` of the job. possible values: `"submitted"`, `"started"`, `"succeeded"`,
`"failed"`, `"canceled"`.<br>
**start_date:** datetime, the `datetime` in UTC the `job` was created.<br>
**name:** string, the `name` of the `job`.<br>
**input_data:** JSON, job data for correlating `job.name` (may be null)<br>
**output_data:** JSON, job data for correlating `job.name` (may be null)<br>

Please find are the details of the job specific data below.

#### the `counts` job response
**input_data:**<br>
- **format:** string, the `datetime` the `job` was created.<br>
- **report_start_date:** date string, format: %Y-%m-%d<br>
- **report_end_date:** date string, format: %Y-%m-%d<br>
- **type:** string, the data type of the `UUID`'s returned for the `job`<br>

**output_data:**<br>
- **count:** integer, the quantity of `UUID`s returned for the `job`.<br>
- **expires:** datetime, the `datetime` in UTC the `file` containing the `job` data expires (5 hours from creation).<br>
- **file_path:** string, the URL of the `file` containing the `job` data. Download this file to obtain the contents of the report you requested. The file will be available at the link for a minimum of 5 hours.<br>

**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/job/0169d8ca-ecb4-4c4e-917f-133c355d7891",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": {
        "create_date": "2020-12-08 19:59:29.340711",
        "finish_date": "2020-12-08 20:00:13.672728",
        "id": "0169d8ca-ecb4-4c4e-917f-133c355d7891",
        "input_data": {
            "format": "json",
            "report_end_date": "2020-08-08",
            "report_start_date": "2020-08-01",
            "type": "hotel_voucher"
        },
        "output_data": {
            "count": 1837,
            "expires": "2020-12-08 21:00:12.481357",
            "file_path": "https://stormx-sxa018-s3airlinejob.s3.amazonaws.com/0169d8ca-ecb4-4c4e-917f-133c355d7891.json?Signature=abcdXd72ggXrNJ8syUdagnQaboc%3D&Expires=1607461213&AWSAccessKeyId=1234WFV2NRA4XBZM67EZ"      
        },
        "name": "counts",
        "start_date": null,
        "status": "succeeded"
    }
}
```

**Response status codes and error codes**<br/>

| HTTP status | meta.error_code | comments |
| ---- | ---- | ---- |
| 200 |               | successful |
| 404 | JOB_NOT_FOUND | There was no job found for the provided id. |



### POST Passenger Physicalize

Passenger Physicalize is used to convert a previously issued welfare on a digital card to a physical card. Typically used when the passenger is unable to use the digital wallet and would like the funds transferred to a physical card.
<br><br>

**Path:** /api/v1/passenger/{context_id}/meal/physicalize<br/>
**Headers:** None<br/>
**Method:** POST<br/>

**Payload fields:**<br/>
**physical_card_id:** string max_length=60<br>

**Payload example:**
```json
{
    "physical_card_id": "some_id"
}
```

**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "/api/v1/passenger/7dfb4fa61076491cbb50836e28599c89/meal/physicalize",
        "message": "OK",
        "method": "POST",
        "status": 201
    },
    "error": false,
    "data": null
}
```

### POST Deactivate Vouchers

This endpoint may be used to deactivate Swiipr meal vouchers.
<br><br>

**Path:** /api/v1/event/{event_id}/deactivate_vouchers<br/>
**Headers:** None<br/>
**Method:** POST<br/>


**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "/api/v1/event/7dfb4fa61076491cbb50836e28599c89/deactivate_vouchers",
        "message": "OK",
        "method": "POST",
        "status": 201
    },
    "error": false,
    "data": null
}
```

### POST Add funds to Swiipr meal voucher for all passengers of an event

Add funds and optionally extend its validity to the passengers registered under an event. This endpoint may be used to add funds to previously issued digital / physical cards.
<br><br>

**Path:** /api/v1/event/{event_id}/swiipr-add-funds<br/>
**Headers:** None<br/>
**Method:** POST<br/>

**Payload fields:**<br/>
**amount:** String Decimal<br>
**extend_days:** int 0-99, 0 means don't extend validity<br>

**Payload example:**
```json
{
    "amount": "10.00",
    "extend_days": 0
}
```

**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "/api/v1/event/7dfb4fa61076491cbb50836e28599c89/swiipr-add-funds",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": null
}
```


### POST Add funds to Swiipr meal voucher for a specific passenger of an event

Add funds and optionally extend its validity to a specific passenger registered under an event. This endpoint may be used to add funds to a previously issued digital / physical card of a passenger.
<br><br>

**Path:** /api/v1/passenger/{context_id}/meal/{event_id}/swiipr-add-funds<br/>
**Headers:** None<br/>
**Method:** POST<br/>

**Payload fields:**<br/>
**amount:** String Decimal<br>
**extend_days:** int 0-99, 0 means don't extend validity<br>

**Payload example:**
```json
{
    "amount": "10.00",
    "extend_days": 0
}
```

**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "/api/v1/passenger/1cbb50836e28599c89/meal/2309abc/swiipr-add-funds",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": null
}
```


## Passenger API

These Passenger API endpoints are meant specifically for building your own Passenger App for distressed passengers.

These API methods require the `ak1` and `ak2` credentials, meaning that access is on behalf of the individual passenger, not on behalf of the airline (see Authentication section).

Note: These API endpoints are NOT meant for building an Airline Agent experience.


### GET Offer/Hotels


Public endpoint for getting hotel availability.
Intended use by the passenger app to display hotel availability for a passenger offer.

**Path:** /api/v1/offer/hotels?ak1={ak1}&ak2={ak2}&room_count={room_count}<br/>
**Headers:** None<br/>
**Method:** GET<br/>
**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/offer/hotels?ak1=0f33dc1e484e408592a9ec20c527515c&ak2=6a2b18171ae541d7b46de926cbef408c&room_count=1",
        "message": "OK",
        "method": "GET",
        "status": 200
    },
    "error": false,
    "data": [
        {
            "address1": "9876 Wilshire Boulevard,",
            "address2": "Beverly Hills",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "available": 60,
            "city": "Los Angeles",
            "country": "USA",
            "currency_code": "USD",
            "distance": 8.658,
            "hard_block_count": 60,
            "contract_block_count": 6,
            "hotel_allowances": {
                "amenity": {
                   "amount": "50.00"
                },
                "meals": {
                    "breakfast": {
                        "rate": "10.00"
                    },
                    "dinner": {
                        "rate": "20.00"
                    },
                    "lunch": {
                        "rate": "30.00"
                    }
                }
            },
            "hotel_id": "tvl-82940",
            "hotel_name": "The Beverly Hilton",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "image_url": "https://i.travelapi.com/hotels/1000000/10000/8000/7962/7962_258_b.jpg",
            "latitude": 34.066833,
            "longitude": -118.413185,
            "passenger_note": "",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "13102747777",
            "post_code": "90210",
            "premium": true,
            "proposed_check_in_date": "2018-05-03",
            "proposed_check_out_date": "2018-05-04",
            "restaurant_on_property": true,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_available_now": true,
            "star_rating": 4,
            "state": "CA"
        },
        {
            "address1": "2701 Main Street ",
            "address2": "Irvine",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "available": 55,
            "city": "Los Angeles",
            "country": "USA",
            "currency_code": "USD",
            "distance": 12.15,
            "hard_block_count": 55,
            "contract_block_count": 55,
            "hotel_allowances": {
                "amenity": null,
                "meals": null
            },
            "hotel_id": "tvl-83823",
            "hotel_name": "Courtyard Irvine John Wayne Airport",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "image_url": "https://i.travelapi.com/hotels/1000000/10000/2800/2717/dfea3820_b.jpg",
            "latitude": 34.052234,
            "longitude": -118.243685,
            "passenger_note": "",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "19497571200",
            "post_code": "92614",
            "premium": true,
            "proposed_check_in_date": "2018-05-03",
            "proposed_check_out_date": "2018-05-04",
            "restaurant_on_property": true,
            "service_pets_allowed": false,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_available_now": true,
            "star_rating": 4,
            "state": "CA"
        }
    ]
}
```


### Book Offer/Hotels


Public endpoint used by the passenger app for accepting a hotel offer and creating a voucher for all `context_ids` passed into the request.
All passengers in a state of offered on the same PNR must accept the offer at the same time through this endpoint.

**Path:** /api/v1/offer/hotels?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** POST<br/>

**Payload fields:**<br/>
* **context_ids:** list of context ids accepting the offer
* **hotel_id:** hotel_id from hotel object returned in the get hotel inventory request
* **room_count:** number of rooms allowed to reserve for voucher

**Payload example:**

```json
{
    "context_ids": ["adultCON","childCON"],
    "hotel_id": "tvl-82940",
    "room_count": 1
}
```

**Response data fields:**
* **data.hotel_voucher:** a [HotelVoucherResponseObjectForPassenger](#hotelvoucherresponseobjectforpassenger) object.
* **data.modified_date** `modified_date` of the voucher.
* **data.status** `status` of the voucher. The value is limited to only `"finalized"` or `"canceled"`.
* **data.passengers:** an array of [VoucherPassengerResponseObjectForPassenger](#voucherpassengerresponseobjectforpassenger).


**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/offer/hotels?ak1=0f33dc1e484e408592a9ec20c527515c&ak2=6a2b18171ae541d7b46de926cbef408c",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": {
        "hotel_voucher": {
            "address1": "9876 Wilshire Boulevard,",
            "address2": "Beverly Hills",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "check_in_date": "2018-04-23",
            "check_out_date": "2018-04-24",
            "city": "Los Angeles",
            "confirmation_id": null,
            "country": "USA",
            "currency_code": "USD",
            "distance": 8.658,
            "hotel_allowances": {
                "amenity": null,
                "meals": null
            },
            "hotel_id": "tvl-82940",
            "hotel_key": "RRIRG",
            "hotel_name": "The Beverly Hilton",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "image_url": "https://i.travelapi.com/hotels/1000000/10000/8000/7962/7962_258_b.jpg",
            "latitude": 34.066833,
            "longitude": -118.413185,
            "passenger_note": "",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "13102747777",
            "post_code": "90210",
            "premium": true,
            "restaurant_on_property": true,
            "rooms_booked": 1,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_tracker_url": null,
            "star_rating": 4,
            "state": "CA"
        },
        "modified_date": "2018-04-24 17:56:48.858999",
        "passengers": [
            {
                "context_id": "adultCON",
                "canceled_date": null,
                "declined_date": null,
                "hotel_accommodation_status": "accepted",
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "meal_accommodation_status": "accepted",
                "meal_vouchers": [
                    {
                        "active_from": "2018-04-24 10:56",
                        "active_to": "2018-04-25 20:59",
                        "amount": "15.00",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001476449",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiREJhOHFUcGNSWE9QZEpzTGVSaDU3OUdDSGZXc3hFRHdvSGtSVWNJZ0R3QSJ9.QuVn0pojbicYBAFXAlCNCWyFpEU/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": null
                    },
                    {
                        "active_from": "2018-04-24 10:56",
                        "active_to": "2018-04-25 20:59",
                        "amount": "12.99",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001476456",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiU1ZMZ3U2cFZUdmVxRzdrY0tqbDVSSW5VaWZ2UkhVdGlpLTBGejFXVlZMQSJ9.cFcoe_mVVkdSr7Vcmo9jBiWPKbM/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl"
                    }
                ],
                "modified_date": "2018-04-24 17:56:58.215387",
                "notifications": [
                    {
                        "notification_type": "offer",
                        "sent_date": "2018-04-24 17:54:49.319183",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "notification_type": "offer",
                        "sent_date": "2018-04-24 17:54:49.355821",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    },
                    {
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-24 17:56:48.984373",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "notification_type": "confirmation",
                        "sent_date": "2018-04-24 17:56:48.996586",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    }
                ],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            },
            {
                "context_id": "childCON",
                "canceled_date": null,
                "declined_date": null,
                "hotel_accommodation_status": "accepted",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "not_offered",
                "meal_vouchers": [],
                "modified_date": "2018-04-24 17:56:58.215387",
                "notifications": [],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            }
        ],
        "status": "finalized"
    }
}
```


### Offer Notifications

Offer notifications endpoint is used to send the passengers confirmation details page to an email or phone number.
This endpoint can only be used if the passengers offer is in a final state and has vouchers.
See PassengerNotificationResponseObject for detailed description of response object.

**Path:** /api/v1/offer/notifications?ak1={ak1}&ak2={ak2}<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>

**Payload fields:**<br/>
**email:** string max_length=45<br>
**text:** string max_length=16

**Email notification payload example:**
```json
{
    "email": "famousjjBro@aol.com"
}
```

**Email notification response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/offer/notifications?ak1=0c07051ef4c24d8eb24b4b70abce8493&ak2=9ac26a6063bb4681aa38e3f67b93fe14",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": {
        "notification_type": "confirmation",
        "sent_date": null,
        "sent_to": "famousjjBro@aol.com",
        "sent_via": "email"
    }
}
```

**Text notification payload example:**
```json
{
    "text": "+13125551234"
}
```

**Text notification response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/offer/notifications?ak1=0c07051ef4c24d8eb24b4b70abce8493&ak2=9ac26a6063bb4681aa38e3f67b93fe14",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": {
        "notification_type": "confirmation",
        "sent_date": null,
        "sent_to": "+13125551234",
        "sent_via": "text"
    }
}
```


### PUT Passenger Decline Offer

This public endpoint may be used to decline an offer if the passengers is in a state of offered only, meaning the passenger has not accepted or canceled an offer yet.
This endpoint will decline the offer for all passengers with the same pax_record_locator_group.
If passengers have meal_accommodation set to `true`, then the meal vouchers will be returned in the response for all passengers.

**Path:** /api/v1/offer/decline?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** PUT<br/>

**Response data fields:**
* **data.hotel_voucher:** always `null` when declining an offer.
* **data.modified_date** `modified_date` of the voucher.
* **data.status** `status` of the voucher. The value is limited to only `"finalized"` or `"canceled"`.
* **data.passengers:** an array of [VoucherPassengerResponseObjectForPassenger](#voucherpassengerresponseobjectforpassenger).


**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/offer/decline?ak1=0c07051ef4c24d8eb24b4b70abce8493&ak2=9ac26a6063bb4681aa38e3f67b93fe14",
        "message": "UPDATED",
        "method": "PUT",
        "status": 200
    },
    "error": false,
    "data": {
        "hotel_voucher": null,
        "modified_date": "2018-04-24 17:36:53.588494",
        "passengers": [
            {
                "context_id": "adultCON",
                "canceled_date": null,
                "declined_date": "2018-04-24 17:36:53.588494",
                "hotel_accommodation_status": "declined",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "accepted",
                "meal_vouchers": [
                    {
                        "active_from": "2018-04-24 10:36",
                        "active_to": "2018-04-25 20:59",
                        "amount": "15.00",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001476423",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiQWc4c2JGMXdUaEdtcVpEUkRVMzhQeUhwMjB5YlcwYWZyNUVRdzJNaVNjOCJ9.VQZur8RiahrbnEx6_-JHDTv4P6I/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": null
                    },
                    {
                        "active_from": "2018-04-24 10:36",
                        "active_to": "2018-04-25 20:59",
                        "amount": "12.99",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001476431",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "04/2021",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiWUtYVjV2M2NUSGFtWmd6NnJuLXBMUVdDM2kyTW5VaFZrY21aWTYtMlBxNCJ9.WtUmwpG6ss8BoU2z_PMzVkGH8gE/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl"
                    }
                ],
                "modified_date": "2018-04-24 17:36:53.572617",
                "notifications": [
                    {
                        "notification_type": "offer",
                        "sent_date": "2018-04-24 17:35:17.364742",
                        "sent_to": "+13125551234",
                        "sent_via": "text"
                    },
                    {
                        "notification_type": "offer",
                        "sent_date": "2018-04-24 17:35:17.412080",
                        "sent_to": "famousjj@aol.com",
                        "sent_via": "email"
                    }
                ],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            },
            {
                "context_id": "childCON",
                "canceled_date": null,
                "declined_date": "2018-04-24 17:36:53.588494",
                "hotel_accommodation_status": "declined",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "not_offered",
                "meal_vouchers": [],
                "modified_date": "2018-04-24 17:36:53.572617",
                "notifications": [],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            }
        ],
        "status": "finalized"
    }
}
```


### PUT Passenger Cancel Offer

This public endpoint may be used to cancel an offer or voucher.
<br><br>
Certain conditions must be met for the cancellation to be successful:<br>
* the offer/voucher is not already canceled.
* the hotel credit card is still locked (has not been unlocked by the hotel).

**Path:** /api/v1/offer/cancel?ak1={ak1}&ak2={ak2}<br/>
**Headers:** None<br/>
**Method:** PUT<br/>

**Response data fields:**
* **data.hotel_voucher:** a [HotelVoucherResponseObjectForPassenger](#hotelvoucherresponseobjectforpassenger) object.
* **data.modified_date** `modified_date` of the voucher.
* **data.status** `status` of the voucher. The value is limited to only `"finalized"` or `"canceled"`.
* **data.passengers:** an array of [VoucherPassengerResponseObjectForPassenger](#voucherpassengerresponseobjectforpassenger).

**Response example:**
```json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/offer/cancel?ak1=2d5599a23e3a4c479938524bea76984d&ak2=7dfb4fa61076491cbb50836e28599c89",
        "message": "UPDATED",
        "method": "PUT",
        "status": 200
    },
    "error": false,
    "data": {
        "hotel_voucher": {
            "address1": "435 Culver Blvd",            
            "address2": "",
            "address3": "",
            "address4": "",
            "airport_code": "LAX",
            "amenities": [
                {
                    "name": "Breakfast"
                },
                {
                    "name": "Lunch"
                },
                {
                    "name": "Dinner"
                },
                {
                    "name": "Pool"
                },
                {
                    "name": "Fitness Center"
                },
                {
                    "name": "Cocktail Hour"
                },
                {
                    "name": "Restaurant on Property"
                }
            ],
            "check_in_date": "2018-05-09",
            "check_out_date": "2018-05-10",
            "city": "Playa del Rey",
            "confirmation_id": null,
            "country": "USA",
            "currency_code": "USD",
            "distance": 2.525,
            "hotel_allowances": {
                "amenity": null,
                "meals": null
            },
            "hotel_id": "tvl-86038",
            "hotel_key": "LZNJX",
            "hotel_name": "Inn at Playa del Rey",
            "hotel_on_airport": false,
            "hotel_on_airport_instructions": "",
            "image_url": "https://i.travelapi.com/hotels/1000000/70000/62600/62552/62552_46_b.jpg",
            "latitude": 33.961629,
            "longitude": -118.445365,
            "passenger_note": "",
            "pets_allowed": false,
            "pets_fee": "0.00",
            "phone_number": "1 310-574-1920",
            "post_code": "90293",
            "premium": true,
            "restaurant_on_property": true,
            "rooms_booked": 1,
            "service_pets_allowed": true,
            "service_pets_fee": "0.00",
            "shuttle": true,
            "shuttle_timing": "0:00 23:59",
            "shuttle_tracker_url": null,
            "star_rating": 4,
            "state": "California"
        },
        "modified_date": "2018-05-09 18:18:11.744225",
        "passengers": [
            {
                "context_id": "adultCON4",
                "canceled_date": "2018-05-09 18:18:11.744225",
                "declined_date": null,
                "hotel_accommodation_status": "canceled_voucher",
                "hotel_allowance_status": {
                    "amenity": "not_offered",
                    "breakfast": "not_offered",
                    "lunch": "not_offered",
                    "dinner": "not_offered"
                },
                "hotel_allowances_voucher": {
                    "amenity": null,
                    "meals": null
                },
                "meal_accommodation_status": "accepted",
                "meal_vouchers": [
                    {
                        "active_from": "2018-05-09 11:17",
                        "active_to": "2018-05-10 20:59",
                        "amount": "15.00",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001597012",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "05/2021",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiODhjYUtHbTdTbU9OZ0ZGSlNjcVJkUnN1LWh2OGhrRjJvMzQ4eENfUTNZNCJ9.pLTKQZH6gJCUXomgUTZ2VDzS-eQ/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl",
                        "url": null
                    },
                    {
                        "active_from": "2018-05-09 11:17",
                        "active_to": "2018-05-10 20:59",
                        "amount": "12.99",
                        "billing_zip_code": "55343",
                        "card_number": "0000000001597020",
                        "card_type": "MASTERCARD",
                        "currency_code": "USD",
                        "cvc2": "1234",
                        "expiration": "05/2021",
                        "qr_code_url": "https://sxa018.tvlinc.com/offer/meal/qr/eyJ1IjoiUGxfbEx4SjBTbXVrTnB6NTJoSnlRdmVIM19UNFJrQmZtV1habENRczVqayJ9.9UAttp7R6luUtRuGFChNdW-pmek/meal.png",
                        "time_zone": "America/Los_Angeles",
                        "provider": "tvl"
                    }
                ],
                "modified_date": "2018-05-09 18:18:11.744225",
                "notifications": [],
                "offer_opened_date": null,
                "transport_accommodation_status": null
            }
        ],
        "status": "canceled"
    }
}
```


### GET Queue Attributes

This endpoint may be used to gather all of the public data on every queue pertaining to your airline.
<br><br>

**Path:** /api/v1/queue/attributes<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>

**Response data fields:**
* **data:** an array of object representations of your airline queues. Each object contains all public SQS queue attributes.

**Response example:**
```json
{
    "meta": {
        "status": 200,
        "message": "OK",
        "method": "GET",
        "href": "https://sxa018.tvlinc.com/api/v1/queue/attributes",
        "error_code": "",
        "error_description": "",
        "error_detail": null
    },
    "data": [
        {
            "QueueArn": "arn:aws:sqs:us-west-2:424518977593:SQS-QueueName-1",
            "ApproximateNumberOfMessages": "44",
            "ApproximateNumberOfMessagesNotVisible": "0",
            "ApproximateNumberOfMessagesDelayed": "0",
            "CreatedTimestamp": "1568918127",
            "LastModifiedTimestamp": "1690822870",
            "VisibilityTimeout": "30",
            "MessageRetentionPeriod": "345600",
            "DelaySeconds": "0",
            "ReceiveMessageWaitTimeSeconds": "0"
        },
        {
            "QueueArn": "arn:aws:sqs:us-west-2:424518977593:SQS-QueueName-2",
            "ApproximateNumberOfMessages": "0",
            "ApproximateNumberOfMessagesNotVisible": "0",
            "ApproximateNumberOfMessagesDelayed": "0",
            "CreatedTimestamp": "1568918265",
            "LastModifiedTimestamp": "1690822304",
            "VisibilityTimeout": "30",
            "MessageRetentionPeriod": "345600",
            "DelaySeconds": "0",
            "ReceiveMessageWaitTimeSeconds": "0"
        }
    ],
    "error": false
}
```


## Pax Pay API 

The Pax Pay API allows for passenger pay via Rakuten as a room provider. 

These 6 endpoints are secured over Basic Auth, meaning your airline credentials are to be used. Once you receive your session id from the `Get Hotels` call, all subsequent endpoint calls pertaining to the particular booking should contain your unique version 4 UUID session id. An overview of the booking flow is as follows:
  - Get Hotels
  - Get Rooms
  - Get Room Policy Details
  - Book Hotel
  - Get Booking Status

### Endpoints

#### Get Hotels

Get hotel availability in the vicinity of a specific airport. Calling this endpoint returns a version 4 UUID `session_id`, which needs to be included in all other API calls.

**Path:**  /api/v1/pax/hotels<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>

**Query Parameters:**
* **adult_count (optional):** integer; minimum length 1, max length 36. The number of adults to query availability for.
* **airport:** string; max length 3. The airport to query availability for.
* **currency (optional):** string; max length 3. The currency to query availability for.
* **format (optional):** string, either "json" or "xlsx". the format of response data
* **locale (optional):** string, max length 5. the locale to query availability for.
* **max_distance (optional):** integer, minimum value 1, max value 100. how many miles to limit availability query to.
* **premium (optional):** boolean. whether to include premium availability in search query.
* **roooms_count (optional):** integer. The number of rooms to query availability for.
* **source_market (optional):** string. The source market to query availability in.
* **stay_date (optional):** string, in yyyy-mm-dd format. The date to limit the availability query to.
* **stay_nights (optional):** integer, minimum value 1. The number of nights to limit the availability query to.

**Response example:**
~~~json
{
    "session_id": "e48b0839-b8e5-4e51-98aa-545e985227f3",
    "hotels": [
      {
        "address": "Tsurunocho Kita-ku",
        "airport_code": "KIX",
        "city": "Osaka",
        "country": "JP",
        "distance": 23.8396,
        "hotel_id": "usj1",
        "hotel_name": "[TEST] Budget hotel USD1",
        "image_url": "",
        "latitude": 34.707555,
        "longitude": 135.500761,
        "phone_number": "81-6-6666-6666",
        "post_code": "530-0014",
        "premium": false,
        "provider": "rakuten",
        "shuttle": false,
        "star_rating": 0,
        "state": "Osaka",
        "room": {
          "beds": [
            {
              "name": "queen",
              "quantity": 2
            }
          ],
          "hotel_id": "usj1",
          "name": "Standard - Standard Room Queen Beds",
          "price": "1.01",
          "room_id": "b8c80743",
          "currency": "USD",
          "created": "2024-11-22T18:09:07.110906Z"
        }
      }
    ]
  }
~~~

#### Get Pax Rooms

This call will return specific room availability for selected hotel

**Path:**  /api/v1/pax/rooms<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Query Parameters:**
* **format (optional):** string, either "json" or "xlsx". the format of response data
* **hotel_id:** string; The id of the hotel
* **session_id:** string; your version 4 UUID session id

**Response example:**
~~~json
{
    "availability": {
      "address": "Tsurunocho Kita-ku",
      "airport_code": "KIX",
      "city": "Osaka",
      "country": "JP",
      "distance": 23.839642113246402,
      "hotel_id": "usj1",
      "hotel_name": "[TEST] Budget hotel USD1",
      "image_url": "",
      "latitude": 34.707555,
      "longitude": 135.500761,
      "phone_number": "81-6-6666-6666",
      "post_code": "530-0014",
      "premium": false,
      "provider": "rakuten",
      "rooms": [
        {
          "beds": [
            {
              "name": "queen",
              "quantity": 2
            }
          ],
          "hotel_id": "usj1",
          "name": "Standard - Standard Room Queen Beds",
          "price": "1.01",
          "room_id": "b8c80743",
          "currency": "USD",
          "created": "2024-11-22T18:10:04.746602Z"
        },
        {
          "beds": [
            {
              "name": "queen",
              "quantity": 2
            }
          ],
          "hotel_id": "usj1",
          "name": "Deluxe - Deluxe Room Queen Beds",
          "price": "1.55",
          "room_id": "4e62f15e",
          "currency": "USD",
          "created": "2024-11-22T18:10:04.746615Z"
        }
      ],
      "shuttle": false,
      "star_rating": 0,
      "state": "Osaka"
    }
  }
~~~

#### Get Room Policy Details

Get additional info for selected room, such as cancelations and final prices

**Path:**  /api/v1/pax/room-policy<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Query Parameters:**
* **format (optional):** string, either "json" or "xlsx". the format of response data
* **hotel_id:** string; The id of the hotel
* **room_id:** string; the id of the room
* **session_id:** string; your version 4 UUID session id

**Response example:**
~~~json
{
    "policy_id": "35577996-a8fd-11ef-abd8-52a6b732b52b",
    "cancellation_policy": {
      "remarks": "Swimming pool will be closed from June 9 to June 20",
      "cancellation_policies": [
        {
          "penalty_percentage": 0,
          "date_from": "2024-11-22T00:00:00Z",
          "date_to": "2024-11-23T00:00:00Z"
        },
        {
          "penalty_percentage": 100,
          "date_from": "2024-11-23T00:00:00Z",
          "date_to": "2024-11-24T00:00:00Z"
        }
      ]
    },
    "check_in_instructions": null,
    "hotel_fees": {
      "resort_fee": {
        "currency": "USD",
        "value": 5
      }
    },
    "room": {
      "beds": [
        {
          "name": "queen",
          "quantity": 2
        }
      ],
      "hotel_id": "usj1",
      "name": "Standard Room Queen Beds",
      "price": "1.01",
      "room_id": "b8c80743",
      "currency": "USD",
      "created": "2024-11-22T18:11:34.739465Z"
    },
    "differs_from_request": false
  }
~~~

#### Book Hotel

Book a hotel

**Path:**  /api/v1/pax/book<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>
**Query Parameters:**
* **format (optional):** string, either "json" or "xlsx". the format of response data

**Request payload example:**
~~~json
{
  "session_id": "e48b0839-b8e5-4e51-98aa-545e985227f3",
  "hotel_id": "usj1",
  "booking_policy_id": "35577996-a8fd-11ef-abd8-52a6b732b52b",
  "client_reference": "string",
  "room_lead_guests": [
    {
      "first_name": "John",
      "last_name": "Doe",
      "nationality": "US"
    }
  ],
  "contact_person": {
    "first_name": "Jane",
    "last_name": "Doe",
    "phone": "string",
    "email": "jd@example.com",
    "city": "Phoenix",
    "state": "AZ",
    "street": "456 N Central Ave #123",
    "postal_code": "85043",
    "country": "US"
  }
}
~~~
**Response example:**
~~~json
{
  "booking_id": "1",
  "status": "ok",
  "cancellation_policy": {
    "remarks": "Swimming pool will be closed from June 9 to June 20",
    "cancellation_policies": [
      {
        "penalty_percentage": 0,
        "date_from": "2024-11-22T00:00:00Z",
        "date_to": "2024-11-23T00:00:00Z"
      },
      {
        "penalty_percentage": 100,
        "date_from": "2024-11-23T00:00:00Z",
        "date_to": "2024-11-24T00:00:00Z"
      }
    ]
  },
  "requested_at": "2024-11-22T18:20:25Z",
  "confirmed_at": "2024-11-22T18:20:25Z",
  "client_reference": "string",
  "booking_details": {
    "customer_reference": "zbr-5400507",
    "customer_booking_id": "",
    "hotel_confirmation_id": "TEST_HCN",
    "additional_policies": "Please check for additional policies.",
    "additional_checkin_instructions": "Yet another string field for check-in instructions, more than likely empty."
  }
}
~~~

#### Get Booking Status

Retrieve the status of a booking

**Path:**  /api/v1/pax/booking-status<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** GET<br/>
**Query Parameters:**
* **booking_id:** string; the id of the booking
* **format (optional):** string, either "json" or "xlsx". the format of response data
* **session_id:** string; your version 4 UUID session id

**Response example:**
~~~json
{
  "status": "bkg-cf"
}
~~~

#### Cancel Booking

Cancel a booking

**Path:**  /api/v1/pax/booking-cancel<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>
**Query Parameters:**
* **format (optional):** string, either "json" or "xlsx". the format of response data

**Request payload example:**
~~~json
{
  "session_id": "e48b0839-b8e5-4e51-98aa-545e985227f3",
  "booking_id": "1"
}
~~~
**Response example:**
~~~json
{
  "status": "bkg-cx"
}
~~~


#### Summary hotel availability for ports

For specified ports will return sums of availability for all the hotels belonging 
to the port. Keys in the response: 
* "sb" - soft block
* "hb" - hard block
* "cb" - contract block

**Path:**  /api/v1/hotels/availability-by-port<br/>
**Headers:** Authorization: Basic {hash}<br/>
**Method:** POST<br/>
**Query Parameters:**
* **format (optional):** string, either "json" or "xlsx". the format of response data

**Request payload example:**
~~~json
{
  "ports": ["JFK", "SFO"]
}
~~~
**Response example:**
~~~json
{
    "meta": {
        "error_code": "",
        "error_description": "",
        "error_detail": [],
        "href": "https://sxa018.tvlinc.com/api/v1/hotels/availability-by-port",
        "message": "OK",
        "method": "POST",
        "status": 200
    },
    "error": false,
    "data": [
        {
            "port": "JFK",
            "sb": 1,
            "hb": 0,
            "cb": 0
        },
        {
            "port": "SFO",
            "sb": 0,
            "hb": 0,
            "cb": 0
        }
    ]
}
~~~

### GET Passenger Offer Vouchers


Endpoint used to get passenger's vouchers.

**Path:** /api/v1/offer/voucher?ak1={ak1}&ak2={ak2}<br/>
**Method:** GET<br/>

**Example response:**
```json
{
    "meta": {
        "status": 200,
        "message": "OK",
        "method": "GET",
        "href": "http://testserver/?ak1=861580a6-78cf-4c01-9e79-98f0835cdf69&ak2=1c6cd628-577e-4238-9a00-8e918b8850fe",
        "error_code": "",
        "error_description": "",
        "error_detail": null
    },
    "data": {
        "passenger": {
            "context_id": "294-2025-08-07-13ef5e3c-6252-40ca-b654-498ec04d3408",
            "airline_id": 294,
            "pax_record_locator": "2025-08-07-maKTy",
            "pax_record_locator_group": "2025-08-07-maKTy",
            "pnr_create_date": "2025-08-07",
            "flight_number": "NOFLT",
            "scheduled_depart": null,
            "disrupt_type": "weather",
            "first_name": "Jane",
            "last_name": "Doe",
            "ticket_level": "",
            "port_origin": "LAX",
            "port_arrival": "ABQ",
            "port_accommodation": "LAX",
            "hotel_accommodation": true,
            "meal_accommodation": false,
            "transport_accommodation": true,
            "create_date": "2025-08-07 19:00:11.404449",
            "expiration_date": "2025-08-08 12:00:00.000000",
            "modified_date": "2025-08-07 19:00:11.404449",
            "service_pet": false,
            "pet": false,
            "handicap": false,
            "notify": false,
            "disrupt_depart": "2025-08-07 00:00",
            "airline_pay": true,
            "number_of_nights": 1,
            "declined_date": null,
            "canceled_date": null,
            "hotel_accommodation_status": "offered",
            "meal_accommodation_status": "not_offered",
            "offer_opened_date": null,
            "ak1": "861580a678cf4c019e7998f0835cdf69",
            "ak2": "1c6cd628577e42389a008e918b8850fe",
            "offer_url": "http://passenger.stormx.test/offer?ak1=861580a678cf4c019e7998f0835cdf69&ak2=1c6cd628577e42389a008e918b8850fe",
            "notifications": [],
            "emails": [],
            "phone_numbers": [],
            "transport_accommodation_status": null,
            "hotel_allowance_status": {
                "breakfast": "not_offered",
                "lunch": "not_offered",
                "dinner": "not_offered",
                "amenity": "not_offered"
            }
        },
        "other_passengers": [],
        "confirmation": null,
        "branding": {
            "airline_id": 294,
            "airline_name": "Purple Rain Airlines",
            "airline_logo": "/static/airline/purple_rain/purple_rain.png",
            "logo_background_color": "#000000",
            "logo_foreground_color": "#ffffff"
        },
        "max_rooms": 1,
        "ride_vouchers": [
            {
                "allowed_products": {
                    "lyft": [
                        "lyft"
                    ]
                },
                "premium": false,
                "number_of_passengers": 1,
                "number_of_vehicles": 1,
                "number_of_days": 1,
                "rule": "available_when_no_shuttle",
                "ride_type": "airport_to_hotel_roundtrip",
                "origin": null,
                "destination": null,
                "surge_override": false,
                "create_date": "2025-08-07 19:00:11.411931",
                "canceled_date": null,
                "status": "offered",
                "id": "04655272-a12c-4839-8dca-6318a3ff8f25",
                "active_from": "2025-08-07 19:00",
                "active_to": "2025-08-09 03:59",
                "time_zone": "America/Los_Angeles",
                "provider": "lyft",
                "passenger": "294-2025-08-07-13ef5e3c-6252-40ca-b654-498ec04d3408",
                "allowed_ride_types": [
                    "hotel_to_airport"
                ],
                "rides": [
                    {
                        "id": "d9bcfc53-9927-40a4-95ef-cdb4330fcad2",
                        "provider": "lyft",
                        "product": "lyft",
                        "status": "accepted",
                        "create_date": "2025-08-07 19:00:11.492483",
                        "origin": {
                            "hotel_id": null,
                            "airport_iata_code": "LAX",
                            "latitude": null,
                            "longitude": null,
                            "name": "Los Angeles International Airport"
                        },
                        "destination": {
                            "hotel_id": null,
                            "airport_iata_code": "LAX",
                            "latitude": null,
                            "longitude": null,
                            "name": "Los Angeles International Airport"
                        },
                        "ride_type": "airport_to_hotel",
                        "cell_phone_number": ""
                    }
                ]
            }
        ]
    },
    "error": false
}
```
