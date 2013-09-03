# The TopoJSON Format Specification

Authors:
  * Mike Bostock (The New York Times)
  * Calvin Metcalf

__Abstract__

TopoJSON is a topological geospatial data interchange format based on GeoJSON.

__Contents__

  * 1\. Introduction
    * 1.1. Examples
    * 1.2. Definitions
  * 2\. TopoJSON Objects
    * 2.1. Topology Objects
      * 2.1.1. Arcs
    * 2.2. Geometry Objects
      * 2.2.1. Positions
      * 2.2.2. Point
      * 2.2.3. MultiPoint
      * 2.2.4. LineString
      * 2.2.5. MultiLineString
      * 2.2.6. Polygon
      * 2.2.7. MultiPolygon
      * 2.2.8. Geometry Collection
  * 3\. Bounding Boxes

## 1. Introduction

TopoJSON is a [JSON](http://json.org/) format for encoding geographic data structures into a shared topology. A TopoJSON topology represents one or geometries that share sequences of positions called _arcs_. TopoJSON, as an extension of [GeoJSON](http://geojson.org/), supports multiple geometry types: Point, LineString, Polygon, MultiPoint, MultiLineString, MultiPolygon, and GeometryCollection. Geometries in TopoJSON may contain additional properties to encode non-geometrical data.

### 1.1. Examples

A TopoJSON topology containing a single object named “example”, itself a geometry collection:

```json
{
  "type": "Topology",
  "objects": {
    "example": {
      "type": "GeometryCollection",
      "geometries": [
        {
          "type": "Point",
          "properties": {
            "prop0": "value0"
          },
          "coordinates": [102, 0.5]
        },
        {
          "type": "LineString",
          "properties": {
            "prop0": "value0",
            "prop1": 0
          },
          "arcs": [0]
        },
        {
          "type": "Polygon",
          "properties": {
            "prop0": "value0",
            "prop1": {
              "this": "that"
            }
          },
          "arcs": [[-2]]
        }
      ]
    }
  },
  "arcs": [
    [[102, 0], [103, 1], [104, 0], [105, 1]],
    [[100, 0], [101, 0], [101, 1], [100, 1], [100, 0]]
  ]
}
```

The same topology, quantized with Q = 10,000:

```json
{
  "type": "Topology",
  "transform": {
    "scale": [0.0005000500050005, 0.00010001000100010001],
    "translate": [100, 0]
  },
  "objects": {
    "example": {
      "type": "GeometryCollection",
      "geometries": [
        {
          "type": "Point",
          "properties": {
            "prop0": "value0"
          },
          "coordinates": [4000, 5000]
        },
        {
          "type": "LineString",
          "properties": {
            "prop0": "value0",
            "prop1": 0
          },
          "arcs": [0]
        },
        {
          "type": "Polygon",
          "properties": {
            "prop0": "value0",
            "prop1": {
              "this": "that"
            }
          },
          "arcs": [[1]]
        }
      ]
    }
  },
  "arcs": [
    [[4000, 0], [1999, 9999], [2000, -9999], [2000, 9999]],
    [[0, 0], [0, 9999], [2000, 0], [0, -9999], [-2000, 0]]
  ]
}
```

### 1.2. Definitions

JavaScript Object Notation (JSON), and the terms “object”, “name”, “value”, “array”, and “number”, are defined in [IETF RTC 4627](http://www.ietf.org/rfc/rfc4627.txt).

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [IETF RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).


### …

All floating point numbers are double precision floats and all integers are 32 bit signed integers.

TopoJSON is quantized, arc values transformed into integers. the quantization value is the minimum difference along an axis between 2 points, it is defined during creation and not needed to decode.

A TopoJSON document consists of a JSON object including the following members:

1. It MUST have a member 'type' with a value of the string "Topology"
2. It MAY have a member 'bbox' with a value of an array of length 4 containing, in order,
   the lowest x-axis coordinate, the lowest y-coordinate, the highest x-axis coordinate,
   and the highest y-axis coordinate in the original coordinate system of the data.
3. It MAY have a 'transform' member with a value of an object:
    1. the transform member MUST contain a 'translate' member with a value of an array of length 2 containing the lowest x-axis coordinate
        and the lowest y-axis coordinate.
    2. the transform member MUST contain a 'scale' member with a value of an array of length 2 containing the x-axis scale and the y-axis
       scale. The scale values can be calculated as the inverse of the quantization value minus 1
       divided by the range of the axis or 1/(q-1/max-min) unless max and min are the same in which case it is 1.
4.  It MUST have a member 'arcs' the value of which is an array of 0 or more arc objects. Arc objects
    are an array of arrays of length 2, which contain 2 values, x and y.

    If the 'transform' member is present then each value array is the difference along the x and y coordinate
    as an integer from the previous value transformed according to the transorm property.
    The first value can be assumed to be preceded by the value [0,0].

    If no 'transform' member is present then each point is the absolute values unmodified.
5. It MUST have a member 'objects' with its value a geojson geometry object.  The member names of this
   object may be any valid json key value, the value is a geojson geometry object modified as followed:
    1. The object MAY have an 'id' member
    2. The object MAY have a 'properties' member
    2. Types 'Point' and 'MultiPoint' coordinates member MUST have transformed integer values.
    3. Types 'LineString', 'MultiLineString', 'Polygon', or 'MultiPolygon' then it contains an 'arcs'
       member whose value is the same as the respective geojson geometry type, but the LinearRing
       object is replaced by an array of integers, these correspond to indexes in in the root 'arcs' object.
       Each integer if positive is the value of the position of the corresponding arc in that array
       (0 index), if negative this denotes that the arc should be stitched backward and the index can be
       calculated by taking the ones' complement (binary inverse). When stitching together arcs to form
       geometries, the last coordinate of the arc must be the same as the first coordinate of the subsequent arc, if any.
