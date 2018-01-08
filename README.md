# AndroidCV — JNILibs
This is just a mirror of the OpenCV JNI libs (https://opencv.org) which I use as a git submodule in my projects to forgo the pain of updating manually.

To include build support in your Android Studio module of choice, include the following into your build.gradle: 
```
android {

    defaultConfig {

    	...

        externalNativeBuild {
            cmake {
                cppFlags "-frtti -fexceptions"
                abiFilters 'x86', 'x86_64', 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'mips', 'mips64'
            }
        }
    }

    ...

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_7
        targetCompatibility JavaVersion.VERSION_1_7
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/AndroidCV-JNILibs']
        }
    }

    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}
```

Also include a cpp folder in your src/main.

Make sure that your CMakeLists.txt includes something along these lines:
```

# Sets the minimum version of CMake required to build the native
# library. You should either keep the default value or only pass a
# value of 3.4.0 or lower.

cmake_minimum_required(VERSION 3.4.1)

# OpenCV stuff
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src/main/AndroidCV-JNILibs/include)
add_library( lib_opencv SHARED IMPORTED )
set_target_properties(lib_opencv PROPERTIES IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/src/main/AndroidCV-JNILibs/${ANDROID_ABI}/libopencv_java3.so)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds it for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             native-opencv

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             # Associated headers in the same location as their source
             # file are automatically included.
             src/main/cpp/native-opencv.cpp )



# Searches for a specified prebuilt library and stores the path as a
# variable. Because system libraries are included in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in the
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       native-opencv

                       # OpenCV lib
                       lib_opencv

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

Finally, your C++ file can look like this (inside your src/main/cpp folder) named opencv-native.cpp:
```
#include <jni.h>
#include <string>
#include <opencv2/core/core.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/features2d/features2d.hpp>

using namespace std;
using namespace cv;

extern "C"
{

// More of an example of how Mat modification works in opencv c++
void JNICALL Java_visionanalysis_CryptoboxDetector_salt(JNIEnv *env, jobject instance,
                                                                           jlong matAddrGray,
                                                                           jint nbrElem) {
    Mat &mGr = *(Mat *) matAddrGray;
    for (int k = 0; k < nbrElem; k++) {
        int i = rand() % mGr.cols;
        int j = rand() % mGr.rows;
        mGr.at<uchar>(j, i) = 255;
    }
}

// Gets the mat between a threshold of two mats instead of two scalars.
void JNICALL Java_hankextensions_vision_opencv_OpenCVJNIHooks_inRangeBetweenMatsNative(JNIEnv *env, jobject instance, jlong toFilterAddr, jlong lowerAddr, jlong upperAddr, jlong destAddr)
{
    inRange(*(Mat *) toFilterAddr, *(Mat *) lowerAddr, *(Mat *) upperAddr, *(Mat *) destAddr);
}

}
```