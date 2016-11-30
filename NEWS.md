## rcloud.shiny 0.4

### New Architecture

`rcloud.shiny` 0.4 is a bottom-up rewrite of the Shiny implementation for RCloud. Apart from
requiring some modifications to RStudio Shiny, the new implementation is much simpler: it clocks in
at just over 100 lines of code.

Prior to 0.4, `rcloud.shiny` did not have a mechanism for HTTP requests to travel from the client to
the Shiny server, so all communication had to travel over the web socket. (`rcloud.shiny` tunnels
Shiny's web socket through the RCloud session's web socket.)

Without independent HTTP requests, it was necessary to rewrite URLs in order to load Shiny
resources. That didn't always work correctly, especially with add-on packages and HTML
Widgets. Also file download, data transfers, serving of temporary files, and timer messages all
require separate HTTP requests, so those features were not working at all.

RCloud 1.4 introduced `proxy.R`, which allows tunneling of HTTP requests through RCloud to ports on
the RCloud compute node where the session is running. Rserve 1.8-5 introduces an idle callback,
which allows calling the Shiny HTTP message handler (`shiny:::serviceApp`) from inside Rserve's
message handler.

With these pieces in place, were finally able to deliver an environment where Shiny can do
everything it needs to do. We put Shiny in an `<iframe>` which fills the window, so that we can
create the R session and start the Shiny server before loading the Shiny page into the iframe
through `proxy.R`. Since all Shiny HTTP requests use relative URLs, the Shiny app has complete
control over the iframe.

### License Status

Because `rcloud.shiny` 0.4 overrides a few functions in RStudio Shiny and those functions are
currently copied into this package, this package is currently licensed under GPL-3.

Those functions are visible in `R/shiny-overrides.R`. The rest of the `rcloud.shiny` code is still
MIT-licenced.

We recognize that this is an ugly and brittle design, but we needed to test it thoroughly to make
sure we're on the right track, before making feature requests and pull requests on Shiny. We hope to
rectify this soon, by contributing a few features to Shiny that make it possible to run in the
RCloud environment.