# Building MongoDB driver with Visual Studio 2015

The [currently instructions](https://mongodb.github.io/mongo-cxx-driver/mongocxx-v3/installation/) to build and use **mongocxx** driver is not 
focused on Windows platform, and to use it with **Visual Studio 2015** can take many hours before a successfuly
compilation of your application.

To help in this task, i wrote this recipe to do it easily.

## Installing the mongocxx driver

### Prerequisites

* Microsoft Windows 7 SPI+
* Visual Studio 2015 Update 1 or later
* [CMake](https://cmake.org/download/) 3.2 (x64) or later
* [Boost](http://www.boost.org/) 1.56 or later
* A [git client](https://git-scm.com/downloads) for Windows

### Installation

#### Step 1: Install the latest version of the MongoDB C driver.

The mongocxx driver builds on top of the MongoDB C driver.

* For mongocxx-3.0.x, we recommend the latest stable version of libmongoc (currently version 1.4.2 at the time this page was written).
* For mongocxx-3.1.0-beta0 or later, libmongoc 1.5.0-rc3 or later is required.

Follow the instructions at Installing [libmongoc](http://mongoc.org/libmongoc/current/installing.html).

If you need static libraries, be sure to use the --enable-static configure option when building libmongoc.


#### Step 2: Download the latest version of the mongocxx driver.

To get the source via git, the releases/stable branch will track the most recent stable release. For example, to work from a shallow checkout of the stable release branch:

	git clone https://github.com/mongodb/mongo-cxx-driver.git \
		--branch releases/stable --depth 1

		
#### Step 3: Configure the driver

Make sure you change to the build directory of whatever source tree you obtain. 
Be aware that in the step #1, you build libmongoc in root folder of C:\ drive, then to configure driver references, we must
set the *libbson* and *mongoc* directory of *libmongoc* as follow:

	cd mongo-cxx-driver/build
	cmake -DCMAKE_BUILD_TYPE=Release \
		-DLIBBSON_DIR=C:\mongo-c-driver-1.4.2\src\libbson \
		-DLIBMONGOC_DIR=C:\mongo-c-driver-1.4.2\src\mongoc \
		-DCMAKE_INSTALL_PREFIX=C:\mongo-cxx-driver ..
		
		
#### Step 4: Build the driver

At "Developer Command Prompt for Visual Studio 2015" you must run this command to compile and install
mongocxx driver (inside of *mongo-cxx-driver/build* folder):

	msbuild.exe ALL_BUILD.vcxproj
	msbuild.exe INSTALL.vcxproj
	
If everything runs fine, a folder *install* should be created inside *build* folder, where you can
find */bin*, */include* and */lib* folders to be used inside of your Visual Studio solution.


#### Step 5: Setting your solution properties

At this point, you have the *libmongoc* and *mongocxx* drivers to include and link in your project.
If you follow the reference in step #1 and #3 you must have two directories in your C:\ disk:

	C:\mongo-c-driver
	C:\mongo-cxx-driver/build/install

Inside of each of this folders you will find *bin*, *include* and *lib* folders to set in your
solution properties. You can move this folders to a more convenient place, in this example I copy
this inside my project directory on a third party library folder, but you can put it inside a "global"
folder.

Open your solution with Visual Studio 2015 and follow this:

* Left click over your **project** item in *Solution Explorer*, and choose *Properties* menu;
* On *VC++ Directory* page, add this entries into *Include Directories* list:
	- $(ProjectDir)third-party\boost_1_56_0
	- $(ProjectDir)third-party\mongo-cxx-driver\include\mongocxx\v_noabi
	- $(ProjectDir)third-party\mongo-cxx-driver\include\bsoncxx\v_noabi
	- $(ProjectDir)third-party\mongo-c-driver\include\libmongoc-1.0
	- $(ProjectDir)third-party\mongo-c-driver\include\libbson-1.0
* On *VC++ Directory* page, add this entries into *Library Directories* list:
	- $(ProjectDir)third-party\mongo-cxx-driver\lib
	- $(ProjectDir)third-party\mongo-c-driver\lib
* On *Linker/Input* page, add this entries into *Additional Dependencies*:
	- bsoncxx.lib
	- libbsoncxx.lib
	- libmongocxx.lib
	- mongocxx.lib
	
**Be aware to include a reference to Boost Library in include directory's list.**


#### Step 6: Test your installation

Save the following source file inside your code to try:

``` c++
#include <iostream>

#include <bsoncxx/builder/stream/document.hpp>
#include <bsoncxx/json.hpp>

#include <mongocxx/client.hpp>
#include <mongocxx/instance.hpp>

int main(int, char**) {
	mongocxx::instance inst{};
	mongocxx::client conn{mongocxx::uri{}};

	bsoncxx::builder::stream::document document{};

	auto collection = conn["testdb"]["testcollection"];
	document << "hello" << "world";

	collection.insert_one(document.view());
	auto cursor = collection.find({});

	for (auto&& doc : cursor) {
		std::cout << bsoncxx::to_json(doc) << std::endl;
	}
}
```
