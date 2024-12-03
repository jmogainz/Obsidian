Analyze my problem that I am getting and explain what is causing the difference in behavior from these two commands:

▷ pytest
============================================================================================================================ test session starts ============================================================================================================================
platform linux -- Python 3.10.13, pytest-8.3.3, pluggy-1.5.0
rootdir: /home/jmoore2/i2dev/remote/ai-reinforcementlearning
configfile: pyproject.toml
plugins: anyio-4.6.0
collected 131 items / 1 error

================================================================================================================================== ERRORS ===================================================================================================================================
__________________________________________________________________________________________________ ERROR collecting tests/utilities_tests/recursive_variant_pybind_test.py __________________________________________________________________________________________________
ImportError while importing test module '/home/jmoore2/i2dev/remote/ai-reinforcementlearning/tests/utilities_tests/recursive_variant_pybind_test.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/conda/lib/python3.10/importlib/__init__.py:126: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
utilities_tests/recursive_variant_pybind_test.py:7: in <module>
    import install.bin.EnvWrapper as EnvWrapper
E   ImportError: /opt/conda/bin/../lib/libstdc++.so.6: version GLIBCXX_3.4.30' not found (required by /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
============================================================================================================================= warnings summary ==============================================================================================================================
../.venv/lib/python3.10/site-packages/pygame/pkgdata.py:25
  /home/jmoore2/i2dev/remote/ai-reinforcementlearning/.venv/lib/python3.10/site-packages/pygame/pkgdata.py:25: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    from pkg_resources import resource_stream, resource_exists

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
========================================================================================================================== short test summary info ==========================================================================================================================
ERROR utilities_tests/recursive_variant_pybind_test.py
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
======================================================================================================================== 1 warning, 1 error in 1.66s ==

Okay as you can see the above one errored.....now look at this:

▷ pytest utilities_tests/recursive_variant_pybind_test.py
============================================================================================================================ test session starts ============================================================================================================================
platform linux -- Python 3.10.13, pytest-8.3.3, pluggy-1.5.0
rootdir: /home/jmoore2/i2dev/remote/ai-reinforcementlearning
configfile: pyproject.toml
plugins: anyio-4.6.0
collected 10 items

utilities_tests/recursive_variant_pybind_test.py ......xxxx                                                                                                                                                                                                           [100%]

======================================================================================================================= 6 passed, 4 xfailed in 0.14s =======
ChatGPT

The difference in behavior between the two pytest commands arises due to discrepancies in how the Python interpreter resolves shared library dependencies, specifically the libstdc++.so.6 library, which is part of the GNU Standard C++ Library (libstdc++). Here's a detailed analysis of what's happening:
Understanding the Error

When you run pytest, you encounter the following error:

ImportError: /opt/conda/bin/../lib/libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)

This error indicates that the shared object EnvWrapper.cpython-310-x86_64-linux-gnu.so depends on a version of libstdc++.so.6 that provides the GLIBCXX_3.4.30 symbol. However, the libstdc++.so.6 found at /opt/conda/lib/ does not have this symbol.
Difference in Behavior Between the Two Commands

    Running pytest:
        Test Collection Phase: When you run pytest without specifying a test file, it collects all test modules in your project.
        Importing Modules: During this collection, it imports all test modules and any modules they import at the top level.
        Environment Variables: The environment variables, particularly LD_LIBRARY_PATH, are set in such a way that the Python interpreter uses the libstdc++.so.6 from the Conda environment (/opt/conda/lib/), which lacks the required GLIBCXX_3.4.30 symbol.
        Result: The import fails during test collection because the necessary symbol is missing, leading to the ImportError.

    Running pytest utilities_tests/recursive_variant_pybind_test.py:
        Specific Test Execution: By specifying the test file, pytest only collects and runs tests from that file.
        Import Timing: The module install.bin.EnvWrapper might be imported during the test execution phase rather than the collection phase, or the environment might be slightly different.
        Environment Variables: The dynamic linker might resolve libstdc++.so.6 from a different location (e.g., the system library path), which contains the required GLIBCXX_3.4.30 symbol.
        Result: The import succeeds, and the tests run as expected.

