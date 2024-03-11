# Checking a package for submission to CRAN

CRAN has some quite detailed requirements on what is/isn't allowed in packages.   These 
requirements are detailed in the [CRAN submission checklist](https://cran.r-project.org/web/packages/submission_checklist.html)
and their [policies document](https://cran.r-project.org/web/packages/policies.html).

The reference checklist from CRAN is pretty dense, so I can recommend also reading two friendlier guides:

* [Preparing your package for a CRAN submission](https://github.com/ThinkR-open/prepare-for-cran) by [ThinkR](https://github.com/ThinkR-open)
* [Extrachecks](https://github.com/DavisVaughan/extrachecks) by [Davis Vaughan](https://github.com/DavisVaughan)

In additional to these general checklists, it's also worth having the computer run lots of automatic checks, and this document 
outlines what I have in place for automatic testing.



* First of all - actually have tests for your package.  I can recommend [`{testthat}`](https://cran.r-project.org/package=testthat) as a solid testing package.
* Locally run tests when you make changes to ensure you haven't broken things
*  Run `R CMD CHECK` often.  Fix all warnings and notes.
    * In rstudio, this is the called "Check Package"
* Run `R CMD CHECK --as-cran` for even stricter checks prior to uploading package to CRAN
* If developing on GitHub, you can use [GitHub Actions](https://docs.github.com/en/actions) to run `R CMD CHECK` on a variety of different machines.
* Upload your package to `win-builder` to test on multiple windows environemnts
* Upload your package to `mac-builder` to test on macOS
* If your package includes C code:
    * Check for implicit type conversion
    * Check for bad pointer arithmetic
    * Build/test package using clang-ASAN
    * Build/test package using valgrind 

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

* Go to github and create a Personal Access Token (PAT) (if you haven't created one already, or your prior one has expired):
    * https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
    * https://github.com/settings/tokens
* Copy your created github PAT to your clipboard
* Use the `{gitcreds}` package for managing git token.   `install.packages('gitcreds')`
* `gitcreds::gitcreds_set()`, select option to set new token, paste the PAT token.
* Install `{rhub2}` - `remotes::install_github('https://github.com/r-hub/rhub2')`
* `rhub_setup()` then commit the new rhub.yaml file
* `rhub_doctor(repo_url)` to check it's all working
* `rhub_check(repo_url)` to select which machines to run tests on 

## Recompile to check implicit type conversions in C code

Add `PKG_CFLAGS+=-Wconversion` to `src/Makevars` and rebuild package.

This will check for implicit type conversion in C code.  This is part of the extra testing run by CRAN after the package has been accepted.

## Recompile to check for bad pointer artithmetic

* Add to `Makevars` `PKG_CFLAGS+=-fsanitize=pointer-overflow -fsanitize-trap=pointer-overflow`
* Run R in the debugger
    * `R -d lldb`
    * enter "run" to start R
    * `testthat::test_local()` to run package tests

## Copyright of included code

* Set author/copyright holder as 'ctb' and 'cph' in Authors@R
* Include 'Copyright:' section in description and refer to 'inst/COPYRIGHTS' file
* Include LICENSE files for any included code in `inst` directory and refer to them in `inst/COPYRIGHTS`
* Lots of examples `COPYRIGHTS` files from current packages in CRAN are in the `copyrights/` directory.

## Miscellaneous fixes for CRAN

Notes about fixes for CRAN I've had to do when submitting a package

* In `DESCRIPTION`, the `Description` text should put all software names in single quotes.
    * A package submission was rejected as I said `C library` when I should have had `'C' library`
* The name for the copyright holder in `LICENSE` file, and the copyright holder in the `Authors` field in `DESCRIPTION` should match

# Reminders

* Have you called `normalizePath()` on all the file paths?
* Does every exported function have `@examples`?
* Correctly assigned copyright?




# Reading list

* https://cran.r-project.org/web/packages/submission_checklist.html
* https://cran.r-project.org/web/packages/policies.html
* https://github.com/ThinkR-open/prepare-for-cran
* https://r-pkgs.org/
* https://github.com/DavisVaughan/extrachecks
* https://github.com/r-hub/rhub2
* https://win-builder.r-project.org/
* https://mac.r-project.org/macbuilder/submit.html
