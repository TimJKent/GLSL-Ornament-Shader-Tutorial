# GLSL-Ornament-Shader-Tutorial
A tutorial on creating shaders on https://glsl.app/ that look like ornaments


## Step 1 - Color

Lets get some color up on the screen! 

```glsl
#version 300 es

precision highp float;

out vec4 out_color;

void main()
{
    //Notice how changing these values changes the color on the display
    out_color = vec4(1.0, 0.0, 0.0, 1.0); // R G B A
}
```

## Step 2 - Sphere

Lets make a sphere for our ornament. We will set the red channel to be based off the height of a sphere using CalculateSphereZ. Feel free to play around with setting the green or blue channel to the sphere height and see how that affects the outcome. What about 1.0-CalculateSphereZ()?

```glsl
#version 300 es

precision highp float;

in vec2 uv; // This provides the position of the current pixel
out vec4 out_color;

float CalculateSphereZ(vec2 pos)
{
  //This returns a "height" value at <pos> based off a 0.5 radius sphere centered at (0.5,0.5)
  return sqrt(1.0 - ((pow(pos.x - 0.5,2.0) + pow(pos.y-0.5, 2.0))/0.150));
}

void main()
{   
    //Notice how setting our red channel to our height value creates a nice sphere effect!
    out_color = vec4(CalculateSphereZ(uv),0.0,0.0, 1.0); 
}
```

## Step 3 - Lighting

Lets add a light we can move around to make the scene more interesting!

```glsl
#version 300 es

precision highp float;

in vec2 uv; // This provides the position of the current pixel
uniform vec4 u_mouse; //This provide the location of the mouse
uniform vec2 u_resolution; //This provides the size of the render display
out vec4 out_color;


//This calculates lighting using diffuse and specular techniques
//Read more about these here: https://learnopengl.com/Lighting/Basic-Lighting
vec3 CalulateLighting(vec3 pos, vec3 Normal, float shininess)
{
  vec2 mouse = u_mouse.xy / u_resolution;
  float light_x_pos = mix(-10.0,10.0, mouse.x);
  float light_y_pos = mix(-10.0,10.0, mouse.y);
  vec3 light_pos = vec3(light_x_pos,light_y_pos, pos.z*6.0);
  vec3 lightColor = vec3(2.0,2.0,2.0);
  vec3 lightDir = normalize(light_pos - pos);
  float diff = max(dot(Normal, lightDir), 0.0);
  vec3 reflectDir = reflect(-lightDir, Normal);
  vec3 halfwayDir = normalize(lightDir + vec3(0.0,0.0,1.0));
  float spec = pow(max(dot(Normal, halfwayDir), 0.0), 2048.0) * shininess;
  float dis = distance(light_pos, pos);
  float brightness = 1.0;
  return (diff + spec+0.1)* brightness * lightColor / (dis*0.5);
}

float CalculateSphereZ(vec2 pos)
{
  //This returns a "height" value at <pos> based off a 0.5 radius sphere centered at (0.5,0.5)
  return sqrt(1.0 - ((pow(pos.x - 0.5,2.0) + pow(pos.y-0.5, 2.0))/0.150));
}

void main()
{   
  //Lets create a position value for our sphere
  vec3 sphere_position = vec3(uv, CalculateSphereZ(uv));
  sphere_position.x -=0.5;
  sphere_position.y -=0.5;

  //Spheres normals are just its position normalized (A normal is a surfaces outwards pointing vector)
  vec3 normal = normalize(sphere_position);

  //Now we can use our light based value for color.
  vec3 sphere_color = vec3(1.0,0.0,0.0);
  sphere_color *= CalulateLighting(sphere_position, normal, 1.0);
  out_color = vec4(sphere_color.rgb,  1.0); 
}
```

## Step 4 - Textures

Its been fun playing with colors up to this point, but lets add a personal touch by using our own image on the sphere.
I'll be using some trigonomtry to get the image to look like its wraping the sphere nicely.
Once you paste this your display screen will go blank. You will need to set a texture by clicking on the "Textures" button and assign an image to TEXTURE_0

```glsl
#version 300 es

precision highp float;

in vec2 uv; // This provides the position of the current pixel
uniform vec4 u_mouse; //This provide the location of the mouse
uniform vec2 u_resolution; //This provides the size of the render display
uniform sampler2D u_texture; //This provides access to a texture (in this case click textures and provide one in TEXTURE 0)
uniform float u_time; //This provides access to a texture (in this case click textures and provide one in TEXTURE 0)
out vec4 out_color;


//This calculates lighting using diffuse and specular techniques
//Read more about these here: https://learnopengl.com/Lighting/Basic-Lighting
vec3 CalulateLighting(vec3 pos, vec3 Normal, float shininess)
{
  vec2 mouse = u_mouse.xy / u_resolution;
  float light_x_pos = mix(-10.0,10.0, mouse.x);
  float light_y_pos = mix(-10.0,10.0, mouse.y);
  vec3 light_pos = vec3(light_x_pos,light_y_pos, pos.z*6.0);
  vec3 lightColor = vec3(2.0,2.0,2.0);
  vec3 lightDir = normalize(light_pos - pos);
  float diff = max(dot(Normal, lightDir), 0.0);
  vec3 reflectDir = reflect(-lightDir, Normal);
  vec3 halfwayDir = normalize(lightDir + vec3(0.0,0.0,1.0));
  float spec = pow(max(dot(Normal, halfwayDir), 0.0), 2048.0) * shininess;
  float dis = distance(light_pos, pos);
  float brightness = 1.0;
  return (diff + spec+0.1)* brightness * lightColor / (dis*0.5);
}

float CalculateSphereZ(vec2 pos)
{
  //This returns a "height" value at <pos> based off a 0.5 radius sphere centered at (0.5,0.5)
  return sqrt(1.0 - ((pow(pos.x - 0.5,2.0) + pow(pos.y-0.5, 2.0))/0.150));
}

void main()
{   
  vec3 sphere_position = vec3(uv, CalculateSphereZ(uv));
  sphere_position.x -=0.5;
  sphere_position.y -=0.5;

  vec3 normal = normalize(sphere_position);

  //We calculate special normals to keep up the illusion of a sphere
  vec2 sphericalUV = vec2(atan(normal.z, normal.x) / (2.0 * 3.1415), 0.5 - asin(normal.y) / -3.1415);
  
  float rotation_speed = 0.1; //Set to 0 if you want the sphere to stop spinning
  sphericalUV.x += (u_time*rotation_speed);

  vec3 sphere_color = texture(u_texture, sphericalUV).rgb; //Now instead of a fixed color we will sample our texture using our new sphericalUV
  sphere_color *= CalulateLighting(sphere_position, normal, 1.0);
  out_color = vec4(sphere_color.rgb,  1.0); 
}
```

### Step 5 Customize

At this point feel free to play with the values in the shader and make the ornament your own!

<img width="648" height="578" alt="image" src="https://github.com/user-attachments/assets/fc323763-d016-4bd0-a738-613fda5588a8" />



