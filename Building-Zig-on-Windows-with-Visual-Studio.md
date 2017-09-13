Zig requires the llvm/clang development libraries (version 5.0).

## Prebuilt Artifacts

* [llvm+clang+lld-5.0.0-win64.tar.xz](https://s3.amazonaws.com/superjoe/temp/llvm%2bclang%2blld-5.0.0-win64.tar.xz) (678 MB) (sha256 c84cf44b11a3425343f8c3d341bd0883116420b1b8068b2b6abc52350d92aae2)

Please consider [donating $1/month](https://www.patreon.com/andrewrk) if you use this link to help cover cost of hosting such a large file.

## Building LLVM/CLANG

Following these build instructions should be sufficient but you can also refer to LLVM's documentation here: https://llvm.org/docs/GettingStartedVS.html

### Requirements

* CMake
* Microsoft Visual Studio 2015, Version 14.0. (download here https://imagine.microsoft.com/en-us/Catalog/Product/101).
  Make sure you have the latest updates installed (Up to Update 3) and that you have the C/C++ compiler.
* Python

### Get the llvm/clang source code.

Option 1: Use Git
```
git clone https://github.com/llvm-mirror/llvm -b release_50
cd llvm/tools
git clone https://github.com/llvm-mirror/clang -b release_50
```

Option 2: Download the sources

### Configure the build

Open a Visual Studio 2015 64-bit Command Prompt. Note that you can convert a normal command prompt to a Visual Studio 2015 command prompt by running
```dos
"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat" amd64
```
`cd` into the llvm source code, then create a directory to build in:
```dos
cd <llvm-source>
mkdir <build-directory>
cd <build-directory>
```

Inside this directory you can use cmake to generate build files.  The default build files for visual studio are msbuild files.  You can either configure/generate the build files using one command line, i.e.
```doc
cmake .. -Thost=amd64 -DCMAKE_INSTALL_PREFIX=<llvm-install-path>
```
or you can run `cmake-gui ..` which allows you to see all the configuration options before generating the build files. The default options should work, however, you may want to use a custom CMAKE_INSTALL_PREFIX which is where llvm will install the final output files.

> Note: if cmake complains it can't find the C/CXX compiler(s), or your build fails immediately with a message saying the build tools for host=amd64 cannot be found, then you can specify them manually by selecting "Specify native compilers" after clicking "Configure" the first time.  The amd64 C/CXX compiler will probably be here "C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64\cl.exe".  Note that even after doing this I still had problems, I had to reconfigure, specify the compilers manually and leave the "toolset" field blank.

When using cmake-gui, run "Configure" to get all the configuration options (make sure to select the "Visual Studio 14 2015 Win64 generator and set the Optional toolset to "host=amd64").  Keep re-running "Configure" until there are no new options (new options are highlighted in red).  Then click "Generate" to generate the build.

### Perform the Build/Install

#### Option 1: Use cmake from the command line
Run the following from the build directory
```dos
:: perform the build
cmake --build .
:: perform the install
cmake --build . --target install
```
#### Option 2: Use msbuild from the command line
Run the following from the build directory
```dos
:: perform the build
msbuild ALL_BUILD.vcxproj
:: perform the install
msbuild INSTALL.vcxproj
```
#### Option 3: Use Visual Studio
* Open llvm.sln
* To Build, in the "Solution Explorer", right-click "ALL_BUILD" and click "Build".
* To Install, in the "Solution Explorer", right-click "INSTALL" and click "Build".

## Building Zig

### Get the source code.

```dos
git clone https://github.com/zig-lang/zig
```

### Configure the build
Open a Visual Studio 2015 64-bit Command Prompt. Note that you can convert a normal command prompt to a Visual Studio 2015 command prompt by running
```dos
"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat" amd64
```
`cd` into the zig repository, then create a directory to build in:
```dos
cd <zig-repo>
mkdir <build-directory>
cd <build-directory>
```

Inside this directory you can use cmake to generate build files.  Zig will need to be able to find where LLVM was installed.  It uses the `find_package` command in cmake.  If you'd like to configure via the command line you can specify the path like this:
```dos
cmake .. -Thost=amd64 -DCMAKE_PREFIX_PATH=<llvm_install_path>/lib/cmake
```
Or course you can also configure the build by running `cmake-gui ..` inside the build directory.  After you run "Configure" you should get an error indicating that the "LLVM" package could not be found, fix this by clicking "Add Entry" and add the variable `CMAKE_PREFIX_PATH` and give it the value `<LLVM_INSTALL_PATH>/lib/cmake`.  Click "Configure" again and you should be ready to "Generate" the build files and start the build (see [Perform the Build/Install](#perform-the-buildinstall) above)