


## Trick or Treat - R You Coding With Manual Scope Constructs?

Halloween is around the corner so it's good time to discuss two of R's scoping mechanisms.  R is rich in features, sometimes making it hard to select amoung them. My alarm bells ring loudly when I see manual scope control.  After I started writing packages, I realized I wasn't fully taking advantage of R's natural beauty to simplify code.  

As I searched for good ideas and tips for customizing `Rprofile.site` and `.Rprofile`, a common construct I found was:

```r
local({
  r <- getOption("repos")
  r["CRAN"] <- "http://cran.revolutionanalytics.com"
  options("repos" = r)
})
```
The use of `local()` prevents the enclosed code from making any changes to the `base` environment which is a good thing.  Functions also work the same way and it just so happens R provides `.First()`.  We can move the code into `.First()` and toss `local()`.  Easy peasy.

The next construct I found is

```r
.env <- new.env()
attach(.env)
 
.env$some_function <- function(x) {
    # your code here
}

.env$different_function <- function(x) {
    # your code here
}
```
This creates a *hidden* namespace, attaches it to the search path, and adds a few utilty functions.  Being hidden prevents it from being deleted when you cleanup your environment with `rm()`. Well, that's only true if the argument to `rm` doesn't include hidden objects.

Defining functions in an initialization file doesn't help portability, compatibility, or sharability.  Where do most functions come from?  Packages!  If you put all your functions into a package and host it in git repo, you have an easy way to share your code, even if it's just among your computers.  Once you `devtools::install_github("my-account/my-package")`, you don't need to attach it to your environment to use any of the functions.  Just invoke `my-package::my-function()`.  Now there's even less clutter in your environment.  Sharing could also mean deploying your Shiny app, where the `rsconnect` package will automatically manage your dependencies.

If you document your functions, the switch to a package format is almost as simple as changing comment delimiters.  RStudio will manage all other aspects of building a package.  You benefit because now you can access your documentation from R's help system.

Ah, but you still want your wonderful functions loaded automatically.  The simplest code is

```r
.First <- function() {
  library("my-package", quietly = TRUE)
}
```
If the library can't be loaded, then `library()` will stop execution with an error.  I don't like stopping the initialization process over a missing optional piece.  Let's try `library()'s` sibling

```r
.First <- function() {
  require("my-package", quietly = TRUE)
}
```
That works, but it displays a warning if the package is missing. Needless noise.

```r
.First <- function() {
  suppressWarnings(require("my-package", quietly = TRUE))
}
```
The initialization process will complete and won't display any warning or informational messages.  Plus a logical is returned for any additional processing.

```r
.First <- function() {
  if (suppressWarnings(require("my-package", quietly = TRUE)) ) {
    options(
      "my-package.option1" = TRUE,
      "my-package.option2" = "my-message"
    )
  }
}
```
If you find yourself trying to twist or bend the scoping rules, think if a function or package would accomplish the same result and provide additional benefits of better organization, sharing, documentation and easy Shiny deployment.

See [Rprofile.site](https://github.com/dgabbe/rprofile.site#rprofilesite), [dgutils](https://github.com/dgabbe/dgutils#dgutils), and [wdprompt](https://github.com/dgabbe/wdprompt#wdprompt) for my customizations.

[![Creative Commons License](https://i.creativecommons.org/l/by-nd/4.0/80x15.png)](https://creativecommons.org/licenses/by-nd/4.0/) [David Gabbé](https://github.com/dgabbe)

Generated on: 02-Nov-2016

