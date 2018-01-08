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
            jniLibs.srcDirs = ['src/main/jniLibs']
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