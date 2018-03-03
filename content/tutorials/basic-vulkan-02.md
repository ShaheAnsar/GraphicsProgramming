---
title: "Basic Vulkan 02"
date: 2018-03-03T00:38:11+05:30
draft: false
---
# Understanding what an instance is
In vulkan, the main object you'll be interacting with is an instance. The instance is similar to Open GL's concept of contexts.
You can think of it as an object that holds details that tells vulkan that we're a vulkan application. In reality, it simply holds a bunch of state
for the vulkan API. Creating it is the first step to creating any vulkan application. You can pass a bunch of information, and we certainly will later down the road.
But today, I'll be showing you how to create an instance.  


__NOTE:__ You WILL not be seeing anything on screen until much later (since this is vulkan). Drawing on it will come even later.  

# Setting up the skeleton of the codebase
Let's continue from the last article's barebones main.cpp file.  
We want to keep the clutter to a minimum. So let's put three functions within the main function  
```cpp
int main(void)
{
	Game::init();
	Game::loop();
	Game::deinit();
}
```
We want to keep the main game code in a separate namespace, so that we'll not clash in the future with other libraries (That isn't going to happen
in this series, but if you want to evolve this into a game engine, starting here is a good idea). Now let's define them in file called `game.hpp`.
I'm not going to be making a separate cpp file because the code will be fairly small.
```cpp
namespace Game
{
	void init()
	{
		//init code
	}
	void loop()
	{
		//loop code
	}
	void deinit()
	{
		//deinit code
	}
};
```
We can finally get to vulkan now. We'll keep that in another namespace called Gfx. So let's add two other files `src/gfx.hpp` and `src/gfx.cpp`  

	
__`src/gfx.hpp`__
```cpp
#define GLFW_VULKAN_INCLUDED //automatically loads all needed vulkan headers
#include <GLFW/glfw3.h>
#include <iostream>
namespace Gfx
{
	class Renderer
	{
		VkInstance _instance; // The underscore in front of it is my convention for a member variable
	public:
		init(); // I'm doing this in a separate function to be explicit while using it. 
			//If you want to, you can use the constructor and destructor, and take advantage of RAII, but I prefer to do it this way
		deinit();
	};
};
```
__`src/gfx.cpp`__
```cpp
#include "gfx.hpp"
void Gfx::Renderer::init()
{

}

void Gfx::Renderer::deinit()
{

}
```
# Actually coding
Now let's talk about what will go inside the above two functions.  
In the init function we want to initialize the instance. We also want to pass in some information about our game / game-engine to the instance's create function.
In the deinit function, we'll have to destroy the data that's put into it.
So let's get to it.  

__`Gfx::Renderer::init()`__
```cpp
VkApplicationInfo appInfo = {};
appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
appInfo.pApplicationName = "Name of our Application";
appInfo.applicationVersion = VK_MAKE_VERSION(0, 1, 0); // Follows this format - (MAJOR, MINOR, PATCH)
appIngo.pEngineName = "Engine Name";
appInfo.engineVersion = VK_MAKE_VERSION(0, 1, 0);
appInfo.apiVersion = VK_API_VERSION_1_0;
```
The above section is completely optional, but still good practice, since engine and game specific optimizations
can be done by the driver for you engine (if it ever becomes big enough).  
But more importantly, this section tells us something very important about vulkan - it's struct based.
You will almost certainly pass a struct before every function call. Think this sucks? Well, deal with it. It's a low
level library that is by design extremely verbose.  
One __very__ important thing to keep in mind is that every structure you'll make that's of a vulkan defined type,
will have the `sType` field. It, of course stands for `structure Type`. You __must__ always fill it up.  
Now onto actually creating the instance-  

__`Gfx::Renderer::init()`__
```cpp
... //Created the appInfo
VkInstanceCreateInfo instanceCreateInfo = {};
instanceCreateinfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
instanceInfo.pApplicationInfo = &appInfo;
instanceInfo.enabledLayerCount = 0; // Debugging stuff. We'll get to this later
instanceInfo.enabledExtensionCount = 0; // Again, we'll get to this later. I'll simply use || to represent that
instanceInfo.ppEnabledExtensionNames = nullptr; // ||
instanceInfo.ppEnabledLayerNames = nullptr; // ||

if(vkCreateInstance(&instanceCreateInfo, nullptr, &_instance) != VK_SUCCESS)
{
	std::cerr << "ERROR: Unable to create instance!!!" << std::endl;
	exit(-1);
}
```
I'll first start explaining a few conventions in the vulkan library, then explain what I did -

* If you're creating anything, you'll be creating a *CreateInfo struct for that.
* All fields and parameters have `p` prepended to them to denote a pointer. Two `p`s denote a double pointer
* Every function that creates something will take a *CreateInfo struct, a pointer to an allocator (will not be used in this series, but will be discussed much later), and a pointer to the object it creates.
* Every function will return a VK_Result, and it'll have the value `VK_SUCCESS` if it succeeds, otherwise, some other value.  

With that said and done, let's talk about a few things in the above piece of code.  
You'll see two pairs of peculiar fields when filling the struct up - `enabledLayerCount and ppEnabledLayerNames` and `enabledExtensionCount and ppEnabledExtensionNames`.  
Layers in particular are interesting. I'll explain them in more detail down the road, but they're a superb debugging mechanism that lets us add hooks to Vulkan library functions and print details if things go wrong.  
Extensions on the other hand let us add extra functionality to the core Vulkan libraries.
For example, since Vulkan is platform agnostic, we need an extension to access the handler to
the surface of a window.  
All of that though is for the next tutorial, where we'll enable a few extensions and layers.  

Now let's fill up the `Gfx::Renderer::deinit()` function, which contains a single line of code-  
__`Gfx::Renderer::deinit()`__
```cpp
vkDestroyInstance(_instance, nullptr);// Also keep in mind that all functions that destroy objects also take
                                      // a parameter to an allocator. We don't need one, so it's a nullptr
```  

Now let's set everything up.  
We want to include all the needed headers and call the proper functions.  
__`main.cpp`__  
```cpp
#include "game.hpp"

```
__`game.hpp`__
```cpp
#include "gfx.hpp
...
	Renderer renderer;
	init()
	{
		renderer.init();
	}
	...
	deinit()
	{
		renderer.deinit();
	}
```
And that's it. We're almost done now. All that's left is modifying the `meson.build` file.

# Modifying meson.build  
Open up the meson.build file and change the following variables like this -  
```ruby`
src = ['src/main.cpp', 'src/gfx.cpp']
```
Now, go to the build directory and run `ninja`. You shouldn't get any errors and have a compiled project
that does nothing for now. Hey, at least we have progress.  
On my next post (which will be pretty big, I think) we'll talk about layers and extensions.  
Till then, ciao
