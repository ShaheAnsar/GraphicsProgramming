---
title: "Basic Vulkan Programming - 01"
date: 2018-03-02T12:54:05+05:30
draft: false
---
# Starting out with Vulkan  


In the following weeks I'll guide you through the basics of vulkan programming.  
  
  
In this particular post, we'll be talking about how to set everything up. It'll be only in the next post that we'll actually write any code.

With that said, let's get to it.

## Setting Everything Up

To make a vulkan application, we need a few things -  

* A Window handling library
* A Vulkan SDK (You can do without this, and I encourage you to if learning, but don't do this on an actual project)
* A Vector Math library
* An Image library

We'll be using GLFW, the LunarG SDK, GLM and stb_image for all of these.

### Installing GLFW, GLM and the SDK
#### Linux
Depends on your distro. I'll list the two distro (archetypes?) that I've had experience with -
##### Arch Linux
`[sudo] pacman -S glfw-x11`  
If you're on wayland get the wayland counterpart.  
`[sudo] pacman -S vulkan-devel`  
`[sudo] pacman -S glm`  

##### Ubuntu-like
`[sudo] apt install libglfw3 libglfw3-dev`  
`[sudo] apt install libvulkan1`  
`[sudo] apt install libglm-dev`  
#### Windows
Just download the files from -

* [GLFW's website] (http://www.glfw.org)
* [LunarG's Website] (https://vulkan.lunarg.com/)
* [GLM's Website] (https://glm.g-truc.net/0.9.8/index.html)  


Then put the include files and lib files where you want,
and then link to the libraries. Remember that the includes might differ depending on how you set your project up.  
<br>
<br>
<br>
As for stb_image, it's a [single header file] (https://github.com/nothings/stb/blob/master/stb_image.h), it should be fairly easy
to integrate in the project.
<br>
<br>
<br>
Now it's time to set everything up. Unfortunately, I won't be providing instructions for Windows, since I don't have it installed.
I'll explain how to set up a meson build system for this project, however, and that supports generating VS project files. For more on installation, visit [this] (http://mesonbuild.com)
### Getting the Build System up and running
First make a directory wherever you want for this project.
Now create a meson.build file in root directory of the project.  

```ruby
project('NAME OF PROJECT', 'cpp')
src = ['src/main.cpp']
include_dirs  = ['include/']
deps = [dependency('vulkan'), dependency('glfw3')]
executable('NAME OF BUILT EXECUTABLE', src,
	dependencies: deps, 
	include_directories: include_directories(include_dirs))
```

This is a very barebones build file, but that's all we need for this series

We'll also create a very basic main.cpp file in the `src/` directory to test if everythin is setup fine-  
```cpp
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>

int main(void)
{
	return 0;
}
```
Now in the source directory, type the following-  
```sh
meson build
cd build
ninja
```

And you should have a barebones executable in the build folder. And that's all for today.  
In the next post we'll learn how to set up a vulkan instance. Till then, ciao.