Root Cause Analysis

    Conda Environment's libstdc++.so.6: The Conda environment provides its own libstdc++.so.6, which may be outdated and lack newer symbols like GLIBCXX_3.4.30.
    System libstdc++.so.6: Your system's standard library (/usr/lib64/libstdc++.so.6, for example) likely includes the newer symbols required by EnvWrapper.
    Dynamic Linking Order: The dynamic linker (ld.so) searches for shared libraries in directories specified by LD_LIBRARY_PATH, rpath in executables, and default system paths. The order in which it searches can cause different versions of libstdc++.so.6 to be loaded in different contexts.
    Test Collection vs. Execution: Importing during test collection might differ in environment setup compared to importing during test execution, especially if environment variables are modified within tests or fixtures.

How to Resolve the Issue

    Update Conda's libstdc++:

        Upgrade the libstdcxx package in your Conda environment to a version that includes GLIBCXX_3.4.30.

    conda update libstdcxx-ng

Adjust LD_LIBRARY_PATH:

    Prepend the system library path to LD_LIBRARY_PATH when running pytest:

    export LD_LIBRARY_PATH=/usr/lib64:$LD_LIBRARY_PATH
    pytest

    Be cautious with this approach, as it might cause conflicts with other Conda packages.

Use Conda's Compiler Stack:

    Rebuild EnvWrapper using the compilers provided by Conda to ensure compatibility with Conda's libstdc++.

Modify the Test Import:

    Delay the import of install.bin.EnvWrapper until it's actually needed in the test functions, reducing the chance of import errors during test collection.

        # Don't import at the top-level
        # import install.bin.EnvWrapper as EnvWrapper

        def test_some_functionality():
            import install.bin.EnvWrapper as EnvWrapper
            # Rest of the test code

    Isolate the Test Environment:
        Run pytest within an environment where the system's libstdc++ is prioritized, possibly by using virtual environments or Docker containers.

Conclusion

The difference in behavior stems from how the dynamic linker resolves shared library dependencies in different contexts. When running pytest on all tests, the Conda environment's outdated libstdc++.so.6 is used, leading to the import error. When running the specific test file, the system's libstdc++.so.6 is used, which contains the required symbol, allowing the tests to pass.

By ensuring that the appropriate version of libstdc++.so.6 is available and correctly referenced during both test collection and execution, you can resolve the import error and achieve consistent test results.
You said:
Okay so here is the weird part though, here is the ldd output for that library:

(.venv) ➜  tests git:(feature/I2AI-274-unit-test-update-unit-test-for-pydantic-models-for-ray-and-stable-baseline) ✗ ldd ../install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so
        linux-vdso.so.1 (0x00007ffd63e96000)
        libenvironmentsshared_d.so.1.0.0 => /home/jmoore2/i2dev/remote/ai-reinforcementlearning/tests/../install/bin/../lib/libenvironmentsshared_d.so.1.0.0 (0x00007fb0ab200000)
        libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fb0aafd4000)

And my coworker has the exact same ldd output on his machine. We are running the same docker container. And his works fine running 'pytest' only. No issues. So it is supposed to be resolving the libstdc++ from the system, not the miniconda installation. What is going on?
ChatGPT

Thank you for providing additional information. The key points to consider are:

    ldd Output: Both you and your coworker have the same ldd output, showing that EnvWrapper.cpython-310-x86_64-linux-gnu.so depends on /usr/lib/x86_64-linux-gnu/libstdc++.so.6.

    Same Docker Container: You're both running the same Docker container, implying the environments should be identical.

    Your Coworker's pytest Works: Despite the same setup, your coworker's pytest runs without errors, while yours encounters the ImportError.

