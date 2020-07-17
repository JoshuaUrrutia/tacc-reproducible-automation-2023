Docker CLI
===============================


Alternatively you can re-use the TAPIS container the same way we did last week:
::
 docker run --rm -it \
            -v ${PWD}:/work \
            -v ${HOME}/.agave:/root/.agave \
            -v /var/run/docker.sock:/var/run/docker.sock \
            tacc/tapis-cli:latest \
            bash

+--------------------------+--------------------------------------------------+
| Option                   | Description                                      |
+==========================+==================================================+
| --rm                     | Automatically remove the container when it exits |
+--------------------------+--------------------------------------------------+
| -i, --interactive        | Keep STDIN open even if not attached             |
+--------------------------+--------------------------------------------------+
| -t, --tty                | Allocate a pseudo-TTY                            |
+--------------------------+--------------------------------------------------+
| -v, --volume list        | Bind mount a volume                              |
+--------------------------+--------------------------------------------------+

But I wouldn't recommend this for several reasons.
First you will be limited to writing only to subdirectories under your current directory $PWD,
because of the local:container volume mount specified in ``-v ${PWD}:/work``.

Moreover, because part of the deploy process is to build a docker container, we would
be building a container inside of a container. Which could be useful in some cases, but
is somewhat contrived.
`Docker in Docker <https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/>`_
To avoid this we can just mount our system docker into our container, so we're
using the same docker engine to build the container inside our container.
Notice there is an extra volume mount ``/var/run/docker.sock:/var/run/docker.sock``.
So our system docker engine is mounted inside the container.

:ref:`Back to Apps Deploy <apps_deploy>`
