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

```
cd D:\GitHub
git clone https://github.com/microsoft/vcpkg
bootstrap-vcpkg.bat
vcpkg install cgal:x64-windows
vcpkg integrate install
```

Now get scikit-geometry and build the wheel:

```
cd D:\GitHub\scikit-geometry
git clone https://github.com/scikit-geometry
C:\Python310\Scripts\pip install wheel
C:\Python310\Scripts\pip install pybind11
C:\Python310\Scripts\pip install numpy
C:\Python310\python setup.py bdist_wheel
```

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