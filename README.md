# CRAN-checks
Notes about fixes for CRAN I've had to do when submitting a package

* In `DESCRIPTION`, the `Description` text should put all software names in single quotes.
    * A package submission was rejected as I said `C library` when I should have had `'C' library`
* Names in `LICENSE` and `Authors` field in `DESCRIPTION` should match

## Run `R CMD CHECK --as-cran`

Run `R CMD CHECK --as-cran` on final source package about to be submitted to CRAN

## Have tests run via github actions on multiple machine types

`usethis::install_github_action("check-standard")` 

This will run `R CMD check` under 5 different machines using github actions.

## Run on "win-builder" site.

Run on multiple versions of windows (including the latest R-devel version) using the win-builder site: https://win-builder.r-project.org/

## Run on "mac-builder" site

Run on multiple versions of MacOS using mac-builder: https://mac.r-project.org/macbuilder/submit.html

## Run `valgrind` and `clang-ASAN` builds using `{rhub2}`

Memory checking using clang-ASAN and valgrind can be very difficult to set up.

Run these checks as a github action using the [`{rhub2}`](https://github.com/r-hub/rhub2) package

## Recompile to check implicit type conversions in C code

Add `PKG_CFLAGS+=-Wconversion` to `src/Makevars` and rebuild package.

This will check for implicit type conversion in C code.  This is part of the extra testing run by CRAN after the package has been accepted.

## Recompile to check for bad pointer artithmetic

* Add to `Makevars` `PKG_CFLAGS+=-fsanitize=pointer-overflow -fsanitize-trap=pointer-overflow`
* Run R in the debugger
    * `R -d lldb`
    * enter "run" to start R
    * `testthat::test_local()` to run package tests
