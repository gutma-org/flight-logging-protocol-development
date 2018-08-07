# UAV/Operator Flight logging Exchange Protocol - Development

_A protocol designed to describe a logged flight from an UAV/Drone


## Read First

### Background
The [published](https://bitbucket.org/global_utm/flight-logging-protocol) "production" version of GUTMA protocol is a terrific start. However, it does not address a number of issues. These issues (detailed below) are complicated and dont have a clear answer at the moment. Therefore, this development version of the protocol was created to provide a platform to iterate, debate and test ideas to move the protocol foward as the requirements from drone operations and operators evolve.

- _Flexibility_: A common comment upon review of the production version of the protocol is that it is too flexble. In that, the protocol does not enforce any requirement on how logging should be done the data units and formats to be used. This has led to individual companies extending the protocol to suit their use case, which is to be expected and encouraged. Since there is no insturctions on how to log and what to log, we have a situation where two operators logging the same thing log in different fashion. This opens up a difficult set of questions around understanding of what is logged, who will make the interpretations etc. 
- _Standardization_: The production version of the protocol simply has one block to log any information the operator wants to keep track of. This is  problematic especially in a scenario where parts (but not all) of the log have to be shared with third parties (public or private). Therefore, this development version specifies splitting the log into 1) Standard logging 2) Extended / capability specific logging. The Standard logging is a GeoJSON FeatureCollection with additional properties. By making the distinction between standard logging (in GeoJSON) and extended logging as specified in the production version, we are ensuring that standard logs can be shared widely using a OGC standard and the extended logs can still be used. 
- _Public Vs Private data_: The production version protocol does not differentitate on the nature of the data. It is very difficult to split data once logged. "Standard log" and "Extended log" solves this problem. The Standard log is GeoJSON so at the very least a set of latitude longtiude is required and any other additional information that the operator may want to share publicly. Additionally, the "standard log" is a GeoJSON object which means that there are a number of existing tools that be used with it (e.g. [awesome-geojson](https://github.com/tmcw/awesome-geojson)). 
- _In flight or  post flight logging_: At the moment, there is no distinction between logging that is used for inflight telemetry and post flight logging. These need not be seperate but the "standard log" and "extended log" concept takes care of this. 
- _Compliance_: By adopting a minimal "standard logging" in GeoJSON, operators implementing this protocol can pre-empt any future legislation around standardization of protocols by adopting a widely accepted format of reporting. 

### Key Differences 

This protocol is backwards-compatible so entities using the production version protocol will not need to make any changes to their software as far as the published protocol is concerned. This protocol however, introduces a new component that is a additional output in GeoJSON. In addition to the Latitude / Longitude that is specified in GeoJSON, the standard log asks that additional attributes such as groundspeed, altitude and a timestamp be written in the properties. Any other information can be added as properties and is left to the operator. 

The conecpt of "Standard Log" is introduced as a GeoJSON and the log format introduced in the production version is kept as is and can be considered as a "Extended Log". 

## Data types
 
### Dates & Times

Dates and times will follow the ISO-8601 formatting standard. Local times are not supported; all times must be in UTC or have a time zone offset specified.

No support is made for just a date or just a time. For comparison of time ranges, the start time is regarded as included in the time range, where the end time is excluded.

The format pattern for date times is YYYY-MM-DDTHH:mm:ss.sssZ where Z is either the character Z to represent UTC, or the +/- timezone offset from UTC. If a message is received with no timezone offset, it should be regarded as an error and the correct error code returned.

The requirement to specify times or dates does not apply to the specification, so all temporal data will be defined as a datetime.

No guarantees that a timezone offset will be preserved should be made.


### Geospatial Data

Geospatial data must be described using a geometry object as defined in the GeoJSON specification with the following specific requirements

Using the default CRS - geographic coordinate reference system, using the WGS84 datum, and with longitude and latitude units of decimal degrees.

Bounding box is not expected.

Latitude and Longitude should not be specified to more than 8 decimal places. This gives an accuracy of approximately 1.1 mm at the equator making any further digits superfluous.


### Distances

All distances (both horizontal and vertical) are specified in metres.

### Speed

All speeds are specified in meters/seconds

## Message structure

The message is divided into three main parts: logging data as a GeoJSON FeatureCollection (__mandatory__). Description of devices used (__optional__),   and informations about the file itself (__optional__).

Here is a short message example:

		{
			"exchange": {
				"exchange_type": "flight_logging",
				"message": {
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
						"mission": "Projet test"
					},
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
					},

					"file": {
						"logging_type": "GUTMA_DX_JSON",
						"filename": "EB-99-01807_0069",
						"creation_dtg": "2017-05-23T08:38:41.306Z",
						"version": "1.0.0"
					},
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
					},
					"message_type": "flight_logging_submission"
				}
			}
		}

	
## Message Details

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
				"mission": "Projet test"
			}
    
This section  contains informations about hardware devices used during the flight:

- Aircraft section contains fields which a describing the aircraft used during the flight. An aircraft is uniquely identified with his manufacturer/model and serial number.
- gcs describes ground control station.
- Payload is used to declare what devices are embedded during the flight. There can be several payloads like gimbal, camera etc. 
- Misson is used to "tag" the flight or indicate if this flight is part of a larger mission.

### flight_logging\_geojson section
This part of the logging is mandatory and must be a valid GeoJSON object. The idea behind mandatory and optional flight logging is to ensure that mandatory logging is broadly compatible with a wide variety of tools using a well known reporting / log format. 

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

	
This is the main part of the file. The format is self describing i.e. it contains the types that are logged. It is must be a valid GeoJSON Point FeatureCollection with the each point having additional mandator properties of timestamp, gps\_altitude and groundspeed.

### flight\_logging\ section
This part of the logging encompasses any additional logging items that need to be logged and provides flexiblity to the logger to log additional items.

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

    "file":  {
    	"logging_type": "GUTMA_DX_JSON",
    	"filename": "EB-99-01807_0069",
    	"creation_dtg": "2017-05-23T08:38:41.306Z"
    }
            
This section contains informations about the file itself. All fields are optional.

* **logging_type**: type of logging
* **filename**: filename, can be with extension
* **creation_dtg**: file date creation
* **version**: protocol version


**LICENSE**

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 
 You may obtain a copy of the License at: http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
  
