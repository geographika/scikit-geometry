# Windows Builds

This page documents building a (x64) Wheel binary for Windows using vcpkg (using PowerShell).  

A couple of modifications are required:

Errors with pybind - https://github.com/scikit-geometry/scikit-geometry/issues/113
Fixed with `git revert 868ef298591992cf1cc3328cd0cf970174e29a70`

Boost libraries needed to be renamed in setup.py or get:

```
# LINK : fatal error LNK1181: cannot open input file 'boost_thread.lib'
```

First setup vcpkg and install CGAL. See https://github.com/microsoft/vcpkg#quick-start-windows. 
Run the following in PowerShell (to run takes ~40 minutes):

```
cd D:\GitHub
git clone https://github.com/microsoft/vcpkg
cd .\vcpkg\
.\bootstrap-vcpkg.bat
./vcpkg install cgal:x64-windows
./vcpkg integrate install
```

Only works with a specific version of CGAL?
See D:\GitHub\vcpkg\ports\cgal\vcpkg.json


Now get scikit-geometry and build the wheel:

```
# git clone https://github.com/scikit-geometry
cd D:\GitHub\scikit-geometry
C:\Python312\python -m venv C:\VirtualEnvs\scikit-build
C:\VirtualEnvs\scikit-build\Scripts\activate.ps1

pip install wheel
pip install pybind11
pip install numpy
pip install setuptools

# run 
python setup.py bdist_wheel
```

# D:\GitHub\vcpkg\installed\x64-windows\include\CGAL/config.h(154): fatal error C1189: #error:  "CGAL requires C++ 17"

Test in a new venv:

```
C:\Python310\python -m venv C:\VirtualEnvs\scikit
C:\VirtualEnvs\scikit\Scripts\activate.ps1
pip install --force-reinstall D:\GitHub\scikit-geometry\dist\skgeom-0.1.2-cp310-cp310-win_amd64.whl
# without bundled DLLs need to set the path
python -c "import os; os.add_dll_directory('D:/GitHub/vcpkg/installed/x64-windows/bin'); import skgeom; print(skgeom.Point2(0, 5))"
# when bundled these are no longer needed
python -c "import skgeom; print(skgeom.Point2(0, 5))"

# for bundling issues check what the wheel installed
pip show -f skgeom
```

The following eerror `ImportError: DLL load failed while importing _skgeom: The specified module could not be found.`
means the DLLs are not found and may need to add os.add_dll_directory to the script. 

## Older Versions

Need CGAL 5.6 to work correctly or get errors.

cd D:\GitHub\vcpkg
./vcpkg list

./vcpkg remove cgal
./vcpkg install cgal

## Alternative Conda Install

See https://stackoverflow.com/questions/71591926/how-to-install-the-package-scikit-geometry
Sample script: http://scikit-geometry.github.io/scikit-geometry/skeleton.html

```
conda create --name scikit-env
conda activate scikit-env

conda config --add channels conda-forge
conda config --set channel_priority strict
conda install -c conda-forge scikit-geometry

python skeletons.py
```