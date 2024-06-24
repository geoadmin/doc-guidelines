# Bash coding guidelines

There are a number of styleguides out there. Check what [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html) proposes, we largely follow this document. A few important things are reproduced here.

- [1. When to use BASH scripts](#1-when-to-use-bash-scripts)
- [2. Best Practices](#2-best-practices)
  - [basic shell script structure](#basic-shell-script-structure)
    - [1. shebang header](#1-shebang-header)
    - [2. function sections](#2-function-sections)
    - [3. main part of the script](#3-main-part-of-the-script)
  - [Variable expansion](#variable-expansion)
  - [Strict mode](#strict-mode)
    - [This option also has some downsides](#this-option-also-has-some-downsides)
  - [Trap](#trap)
  - [Multiline comments](#multiline-comments)
  - [Bash functions command groups vs sub shell](#bash-functions-command-groups-vs-sub-shell)
- [3. Linting / shellcheck](#3-linting--shellcheck)
  - [shellcheck options](#shellcheck-options)
- [4. Unit Tests / shellspec](#4-unit-tests--shellspec)
  - [Installation of shellspec](#installation-of-shellspec)
  - [Add new unit tests](#add-new-unit-tests)
  - [Execute unit tests](#execute-unit-tests)
  - [mocking](#mocking)
    - [overwrite functions / variables](#overwrite-functions--variables)
    - [set up / tear down functions for more complex input (folders and files, json objects, etc.)](#set-up--tear-down-functions-for-more-complex-input-folders-and-files-json-objects-etc)
  - [To be noted](#to-be-noted)

With this [handy tool](https://explainshell.com/) you can analyze and explain bash command-lines.

**The foremost goal is that reading and understanding your bash script is easy for someone else (or yourself in a few months time or after a memory reset)**

**Another important goal is to follow the ``Parting words`` section in this document: https://google.github.io/styleguide/cppguide.html#Parting_Words**

## 1. When to use BASH scripts

Bash scripts should only be used for:

- helper scripts
- wrapper scripts (p.e data integration ogr / psql wrapper, db deploy scripts, ...)
- small utilities

See also this overview [here](https://opensource.com/article/19/4/bash-vs-python) for a comparison of bash and python pros and cons.

If you decide to write a bash script, make sure to follow the descriptions and best practices below.

## 2. Best Practices

### basic shell script structure

A basic Bash script has three sections.

#### 1. shebang header

The first line of the bash script should be the shebang header, ``#!/bin/bash`` in our case for Bourne-again Shell.

#### 2. function sections

In this section you should

- define functions, variables and constants
- include other scripts

At the end of this section you should add this line:

```bash
[ "$0" = "${BASH_SOURCE[*]}" ] || return 0
```

Everything under this line wont be executed when you source the file with

```bash
$ source script.s
# or
$ . script.sh
```

#### 3. main part of the script

This can be a single bash statement, several function calls or thousands of lines of code.

### Variable expansion

Variable or parameter expansion is a really powerful feature in bash.
For example, you can react to different states of variables and assign default values:

```bash
   +----------------------+------------+-----------------------+-----------------------+
   |   if VARIABLE is:    |    set     |         empty         |        unset          |
   +----------------------+------------+-----------------------+-----------------------+
 - |  ${VARIABLE-default} | $VARIABLE  |          ""           |       "default"       |
 = |  ${VARIABLE=default} | $VARIABLE  |          ""           | $(VARIABLE="default") |
 ? |  ${VARIABLE?default} | $VARIABLE  |          ""           |       exit 127        |
 + |  ${VARIABLE+default} | "default"  |       "default"       |          ""           |
   +----------------------+------------+-----------------------+-----------------------+
:- | ${VARIABLE:-default} | $VARIABLE  |       "default"       |       "default"       |
:= | ${VARIABLE:=default} | $VARIABLE  | $(VARIABLE="default") | $(VARIABLE="default") |
:? | ${VARIABLE:?default} | $VARIABLE  |       exit 127        |       exit 127        |
:+ | ${VARIABLE:+default} | "default"  |          ""           |          ""           |
   +----------------------+------------+-----------------------+-----------------------+
```

[Here](https://wiki-dev.bash-hackers.org/syntax/pe#indirection) you will find more examples of what else is possible with variable expansion.

### Strict mode

In order to make our scripts more solid, we always set the following options on top of the script:

```bash
set -euo pipefail
```

The ``set -e`` option instructs bash to immediately exit if any command [1] has a non-zero exit status.
The ``set -u`` option affects variables. When set, a reference to any variable you haven't previously defined - with the exceptions of ``$*`` and ``$@`` - is an error, and causes the program to immediately exit.

#### This option also has some downsides

if you try to **test if a variable exists**. In that case it is recommended to use variable expansion:

```bash
# For those that are looking to check for unset or empty when in a script with set -u:
if [ -z "${var-}" ]; then
   echo "Must provide var environment variable. Exiting...."
   exit 1
fi
```

if you try to count the number of elements in an optional array (which could also be a variable of type text), you would have to use something like this:

```bash
if declare -p my_array | grep -q '^declare \-a'; then
        length=${#my_array[@]}
else
        length=0
fi
```

The ``set -o pipefail`` option will make the bash shell look at the exit code of all the commands in a pipeline. Without this option, the shell option ``-e`` only reacts on the exit status of the last command of the pipeline.

### Trap

Trap allows you to catch system signals and some user signals and execute code when they occur. This is useful when you want to do some clean-up, remove lock files or unblock external resources etc. when the script exits abnormally (or normally).
You will find a lot of examples in our [data processing scripts](https://github.com/geoadmin/bgdi-scripts/search?q=trap&unscoped_q=trap).

### Multiline comments

multiline comments are best done this way:

```bash
#!/bin/bash
: '
This is a
very neat comment
in bash
'
```

### Bash functions command groups vs sub shell

There are two ways to to [group commands](https://www.gnu.org/software/bash/manual/html_node/Command-Grouping.html) and create functions in bash:

```bash
function_1() {
# function defined with command group

# commands in a command group are executed in the current shell
# they have read-write access to all the variables of the current shell
# exit will stop the current shell and the script itself
}

# or
function_2() (
# function defined with sub shell

# the scope of all variables defined here is limited to the subshell
# you cant change the environment variables of the calling shell
# exit will kill stop the subshell only
)
```

try read and understand first, when you're editing an existing script: https://google.github.io/styleguide/cppguide.html#Parting_Words

## 3. Linting / shellcheck

Each script should be checked with [shellcheck](https://github.com/koalaman/shellcheck).

ShellCheck is a static analysis and linting tool for sh/bash scripts, the goals of ShellCheck are

- To point out and clarify typical beginner's syntax issues that cause a shell to give cryptic error messages.
- To point out and clarify typical intermediate level semantic problems that cause a shell to behave strangely and counter-intuitively.
- To point out subtle caveats, corner cases and pitfalls that may cause an advanced user's otherwise working script to fail under future circumstances.

There are different ways to use shellcheck. You can use shellcheck from the terminal or you can add it as plugin to vi, nvim, tmux, etc.

### shellcheck options

You can configure shellcheck with [directives](https://github.com/koalaman/shellcheck/wiki/Directive).
If you want to exclude some shellcheck codes from the report you can define the excluded codes either as command line argument:

```text
       -e CODE1[,CODE2...], --exclude=CODE1[,CODE2...]
              Explicitly exclude the specified codes from the report.  Subsequent -e options  are
              cumulative, but all the codes can be specified at once, comma-separated as a single
              argument.
```

or as a comment in the first line after the shebang of the file:

```bash
#!/bin/bash
# shellcheck disable=CODE1,CODE2,...
```

Starting with version ``0.7.0`` of shellcheck you can set most of the options in a ``.shellcheckrc`` rc file in the project’s root directory (or your home directory). This allows you to easily apply the same options to all scripts on a per-project/directory basis.

We are using the following options:

- executables

   ```bash
   # SC2029: https://github.com/koalaman/shellcheck/wiki/SC2029 you can disable this check if you want your string to be expanded client-side
   # SC2087: https://github.com/koalaman/shellcheck/wiki/SC2087 you can disable this check if you want the here document to be expanded on client-side
   disable=SC2029,SC2087
   ```

- libraries, includes

   ```bash
   # SC2034: https://github.com/koalaman/shellcheck/wiki/SC2034 you can disable this check if you declare variables which are used in the sourcing scripts
   disable=SC2034
   ```

## 4. Unit Tests / shellspec

Bash unit tests can be configured and executed with the [shellspec framework](https://shellspec.info/).
The shellspec framework has been chosen because it is:

- powerful
- easy to configure
- has different reporting formats (p.e TAP, JUnit)
- can be used in CI
- based on pure shell
- supports different shells
- can be installed anywhere ``curl <url> | sh``

documentation : [md files on github](https://github.com/shellspec/shellspec/blob/master/docs)

Currently there are bash unit tests for the following projects:

- [dataprocessing crontab scripts / includes](https://github.com/geoadmin/bgdi-scripts/blob/master/geodata_cron_skripte/bgdipg01t/spec/bgdipg01t_spec.sh)
- [db deploy script](https://github.com/geoadmin/deploy/blob/master/db_master_deploy/spec/db_master_deploy_spec.sh)

### Installation of shellspec

shellcheck can be installed anywhere with:

```bash
curl -fsSL https://git.io/shellspec | sh -s <<SHELLSPEC_VERSION>> -p <<INSTALL_DIRECTORY>> -y
```

If you install it to a custom directory, make sure to add the directory to the PATH variable.
Please check the documentation for further information about installation, configuration, etc.

You can also run it with docker using the image described here https://github.com/geoadmin/docker-shellspec

In our projects we have a Makefile target for the installation and execution of the unit tests.

```bash
# installation
make all
# execution
make bash_unit_tests
```

### Add new unit tests

If you want to create unit tests, you have execute the following command in the folder with the shell scripts first:

```bash
shellspec --init

# this will create the following files and folders:
#  create   .shellspec  -> configuration of default options
#  create   spec/spec_helper.sh 
#  create   spec/geodata_cron_skripte_spec.sh -> <<current folder>>_spec.sh
```

Then you can add the unit tests to the file ``spec/<<current_folder>>_spec.sh``.
Once the tests are ready, do not forget to add them to github.

### Execute unit tests

Make sure to go down to the parent folder of the unit test folder ``spec`` before you run the tests:

```bash
cd geodata_cron_skripte/bgdipg01t/
# run the tests
shellspec -s bash spec/bgdipg01t_spec.sh
# or choose an example by pattern
shellspec -s bash spec/bgdipg01t_spec.sh --example '*data_env*'
```

### mocking

You can simulate a **stable input** for your unit tests with **mocking**. Mocking is creating objects or function input that simulate the behavior of real objects. There are several ways to create your stable unit test environment.

#### overwrite functions / variables

You can find a simple example of mocking [here](https://shellspec.info/#easy-to-mock--stub).

#### set up / tear down functions for more complex input (folders and files, json objects, etc.)

you can initialize your test data / input / objects with a function which is executed at the beginning of the unit tests and which is creating the test data / input / objects in a reproducible / isolated way.

```bash
Describe 'some unit tests which need test folder with test data as input'
Include ./file_with_functions_to_test.sh
mock_set_up() {
# create test data for the following unit tests
# we need a folder mock_data with one file
# mock_data/
# └── input.txt

rm -rf mock_data &> /dev/null || :
cat << EOF > "mock_data/input.txt"
Title: unit test
Date and time: 2017-08-25T11:00+02:00
Abstract: Bondo Rockfall 23.08.2017, quick orthophoto from 25.08.2017 taken with the aerial sensor ADS100, GSD 14cm
Link to viewer: https://s.geo.admin.ch/86166fc2f1
Link to event:
Image type: RGBN
EOF 
}
# Set Up / Initialize Test data
mock_set_up

Sample 'function1_success'
...
End

Sample 'function1_fail'
# if you first test has changed the test data, re-run the function
mock_set_up
...
End
...

# Tear Down
# In the end clear test data
rm -rf mock_data &> /dev/null || :
End
```

You will find more examples in the unit tests folder of the cron scripts: ``geodata_cron_skripte/bgdipg01t/spec/bgdipg01t_spec.sh``

### To be noted

- If you want to write unit tests on the functions of an existing script you have to include the script in the shellspec Example / unit tests with ``Include myscript.sh``
Before you include an existing script, make sure that nothing is executed when you Include the file. Just add the following line before the first execution of a command in the script:

```bash
# do not execute anything if file is sourced for unit tests
[ "$0" = "$BASH_SOURCE" ] || return 0
```

- There are different ways to run commands or function. Please read the [documentation](https://github.com/shellspec/shellspec/blob/7f6dd12d356d5259bbcf83e7ee7fd6b25a885b37/docs/references.md#example) about that.
In most cases you can use either
  - ``When call`` or
  - ``When run``

With ``When call`` you have access to the variables which are defined inside the function
p.e. ``The variable RESULT_MAIL_CONTENT should include ">f+++++++++ event1/data.tif"``.