Given that, let's delve deeper to identify what's causing the discrepancy on your machine.
Understanding the Issue

    ldd vs. Runtime Linking:
        ldd shows the shared library dependencies based on how the binary was linked at compile time.
        At runtime, the dynamic linker (ld.so) searches for libraries based on environment variables like LD_LIBRARY_PATH and other factors.
        Therefore, even if ldd shows the dependency on /usr/lib/x86_64-linux-gnu/libstdc++.so.6, at runtime, the linker might still load a different libstdc++.so.6 if it's found earlier in the search path.

    Environment Variables:
        The environment variables, especially LD_LIBRARY_PATH, can alter the library loading behavior.
        Conda Environments: Conda may set LD_LIBRARY_PATH to include its own library directories (e.g., /opt/conda/lib), which can take precedence over system libraries.

Investigating the Root Cause
1. Check LD_LIBRARY_PATH

    Print LD_LIBRARY_PATH: Before running pytest, check the value of LD_LIBRARY_PATH.

    echo $LD_LIBRARY_PATH

    Compare with Coworker: Ask your coworker to check their LD_LIBRARY_PATH as well.

2. Use ldd with Environment Variables

    Run ldd with the Environment:

    LD_LIBRARY_PATH=$LD_LIBRARY_PATH ldd ../install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so

    **Check if libstdc++.so.6 is being resolved to /opt/conda/lib/libstdc++.so.6.

3. Use LD_DEBUG to Trace Library Loading

    Set LD_DEBUG:

    LD_DEBUG=libs pytest

    Analyze Output: This will produce verbose output showing how the dynamic linker searches for and loads shared libraries.

    Look for libstdc++.so.6: See which libstdc++.so.6 is being loaded during the pytest run.

4. Compare Conda Environments

    Check Conda's libstdc++.so.6:

strings /opt/conda/lib/libstdc++.so.6 | grep GLIBCXX

Check for GLIBCXX_3.4.30: See if the required symbol exists.

Update Conda Packages: If it's outdated, try updating Conda's libstdc++:

    conda update libstdcxx-ng

5. Examine Differences in pytest Runs

    Check for Imports During Collection:
        When you run pytest, it collects all tests and imports modules during collection.
        If any module or test modifies LD_LIBRARY_PATH, it can affect subsequent imports.

    Specific Test File Behavior:
        Running pytest utilities_tests/recursive_variant_pybind_test.py might not trigger the same import behavior during collection.
        The environment at the time of importing EnvWrapper may differ.

6. Examine the Conda Activation Scripts

    Conda's Activation May Differ:
        Conda activation scripts can modify environment variables differently on different machines or under different conditions.

    Check for Custom Scripts:
        Ensure there are no custom scripts or configurations in your environment that could alter LD_LIBRARY_PATH.

Possible Explanations

    Different LD_LIBRARY_PATH Settings:
        Your LD_LIBRARY_PATH includes /opt/conda/lib before system libraries, causing the linker to load the Conda version of libstdc++.so.6.
        Your coworker's LD_LIBRARY_PATH might prioritize system libraries or not include /opt/conda/lib.

    Conda's libstdc++.so.6 Version:
        The Conda environment on your machine has an outdated libstdc++.so.6.
        Your coworker's Conda environment has a newer version that includes GLIBCXX_3.4.30.

    Difference in Environment Activation:
        Your environment might be activated differently, causing a variation in the runtime library search path.

How to Fix the Issue
Option 1: Adjust LD_LIBRARY_PATH

    Modify LD_LIBRARY_PATH:

export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH

Run pytest:

    pytest

    Note: Be cautious, as changing LD_LIBRARY_PATH can affect other applications.

Option 2: Update Conda's libstdc++

    Update the Package:

conda update libstdcxx-ng

Verify the Symbol:

    strings /opt/conda/lib/libstdc++.so.6 | grep GLIBCXX_3.4.30

Option 3: Use Conda's Compiler to Rebuild

    Rebuild EnvWrapper:
        Use Conda's GCC to compile EnvWrapper, ensuring compatibility with Conda's libstdc++.

Option 4: Isolate the Conda Environment

    Deactivate Conda Environment:

conda deactivate

Reactivate Without Modifying LD_LIBRARY_PATH:

    CONDA_CHANGEPS1=false conda activate myenv

    Ensure LD_LIBRARY_PATH is Not Altered:
        Check that activating Conda does not prepend /opt/conda/lib to LD_LIBRARY_PATH.

