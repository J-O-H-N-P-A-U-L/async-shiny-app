# Async process blocks shiny app within "user session" #23

In shiny apps, one R process can be shared by multiple users. If one user submits a long running task, then all the other users sharing the same underlying R process are blocked. The goal of promises is to avoid this. So promises will prevent blocking between "user sessions" within one R process but not within a single "user session".

The author of the package mentioned that this behavior is by design. He goes into some detail about how this works in this section of the docs: https://rstudio.github.io/promises/articles/shiny.html#the-flush-cycle

The goal, at least for this release of Shiny, is not to allow this kind of intra-session responsiveness, but rather, inter-session; i.e., running an async operation won't make its owning session more responsive, but rather will allow other sessions to be more responsive.

If you really must have this kind of behavior, there is this alternativeway to work around it. this strategy for "Submit and forget" is to invoke a R script in a separate process with a system call, example:



system("Rscript /path/to/script.R arg1 arg2 ...", wait = FALSE)

(callr doesn't yet integrate with promises automatically but I suspect we'll do that sooner rather than later--it should be very straightforward.)

This also does exactly what I am looking for since the async process will terminate on its own when it has completed processing.

The script updates some kind of database or writes to some logs to allow to track its status.

tryCatch() can be used in the script to manage errors (and update the status via the db or logs to let us know it failed).

(Instead of calling system() directly for "submit and forget", you might use callr::r_bg(..., supervise = FALSE). I haven't used this approach myself, but it should work and I think it is likely easier to pass parameters this way (without worrying about manually escaping, serializing, etc.). And this way you at least have the option to retrieve the result from the parent process if you want to.

Refer: https://github.com/HenrikBengtsson/future.callr
)


## This use case

* The approach indeed isn’t blocking the whole app, but it seems to slow down the execution of the “fast” observer (which is not the case using the callr-approach) while the promise isn’t resolved – also among multiple local sessions (have a look at the refreshing-rate of the random number – 5 seconds fast – 5 seconds slow).
