
<!-- README.md is generated from README.Rmd. Please edit that file -->

# dplyr-cli

This a fork of a very nice tool written as a Rscript uses the `Rscript`
executable to run dplyr commands on CSV files in the terminal. This
version, however, changes the code so it reads any filetype supported by
`data.table::fread()` and manipulates the data as a `data.table` object.

`dplyr-cli` makes use of the terminal pipe `|` instead of the magrittr
pipe (`%>%`) to run sequences of commands.

    cat mtcars.csv | group_by cyl | summarise "mpg = mean(mpg)" | kable
    #> | cyl|      mpg|
    #> |---:|--------:|
    #> |   4| 26.66364|
    #> |   6| 19.74286|
    #> |   8| 15.10000|

## Motivation

This modification of the original code was a result of necessity while
working with genotype data. It is by no means the quickest way of doing
this type of manipulation but it reduces learning curve while keeping
the process relatively quick due to `data.table` use.

## What dplyr commands are supported?

Any command of the form:

  - `dplyr::verb(.data, code)`
  - `dplyr::*_join(.data, .rhs)`

Currently two extra commands are supported which are not part of
`dplyr`.

  - `csv` performs no dplyr command, but only outputs the input data as
    CSV to stdout
  - `kable` performs no dplyr command, but only outputs the input data
    as a `knitr::kable()` formatted string to stdout

## Limitations

  - Only tested under ‘bash’ and ‘zsh’ on OSX and Linux. YMMV.
  - Every command runs in a separate R session.
  - When using special shell characters such as `()`, you’ll have to
    quote your code arguments. Some shells will require more quoting
    than others.
  - “joins” (such as `left_join`) do not currently let you specify the
    `by` argument, so there must be columns in common to both dataset

## Usage

``` sh
dplyr --help
```

    #  dplyr-cli      [ldearaujo@unimelb.edu.au, mikefc@coolbutuseless.com]
    #  
    #  Run chains of dplyr commands in the terminal. This update
    #  allows guessing and fast reading with fread, as well as,
    #  managing the data as fast as possible with data.table
    #  
    #  * run any dplyr command of the form dplyr::verb(.data, code)
    #  * can set input file to be a CSV or RDS file
    #  * if reading data from stdin (the default), assume that it is CSV format
    #  
    #  History
    #    v0.1.0  2020-04-20 Initial release
    #    v0.1.1  2020-04-21 Switch to Rscript executable
    #    v0.1.2  2020-04-21 Support for joins
    #    v0.1.3  2020-04-22 More robust tmpdir handling
    #    v0.1.4  2022-01-23 Fix for newer read_csv handling
    #    v0.2.0  2020-06-02 General guessing with fread
    #  
    #  You may encounter errors in the first ever run, due to package installation. Test a second time if this is the case.
    #  
    #  Usage:
    #      dplyr <command> [--file=fn] [--csv | -c] [--verbose | -v] [<code>...]
    #      dplyr -h | --help
    #  
    #  Options:
    #      -h --help            show this help text
    #      --version     Show version.
    #      -f FILE --file=FILE  input CSV or RDS filename. If reading from stdin, assumes CSV [default: stdin]
    #      -c --csv             write output to stdout in CSV format (instead of default RDS file)
    #      -v --verbose         be verbose

## History

#### v0.1.0 2020-04-20

  - Initial release

#### v0.1.1 2020-04-21

  - Switch to ‘Rscript’ for easier install for users
  - rename ‘dplyr.sh’ to just ‘dplyr’

#### v0.1.2 2020-04-21

  - Support for joins e.g. `left_join`

#### v0.1.3 2020-04-22

  - More robust tmpdir handling

#### v0.1.4 2022-01-23

  - Fix handling for latest `read_csv()`. Fixes \#9

#### v0.2.0 2020-06-02

  - Converted to use fread and data.table

