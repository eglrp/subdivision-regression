subdivision-regression
======================

![Doo-Sabin subdivision regression](https://github.com/rstebbing/subdivision-regression/raw/master/README.png)

This repository provides the code necessary to fit Doo-Sabin subdivision surfaces to unstructured 3D point data. It is the 3D analog of [Uniform B-spline Regression][1].

Author: Richard Stebbing

License: MIT (refer to LICENSE).

Dependencies
------------
Core:
* [Ceres Solver][3]
* [rstebbing/subdivision][6]
* [rstebbing/common][5]
* [gflags][4]
* [rapidjson][2]

Python Example & Visualisation:
* Numpy
* Scipy
* scikit-learn
* VTK

Building
--------
To build `doosabin_regression`:

1. Run CMake with an out of source build.
2. Set `COMMON_CPP_INCLUDE_DIR` to the full path to [rstebbing/common/cpp](https://github.com/rstebbing/common/tree/master/cpp).
3. Set `DOOSABIN_INCLUDE_DIR` to the full path to [rstebbing/subdivision/cpp/doosabin/include](https://github.com/rstebbing/subdivision/tree/master/cpp/doosabin/include).
3. Set `Ceres_DIR` to the directory containing `CeresConfig.cmake`.
4. Set `GFLAGS_INCLUDE_DIR`, `GFLAGS_LIBRARY` and `RAPID_JSON_INCLUDE_DIR`.
(Add `-std=c++11` to `CMAKE_CXX_FLAGS` if compiling with gcc.)
5. Configure.
6. Build.

Example
-------
To generate an example regression problem:
```
python python/generate_example_doosabin_regression_problem.py 2048 example-2048.json --seed=0
```

The generated JSON file contains five fields: `Y`, `raw_face_array`, `X`, `p`, and `U`:
* `Y` is the flattened `(num_data_points, 3)` array of data points. Here, they are synthetically generated by uniformly sampling a sphere and displacing the points radially with zero-mean Gaussian noise.
* `raw_face_array` is the mesh topology (see [`sequence_to_raw_face_array`](https://github.com/rstebbing/common/blob/master/rscommon/face_array.py#L13)) that defines the Doo-Sabin subdivision surface,
* `X` is the flattened `(num_control_points, 3)` array of control points that defines the _geometry_ of the Doo-Sabin surface.
* Each data point has an (unknown) _correspondence_ (_preimage_) on the surface. `p` is the array of `(num_data_points,)` patch indices; `U` is the flattened `(num_data_points, 2)` array of patch coordinates.

To visualise the problem and initial state:
```
python python/visualise_doosabin_regression.py example-2048.json
```
(Add `-d U` to disable the tubes between each data point and its correspondence.)

The purpose of `doosabin_regression` is to update `X`, `U`, and `p` so that the evaluated surface "fits" the data points `Y`. To run:
```
<build-directory>/bin/doosabin_regression example-2048.json 1e-4 example-2048-1.json -v 1 --min_trust_region_radius=1e-6 --max_num_iterations=10
```
where the compiled binary is located at `<build-directory>`, `1e-4` is the amount of regularisation ("force" betwen mesh control points), `example-2048-1.json` is the output path, `-v 1` enables verbose output, and `--min_trust_region_radius=1e-6` and `--max_num_iterations=10` control termination of the optimiser.

To visualise:
```
python python/visualise_doosabin_regression.py example-2048-1.json
```

Additional arguments to [python/generate_example_doosabin_regression_problem.py](https://github.com/rstebbing/subdivision-regression/blob/master/python/generate_example_doosabin_regression_problem.py), [python/visualise_doosabin_regression.py](https://github.com/rstebbing/subdivision-regression/blob/master/python/visualise_doosabin_regression.py), and [doosabin_regression](https://github.com/rstebbing/subdivision-regression/blob/master/src/doosabin_regression.cpp) can be found with `--help`.

[1]: https://github.com/rstebbing/bspline-regression
[2]: https://github.com/miloyip/rapidjson
[3]: https://ceres-solver.org/building.html#building-installation
[4]: https://github.com/gflags/gflags
[5]: https://github.com/rstebbing/common
[6]: https://github.com/rstebbing/subdivision