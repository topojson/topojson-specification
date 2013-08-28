# Draft TopoJSON Specification

All floating point numbers are double precision floats and all integers are 32 bit signed integers. 

TopoJSON is quantized, arc values transformed into integers. the quantization value is the minimum difference along an axis between 2 points, it is defined during creation and not needed to decode. 

A TopoJSON document consists of a JSON object including the following members:

1. It MUST have a member 'type' with a value of the string "Topology"
2. It MAY have a member 'bbox' with a value of an array of length 4 containing, in order, 
   the lowest x-axis coordinate, the lowest y-coordinate, the highest x-axis coordinate,
   and the highest y-axis coordinate in the original coordinate system of the data. 
3. It MUST have a member 'transform' with a value of an object, they object has 2 members:
    1. 'translate' with a value of an array of length 2 containing the lowest x-axis coordinate 
        and the lowest y-axis coordinate. 
    2. 'scale' with a value of an array of length 2 containing the x-axis scale and the y-axis
       scale. The scale values can be calculated as the inverse of the quantization value minus 1
       divided by the range of the axis or 1/(q-1/max-min) unless max and min are the same in which case it is 1.
4.  It MUST have a member 'arcs' the value of which is an array of 0 or more arc objects. Arc objects
    are an array of arrays of length 2, which contain 2 delta encoded values, x and y.
    Each delta encoded value array is the difference along the x and y coordinate as an integer
    from the previous value.  The first value can be assumed to be preceded by the value [0,0]
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
