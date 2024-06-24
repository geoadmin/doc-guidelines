# Python Packaging

Some python tool, framework or library might be published on PyPI index. This way they can be easily shared within projects or within the community.

## Table of Content

- [Table of Content](#table-of-content)
- [Create python package](#create-python-package)
- [License](#license)
- [setup.py](#setuppy)
- [Makefile](#makefile)
- [Publishing](#publishing)

## Create python package

Python packages should be created using the official [PyPA](https://packaging.python.org/tutorials/packaging-projects/) best practices.

The following rules needs to be applied:

- version should follow [Semantic Versioning 2.0.0](https://semver.org/)
  - NOTE: version 0.x.x are for unstable release and don't appears in pypi search
- version should be placed in the `<package-dir>/__init__.py` file as follow

    ```python
    VERSION = (0, 1, 0, 'alpha0')
    if isinstance(VERSION[-1], str):
        # Support for alpha version: 0.1.0-alpha1
        __version__ = "-".join([".".join(map(str, VERSION[:-1])), VERSION[-1]])
    else:
        __version__ = ".".join(map(str, VERSION))
    ```

    **NOTE:** remove the `'alpha0'` string in the `VERSION` tuple for official version

- version should be read from `<package-dir>/__init__.py` in the _setup.py_ file
- long description in _setup.py_ should be read from _README.md_
- package MUST have a license
- license should be _BSD 3-Clause License_ (see [License](#license))
- package should have unittest
- package should be verified by a travis project

## License

```text
BSD 3-Clause License

Copyright (c) 2020, swisstopo
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
this list of conditions and the following disclaimer in the documentation
and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its
contributors may be used to endorse or promote products derived from
this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

## setup.py

Here below an example of the _setup.py_ file

```python
#!/usr/bin/env python

from setuptools import setup, find_packages

# Get description from README.md
LONG_DESCRIPTION = ''
with open('README.md', encoding='utf-8') as rd:
    LONG_DESCRIPTION = rd.read()

# Here we cannot import the version but need to read it otherwise we might have an ImportError
# during execution of the setup.py if the package contains other libraries.
VERSION_LINE = list(filter(lambda l: l.startswith('VERSION'),
                           open('./logging_utilities/__init__.py')))[0]


def get_version(version_line):
    # pylint: disable=eval-used
    version_tuple = eval(version_line.split('=')[-1])
    return ".".join(map(str, version_tuple))


setup(
    name='logging-utilities',
    version=get_version(VERSION_LINE),
    description=('Python logging utilities'),
    long_description=LONG_DESCRIPTION,
    long_description_content_type="text/markdown",
    platforms=["all"],
    classifiers=[
        'Development Status :: 3 - Alpha',
        'Intended Audience :: Developers',
        'License :: OSI Approved :: BSD License',
        'Operating System :: OS Independent',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3 :: Only',
        'Topic :: Utilities',
        'Topic :: System :: Logging'
    ],
    python_requires='>=3.0',
    author='ltshb',
    author_email='brice.schaffner@swisstopo.ch',
    url='https://github.com/geoadmin/lib-py-logging-utilities',
    license='BSD 3-Clause License',
    packages=find_packages()
)
```

## Makefile

In order to simplify the packaging and publishing the following makefile targets should be added

```makefile
# general targets timestamps
TIMESTAMPS = .timestamps
PREP_PACKAGING_TIMESTAMP = $(TIMESTAMPS)/.prep-packaging.timestamp

# PyPI credentials
PYPI_USER ?=
PYPI_PASSWORD ?=

# Get package version from package info file
PACKAGE_VERSION = $(shell awk '/^Version:/ {print $$2}' logging_utilities.egg-info/PKG-INFO)

.PHONY: help
help:
	@echo "Usage: make <target>"
	@echo
	@echo "Possible targets:"
	@echo -e " \033[1mPACKAGING TARGETS\033[0m "
	@echo "- package            Create package"
	@echo "- publish            Tag and publish package to PyPI"
	@echo -e " \033[1mCLEANING TARGETS\033[0m "
	@echo "- clean              Clean generated files"

# Packaging target

.PHONY: package
package: $(PREP_PACKAGING_TIMESTAMP)
	python3 setup.py sdist bdist_wheel


.PHONY: publish
publish: publish-check clean test package
	@echo "Tag and upload package version=$(PACKAGE_VERSION)"
	@# Check if we use interactive credentials or not
	@if [ -n "$(PYPI_PASSWORD)" ]; then \
	    python3 -m twine upload -u $(PYPI_USER) -p $(PYPI_PASSWORD) dist/*; \
	else \
	    python3 -m twine upload dist/*; \
	fi
	git tag -am $(PACKAGE_VERSION) $(PACKAGE_VERSION)
	git push origin $(PACKAGE_VERSION)


.PHONY: clean
clean:
	rm -rf $(TIMESTAMPS)
	rm -rf dist
	rm -rf build
	rm -rf *.egg-info


$(TIMESTAMPS):
	mkdir -p $(TIMESTAMPS)

$(PREP_PACKAGING_TIMESTAMP): $(TIMESTAMPS)
	python3 -m pip install --user --upgrade setuptools wheel twine
	@touch $(PREP_PACKAGING_TIMESTAMP)


publish-check:
	@echo "Check if publish is allowed"
	@if [ -n "`git status --porcelain`" ]; then echo "ERROR: Repo is dirty !" >&2; exit 1; fi
	@# "Check if TAG=${PACKAGE_VERSION} already exits"
	@if [ -n "`git ls-remote --tags --refs origin refs/tags/${PACKAGE_VERSION}`" ]; then \
		echo "ERROR: Tag ${PACKAGE_VERSION} already exists on remote" >&2;  \
		exit 1; \
	fi
```

## Publishing

In order to publish to PyPI index we use the *pypi/swisstopo* user. This user can be found in _gopass_ and you can use _summon_ to publish the package as follow:

```shell
summon -p gopass make publish
```
