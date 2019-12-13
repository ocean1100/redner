# redner: Differentiable rendering without approximation

![](https://people.csail.mit.edu/tzumao/diffrt/teaser.jpg)

redner is a differentiable renderer that can take the derivatives of rendering output with respect to arbitrary 
scene parameters, that is, you can backpropagate from the image to your 3D scene. One of the major usages of redner is inverse rendering (hence the name redner) through gradient descent. What sets redner apart are: 1) it computes correct rendering gradients stochastically without any approximation and 2) it has a physically-based mode -- which means it can simulate photons and produce realistic lighting phenomena, such as shadow and global illumination, and it handles the derivatives of these features correctly. You can also use redner in a [fast deferred rendering mode](https://github.com/BachiLi/redner/wiki/Tutorial-4%3A-fast-deferred-rendering) for local shading: in this mode it still has correct gradient estimation and more elaborate material models compared to most differentiable renderers out there.

For more details on the renderer, what it can do, and the techniques it use for computing the derivatives, please
take a look at the paper:
[Differentiable Monte Carlo Ray Tracing through Edge Sampling](https://people.csail.mit.edu/tzumao/diffrt/), Tzu-Mao Li, Miika Aittala, Fredo Durand, Jaakko Lehtinen.
Since the submission we have improved the renderer quite a bit. In particular we implemented a CUDA backend and accelerated
the continuous derivatives significantly by replacing automatic differentiation with manually-derived derivatives. See Tzu-Mao Li's [thesis](https://people.csail.mit.edu/tzumao/phdthesis/phdthesis.pdf) for even more details. Also see the "News" section below for the changelog.

## Installation

With either [PyTorch](https://pytorch.org/) (any version >= 1.0) or [TensorFlow](https://www.tensorflow.org/) (version >= 2.0) installed in your current Python environment. For GPU accelerated version (Linux only, CUDA 10.0 required):
```
pip install redner-gpu
```
otherwise (Linux and OS X): 
```
pip install redner
```
You can also build from source. See the [wiki](https://github.com/BachiLi/redner/wiki/Installation) for building instructions.
Preliminary windows support (CPU-only) made by [Markus Worchel](https://github.com/mworchel) can be accessed through building from source.

## Documentation

A good starting point to learn how to use redner is to look at the [wiki](https://github.com/BachiLi/redner/wiki). The API documetation is [here](https://redner.readthedocs.io/en/latest/).
You can also take a look at the tests directories ([PyTorch](tests) and [TensorFlow](tests_tensorflow)) to have some ideas.

## News

12/12/2019 - Preliminary Windows support (CPU-only) is available thanks to the contribution of [Markus Worchel](https://github.com/mworchel).  
12/09/2019 - Added many tutorials in wiki using Google colab. Added a sphinx-generated documentation.  
12/09/2019 - Fixed a bug in the Wavefront obj loader. Thanks Dejan Azinović for reporting!  
12/01/2019 - Redo the build systems and setup Python wheel installation. Redner is now on PyPI ([https://pypi.org/project/redner-gpu/](https://pypi.org/project/redner-gpu/) and [https://pypi.org/project/redner/](https://pypi.org/project/redner/)).  
11/04/2019 - Added a "generic texture" field in the material which can have arbitrary number of channels. This can be useful for deep learning application where the texture is generated by a network, and the output is fed into another network for further processing. See [test_multichannels.py](https://github.com/BachiLi/redner/blob/master/tests/test_multichannels.py) for usages. Thanks [François Ruty](https://github.com/francoisruty) for the suggestion and the help on implementation.   
11/04/2019 - Fixed several bugs in the OBJ/Mitsuba loaders introduced by the `uv_indices` and `normal_indices` change.  
10/19/2019 - Added vertex color support. See [test_vertex_color.py](https://github.com/BachiLi/redner/blob/master/tests/test_vertex_color.py).  
10/08/2019 - Added automatic uv computation through the [xatlas](https://github.com/jpcy/xatlas) library. See the `compute_uvs` function in shape.py and test_compute_uvs.py.  
10/08/2019 - Slightly changed the Shape class interface. The constructor order is different and it now takes extra 'uv_indices' and 'normal_indices' arguments for dealing with seams in UV mapping and obj loading. See pyredner/shape.py and tutorial 2.  
09/22/2019 - We now handle inconsistencies between shading normals and geometry normals more gracefully (instead of just return zero in most cases). This helps with rendering models in the wild, say, models in ShapeNet.  
09/21/2019 - Fixed a serious buffer overrun bug in the deferred rendering code when there is no radiance output channel. If things didn't work for you, maybe try again.  
08/16/2019 - Added docker files for easier installation. Thanks [Seyoung Park](https://github.com/SuperShinyEyes) for the contribution again. Also I significantly improved the [wiki installation guide](https://github.com/BachiLi/redner/wiki).  
08/13/2019 - Added normal map support. See tests/test_teapot_normal_map.py.  
08/10/2019 - Significantly simplified the derivatives accumulation code (segmented reduction -> atomics). Also GPU backward pass got 20~30% speedup.  
08/07/2019 - Fixed a roughness texture bug.  
07/27/2019 - Tensorflow 1.14 support! Currently only support eager execution. We will support graph execution after tensorflow 2.0 becomes stable. See tests_tensorflow for examples (I recommend starting from tests_tensorflow/test_single_triangle.py). The CMake files should automatically detect tensorflow in python and install corresponding files. Tutorials are work in progress. Lots of thanks go to [Seyoung Park](https://github.com/SuperShinyEyes) for the contribution!  
06/25/2019 - Added orthographic cameras (see examples/two_d_mesh.py).  
05/13/2019 - Fixed quite a few bugs related to camera derivatives. If something didn't work for you before, maybe try again.  
04/28/2019 - Added QMC support (see tests/test_qmc.py and the documentation in pyredner.serialize_scene()).  
04/01/2019 - Now support multi-GPU (see pyredner.set\_device).  
03/31/2019 - Brought back the hierarchical edge sampling method in the paper.  
02/02/2019 - The [wiki](https://github.com/BachiLi/redner/wiki) now contains a series of tutorial. The plan is to further expand the examples.  

## Dependencies

redner depends on a few libraries/systems, which are all included in the repository:
- [Python 3.6 or above](https://www.python.org)
- [pybind11](https://github.com/pybind/pybind11)
- [PyTorch 1.0 or above](https://pytorch.org) (optional, required if TensorFlow is not installed)
- [Tensorflow 2.0](https://www.tensorflow.org/) (optional, required if PyTorch is not installed)
- [OpenEXR](https://github.com/openexr/openexr)
- [Embree](https://embree.github.io)
- [CUDA 10](https://developer.nvidia.com/cuda-downloads) (optional, need GPU at Kepler class or newer)
- [optix prime V6.5 or older](https://developer.nvidia.com/optix) (optional, required when compiled with CUDA)
- [OpenEXR Python](https://github.com/jamesbowman/openexrpython)
- [Thrust](https://thrust.github.io)
- [miniz](https://github.com/richgel999/miniz)
- [xatlas](https://github.com/jpcy/xatlas)
- A few other python packages: numpy, scikit-image


## Roadmap

The current development plan is to enhance the renderer. Following features will be added in the near future (not listed in any particular order):
- More BSDFs e.g. glass/GGX
- Support for edge shared by more than two triangles
  (The code currently assumes every triangle edge is shared by at most two triangles.
   If your mesh doesn't satisfy this, you can preprocess it in other mesh processing softwares such as [MeshLab](http://www.meshlab.net))
- Source-to-source automatic differentiation
- Improve mipmapping memory usage, EWA filtering, covariance tracing
- Russian roulette
- Distribution effects: depth of field/motion blur
- Proper pixel filter (currently only support 1x1 box filter)
- Volumetric path tracing (e.g. [http://www.cs.cornell.edu/projects/translucency/#acquisition-sa13](http://www.cs.cornell.edu/projects/translucency/#acquisition-sa13))
- Spectral rendering
- Gradient visualization
- Spherical light sources

If you have any questions/comments/bug reports, feel free to open a github issue or e-mail to the author
Tzu-Mao Li (tzumao@mit.edu)
