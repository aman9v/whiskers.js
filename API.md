# Eclipse Whiskers API

This document will explain the classes and methods in the Eclipse Whiskers client library.

**This is a very early version of the API. It may change considerably before being stabilized.** If you are using an early version it is a very good idea to lock to a specific version of Whiskers — a loose version constraint could cause you to unintentionally upgrade to an incompatible version of the API. This is not a good experience and we plan to have a stable API at version 1.0 that does not introduce breaking changes.

## Models

There are currently 8 models in the client library.

* Backend
* Datastream
* Generic
* Location
* Observation
* ObservedProperty
* Sensor
* Thing

### Backend

This class represents a connection to a remote instance of SensorThings. Multiple instances of this class could be used to communicate with multiple SensorThings servers.

#### `new Backend(url)`

Creating a new Backend is done with the URL of the instance.

````javascript
    var remote = new Backend('http://example.com/OGCSensorThings/v1.0/');
````

#### `Backend.getLinks()`

Returns an object containing key-value pairs for the SensorThings collections on the remote host. Will execute a GET request on the first call, and use a cached value on subsequent calls. Returns a Q promise with the links object or the links object if already cached.

````javascript
    {
      "Things": "http://example.com/OGCSensorThings/v1.0/Things",
      "Locations": "http://example.com/OGCSensorThings/v1.0/Locations",
      "HistoricalLocations": "http://example.com/OGCSensorThings/v1.0/HistoricalLocations",
      "Datastreams": "http://example.com/OGCSensorThings/v1.0/Datastreams",
      "Sensors": "http://example.com/OGCSensorThings/v1.0/Sensors",
      "Observations": "http://example.com/OGCSensorThings/v1.0/Observations",
      "ObservedProperties": "http://example.com/OGCSensorThings/v1.0/ObservedProperties",
      "FeaturesOfInterest": "http://example.com/OGCSensorThings/v1.0/FeaturesOfInterest"
    }

````

This is used internally in the Backend class for accessing collections without having the paths hard-coded.

#### `Backend.getRoot(options)`

Retrieves the response body from the SensorThings root collection path. Returns a Q promise with the JQuery jqXHR object.

