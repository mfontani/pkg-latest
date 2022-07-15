# pkg-latest

Update latest version of debian and alpine packages in Dockerfiles.

# Why?

Pinning package versions in dockerfiles is considered best practice, supported
by hadolint:

```bash
$ command cat Dockerfile
FROM debian@sha256:e2fe52e17d649812bddcac07faf16f33542129a59b2c1c59b39a436754b7f146
RUN apt-get install -y -q curl --no-install-recommends
$ hadolint Dockerfile
Dockerfile:2 DL3008 warning: Pin versions in apt get install. Instead of `apt-get install <package>` use `apt-get install <package>=<version>`
```

# How?

This tool helps me keep the "version" updated, i.e. via:

```bash
pkg-latest < Dockerfile | sponge Dockerfile
```

... assuming your `Dockerfile` contains some "hints" which tell `pkg-latest`
which distro type, branch, and repository it should look for version updates
in, i.e. _instead_ of writing:

```Dockerfile
RUN apt-get install -y -q \
    curl=1234 \
    --no-install-recommends
```

... you can write:

```Dockerfile
RUN apt-get install -y -q \
    curl=1234 $(: pkg-latest debian buster/main amd64 ) \
    --no-install-recommends
```

... and `pkg-latest` will "work" on that line, changing the `=1234` version
part to the "right" (latest) version acccording to the definition (in the above
case, for alpine 3.13, from the main repository for the x86_64 architecture),
i.e. at the time of writing:

```bash
$ command cat Dockerfile
FROM debian@sha256:e2fe52e17d649812bddcac07faf16f33542129a59b2c1c59b39a436754b7f146
RUN apt-get install -y -q \
    curl=1235 $(: pkg-latest debian buster/main amd64 ) \
    --no-install-recommends
$ ./pkg-latest < Dockerfile
FROM debian@sha256:e2fe52e17d649812bddcac07faf16f33542129a59b2c1c59b39a436754b7f146
RUN apt-get install -y -q \
    curl=7.64.0-4+deb10u2 $(: pkg-latest debian buster/main amd64 ) \
    --no-install-recommends
```

You likely want to use `sponge` to make updates simpler, i.e.:

```bash
pkg-latest < Dockerfile | sponge Dockerfile
```

On Debian, you can get it from the `moreutils` package.

# LICENSE

Copyright 2022 Marco Fontani <MFONTANI@cpan.org>

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice,
   this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors
   may be used to endorse or promote products derived from this software
   without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.
