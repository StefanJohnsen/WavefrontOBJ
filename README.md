# WavefrontOBJ

Empower your project with a self-contained header-only C++ 11 OBJ parser.

### Compatibility and Dependencies
- C++ 11 Standard
- Standard Template Library (STL)

This project is designed to be compatible with C++11 and utilizes solely C++ Standard Template Library (STL) components. It has been thoughtfully engineered to function independently without the need for any external libraries or dependencies. This design ensures the project remains self-contained, promoting simplicity and ease of integration into your development environment.<br>

*If you require the use of **C++98**, see [Wavefront98](https://github.com/StefanJohnsen/Wavefront98) repository.

### Need a Wavefront Material Template Library (MTL) parser?
*When working with Wavefront OBJ files, having a robust Wavefront Material Template Library (MTL) parser can greatly enhance your workflow.
WavefrontOBJ is designed to complement and work seamlessly with the [WavefrontMTL](https://github.com/StefanJohnsen/WavefrontMTL) repository.*

### OS Support
- Windows
- Linux
- macOS

### Speed and memory
- By supporting C++ 11 Standard only and without use of multithreading, we have achieved a good level of speed.
- This solution boasts minimal memory usage, typically averaging twice the file size in memory consumption.
  
### Usage
Copy `WavefrontOBJ.h` to your project and include the file.

```
#include "WavefrontOBJ.h"
```

### Parsed examples
*WavfrontOBJ and [WavefrontMTL](https://github.com/StefanJohnsen/WavefrontMTL) parsers have been integrated into the [Polyscope 3D viewer](https://github.com/nmwsharp/polyscope), showcasing the parsing capabilities.*

![Polyscope](https://github.com/StefanJohnsen/WavefrontOBJ/blob/main/Pictures/polyscope.jpg)
*[3D model Mars Perseverance Rover](https://mars.nasa.gov/resources/25042/mars-perseverance-rover-3d-model/),  from NASA's Jet Propulsion Laboratory*

## 3D Model Data Structures
After parsing the obj file, users gain access to geometry by utilizing the following lists.
	
| Component | Description                                    |
|-----------|------------------------------------------------|
| Vertex    | Defines points in 3D space, shaping the model. |
| Texture   | Manages texture details for enhanced visuals.  |
| Normal    | Specifies surface orientation for lighting.    |
| Face      | Contains indices for creating flat surfaces.   |
| Line      | Stores indices, useful for wireframe models.   |
| Point     | Manages indices for individual 3D points.      |

WavefrontOBJ supports various vertex and texture coordinate lists, allowing you to copy coordinates to your list formats. 

### Supported user vertex list formats

- **Classic Format (x, y, z):** Represents vertices with x, y, and z coordinates in 3D space.<br>*{x,y,z, x,y,z, x,y,z, ....} => obj::VertexFormat::xyz*
  
- **Homogeneous Format (x, y, z, w):** Extends the classic format with an additional component, w, often used for transformations.<br>*{x,y,z,w, x,y,z,w, x,y,z,w, ....} => obj::VertexFormat::xyzw*
  
- **Ending RGB Format (x, y, z, r, g, b):** Combines vertex position with colour attributes, enhancing visual representation.<br>*{x,y,z,r,g,b, x,y,z,r,g,b, x,y,z,r,g,b, ....} => obj::VertexFormat::xyzrgb*

### Supported user texture list formats

- **Classic UV Format (u, v):** Represents texture coordinates using U and V values.<br>*{u,v, u,v, u,v, ....} => obj::TextureFormat::uv*
  
- **UVW Format (u, v, w):** Extends the UV format with an additional component, w, useful for specific texture mapping scenarios.<br>*{u,v,w, u,v,w, u,v,w, ....} => obj::TextureFormat::uvw*
  
When copying data, these formats follow your coordinate data structures. 

If your list aligns with any of the examples below, WavefrontOBJ allows you to copy the internal list to your list using a single copy operation.

```cpp
std::vector<float> vertex;
std::vector<float> normal;
std::vector<float> texture;

std::vector<double> vertex;
std::vector<double> normal;
std::vector<double> texture;

std::vector<std::vector<float>> vertex;
std::vector<std::vector<float>> normal;
std::vector<std::vector<float>> texture;

std::vector<std::vector<double>> vertex;
std::vector<std::vector<double>> normal;
std::vector<std::vector<double>> texture;
```
If not, you'll need to copy the internal list to your list manually.<br>
*See section [Load and get vertex manually](https://github.com/StefanJohnsen/WavefrontOBJ#sample-code).*

## Examples

### Load and count
```cpp
#include "WavefrontOBJ.h"
#include <iostream>

int main()
{
	obj::Load file;

	if( !file.load("C:\\temp\\example.obj") )
		return 1;

	std::cout << "Geometric vertices: " << obj.vertex.size() << std::endl;
	std::cout << "Texture vertices: " << obj.texture.size() << std::endl;
	std::cout << "Normal vertices: " << obj.normal.size() << std::endl;
	
	std::cout << "Vertex indices: " << obj.face.vertex.size() << std::endl;
	std::cout << "Texture indices: " << obj.face.texture.size() << std::endl;
	std::cout << "Normal indices: " << obj.face.normal.size() << std::endl;

	return 0;
}
```

### Load and get coordinates with the same list types
```cpp
obj::Load obj;

if (!obj.load("C:\\temp\\example.obj"))
	return 1;

std::vector<float> vertex;
std::vector<float> normal;
std::vector<float> texture;

obj::copy(obj.vertex, vertex, obj::VertexFormat::xyz); // xyz is default
obj::copy(obj.texture, texture, obj::TextureFormat::uvw); //uvw is defalt
obj::copy(obj.normal, normal);
```
*WavfrontOBJ will always try to move the list instead of copying the list.<br>
If all vertex in the above obj.vertex list has the same format as xyz, the obj.vertex list will be moved to vertex list.*

### Load and get coordinates with different list types
```cpp
obj::Load obj;

if (!obj.load("C:\\temp\\example.obj"))
	return 1;

{
	std::vector<double> vertex;
	std::vector<double> normal;
	std::vector<double> texture;

	obj::copy(obj.vertex, vertex);
	obj::copy(obj.texture, texture);
	obj::copy(obj.normal, normal);
}

{
	std::vector<std::vector<double>> vertex;
	std::vector<std::vector<double>> normal;
	std::vector<std::vector<double>> texture;

	obj::copy(obj.vertex, vertex);
	obj::copy(obj.texture, texture);
	obj::copy(obj.normal, normal);
}

```
*WavfrontOBJ will copy all lists above.<br>
Above obj.vertex list will be copied into vertex list with format xyz (default).<br>
You can choose xyz, xyzw or xyzrgb. Ex. If you choose xyzrgb and obj.vertex don't have that format, your list will contains zero values for rgb {x,y,z,0,0,0}*

### Load and get face colors
WavefrontOBJ uses [WavefrontMTL](https://github.com/StefanJohnsen/WavefrontMTL) to simplify the process of loading materials and colors. To easily integrate this feature, copy the WavefrontMTL.h file from the WavefrontMTL repository into the same directory as WavefrontOBJ.h. This enables straightforward handling of materials and colors within your project.

```cpp
#include "WavefrontOBJ.h"
#include "WavefrontMTL.h"
#include <iostream>

int main()
{
	obj::Load obj;

	if (!obj.load("C:\\temp\\example.obj"))
		return 1;

	mtl::Load mtl;

	if (!mtl.load(obj.mtllib()))
		return 1;

	std::vector<float> color;
	
	obj::Copy(obj, mtl, color);

	return 0;
}
```

Possible color lists:

```cpp
std::vector<float> color;
std::vector<double> color;

std::vector<std::vector<float>> color;
std::vector<std::vector<double>> color;
```

### Triangulation
In the Wavefront OBJ file format, 3D models are commonly described using triangles due to their simplicity and broad compatibility. However, the format also supports faces with polygons, which means more than three vertices. 
While some applications struggle to handle these polygons, many prefer triangles for predictable rendering.

To bridge this gap, a solution known as triangulation is provided. This process converts polygons into triangles, ensuring compatibility with applications that only handle triangles. 
This solution guarantees smoother integration and better rendering performance by automatically breaking down complex polygons. 

To perform triangulation on an OBJ file, specify the following object definition.
```cpp
obj::Load file(true);
```
By default, this setting is set to false, and there's a specific reason for that choice. Triangulation involves a resource-intensive process, so the default false setting prevents unnecessary performance overhead.

*See section [Triangulate polygons](https://github.com/StefanJohnsen/WavefrontOBJ#triangulate-polygons) at the end of this document.*

## Benchmark
The benchmark was conducted on a computer with the following specifications:

- **Processor:** Intel(R) Core(TM) i7-10750H CPU @ 2.60GHz   2.59 GHz
- **Memory:** 16.0 GB
- **Operating System:** 64-bit operating system, x64-based processor
- **Compiler:** Visual Studio Version 2022 (v143, C++ 14 Standard)
- **Release Mode:** Visual Studio release mode was used for benchmarking.

| File Name    | File Size (KB) | Load time (Milliseconds)  | Memory Usage   |
|--------------|---------------:|:-------------------------:|:--------------:|
|  rungholt    |     269 215    |           1271            |    594MB       |
|  powerplant  |     798 723    |           3563            |    1.6GB       |
|  san-miguel  |   1 116 252    |           4982            |    2.1GB       |

Time was evaluated through 10 iterative runs, calculating the average execution time.<br>
Memory usage was analyzed using Visual Studio 2022 C++17 Diagnostic Tools in Release mode.<br>
*See section [Benchmark code](https://github.com/StefanJohnsen/WavefrontOBJ#benchmark-code) at the end of this document.*<br>

The files above are sourced from [**Morgan McGuire, Computer Graphics Archive, July 2017**](https://casual-effects.com/g3d/data10).<br><br>
*The benchmark results vary based on the computer's hardware and software configuration.*

## References
The following sources have been utilized in developing this Wavefront OBJ parser.

[Paul Bourke: Object Files (OBJ)](http://paulbourke.net/dataformats/obj/)

[Wikipedia: Wavefront .obj file](https://en.wikipedia.org/wiki/Wavefront_.obj_file)

## License
This software is released under the GNU General Public License v3.0 terms.<br> 
Details and terms of this license can be found at: https://www.gnu.org/licenses/gpl-3.0.html<br><br>
For those who require the freedom to operate without the constraints of the GPL,<br>
a commercial license can be obtained by contacting the author at stefan.johnsen@outlook.com

## Solution comments

### Triangulate polygons

To illustrate the concept of triangulation, let's consider a straightforward OBJ file featuring a clock-like polygon. This polygon is constructed with vertices resembling a clock, where the 12 o'clock position naturally resides at the top.
<pre>
# .obj file for a circle using clock numbers as vertices

v 0.0 1.0 0.0       # 12 o'clock
v 0.5 0.866 0.0     # 1 o'clock
v 0.866 0.5 0.0     # 2 o'clock
v 1.0 0.0 0.0       # 3 o'clock
v 0.866 -0.5 0.0    # 4 o'clock
v 0.5 -0.866 0.0    # 5 o'clock
v 0.0 -1.0 0.0      # 6 o'clock
v -0.5 -0.866 0.0   # 7 o'clock
v -0.866 -0.5 0.0   # 8 o'clock
v -1.0 0.0 0.0      # 9 o'clock
v -0.866 0.5 0.0    # 10 o'clock
v -0.5 0.866 0.0    # 11 o'clock

g Polygon
f 1 2 3 4 5 6 7 8 9 10 11 12
</pre>

As observed, a polygon is present.<br>

![Polygon](https://github.com/StefanJohnsen/WavefrontOBJ/blob/main/Pictures/triangulation.jpg)

When dealing with polygons within a .obj file, we can follow a routine to convert these polygons into triangles for better compatibility and rendering. 

Let's assume we have a face list denoted as f(n) = {f0, f1, f2, ... pN}. We want to perform a process that generates triangles using these indices.

For each index in the list, the following steps are conducted:

- We begin with the initial polygon, f(0), from the list.

- Iterate over the range of indices i from 0 to N-1 (where N is the total number of indices in the list).

- For each i, we create a triangle using the indices of three points: f(0), f(i), and f(i+1). 
This forms a triangle that connects the starting indices(0) of the polygon, the current indices(i), and the next polygon indices(i+1) in the list.

- Repeat the triangle creation process for each i within the specified range, effectively creating a series of triangles that approximate the original polygon.

Following this routine, we break down complex polygons into simpler triangular elements, making it easier to work with them in 3D rendering and other applications that expect triangles. This conversion process ensures that the resulting triangles provide a good representation of the original polygon's shape and structure.

*Another algorithm in existence is the "ear clipping triangulation algorithm." This algorithm involves the process of identifying an "ear" within the current polygon and subsequently removing it. In its initial form, Meister's version of the ear clipping algorithm has a time complexity of O(n^3), with the majority of the time being allocated to verifying the validity of newly formed triangles.*

Our routine is straightforward and efficient, characterized by a time complexity of O(n). The procedure will prove effective and correct for all concave polygons. However, convex or intricate polygons may not yield accurate triangulation results. 

---

## Sample codes

### Load and get vertex manually
```cpp
obj::Load obj;

if (!obj.load("C:\\temp\\example.obj"))
	return 1;

auto vertex = obj.vertex.v.begin();

for (const auto& size : obj.vertex.s)
{
	obj::VertexFormat format = obj::vertexFormat(size);

	switch (format)
	{
		case obj::xyz:
		{
			double x = *(vertex + 0);
			double y = *(vertex + 1);
			double z = *(vertex + 2);

//			... your code here

			break;
		}
	
		case obj::xyzw:
		{
			double x = *(vertex + 0);
			double y = *(vertex + 1);
			double z = *(vertex + 2);
			double w = *(vertex + 3);
			break;
		}
	
		case obj::xyzrgb:
		{
			double x = *(vertex + 0);
			double y = *(vertex + 1);
			double z = *(vertex + 2);
			double r = *(vertex + 3);
			double g = *(vertex + 4);
			double b = *(vertex + 5);
			break;
		}
	}

	vertex += size;
}
```

### Load and get texture manually
```cpp
obj::Load obj;

if (!obj.load("C:\\temp\\example.obj"))
	return 1;

auto texture = obj.texture.v.begin();

for (const auto& size : obj.texture.s)
{
	obj::TextureFormat format = obj::textureFormat(size);

	switch (format)
	{
		case obj::uv:
		{
			double u = *(texture + 0);
			double v = *(texture + 1);
			break;
		}

		case obj::uvw:
		{
			double u = *(texture + 0);
			double v = *(texture + 1);
			double w = *(texture + 2);
			break;
		}
	}

	vertex += size;
}
```

### Load and get normal manually
```cpp
obj::Load obj;

if (!obj.load("C:\\temp\\example.obj"))
	return 1;

auto normal = obj.normal.v.begin();

for (const auto& size : obj.normal.s)
{
	double x = *(normal + 0);
	double y = *(normal + 1);
	double z = *(normal + 2);

	normal += size;
}
```

## Benchmark code
This code snippet demonstrates the file-loading performance for OBJ and MTL files. The program loads each file 10 times and calculates the average time taken for loading. The resulting average loading times are then displayed, providing insights into the efficiency of loading processes enabled by the WavefrontOBJ and WavefrontMTL libraries.

```cpp
#include "WavefrontOBJ.h"
#include "WavefrontMTL.h"
#include <iostream>
#include <chrono>

using namespace std::chrono;

template<typename Wavefront>
void SpeedTest(const std::string& file)
{
	std::cout << std::endl << "File " << file << " ";

	int count(0);

	int number_of_runs(10);

	microseconds microSeconds(0);

	while (count++ < number_of_runs)
	{
		Wavefront load;

		std::cout << ".";

		auto start = high_resolution_clock::now();

		if (!load.load(file)) return;

		auto stop = high_resolution_clock::now();

		microSeconds += duration_cast<microseconds>(stop - start);
	}

	microSeconds = std::chrono::microseconds(microSeconds.count() / number_of_runs);

	if(microSeconds.count() > 1000)
	{
		auto milliSeconds = duration_cast<milliseconds>(microSeconds);

		std::cout << std::endl << "Loading was completed in " << milliSeconds.count() << " milliseconds" << std::endl;
	}
	else
		std::cout << std::endl << "Loading was completed in " << microSeconds.count() << " microseconds" << std::endl;
}

int main()
{
	SpeedTest<obj::Load>("C:\\temp\\rungholt.obj");
	SpeedTest<mtl::Load>("C:\\temp\\rungholt.mtl");

	return 0;
}
```
<pre>
File C:\temp\rungholt.obj ..........
Loading was completed in 1164 milliseconds

File C:\temp\rungholt.mtl ..........
Loading was completed in 783 microseconds
</pre>

