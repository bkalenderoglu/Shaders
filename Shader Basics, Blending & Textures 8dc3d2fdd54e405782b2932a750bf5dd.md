# Shader Basics, Blending & Textures

Shaders → code running on GPU, rendered graphics onto screen

Shaders are completely unrelated to textures. Textures are input data that shaders can use (optional). Normal maps and diffuse textures are examples of data that we often use in shaders but, shaders is more about taking the information from textures or sometimes use just other parameters or values or colors and running bunch of math and outputting to screen.

Normal maps are kind of a way to add more geometry data or what looks like geometry data without actually adding geometry.

Shaders can be used in UI, shadowing.

![Untitled](Shader%20Basics,%20Blending%20&%20Textures%208dc3d2fdd54e405782b2932a750bf5dd/Untitled.png)

**Subsurface scattering** → Subsurface scattering (SSS), also known as subsurface light transport (SSLT), is **a mechanism of light transport in which light that penetrates the surface of a translucent object is scattered by interacting with the material and exits the surface at a different point**.

![Untitled](Shader%20Basics,%20Blending%20&%20Textures%208dc3d2fdd54e405782b2932a750bf5dd/Untitled%201.png)

**FRESNEL(frenel)** → glowy effect to a faces away from you, almost an outline effect, highligting objects

- Fresnel works best for surfaces that have relatively smooth surfaces. As soon as things get sharp, it becomes really hard to get same outline effect.

## Structure of a shader

- .shader (unity file of shader)
    - Properties (Input data)
        - Colors
        - Values(ex. health value for health bar)
        - Textures
        - Mesh thats going to be rendered (Implicitly passed data)
        - Matrix(4x4) for that mesh contains the transform data of where it is, how it’s rotated and scaled (Implicitely passed data)
    - SubShader
        - Pass (render pass)
            - Vertex Shader
            - Fragment Shader
            
               HLSL - actual shader code
            

## Vertex Shader

- Basically takes all the vertices of the mesh.
- It’s kind of like foreach loop over all of the vertices and then you can able to access data from that all of those vertices (foreach Vertex)
- Let say we have a cube that we want to put somewhere in the world. One problem is that vertex shader doesn’t want you to put things in world space, not in local space either and not even in view space. The vertex shader wants you to say where are these vertices are going to be in what’s called **clip space.**

![Untitled](Shader%20Basics,%20Blending%20&%20Textures%208dc3d2fdd54e405782b2932a750bf5dd/Untitled%202.png)

- Clip space is like a normalized space from -1 to 1 inside of your current view or render target.
- You basically get local space coordinate of all of the vertices and then you transform that using **mvp matrix** to then convert it to clip space.
- Used for animating water, leaves that sway, grass that sway in the wind
- Either set the position of vertices or pass data to fragment shader

## Fragment Shader

![Untitled](Shader%20Basics,%20Blending%20&%20Textures%208dc3d2fdd54e405782b2932a750bf5dd/Untitled%203.png)

- foreach loop over every fragment
- The reason its called fragment not a pixel is because pixel is usually one to one corresponding to pixel on your screen but in shaders that’s not always the case
- fragment shader set the color of every fragment
- there is always one way communication that you can pass information through vertex shader to fragment shader

### Shader vs Material

- Properties (Input data)
    - Colors
    - Values(ex. health value for health bar)
    - Textures
    - Mesh thats going to be rendered (Implicitly passed data)
    - Matrix(4x4) for that mesh contains the transform data of where it is, how it’s rotated and scaled (Implicitely passed data)
    
    Green ones are usually called properties that you can configure before you’re rendering your objects and then you have some extra data (blue ones) mesh that you’re gonna render and the transform of that mesh itself. These things are usually like automatically supplied by your mesh renderer components but these properties you have to define yourself. Usually the way the define them is using material. So the material itself contains explicit values that we can put our shader
    
- Material also has a reference to the shader. You can never add a shader to an object in unity. In unity, you apply a material to an object and that material has a reference to the shader itself.
- Materials are sort of pre-configured parameters to be used when rendering something with a shader.
- You can have multiple materials that all use the same shader

Subshaders has information like it is opaque, transperent, render queue etc.

![Untitled](Shader%20Basics,%20Blending%20&%20Textures%208dc3d2fdd54e405782b2932a750bf5dd/Untitled%204.png)

Fragment shader gets the color information on each vertex and blend them and its called **barycentric interpolation**. But the fragment itself does not have the color information of vertices that is blended by shader. It only has the color information of its own.

## Blending Modes

- color out of a fragment shader → called source color in terms of blending (src)
- background as in destination that we’re rendering to, whatever is already behind the object (dst)
- blending modes are defined in Pass

![Untitled](Shader%20Basics,%20Blending%20&%20Textures%208dc3d2fdd54e405782b2932a750bf5dd/Untitled%205.png)

- **Additive Blending**(make things brighter usually)
    - takes the background and add it straight up
    - flashy effects, light effects, flame particles
    - set A to 1, operator(+), set B to 1
- **Multiplicative Blending**
    - set A to dst, operator(+), set B to 0

### Depth Buffer

- big screen space texture where some shaders write a depth value which is between 0 - 1 and when other shaders want to render they check this depth buffer to see is this fragment behind or in front of the depth buffer (if its behind it will not render)
- In unity’s built in render pipeline, there’s an order in which different types of geometry tends to render. First thing that renders a skybox. Secondly opaque or it’s called geometry, and then transparent after that overlays, lens flares.

Culling → removal of something (Back/Front/Off)

ZTest → LEqual - if the depth of this object less than or equal to depth buffer show it, otherwise dont

ZTest → Always - ignores depth buffer

ZTest → GEqual - only going to draw i it’s behind the something

### Vertex Offset

## Textures

- uvs can be used for just passing data that is completely unrelated to the textures and textures don’t have to use uv coordinates even though that is very common.

![Untitled](Shader%20Basics,%20Blending%20&%20Textures%208dc3d2fdd54e405782b2932a750bf5dd/Untitled%206.png)

### **Mip Maps**

The reason for having mip maps is that they are down sampled so that, when the shader is going to sample from texture, it sort of guess how far away the texture is by using partial derivatives. Based on the rate of change of uvs that you’re sampling with, it can pick a lower mip that matches the distance. This way we can get rid of noisy look.

**Screen Space Partial Derivative** → Approximation of the rate of change of input value given. So if we know how quickly the signed distance field is changing in screen space, we can use that as a way to do a blending depending on how far/close you are. These types of partial derivatives is actually how the texture samplers figure out how mip level to pick in the textures as well. (fwidth())