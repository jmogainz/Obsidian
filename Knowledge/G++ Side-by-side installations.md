### Unique to every version
- The includes at the location of the gcc suite install
	- This location is hardcoded into the g++ binary at compilation time, and when you run the g++ compiler, it will use that path as the default include search paths before any extra are added from cmd line (cmake adds these paths to the cmd line when it calls the compiler) 
### Not unique to every version
- The libstd++.so. This contains the compiled definitions/ implementation of the C++ Standard Library including the STL.
	- This is unique to versions that were released at the time of an ABI change (cxx11 for example), when new C++ versions versions begin being supported, and even when minor changes to optimize/improve the code occur.
	- In many cases, different g++ version compilers with unique includes accompanying them, can use the same exact libstdc++.so.
	- The location of the correct libstdc++ to be used at compilation time is determined by the linker using LD_LIBRARY_PATH, , or standard system library locations.
- The ld linker.
	- The system wide linker can most often be used for any gcc version. 