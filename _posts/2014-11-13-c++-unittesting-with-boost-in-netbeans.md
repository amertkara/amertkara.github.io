---
layout: post
title: C++ Unit-testing with Boost library in Netbeans
tags: dev, testing

---

I can say that by far Netbeans is the best IDE for C++ development in OSX. Also I keep an eye on [CLion](https://www.jetbrains.com/clion/ "CLion") project from Jetbrains, who has made decent IDEs for major languages so far. Nevertheless, Netbeans is my ultimate choice because it provides a clean interface with fully integration with make. 

In this short tutorial, I want to show how to integrate Boost Test library with Netbeans.

## Installing Requirements ##

Boost library is the only requirement for this tutorial. In my case, I am working on OSX (Mavericks) so I use Homebrew, which is a package manager for OSX. The command is `brew install boost`:

![Brew Install Boost](/public/images/2014/11/brew-install-boost.png)

For unix variants, there must be a package manager which will probably have a similar command. Windows folks can go-get their ready meals in [here](http://sourceforge.net/projects/boost/files/boost-binaries "Boost SF binaries download").


## Goto Netbeans ##

I am using Netbeans 8.0.1 so if you have a different version, some of the menu items might be named/placed differently. Now, I will create a new project:

![Netbeans New C++ Project](/public/images/2014/11/netbeans-new-c++-project.png)
 
 Set the project details, mostly leaving everything as default
 
![Netbeans New C++ Project Details](/public/images/2014/11/netbeans-new-c++-project-details.png)
 
Netbeans creates a nice project structure ready for development.
 
![Netbeans C++ Project Structure](/public/images/2014/11/netbeans-c++-project-structure.png)
 
If you noticed, it also created a virtual folder for Test Files, namely `Test Files` folder. Out of box, netbeans gives support for [CppUnit](http://sourceforge.net/projects/cppunit/, "CppUnit SF page"), which is another C++ test framework. It is a widely used framework with some awesome features but I personally prefer Boost over any library because it is as good as the stdlib.
 
Now it is time to write a simple test case. I will try to keep it as basic as possible for the sake of the tutorial. You can learn more about Boost test framework in its [documentation](http://www.boost.org/doc/libs/1_57_0/libs/test/doc/html/index.html "Boost Test Documentation for version 1.57").
 
Right-click on Source Files > New C++ Source File. Type in:
 
{% highlight c++ %}
#define BOOST_TEST_DYN_LINK
#define BOOST_TEST_MODULE SIMPLEST_TEST_SUITE
#include <boost/test/unit_test.hpp>

 
BOOST_AUTO_TEST_CASE(simplest_test_case)
{
    BOOST_CHECK(5 == 5);
}
{% endhighlight %}

Now that we have a source file with a simple test case, it is time to put the source file under the `Test Files` so that it gets picked when `make test` runs.

Right-click on Test Files > New Test Folder. This will create a folder under `Test Files` which is a virtual folder. Finally drag and drop the test source file under the test folder in the `Projects` window of Netbeans:

![Netbeans Test Folder](/public/images/2014/11/netbeans-test-folder.png)

## Almost There ##

Now that we have the test case prepared, it is about configuring Netbeans to pick up the linked library as well as right CCFlag for build up.

Right-click on Project root folder > Click on Properties (a.k.a. Project Settings) :

\> Build > C++ Compiler

`Include Directories=/usr/local/include`

`Additional Options=-lboost_unit_test_framework`

![Netbeans Boost Settings](/public/images/2014/11/netbeans-boost-settings.png)
 
 Now that the settings are intact, you can run the tests:
 
 Drill Down Important Files > Right-click on the Makefile > Make Target > test
 
![Netbeans Boost Make Test](/public/images/2014/11/netbeans-boost-make-test.png)

As you see, it ran the test class and returned:

{% highlight bash %}
"/Applications/Xcode.app/Contents/Developer/usr/bin/make" -f nbproject/Makefile-Debug.mk QMAKE= SUBPROJECTS= .build-conf
"/Applications/Xcode.app/Contents/Developer/usr/bin/make"  -f nbproject/Makefile-Debug.mk dist/Debug/GNU-MacOSX/boosttest
mkdir -p dist/Debug/GNU-MacOSX
g++ -lboost_unit_test_framework    -o dist/Debug/GNU-MacOSX/boosttest build/Debug/GNU-MacOSX/main.o 
"/Applications/Xcode.app/Contents/Developer/usr/bin/make" -f nbproject/Makefile-Debug.mk SUBPROJECTS= .build-tests-conf
"/Applications/Xcode.app/Contents/Developer/usr/bin/make"  -f nbproject/Makefile-Debug.mk dist/Debug/GNU-MacOSX/boosttest
make[2]: `dist/Debug/GNU-MacOSX/boosttest' is up to date.
mkdir -p build/Debug/GNU-MacOSX/tests
rm -f "build/Debug/GNU-MacOSX/tests/unittest.o.d"
g++ -lboost_unit_test_framework   -c -g -I/usr/local/include -I. -I. -I. -MMD -MP -MF "build/Debug/GNU-MacOSX/tests/unittest.o.d" -o build/Debug/GNU-MacOSX/tests/unittest.o unittest.cpp
clang: warning: -lboost_unit_test_framework: 'linker' input unused
mkdir -p build/Debug/GNU-MacOSX/tests/TestFiles
g++ -lboost_unit_test_framework      -o build/Debug/GNU-MacOSX/tests/TestFiles/f1 build/Debug/GNU-MacOSX/tests/unittest.o build/Debug/GNU-MacOSX/main_nomain.o  
"/Applications/Xcode.app/Contents/Developer/usr/bin/make" -f nbproject/Makefile-Debug.mk SUBPROJECTS= .test-conf
Running 1 test case...
*** No errors detected

MAKE SUCCESSFUL (total time: 1s)
{% endhighlight %}

Alternatively, you can go to the Netbeans project folder with Terminal app and run the make commands from there:

![Make Test Terminal](/public/images/2014/11/make-test-terminal.png)

That's it!