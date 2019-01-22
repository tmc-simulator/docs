# TMC Simulator Documentation

This repository contains the documentation for the TMC Simulator project. The documentation is generated using the Sphinx python package.

## How to add documentation

1. Clone the repository `git clone git@github.com:tmc-simulator/docs.git`
1. Create the conda environment `conda env create -f environment.yml`.
1. Add the `.rst` file to the appropriate directory.
1. Add the new file's path to the index.rst file in the projects root directory.
1. Add contents to the `.rst` file.
1. Run `make html` and open the generated `_build/html/index.html` file to view.
