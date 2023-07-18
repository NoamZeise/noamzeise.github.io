---
layout: post
title: Reforestation - Board Game in 3D (GMTK2023)
category: GameJam
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/has_7hJQwrI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

My first game in 3D, this was made in 48 hours for GMTK Jam 2023 using my [graphics framework](https://github.com/NoamZeise/Graphics-Environment)

<!-- more -->

I used this jam as an excuse to do a game in 3D. This was a very different experience to previous games, and I underestimated the challenges I would face. I had to change the complexity of the game significantly due to time constraints, and could only make a few levels. Overall I'm happy with the result, and look forward to doing more 3D games in the future.


I used raycasting to get the position on the box that the cursor was hovering over to let the user place things on the board. After correcting for scaling and resolution that the graphics framework does to keep the game in the target resolution, I had values for the mouse position between -1 and 1 in the x and y direction. Using this we can calculate an outward facing ray and check at what position it would intersect a plane with a certain normal. I had the board facing up at a height of zero. I also precalculated the view and projection matrix inverses, and only changes these when the matrices changed.

Here is the raycasting function:

```
// mouse position on the screen ([-1, 1], [-1, 1])
glm::vec4 rayClip(xPos, yPos, -1.0f, 1.0f);
    
// using the inverse of the projection matrix gets
// the mouse position in camera space
glm::vec4 rayCam = projInverse * rayClip;
rayCam.z = -1.0f;
rayCam.w = 0.0f;
    
// we can then use the inverse of the view matrix
// to go from camera space to world space
// this means the ray's origin is the camera position.
glm::vec4 rayWorld4 = viewInverse * rayCam;
glm::vec3 rayWorld(rayWorld4.x, rayWorld4.y, rayWorld4.z);
rayWorld = glm::normalize(rayWorld);
	
glm::vec3 planeNormal(0.0f, 0.0f, 1.0f); //facing up
float denom = glm::dot(rayWorld, planeNormal);
if(denom != 0) { //ray is not orthogonal to the plane
	
    // solve for the intersection distance from the ray origin
   	// (camera position) to the plan.e 
	// + 0.0f, this is the distance of the plane along it's normal
	float t = -(glm::dot(cam.getPos(), planeNormal) + 0.0f) / denom; 
		
	// get the hit position by substituting the parameter
	// for the the equation of a ray with t
	glm::vec3 hit = cam.getPos() + rayWorld * t;
	return hit;
}
// return nonsense if the ray is orthogonal to the plane
return glm::vec3(10000.0f);
```

Then it was a simple matter of passing the intersection position to the board renderer
and checking which tile it struck, so that the tile's colour could be changed to red.


I didn't use any text or audio, so I could remove some of the heavy dependencies of the game (freetype, libsndfile, portaudio). I only used .obj models, so I could build assimp with only one model type. These saves reduced the final size of the game. The game is 2mb, not including the windows C/C++ runtime dlls I package with the game that will already be present on most user's systems. 