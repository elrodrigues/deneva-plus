Deneva
=======

Deneva is a testbed of an OLTP distributed database management system (DBMS). It supports 6 concurrency control algorithms.

This testbed is based on the DBx1000 system, whose concurrency control scalability study can be found in the following paper:

    Staring into the Abyss: An Evaluation of Concurrency Control with One Thousand Cores
    Xiangyao Yu, George Bezerra, Andrew Pavlo, Srinivas Devadas, Michael Stonebraker
    http://voltdb.com/downloads/datasheets_collateral/vdb_whitepaper_staring_into_the_abyss.pdf

This project, "Deneva Plus", aims to add more features and concurrency control algorithms to the Deneva testbed. Details
about Deneva can be found in the original paper:

    An Evaluation of Distributed Concurrency Control
    Rachael Harding, Dana Van Aken, Andrew Pavlo, Michael Stonebraker
    https://vldb.org/pvldb/vol10/p553-harding.pdf
Setup
------------
Deneva has three required dependencies that need to be installed:

* [Boost](https://www.boost.org/)
* [jemalloc](https://github.com/jemalloc/jemalloc/releases/tag/5.3.0)
* [nanomsg](https://github.com/nanomsg/nanomsg/releases/tag/1.2.1)

If you'd like to measure energy consumption too, you will need the following optional dependencies (INTEL ONLY):
* [powercap](https://github.com/powercap/powercap)
* [raplcap](https://github.com/powercap/raplcap)

You will need to install the Boost library through your package manager, while the two other dependencies have been included as submodules that can be pulled when you run

    git clone --recurse-submodules https://github.com/elrodrigues/deneva-plus.git

You can build `jemalloc` without installing by running the following inside `jemalloc/`

    ./autogen.sh --with-jemalloc-prefix="je_"
    make -j

See `powercap` README on how to build and install it. Since this Deneva implementation
uses the powercap version of `raplcap`, you will need to install `powercap` too on your
target system and have the `intel_rapl_msr` kernel module loaded.

See `raplcap` README on how to build it, with the following exception: instead of just running `cmake ..` ,
run the following when generating build files for `raplcap` inside `deneva-plus/raplcap/_build/`:

    cmake .. -DBUILD_SHARED_LIBS=On -DCMAKE_BUILD_TYPE=Release

If your target system exposes the `msr` registers in `/dev/cpu`, then you can build Deneva
Plus against the MSR version of `raplcap` and skip installing `powercap`. You can do this
by changing `RPLCAPTYPE` option in the Makefile from `powercap` to `msr` before building.
While you can compile and link Deneva against a local build of `raplcap`, you may need to
copy the `libraplcap-xxx.so` shared library to `/usr/lib` or `/usr/local/lib`. Note that
energy measurements usually require additional privileges, so you may need to run target
servers with `sudo` or `doas`.

See `nanomsg` README on how to build it. You do not need to install it
since Deneva searches for the shared object locally when building. However,
you may need to copy or move `libnanomsg.so` and its soft links to a path
in your shared library cache like `/usr/lib`, `/usr/lib64`, or
`/usr/local/lib`. You may also need to refresh the runtime shared library
cache by running

    sudo ldconfig

To be able to make the code successfully there needs to be a directory named obj.
Run this in the project root

    mkdir obj

Build & Test
------------
To build the database.

    make deps
    make -j

Configuration
-------------

DBMS configurations can be changed in the `config.h` file. Please refer to README for the meaning of each configuration. Here we only list several most important ones.

    NODE_CNT          : Number of server nodes in the database
    THREAD_CNT        : Number of worker threads running per server
    WORKLOAD          : Supported workloads include YCSB and TPCC
    CC_ALG            : Concurrency control algorithm. Six algorithms are supported
                        (NO_WAIT, WAIT_DIE, TIMESTAMP, MVCC, OCC, CALVIN)
    MAX_TXN_IN_FLIGHT  : Maximum number of active transactions at each server at a given time
    DONE_TIMER        : Amount of time to run experiment

Configurations can also be specified as command argument at runtime. Run the following command for a full list of program argument.

    ./rundb -h

Run
---

Create a file called ifconfig.txt with IP addresses for the servers and clients, one per line.

The DBMS can be run with

    ./rundb -nid[N]
    ./runcl -nid[M]

where N and M are the ID of a server and client, respectively

For example, if you are running with a `THREAD_CNT` of 4 you would run

    ./rundb -nid0
    ./rundb -nid1
    ./runcl -nid2
    ./runcl -nid3

There is also the option to run scripts. From the scripts directory run

    python3 run_experiments -e [experiment]

* List of available experiments can be found [here](https://github.com/mitdbg/deneva/blob/master/scripts/experiments.py)

After running an experiment, the results can be plotted by running

    python3 plot.py [experiment]