Use `options` to pass in an object that is compatible with [JQuery's ajax settings object](http://api.jquery.com/jquery.ajax/#jQuery-ajax-settings). The `options` parameter can be safely omitted.

#### `Backend.getThing(id, options)`

Retrieves the Thing resource from SensorThings with the corresponding `id` and returns a Thing instance. Returns a Q promise with the Thing object.

Use `options` to pass in an object that is compatible with [JQuery's ajax settings object](http://api.jquery.com/jquery.ajax/#jQuery-ajax-settings). The `options` parameter can be safely omitted.

#### `Backend.getThings(options)`

Retrieves the Things collection resource from SensorThings and returns an array of Thing instances. Returns a Q promise with the array of Thing objects.

Use `options` to pass in an object that is compatible with [JQuery's ajax settings object](http://api.jquery.com/jquery.ajax/#jQuery-ajax-settings). The `options` parameter can be safely omitted.

If the remote SensorThings server applies a limit to the response then only that many entities will be returned in the array.

### Datastream

This class represents the Datastream model in the SensorThings specification (section 8.3.4). A Datastream is a group of observations that have the same Observed Property, same Sensor, and same Thing.

#### `new Datastream(data)`

Create a new Datastream with an object containing the attribute keys of a SensorThings Datastream entity.

````javascript
    var stream = new Datastream({
      "@iot.id": 1,
      "@iot.selfLink": "http://example.com/OGCSensorThings/v1.0/Datastreams(1)",
      "Thing@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Things(1)",
      "Sensor@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Sensors(1)",
      "ObservedProperty@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/ObservedProperties(1)",
      "Observations@iot.navigationLink": "http://example.com/OGCSensorThings/v1.0/Observations(1)",
      "description": "This is a datastream measuring the temperature in an oven.",
      "unitOfMeasurement": {
        "name": "degree Celsius",
        "symbol": "°C",
        "definition": "http://unitsofmeasure.org/ucum.html#para-30"
      },
      "observationType": "http://www.opengis.net/def/observationType/OGC- OM/2.0/OM_Measurement",
      "observedArea": {
        "type": "Polygon",
        "coordinates": [[[100,0],[101,0],[101,1],[100,1],[100,0]]]
      },
      "phenomenonTime": "2014-03-01T13:00:00Z/2015-05-11T15:30:00Z",
      "resultTime": "2014-03-01T13:00:00Z/2015-05-11T15:30:00Z"
    });
````

There is currently no validation of the attributes added in this contructor, so it is possible to add invalid SensorThings attributes to a Datastream instance.

This **does not** create an instance on the remote server, only a local object.

#### `Datastream.get()`/`Datastream.set()`

As Datastream is a Generic-type model class, it has the `get` and `set` methods for accessing the entity attributes. Entity attributes are **separate** from JavaScript object properties and are stored in the `Datastream.attributes` object.

To retrieve an attribute, call `get()` with the key:

````javascript
    var selfLink = stream.get("@iot.selfLink");
````

To create or update an attribute, call `set()` with the key and value:

````javascript
    stream.set("result", 0);
````

#### `Datasteam.getAllObservations(options)`

Iteratively retrieve *all* the observations for a Datastream from the remote server. Uses the `Observations@iot.navigationLink` attribute to determine fetch URL. Will retrieve the first set of Observations, and if `@iot.nextLink` is defined in the response, will follow that link for more Observations. This will continue until `@iot.nextLink` is not defined in a response, then an array of all the Observations collected will be returned in a promise object.

Use `options` to pass in an object that is compatible with [JQuery's ajax settings object](http://api.jquery.com/jquery.ajax/#jQuery-ajax-settings). The `options` parameter can be safely omitted.

````javascript
    Q(stream.getAllObservations())
    .then(function(observations) {
      // do something with observations
    })
    .done();
````

This method does not return until all Observations have been collected, which may take a noticeable amount of time to load for the user.

#### `Datasteam.getObservations(options)`

Retrieve the observations for a Datastream from the remote server. If the remote server has pagination, then only the initial set will be returned. Will return a Q promise object, which will resolve with an array of Observation objects.

Use `options` to pass in an object that is compatible with [JQuery's ajax settings object](http://api.jquery.com/jquery.ajax/#jQuery-ajax-settings). The `options` parameter can be safely omitted. For example, you can pass in SensorThings `$filter`, `$orderby`, or `$skip` parameters:

````javascript
    // Retrieve observations in this stream that have a phenomenonTime between May 1 and June 1 2016 (UTC zone).
    Q(stream.getObservations({
      data: {
        "$filter": "phenomenonTime ge 2016-05-01T00:00:00Z and phenomenonTime le 2016-06-01T00:00:00Z",
        "$orderby": "phenomenonTime desc"
    }
    }))
    .then(function(observations) {
      // do something with observations
    })
    .done();
````

### Generic

This class is a base class for SensorThing entity classes. It only implements a getter/setter for the entity's attributes, and will set the attributes on creation. This class isn't likely to be used often directly and should instead be subclassed as per [ECMAScript 6](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes).

#### `new Generic(data)`

Creates a new object with the object's `attributes` property as an object equal to `data`.

#### `Generic.get()`/`Generic.set()`

Get and Set are methods for accessing the attributes object. Entity attributes are **separate** from JavaScript object properties and are stored in the `Generic.attributes` object.

To retrieve an attribute, call `get()` with the key:

````javascript
    var selfLink = object.get("@iot.selfLink");
````

To create or update an attribute, call `set()` with the key and value:

````javascript
    object.set("property", 100);
````

### Location

This class represents the Location model in the SensorThings specification (section 8.3.2). A Location is a geographic entity that is used to geo-reference one or more Thing entities.

#### `new Location(data)`

Create a new Location with an object containing the attribute keys of a SensorThings Location entity.

````javascript
    var location = new Location({
      "@iot.id": 1,
      "@iot.selfLink": "http://example.org/v1.0/Locations(1)",
      "Things@iot.navigationLink": "Locations(1)/Things",
      "HistoricalLocations@iot.navigationLink": "Locations(1)/HistoricalLocations",
      "encodingType": "application/vnd.geo+json",
      "location": {
        "type": "Point",
        "coordinates": [-114.06, 51.05]
      }
    });
````

There is currently no validation of the attributes added in this contructor, so it is possible to add invalid SensorThings attributes to a Location instance.

This **does not** create an instance on the remote server, only a local object.

#### `Location.get()`/`Location.set()`

As Location is a Generic-type model class, it has the `get` and `set` methods for accessing the entity attributes. Entity attributes are **separate** from JavaScript object properties and are stored in the `Location.attributes` object.

To retrieve an attribute, call `get()` with the key:

````javascript
    var selfLink = location.get("@iot.selfLink");
````

To create or update an attribute, call `set()` with the key and value:

````javascript
    location.set("location", geoJSONObject);
````

### Observation

This class represents the Observation model in the SensorThings specification (section 8.3.7). An Observation is an act of measuring or otherwise determining the value of a property (Observed Property).

#### `new Observation(data)`

Create a new Observation with an object containing the attribute keys of a SensorThings Observation entity.

````javascript
    var observation = new Observation({
      "@iot.id": 1,
      "@iot.selfLink": "http://example.org/v1.0/Observations(1)",
      "FeatureOfInterest@iot.navigationLink": "Observations(1)/FeatureOfInterest", "Datastream@iot.navigationLink":"Observations(1)/Datastream",
      "phenomenonTime": "2014-12-31T11:59:59.00+08:00",
      "resultTime": "2014-12-31T11:59:59.00+08:00",
      "result": 70.4
    });
````

There is currently no validation of the attributes added in this contructor, so it is possible to add invalid SensorThings attributes to an Observation instance.

This **does not** create an instance on the remote server, only a local object.

#### `Observation.get()`/`Observation.set()`

As Observation is a Generic-type model class, it has the `get` and `set` methods for accessing the entity attributes. Entity attributes are **separate** from JavaScript object properties and are stored in the `Observation.attributes` object.

To retrieve an attribute, call `get()` with the key:

````javascript
    var selfLink = observation.get("@iot.selfLink");
````

To create or update an attribute, call `set()` with the key and value:

````javascript
    observation.set("result", 0);
````

### ObservedProperty

This class represents the ObservedProperty model in the SensorThings specification (section 8.3.6). An ObservedProperty specifies the phenomenon of an Observation.

#### `new ObservedProperty(data)`

Create a new ObservedProperty with an object containing the attribute keys of a SensorThings ObservedProperty entity.

````javascript
    var observedProperty = new ObservedProperty({
      "@iot.id": 1,
      "@iot.selfLink": "http://example.org/v1.0/ObservedProperties(1)",
      "Datastreams@iot.navigationLink": "ObservedProperties(1)/Datastreams",
      "description": "The dewpoint temperature is the temperature to which the air must be cooled, at constant pressure, for dew to form. As the grass and other objects near the ground cool to the dewpoint, some of the water vapor in the atmosphere condenses into liquid water on the objects.",
      "name": "DewPoint Temperature",
      "definition": "http://dbpedia.org/page/Dew_point"
    });
````

There is currently no validation of the attributes added in this contructor, so it is possible to add invalid SensorThings attributes to an ObservedProperty instance.

This **does not** create an instance on the remote server, only a local object.

#### `ObservedProperty.get()`/`ObservedProperty.set()`

As ObservedProperty is a Generic-type model class, it has the `get` and `set` methods for accessing the entity attributes. Entity attributes are **separate** from JavaScript object properties and are stored in the `ObservedProperty.attributes` object.

To retrieve an attribute, call `get()` with the key:

````javascript
    var selfLink = observedProperty.get("@iot.selfLink");
````

To create or update an attribute, call `set()` with the key and value:

````javascript
    observedProperty.set("description", "Pedestrian foot traffic");
````

### Sensor

This class represents the Sensor model in the SensorThings specification (section 8.3.5). A Sensor is an instrument that observes a property or phenomenon with the goal of producing an estimate of the value of the property.

#### `new Sensor(data)`

Create a new Sensor with an object containing the attribute keys of a SensorThings Sensor entity.

````javascript
    var sensor = new Sensor({
      "@iot.id": 1,
      "@iot.selfLink": "http://example.org/v1.0/Sensors(1)",
      "Datastreams@iot.navigationLink": "Sensors(1)/Datastreams",
      "description": "TMP36 - Analog Temperature sensor",
      "encodingType": "application/pdf",
      "metadata": "http://example.org/TMP35_36_37.pdf"
    });
````

There is currently no validation of the attributes added in this contructor, so it is possible to add invalid SensorThings attributes to a Sensor instance.

This **does not** create an instance on the remote server, only a local object.

#### `Sensor.get()`/`Sensor.set()`

As Sensor is a Generic-type model class, it has the `get` and `set` methods for accessing the entity attributes. Entity attributes are **separate** from JavaScript object properties and are stored in the `Sensor.attributes` object.

To retrieve an attribute, call `get()` with the key:

````javascript
    var selfLink = sensor.get("@iot.selfLink");
````

To create or update an attribute, call `set()` with the key and value:

````javascript
    sensor.set("description", "Generic Webcam 3C Model");
````
