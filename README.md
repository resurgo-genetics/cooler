# Cooler

[![Build Status](https://travis-ci.org/mirnylab/cooler.svg?branch=master)](https://travis-ci.org/mirnylab/cooler)
[![Documentation Status](https://readthedocs.org/projects/cooler/badge/?version=latest)](http://cooler.readthedocs.org/en/latest/)
[![Binder](http://mybinder.org/badge.svg)](http://mybinder.org:/repo/mirnylab/cooler-binder)

## A cool place to store your Hi-C

Cooler is a support library for a **sparse, compressed, binary** persistent storage format for Hi-C contact matrices, called `cool`, which is based on HDF5.

Cooler aims to provide the following functionality:

- Generate contact matrices from contact lists at arbitrary resolutions.
- Store contact matrices efficiently in `cool` format based on the widely used [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format) container format.
- Perform out-of-core genome wide contact matrix normalization (a.k.a. balancing)
- Perform fast range queries on a contact matrix.
- Convert contact matrices between formats.
- Provide a clean and well-documented Python API to work with Hi-C data.


To get started:

- Documentation is available [here](http://cooler.readthedocs.org/en/latest/).
- Walkthrough with a [Jupyter notebook](https://github.com/mirnylab/cooler-binder).
- Some published data sets are available at `ftp://cooler.csail.mit.edu/coolers`.


### Installation

Requirements:

- Python 2.7/3.3+
- libhdf5 and Python packages `numpy`, `scipy`, `pandas`, `h5py`. We highly recommend using the `conda` package manager to install scientific packages like these. To get it, you can either install the full [Anaconda](https://www.continuum.io/downloads) Python distribution or just the standalone [conda](http://conda.pydata.org/miniconda.html) package manager.

Install from PyPI using pip.
```sh
$ pip install cooler
```

See the [docs](http://cooler.readthedocs.org/en/latest/) for more information.


### Command line interface

The `cooler` library includes utilities for creating and querying `cool` files and for performing contact matrix balancing on a `cool` file of any resolution.

```bash
$ cooler makebins $CHROMSIZES_FILE $BINSIZE > bins.10kb.bed
$ cooler cload bins.10kb.bed $CONTACTS_FILE out.cool
$ cooler balance -p 10 out.cool
$ cooler dump -b -t pixels --header --join -r chr3:10,000,000-12,000,000 -r2 chr17 out.cool | head
```

```
chrom1  start1  end1    chrom2  start2  end2    count   balanced
chr3    10000000        10010000        chr17   0       10000   1       0.810766
chr3    10000000        10010000        chr17   520000  530000  1       1.2055
chr3    10000000        10010000        chr17   640000  650000  1       0.587372
chr3    10000000        10010000        chr17   900000  910000  1       1.02558
chr3    10000000        10010000        chr17   1030000 1040000 1       0.718195
chr3    10000000        10010000        chr17   1320000 1330000 1       0.803212
chr3    10000000        10010000        chr17   1500000 1510000 1       0.925146
chr3    10000000        10010000        chr17   1750000 1760000 1       0.950326
chr3    10000000        10010000        chr17   1800000 1810000 1       0.745982
```

See also:

- [CLI Reference](http://cooler.readthedocs.io/en/latest/cli.html).
- Jupyter Notebook [walkthrough](https://github.com/mirnylab/cooler-binder).

### Python API

The `cooler` library provides a thin wrapper over the excellent [h5py](http://docs.h5py.org/en/latest/) Python interface to HDF5. It supports creation of cooler files and the following types of **range queries** on the data:

- Tabular selections are retrieved as Pandas DataFrames and Series.
- Matrix  selections are retrieved as SciPy sparse matrices.
- Metadata is retrieved as a json-serializable Python dictionary.
- Range queries can be supplied using either integer bin indexes or genomic coordinate intervals.

```python

>>> import cooler
>>> import matplotlib.pyplot as plt
>>> c = cooler.Cooler('bigDataset.cool')
>>> resolution = c.info['bin-size']
>>> mat = c.matrix(balance=True).fetch('chr5:10,000,000-15,000,000')
>>> plt.matshow(np.log10(mat.toarray()), cmap='YlOrRd')
```

```python
>>> import multiprocessing as mp
>>> import h5py
>>> pool = mp.Pool(8)
>>> f = h5py.File('bigDataset.cool', 'r')
>>> weights = cooler.ice.iterative_correction(f, map=pool.map, ignore_diags=3, min_nnz=10)
```

See also:

- [API Reference](http://cooler.readthedocs.io/en/latest/api.html).
- Jupyter Notebook [walkthrough](https://github.com/mirnylab/cooler-binder).

### Schema

The `cool` [format](http://cooler.readthedocs.io/en/latest/datamodel.html) implements a simple schema that stores a contact matrix in a sparse representation, crucial for developing robust tools for use on increasingly high resolution Hi-C data sets, including streaming and [out-of-core](https://en.wikipedia.org/wiki/Out-of-core_algorithm) algorithms.

The data tables in a `cool` file are stored in a **columnar** representation as HDF5 groups of 1D array datasets of equal length. The contact matrix itself is stored as a single table containing only the **nonzero upper triangle** pixels.


### Contributing

[Pull requests](https://akrabat.com/the-beginners-guide-to-contributing-to-a-github-project/) are welcome. The current requirements for testing are `nose` and `mock`.

For development, clone and install in "editable" (i.e. development) mode with the `-e` option. This way you can also pull changes on the fly.
```sh
$ git clone https://github.com/mirnylab/cooler.git
$ cd cooler
$ pip install -e .
```

### License

BSD (New)
