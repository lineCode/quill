<h2> <img src="https://i.postimg.cc/FssWB25k/quill-logo.png" alt="Quill logo" width="140"><br>Asynchronous Low Latency Logging Library</h2>

[![project_status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.][badge.project_status]][project_status]
[![travis][badge.travis]][travis]
[![appveyor][badge.appveyor]][appveyor]
[![codecov][badge.codecov]][codecov]
[![language][badge.language]][language]
[![license][badge.license]][license]

[badge.project_status]: https://www.repostatus.org/badges/latest/wip.svg
[badge.travis]: https://img.shields.io/travis/odygrd/quill/master.svg?logo=travis
[badge.appveyor]: https://img.shields.io/appveyor/ci/odygrd/quill/master.svg?logo=appveyor
[badge.codecov]: https://img.shields.io/codecov/c/gh/odygrd/quill/master.svg?logo=codecov 
[badge.language]: https://img.shields.io/badge/language-C%2B%2B14-red.svg
[badge.license]: https://img.shields.io/badge/license-MIT-blue.svg

[project_status]: https://www.repostatus.org/#wip 
[travis]: https://travis-ci.org/odygrd/quill
[appveyor]: https://ci.appveyor.com/project/odygrd/quill
[codecov]: https://codecov.io/gh/odygrd/quill
[language]: https://en.wikipedia.org/wiki/C%2B%2B14
[license]: http://opensource.org/licenses/MIT

