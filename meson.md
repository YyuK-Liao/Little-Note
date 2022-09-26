## WHY
雖然大學已經畢業了，也寫過大大小小的C++專案，但當時只追求「能動就好」，在這之前完全沒考慮過跨機器、跨平台的需求。

大學專題時我用C++寫的推理應用，也是這段時間最大的一個C++專案了，當時在Windows上用vcpkg和VS2019等工具來開發，手動設定了CUDA、TensorRT、OpenCV、gRPC等等的函式庫，這種作法也完全不具遷移性，也因此現在在學習自動構建的工具。

當時是在reddit找，看到滿多人吐槽CMake的語法太噁心，另外推薦了Meson，所以我也就開始學Meson了。

---
這篇是伴隨著[YyuK-Liao/BRA](https://github.com/YyuK-Liao/BRA)的C++版開發而開始的，所以紀錄的都是重頭開始架構Meson C++ BRA。
### HOW
#### 1. init
在專案開始時我們需要Meson所需要的文件->meson.build，類似CMake的CMakeList.txt，可以手動創建，也可以使用```meson init <args>```指令創建，參數如下：
```
$ meson init --help
usage: meson init [-h] [-C WD] [-n NAME] [-e EXECUTABLE] [-d DEPS]
                  [-l {c,cpp,cs,cuda,d,fortran,java,objc,objcpp,rust,vala}] [-b] [--builddir BUILDDIR] [-f]
                  [--type {executable,library}] [--version VERSION]
                  [sourcefile [sourcefile ...]]

positional arguments:
  sourcefile                                                source files. default: all recognized files in current
                                                            directory

optional arguments:
  -h, --help                                                show this help message and exit
  -C WD                                                     directory to cd into before running
  -n NAME, --name NAME                                      project name. default: name of current directory
  -e EXECUTABLE, --executable EXECUTABLE                    executable name. default: project name
  -d DEPS, --deps DEPS                                      dependencies, comma-separated
  -l {c,cpp,cs,cuda,d,fortran,java,objc,objcpp,rust,vala}, --language {c,cpp,cs,cuda,d,fortran,java,objc,objcpp,rust,vala}
                                                            project language. default: autodetected based on source
                                                            files
  -b, --build                                               build after generation
  --builddir BUILDDIR                                       directory for build
  -f, --force                                               force overwrite of existing files and directories.
  --type {executable,library}                               project type. default: executable based project
  --version VERSION                                         project version. default: 0.1
```
最常用的大概是```-l```來指定專案所使用的程式語言，第二常用的大概是```-b```指定一個編譯用環境。

需要注意，```meson init```有一個```sourcefile```的參數，如果目標專案的資料夾是空的話，可以不需要```sourcefile```參數，會直接創建一個初始的原始碼檔案，以C++為例，執行```meson init```會產生一個```meson.build```和```${project name}.cpp```。

**但是如果資料夾不為空的話，沒有指定```sourcefile```參數會報錯**
```
No recognizable source files found.
Run meson init in an empty directory to create a sample project.
 ```
透過隨便建立一個原始碼文件就可以解決，例如我們可以在專案的任意資料夾中創建一個```work.cpp```，並將該檔案的位置作為參數傳給```meson init xxx/work.cpp```。

#### 2. setup
```meson setup```的作用是「建立個編譯用環境」，也就和```meson init -b```做的事一樣，參數如下：
```
$ meson setup --help
usage: meson setup [-h] [--prefix PREFIX] [--bindir BINDIR] [--datadir DATADIR] [--includedir INCLUDEDIR]
                   [--infodir INFODIR] [--libdir LIBDIR] [--libexecdir LIBEXECDIR] [--localedir LOCALEDIR]
                   [--localstatedir LOCALSTATEDIR] [--mandir MANDIR] [--sbindir SBINDIR]
                   [--sharedstatedir SHAREDSTATEDIR] [--sysconfdir SYSCONFDIR] [--auto-features {enabled,disabled,auto}]                   [--backend {ninja,vs,vs2010,vs2012,vs2013,vs2015,vs2017,vs2019,vs2022,xcode}]
                   [--buildtype {plain,debug,debugoptimized,release,minsize,custom}] [--debug]
                   [--default-library {shared,static,both}] [--errorlogs] [--install-umask INSTALL_UMASK]
                   [--layout {mirror,flat}] [--optimization {0,g,1,2,3,s}] [--prefer-static] [--stdsplit] [--strip]
                   [--unity {on,off,subprojects}] [--unity-size UNITY_SIZE] [--warnlevel {0,1,2,3}] [--werror]
                   [--wrap-mode {default,nofallback,nodownload,forcefallback,nopromote}]
                   [--force-fallback-for FORCE_FALLBACK_FOR] [--pkgconfig.relocatable]
                   [--python.install-env {auto,prefix,system,venv}] [--python.platlibdir PYTHON.PLATLIBDIR]
                   [--python.purelibdir PYTHON.PURELIBDIR] [--pkg-config-path PKG_CONFIG_PATH]
                   [--build.pkg-config-path BUILD.PKG_CONFIG_PATH] [--cmake-prefix-path CMAKE_PREFIX_PATH]
                   [--build.cmake-prefix-path BUILD.CMAKE_PREFIX_PATH] [-D option] [--native-file NATIVE_FILE]
                   [--cross-file CROSS_FILE] [--vsenv] [-v] [--fatal-meson-warnings] [--reconfigure] [--wipe]
                   [builddir] [sourcedir]

positional arguments:
  builddir
  sourcedir

optional arguments:
  -h, --help                                                show this help message and exit
  --prefix PREFIX                                           Installation prefix (default: /usr/local).
  --bindir BINDIR                                           Executable directory (default: bin).
  --datadir DATADIR                                         Data file directory (default: share).
  --includedir INCLUDEDIR                                   Header file directory (default: include).
  --infodir INFODIR                                         Info page directory (default: share/info).
  --libdir LIBDIR                                           Library directory (default: lib/x86_64-linux-gnu).
  --libexecdir LIBEXECDIR                                   Library executable directory (default: libexec).
  --localedir LOCALEDIR                                     Locale data directory (default: share/locale).
  --localstatedir LOCALSTATEDIR                             Localstate data directory (default: var).
  --mandir MANDIR                                           Manual page directory (default: share/man).
  --sbindir SBINDIR                                         System executable directory (default: sbin).
  --sharedstatedir SHAREDSTATEDIR                           Architecture-independent data directory (default: com).
  --sysconfdir SYSCONFDIR                                   Sysconf data directory (default: etc).
  --auto-features {enabled,disabled,auto}                   Override value of all 'auto' features (default: auto).
  --backend {ninja,vs,vs2010,vs2012,vs2013,vs2015,vs2017,vs2019,vs2022,xcode}
                                                            Backend to use (default: ninja).
  --buildtype {plain,debug,debugoptimized,release,minsize,custom}
                                                            Build type to use (default: debug).
  --debug                                                   Enable debug symbols and other information
  --default-library {shared,static,both}                    Default library type (default: shared).
  --errorlogs                                               Whether to print the logs from failing tests
  --install-umask INSTALL_UMASK                             Default umask to apply on permissions of installed files
                                                            (default: 022).
  --layout {mirror,flat}                                    Build directory layout (default: mirror).
  --optimization {0,g,1,2,3,s}                              Optimization level (default: 0).
  --prefer-static                                           Whether to try static linking before shared linking
  --stdsplit                                                Split stdout and stderr in test logs
  --strip                                                   Strip targets on install
  --unity {on,off,subprojects}                              Unity build (default: off).
  --unity-size UNITY_SIZE                                   Unity block size (default: (2, None, 4)).
  --warnlevel {0,1,2,3}                                     Compiler warning level to use (default: 1).
  --werror                                                  Treat warnings as errors
  --wrap-mode {default,nofallback,nodownload,forcefallback,nopromote}
                                                            Wrap mode (default: default).
  --force-fallback-for FORCE_FALLBACK_FOR                   Force fallback for those subprojects (default: []).
  --pkgconfig.relocatable                                   Generate pkgconfig files as relocatable
  --python.install-env {auto,prefix,system,venv}            Which python environment to install to (default: prefix).
  --python.platlibdir PYTHON.PLATLIBDIR                     Directory for site-specific, platform-specific files
                                                            (default: ).
  --python.purelibdir PYTHON.PURELIBDIR                     Directory for site-specific, non-platform-specific files
                                                            (default: ).
  --pkg-config-path PKG_CONFIG_PATH                         List of additional paths for pkg-config to search (default:
                                                            []). (just for host machine)
  --build.pkg-config-path BUILD.PKG_CONFIG_PATH             List of additional paths for pkg-config to search (default:
                                                            []). (just for build machine)
  --cmake-prefix-path CMAKE_PREFIX_PATH                     List of additional prefixes for cmake to search (default:
                                                            []). (just for host machine)
  --build.cmake-prefix-path BUILD.CMAKE_PREFIX_PATH         List of additional prefixes for cmake to search (default:
                                                            []). (just for build machine)
  -D option                                                 Set the value of an option, can be used several times to set                                                            multiple options.
  --native-file NATIVE_FILE                                 File containing overrides for native compilation
                                                            environment.
  --cross-file CROSS_FILE                                   File describing cross compilation environment.
  --vsenv                                                   Setup Visual Studio environment even when other compilers
                                                            are found, abort if Visual Studio is not found. This option
                                                            has no effect on other platforms than Windows. Defaults to
                                                            True when using "vs" backend.
  -v, --version                                             show program's version number and exit
  --fatal-meson-warnings                                    Make all Meson warnings fatal
  --reconfigure                                             Set options and reconfigure the project. Useful when new
                                                            options have been added to the project and the default value                                                            is not working.
  --wipe                                                    Wipe build directory and reconfigure using previous command
                                                            line options. Useful when build directory got corrupted, or
                                                            when rebuilding with a newer version of meson.
 ```
參數量十分的多，這些參數影響的是編譯的指令，最常用的是指定後端類型的```--backend```，以及指定編譯器的環境竟變數```CC=clang-14 CXX=clang++-14 meson setup build```

#### 3. compile
建立好編譯環境後就能來編譯程式了，使用的是```meson compile```指令，參數如下：
```
$ meson compile --help
usage: meson compile [-h] [--clean] [-C WD] [-j JOBS] [-l LOAD_AVERAGE] [-v] [--ninja-args NINJA_ARGS] [--vs-args VS_ARGS] [--xcode-args XCODE_ARGS] [TARGET [TARGET ...]]

positional arguments:
  TARGET                                        Targets to build. Target has the following format: [PATH_TO_TARGET/]TARGET_NAME[:TARGET_TYPE].

optional arguments:
  -h, --help                                    show this help message and exit
  --clean                                       Clean the build directory.
  -C WD                                         directory to cd into before running
  -j JOBS, --jobs JOBS                          The number of worker jobs to run (if supported). If the value is less than 1 the build program will guess.
  -l LOAD_AVERAGE, --load-average LOAD_AVERAGE  The system load average to try to maintain (if supported).
  -v, --verbose                                 Show more verbose output.
  --ninja-args NINJA_ARGS                       Arguments to pass to `ninja` (applied only on `ninja` backend).
  --vs-args VS_ARGS                             Arguments to pass to `msbuild` (applied only on `vs` backend).
  --xcode-args XCODE_ARGS                       Arguments to pass to `xcodebuild` (applied only on `xcode` backend).
 ```
參數相對少，Meson所需的編譯參數大多不是在meson.build定義了，少部分是在```meson setup```時定義，compile只重在「編譯」的這個動作。

我個人覺得```-v```在除錯時很有用，會顯示每個步驟所下的指令。
