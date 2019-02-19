# Bidirectional conversion between JSON(-LD) and iRODS AVUs

## Rationale

JSON is a flexible and easy to use format for storing (nested) data. At the same time 
it can remain human readable. It can therefor be an ideal method for 
storing metadata in iRODS. However, iRODS uses Attribute, Value, Unit triples. Its 
largest drawback being the lack of nesting. 

This script describes a method for converting JSON to AVU triples and back again 
(bidirectional).

## Design goals

* Bijection between JSON <-> AVU
  * i.e no limit on the characters used in an attribute
  * i.e being able to maintain order in arrays
* Lean JSON -> AVU conversion. 
  * Don't explode the JSON unnecessarily in AVUs
* Keep Attribute->Value pairs the same in JSON and AVUs. So values remain easily accessible from within iRODS
* Compatible with existing or additional AVUs 
* Compatible/aware of JSON-LD


## Implementation
Took ideas from RDF.

## How to run

Compatible with Python 2 and 3.

```bash
python conversion.py inputs/basic.json
```

## Example output
```
Source:
{
    "k2": {
        "k3": "v2", 
        "k4": "v3"
    }, 
    "k1": "v1", 
    "k6": [
        {
            "k8": "v7", 
            "k7": "v6"
        }
    ], 
    "k5": [
        "v4", 
        "v5"
    ]
}
AVUs:
      A        V          U
      P        O          S
     k2      _b0       root
     k3       v2        _b0
     k4       v3        _b0
     k1       v1       root
     k6      _b1     root#0
     k8       v7        _b1
     k7       v6        _b1
     k5       v4     root#0
     k5       v5     root#1
JSON:
{
    "k2": {
        "k3": "v2", 
        "k4": "v3"
    }, 
    "k1": "v1", 
    "k6": [
        {
            "k8": "v7", 
            "k7": "v6"
        }
    ], 
    "k5": [
        "v4", 
        "v5"
    ]
}
```

## Limits/bugs

On the AVU side
* If two AVUs have the same attribute but different values only the last one ends up in the JSON (possible in AVUs!)
* AVUs already containing data in the unit column will be ignored (possibly a good thing)
* Nested arrays, can be stored, but will not be converted from AVUs to JSON (fixable)

On the JSON side
* Empty objects will become blank nodes, but not converted back to empty objects
* Empty key/value pairs cannot be converted, as iRODS does not allow an AVU with an empty value
* Other JSON types: integers, booleans and null

On the JSON-LD side
* Not handling "@context" properly yet