#  Chilitags.js
Chilitags.js is a direct port of Chilitags ([https://github.com/chili-epfl/chilitags](https://github.com/chili-epfl/chilitags)) to JavaScript, using Emscripten.

Chilitags.js was developed internally for projects of the [CHILI lab](http://chili.epfl.ch/) (Computer-Human Interaction in Learning and Instruction, formerly CRAFT).

##  Content
This repository of Chilitags.js consists of two components:
* the library itself.
* `src/`, source codes to make chilitags.js.
    * `src/jschilitags.cpp`: C wrapper of Chilitags C++ codes.
    * `src/chilitags-javascript.js`: JavaScript snippet of end-user API.
* `samples/`, the sample programs illustrating how to use the library.

##  How to build
### Install Emscripten
Refer to [Emscripten install documentation](https://github.com/kripken/emscripten/wiki/Emscripten-SDK)

We assume hereafter that `emscripten` is installed in `$EMSCRIPTEN_ROOT` (and
that `$EMSCRIPTEN_ROOT` is in your path, ie, you can call `emcc` and `em++`
from anywhere).



### Build OpenCV
* Build OpenCV with emscripten
```
$ cd build
$ emconfigure cmake -DCMAKE_INSTALL_PREFIX=$EMSCRIPTEN_ROOT/system ..
```

* Set `CMAKE_BUILD_TYPE=None`, `CMAKE_CXX_FLAGS="-O2 -DNDEBUG"`
* Set `CMAKE_INSTALL_PREFIX=EMSCRIPTEN_ROOT/system`
* It is recommended that all flags except `BUILD_SHARED_LIBS`, `BUILD_opencv_calib3d`, `BUILD_opencv_core`, `BUILD_opencv_features2d`, `BUILD_opencv_flann`, `BUILD_opencv_highgui`, `BUILD_opencv_imgproc`, `BUILD_opencv_ts`, `ENABLE_OMIT_FRAME_POINTER` are set to OFF.

Then:
```
$ make -j4 && make install
```
### Build chilitags.js

First install and compile `chilitags`:

```
$ git clone https://github.com/chili-epfl/chilitags.git
$ cd chilitags
$ mkdir build-emcc && cd build-emcc
$ emconfigure cmake -DCMAKE_INSTALL_PREFIX=$EMSCRIPTEN_ROOT/system ..
$ make -j4 && make install
```

Then:
```
$ git clone https://github.com/chili-epfl/AR.js.git
$ cd AR.js/chilitags.js
$ mkdir build-emcc && cd build-emcc
$ em++ -std=c++11 -O2 -s OUTLINING_LIMIT=40000 ../src/jschilitags.cpp -lchilitags -lopencv_core -lopencv_imgproc -lopencv_calib3d -o chilitags.js -s EXPORTED_FUNCTIONS="['_setCameraConfiguration', '_getProjectionMatrix', '_setMarkerConfig', '_findTagsOnImage', '_get3dPosition']" --post-js ../src/chilitags-javascript.js
```
##  API documentation

### 2D detection
#### Chilitags.findTagsOnImage(canvas, drawLine)
* canvas: `object` (`<canvas>` element)
* drawLine: `bool`

Returns the object that includes pairs of tag ID and array of positions of its corners.

### 3D detection
#### Chilitags.get3dPose(canvas, rectification)
* canvas: `object` (`<canvas>` element)
* rectification: `bool`

Returns the object that includes pairs of tag name and its transformation matrix.

#### Chilitags.getProjectionMatrix(width, height, near, far)
* width: `float`
* height: `float`
* near: `float`
* far: `float`

Returns `Float32Array` (16 elements) of projection matrix of camera.

### Camera calibration
#### Chilitags.setNewCamera(file)
* file: XML/YAML file in OpenCV format(See [Camera calibration With OpenCV](http://docs.opencv.org/doc/tutorials/calib3d/camera_calibration/camera_calibration.html))

Set intrinsic parameters and distortion coefficients of camera.


### Marker configuration
#### Chilitags.setMarkerConfig(file)
* file: YAML file (e.g. [chilitags/share/markers_configuration_sample.yml](https://github.com/chili-epfl/chilitags/blob/master/share/markers_configuration_sample.yml))

Set marker configuration.

