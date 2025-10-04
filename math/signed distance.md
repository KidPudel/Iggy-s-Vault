Signed distance is a method that calculates what point of the shape is inside, outside, at the edge.
- negative value → inside the shape
- zero → exactly on the edge
- positive → outside the shape

This is done by making the center of the shape `[0.0, 0.0]` and stripping the half (or less if we want to strip from the shape)


```c
// general purpose signed-distance, inside is negative due to being centered and subtracted with the guide on how much we want to strip
float sd_round_rect(vec2 p, vec2 half_size, float rad) {
	// positive outside of the corners, negative inside of the coreners
	// since we fold in half with abs, since 0.0 is center
	vec2 q = abs(p) - half_size;
	// max makes all the outside inside parts are 0, and positive is the distance pass the edge.
	// length is the Euclidean distance to the rectangles border
	// the - radius actually shifts the edge to be - an
	return length(max(q, vec2(0.0))) - rad;
}

// --- Rounded corners ---
	// center to make life easier
	vec2 centered_p = UV - vec2(0.5);
	
	// half_size for signed distance field with included corner_radius to shift it, so the edges now outside
	vec2 hs = vec2(0.5 - corner_radius, 0.5 - corner_radius);
	// signed distance with corner radius stripping
	float sd = sd_round_rect(centered_p, hs, corner_radius);
	// anti-aliasing. how much sd changes across one pixel, for small smoothing band
	float aa = fwidth(sd) * 1.5;
	// flip the signed distance positive inside, negative outside
	// 0 to 1 with anti-aliasing the result as mask
	// 0 will be at the corners where values had x and y the greater and then flipped
	float round_mask = smoothstep(0.0, aa, -sd);
	
	
	ALPHA = opacity * round_mask;

```