Option 5: Modify the Import in Tests

    Lazy Importing:
        Change your test files to import EnvWrapper inside test functions instead of at the module level.

    def test_function():
        import install.bin.EnvWrapper as EnvWrapper
        # Rest of the test

    Reasoning:
        This delays the import until the test execution phase, potentially avoiding issues during test collection.

Conclusion

The discrepancy arises because, during your pytest run, the dynamic linker is loading libstdc++.so.6 from Conda's library directory (/opt/conda/lib), which lacks the required GLIBCXX_3.4.30 symbol. Your coworker's environment does not have this issue, possibly due to differences in LD_LIBRARY_PATH or the Conda libstdc++.so.6 version.

By adjusting LD_LIBRARY_PATH to prioritize system libraries or updating Conda's libstdc++, you can resolve the ImportError. Additionally, delaying the import of EnvWrapper in your tests or ensuring that your environment variables are consistent with your coworker's setup can help eliminate the discrepancy.
Recommended Steps

    Check and Compare LD_LIBRARY_PATH Values:

echo $LD_LIBRARY_PATH

Update Conda's libstdc++:

conda update libstdcxx-ng

Modify LD_LIBRARY_PATH Temporarily for Testing:

    export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
    pytest

    Discuss with Coworker:
        Compare environment variables and Conda package versions to identify any differences.

    Consider Rebuilding EnvWrapper:
        Use the same compiler and environment that matches your runtime setup.

By following these steps, you should be able to align your environment with your coworker's and resolve the ImportError.
You said:
Here is the output from running it with ld_debug. Also I check the runpath for that .so and it is to the system library folder correctly and that is the only runpath. So something strange is happening during the collection phase...here is debug from it:

============================= test session starts ==============================
platform linux -- Python 3.10.13, pytest-8.3.3, pluggy-1.5.0
rootdir: /home/jmoore2/i2dev/remote/ai-reinforcementlearning
configfile: pyproject.toml
plugins: anyio-4.6.0
collected 131 items / 1 error

==================================== ERRORS ====================================
___ ERROR collecting tests/utilities_tests/recursive_variant_pybind_test.py ____
ImportError while importing test module '/home/jmoore2/i2dev/remote/ai-reinforcementlearning/tests/utilities_tests/recursive_variant_pybind_test.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/conda/lib/python3.10/importlib/__init__.py:126: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
utilities_tests/recursive_variant_pybind_test.py:7: in <module>
    import install.bin.EnvWrapper as EnvWrapper
