#### Zig master branch notable changes since release 0.5.0

##### 2020 March
- breaking: std.os read/write functions + sendfile <sup>[c81345c8a](https://github.com/ziglang/zig/commit/c81345c8aec56a108f6f98001666a1552d65ce85)</sup>
- breaking: std.mem.len no longer takes a type argument <sup>[ef3d761da](https://github.com/ziglang/zig/commit/ef3d761da545a3a72928ed0e0ba3b749a4cb74d8)</sup>
- add new functions to std.mem and deprecate others <sup>[5b26128ba](https://github.com/ziglang/zig/commit/5b26128bacddf594dfe45958a236bfa2459f878b)</sup>

##### 2020 February
- remove `@memberCount`, `@memberName`,  `@memberType`, `@typeId`, `@IntType` and `@ArgType` from the language <sup>[#4547](https://github.com/ziglang/zig/pull/4547)</sup>
- remove `@bytesToSlice`, `@sliceToBytes` from the language <sup>[#4516](https://github.com/ziglang/zig/pull/4516)</sup>
- sub-architecture annihilation <sup>[#4509](https://github.com/ziglang/zig/pull/4509)</sup>
- remove `std.io.readLine` <sup>[#4514](https://github.com/ziglang/zig/pull/4514)</sup>
- self-hosted libc and dynamic linker detection <sup>[#4478](https://github.com/ziglang/zig/pull/4478)</sup>
  - move `std.fs.File.access` to `std.fs.Dir.access`. The API now encourages use with an open directory handle.
  - Add std.os.faccessat and related functions.
  - Deprecate the "C" suffix naming convention for null-terminated parameters. "C" should be used when it is related to libc. However null-terminated parameters often have to do with the native system
ABI rather than libc. "Z" suffix is the new convention. For example, `std.os.openC` is deprecated in favour of `std.os.openZ`.
  - Add `std.mem.dupeZ` for using an allocator to copy memory and add a null terminator.
  - Introduce `std.event.Batch`. This API allows expressing concurrency without forcing code to be async. It requires no Allocator and does not introduce any failure conditions. However it is not thread-safe.
  - `std.os.AccessError` gains `FileBusy`, `SymLinkLoop`, and `ReadOnlyFileSystem`. Previously these error codes were all reported as `PermissionDenied`.

##### 2020 January

- rework and improve some of the zig build steps
<sup>[a690a5085](https://github.com/ziglang/zig/commit/a690a5085ddbfb540cf07db146645a9f8a4e92f6)</sup>

- json: disallow overlong and out-of-range UTF-8
<sup>[2933a8241](https://github.com/ziglang/zig/commit/2933a8241a54af436f2df5eac73aa2acf5eabd40)</sup>

- `std.mem.compare`: breaking API changes
<sup>[5575e2a16](https://github.com/ziglang/zig/commit/5575e2a168c07d2dcc0e58146231e490ef8a898e)</sup>

##### 2019 December

- breaking API changes to all readInt/writeInt functions & more
<sup>[b883bc873](https://github.com/ziglang/zig/commit/b883bc873df7f1a8fa3a13800402e1ec8da74328)</sup>

- replace var args with [anonymous list literals](https://ziglang.org/documentation/master/#Anonymous-List-Literals)
<sup>[a3f6a58c7](https://github.com/ziglang/zig/commit/a3f6a58c7785e7958f9d0b96d54356944bf34e32)</sup>
  - see [hello world](https://ziglang.org/documentation/master/#Hello-World)

- JSON unescape
<sup>[6af39aa49](https://github.com/ziglang/zig/commit/6af39aa49afeb3498d6c5dfa0b60a0fdc15ca47c)</sup>

- rename `@typeOf` → [`@TypeOf`](https://ziglang.org/documentation/master/#TypeOf)
<sup>[f0ee0688f](https://github.com/ziglang/zig/commit/f0ee0688f20dd012b4e069324abdba081ff19369)</sup>

##### 2019 November
- remove type coercion from array values to references
<sup>[bf3ac6615](https://github.com/ziglang/zig/commit/bf3ac6615051143a9ef41180cd74e88de5dd573d)</sup>

- string literals are now null terminated and special syntax for C interoperable string literals (eg. `c"hello"`) is no longer necessary and has been dropped
<sup>[47f06be36](https://github.com/ziglang/zig/commit/47f06be36943f808aa9798c19172363afe6ae35c)</sup>
  - see [string literals](https://ziglang.org/documentation/master/#String-Literals-and-Character-Literals)
  - see [sentinel terminated slices](https://ziglang.org/documentation/master/#Sentinel-Terminated-Slices)
  - see [sentinel terminated pointers](https://ziglang.org/documentation/master/#Sentinel-Terminated-Pointers)
  - see [sentinel terminated arrays](https://ziglang.org/documentation/master/#Sentinel-Terminated-Arrays)

- add [`@as`](https://ziglang.org/documentation/master/#as) for type coercion and drop `type(value)` syntax
<sup>[6d5abf87e](https://github.com/ziglang/zig/commit/6d5abf87ecd3509c6fb8b9c917b73b4db2ae59ff)</sup>

##### 2019 October

- improve `std.fs` directory handling API
<sup>[5b1a49201](https://github.com/ziglang/zig/commit/5b1a492012241276a4b7539ca6664234f0629c79)</sup>

- standardize std.os execve functions
<sup>[8cf3a4d58](https://github.com/ziglang/zig/commit/8cf3a4d586b675a239c9cfa1ea07fa9f59ebf0a4)</sup>
