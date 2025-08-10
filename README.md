# phs - PipeLine Haskell Script

A simple command-line tool that applies Haskell functions to each line of standard input.

## Philosophy

`phs` brings the elegance of Haskell's pure functions to the UNIX shell. It's designed as a spiritual successor to AWK, replacing imperative scripting with functional composition. The tool embodies the perfect synergy between Haskell's concise, elegant functions and shell pipelines - two paradigms that have always been naturally compatible.

Where AWK requires verbose syntax for simple transformations, `phs` leverages Haskell's point-free style and rich standard library to express complex operations in just a few characters. This isn't just about shorter code - it's about bringing type-safe, composable, and mathematically elegant transformations to everyday shell scripting.

## Overview

`phs` (PipeLine Haskell Script) is a lightweight script for text processing in pipelines. You can specify any Haskell function to transform each line of input. The function can be of any type `String -> a` where `a` has a Show instance, making it incredibly flexible - from simple string transformations to complex data analysis.

With just 3 simple options (`-c` for config, `-a` for all-lines mode, `-h` for help), it maintains UNIX tool simplicity while offering the full power of Haskell's type system and standard library.

## Requirements

- GHC (Glasgow Haskell Compiler)

## Built-in Functions

`phs` provides commonly used functions from `Data.List` and `Data.Char` as built-ins:

- **List functions**: `sort`, `nub`, `group`
- **Character functions**: `isDigit`, `isAlpha`, `isSpace`, `toUpper`, `toLower`

These are available without any imports or configuration:

```bash
# Character functions
echo "Hello123" | ./phs 'filter isDigit'    # 123
echo "hello" | ./phs 'map toUpper'          # HELLO
echo "HELLO" | ./phs 'map toLower'          # hello

# List functions (with --all mode)
printf "3\n1\n2" | ./phs --all 'sort'       # ["1","2","3"]
echo -e "a\nb\na\nc" | ./phs --all 'nub'    # ["a","b","c"]
echo "aabbcc" | ./phs 'group'               # ["aa","bb","cc"]
```

## Installation

```bash
git clone https://github.com/Kohei-Wada/phs.git
cd phs
chmod +x phs
```

## Usage

### Command Line Options

```bash
# Use default config (~/.phsrc)
command | phs [FUNCTION]

# Specify custom config file
command | phs -c /path/to/config.hs [FUNCTION]
command | phs --config ./mylib.hs 'myCustomFunc'

# Process all lines at once (instead of line-by-line)
command | phs --all [FUNCTION]
command | phs -a 'sort'

# Show help
phs -h
phs --help
```

### Basic Usage

```bash
# Default (id function) - outputs input as-is
echo "hello world" | ./phs

# String -> Int: Show length of each line
echo "hello world" | ./phs 'length'
# Output: 11

# String -> String: Reverse each line
echo "hello world" | ./phs 'reverse'
# Output: dlrow olleh

# String -> Bool: Check if all digits
echo "12345" | ./phs 'all isDigit'
# Output: True

# String -> [String]: Split into words
echo "hello world" | ./phs 'words'
# Output: ["hello","world"]

# String -> (Int, String): Return tuple with length and original
echo "test" | ./phs '\s -> (length s, s)'
# Output: (4,"test")
```

### Processing All Lines at Once (--all mode)

With the `--all` option, the function receives all lines as a list `[String]` instead of processing each line individually:

```bash
# Sum all numbers
seq 1 10 | ./phs --all 'sum . map read'
# Output: 55

# Sort all lines
printf "cherry\napple\nbanana" | ./phs --all 'sort'
# Output: ["apple","banana","cherry"]

# Take first 5 lines
seq 1 100 | ./phs --all 'take 5'
# Output: ["1","2","3","4","5"]

# Reverse line order (not line content)
printf "first\nsecond\nthird" | ./phs --all 'reverse'
# Output: ["third","second","first"]

# Count total lines
ls -1 | ./phs --all 'length'
# Output: 42

# Get unique lines
echo -e "a\nb\na\nc\nb" | ./phs --all 'nub'
# Output: ["a","b","c"]

# Join all lines into one
echo -e "hello\nworld" | ./phs --all 'unwords'
# Output: "hello world"
```

### Processing Files

```bash
# Process each line of a file
cat file.txt | ./phs 'reverse'

# Add line numbers
cat file.txt | nl | ./phs 'id'
```

### Practical Examples

```bash
# Show character count for each line
ls -1 | ./phs 'length'

# Convert filenames to uppercase
ls -1 | ./phs 'map toUpper'

```