E   ImportError: /opt/conda/bin/../lib/libstdc++.so.6: version GLIBCXX_3.4.30' not found (required by /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
------------------------------- Captured stderr --------------------------------
     92525:	find library=libenvironmentsshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/lib/AdvanceYamlNodeWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/glibc-hwcaps/x86-64-v3/libenvironmentsshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/glibc-hwcaps/x86-64-v2/libenvironmentsshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/libenvironmentsshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/glibc-hwcaps/x86-64-v3/libenvironmentsshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/glibc-hwcaps/x86-64-v2/libenvironmentsshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libenvironmentsshared_d.so.1.0.0
     92525:	
     92525:	find library=libobservationsshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/lib/AdvanceYamlNodeWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/libobservationsshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libobservationsshared_d.so.1.0.0
     92525:	
     92525:	find library=librewardsshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/lib/AdvanceYamlNodeWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/librewardsshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/librewardsshared_d.so.1.0.0
     92525:	
     92525:	find library=libspacesshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/lib/AdvanceYamlNodeWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/libspacesshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libspacesshared_d.so.1.0.0
     92525:	
     92525:	find library=libexceptionutilitiesshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/lib/AdvanceYamlNodeWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/libexceptionutilitiesshared_d.so.1.0.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libexceptionutilitiesshared_d.so.1.0.0
     92525:	
     92525:	find library=libserviceshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libserviceshared_d.so.3.8.0
     92525:	
     92525:	find library=libserviceallocatorshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libserviceallocatorshared_d.so.1.0.0
     92525:	
     92525:	find library=libactionsshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libactionsshared_d.so.1.0.0
     92525:	
     92525:	find library=libterrainUtilitiesShared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libterrainUtilitiesShared_d.so.1.0.0
     92525:	
     92525:	find library=libthreadUtilitiesShared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libthreadUtilitiesShared_d.so.1.0.0
     92525:	
     92525:	find library=libgenericLibraryLoaderShared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libgenericLibraryLoaderShared_d.so.1.0.0
     92525:	
     92525:	find library=libdataextractionshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libdataextractionshared_d.so.1.0.0
     92525:	
     92525:	find library=libclairvoyanttrackrepositoryshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libclairvoyanttrackrepositoryshared_d.so.1.0.0
     92525:	
     92525:	find library=libinstructionsshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libinstructionsshared_d.so.1.0.0
     92525:	
     92525:	find library=libsensorlimitsrepositoryshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libsensorlimitsrepositoryshared_d.so.1.0.0
     92525:	
     92525:	find library=libpositionrepositoryshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libpositionrepositoryshared_d.so.1.0.0
     92525:	
     92525:	find library=libtrackrepositoryshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libtrackrepositoryshared_d.so.1.0.0
     92525:	
     92525:	find library=libstringUtilitiesShared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libstringUtilitiesShared_d.so.1.0.0
     92525:	
     92525:	find library=librepositoriesshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/librepositoriesshared_d.so.3.8.0
     92525:	
     92525:	find library=libterraindatarepositoryshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libterraindatarepositoryshared_d.so.1.0.0
     92525:	
     92525:	find library=libdatarecordershared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libdatarecordershared_d.so.1.0.0
     92525:	
     92525:	find library=libterrainshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libterrainshared_d.so.1.0.0
     92525:	
     92525:	find library=libmessagesshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libmessagesshared_d.so.3.8.0
     92525:	
     92525:	find library=libconfigshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libconfigshared_d.so.3.8.0
     92525:	
     92525:	find library=libtimelibshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libtimelibshared_d.so.3.8.0
     92525:	
     92525:	find library=libmapUtilitiesShared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libmapUtilitiesShared_d.so.1.0.0
     92525:	
     92525:	find library=libloggershared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libloggershared_d.so.3.8.0
     92525:	
     92525:	find library=libdataextractioninstructionparsershared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libdataextractioninstructionparsershared_d.so.1.0.0
     92525:	
     92525:	find library=libmathshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libmathshared_d.so.3.8.0
     92525:	
     92525:	find library=libvectorUtilitiesShared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libvectorUtilitiesShared_d.so.1.0.0
     92525:	
     92525:	find library=libtrackcorrelatorshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libtrackcorrelatorshared_d.so.1.0.0
     92525:	
     92525:	find library=libtracksshared_d.so.3.7.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libtracksshared_d.so.3.7.0
     92525:	
     92525:	find library=libtrackflatbufferfacadeshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libtrackflatbufferfacadeshared_d.so.1.0.0
     92525:	
     92525:	find library=libdismessageshared_d.so.3.7.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libdismessageshared_d.so.3.7.0
     92525:	
     92525:	find library=libgeogadgetryflatbufferfacadeshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libgeogadgetryflatbufferfacadeshared_d.so.1.0.0
     92525:	
     92525:	find library=libdisidflatbufferfacadeshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libdisidflatbufferfacadeshared_d.so.1.0.0
     92525:	
     92525:	find library=libsensorlimitsflatbufferfacadeshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libsensorlimitsflatbufferfacadeshared_d.so.1.0.0
     92525:	
     92525:	find library=libpositionflatbufferfacadeshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libpositionflatbufferfacadeshared_d.so.1.0.0
     92525:	
     92525:	find library=libterrainflatbufferfacadeshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libterrainflatbufferfacadeshared_d.so.1.0.0
     92525:	
     92525:	find library=libfilesystemutilitiesshared_d.so.1.0.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libfilesystemutilitiesshared_d.so.1.0.0
     92525:	
     92525:	find library=libfilelibshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libfilelibshared_d.so.3.8.0
     92525:	
     92525:	find library=libtoolbox_utilsshared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libtoolbox_utilsshared_d.so.3.8.0
     92525:	
     92525:	find library=libcdxmeshishared_d.so.3.8.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libcdxmeshishared_d.so.3.8.0
     92525:	
     92525:	find library=libkdis_d.so.0 [0]; searching
     92525:	 search path=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v3:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v2:/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu		(RUNPATH from file /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so)
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v3/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/glibc-hwcaps/x86-64-v2/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/lib/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v3/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/glibc-hwcaps/x86-64-v2/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v3/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/glibc-hwcaps/x86-64-v2/libkdis_d.so.0
     92525:	  trying file=/home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/../lib/../lib/x86_64-linux-gnu/libkdis_d.so.0
     92525:	
     92525:	/opt/conda/bin/../lib/libstdc++.so.6: error: version lookup error: version GLIBCXX_3.4.30' not found (required by /home/jmoore2/i2dev/remote/ai-reinforcementlearning/install/bin/EnvWrapper.cpython-310-x86_64-linux-gnu.so) (fatal)
