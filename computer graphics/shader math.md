# sin/cos - wave function
- `sin` is a wave function that take the input and return the value between -1 and 1 and repeats every 2π. (starts at 0)
- `cos` is just like `sin`, but shifted left by pi/2 (starts at 1)

> TIP: You use `* 2.0 * PI` because a **full sine wave cycle** takes **2π radians**.
- sin(x) completes **one full wave** (0 → 1 → 0 → -1 → 0) every 2π units of x
- So if you want:
    - loop_time from 0.0 to 1.0
    - to map that to a full sine wave

→ you need to **stretch** the input to sin() across 0 → 2π:

# lerp/mix - linear interpolation
$\text{mix}(a, b, t) = a \cdot (1 - t) + b \cdot t$
- When t = 0 → result is a
- When t = 1 → result is b
- When t = 0.5 → halfway between a and b

Think of it like “Go t percent of the way from a to b”
```c
mix(0.1, 0.5, 0.0) = 0.1
mix(0.1, 0.5, 1.0) = 0.5
mix(0.1, 0.5, 0.5) = 0.3

vec3 a = vec3(1.0, 0.0, 0.0);
vec3 b = vec3(0.0, 0.0, 1.0);
vec3 result = mix(a, b, 0.5); // result = purple (0.5, 0.0, 0.5)
```

# clamp - constraint value between min and max values
```c
clamp(10, 0, 1) = 1
clamp(-1, 0, 1) = 0
clamp(0.5, 0, 1) = 0.5
```

# fract - fractional part of the number
```c
fract(0.0)   = 0.0
fract(0.25)  = 0.25
fract(0.99)  = 0.99
fract(1.0)   = 0.0
fract(1.75)  = 0.75
```

# distance(a, b) - returns the straight-line distance between a and b.
> “How far is point a from point b?”

$\text{distance}(a, b) = \sqrt{(a.x - b.x)^2 + (a.y - b.y)^2}$

```c
distance(0.0, 1.0)         → 1.0
distance(vec2(0.0, 0.0), vec2(1.0, 0.0)) → 1.0
distance(vec2(0.0, 0.0), vec2(1.0, 1.0)) → 1.414
```

# step(a, b) - returns hard threshold
$\text{step}(e, x) = \begin{cases} 0.0 & \text{if } x < e \\ 1.0 & \text{if } x \ge e \end{cases}$
> It acts like a **binary switch**: off before edge, on after.
> “Cut off at a certain point”
> 
```c
step(0.5, 0.3) = 0.0  // x < edge
step(0.5, 0.5) = 1.0  // x ≥ edge
step(0.5, 0.8) = 1.0
```

# smoothstep(edge0, edge1, x) - smooth curve (ease-in, ease-out) step

Returns a **smooth transition** from 0.0 to 1.0 indicating how much progress we've made as x moves from edge0 to edge1.
```c
smoothstep(0.2, 0.8, 0.5) = ~0.5
smoothstep(0.2, 0.8, 0.2) = 0.0
smoothstep(0.2, 0.8, 0.8) = 1.0
```

# dot - measures dot product of a and b vectors
> How much two vectors are pointing in the same direction
- If they point the **same way** → dot is **large (positive)**
- If **perpendicular** → dot is **0**
- If opposite → dot is **negative**
```c
vec2 a = vec2(1.0, 0.0);
vec2 b = vec2(0.0, 1.0);
dot(a, b) = 0.0  // perpendicular. 90 degrees

vec2 c = vec2(1.0, 1.0);
dot(c, c) = 2.0  // max alignment with itself
```