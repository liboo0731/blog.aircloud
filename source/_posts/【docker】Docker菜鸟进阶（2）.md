---
layout: postst
title: 【docker】Docker菜鸟进阶（2）
date: 2019-05-23 21:13:34
tags:
- docker
---

### Dockerfile参考文档

[Dockerfile介绍](http://www.dockerinfo.net/dockerfile%e4%bb%8b%e7%bb%8d)

[Dockerfile instructions](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#dockerfile-instructions)



## Dockerfile instructions

These recommendations are designed to help you create an efficient and maintainable `Dockerfile`.

### FROM

[Dockerfile reference for the FROM instruction](https://docs.docker.com/engine/reference/builder/#from)

Whenever possible, use current official images as the basis for your images. We recommend the [Alpine image](https://hub.docker.com/_/alpine/) as it is tightly controlled and small in size (currently under 5 MB), while still being a full Linux distribution.

### LABEL

[Understanding object labels](https://docs.docker.com/config/labels-custom-metadata/)

You can add labels to your image to help organize images by project, record licensing information, to aid in automation, or for other reasons. For each label, add a line beginning with `LABEL` and with one or more key-value pairs. The following examples show the different acceptable formats. Explanatory comments are included inline.

> Strings with spaces must be quoted **or** the spaces must be escaped. Inner quote characters (`"`), must also be escaped.

```Dockerfile
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor1="ACME Incorporated"
LABEL vendor2=ZENITH\ Incorporated
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```

An image can have more than one label. Prior to Docker 1.10, it was recommended to combine all labels into a single `LABEL`instruction, to prevent extra layers from being created. This is no longer necessary, but combining labels is still supported.

```Dockerfile
# Set multiple labels on one line
LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"
```

The above can also be written as:

```Dockerfile
# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

See [Understanding object labels](https://docs.docker.com/config/labels-custom-metadata/) for guidelines about acceptable label keys and values. For information about querying labels, refer to the items related to filtering in [Managing labels on objects](https://docs.docker.com/config/labels-custom-metadata/#managing-labels-on-objects). See also [LABEL](https://docs.docker.com/engine/reference/builder/#label) in the Dockerfile reference.

### RUN

[Dockerfile reference for the RUN instruction](https://docs.docker.com/engine/reference/builder/#run)

Split long or complex `RUN` statements on multiple lines separated with backslashes to make your `Dockerfile` more readable, understandable, and maintainable.

#### APT-GET

Probably the most common use-case for `RUN` is an application of `apt-get`. Because it installs packages, the `RUN apt-get`command has several gotchas to look out for.

Avoid `RUN apt-get upgrade` and `dist-upgrade`, as many of the “essential” packages from the parent images cannot upgrade inside an [unprivileged container](https://docs.docker.com/engine/reference/run/#security-configuration). If a package contained in the parent image is out-of-date, contact its maintainers. If you know there is a particular package, `foo`, that needs to be updated, use `apt-get install -y foo` to update automatically.

Always combine `RUN apt-get update` with `apt-get install` in the same `RUN` statement. For example:

```Dockerfile
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo
```

Using `apt-get update` alone in a `RUN` statement causes caching issues and subsequent `apt-get install` instructions fail. For example, say you have a Dockerfile:

```Dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl
```

After building the image, all layers are in the Docker cache. Suppose you later modify `apt-get install` by adding extra package:

```Dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

Docker sees the initial and modified instructions as identical and reuses the cache from previous steps. As a result the `apt-get update` is *not* executed because the build uses the cached version. Because the `apt-get update` is not run, your build can potentially get an outdated version of the `curl` and `nginx` packages.

Using `RUN apt-get update && apt-get install -y` ensures your Dockerfile installs the latest package versions with no further coding or manual intervention. This technique is known as “cache busting”. You can also achieve cache-busting by specifying a package version. This is known as version pinning, for example:

```Dockerfile
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo=1.3.*
```

Version pinning forces the build to retrieve a particular version regardless of what’s in the cache. This technique can also reduce failures due to unanticipated changes in required packages.

Below is a well-formed `RUN` instruction that demonstrates all the `apt-get` recommendations.

```Dockerfile
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```

The `s3cmd` argument specifies a version `1.1.*`. If the image previously used an older version, specifying the new one causes a cache bust of `apt-get update` and ensures the installation of the new version. Listing packages on each line can also prevent mistakes in package duplication.

In addition, when you clean up the apt cache by removing `/var/lib/apt/lists` it reduces the image size, since the apt cache is not stored in a layer. Since the `RUN` statement starts with `apt-get update`, the package cache is always refreshed prior to `apt-get install`.

> Official Debian and Ubuntu images [automatically run `apt-get clean`](https://github.com/moby/moby/blob/03e2923e42446dbb830c654d0eec323a0b4ef02a/contrib/mkimage/debootstrap#L82-L105), so explicit invocation is not required.

#### USING PIPES

Some `RUN` commands depend on the ability to pipe the output of one command into another, using the pipe character (`|`), as in the following example:

```Dockerfile
RUN wget -O - https://some.site | wc -l > /number
```

Docker executes these commands using the `/bin/sh -c` interpreter, which only evaluates the exit code of the last operation in the pipe to determine success. In the example above this build step succeeds and produces a new image so long as the `wc -l`command succeeds, even if the `wget` command fails.

If you want the command to fail due to an error at any stage in the pipe, prepend `set -o pipefail &&` to ensure that an unexpected error prevents the build from inadvertently succeeding. For example:

```Dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

> Not all shells support the `-o pipefail` option.
>
> In cases such as the `dash` shell on Debian-based images, consider using the *exec* form of `RUN` to explicitly choose a shell that does support the `pipefail` option. For example:
>
> ```Dockerfile
> RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
> ```

### CMD

[Dockerfile reference for the CMD instruction](https://docs.docker.com/engine/reference/builder/#cmd)

The `CMD` instruction should be used to run the software contained by your image, along with any arguments. `CMD` should almost always be used in the form of `CMD ["executable", "param1", "param2"…]`. Thus, if the image is for a service, such as Apache and Rails, you would run something like `CMD ["apache2","-DFOREGROUND"]`. Indeed, this form of the instruction is recommended for any service-based image.

In most other cases, `CMD` should be given an interactive shell, such as bash, python and perl. For example, `CMD ["perl", "-de0"]`, `CMD ["python"]`, or `CMD ["php", "-a"]`. Using this form means that when you execute something like `docker run -it python`, you’ll get dropped into a usable shell, ready to go. `CMD` should rarely be used in the manner of `CMD ["param", "param"]` in conjunction with [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint), unless you and your expected users are already quite familiar with how `ENTRYPOINT` works.

### EXPOSE

[Dockerfile reference for the EXPOSE instruction](https://docs.docker.com/engine/reference/builder/#expose)

The `EXPOSE` instruction indicates the ports on which a container listens for connections. Consequently, you should use the common, traditional port for your application. For example, an image containing the Apache web server would use `EXPOSE 80`, while an image containing MongoDB would use `EXPOSE 27017` and so on.

For external access, your users can execute `docker run` with a flag indicating how to map the specified port to the port of their choice. For container linking, Docker provides environment variables for the path from the recipient container back to the source (ie, `MYSQL_PORT_3306_TCP`).

### ENV

[Dockerfile reference for the ENV instruction](https://docs.docker.com/engine/reference/builder/#env)

To make new software easier to run, you can use `ENV` to update the `PATH` environment variable for the software your container installs. For example, `ENV PATH /usr/local/nginx/bin:$PATH` ensures that `CMD ["nginx"]` just works.

The `ENV` instruction is also useful for providing required environment variables specific to services you wish to containerize, such as Postgres’s `PGDATA`.

Lastly, `ENV` can also be used to set commonly used version numbers so that version bumps are easier to maintain, as seen in the following example:

```Dockerfile
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

Similar to having constant variables in a program (as opposed to hard-coding values), this approach lets you change a single `ENV`instruction to auto-magically bump the version of the software in your container.

Each `ENV` line creates a new intermediate layer, just like `RUN` commands. This means that even if you unset the environment variable in a future layer, it still persists in this layer and its value can be dumped. You can test this by creating a Dockerfile like the following, and then building it.

```Dockerfile
FROM alpine
ENV ADMIN_USER="mark"
RUN echo $ADMIN_USER > ./mark
RUN unset ADMIN_USER
$ docker run --rm test sh -c 'echo $ADMIN_USER'

mark
```

To prevent this, and really unset the environment variable, use a `RUN` command with shell commands, to set, use, and unset the variable all in a single layer. You can separate your commands with `;` or `&&`. If you use the second method, and one of the commands fails, the `docker build` also fails. This is usually a good idea. Using `\` as a line continuation character for Linux Dockerfiles improves readability. You could also put all of the commands into a shell script and have the `RUN` command just run that shell script.

```Dockerfile
FROM alpine
RUN export ADMIN_USER="mark" \
    && echo $ADMIN_USER > ./mark \
    && unset ADMIN_USER
CMD sh
$ docker run --rm test sh -c 'echo $ADMIN_USER'
```

### ADD or COPY

- [Dockerfile reference for the ADD instruction](https://docs.docker.com/engine/reference/builder/#add)
- [Dockerfile reference for the COPY instruction](https://docs.docker.com/engine/reference/builder/#copy)

Although `ADD` and `COPY` are functionally similar, generally speaking, `COPY` is preferred. That’s because it’s more transparent than `ADD`. `COPY` only supports the basic copying of local files into the container, while `ADD` has some features (like local-only tar extraction and remote URL support) that are not immediately obvious. Consequently, the best use for `ADD` is local tar file auto-extraction into the image, as in `ADD rootfs.tar.xz /`.

If you have multiple `Dockerfile` steps that use different files from your context, `COPY` them individually, rather than all at once. This ensures that each step’s build cache is only invalidated (forcing the step to be re-run) if the specifically required files change.

For example:

```Dockerfile
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

Results in fewer cache invalidations for the `RUN` step, than if you put the `COPY . /tmp/` before it.

Because image size matters, using `ADD` to fetch packages from remote URLs is strongly discouraged; you should use `curl` or `wget` instead. That way you can delete the files you no longer need after they’ve been extracted and you don’t have to add another layer in your image. For example, you should avoid doing things like:

```Dockerfile
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

And instead, do something like:

```Dockerfile
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

For other items (files, directories) that do not require `ADD`’s tar auto-extraction capability, you should always use `COPY`.

### ENTRYPOINT

[Dockerfile reference for the ENTRYPOINT instruction](https://docs.docker.com/engine/reference/builder/#entrypoint)

The best use for `ENTRYPOINT` is to set the image’s main command, allowing that image to be run as though it was that command (and then use `CMD` as the default flags).

Let’s start with an example of an image for the command line tool `s3cmd`:

```Dockerfile
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

Now the image can be run like this to show the command’s help:

```
$ docker run s3cmd
```

Or using the right parameters to execute a command:

```
$ docker run s3cmd ls s3://mybucket
```

This is useful because the image name can double as a reference to the binary as shown in the command above.

The `ENTRYPOINT` instruction can also be used in combination with a helper script, allowing it to function in a similar way to the command above, even when starting the tool may require more than one step.

For example, the [Postgres Official Image](https://hub.docker.com/_/postgres/) uses the following script as its `ENTRYPOINT`:

```
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

> Configure app as PID 1
>
> This script uses [the `exec` Bash command](http://wiki.bash-hackers.org/commands/builtin/exec) so that the final running application becomes the container’s PID 1. This allows the application to receive any Unix signals sent to the container. For more, see the [`ENTRYPOINT` reference](https://docs.docker.com/engine/reference/builder/#entrypoint).

The helper script is copied into the container and run via `ENTRYPOINT` on container start:

```Dockerfile
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```

This script allows the user to interact with Postgres in several ways.

It can simply start Postgres:

```
$ docker run postgres
```

Or, it can be used to run Postgres and pass parameters to the server:

```
$ docker run postgres postgres --help
```

Lastly, it could also be used to start a totally different tool, such as Bash:

```
$ docker run --rm -it postgres bash
```

### VOLUME

[Dockerfile reference for the VOLUME instruction](https://docs.docker.com/engine/reference/builder/#volume)

The `VOLUME` instruction should be used to expose any database storage area, configuration storage, or files/folders created by your docker container. You are strongly encouraged to use `VOLUME` for any mutable and/or user-serviceable parts of your image.

### USER

[Dockerfile reference for the USER instruction](https://docs.docker.com/engine/reference/builder/#user)

If a service can run without privileges, use `USER` to change to a non-root user. Start by creating the user and group in the `Dockerfile` with something like `RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres`.

> Consider an explicit UID/GID
>
> Users and groups in an image are assigned a non-deterministic UID/GID in that the “next” UID/GID is assigned regardless of image rebuilds. So, if it’s critical, you should assign an explicit UID/GID.

> Due to an [unresolved bug](https://github.com/golang/go/issues/13548) in the Go archive/tar package’s handling of sparse files, attempting to create a user with a significantly large UID inside a Docker container can lead to disk exhaustion because `/var/log/faillog` in the container layer is filled with NULL (\0) characters. A workaround is to pass the `--no-log-init` flag to useradd. The Debian/Ubuntu `adduser` wrapper does not support this flag.

Avoid installing or using `sudo` as it has unpredictable TTY and signal-forwarding behavior that can cause problems. If you absolutely need functionality similar to `sudo`, such as initializing the daemon as `root` but running it as non-`root`), consider using [“gosu”](https://github.com/tianon/gosu).

Lastly, to reduce layers and complexity, avoid switching `USER` back and forth frequently.

### WORKDIR

[Dockerfile reference for the WORKDIR instruction](https://docs.docker.com/engine/reference/builder/#workdir)

For clarity and reliability, you should always use absolute paths for your `WORKDIR`. Also, you should use `WORKDIR` instead of proliferating instructions like `RUN cd … && do-something`, which are hard to read, troubleshoot, and maintain.

### ONBUILD

[Dockerfile reference for the ONBUILD instruction](https://docs.docker.com/engine/reference/builder/#onbuild)

An `ONBUILD` command executes after the current `Dockerfile` build completes. `ONBUILD` executes in any child image derived `FROM` the current image. Think of the `ONBUILD` command as an instruction the parent `Dockerfile` gives to the child `Dockerfile`.

A Docker build executes `ONBUILD` commands before any command in a child `Dockerfile`.

`ONBUILD` is useful for images that are going to be built `FROM` a given image. For example, you would use `ONBUILD` for a language stack image that builds arbitrary user software written in that language within the `Dockerfile`, as you can see in [Ruby’s `ONBUILD`variants](https://github.com/docker-library/ruby/blob/master/2.4/jessie/onbuild/Dockerfile).

Images built from `ONBUILD` should get a separate tag, for example: `ruby:1.9-onbuild` or `ruby:2.0-onbuild`.

Be careful when putting `ADD` or `COPY` in `ONBUILD`. The “onbuild” image fails catastrophically if the new build’s context is missing the resource being added. Adding a separate tag, as recommended above, helps mitigate this by allowing the `Dockerfile` author to make a choice.



## Additional resources:

- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [More about Base Images](https://docs.docker.com/develop/develop-images/baseimages/)
- [More about Automated Builds](https://docs.docker.com/docker-hub/builds/)
- [Guidelines for Creating Official Images](https://docs.docker.com/docker-hub/official_images/)