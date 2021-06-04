GraInvRect
===========
## 1 Introduction
It is a 3D gravity inversion program implemented with C++. The inversion is based on the rectilinear mesh in the Cartesian coordinate system. The inversion mesh can be adaptively refined to boost computational performance. Furthermore, a-priori information can be incorporated in inversion through cross-gradient coupling or direct parameter relationship.

Any combination of gravity field components or gravity gradient components ($g_z$, $g_x$, $g_y$, $T_{zz}$, $T_{xz}$, $T_{yz}$, $T_{xx}$, $T_{xy}$, $T_{yy}$) can be used as input data.  Exact analytical solutions of gravity field and gravity gradient tensor are used to ensure accuracy of the forward modeling. 

## 2 Installation

### 2.1 Dependencies
- [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page): A C++ template library for linear algebra (It has been contained in the ./contrib/ folder )
- [Linterp](https://rncarpio.github.io/linterp/): A multidimensional interpolation class which is dependent on the Boost library (It is contained in ./contrib folder)
- [netcdf](https://www.unidata.ucar.edu/software/netcdf/) [*optional*]: A library to write and read data in .nc format. NetCDF is a popular format to store and share data. Data of this format can be read in GMT (generic mapping tool) or python. 

### 2.2 Buiding

#### (1) Build

Build the program using Make,

```bash
cd GraInvRect
make
```
By default, the program is built without netcdf support. In this way, you don't have to install the netcdf library in advance, but there are not *.nc files generated after inversion. The results are written in  *.vtk file, which can be visulaized by [Paraview](https://www.paraview.org/).

#### (2) Use Intel compilers

The default compiler is g++. To build with an [Intel C++ compiler](https://software.intel.com/content/www/us/en/develop/tools/oneapi/base-toolkit/download.html) (`icpc`, `icpx`, `dpcpp`), use the CXX option,

```bash
make CXX=icpx
```

> The latest Intel compiler is the Intel oneAPI DPC++/C++ Compiler (now free to use), whose command is `icpx` or `dpcpp`. The classic one is `icpc`. It seems that the icpx or icpc shows a slightly better performance than g++.

#### (3) Build with NetCDF library

If you have installed netcdf library (including netcdf C++ interfaces) and wish to store your inversion model in .nc file, you can build the program with netcdf support by using the USE_NETCDF option,
```bash
make USE_NETCDF=1
```

In this way, you have to install netcdf-c and netcdf-cxx libraries in advance. 

The fastest way to install netcdf:
```shell
#For ubuntu users
sudo apt-get install libnetcdf-dev
sudo apt-get install libnetcdf-c++4-dev

#For Fedora users
sudo yum install netcdf-devel
sudo yum install netcdf-cxx-devel
```

> For CentOS user, recommend to build netcdf-c and netcdf-cxx from the source code. See https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html for netcdf-c and https://github.com/Unidata/netcdf-cxx4 for the netcdf c++ library.



## 3 Examples

### 3.1 A simple example
Now, you have 2 executable programs in the GraInvRect folder: **GraInvRect** and **Synthetic_data1**.

The **GraInvRect** program is the  inversion program, and **Synthetic_data1** is used to generate a synthetic data set for validation of the inversion program.

1. `cd GraInvRect/Examples` (change from the current directory to GraInvRect/Examples), run

```
../Synthetic_data1
```
to generate synthetic data with noise. Then we get a text file named *dobs_g_z* which contains vertical gravity data $g_z$, and a file named *dobs_T_zz* which contains vertical gravity gradient data $T_{zz}$. 

2. `cd GraInvRect/Examples/Inv_gz` In this folder, we have prepared the following configuration files:  **config**, **config_data**, **config_inversion** and **config_model**. 

   The file **config** is the only parameter required by the inversion program **GraInvRect**, which specifies the names of another 3 configuration files: **config_data**, **config_inversion** and **config_model**. 

   - The file **config_data** contains information about data. 

   - The file **config_inversion** specifies controlling parameters for the inversion. 

   - The file **config_model** specifies  the inversion region (size and initial mesh discretization). 

   These configuration files allow comments that start with '#'.  See the comments in the configuration files for detailed explanation. 

3. Then, let's run the inversion

```
../../GraInvRect config
```
After the inversion is finished,  the result is written into **gz_result.vtk** file. The vtk file can be visualized in [Paraview](https://www.paraview.org/). 

If `GraInvRect` is compiled with NetCDF (`make USE_NETCDF=1`), the resulting model is also written into a ***.nc** file. A python script `crossSections2.py`  is used to read the result in *.nc file and plot cross sections.

4. Similarly, `cd GraInvRect/Examples/Inv_Tzz` and run the inversion of $T_{zz}$ data with`../../GraInvRect config`.

### 3.2 Inversion using  multiple components of gravity gradient tensor

The file **config_data** tells the program which data to be used. 

1. Enter `Inv_Txz_Tyz_Tzz` folder,  open **config_data**, and compare it with the ones in `Inv_Tzz` folder and `Inv_gz` folder.
2. Run the inversion

```
../../GraInvRect config
```

3. Show the inversion result in Paraview, or using python:

```
python crossSections2.py
python Txz_Tyz_Tzz.py
```



### 3.3 Inversion using L1-norm regularization

1. Enter `L1_inv_gz` folder,  open **config_inversion**, and compare it with the one in `Inv_gz` folder. Line 14 in **config_inversion** contains 2 parameters about Lp-norm inversion. The first parameter is the "p" of Lp-norm, the second parameter is a small value which is used to avoid singularity.
2.  Run the inversion

```
../../GraInvRect config
```

3. Show the inversion result in Paraview, or using python:

```
python crossSections2.py
```



### 3.2 Examples of inversion with a priori information

A priori information (e.g. existing velocity models) can be included in the inversion to improve the reliability of the inversion result.  The a priori model should be gridded data given in a XYZ table.

#### (1) Reference and initial model

Using direct empirical relations between density and the known parameter, we can obtain a reference density model. In this algorithm, if a reference model is specified, it will also serve as the initial model. 

Here, we have prepared a file **ref_model** in the GraInvRect/Examples folder, which contains gridded data of a reference model.

1. cd GraInvRect/Examples/Inv_with_ref

```
../../GraInvRect config
```

2. Run

```
../../GraInvRect config
```

#### (2) Cross gradient constraint

1. cd GraInvRect/Examples/Inv_with_cross_gradient_constraint. Notice that the model used for cross gradient constraint is specified in file **config_inversion**, line 49 and line 54.

2. Run

```
../../GraRect config
```







