### Troubleshooting

#### Dual-Abi Linking

If you get one of these:

```
undefined reference to `_ZNK4llvm17SubtargetFeatures9getStringB5cxx11Ev'
undefined reference to `llvm::SubtargetFeatures::getString() const'
```

This is because of
[C++'s Dual ABI](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html).
Most likely LLVM was compiled with one compiler while Zig was compiled with a
different one, for example GCC vs clang.

LLVM, Clang, and Zig must all be compiled with the same C++ compiler.

#### LLVM-6.0 Libxml2 Linking

 * This has been fixed upstream and will be solved with the release of LLVM 6.0.1. Read on for the workaround.

If you get the following:

```
/home/me/local/lib/libLLVMWindowsManifest.a(WindowsManifestMerger.cpp.o): In function `stripComments(_xmlNode*) [clone .isra.7]':
WindowsManifestMerger.cpp:(.text._ZL13stripCommentsP8_xmlNode.isra.7+0x53): undefined reference to `xmlUnlinkNode'
WindowsManifestMerger.cpp:(.text._ZL13stripCommentsP8_xmlNode.isra.7+0x5e): undefined reference to `xmlFreeNode'
WindowsManifestMerger.cpp:(.text._ZL13stripCommentsP8_xmlNode.isra.7+0xbe): undefined reference to `xmlUnlinkNode'
WindowsManifestMerger.cpp:(.text._ZL13stripCommentsP8_xmlNode.isra.7+0xc9): undefined reference to `xmlFreeNode'
WindowsManifestMerger.cpp:(.text._ZL13stripCommentsP8_xmlNode.isra.7+0x10e): undefined reference to `xmlUnlinkNode'
WindowsManifestMerger.cpp:(.text._ZL13stripCommentsP8_xmlNode.isra.7+0x119): undefined reference to `xmlFreeNode'
/home/me/local/lib/libLLVMWindowsManifest.a(WindowsManifestMerger.cpp.o): In function `searchOrDefine(unsigned char const*, _xmlNode*)':
WindowsManifestMerger.cpp:(.text._ZL14searchOrDefinePKhP8_xmlNode+0x19f): undefined reference to `xmlNewNs'
/home/me/local/lib/libLLVMWindowsManifest.a(WindowsManifestMerger.cpp.o): In function `llvm::windows_manifest::WindowsManifestMerger::WindowsManifestMergerImpl::~WindowsManifestMergerImpl()':
WindowsManifestMerger.cpp:(.text._ZN4llvm16windows_manifest21WindowsManifestMerger25WindowsManifestMergerImplD2Ev+0x20): undefined reference to `xmlFreeDoc'
WindowsManifestMerger.cpp:(.text._ZN4llvm16windows_manifest21WindowsManifestMerger25WindowsManifestMergerImplD2Ev+0x36): undefined reference to `xmlFree'
```

This is because of an unspecified libxml2 linker argument requirement for llvm.

Add the following to your CMakeLists.txt:

```
diff --git a/CMakeLists.txt b/CMakeLists.txt
index 3c882780..c26be0c6 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -707,6 +707,7 @@ target_link_libraries(zig LINK_PUBLIC
     ${CLANG_LIBRARIES}
     ${LLD_LIBRARIES}
     ${LLVM_LIBRARIES}
+    xml2
     ${CMAKE_THREAD_LIBS_INIT}
 )
 if(ZIG_DIA_GUIDS_LIB)
```

If you want to run the compiler tests, you will also need to modify the build script as follows:

```
diff --git a/build.zig b/build.zig
index 0a7795d6..90dd44ad 100644
--- a/build.zig
+++ b/build.zig
@@ -76,6 +76,7 @@ pub fn build(b: &Builder) !void {
     }
 
     exe.linkSystemLibrary("c");
+    exe.linkSystemLibrary("xml2");
 
     b.default_step.dependOn(&exe.step);
```
