Robust system = without any cracks (strong even on the edges)

Build on Windows
	CMAKE with cmake spec file
		VS sln with VS projects created
	MSBUILD build target to link and create .o files
	MSBUILD install to compile to executable
Build on Linux
	Create makefile
		Possibly with cmake
	Run make

If you build from CMAKE (compile to visual studio) on a dependency for another application, you will have to rebuild from CMAKE for that application as well. Library links change

Whichever branch you build will be the one present in install

Interesting...CMakeLists.txt can provide c++ include directory locations so that no matter where the cpp file moves (as long as its on the same machine), it will always be able to find its dependencies.

ALWAYS:
	Transfer changes from install folder to main branch before recompiling
	
taskkill /IM cmd.exe
	