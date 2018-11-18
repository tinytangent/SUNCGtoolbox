# SUNCG Data Documentation 
We provide a simple C++ based toolbox for the SUNCG dataset.  
Please see our [webpage](http://suncg.cs.princeton.edu) and [paper](https://arxiv.org/pdf/1611.08974v1.pdf) for more details about the data.

If you are a researcher and would like to get access to the data, please print and sign this [agreement](http://suncg.cs.princeton.edu/form.pdf)  and email it to suncgteam@googlegroups.com. 

### Bibtex

If you find SUNCG useful in your research, please consider citing:

    @article{song2016ssc, 
        title= {Semantic Scene Completion from a Single Depth Image}, 
        author= {Song, Shuran and Yu, Fisher and Zeng, Andy and Chang, Angel X and Savva, Manolis and Funkhouser, Thomas}, 
        journal={Proceedings of 29th IEEE Conference on Computer Vision and Pattern Recognition}, 
        year={2017} 
    }


### Contents
0. [Data Organization](#data-organization)
0. [Data Format](#data-format)
0. [C++ Toolbox](#c++-toolbox)
0. [Basic Functionalities](#basic-functionalities)
0. [Resources](#resources)
0. [Simulation evironments](#simulation-evironments-support-suncg)

### Data Organization 
The downloaded and unzipped SUNCG files should be organized as follows:


[!Important!]: If you are running the download script with default arguments, latest release with no room architectures will be downloaded. Run ./download_suncg.py -v v1 -t room to download room architectures.

```shell
data_root
    |-- house
        |-- <sceneid>
            |-- house.json
    |-- room
        |-- <sceneid>
            |- fr_0rm_0c.mtl    
            |- fr_0rm_0c.obj
            ...
    |-- object
        |-- <objectid>
            |--objectid.obj
    |-- texture
```


### Data Format
Each 3D scene is saved as "house.json", and structured as follows:

house.json
- id 
- front
- up
- scaleToMeters
- levels : array of level
    - id
    - bbox: axis aligned bounding box of level 
       - “min”
       - “max”
    - nodes: array of nodes
        - "id": string of the form “floorIdx_itemIdx” (floorIdx is the index of floor, itemIdx is the index of node)
        - "type": indicates one of the following four node types: Object, Box, Ground, Room
        - "valid" : boolean (1 or 0) - skip loading for this object if it is false (0)
        - "modelId" : corresponding model name for loading obj file 
        - "material": array of materials with the following fields:
          - name
          - texture
          - color 
        - "dimensions": dimensions of box in scene space units for Box only
        - "transform": 4x4 column-major transformation matrix from object coordinates to scene coordinates
        - "isMirrored": boolean variable indicating whether we need to reverse normals due to flipping
        - "roomTypes": for room nodes only (array of strings) 
        - "hideCeiling": boolean
        - "hideFloor": boolean 
        - "hideWalls": boolean 
        - "nodeIndices": indicates (itemIdx) which nodes are belong to this room starts with zero
        - "state": different state from base model 0=open, 1=closed




### C++ Toolbox
The code is organized as follows:

```shell
gaps
    |-- pkgs - source and include files for all packages (software libraries)
    |-- apps - source files for several applications and example programs
    |-- makefiles - unix-style make file definitions
    |-- vc - visual studio solution files
    |-- lib - archive library (.lib) files (created during compilation)
    |-- bin - executable files (created during compilation)
metadata
    |-- ModelCategoryMapping.csv
    |-- suncgModelLights.json
```


###  Compilation
Compile with OpenGL, it will use GPU for rendering:

```shell
cd gaps
make clean 
make 
```

Compile with OSMesa, it will use CPU for off-screen rendering:

```shell
cd gaps
make clean 
make mesa
```


###  Basic Functionalities 

#### Viewing the scene (requires compiling with GPU)

```shell
cd data_root/house/<sceneid>/
gaps/bin/x86_64/scnview house.json -v 
```

Camera Controls
- Left click: Orbit view
- Right click: Pan view 
- Mouse wheel: Zoom view
- Double click on an object to change the viewer's rotation center

Other function keys
- "b": show back faces
- "h": show bounding boxes
- "c": show cameras
- "e": show edges
- "a": show axes
- "Esc": exit program
- "Space": print out current camera 

  
#### Convert to OBJ+MTL

```shell
cd data_root/house/<sceneid>/
gaps/bin/x86_64/scn2scn house.json house.obj
```

It will write out "house.obj" and "house.mtl" in the current folder.


#### Generating Cameras

```shell
cd data_root/house/<sceneid>/
gaps/bin/x86_64/scn2cam house.json outputcamerasfile  -categories ModelCategoryMapping.csv  -v
```

Input options
    -width: Output image width
    -height: Output image height
    -xfov: Camera FOV
    -eye_height: Camera height

Use the following command to check the generated camera file. 
Press key "c" to show the camera. Press key "v" to go though each view point.

```shell
gaps/bin/x86_64/scnview house.json -cameras outputcamerasfile
```


#### Render Images and Ground Truth

```shell
cd data_root/house/<sceneid>/
gaps/bin/x86_64/scn2img house.json inputcamerasfile outputimagedirectory -categories ModelCategoryMapping.csv  -v
```

The program will generate the following files, you can use the corresponding flags to choose which files to generate:


- capture_color_images: OpenGL rendering of RGB image saved as JPEG.
- capture_depth_images: Depth image saved as uint16 PNG
- capture_kinect_images: Depth image with simulated Kinect noise saved as uint16 PNG.
- capture_normal_images: Normal map saved as 3 uint16 PNG image for x y z.
- capture_albedo_images: Albedo image saved as JPEG.
- capture_brdf_images: Brdf image saved as JPEG.
- capture_material_images: Material index map saved as PNG.
- capture_node_images: Node index map (instance segmentation).
- capture_category_images: Object category map saved as PNG (need  input "-categories ModelCategoryMapping.csv" to map from modelId to index).
- capture_boundary_images: Combine boundary map with node boundaries (first bit), silhouette boundaries (second bit), where depth difference is > 10% of depth (third bit), crease boundaries (fourth bit).



### Resources 
Room category:
living room, kitchen, bedroom, child room, dining room, bathroom, 
toilet, hall, hallway, office, guest room, wardrobe,
room, lobby, storage, boiler room, balcony, loggia, terrace,
entryway, passenger elevator, freight elevator, aeration, garage and gym.

Object category mapping between SUNCG to NYU depth V2 40 object class can be found in ``$data_root/metadata/ModelClassMapping.csv``.


### Simulation evironments support SUNCG
[House3D: A Rich and Realistic 3D Environment](https://github.com/facebookresearch/House3D)
Yi Wu, Yuxin Wu, Georgia Gkioxari and Yuandong Tian 

[MINOS: Multimodal Indoor Simulator for Navigation in Complex Environments](https://github.com/minosworld/minos)
Manolis Savva, Angel X. Chang, Alexey Dosovitskiy, Thomas Funkhouser and Vladlen Koltun 

[HoME: a Household Multimodal Environment](https://github.com/HoME-Platform/home-platform)
Simon Brodeur, Ethan Perez, Ankesh Anand, Florian Golemo, Luca Celotti, Florian Strub, Jean Rouat, Hugo Larochelle, Aaron Courville 


### GAPS README ###
This toolbox is adapted from GAPS written by [Thomas Funkhouser](http://www.cs.princeton.edu/~funk/). Here is the orignal GAPS README:

GAPS Users -

This directory contains all code for the GAPS software library.
There are several subdirectories:

    pkgs - source and include files for all packages (software libraries).
    apps - source files for several application and example programs. 
    makefiles - unix-style make file definitions
    vc - visual studio solution files
    lib - archive library (.lib) files (created during compilation).
    bin - executable files (created during compilation).

If you are using linux or cygwin and have gcc and OpenGL development
libraries installed, or if you are using MAC OS X with the xcode
development environment, you should be able to compile all the code by
typing "make clean; make" in this directory.  If you are using Windows
Visual Studio 10 or later, then you should be able to open the
solution file vc.sln in the vc subdirectory and then "Rebuild
Solution."  For other development platforms, you should edit the shared
compilation settings in the makefiles/Makefiles.std to meet your needs.

To write a program that uses the GAPS pkgs, then you should include
"-I XXX/gaps/pkgs" in your compile flags (CFLAGS) and "-L
XXX/gaps/lib" in your link flags (LDFLAGS), where XXX is the directory
where you installed the gaps software.

The software is distributed under the MIT license (see LICENSE.txt)
and thus can be used for any purpose without warranty, any liability,
or any suport of any kind.

- Tom Funkhouser
