layout qualifier is a type that allows to provide metadata or additional information on how a particular variable or input should be processed, such as its [[binding]] point, location, or other implementation-specific attributes
which allows for precise control over resource placement

layout types
std140 
- have stricter alignment requirements
- each array element is rounded up to vec4 (16 bytes),
- significant padding between elements
std430
- tighter packing
- elements aligned only to their natural size
- minimizes padding between elements