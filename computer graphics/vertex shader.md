Processes individual vertices ([[vertex]] points in 3D space) of a 3D model.
It **transforms** vertex positions from 3D world to 3d screen coordinates, then it passes data (texture coordinates, normals, acolors) to the next date of [[graphics pipeline]]
NOTE: vertex shader only works with existing vertices, it cannot add or delete them.


```c
struct INPUT {
	float4 position: POSITION;
	float2 uv: TEXCOORD;
	float3 normal: NORMAL;
	uint id: SV_VetexId;
};

struct OUTPUT {
	float4 position: POSITION;
	float2 uv: TEXCOORD;
	float3 color: COLOR;
};


// external data
cbuffer Buffer {
	float4x4 matrix;
};


void main(in INPUT input, out OUTPUT output) {
	output.position = mul(input.position, matrix);
	output.uv = input.uv;
	// just as an example
	output.color = input.normal / input.id;
}

```