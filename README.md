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
      * 2.1.2. Transforms
      * 2.1.3. Arcs
      * 2.1.4. Objects
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

TopooJSON always consists of a single topology object of type “Topology”. A topology may contain any number of named geometry objects. The term “TopoJSON object” may refer to either a topology or a geometry object it contains.

  * A TopooJSON object may have any number of members (name/value pairs).
  * A TopooJSON object must have a member with the name “type”. This member’s value is a string that determines the type of the GeoJSON object.
  * The value of the type member must be one of: “Topology”, “Point”, “MultiPoint”, “LineString”, “MultiLineString”, “Polygon”, “MultiPolygon” or “GeometryCollection”. The case of the type member values must be as shown here.
  * A TopoJSON object may have a ”bbox” member, the value of which must be a bounding box array.

### 2.1. Topology Objects

A topology is a TopoJSON object where the type member's value is “Topology”.

  * A topology must have a member with the name “type” whose value is “Topology”.
  * A topology must have a member with the name “objects” whose value is another object, each of which is a named geometry object.
  * A topology must have a member with the name “arcs” whose value is an array of arcs.
  * A topology may have a “transform” member, the value of which must be a transform.
  * A topology may have a ”bbox” member, the value of which must be a bounding box array.

#### 2.1.1. Positions

A position is represented by an array of numbers. There must be at least two elements, and may be more. The order of elements are recommended to follow _x_, _y_, _z_ order (easting, northing, altitude for coordinates in a projected coordinate reference system, or longitude, latitude, altitude for coordinates in a geographic coordinate reference system). Any number of additional elements are allowed — interpretation and meaning of additional elements is beyond the scope of this specification.

The “coordinates” member of a geometry object of type “Point” is composed of one position, and an array of positions for type “MultiPoint”. The “arcs” member of a topology is likewise an array of arrays of positions; however, note that positions must be delta-encoded for a topology with a “transform” member.

#### 2.1.2. Transform

A topology may have a “transform“ member, making it a quantized topology wherein all _x_ and _y_ position elements must be defined as integers.

  * A transform must have a member with the name “scale” whose value is a two-element array of numbers.
  * A transform must have a member with the name “translate” whose value is a two-element array of numbers.

Positions must be transformed by first multiplying by the transform’s scale and then adding the transform’s translate.

#### 2.1.3. Arcs

A topology must have an “arcs” member whose value is an array of arrays of positions. Each arc must be an array of two or more positions.

If the topology has a transform, the positions of each arc must be delta-encoded. The first position of the arc is a normal position [_x₁_, _y₁_]. The second position [_x₂_, _y₂_] is encoded as [_Δx₂_, _Δy₂_], where _x₂_ = _x₁_ + _Δx₂_ and _y₂_ = _y₁_ + _Δy₂_. The third position [_x₃_, _y₃_] is encoded as [_Δx₃_, _Δy₃_], where _x₃_ = _x₂_ + _Δx₃_ and _y₃_ = _y₂_ + _Δy₃_ and so on.

The following JavaScript reference implementation decodes a single arc from the given quantized topology:

```js
function decodeArc(topology, arc) {
  var x = 0,
      y = 0;
  return arc.map(function(point) {
    x += point[0];
    y += point[1];
    return [
      x * topology.transform.scale[0] + topology.transform.translate[0],
      y * topology.transform.scale[1] + topology.transform.translate[1]
    ];
  });
}
```

A geometry object consisting of LineStrings or LinearRings (LineString, MultiLineString, Polygon or MultiPolygon) must be constructed from arcs, referenced by index into the containing topology’s arcs array. A zero-based index is used: 0 refers to the first arc, 1 refers to the second arc, and so on. A negative index indicates that the referenced arc must be reversed to reconstruct the geometry. A negative arc index refers to the reverse of arc at the ones’ complement of the negative index: -1 refers to the reversed first arc, -2 refers to the reversed second arc, and so on.

If more than one arc is referenced to construct a LineString or LinearRing, the first position of a subsequent arc must be equal to the last position of the previous arc. Then, when reconstructing the geometry, the first position of each arc except the first may be dropped; equivalently, the last position of each arc except the last may be dropped.

#### 2.1.4. Objects

A topology must have an “objects” member whose value is an object. This object may have any number of members, whose value must be a geometry object.

### 2.2. Geometry Objects

A geometry is a TopoJSON object where the type member’s value is one of the following strings: “Point”, “MultiPoint”, “LineString”, “MultiLineString”, “Polygon”, “MultiPolygon”, or “GeometryCollection”.

A TopoJSON geometry object of type “Point” or “MultiPoint” must have a member with the name “coordinates”. A TopoJSON geometry object of type “LineString”, “MultiLineString”, “Polygon”, or “MultiPolygon” must have a member with the name “arcs”. The value of the arcs and coordinates members is always an array. The structure for the elements in this array is determined by the type of geometry.

If a geometry has a commonly used identifier, that identifier should be included as a member of the geometry object with the name “id”. A geometry object may have a member with the name “properties”. The value of the properties member is an object (any JSON object or a JSON null value).

#### 2.2.1. Point

For type “Point”, the “coordinates” member must be a single position.

#### 2.2.2. MultiPoint

For type “MultiPoint”, the “coordinates” member must be an array of positions.

#### 2.2.3. LineString

For type “LineString”, the “arcs” member must be an array of arc indexes.

#### 2.2.4. MultiLineString

For type “LineString”, the “arcs” member must be an array of LineString arc indexes.

#### 2.2.5. Polygon

For type “Polygon”, the “arcs” member must be an array of LinearRing arc indexes. For Polygons with multiple rings, the first must be the exterior ring and any others must be interior rings or holes.

#### 2.2.6. MultiPolygon

For type “MultiPolygon”, the “arcs” member must be an array of Polygon arc indexes.

#### 2.2.7. Geometry Collection

A TopoJSON object with type “GeometryCollection” is a geometry object which represents a collection of geometry objects.

A geometry collection must have a member with the name “geometries” whose value is an array. Each element in this array is a TopoJSON geometry object.

## 3. Bounding Boxes

To include information on the coordinate range for a topology or geometry a TopoJSON object may have a member named “bbox”. The value of the bbox member must be a 2*<i>n</i> array where _n_ is the number of dimensions represented in the contained geometries, with the lowest values for all axes followed by the highest values. The axes order of a bbox follows the axes order of geometries. The bounding box should not be transformed using the topology’s transform, if any.
