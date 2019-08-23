Documentation Author: Niko Procopi 2019

This tutorial was designed for Visual Studio 2017 / 2019
If the solution does not compile, retarget the solution
to a different version of the Windows SDK. If you do not
have any version of the Windows SDK, it can be installed
from the Visual Studio Installer Tool

Welcome to the Customize Lights Tutorial!
Prerequesites: Loop reflection

In this tutorial, we make it so all lights
can have their own position, range, color,
and brightness

In previous tutorials, we have a structure of a triangle,
and we have an array of 26 triangles. In this tutorial,
we edit that array, to change the shape of the blocks,
as well as change the colors of the blocks. We also set
the floor to a white color.

We add a new structure for lighting. Each light has a vec3
for position, a vec3 for color, a float for radius, and
a float for brightness. We also now have an array of 4 
lights. Feel free to adjust this array. They can be found
near the top of the fragment shader.

Now that each light has its own brightness, we don't need
the lightIntensity variable, so that has been removed everywhere

addLightColorToPixColor now has 3 parameters: Light, direction, hitInfo
addReflectionToPixColor now has 4 parameters: Light, direction, hitInfo, maxBounces

Our trace() function is much simpler, now that we aren't creating an
array of light positions every time it is called. The trace function
simply sends a ray from the eye to the scene, then adds color to a pixel
if the ray hits geometry, then adds reflections, and then returns the color.

addLightColorToPixColor has a new optimization that was not previously possible.
In other tutorials, lights did not have a fixed radius. That means, every pixel
had to calculate each light. In this tutorial, lights have a fixed radius. That
means, we can determine if a pixel is outside of a light's radius, and then
we don't need to process the light on that pixel. Some pixels in the scene will
have no lights calculated for them at all. We can do this with a simple check:
	if(dist > L.radius)
		return vec3(0);
We do not return any color of a pixel is not in the light's radius
Next, we do the calculation for range-based attenuation, which
is the same as previous lighting tutorials:
	float atten = 1.0 - (dist*dist) / (L.radius*L.radius);
	atten = clamp(atten, 0.0, 1.0);

How to improve:

Uniform buffers
	Keep the triangles array, make it empty
	Put the plane, and each cube in a seperate uniform buffer
	Give them each a model matrix
	Use a compute shader to move them, each with their own model matrix
	Have the compute shader write to the triangles array, just like
	how the compute shader wrote the the vertex buffer in Basic Compute Particles
	
More Optimization:
	Give each model a spherical radius, so that rays check for collision
	with the spheres, before checking for collision with each polygon that
	is inside the sphere. After this, keep doing more spatial partitioning

Textures
	Between "float lightIntensity = 6.0;" and "vec3 pixColor = ..."

	i.point is the 3D coordinate that a ray hits a triangle
	triangles[i.index].a is one point on the triangle
	triangles[i.index].b is one point on the triangle
	triangles[i.index].c is one point on the triangle
	UV coordinates dont exist yet
	Use barycentric coordinates to get (a+b+c=1) variables
	to compare points of triangle to point intersecting
	Then use (a+b+c=1) to compare points of triangle's UVs
		to get the UV coordinate of the point you hit
	For Debugging, do pixColor = uv * 0.1
		Then do vec3 pixColor = sample(texture, uv) * 0.1
	However
		This will not change the color of squares in reflection
		triangles[eyeHitPoint.index].color needs to be replaced with 
		reflectHit.point (like i.point) to calculate UVs all over again
		
Skybox
	On lines 274, in Reflection FragmentShader.glsl
	When you see if(intersectTriangles ...
	Should have "else ..." to get skybox color
	We already have direction, we just need color from cubemap
	We do NOT change the if(intersectTriangles ... at line 241,
	because that is for shadows
	On line 346, change "return vec4(0,0,0,1)" to 
	return the skybox color. This will be the actual background
	wallpaper skybox