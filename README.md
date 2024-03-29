# busybash

[cbugk](https://github.com/cbugk)'s doctrine of writing
[busybox](https://www.busybox.net/) and
[bash](https://www.gnu.org/software/bash/) scripts.

## How to use?

Follow these steps to use this repository:

1. **Clone the Repository**: First, clone this repository to your local machine using the following command:

2. **Create a Program**: Next, create a directory for your program under the `cmd` directory. In this directory, you should add a `Main.sh` file that contains the main function of your program. You can also add an `env.sh` file if you want to separate variables from the `Main.sh` file.

3. **Add Main Function**: In the `Main.sh` file, define your main function. You can use the `WHO` variable to print a personalized message.

4. **Add Environment Variables**: If you want to separate variables from the `Main.sh` file, you can create an `env.sh` file in the same directory. In this file, you can define the `WHO` variable.

5. **Build the Program**: Once you've added your main script and environment variables, you can build your script using the build.sh script in the bb directory. This will combine your main script, environment variables, and any functions you've defined into a single script.

6. **Run the Program**: After building your script, you can run it using the generated script in the `out` directory.
```bash
git clone https://github.com/cbugk/busybash
cd busybash

mkdir cmd/hello

# Mind the tab characters and <<-
cat > cmd/hello/Main.sh <<-MAIN
	#!/bin/bash
	
	function Main() {
	  printf 'Hello World!\n'
	  printf "This is ${WHO}\n"
	}; export -f Main;
MAIN

# Mind that ${USER} will be evaluated
cat > cmd/hello/env.sh <<-ENVSH
	WHO=${USER}
ENVSH

chmod u+x ./bb/bind.sh # make executable
./bb/build.sh cmd/hello
./out/hello.sh
```

## Motivation

Shell scripting is a tedious work when performed without structure. Even then
it lacks the ease of use, which compilers provide for programing languages.

busybash repository aims to set a standard for writing, sharing, and deploying
my (cbugk) shell scripts. For instance, providing traceback and immediate halt
of execution are essential.


## Aimed at

I tried using ash and busybox only. Unfortunately, bashisms are, let alone
convienient, straight necessary to be able to use
`declare -n alias="${original_var}"` by-reference variable passing or
`export -f` function exports.

However, I do not think this is the case for GNU Utils; busybox should be
small, portable, and standard enough.

> Alpine-based containers are the base target.
> However, embedded linux systems can also benefit from this approach.

Note that, user code __is not subject__ to alpine/busybox restriction,
so _GNU core-utils divergences can be used_ on custom functions.


## Structure

Roles of directories:
* `bb`: Stores busybash commands
  * `bb/core`: Stores shell options, traps, and templates for creating
    monoscripts.
* `cmd`: Stores program entry point functions.
  * Imported at the end of generated scripts.
  * `main.bash` should be namespaced by resulting monoscript's name in case
    of multiple different scripts.
  * `env.sh` file beside `Main.sh` will be concatenated within and at the top
    of main function. Can be used to override generic `env.sh` file under
    _sct_.
* `fcn`: Stores any user function.
  * Scripts are imported by path, directory trees represent namespacing.
  * Functions and variables should conform to naming schema.
  * Scripts should only define and export a function.
* `sct`: Stores conventional scripts.
  * Should be __snake_case__ and have all lowercase letters.
  * `env.sh` should reside here and only define global variables.
* `out`: Stores bound monoscripts.
  * filename is the same as parent directory of entrypoint script
    (e.g. `Main.sh`)

Roles of `bb` scripts:
* `bind.sh`: Generates self-contained scripts from `main` functions.
* `run.sh`: Executes provided main function with dependencies sourced
  and shell arguments passed.
* `test.sh`: Checks unit test of a single function.


## Etiquette

Following are to overcome short-comings of shell scripting at the expense of
long variable/function names. These are my (cbugk) personal preferences,
betterments are welcome.
* File names:
  1. Script files must have the __same__ name as defined function without
     its namespace. Namespacing is done through directory structure.
  2. Script file extensions must be `.sh`.
  3. _Conventional scripts_ are placed under _sct_ and can be namespaced.
  4. _Conventional scripts_ should have a descriptive name for their task. It
     is suggested that, these only be used sparsely and instead _function_s be
     utilized.
* Globals:
  1. Globals are discouraged. Using globals to pass values is further
     discouraged. Exceptions are following, are expected to not change through
     out the programs life.
     * Functions.
     * __CONSTANT__s, either from shell or within `main()`.
  2. Any variable which needs to be used by another function must be
     explicitly provided as an _out-parameter_ into it. They are by-reference
     variable aliases or expansion indirection:
     * `var1=foobar; declare -n alias1=var1; printf "${alias1}\n"`
     * `var1=foobar; var2='var1'; printf "${!var2}\n`)
* Values/Properties:
  1. Names are __snake_case__ with lower-case characters only, e.g.
     `foo_bar="baz";`.
     * Use of two consecutive underscores (i.e. `__`) is reserved
       for namespacing thus prohibited.
     * Use of leading underscore (e.g. `_foo_bar`) is forbidden for namespacing
       as well.
  2. Variables must be expanded with `"${}"`, braces are obligatory.
     * This allows use of more than 9 parameters without `shift` in while
       loops, i.e. `"${10}"`.
  3. ___Each and every___ variable expansion must have quotation.
     * `declare -r foo="${2}bar"`, not `foo=$2bar`.
     * This is to make sure no empty expansion goes unnoticed and changes
       position or arguments of functions.
  4. Every property (variable) is declared with `declare`/`local`.
     * With `-i` flag for integers.
     * With `-r` flag for read-only values. __CONSTANT__ is not appropriate.
     * With `-n` flag for (pass-by-reference) _out-parameters_.
     * See [GNU BASH Docs](https://www.gnu.org/software/bash/manual/bash.html)
       for more
  5. Names must be uniqely identifiable. Namespacing is highly encouraged.
     * e.g. `baz__foo_bar` where `baz` is the namespace and `__` is variable
       namespace seperator.
* Functions:
  1. Names are __PascalCase__ e.g. `FooBar`, however, namespaces
     exceptionally are seperated by underscore e.g. `Abc_FooBar`.
  2. Functions are declared with __function__ keyword and __()__ is
     adjacent, whereas brace opening is not.
     * e.g. `function FooBar() {`
  3. To ensure subprocess function calls work, every function is exported right
     after being declared. Same holds for nested functions, if any.
     * e.g. `}; export -f FooBar;`.
  4. Names must be uniquely identifiable. Namespacing is highly encouraged.
     * e.g. `function Abc_FooBar() {` where `Abc` is the namespace and `_` is
     function namespace seperator.
  5. Properties should be namespaced with the owner function, e.g.
     `abc_foobar__baz`.
     * Although a mouthful, this overcomes unintentionally overwriting other
       functions' properties.
     * `abc_foobar` is the namespace for the variable `abc_foobar__baz`,
       whereas `Abc` is the namespace for the owner function `Abc_FooBar`.
  6. `local` is preffered over `declare` for function properties. Not a
     requirement though.
  7. _Tests_ are created alongside their respective functions.
     * Names should be prefixed with **TEST_**.
       * e.g. `function TEST_Abc_FooBar() {` for `function Abc_FooBar() {`.
     * Prefix is in capitals to distinguish from namespaces.
* Subprocesses:
  1. Use of `` `command` `` is highly discouraged, instead use
     `"$(some_command "$(another_command)")"` for readability and nestability
* Redirections:
  1. Place a space or a dash between here-doc and termination word.
     * To preserve initial tab characters, use `<< EOF`.
     * To remove tab characters and be able to allign code, use `<<-EOF`
  2. For here-doc termination word, try to avoid `EOF` and use something
     descriptive instead.
     * e.g. `<<-SCRIPT`, `<< CSV`, `<<-MOCK_DATA`.


## Post Scriptum

Remember, this is a shell scripting guide at best. Ideas here are an
amalgam of others'. Suggestions, issues, and merge requests are welcome.
