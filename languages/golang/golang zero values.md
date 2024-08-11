1. Numeric types:
   - int, int8, int16, int32, int64: 0
   - uint, uint8, uint16, uint32, uint64: 0
   - float32, float64: 0.0
   - complex64, complex128: 0+0i
   - byte (alias for uint8): 0
   - rune (alias for int32): 0

2. Boolean:
   - bool: false

3. String:
   - string: "" (empty string)
4. Pointer types:
   - *T (any pointer): nil
5. Function types:
   - func(): nil
6. Interface types:
   - interface{}: nil
7. Slice types:
   - []T: nil
8. Map types:
   - map[K]V: nil
9. Channel types:
   - chan T: nil
10. Array types:
    - [n]T: Array of n zero values of type T
11. Struct types:
    - struct{}: All fields set to their zero values