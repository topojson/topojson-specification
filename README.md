# The TopoJSON Format Specification

Authors:
  * Mike Bostock (The New York Times)
  * Calvin Metcalf (Massachusetts Department of Transportation)

__Abstract__

TopoJSON is a topological geospatial data interchange format based on GeoJSON.

__Contents__

  * 1\. Introduction
    * 1.1. Examples
    * 1.2. Definitions
  * 2\. TopoJSON Objects
    * 2.1. Topology Objects
      * 2.1.1. Positions
      * 2.1.2. Arcs
      * 2.1.3. Objects
      * 2.1.4. Transform
    * 2.2. Geometry Objects
      * 2.2.1. Point
      * 2.2.2. MultiPoint
      * 2.2.3. LineString
      * 2.2.4. MultiLineString
      * 2.2.5. Polygon
      * 2.2.6. MultiPolygon
      * 2.2.7. Geometry Collection
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

All floating point numbers MUST be treated as double precision floats and all integers must be 32 bit signed integers.

TopoJSON geometries may be quantized, is it is then a quantization value must be selected, this value is the minimum difference along an axis between 2 points.

## 2. TopoJSON Objects

TopooJSON always consists of a single Topology object. This object (referred to as the Topology object below) represents a topology of a geometry, a feature, or a collection of features.

- The TopooJSON object may have any number of members (name/value pairs).
- The TopooJSON object must have a member with the name "type" and value 'Topology'.
- The TopoJSON object must have a member with the name "arcs", see 2.1.2.
- The TopoJSON object may have a member with the name "transform", see 2.1.4.
- The TopoJSON object must have a member with the name "objects", see 2.1,3.
- The TopoJSON object may have a "bbox" member, the value of which must be a bounding box array (see 3. Bounding Boxes).

### 2.1. Topology Objects

#### 2.1.1. Positions

A position is the fundamental geometry construct. The "coordinates" member of a Point object is composed of one position while the "coordinates" member or a  MultiPoint object and arc objects are composed of arrays of positions.

A position is represented by an array of numbers. There MUST be at least two elements, and MAY be more. All positions in the topology MUST have the same number of elements. The order of elements is beyond the scope of this specification.

If the topology is quantized (see 2.1.3) the array elements MUST be transformed integers, otherwise they may be integers or floats.

Positions are transformed from their raw position by subtracting each member of the 'transform' members 'translate' member from each member of the position. Each member of the result is divided by the 'scale' member of the 'translate' member.

#### 2.1.2. Arcs

The 'arcs' member of the TopoJSON object is an array consisting of one or more arrays each of which contains one or more positions.

If the topology is quantized (see 2.1.3) the array elements MUST be delta encoded. Instead of encoding the position they represent with the absolute coordinates of that position, the element represents the difference between it and the previous position.  The first position represents the difference between it and the origin.

#### 2.1.3 Objects

The 'objects' member of the topology is a JSON object, the values it contains are TopoJSON geometry objects (see 2.2).

#### 2.1.4 Transform

The 'transform' member of the TopoJSON object is a JSON object (transform object),

The transform object MUST contain a 'translate' member with a value of an array of length equal to the length of the positions, each member being the minimum value on that axis.

The transform object MUST contain a 'scale' member with a value of an array of length equal to the length of the positions. The scale values can be calculated as the inverse of the quantization value minus 1 divided by the range of the axis or 1/(q-1/max-min), unless this would result in an undefined mathematical operation or a value of 0, in which case the value is 1.

### 2.2. Geometry Objects

A TopoJSON geometry object is a JSON object which must have a member 'type' the type member's value is one of the following strings: "Point", "MultiPoint", "LineString", "MultiLineString", "Polygon", "MultiPolygon", or "GeometryCollection".

A TopoJSON geometry object of type other "Point" or "MultiPoint" must have a member with the name "coordinates".

A TopoJSON geometry object of type "LineString", "MultiLineString", "Polygon", or "MultiPolygon" must have a member with the name "arcs".

The value of the coordinates and arcs member is always an array. The structure for the elements in this array is determined by the type of geometry.

A TopoJSON geometry object MAY have the member 'id' which MUST be unique across the topology.

A TopoJSON geometry object may have a member with the name "properties". The value of the properties member is an object (any JSON object or a JSON null value).

#### 2.2.1 Point

For type "Point", it must contain a "coordinates" member which must be a single position.

#### 2.2.2. MultiPoint

For type "MultiPoint", it must contain a "coordinates" member which must be an array of position.

#### 2.2.3. LineString

For type "LineString", it must contain an "arcs" member which must contains an 'LinearArc', of integers. Each integer if positive is the value of the position of the corresponding arc in that array (0 index), if negative this denotes that the arc should be stitched backward and the index can be calculated by taking the ones' complement (binary inverse). The arcs are stitching together to form     the LinearArc, the last coordinate of the arc must be the same as the first coordinate of the subsequent arc, if any. The LinearArc MUST have at least 2 distinct positions.

An ArcRing is a closed LinearArc with 4 or more positions. The first and last positions are equivalent (they represent equivalent points). Though a ArcRing is not explicitly represented as a GeoJSON geometry type, it is referred to in the Polygon geometry type definition.

#### 2.2.4. MultiLineString

For type "MultiLineString", it must contain an "arcs" member which must contains an array of LinearArcs.

#### 2.2.5. Polygon

For type "Polygon", the "arcs" member must be an array of ArcRing coordinate arrays. For Polygons with multiple rings, the first must be the exterior ring and any others must be interior rings or holes.

#### 2.2.6. MultiPolygon

For type "MultiPolygon", the "arcs" member must be an array of Polygon arc arrays.

#### 2.2.7. Geometry Collection

A TopoJSON object with type "GeometryCollection" is a geometry object which represents a collection of geometry objects.

A geometry collection must have a member with the name "geometries". The value corresponding to "geometries" is an array. Each element in this array is a TopoJSON geometry object.

## 3. Bounding Boxes

To include information on the coordinate range for a topology a TopoJSON object may have a member named "bbox". The value of the bbox member must be a 2*n array where n is the number of dimensions represented in the contained geometries, with the lowest values for all axes followed by the highest values. The axes order of a bbox follows the axes order of geometries.