- [Design Rationale](#design-rationale)
- [Features](#features)
- [Performance](#performance)
- [Supported Platforms And Compilers](#supported-platforms-and-compilers)
- [Integration](#integration)
  - [CMake](#cmake)
  - [Package Managers](#package-managers)
- [Basic Usage](#basic-usage)
- [Documentation](#documentation)
- [License](#license)
- [Disclaimer](#disclaimer)

## Design Rationale
The library aims to make logging significantly easier for the application developer while at the same time reduce the overhead of logging in the critical path as much as possible.

The main goals of the library are:

- **Simplicity** A small example code snippet should be enough to get started and use most of features.
- **Performance** Ultra low latency for the caller threads, no string formatting on the fast-path, no heap allocations after initialisation, asynchronous only mode.
- **Convenience** While keeping low latency on the fast-path, the library aims to assist the developer in debugging the application by providing a nicely formatted direct textual output with all log statements ordered by timestamp

## Features
 * Clean warning-free codebase even on high warning levels
 * Safety. Extensive set of unit tests. Tested with Adress Sanitizer, Thread Sanitizer, Valgrind
 * Thread and Type safe with compile time checks
 * Python style formatting with build in support for logging STL containers, std::pair, std::tuple, std::chrono, user defined types and much more by using the excellent [{fmt}](https://github.com/fmtlib/fmt) library
 * Configurable
 * Custom log patterns. Log statements can be formatted by providing a simple pattern
 * Log levels can be stripped out at compile time in release builds
 * Log records are written in timestamp order even if they were created by different threads
 * Guaranteed logging. Log messages are never dropped
 * Support for wide character logging and wide character filenames (Windows only)
 * Various log targets (Handlers)
   * Console logging 
   * Rotating log files [Work in progress]
   * Daily log files [Work in progress]

## Performance

### 1 Thread Logging

| Library            | 50th     | 75th     | 90th     | 95th     |  99th    | 99.9th   | Worst     |
|--------------------|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:---------:|
|[Quill](https://github.com/odygrd/quill)               | 16       | 17       | 17       | 17       | 1625     | 2961     | 4709      |
|[PlatformLab NanoLog](https://github.com/PlatformLab/NanoLog) | 18       | 18       | 19       | 21       | 24       | 46       | 32407     |
|[Reckless](https://github.com/mattiasflodin/reckless)            | 50       | 51       | 53       | 54       | 57       | 106982   | 600779234 |
|[Iyengar NanoLog](https://github.com/Iyengar111/NanoLog)     | 62       | 63       | 75       | 1042     | 1154     | 3187     | 13002667  |
|[spdlog](https://github.com/gabime/spdlog)              | 183      | 226      | 369      | 2842     | 2906     | 5598     | 1079157   |

[![Benchmark](https://github.com/odygrd/quill/blob/master/images/benchmark_90th.png?raw=true)](https://charts.hohli.com/embed.html?created=1582255481780#w=640&h=480&d={"containerId":"chart","dataTable":{"cols":[{"label":"A","type":"string","p":{}},{"label":"B","type":"number","p":{}},{"label":"C","type":"number","p":{}},{"label":"D","type":"number","p":{}},{"label":"E","type":"number","p":{}},{"label":"F","type":"number"}],"rows":[{"c":[{"v":"Spdlog"},{"v":369},{"v":0},{"v":0},{"v":0},{"v":0}]},{"c":[{"v":"Lyengar%20NanoLog"},{"v":0},{"v":75},{"v":0},{"v":0},{"v":0}]},{"c":[{"v":"Reckless"},{"v":0},{"v":0},{"v":53},{"v":0},{"v":0}]},{"c":[{"v":"PlatformLab%20NanoLog"},{"v":0},{"v":0},{"v":0},{"v":19},{"v":0}]},{"c":[{"v":"Quill"},{"v":0},{"v":0},{"v":0},{"v":0},{"v":17}]}]},"options":{"legacyScatterChartLabels":true,"isStacked":true,"booleanRole":"certainty","vAxes":[{"minValue":null,"maxValue":null,"viewWindow":null,"viewWindowMode":null,"useFormatFromData":true},{"useFormatFromData":true}],"hAxis":{"viewWindow":{"max":90,"min":0},"minValue":0,"maxValue":90,"useFormatFromData":true,"logScale":false,"title":"Time%20(ns)","titleTextStyle":{"color":"#222","fontSize":12,"italic":true,"bold":true},"viewWindowMode":"explicit","gridlines":{"count":"6"},"minorGridlines":{"count":"0"},"slantedText":false},"legend":"none","width":640,"height":480,"title":"1%20Thread-%2090th%20Percentile%20Latency%20","titleTextStyle":{"color":"#000","fontSize":"13","bold":true,"italic":false},"fontName":"Tahoma"},"state":{},"view":{"columns":null,"rows":null},"isDefaultVisualization":false,"chartType":"BarChart"})

The benchmarks aare done on Linux (Ubuntu/RHEL) with GCC 9.1. The following message is logged 100'000 times ```LOG_INFO(logger, "Logging str: {}, int: {}, double: {}", str, i, d)```.  

The benchmark code can be found [here](https://github.com/odygrd/logger_benchmarks).  
More benchmarks can be found [here](https://github.com/odygrd/logger_benchmarks/blob/master/results.txt).

## Supported Platforms And Compilers
Quill requires a C++14 compiler. Minimum required versions of supported compilers are shown in the below table.

| *****    | Compiler  | Notes            |
|----------|-----------|------------------|
|![gcc]    | GCC       | version >= 5.0   |
|![llvm]   | Clang     | version >= 5.0   |      
|![msvc]   | MSVC++    | version >= 14.3  |

| *****    | Platform           | Notes                                                   |
|----------|--------------------|---------------------------------------------------------|
|![ubuntu] | Ubuntu             |                                                         |
|![rhel]   | Red Hat Enterprise | Should also work on Fedora                              |
|![centos] | CentOs             |                                                         |
|![win10]  | Windows            | Windows 10 - version 1607, Windows Server 2016 required |
|![mac]    | macOS              | Tested with Xcode 9.4                                   |

[gcc]: https://github.com/odygrd/quill/blob/master/images/gcc_logo.png?raw=true
[llvm]: https://github.com/odygrd/quill/blob/master/images/llvm_logo.png?raw=true
[msvc]: https://github.com/odygrd/quill/blob/master/images/msvc_logo.png?raw=true
[win10]: https://github.com/odygrd/quill/blob/master/images/win10_logo.png?raw=true
[mac]: https://github.com/odygrd/quill/blob/master/images/macos_logo.png?raw=true
[ubuntu]: https://github.com/odygrd/quill/blob/master/images/ubuntu-logo.png?raw=true
[rhel]: https://github.com/odygrd/quill/blob/master/images/rhel_logo.png?raw=true
[centos]: https://github.com/odygrd/quill/blob/master/images/centos_logo.png?raw=true

## Integration

### CMake

#### External

To use this library from a CMake project, you can locate it directly with `find_package()` and use the namespaced imported target from the generated package configuration:

```cmake
# CMakeLists.txt
find_package(quill 3.2.0 REQUIRED)
...
add_library(foo ...)
...
target_link_libraries(foo PRIVATE quill::quill)
```

#### Embedded

To embed the library directly into an existing CMake project, place the entire source tree in a subdirectory and call `add_subdirectory()` in your `CMakeLists.txt` file:

```cmake
add_subdirectory(quill)
...
add_library(foo ...)
...
target_link_libraries(foo PRIVATE quill::quill)
```

### Package Managers

## Basic usage

```c++
#include "quill/Quill.h"

int main()
{
  // Start the logging backend thread
  quill::start();
  
  // Get a pointer to the default logger
  quill::Logger* dl = quill::get_logger();

  LOG_INFO(dl, "Welcome to Quill!");
  LOG_ERROR(dl, "An error message with error code {}, error message {}", 123, "system_error");

  LOG_WARNING(dl, "Support for int: {0:d};  hex: {0:x};  oct: {0:o}; bin: {0:b}", 42);
  LOG_CRITICAL(dl, "Easy padding in numbers like {:08d}", 12);

  LOG_DEBUG(dl, "This message and any message below this log level will not be displayed..");

  // Enable additional log levels on this logger
  dl->set_log_level(quill::LogLevel::TraceL3);

  LOG_DEBUG(dl, "The answer is {}", 1337);
  LOG_TRACE_L1(dl, "{:>30}", "right aligned");
  LOG_TRACE_L2(dl, "Positional arguments are {1} {0} ", "too", "supported");
  LOG_TRACE_L3(dl, "Support for floats {:03.2f}", 1.23456);
}
```

### Output
By default Quill outputs to stdout using the default formatting pattern:

`ascii_time [thread_id] filename:line log_level logger_name - message`

```
01:29:06.190725386 [1783860] example_01.cpp:11 LOG_INFO     root - Welcome to Quill!
01:29:06.190727584 [1783860] example_01.cpp:12 LOG_ERROR    root - An error message with error code 123, error message system_error
01:29:06.190731526 [1783860] example_01.cpp:14 LOG_WARNING  root - Support for int: 42;  hex: 2a;  oct: 52; bin: 101010
01:29:06.190732157 [1783860] example_01.cpp:15 LOG_CRITICAL root - Easy padding in numbers like 00000012
01:29:06.190732723 [1783860] example_01.cpp:22 LOG_DEBUG    root - The answer is 1337
01:29:06.190733093 [1783860] example_01.cpp:23 LOG_TRACE_L1 root -                  right aligned
01:29:06.190735322 [1783860] example_01.cpp:24 LOG_TRACE_L2 root - Positional arguments are supported too 
01:29:06.190736334 [1783860] example_01.cpp:25 LOG_TRACE_L3 root - Support for floats 1.23
```

## Documentation
Advanced usage and additional documentation can be found in the [wiki](https://github.com/odygrd/quill) pages.

## License
Quill is licensed under the [MIT License](http://opensource.org/licenses/MIT)

Quill depends on third party libraries with separate copyright notices and license terms. 
Your use of the source code for these subcomponents is subject to the terms and conditions of the following licenses.

   * ([MIT License](http://opensource.org/licenses/MIT)) {fmt} (https://github.com/fmtlib/fmt/blob/master/LICENSE.rst)
   * ([MIT License](http://opensource.org/licenses/MIT)) invoke.hpp (https://github.com/BlackMATov/invoke.hpp/blob/master/LICENSE.md)

## Disclaimer
Icons used in this readme in Supported Platforms And Compilers section are solely for information readability purposes. I do not own these icons. 
