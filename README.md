# js-opencv.js-example
Examples of using [opencv.js](https://docs.opencv.org/master/d5/d10/tutorial_js_root.html). 

Link to official [installation guide](https://docs.opencv.org/master/d4/da1/tutorial_js_setup.html).
 
## Prerequisites 

### 1. cmake

[cmake](https://cmake.org/) has to be installed on your machine.

#### For macOS

> [Homebrew](https://brew.sh/) (also known as brew) has to be installed.

Execute in terminal:
```
brew install cmake
```  

Export path by executing:
```
export PATH="/usr/local/bin:$PATH"
```

#### For other platforms

Follow official [installation guide](https://cmake.org/install/).

### 2. emscripten

[emscripten](https://kripken.github.io/emscripten-site/docs/introducing_emscripten/about_emscripten.html) is used to compile C and C++ code into JavaScript. 
OpenCV is written in C++ that's why we need it.

Link to official [installation guide](https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html).

#### For macOS and Unix (Ubuntu)

Get the emsdk repo: 
```
git clone https://github.com/juj/emsdk.git
```

Go to the downloaded folder: 
```
cd emsdk
```

Download and install the latest SDK tools:
 ```
 ./emsdk install latest
 ```

Make the "latest" SDK "active" for the current user. (writes ~/.emscripten file):
```
./emsdk activate latest
```

Activate PATH and other environment variables in the current terminal: 
```
source ./emsdk_env.sh
```

> Note that PATH and other environment variables will be set only in the current terminal, if you close the terminal, you will have to run the command again.

### 3. Python v3 or higher

Python will be used for building OpenCV.js file. Building with version 2.7.15 turned out to show errors during building the file. 
It is recommended to use version higher than 3.0. 

> In [one of many installation guides](https://www.pyimagesearch.com/2018/08/17/install-opencv-4-on-macos/) it was mentioned to use exactly version 3.6 instead of 3.7 or the latest. My OpenCV.js 
file was built using 3.7.2 without any issues.

#### For macOS

> [Homebrew](https://brew.sh/) (also known as brew) has to be installed.

Execute in terminal:
```
brew install python3
```

Check the version (note that "V" is capital):
```
python3 -V
```

> When you need to use python v3 use `python3 some_command` instead of `python some_command`.

## Building OpenCV.js file

#### 1. Ensure `emscripten` environment is setup correctly

Set up PATH and other environment variables by executing:
```
source ./emsdk_env.sh
``` 
then check that the environment is setup by:
```
echo ${EMSCRIPTEN}
```

The output should be something like:  
```
/Users/n.riabchenko/tools/emsdk/emscripten/1.38.24
```

#### 2. Download source code of OpenCV

Go to the [download page](https://opencv.org/releases.html) and download `Sources` of the latest release.

Unpack the downloaded zip file.

#### 3. Go to the unpacked folder

The latest version of the OpenCV for the moment of writing this guide was 4.0.1, so the unpacked folder had name `opencv-4.0.1`.

To go to the folder run 
```
cd opencv-4.0.1
```

#### 4. Build the OpenCV.js

In the folder of OpenCV sources run: 
```
python3 ./platforms/js/build_js.py build_js
``` 

Instead of `build_js` you can pass any name of the folder you want to compile the OpenCV to. 

#### 5. Get opencv.js file

The files are stored in `./build_js/bin` folder. 

## Installing project dependencies

In the root of the project execute:
```
npm install
```

## One interesting thing to keep in mind

To check if the file was built successfully you can run `test.js` with the following code:

```javascript
const cv = require('./opencv.js');

let mat = new cv.Mat();
console.log(mat.size());
mat.delete();
```

To run the file you have to put `test.js` into the folder with `opencv.js`. Then run:
```
node test.js
```

You will get the error:
```
TypeError: cv.Mat is not a constructor
    at Object.<anonymous> (/Users/n.riabchenko/tools/opencv-4.0.1/build_js/bin/test.js:14:11)
    at Module._compile (module.js:653:30)
    at Object.Module._extensions..js (module.js:664:10)
    at Module.load (module.js:566:32)
    at tryModuleLoad (module.js:506:12)
    at Function.Module._load (module.js:498:3)
    at Function.Module.runMain (module.js:694:10)
    at startup (bootstrap_node.js:204:16)
    at bootstrap_node.js:625:3
```

We need to add some wait.

#### Solution #1

```javascript
const cv = require('./opencv.js');

setTimeout(()=>{
        (()=>{
                let mat = new cv.Mat();
                console.log(mat.size());
                mat.delete();
        })();
},10000);
```

This time there will be no errors:

```
{ width: 0, height: 0 }
```

#### Solution #2

Solution #1 was just a typical timeout which is not an efficient solution. 

`emscipten`, which was used to compile C++ sources of OpenCV, emits `onRuntimeInitialized` when it's ready to call the main function. 
So it better to use the code below.

```javascript
const cv = require("./opencv.js");

cv['onRuntimeInitialized']=()=>{
          let mat = new cv.Mat();
          console.log(mat.size());
          mat.delete();
};
```

We can check that there are no error when running the file:

```
{ width: 0, height: 0 }
```

The issue was found by [Junichi Kajiwara](https://dev.to/kjunichi) and described [here](https://dev.to/kjunichi/requireopencvjs-is-not-enough-for-using-opencvjs-8ff).

### Possible issues

Some additional libraries may be required. According to the [article](https://www.pyimagesearch.com/2018/08/17/install-opencv-4-on-macos/) you might need to install:
- jpeg
- libpng
- libtiff 
- openexr 
- eigen
- tbb

#### For macOS

> [Homebrew](https://brew.sh/) (also known as brew) has to be installed.

To install the libraries mentioned above execute:  
```
brew install jpeg libpng libtiff openexr eigen tbb
```