### AWK vs phs Comparison

```bash
# Count characters - AWK
awk '{print length($0)}'
# phs
./phs 'length'

# Select first 10 characters - AWK
awk '{print substr($0, 1, 10)}'
# phs
./phs 'take 10'

# Filter lines with uppercase only - AWK
awk '/^[A-Z]+$/'
# phs
./phs 'filter isUpper'

# Sum numbers on each line - AWK
awk '{sum=0; for(i=1;i<=NF;i++) sum+=$i; print sum}'
# phs
./phs 'sum . map read . words'

# Fibonacci sequence - AWK (complex iterative approach)
awk 'BEGIN{a=1;b=1;for(i=1;i<=10;i++){print a;c=a+b;a=b;b=c}}'
# phs (elegant recursive definition)
seq 1 10 | ./phs 'let fib n = if n <= 2 then 1 else fib (n-1) + fib (n-2) in fib . read'

# Power set - AWK (would require complex multi-dimensional arrays)
# Practically impossible to implement cleanly in AWK
# phs (mathematical elegance)
echo "abc" | ./phs 'let powerset [] = [[]]; powerset (x:xs) = powerset xs ++ map (x:) (powerset xs) in powerset'

# Collatz conjecture - AWK (verbose imperative code)
awk '{n=$1; while(n>1){if(n%2==0)n=n/2;else n=3*n+1; print n}}'
# phs (concise functional definition)
seq 10 | ./phs 'let collatz n = if n == 1 then [1] else n : collatz (if even n then n `div` 2 else 3*n+1) in collatz . read'
```

The power of Haskell's function composition and recursive definitions shines through in mathematical computations that would require verbose, imperative code in AWK.

## Configuration

You can define custom functions by creating a `~/.phsrc` file. This file should contain valid Haskell function definitions that will be available in all phs invocations.

You can also specify a different config file using the `-c` or `--config` option:

```bash
# Use project-specific functions
cat data.txt | phs -c ./project-functions.hs 'processData'

# Use different function libraries
echo "test" | phs --config ~/lib/string-utils.hs 'normalize'
```

### Example ~/.phsrc

```haskell
-- Column extraction (1-indexed)
col :: Int -> String -> String
col n = (!! (n-1)) . words

-- Trim whitespace
trim :: String -> String
trim = dropWhile isSpace . reverse . dropWhile isSpace . reverse

-- Get file basename
basename :: String -> String
basename = reverse . takeWhile (/= '/') . reverse

-- Convert to lowercase
lower :: String -> String
lower = map toLower
```

With these defined, you can use:

```bash
# Extract 5th column
ls -l | ./phs 'col 5'

# Trim whitespace
echo "  hello  " | ./phs 'trim'

# Get basename from paths
find . -type f | ./phs 'basename'

# Convert to lowercase
echo "HELLO" | ./phs 'lower'
```

See `.phsrc.example` for more custom function examples.

## Design Philosophy: Standard Library Only

`phs` deliberately uses only GHC's standard library - no external packages. This is a feature, not a limitation:

- **Zero dependencies** - Works immediately on any system with GHC
- **No package management** - No cabal, stack, or dependency resolution needed
- **True portability** - Copy the script anywhere and it just works
- **UNIX philosophy** - Like `grep` or `sed`, it does one thing well with built-in capabilities

If you need external libraries (JSON parsing, HTTP requests, database access), that's a sign you should create a proper Haskell project. `phs` is for quick, elegant text transformations - the sweet spot between shell scripting and full Haskell development.

```bash
# phs excels at simple, elegant transformations
cat data.csv | ./phs 'filter (not . null)'  # Remove empty lines
ls -la | ./phs 'words >>> (!! 8)'           # Extract filename column

# Need more? Pipe to a real Haskell program
cat raw.json | ./phs 'lines' | stack exec json-processor
```

## How It Works

`phs` internally uses GHC's evaluation mode (`ghc -e`) to dynamically construct and execute a Haskell program. The implementation is surprisingly simple - just 122 lines of bash that:

1. Parses command-line options
2. Loads custom functions from config file (if exists)
3. Constructs a Haskell program with the user's function
4. Executes it via `ghc -e`

In default mode, it applies the function to each line (`map f . lines`). With `--all`, it applies the function to all lines at once (`f . lines`). Custom functions from config files are injected into the `let` binding, making them immediately available.

## License

MIT

## Author

Kohei Wada
