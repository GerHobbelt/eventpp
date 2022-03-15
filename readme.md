# eventpp -- C++ library for event dispatcher and callback list

- [eventpp -- C++ library for event dispatcher and callback list](#eventpp----c-library-for-event-dispatcher-and-callback-list)
  - [Facts and features](#facts-and-features)
  - [License](#license)
  - [Version 0.1.2](#version-012)
  - [Source code](#source-code)
  - [Supported compilers](#supported-compilers)
  - [C++ standard requirements](#c-standard-requirements)
  - [Quick start](#quick-start)
    - [Namespace](#namespace)
    - [Use eventpp in your project](#use-eventpp-in-your-project)
    - [Using CallbackList](#using-callbacklist)
    - [Using EventDispatcher](#using-eventdispatcher)
    - [Using EventQueue](#using-eventqueue)
  - [Documentations](#documentations)
  - [Build the test code](#build-the-test-code)
  - [Motivations](#motivations)
  - [Change log](#change-log)

eventpp is a C++ event library for callbacks, event dispatcher, and event queue. With eventpp you can easily implement signal and slot mechanism, publisher and subscriber pattern, or observer pattern.

## Facts and features

- **Powerful**
    - Supports synchronous event dispatching and asynchronous event queue.
    - Configurable and extensible with policies and mixins.
    - Supports event filter via mixins.
- **Robust**
    - Supports nested event. During the process of handling an event, a listener can safely dispatch event and append/prepend/insert/remove other listeners.
    - Thread safety. Supports multi-threading.
    - Exception safety. Most operations guarantee strong exception safety.
    - Well tested. Backed by unit tests.
- **Fast**
    - The EventQueue can process 10M events in 1 second (10K events per millisecond).
    - The CallbackList can invoke 100M callbacks in 1 second (100K callbacks per millisecond).
    - The CallbackList can add/remove 5M callbacks in 1 second (5K callbacks per millisecond).
- **Flexible and easy to use**
    - Listeners and events can be of any type and do not need to be inherited from any base class.
    - Utilities that can ease the usage, such as auto disconnecting, one shot listener, argument type adapter, etc.
    - Header only, no source file, no need to build. Does not depend on other libraries.
    - Requires C++ 11.
    - Written in portable and standard C++, no hacks or quirks.

## License

Apache License, Version 2.0  

## Version 0.1.2
![CI](https://github.com/wqking/eventpp/workflows/CI/badge.svg)

The master branch is usable and stable.  
There are some releases on Github, but usually the releases are far behind the latest code.  
You should prefer to clone or fork the master branch instead of downloading the releases.  
Don't worry about the large timespan between commits and releases. The library is actively maintained.  
The master branch is currently fully back compatible with the first version. So your project won't get any back compatible issues.  
If you find any back compatible issue which is not announced, please report a bug.

## Source code

[https://github.com/wqking/eventpp](https://github.com/wqking/eventpp)

## Supported compilers

Tested with MSVC 2019, MinGW (Msys) GCC 7.2, Ubuntu GCC 5.4, and MacOS GCC.
GCC 4.8.3 can compile the library, but we don't support GCC prior to GCC 5.
In brief, MSVC, GCC, Clang that has well support for C++11, or released after 2019, should be able to compile the library.

## C++ standard requirements
* To Use the library  
    * The library: C++11.  
* To develop the library
    * Unit tests: C++17.
    * Tutorials: C++11.
    * Benchmarks: C++11.

## Quick start

### Namespace

`eventpp`

### Use eventpp in your project

There are various methods to use eventpp

1, Include the source code in your project directly.

eventpp is header only library. Just clone the source code, then add the 'include' folder inside eventpp to your project include directory, then you can use the library.  
You don't need to link to any source code.  

2, Or use vcpkg

```
vcpkg install eventpp
```

Then in your project CMakeLists.txt file, put below code, remember to replace ${TARGET} with your target,

```
find_package(eventpp CONFIG REQUIRED)
target_link_libraries(${TARGET} PRIVATE eventpp::eventpp)
find_path(EVENTPP_INCLUDE_DIR eventpp/eventqueue.h)
include_directories(${EVENTPP_INCLUDE_DIR})
```

Then run cmake, note you need -DCMAKE_TOOLCHAIN_FILE to specifiy vcpkg, and replace -G with your generator
```
cmake .. -DCMAKE_TOOLCHAIN_FILE=VCPKGDIR/vcpkg/scripts/buildsystems/vcpkg.cmake -G"MinGW Makefiles"
```

Note with vcpkg, only the released version can be used, which may be behind the latest update. You can't use the minor update on the master branch.

3, Or install using CMake and use it in CMake

If you are going to use eventpp in CMake managed project, you can install eventpp then use it in CMake.  
In eventpp root folder, run the commands,  
```
mkdir build
cd build
cmake ..
sudo make install
```

Then in the project CMakeLists.txt,   
```
# the project target is mytest, just for example
add_executable(mytest test.cpp)

find_package(eventpp)
if(eventpp_FOUND)
    target_link_libraries(mytest eventpp::eventpp)
else(eventpp_FOUND)
    message(FATAL_ERROR "eventpp library is not found")
endif(eventpp_FOUND)
```

Note: the method 2 doesn't work well with MingW on Windows. It works well on Linux.

### Using CallbackList
```c++
#include "eventpp/callbacklist.h"
eventpp::CallbackList<void (const std::string &, const bool)> callbackList;
callbackList.append([](const std::string & s, const bool b) {
    std::cout << std::boolalpha << "Got callback 1, s is " << s << " b is " << b << std::endl;
});
callbackList.append([](std::string s, int b) {
    std::cout << std::boolalpha << "Got callback 2, s is " << s << " b is " << b << std::endl;
});
callbackList("Hello world", true);
```

### Using EventDispatcher
```c++
#include "eventpp/eventdispatcher.h"
eventpp::EventDispatcher<int, void ()> dispatcher;
dispatcher.appendListener(3, []() {
    std::cout << "Got event 3." << std::endl;
});
dispatcher.appendListener(5, []() {
    std::cout << "Got event 5." << std::endl;
});
dispatcher.appendListener(5, []() {
    std::cout << "Got another event 5." << std::endl;
});
// dispatch event 3
dispatcher.dispatch(3);
// dispatch event 5
dispatcher.dispatch(5);
```

### Using EventQueue
```c++
eventpp::EventQueue<int, void (const std::string &, const bool)> queue;

queue.appendListener(3, [](const std::string s, bool b) {
    std::cout << std::boolalpha << "Got event 3, s is " << s << " b is " << b << std::endl;
});
queue.appendListener(5, [](const std::string s, bool b) {
    std::cout << std::boolalpha << "Got event 5, s is " << s << " b is " << b << std::endl;
});

// The listeners are not triggered during enqueue.
queue.enqueue(3, "Hello", true);
queue.enqueue(5, "World", false);

// Process the event queue, dispatch all queued events.
queue.process();
```

## Documentations

* Core classes and functions
    * [Overview](doc/introduction.md)
    * [Tutorials of CallbackList](doc/tutorial_callbacklist.md)
    * [Tutorials of EventDispatcher](doc/tutorial_eventdispatcher.md)
    * [Tutorials of EventQueue](doc/tutorial_eventqueue.md)
    * [Class CallbackList reference](doc/callbacklist.md)
    * [Class EventDispatcher reference](doc/eventdispatcher.md)
    * [Class EventQueue reference](doc/eventqueue.md)
    * [Policies -- configure eventpp](doc/policies.md)
    * [Mixins -- extend eventpp](doc/mixins.md)
* Utilities
    * [Utility argumentAdapter -- adapt pass-in argument types to the types of the functioning being called](doc/argumentadapter.md)
    * [Utility conditionalFunctor -- pre-check the condition before calling a function](doc/conditionalfunctor.md)
    * [Utility class CounterRemover -- auto remove listeners after triggered certain times](doc/counterremover.md)
    * [Utility class ConditionalRemover -- auto remove listeners when certain condition is satisfied](doc/conditionalremover.md)
    * [Utility class ScopedRemover -- auto remove listeners when out of scope](doc/scopedremover.md)
    * [Utility class OrderedQueueList -- make EventQueue ordered](doc/orderedqueuelist.md)
    * [Utility class AnyId -- use various data types as EventType in EventDispatcher and EventQueue](doc/anyid.md)
    * [Utility header eventmaker.h -- auto generate event classes](doc/eventmaker.md)
    * [Document of utilities functions](doc/eventutil.md)
* Miscellaneous
    * [Performance benchmarks](doc/benchmark.md)
    * [FAQs, tricks, and tips](doc/faq.md)
* Heterogeneous classes and functions, usually you don't need them
    * [Overview of heterogeneous classes](doc/heterogeneous.md)
    * [Class HeterCallbackList](doc/hetercallbacklist.md)
    * [Class HeterEventDispatcher](doc/hetereventdispatcher.md)
    * [Class HeterEventQueue](doc/hetereventqueue.md)

## Build the test code

The library itself is header only and doesn't need building.  
There are three parts of code to test the library,

- unittests: tests the library. They require C++17 since it uses generic lambda and `std::any` (the library itself only requires C++11).
- tutorials: sample code to demonstrate how to use the library. They require C++11. If you want to have a quick study on how to use the library, you can look at the tutorials.
- benchmarks: measure the library performance.

All parts are in the `tests` folder.

All three parts require CMake to build, and there is a makefile to ease the building.  
Go to folder `tests/build`, then run `make` with different target.
- `make vc19` #generate solution files for Microsoft Visual Studio 2019, then open eventpptest.sln in folder project_vc19
- `make vc17` #generate solution files for Microsoft Visual Studio 2017, then open eventpptest.sln in folder project_vc17
- `make vc15` #generate solution files for Microsoft Visual Studio 2015, then open eventpptest.sln in folder project_vc15
- `make mingw` #build using MinGW
- `make linux` #build on Linux
- `make mingw_coverage` #build using MinGW and generate code coverage report

## Motivations

I (wqking) am a big fan of observer pattern (publish/subscribe pattern), and I used this pattern extensively in my code. I either used GCallbackList in my [cpgf library](https://github.com/cpgf/cpgf) which is too simple and unsafe (not support multi-threading or nested events), or repeated coding event dispatching mechanism such as I did in my [Gincu game engine](https://github.com/wqking/gincu) (the latest version has be rewritten to use eventpp). Both methods are not fun nor robust.  
Thanking to C++11, now it's quite easy to write a reusable event library with beautiful syntax (it's a nightmare to simulate the variadic template in C++03), so here is `eventpp`.

## Change log

**Version 0.1.2**  Mar 11, 2022  
Bug fix.  
Added more unit tests.  
Added utilities argumentAdapter and conditionalFunctor.  
Added utilities AnyId.  
Added event maker macros.  
  
**Version 0.1.1**  Dec 13, 2019  
Added HeterCallbackList, HeterEventDispatcher, and HeterEventQueue.

**Version 0.1.0**  Sep 1, 2018  
First version.  
Added CallbackList, EventDispatcher, EventQueue, CounterRemover, ConditionalRemover, ScopedRemover, and utilities.