## Contributors

  - [coolbutuseless](https://github.com/coolbutuseless) - original
    author
  - [aborusso](https://github.com/aborruso) - documentation

## Installation

Because this script straddles a great divide between R and the shell,
you need to ensure both are set up correctly for this to work.

1.  \~Install R packages\~. This was also edited to perform installation
    of dependencies in first run. So in the very first run, expect it
    not to work. I still have to figure out a way of echoing meaninful
    warnings in this case.
2.  Clone this repo and put `dplyr` in your path

#### Clone this repo and put `dplyr` in your path

You’ll then need to download the shell script from this repository and
put `dplyr` somewhere in your path.

    git clone https://github.com/coolbutuseless/dplyr-cli
    cp dplyr-cli/dplyr ./somewhere/in/your/search/path

# Example data

Put an example CSV file on the filesystem. Note: This CSV file is now
included as `mtcars.csv` as part of this git repository, as is a second
CSV file for demonstrating joins - `cyl.csv`

``` r
write.csv(mtcars, "mtcars.csv", row.names = FALSE)
```

# Example 1 - Basic Usage

``` sh
# cat contents of input CSV into dplyr-cli.  
# Use '-c' to output CSV if this is the final step
cat mtcars.csv | dplyr filter -c "mpg == 21"
```

    #  "mpg","cyl","disp","hp","drat","wt","qsec","vs","am","gear","carb"
    #  21,6,160,110,3.9,2.62,16.46,0,1,4,4
    #  21,6,160,110,3.9,2.875,17.02,0,1,4,4

``` sh
# Put quotes around any commands which contain special characters like <>()
cat mtcars.csv | dplyr filter -c "mpg < 11"
```

    #  "mpg","cyl","disp","hp","drat","wt","qsec","vs","am","gear","carb"
    #  10.4,8,472,205,2.93,5.25,17.98,0,0,3,4
    #  10.4,8,460,215,3,5.424,17.82,0,0,3,4

``` sh
# Combine dplyr commands with shell 'head' command
dplyr select --file mtcars.csv -c cyl | head -n 6
```

    #  "cyl"
    #  6
    #  6
    #  4
    #  6
    #  8

# Example 2 - Simple piping of commands (with shell pipe, not magrittr pipe)

``` sh
cat mtcars.csv | \
   dplyr mutate "cyl2 = 2 * cyl"  | \
   dplyr filter "cyl == 8" | \
   dplyr kable
```

    #  /tmp//dplyr-cli-9c12d5ec64f9a.rds
    #  /tmp//dplyr-cli-9c12e45f38c90.rds
    #  |  mpg| cyl|  disp|  hp| drat|    wt|  qsec| vs| am| gear| carb| cyl2|
    #  |----:|---:|-----:|---:|----:|-----:|-----:|--:|--:|----:|----:|----:|
    #  | 18.7|   8| 360.0| 175| 3.15| 3.440| 17.02|  0|  0|    3|    2|   16|
    #  | 14.3|   8| 360.0| 245| 3.21| 3.570| 15.84|  0|  0|    3|    4|   16|
    #  | 16.4|   8| 275.8| 180| 3.07| 4.070| 17.40|  0|  0|    3|    3|   16|
    #  | 17.3|   8| 275.8| 180| 3.07| 3.730| 17.60|  0|  0|    3|    3|   16|
    #  | 15.2|   8| 275.8| 180| 3.07| 3.780| 18.00|  0|  0|    3|    3|   16|
    #  | 10.4|   8| 472.0| 205| 2.93| 5.250| 17.98|  0|  0|    3|    4|   16|
    #  | 10.4|   8| 460.0| 215| 3.00| 5.424| 17.82|  0|  0|    3|    4|   16|
    #  | 14.7|   8| 440.0| 230| 3.23| 5.345| 17.42|  0|  0|    3|    4|   16|
    #  | 15.5|   8| 318.0| 150| 2.76| 3.520| 16.87|  0|  0|    3|    2|   16|
    #  | 15.2|   8| 304.0| 150| 3.15| 3.435| 17.30|  0|  0|    3|    2|   16|
    #  | 13.3|   8| 350.0| 245| 3.73| 3.840| 15.41|  0|  0|    3|    4|   16|
    #  | 19.2|   8| 400.0| 175| 3.08| 3.845| 17.05|  0|  0|    3|    2|   16|
    #  | 15.8|   8| 351.0| 264| 4.22| 3.170| 14.50|  0|  1|    5|    4|   16|
    #  | 15.0|   8| 301.0| 335| 3.54| 3.570| 14.60|  0|  1|    5|    8|   16|

# Example 3 - set up some aliases for convenience

``` sh
alias mutate="dplyr mutate"
alias filter="dplyr filter"
alias select="dplyr select"
alias summarise="dplyr summarise"
alias group_by="dplyr group_by"
alias ungroup="dplyr ungroup"
alias count="dplyr count"
alias arrange="dplyr arrange"
alias kable="dplyr kable"


cat mtcars.csv | group_by cyl | summarise "mpg = mean(mpg)" | kable
```

    #  /tmp//dplyr-cli-9c1515db501ef.rds
    #  /tmp//dplyr-cli-9c15260f0e716.rds
    #  |      mpg|
    #  |--------:|
    #  | 20.09062|

# Example 4 - joins

Limitations:

  - first argument after a join command must be an existing file
  - You can’t yet specify a `by` argument for a join, so there must be a
    column in common to join by

<!-- end list -->

``` sh
cat cyl.csv
```

    #  cyl,description
    #  4,four
    #  6,six

``` sh
cat mtcars.csv | dplyr inner_join cyl.csv | dplyr kable
```

    #  /tmp//dplyr-cli-9c176730650dd.rds
    #  | cyl|  mpg|  disp|  hp| drat|    wt|  qsec| vs| am| gear| carb|description |
    #  |---:|----:|-----:|---:|----:|-----:|-----:|--:|--:|----:|----:|:-----------|
    #  |   4| 22.8| 108.0|  93| 3.85| 2.320| 18.61|  1|  1|    4|    1|four        |
    #  |   4| 24.4| 146.7|  62| 3.69| 3.190| 20.00|  1|  0|    4|    2|four        |
    #  |   4| 22.8| 140.8|  95| 3.92| 3.150| 22.90|  1|  0|    4|    2|four        |
    #  |   4| 32.4|  78.7|  66| 4.08| 2.200| 19.47|  1|  1|    4|    1|four        |
    #  |   4| 30.4|  75.7|  52| 4.93| 1.615| 18.52|  1|  1|    4|    2|four        |
    #  |   4| 33.9|  71.1|  65| 4.22| 1.835| 19.90|  1|  1|    4|    1|four        |
    #  |   4| 21.5| 120.1|  97| 3.70| 2.465| 20.01|  1|  0|    3|    1|four        |
    #  |   4| 27.3|  79.0|  66| 4.08| 1.935| 18.90|  1|  1|    4|    1|four        |
    #  |   4| 26.0| 120.3|  91| 4.43| 2.140| 16.70|  0|  1|    5|    2|four        |
    #  |   4| 30.4|  95.1| 113| 3.77| 1.513| 16.90|  1|  1|    5|    2|four        |
    #  |   4| 21.4| 121.0| 109| 4.11| 2.780| 18.60|  1|  1|    4|    2|four        |
    #  |   6| 21.0| 160.0| 110| 3.90| 2.620| 16.46|  0|  1|    4|    4|six         |
    #  |   6| 21.0| 160.0| 110| 3.90| 2.875| 17.02|  0|  1|    4|    4|six         |
    #  |   6| 21.4| 258.0| 110| 3.08| 3.215| 19.44|  1|  0|    3|    1|six         |
    #  |   6| 18.1| 225.0| 105| 2.76| 3.460| 20.22|  1|  0|    3|    1|six         |
    #  |   6| 19.2| 167.6| 123| 3.92| 3.440| 18.30|  1|  0|    4|    4|six         |
    #  |   6| 17.8| 167.6| 123| 3.92| 3.440| 18.90|  1|  0|    4|    4|six         |
    #  |   6| 19.7| 145.0| 175| 3.62| 2.770| 15.50|  0|  1|    5|    6|six         |

## Security warning

`dplyr-cli` uses `eval(parse(text = ...))` on user input. Do not expose
this program to the internet or random users under any circumstances.

## Inspirations

  - [xsv](https://github.com/BurntSushi/xsv) - a fast CSV command line
    toolkit written in Rust
  - [jq](https://stedolan.github.io/jq/) - a command line JSON
    processor.
  - [miller](http://johnkerl.org/miller/doc/)
