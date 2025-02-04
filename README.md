# Skia for Aseprite and laf

**Pre-built binaries of Skia** in the [releases page](https://github.com/aseprite/skia/releases).

Skia is a 2D graphic library developed by Google Inc., you can find
the official website in [skia.org](https://skia.org).

This fork is used to compile Skia automatically for
[laf](https://github.com/aseprite/laf) and
[Aseprite](https://github.com/aseprite/aseprite) using GitHub Actions.

# Building Skia

In the following sections you will find straightforward steps to
compile Skia. You can always check the [official Skia
instructions](https://skia.org/docs/user/build) and select the OS you are
building for. [Aseprite](https://github.com/aseprite/aseprite) and
[laf](https://github.com/aseprite/laf) use the **`aseprite-m122`** branch.
So remember to checkout that specific branch.

These are the platform-specific steps to compile Skia:

* [Skia on Windows](#skia-on-windows)
* [Skia on macOS](#skia-on-macos)
* [Skia on Linux](#skia-on-linux)

After this you should have all Skia libraries compiled. After that,
when you compile laf or Aseprite remember to add
`-DSKIA_DIR=$HOME/deps/skia` parameter to your `cmake` call and all
other parameters.

## Skia on Windows

Download
[Google depot tools](https://storage.googleapis.com/chrome-infra/depot_tools.zip)
and uncompress it in some place like `C:\deps\depot_tools`.

[It's recommended to compile Skia with Clang](https://github.com/google/skia/blob/master/site/user/build.md#a-note-on-software-backend-performance)
to get better performance. So you will need to [download Clang](https://github.com/llvm/llvm-project/releases/download/llvmorg-17.0.6/LLVM-17.0.6-win64.exe),
and install it on a folder like `C:\deps\llvm` (a folder without whitespaces).

Open a command prompt window (`cmd.exe`) and call:

    call "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\VsDevCmd.bat" -arch=x64

The command above is required while attempting to compile the 64-bit version of skia. When compiling the 32-bit version, it is possible to open a [developer command prompt](https://docs.microsoft.com/en-us/dotnet/framework/tools/developer-command-prompt-for-vs) instead.

Then:

    set PATH=C:\deps\depot_tools;%PATH%
    cd C:\deps\depot_tools
    gclient sync

(The `gclient` command might print an error like
`Error: client not configured; see 'gclient config'`.
Just ignore it.)

    cd C:\deps
    git clone -b aseprite-m122 https://github.com/aseprite/skia.git
    cd skia
    set GIT_EXECUTABLE=git.bat
    python3 tools/git-sync-deps
    python3 bin/fetch-ninja

(The `tools/git-sync-deps` will take some minutes because it downloads
a lot of packages, please wait and re-run the same command in case it
fails.)

Finally, if you've downloaded Clang, use this command:

    set PATH=C:\deps\llvm\bin;%PATH%
    gn gen out/Release-x64 --args="is_debug=false is_official_build=true is_trivial_abi=false skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_sfntly=false skia_use_freetype=true skia_use_harfbuzz=true skia_pdf_subset_harfbuzz=true skia_use_system_freetype2=false skia_use_system_harfbuzz=false target_cpu=""x64"" cc=""clang"" cxx=""clang++"" clang_win=""c:\deps\llvm"" clang_win_version=""17.0.6"" win_vc=""C:\Program Files\Microsoft Visual Studio\2022\Community\VC"" extra_cflags=[""-MT""]"
    ninja -C out/Release-x64 skia modules

If you haven't installed Clang, and want to compile Skia with MSVC
(anyway it's not recommended because the performance penalty is too
big), you can use the following commands instead:

    gn gen out/Release-x64 --args="is_debug=false is_official_build=true is_trivial_abi=false skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_sfntly=false skia_use_freetype=true skia_use_harfbuzz=true skia_pdf_subset_harfbuzz=true skia_use_system_freetype2=false skia_use_system_harfbuzz=false target_cpu=""x64"" win_vc=""C:\Program Files\Microsoft Visual Studio\2022\Community\VC"" extra_cflags=[""-MT""]"
    ninja -C out/Release-x64 skia modules

### For debugging

For debugging purposes you can compile the debug version of the
library, e.g. for LLVM:

    set PATH=C:\deps\llvm\bin;%PATH%
    gn gen out/Debug-x64 --args="is_debug=true is_official_build=false is_trivial_abi=false skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_sfntly=false skia_use_freetype=true skia_use_harfbuzz=true skia_pdf_subset_harfbuzz=true skia_use_system_freetype2=false skia_use_system_harfbuzz=false target_cpu=""x64"" cc=""clang"" cxx=""clang++"" clang_win=""c:\deps\llvm"" clang_win_version=""17.0.6"" win_vc=""C:\Program Files\Microsoft Visual Studio\2022\Community\VC"" extra_cflags=[""-MTd""]"
    ninja -C out/Debug-x64 skia modules

## Skia on macOS

These steps will create a `deps` folder in your home directory with a
couple of subdirectories needed to build Skia (you can change the
`$HOME/deps` with other directory). Some of these commands will take
several minutes to finish:

    mkdir $HOME/deps
    cd $HOME/deps
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    git clone -b aseprite-m122 https://github.com/aseprite/skia.git
    export PATH="${PWD}/depot_tools:${PATH}"
    cd skia
    python3 tools/git-sync-deps
    python3 bin/fetch-ninja
    gn gen out/Release-x64 --args="is_debug=false is_official_build=true skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_sfntly=false skia_use_freetype=true skia_use_harfbuzz=true skia_pdf_subset_harfbuzz=true skia_use_system_freetype2=false skia_use_system_harfbuzz=false target_cpu=\"x64\" extra_cflags=[\"-stdlib=libc++\", \"-mmacosx-version-min=10.9\"] extra_cflags_cc=[\"-frtti\"]"
    ninja -C out/Release-x64 skia modules

### Skia on macOS for Apple Silicon

If you want to compile Skia for Apple Silicon (e.g. M1), you have to
specify the `arm64` CPU architecture:

    gn gen out/Release-arm64 --args="is_debug=false is_official_build=true skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_sfntly=false skia_use_freetype=true skia_use_harfbuzz=true skia_pdf_subset_harfbuzz=true skia_use_system_freetype2=false skia_use_system_harfbuzz=false target_cpu=\"arm64\" extra_cflags=[\"-stdlib=libc++\", \"-mmacosx-version-min=11.0\"] extra_cflags_cc=[\"-frtti\"]"
    ninja -C out/Release-arm64 skia modules

## Skia on Linux

These steps will create a `deps` folder in your home directory with a
couple of subdirectories needed to build Skia (you can change the
`$HOME/deps` with other directory). Some of these commands will take
several minutes to finish:

    mkdir $HOME/deps
    cd $HOME/deps
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    git clone -b aseprite-m122 https://github.com/aseprite/skia.git
    export PATH="${PWD}/depot_tools:${PATH}"
    cd skia
    python3 tools/git-sync-deps
    python3 bin/fetch-ninja
    gn gen out/Release-x64 --args="is_debug=false is_official_build=true skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false skia_use_sfntly=false skia_use_freetype=true skia_use_harfbuzz=true skia_pdf_subset_harfbuzz=true skia_use_system_freetype2=false skia_use_system_harfbuzz=false"
    ninja -C out/Release-x64 skia modules

# Internal Details

* `is_trivial_abi`: As we are mixing MSVC/Clang, the Windows
  compilation needs the `is_trivial_abi=false` option as indicated
  [here](https://groups.google.com/g/skia-discuss/c/ze-_PUljtkk/m/sdTNWL3rAQAJ),
  in other case we can get an infinite loop `SkOnce::operator()`

* `-MT`: We are creating a static-linked version of Skia library,
  that's why we have the `extra_cflags=["-MTd"]` option, so we don't
  need to distribute the MSVC C++ runtime DLLs.

* `-frtti`: On macOS we've detected a [strange bug](https://bugs.llvm.org/show_bug.cgi?id=37487)
  where linking object files compiled with `-frtti` and `-fno-rtti`
  (the default Skia option) breaks the normal function of
  `std::exception` hierarchy. That's why we're compiling with run-time
  type information `extra_cflags_cc=["-frtti"]`.
