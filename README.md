A script to check for differences in a website. Example uses include in stock alerts, follower count change, or [checking Steam every 10 minutes for half-life 3](https://store.steampowered.com/search/?term=half+life+3).

### Usage

One-liner, no download required:
```
wget -qO- https://raw.githubusercontent.com/cjcarrick/site-diff/master/site-diff | bash -s -- -l <URL> [options]
```

Or [download the script](https://raw.githubusercontent.com/cjcarrick/site-diff/master/site-diff) and run it manually:
```
site-diff -l <link> [options]
```

### Options
```
-l, --link <link>            Required. URL to retrieve the page from.
-d, --dir <directory>        Specify a specific directory to use instead of the
                             environment variable $TMPDIR.
-s, --selector <selector>    Requires pup. Match HTML according to CSS selectors.
                             Pretty-print HTML using an empty selector.
-i, --interval <number>      Override the default interval.

-p, --pupargs <options>      Supply additional options for pup.
-c, --curlargs <options>     Supply your own options for cURL, overriding the 
                             default (-s in quiet mode, -v in verbose mode).
-f, --diffargs <options>     Override the default options (--ignore-all-space
                             --minimal --ignore-case --side-by-side --text
                             --suppress-common-lines) for diff.

-h, --help                   Print this help message and exit.
-q, --quiet                  Does not print anything to STDOUT.
-v, --verbose                Print debugging information for cURL and more info
                             about what the script is doing.
-w, --write                  Write diffs to STDOUT. If used with --quiet, diffs 
                             will still be written to STDOUT.
-o, --open                   Attempt to open the URL in your default browser
                             when a change is found.
-e, --exit                   Exit when a diff is found. If used with --write
                             and --open, the browser will open and diffs will
                             be printed, then the script exit.
```

### If you intend to use selectors

You need to have [`pup`](https://github.com/ericchiang/pup/). Pass in selectors like this:

```
# Check for a change in the number of like at https://www.youtube.com/watch?v=bXEddCLW3SM
site-diff -l https://www.youtube.com/watch?v=bXEddCLW3SM -s "style-scope.ytd-menu-renderer.force-icon-button.style-text"
```

See pup's docs for info on supported selectors.

### Formatting

Especially when printing diffs, it can be useful to have pretty-printed html. Site-diff can do this using [`pup`](https://github.com/ericchiang/pup/). If you want to format the HTML but load the entire page, pass `--selector ""`. 

### Limitations and Some Things to Note

1. Some sites, like amazon.com, can tell when you're using cURL:

    ```
    ...
    <!--
            To discuss automated access to Amazon data please contact api-services-support@amazon.com.
            For information about migrating to our APIs refer to our Marketplace APIs at https://developer.amazonservices.com/ref=rm_5_sv, or our Product Advertising API at https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html/ref=rm_5_ac for advertising use cases.
    -->
    ...
    ```

    and won't give you the page you were expecting. There's probably a way to get around this, but I don't know how.

2. Some sites, like randomwordgenerator.com, run some JS on the DOM just after the page loads. cURL doesn't take this into account. You may be able to get it to work using PhantomJS, but that would probably require nodeJS and that sounds like a lot of work.

3. Some sites, like google.com, have randomly (?) generated elements. For example, this is the result of running `diff` on pages retrieved just 10 seconds apart:

    ```
    $ diff --ignore-all-space --side-by-side --text --suppress-common-lines ./tmp/site-diff-httpswwwgooglecom/site-diff.temp ./tmp/site-diff-httpswwwgooglecom/site-diff.temp.old

            <script nonce="PMhdLGzz0CMN6izTKYfwdg==">             |         <script nonce="4rZa+9ALImmnlT7REBghJw==">
                        kEI: 'i1NsYYq-CpTW-wS5nJ_ADA',            |                     kEI: 'gFNsYcn3NYv0-gS-u73ADA',
                        kEXPI: '0,1302536,56873,6059,206,4804,231 |                     kEXPI: '0,18168,1284368,56873,1710,4348,2
            <script nonce="PMhdLGzz0CMN6izTKYfwdg==">             |         <script nonce="4rZa+9ALImmnlT7REBghJw==">
            <script nonce="PMhdLGzz0CMN6izTKYfwdg==">             |         <script nonce="4rZa+9ALImmnlT7REBghJw==">
                                        <script nonce="PMhdLGzz0C |                                     <script nonce="4rZa+9ALIm
                                            value="ALs-wAMAAAAAYW |                                         value="ALs-wAMAAAAAYW
                    <script nonce="PMhdLGzz0CMN6izTKYfwdg==">     |                 <script nonce="4rZa+9ALImmnlT7REBghJw==">
            <script nonce="PMhdLGzz0CMN6izTKYfwdg==">             |         <script nonce="4rZa+9ALImmnlT7REBghJw==">
            <script nonce="PMhdLGzz0CMN6izTKYfwdg==">             |         <script nonce="4rZa+9ALImmnlT7REBghJw==">
            <script nonce="PMhdLGzz0CMN6izTKYfwdg==">             |         <script nonce="4rZa+9ALImmnlT7REBghJw==">
                        '{\x22d\x22:{},\x22sb_he\x22:{\x22agen\x2 |                     '{\x22d\x22:{},\x22sb_he\x22:{\x22agen\x2
    ```

    You can see that a new nonce is being set each time, for this reason, it would be best to use a selector on google.com.
