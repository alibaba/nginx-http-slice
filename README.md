# Nginx HTTP Slice module

## Introduction 

This is a module that is distributed with
[tengine](http://tengine.taobao.org) which is a distribution of
[Nginx](http://nginx.org) that is used by the e-commerce/auction site
[Taobao.com](http://en.wikipedia.org/wiki/Taobao). This distribution
contains some modules that are new on the Nginx scene. The
`ngx_http_slice_module` module is one of them.

This module can be thought out as a *reverse byte-range* request
header. It's main utility is to allow Nginx to slice a big file in
small pieces (byte-ranges) while permitting to use on-the-fly gzip
compression.

A typical example is for allowing someone to download a large video
file while keeping the bandwith usage minimal. This might also be used
as device for selling a video file by pieces where each link points to
different zones of the file splitted by file ranges.

Other use would be to use a generic CSS file and use only part of it
for each section of a site. Granted that byte-range slicing isn't the
most intuitive for such.

Note also that using arguments is more **useful** than byte-ranges in
the sense that they can be set in a normal link, while byte ranges
require a special [HTTP header](https://en.wikipedia.org/wiki/Byte_serving).

## Configuration example

    location ^~ /video-dump/ {
        slice; # enable slicing
        slice_start_arg s;
        slice_end_arg e;    
    }

So we would request the first 1k of the file like this:

    http://example.com/video-dump/large_vid.mp4?s=0&e=1024

Notice `s=0`, start at `0` and `e=1024`, stop at `1024` bytes (1k). 

## Module directives

**slice**

**context:** `location`

It enables the content slicing in a given location.

<br/>
<br/>

**slice_arg_begin** `string`

**default:** `begin`

**context:** `http, server, location`

Defines the argument that defines the request range of bytes **start**.

<br/>
<br/>

**slice_arg_end** `string`

**default:** `end`

**context:** `http, server, location`

Defines the argument that defines the request range of bytes **end**.

<br/>
<br/>

**slice_header** `string`

**context:** `http, server, location`

Defines the string to be used as the **header** of each slice being
served by Nginx.

<br/>
<br/>

**slice_footer** `string`

**context:** `http, server, location`

Defines the string to be used as the **footer** of each slice being
served by Nginx.

<br/>
<br/>

**slice_header_first** `on` | `off`

**default:** `on`

**context:** `http, server, location`

If set to `off` and when requesting the **first** byte of the file do **not
serve** the header.

This directive is particularly useful to differentiate the **first**
slice from the remaining slices. The first slice is the one which has
**no** header.

<br/>
<br/>

**slice_footer_last** `on` |  `off `

**default:** `on`

**context:** `http, server, location`

If set to `off` and when requesting the **last** byte of the file do **not
serve** the header.

This directive is particularly useful to differentiate the **last**
slice from the remaining slices. The last slice is the one which has
**no** footer.

## Assorted examples

Here's some examples that explore all the options.

### Serve a huge DB file while sending headers except on the first slice

    location ^~ /dbdumps/ {
        slice; # enable slicing
        slice_start_arg first;
        slice_end_arg last;
        slice_header '-- **db-slice-start**';
        slice_header_first off;
    }

Then a request like this:

    http://example.com/dbdumps/somedb.sql?first=0&last=1048576
    
Send the first 1M and skip the `-- **db-slice-start**` header.


### Serve a huge DB file while sending headers except on the first slice

    location ^~ /dbdumps/ {
        slice; # enable slicing
        slice_start_arg first;
        slice_end_arg last;
        slice_header '-- **db-slice-start**';
        slice_header_first off;
        slice_footer '-- **db-slice-end**';
    }
    
This differs from the previous in the sense that it sends a footer.

### Serve a huge DB file while sending headers except on the first slice and send footer except on the last slice

    location ^~ /dbdumps/ {
        slice; # enable slicing
        slice_start_arg first;
        slice_end_arg last;
        slice_header '-- **db-slice-start**';
        slice_header_first off;
        slice_footer '-- **db-slice-end**';
        slice_footer_last off; 
    }

Then a request like this:

    http://example.com/dbdumps/somedb.sql?first=0&last=1048576

Send the first 1M and skip the `-- **db-slice-start**` header.

If the file is 200MB, we get the last slice with:

    http://example.com/dbdumps/somedb.sql?first=208666624&last=209715200

this last slice has no footer.

## Installation

 1. Clone the git repo.
     
        git clone git://github.com/alibaba/nginx-http-slice.git

 2. Add the module to the build configuration by adding
    `--add-module=/path/to/nginx-http-slice`.

 3. Build the nginx binary.
 
 4. Install the nginx binary.
 
 5. Configure contexts where concat is enabled.
 
 6. Build your links such that the above format, i.e., all URIs that
    correspond to specific ranges. As example here's how to link to
    the first 4k of a file.
    
        <a href="http://example.com/datadumps/dump0.sql?start=0&end=4096" />db dump</a>
              
 7. Done.   

## Tagging releases 

I'm tagging each release in synch with the
[Tengine](http://tengine.taobao.org) releases.
 
## Other tengine modules on Github

 + [http concat](https://github.com/taobao/nginx-http-concat):
   allows to concatenate a given set of files and ship a single
   response from the server. It's particularly useful for **aggregating**
   CSS and Javascript files.

 + [footer filter](https://github.com/taobao/nginx-http-footer-filter):
   allows to add some extra data (markup or not) at the end of a
   request body. It's pratical for things like adding time stamps or
   other miscellaneous stuff without having to tweak your application.

## Other builds

 1. As referred at the outset this module is part of the
    [`tengine`](http://tengine.taobao.org) Nginx distribution. So you
    might want to save yourself some work and just build it from
    scratch using `tengine` in lieu if the official Nginx source.

 2. If you fancy a bleeding edge Nginx package (from the dev releases)
    for Debian made to measure then you might be interested in my
    [debian](http://debian.taobao.net/unstable) Nginx
    package. Instructions for using the repository and making the
    package live happily inside a stable distribution installation are
    [provided](http://debian.taobao.net).
        
## Acknowledgments

Thanks to [Joshua Zhu](http://blog.zhuzhaoyuan.com) and the Taobao
platform engineering team for releasing `tengine`. Also for being kind
enough to clarify things regarding this module on the
[Tengine mailing list](http://code.taobao.org/mailman/listinfo/tengine).

## License

Copyright (C) 2010-2012 Alibaba Group Holding Limited

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:
 
 1. Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.
    
 2. Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY AUTHOR AND CONTRIBUTORS "AS IS" AND ANY
EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL AUTHOR OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR
BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN
IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