=============================== warnings summary ===============================
../.venv/lib/python3.10/site-packages/pygame/pkgdata.py:25
  /home/jmoore2/i2dev/remote/ai-reinforcementlearning/.venv/lib/python3.10/site-packages/pygame/pkgdata.py:25: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    from pkg_resources import resource_stream, resource_exists

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
ERROR utilities_tests/recursive_variant_pybind_test.py
!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!
========================= 1 warning, 1 error in 1.62s ==========================
ChatGPT

Thank you for providing the LD_DEBUG output. This gives us valuable insight into what's happening during the test collection phase. Here's what's going on:
Key Observations from LD_DEBUG Output

    The dynamic linker is ultimately failing with:

    92525:	/opt/conda/bin/../lib/libstdc++.so.6: error: version lookup error: version `GLIBCXX_3.4.30' not found (required by /home/jmoore2/.../EnvWrapper.cpython-310-x86_64-linux-gnu.so) (fatal)

    Despite the .so file having the correct RUNPATH pointing to the system library folder, the dynamic linker is using /opt/conda/lib/libstdc++.so.6 instead of the system version.

Understanding the Issue

When the dynamic linker loads shared libraries, it follows a specific search order:

    Already Loaded Libraries:
        If a library with the same SONAME (e.g., libstdc++.so.6) is already loaded into the process, the linker reuses it.

    LD_PRELOAD:
        Libraries specified in LD_PRELOAD are loaded before any other libraries.

    DT_RPATH/DT_RUNPATH:
        The RUNPATH specified in the shared object is used to search for dependencies.

    LD_LIBRARY_PATH:
        The environment variable LD_LIBRARY_PATH is searched.

    Default System Paths:
        Standard system library paths like /lib, /usr/lib, etc.

In your case, the problem arises because:

    Conda's libstdc++.so.6 is already loaded into the process before your module is imported.

    This occurs during the test collection phase when other modules or the Python interpreter itself load Conda's libstdc++.so.6.

Why Does This Happen?

    Python Interpreter or Other Modules:
        The Python interpreter or some modules imported during test collection are linked against Conda's libstdc++.so.6, causing it to be loaded into the process early.

    Shared Library Reuse:
        When your module (EnvWrapper.cpython-310-x86_64-linux-gnu.so) tries to load libstdc++.so.6, the dynamic linker uses the already loaded version from Conda, which lacks GLIBCXX_3.4.30.

Why Does It Work When Running a Specific Test File?

    Reduced Imports During Collection:
        When running pytest utilities_tests/recursive_variant_pybind_test.py, fewer modules are imported during the collection phase.
    Delayed Loading:
        The critical module that triggers the loading of Conda's libstdc++.so.6 may not be imported until after your module is loaded, allowing your module to load the correct system libstdc++.so.6.

Confirming the Hypothesis
1. Check Which libstdc++.so.6 Is Loaded by Python

    Use lsof to List Open Files:

    lsof -p $(pgrep -f python)

    Check for libstdc++.so.6:
        Look for entries pointing to /opt/conda/lib/libstdc++.so.6.

2. Use ldd on Python Interpreter

    Run:

    ldd $(which python)

    Check for Dependencies:
        See if libstdc++.so.6 is listed and where it's coming from.

3. Use ldd on Other Imported Modules

    Identify Modules Imported During Collection:
        These could be modules in your site-packages that depend on libstdc++.so.6.

    Run ldd on These Modules:

    ldd path/to/module.so

    Check Their Dependencies:
        See if they depend on Conda's libstdc++.so.6.

Potential Solutions
Option 1: Update Conda's libstdc++.so.6

    Update the Package:

conda update libstdcxx-ng

Verify the Symbol Exists:

    strings /opt/conda/lib/libstdc++.so.6 | grep GLIBCXX_3.4.30

    Explanation:
        Updating ensures that Conda's version includes the required symbol.

Option 2: Adjust LD_PRELOAD

    Force the Use of System libstdc++.so.6:

    export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
    pytest

    Caution:
        This can have unintended side effects if other libraries expect the Conda version.

Option 3: Modify LD_LIBRARY_PATH

    Prepend System Library Path:

    export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
    pytest

    Explanation:
        This makes the dynamic linker search the system path before Conda's libraries.

Option 4: Rebuild EnvWrapper Against Conda's libstdc++.so.6

    Compile Using Conda's Compiler:
        Ensure that EnvWrapper is built using the same libstdc++.so.6 as in Conda.

    Explanation:
        This aligns your module with the Conda environment, avoiding symbol mismatches.

Option 5: Delay Imports in Tests

    Modify Test Files:

    def test_function():
        import install.bin.EnvWrapper as EnvWrapper
        # Test code here

    Explanation:
        Delaying the import may allow your module to load its dependencies before Conda's versions are loaded.

Recommended Approach

Update Conda's libstdc++.so.6 to Include GLIBCXX_3.4.30.

    This aligns with the fact that your coworker, running the same Docker container, doesn't face the issue—likely because their Conda environment is up-to-date.

Steps to Resolve

    Update Conda Packages

conda update libstdcxx-ng

Verify the Update

strings /opt/conda/lib/libstdc++.so.6 | grep GLIBCXX_3.4.30

Clean Conda's Cache (Optional)

    If issues persist, clean Conda's package cache:

    conda clean --all

Re-run Tests

    pytest

Additional Diagnostics

    Compare Environments with Coworker

        Conda Package Versions:

conda list

Environment Variables:

        env | grep LD_LIBRARY_PATH

    Check for Differences in Activation Scripts
        Ensure there are no custom scripts altering the environment differently on your machine.

Understanding the Underlying Mechanics

    Dynamic Linker Behavior:
        The dynamic linker avoids loading multiple versions of the same library (libstdc++.so.6).

    Order of Loading Libraries:
        Libraries loaded earlier take precedence.

    Impact of Conda Environment:
        Conda environments can introduce their own versions of libraries, which may conflict with system libraries if versions differ.

Why This Matters

    Consistency Across Environments:
        Ensuring that all developers have the same environment setup prevents such issues.

    Compatibility:
        Mixing libraries from different sources (system vs. Conda) can lead to symbol mismatches.

Conclusion

The error occurs because during the test collection phase, libstdc++.so.6 from Conda (which lacks GLIBCXX_3.4.30) is already loaded into the process. Updating Conda's libstdc++.so.6 to a version that includes the required symbol should resolve the issue.
Final Thoughts

    Avoid Forcing Library Loading Unless Necessary:
        While LD_PRELOAD and modifying LD_LIBRARY_PATH can work, they may introduce other issues.

    Aim for Environment Consistency:
        Keeping your Conda environment up-to-date ensures compatibility.

    Monitor Library Dependencies:
        Be aware of how different libraries and environments interact, especially in complex projects.

Let me know if you have any questions or need further assistance.