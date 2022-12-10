Install CMake and Git by opening PowerShell in Administrator Mode and running these commands:

```
winget install --id Git.Git -e --source winget
winget install --id Kitware.CMake -e --source winget
```

Cross-compile Ninja for aarch64-windows:

```
git clone https://github.com/ninja-build/ninja
cd ninja
git checkout v1.11.1
```

Apply this patch:

```diff
diff --git a/CMakeLists.txt b/CMakeLists.txt
index 70fc5e9..8b90dad 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -5,15 +5,7 @@ include(CheckIPOSupported)
 
 project(ninja)
 
-# --- optional link-time optimization
-check_ipo_supported(RESULT lto_supported OUTPUT error)
-
-if(lto_supported)
-	message(STATUS "IPO / LTO enabled")
-	set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE TRUE)
-else()
-	message(STATUS "IPO / LTO not supported: <${error}>")
-endif()
+set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE FALSE)
 
 # --- compiler flags
 if(MSVC)
```

```
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$(pwd)/dist -DCMAKE_AR="$HOME/local/llvm15-release/bin/llvm-ar" -DCMAKE_CXX_COMPILER_AR="$HOME/local/llvm15-release/bin/llvm-ar" -DCMAKE_RANLIB="$HOME/local/llvm15-release/bin/llvm-ranlib" -DCMAKE_CXX_COMPILER_RANLIB="$HOME/local/llvm15-release/bin/llvm-ranlib" -DCMAKE_C_COMPILER_AR="$HOME/local/llvm15-release/bin/llvm-ar" -DCMAKE_C_COMPILER_RANLIB="$HOME/local/llvm15-release/bin/llvm-ranlib" -DCMAKE_CROSSCOMPILING=True -DCMAKE_SYSTEM_NAME=Windows -DCMAKE_RC_COMPILER="$HOME/local/llvm15-release/bin/llvm-rc"
make install
```

This produces `dist/bin/ninja.exe` which you can send to your Windows machine with [WebWormhole](https://webwormhole.io/).

You must then put ninja.exe into one of the directories in the system-wide `Path` environment variable (or add a new directory to that environment variable if necessary). This is how CMake detects the existence of Ninja.