# UAV/Operator Flight logging Exchange Protocol - Development

A protocol designed to describe a logged flight from an UAV/Drone

- [UAV/Operator Flight logging Exchange Protocol - Development](#uavoperator-flight-logging-exchange-protocol---development)
	- [Read First](#read-first)
		- [Key Differences](#key-differences)
		- [Background](#background)
	- [Data types](#data-types)
		- [Dates & Times](#dates--times)
		- [Geospatial Data](#geospatial-data)
		- [Distances](#distances)
		- [Speed](#speed)
	- [Message structure](#message-structure)
	- [Message Details](#message-details)
		- [exchange_type section](#exchangetype-section)
		- [flight_data section](#flightdata-section)
		- [flight_logging\_geojson section](#flightlogginggeojson-section)
		- [flight\_logging section](#flightlogging-section)
		- [file section](#file-section)
		- [message_type section](#messagetype-section)

## Read First

### Key Differences 

This protocol is backwards-compatible so entities using the production version protocol will not need to make any changes to their software. This protocol however, introduces a new component that is a additional output in GeoJSON. This GeoJSON component is called "Standard Log" and the original log format introduced in the production version is kept as is called "Extended Log". In addition to the Latitude / Longitude that is specified in GeoJSON, the standard log asks that additional attributes such as ground speed, altitude and a timestamp be written in the properties. Any other information can be added as properties and is left to the operator. 

### Background

The [published](https://github.com/gutma-org/flight-logging-protocol) "production" version of GUTMA protocol is a terrific start. However, there are some open questions still. These issues (detailed below) are complicated and dont have a clear answer at the moment. Therefore, this development version of the protocol was created to provide a platform to iterate, debate and test ideas to move the protocol forward as the requirements from drone operations and operators evolve.

1. _Flexibility_: A common comment upon review of the production version of the protocol is that it is too flexible. In that, the protocol does not enforce any requirement on how logging should be done the data units and formats to be used. This has led to individual companies extending the protocol to suit their use case, which is to be expected and encouraged. Since there is no instructions on how to log and what to log, we have a situation where two operators logging the same data log in different formats and fashion. This opens up a difficult set of questions around understanding of what is logged, who will make the interpretations, interoperability etc. 
2. _Use Case_ : Flight logging is actually two different set of things: 1) In-flight live / realtime telemetry and 2) Post-flight logging. These are two different use cases. Realtime telemetry might be used during operations by the operator but post-flight logging could be used to share the flight data externally. These are different use cases and the production protocol does not handle it well. 
3. _Standardization_: The production version of the protocol simply has one block to log any information the operator wants to keep track of. This is  problematic especially in a scenario where parts (but not all) of the log have to be shared with third parties (public or private). Therefore, this development version specifies splitting the log into 1) Standard logging 2) Extended / capability specific logging. The Standard logging is a GeoJSON FeatureCollection with additional properties. By making the distinction between standard logging (in GeoJSON) and extended logging as specified in the production version, we are ensuring that standard logs can be shared widely using a OGC standard and the extended logs can still be used. The goal of the the standard log is to be used for "post-flight" sharing while the extended log used for in-flight telemetry. 
4. _Public Vs Private data_: The production version protocol does not differentiate on the nature of the data. It is very difficult to split data once logged. "Standard log" and "Extended log" solves this problem. The Standard log is GeoJSON so at the very least a set of latitude longtiude is required and any other additional information that the operator may want to share publicly. Additionally, the "standard log" is a GeoJSON object which means that there are a number of existing tools that be used with it (e.g. [awesome-geojson](https://github.com/tmcw/awesome-geojson)). 
5. _Partial and Batch logging_: "Standard log" and "Extended log" is compatible with partial and batch logging, in case of partial or batch logging, as long as the GeoJSON object is valid, the protocol does not mandate what is to be logged so a series of points can be logged between a time duration. 
6. _Compliance_: By adopting a minimal "standard logging" in GeoJSON, operators implementing this protocol can pre-empt any future legislation around standardization of protocols by adopting a widely accepted format of reporting. 


## Data types
 
### Dates & Times

Dates and times will follow the [ISO-8601](https://www.iso.org/iso-8601-date-and-time-format.html) formatting standard. Local times are not supported; all times must be in UTC or have a time zone offset specified.

No support is made for just a date or just a time. For comparison of time ranges, the start time is regarded as included in the time range, where the end time is excluded.

The format pattern for date times is YYYY-MM-DDTHH:mm:ss.sssZ where Z is either the character Z to represent UTC, or the +/- timezone offset from UTC. If a message is received with no timezone offset, it should be regarded as an error and the correct error code returned.

The requirement to specify times or dates does not apply to the specification, so all temporal data will be defined as a datetime.

No guarantees that a timezone offset will be preserved should be made.


### Geospatial Data

Geospatial data must be described using a geometry object as defined in the GeoJSON specification with the following specific requirements:

1. Using the default CRS - geographic coordinate reference system, using the WGS84 datum, and with longitude and latitude units of decimal degrees.

2. Bounding box is not expected.

3. Latitude and Longitude should not be specified to more than 8 decimal places. This gives an accuracy of approximately 1.1 mm at the equator making any further digits superfluous.


### Distances

All distances (both horizontal and vertical) are specified in metres.

### Speed

All speeds are specified in meters/seconds

## Message structure

The message is divided into three main parts: logging data as a GeoJSON FeatureCollection (_called Standard log and is mandatory_). Description of devices used (_called Extended log and is optional_), and information about the file itself (_optional_).

Here is a short message example: [Example Flight Log](https://github.com/hrishiballal/flight-logging-protocol-development/blob/master/GUTMA_flight_log_example_v1.json)


## Message Details
The message above has three sections: 
1. Exchange Type (exchange_type)
2. Flight Data Section (flight_data)
3. Standard Flight Logging (flight_logging_geojson)
4. Extended Flight Logging (flight_logging)
5. File Section (file)
6. Message Type Section (message_type)


### exchange_type section
This section details the type of message in the exchange. 

	"message_type": "flight_logging"
            
* **message_type**: type of logging that the exchange contains. At the moment only allowed type is flight_logging
   
We cover each of these sections in detail below. 

### flight_data section

	"flight_data": {
		"aircraft": {
			"firmware_version": "2.01b",
			"hardware_version": "1.00B",
			"manufacturer": "senseFly",
			"model": "eBee",
			"name": "John doe Drone",
			"serial_number": "EB-99-01807"
		},
		"gcs": {
			"manufacturer": "senseFly",
			"model": "eMotion",
			"version": "1.1"
		},
		"payload": [{
			"firmware_version": "1.23",
			"hardware_version": "0",
			"model": "WX RGB",
			"serial_number": "2352342141"
		}],
		"mission": "Project test"
	}

This section  contains information about hardware devices used during the flight:

- Aircraft section contains fields which a describing the aircraft used during the flight. An aircraft is uniquely identified with his manufacturer/model and serial number.
- gcs describes ground control station.
- Payload is used to declare what devices are embedded during the flight. There can be several payloads like gimbal, camera etc. 
- Mission is used to "tag" the flight or indicate if this flight is part of a larger mission.

### flight_logging\_geojson section
This part of the logging is called "Standard Logging" and is mandatory and must be a valid GeoJSON object. It could be a full flight log or partial one. 

	"flight_logging_geojson": {
		"flight_path": {
			"type": "FeatureCollection",
			"features": [{
					"type": "Feature",
					"properties": {
						"time": "2017-05-16T13:19:22.250Z",
						"altitude": 100,
						"groundspeed": 20,
						"event_type": "CONTROLER_EVENT",
						"event_info": "TAKE-OFF"
					},
					"geometry": {
						"type": "Point",
						"coordinates": [-6.201825141906738,
							53.50841695106615
						]
					}
				},
				{
					"type": "Feature",
					"properties": {
						"time": "2017-05-16T13:19:23.250Z",
						"altitude": 120,
						"groundspeed": 20
					},
					"geometry": {
						"type": "Point",
						"coordinates": [-6.199507713317871,
							53.509246406536995
						]
					}
				},
				{
					"type": "Feature",
					"properties": {
						"time": "2017-05-16T13:19:25.250Z",
						"altitude": 120,
						"groundspeed": 20,
						"event_type": "CONTROLER_EVENT",
						"event_info": "LANDING"
					},
					"geometry": {
						"type": "Point",
						"coordinates": [-6.20075225830078,
							53.50916984209669
						]
					}
				}
			]
		},
		"altitude_system": "WGS84",
		"logging_start_dtg": "2017-05-16T13:19:25.250Z",
		"tick_increment_modulo": "1",
		"tick_increment_seconds": "1",
		"uom_system": "Metric"
	}


 The format is self describing i.e. it contains the geometry types that are logged. It is must be a valid GeoJSON FeatureCollection with the each geometry having additional mandatory properties of timestamp, gps\_altitude and groundspeed. In case of declaring in linestrings, each co-ordinate should be accompanied by timestamps. 

### flight\_logging section
This part of the logging encompasses any additional logging items that need to be logged and provides flexibility to the logger. This is the "Extended log" 

	"flight_logging": {
		"flight_logging_items": [
			[0.5, 0, 0, 0],
			[1, 0, 0, 0],
			[1.5, 0, 0, 0]
		],
		"flight_logging_keys": [
			"timestamp", "speed_vx", "speed_vy", "battery_voltage"
		],
		"events": [{
			"event_type": "CONTROLER_EVENT",
			"event_info": "TAKE-OFF",
			"event_timestamp": "0.5"
		}],
		"altitude_system": "WGS84",
		"logging_start_dtg": "2017-05-16T13:19:25.250Z"
	}

* **Timestamp** : number of the seconds elapsed since logging_start_dtg. It is a float, with max 3 decimals (so precision is milliseconds). By extension, the last timestamp will be equal to the duration flight in seconds.
* **gps\_lon, gps\_lat, gps\_altitude**: GPS coordinates
*  **speed**: ground speed in m/s (float)
* **battery\_voltage**: voltage in volt (float)
* **logging\_start\_dtg**: describes the beginning of the flight. It is mandatory. 
* **altitude\_system**: indicates the type of altitude reported: "agl", "amsl", "sps" or "WGS84"
* **events**: The events propoerty is used to log key events in the flight: "start-up-request", "start-up", "take-off", "en-route", "landing", "emergency", "obstacle-avoidance"

### file section
This section contains information about the file itself. All fields are optional.

    "file":  {
    	"logging_type": "GUTMA_DX_JSON",
    	"filename": "EB-99-01807_0069",
    	"creation_dtg": "2017-05-23T08:38:41.306Z"
    }
            
* **logging_type**: type of logging
* **filename**: filename, can be with extension
* **creation_dtg**: file date creation
* **version**: protocol version

### message_type section
This section details the type of message in the exchange. 

	"message_type": "flight_logging_submission"
            
* **message_type**: type of logging that the exchange contains. At the moment only allowed type is flight_logging_submission