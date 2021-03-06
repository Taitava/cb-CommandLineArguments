# CommandLineArguments

A [POSIX style](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html) commandline option/argument parser for [CoolBasic](https://coolbasic.com).

Say that you call your program from the commandline like this: `MyProgram.exe --output-file "C:\My output.txt" -c 1 -n --verbose -xyz -- get everything after two dashes into a single argument`
This library helps you parse that string of options and arguments like so:
 - `--output-file` is a long option (can have a short alias too, e.g. `-o`) that takes an argument. The argument happens to contain a space, so it's enclosed in double quotes. CLA removes the double quotes at the end for you.
 - `-c` is a short option (can have a long alias too, e.g.  `--count`) and takes an argument. No quotes are needed for this particular argument.
 - `-n` is a short option without an argument.
 - `--verbose` is a long option without an argument.
 - `-xyz` are three short options joined together. It's the same as `-x -y -z`, and the order does not matter, it could be e.g. `-zxy` too. The last option can also have an argument assigned to it: `-xyz some-argument`. 
 - `-- terminate options and get everything after two dashes into a single argument` The two dashes (without an option name, meaning separated by a space from whatever comes next) will terminate option parsing and include the rest of the command line into a single non-option argument.

While CoolBasic programs are designed for Windows only, this library is still designed not to use the Windows style command line options (e.g. `MyProgram.exe /N`). I'm just more familiar with the POSIX style.

## Requirements
- [CoolBasic](https://coolbasic.com) development environment.
- [GetWord2() & CountWords2()](http://www.cbrepository.com/codes/code/11/) functions need to be included separately.

## Installation
1. Make sure that you have `GetWord2()` and `CountWords2()` functions included in your program.
2. `Include "CommandLineArguments.cb"` in your program.

You do not need to include anything from the *tests* folder. It contains [unit tests](https://github.com/Taitava/cbUnit) that only need to be used if you develop or debug this library.

## Usage

### 1. Define your accepted commandline options
Call `DefineCommandLineOptions(options$)` which takes just one parameter. `options$` is a string consisting of any of the following words:
- `--` denotes a long option name, e.g. `--verbose`.
- `-` denotes a short option name, e.g. `-v`.
- `:` denotes both a long and a short option name, e.g. `--verbose:v` (also `--verbose:-v` is valid)
- `+` after a short/long option name tells that this option takes an argument and the argument is mandatory. E.g. `--output-file+` or `-o+` or `--output-file:-o+`.
- `*` as a last word in the whole string defines that *non-option arguments* are allowed. Otherwise they are denied.
- `--` (without a trailing option name) anywhere in the definition string denotes that it's allowed to use `--` in the command line to terminate option parsing, in which case the rest of the command line string will be read as a single argument into an option named `--`.
- Each word must be separated by a space
- Complete example: `DefineCommandLineOptions("--verbose:-v -c+ -n *")`

*Non-option arguments*: values in the commandline that do not have an option name assigned with them. For example executing `MyProgram.exe output.txt` uses a *non-option argument* *output.txt*, whereas `MyProgram.exe --output-file output.txt` uses an *option* which has *output.txt* as its *argument*.  Depending on how you want to design your options, you can choose if you want to enable *non-option arguments* or not. They are disabled by default.

`DefineCommandLineOptions(options$)` returns true if it was able to understand your option definitions, false otherwise. This phase is just a predefinition, and it does not yet parse the commandline string in any way.

### 2. Parse the commandline string
Ensure that you **call DefineCommandLineOptions() before** trying to parse the commandline string!

Call `ParseCommandLineOptions()`. It will read CoolBasic's `CommandLine()` value and split it into parts. For an example of the format accepted by `ParseCommandLineOptions()`, see [the beginning of this readme](#CommandLineArguments).

The function returns true if the parsing succeeded or false if the program was executed with either undefined option names, non-option arguments (if those are disabled), or illegal format. You can then display your own error message that guides which options can be used. Even if the result is false, some options may be already parsed and usable, but not all.

If you want the function to use another source than `CommandLine()` for the commandline string, just pass it a custom string as a parameter (optional). This is used at least for unit testing, but you do not need to do that.

### 3. Read options

- Call `isCommandLineOptionPresent(option_name$)` to see if a specific option was presented in the commandline.
- Call `getCommandLineArgument(option_name$, if_not_present$="")` to get an argument related to a specific option. Use the optional second parameter to define what should be returned if the option was not present in the commandline.

To access *non-option arguments*, there is currently no function support for that (`getCommandLineArgument()` will support it in the future), but you can iterate the `CommandLineNonOptionArgument` type list.

Counts of *options* and *non-option arguments* can be read from:
 - `CountCommandLineOptionsDefined` - How many options you have defined in the code.
 - `CountCommandLineOptionsPresent` - How many of them are present in the commanline string.
 - `CountCommandLineNonOptionArgumentsPresent`

Currently there is no function support for getting all options that are present in the commandline with one function call.

## Contributing

Ideas, bug reports, and pull requests are all welcome. Use the GitHub issue tracker to get in touch! :)

## Author
This library is written by me, Jarkko Linnanvirta.