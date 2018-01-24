# Example conda recipes for creating selectable, ordered build variants.  

## Introduction

This repository contains conda recipes which create a number of [build
variants](https://conda.io/docs/user-guide/tasks/build-packages/variants.html)
of an fictitious **tflow** package.  A particular variant can be selected using
the variant specific meta-packages, **tflow-default**, **tflow-sandybridge**,
**tflow-broadwell**, and **tflow-gpu**.  The preference for installing the
default variant of the tflow package is specified by the **_tflow_variant**
package which has two versions, 1.0 and 0.0, which are used to define the
default and non-default variants.  Two downstream packages, provide examples of
package which require any of the variants, **downstream_foo**, or a particular
variant, **downstream_bar**.

Packages created from these example recipes for the linux-64 platform can be
found with the ["variant_example" label in the jjhelmus
channel](https://anaconda.org/jjhelmus/repo?label=variant_example) on Anaconda
Cloud. Conda can be used with the `-c jjhelmus/label/variant_example` argument
to select these packages.

## Build variants

Introduced in conda build 3, [build
variants](https://conda.io/docs/user-guide/tasks/build-packages/variants.html)
allows multiple packages with the same name but different build configurations
to be created. These build variants can be used to create packages with
different usage requirements.  These differences may be a result of differences
in binary compatibility; for example variants which require different versions
of the libhdf5 library, different levels of optimization; variants which are
optimized for specific CPU architectures, or different features enabled;
variants which support GPU acceleration or include GUI support. 

## Build variant selection challenges

One challenge with build variants is the inability for users to select an
appropriate variant. Build variants are presented as packages with build string
which contain hashes, py36h69895a4 vs py36h9c2be89_0. There is no simple method
to determine the build configuration from the hash.  When build variants
provide packages with different binary compatibility this is not a large
concern as other packages in the conda environment will cause the correct
variant to be selected.  If build variants are used to provide packages with
different levels of optimization or package features, this lack of selection is
concerning. Users would like to select a variant with a particular optimization
or certain features enabled but cannot.

## Variant specific meta-packages

One solution to this variant selection challenge is *variant specific
meta-packages*.  These are conda packages with a descriptive name which install
a specific variant of the base package.  

In the example in this repository, the base package is a fictitious python
library named **tflow** which can can be built with different CPU optimization
as well as with GPU support.  In the example, these different build
configurations are represented by different `tflow.py` files in the recipe but
in a real software these would likely be the result of different compiler
arguments or environment flags which effect the build process.  The variant
specific meta-packages, **tflow-default**, **tflow-sandybridge**,
**tflow-broadwell**, and **tflow-gpu** are created at the same time as the
**tflow** variant package using [multiple
outputs](https://conda.io/docs/user-guide/tasks/build-packages/define-metadata.html#outputs-section)
and have a run requirement on the **tflow** build variant they were created
with.  

Installing a variant specific meta-package will select the correct **tflow**
variant package.  For example `conda install tflow-gpu` will install the
**tflow** variant which was build with GPU support.  

It is not possible for two variant specific meta-packages to be installed at
the same time as they require different versions of the base package.
**tflow-sandybridge** and **tflow-broadwell** cannot be install at the same
time as each requires a different build of the **tflow** package.

## Variant ordering meta-packages

Build variants and variant specific meta-packages allow packages with different
build configurations to be created and selected for installation.  Using conda
to install a variant specific meta-package ensures that a particular variant is
installed.  Unfortunately, the behavior when installing the base package is not
well defined.  `conda install tflow-gpu` will install a tflow variant with GPU
support, but the variant installed via `conda install tflow` is unspecified.  

To solve this issue a **variant ordering meta-package** can be used.  This is
an empty conda packages whose version number is used to define a default
variant.  Conda prefers to install a higher version of package and will install
a variant which requires a higher version of the variant ordering meta-package
over one which requires a lower version.

In the example, **_tflow_variant** is used as a variant ordering meta-package.
Two versions of the package are available, 1.0 and 0.0.  The default **tflow**
variant, which the **tflow-default** meta-package selects, requires version 1.0
of this package.  The other, non-default, build variants requires version 0.0.
`conda install tflow` will install the default **tflow** variant which
maximizes the version of **_tflow_variant**.  `conda install tflow-gpu` or
other variant specific meta-packages will install version 0.0 of the
**_tflow_variant** package.

In addition to the version number, custom build strings of "default" and
"not_default" were defined for the **_tflow_variant** meta-package.  The
**tflow** recipes pin both the version and build string of the
**_tflow_variant** package.  Although not necessary, this is done to make it
more explicit if a variant is the default or non-default variant.  Expressing
the requirement as `_tflow_variant =1=default` and `_tflow =0=not_defaults`
hould lead to fewer mistakes than `_tflow_variant =1` and `_tflow_variant =0`. 

## Use in downstream packages

Downstream packages can require either a specific variant or the base package
in which case any variant can be installed. The example **downstream_foo**
library works with any variant of the **tflow** package and uses the tflow to
express this requirement.  The **downstream_bar** library requires a GPU
accelerated variant of tflow and requires the **tflow-gpu** package to express
this requirement.

## User experience

Example of the user experience with the example tflow package are show below.
This assumes that jjhelmus/labl/variant_example has been added to the list of
channels, if not add `-c jjhelmus/label/variant_example` to the commands.

### search

Conda users can search for tflow packages from the command line.  The result
show the multiple variants.

```
~$ conda search tflow
Loading channels: done
Name                       Version                   Build  Channel        
tflow                      1.0              py36h69895a4_0  jjhelmus/label/variant_example
tflow                      1.0              py36h9c2be89_0  jjhelmus/label/variant_example
tflow                      1.0              py36hdd89744_0  jjhelmus/label/variant_example
tflow                      1.0              py36he58b317_0  jjhelmus/label/variant_example
```

Another will discover the variant specific meta-package which they can install.

```
~$ conda search "tflow-*"
Loading channels: done
Name                       Version                   Build  Channel        
tflow-broadwell            1.0                  he58b317_0  jjhelmus/label/variant_example
tflow-default              1.0                  h69895a4_0  jjhelmus/label/variant_example
tflow-gpu                  1.0                  hdd89744_0  jjhelmus/label/variant_example
tflow-sandybridge          1.0                  h9c2be89_0  jjhelmus/label/variant_example
```

### install a specific variant

A specific variant can be installed via the command line:

```
~$ conda create -n flower python=3.6 tflow-sandybridge
Solving environment: done

## Package Plan ##

  environment location: /home/jhelmus/anaconda3/envs/flower

  added / updated specs: 
    - python=3.6
    - tflow-sandybridge


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    _tflow_variant-0.0         |      not_default           4 KB  jjhelmus/label/variant_example
    tflow-1.0                  |   py36h9c2be89_0           5 KB  jjhelmus/label/variant_example
    tflow-sandybridge-1.0      |       h9c2be89_0           5 KB  jjhelmus/label/variant_example
    ------------------------------------------------------------
                                           Total:          14 KB

The following NEW packages will be INSTALLED:

    _tflow_variant:    0.0-not_default       jjhelmus/label/variant_example
    ca-certificates:   2017.08.26-h1d4fec5_0 defaults                      
    libedit:           3.1-heed3624_0        defaults                      
    libffi:            3.2.1-hd88cf55_4      defaults                      
    libgcc-ng:         7.2.0-h7cc24e2_2      defaults                      
    libstdcxx-ng:      7.2.0-h7a57d05_2      defaults                      
    ncurses:           6.0-h9df7e31_2        defaults                      
    openssl:           1.0.2n-hb7f436b_0     defaults                      
    python:            3.6.4-hc3d631a_1      defaults                      
    readline:          7.0-ha6073c6_4        defaults                      
    sqlite:            3.21.0-h1bed415_0     defaults                      
    tflow:             1.0-py36h9c2be89_0    jjhelmus/label/variant_example
    tflow-sandybridge: 1.0-h9c2be89_0        jjhelmus/label/variant_example
    tk:                8.6.7-hc745277_3      defaults                      
    xz:                5.2.3-h55aa19d_2      defaults                      
    zlib:              1.2.11-ha838bed_2     defaults                      

Proceed ([y]/n)? y


Downloading and Extracting Packages
_tflow_variant 0.0: ################################################################################### | 100% 
tflow 1.0: ############################################################################################ | 100% 
tflow-sandybridge 1.0: ################################################################################ | 100% 
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use:
# > source activate flower
#
# To deactivate an active environment, use:
# > source deactivate
#

~$ source activate flower
(flower) ~$ python -c "import tflow"
Hello from tflow, I am the sandybridge version, I have optimizations
```

### switch to a different variant

A different variant can be installed and the other variant will be removed.

```
(flower) ~$ conda install tflow-gpu
Solving environment: done

## Package Plan ##

  environment location: /home/jhelmus/anaconda3/envs/flower

  added / updated specs: 
    - tflow-gpu


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    tflow-gpu-1.0              |       hdd89744_0           5 KB  jjhelmus/label/variant_example
    tflow-1.0                  |   py36hdd89744_0           5 KB  jjhelmus/label/variant_example
    ------------------------------------------------------------
                                           Total:          10 KB

The following NEW packages will be INSTALLED:

    tflow-gpu:         1.0-hdd89744_0     jjhelmus/label/variant_example

The following packages will be REMOVED:

    tflow-sandybridge: 1.0-h9c2be89_0     jjhelmus/label/variant_example

The following packages will be UPDATED:

    tflow:             1.0-py36h9c2be89_0 jjhelmus/label/variant_example --> 1.0-py36hdd89744_0 jjhelmus/label/variant_example

Proceed ([y]/n)? y


Downloading and Extracting Packages
tflow-gpu 1.0:                                                                                          |   0% tflow-gpu 1.0: ######################################################################################## | 100% 
tflow 1.0: ############################################################################################ | 100% 
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
(flower) ~$ python -c "import tflow"
Hello from tflow, I am the gpu version, I support GPUs!
```

Packages are not automatically remove in conda 4.3:

```
(flower) ~$ python -c "import tflow"
Hello from tflow, I am the sandybridge version, I have optimizations
(flower) ~$ conda install tflow-gpu
Fetching package metadata .............
Solving package specifications: .

UnsatisfiableError: The following specifications were found to be in conflict:
  - tflow-gpu -> tflow 1.0 py36hdd89744_0
  - tflow-sandybridge -> tflow 1.0 py36h9c2be89_0
Use "conda info <package>" to see the dependencies for each package.
```

### install any variant

Installing tflow directly install the default variant.

```
~$ conda create -n tulip python=3.6 tflow
Solving environment: done

## Package Plan ##

  environment location: /home/jhelmus/anaconda3/envs/tulip

  added / updated specs: 
    - python=3.6
    - tflow


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    _tflow_variant-1.0         |          default           4 KB  jjhelmus/label/variant_example
    tflow-1.0                  |   py36h69895a4_0           5 KB  jjhelmus/label/variant_example
    ------------------------------------------------------------
                                           Total:           9 KB

The following NEW packages will be INSTALLED:

    _tflow_variant:  1.0-default           jjhelmus/label/variant_example
    ca-certificates: 2017.08.26-h1d4fec5_0 defaults                      
    libedit:         3.1-heed3624_0        defaults                      
    libffi:          3.2.1-hd88cf55_4      defaults                      
    libgcc-ng:       7.2.0-h7cc24e2_2      defaults                      
    libstdcxx-ng:    7.2.0-h7a57d05_2      defaults                      
    ncurses:         6.0-h9df7e31_2        defaults                      
    openssl:         1.0.2n-hb7f436b_0     defaults                      
    python:          3.6.4-hc3d631a_1      defaults                      
    readline:        7.0-ha6073c6_4        defaults                      
    sqlite:          3.21.0-h1bed415_0     defaults                      
    tflow:           1.0-py36h69895a4_0    jjhelmus/label/variant_example
    tk:              8.6.7-hc745277_3      defaults                      
    xz:              5.2.3-h55aa19d_2      defaults                      
    zlib:            1.2.11-ha838bed_2     defaults                      

Proceed ([y]/n)? y


Downloading and Extracting Packages
_tflow_variant 1.0: ################################################################################### | 100% 
tflow 1.0: ############################################################################################ | 100% 
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate tulip
#
# To deactivate an active environment, use
#
#     $ conda deactivate

~$ conda activate tulip
(tulip) ~$ python -c "import tflow"
Hello from tflow, I am the default version, I have no optimizations.
```
