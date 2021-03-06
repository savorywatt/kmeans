This benchmark is born to compare the performance of Pharo 3 in executing a simple machine learning algorithm with a reference implementation in Python and Scala. Since then, it got a little out of hand, and a few other implementations are available.

Rules
=====

The implementations should all follow the same algorithm, and be optimized for idiomatic code and not for speed. The example is intended to compare time of execution for a typical machine learning algorithm, ideally during an interactive session, instead of highly optimized production code. As such, it is important that the code is straightforward and that there is no separate phase to prepare the caches.

The points are in `points.json`, and are to be grouped into 10 clusters, using 15 iterations of kmeans. The initial centroids are initialized to the first 10 points, and we take an average over 100 runs.

Results
=======

Time for running on my laptop are available under `results`. A few surprises:

* Writing a working Rust implementation was surprisingly difficult; writing one that would perform decently even more so. I had to rely frequently on help from people online.
* PyPy is able to outperform Scala
* Factor is pretty impressive, given that it is a fairly small project with a dedicated VM. With an implementation in 8 (!) lines, we get the a fairly performing dynamic language
* Nim was also quite impressive: my first implementation was as easy as Python, and it was just behind Rust; when an unnecessary copy was removed, it turned out to be the fastest.

Contribute
==========

If you want to contribute an implementation in a different language, please file a PR. Try to follow the same logic that is used in the examples in other languages - for instance, using a group by operation where available. As you may notice, the algorithm is not optimized, and intentionally so: while K-means in particular has various possible optimizations, other similar algorithms may fail to have the particular shape that makes these optimizations viable.

For the curious folks, I have tried a more optimized (single-threaded) implementation in Nim, that avoids the square root in the distance and accumulates the sum of points near a centroid, rather than putting them into a data structure. For comparison, this version runs in 67 ms. Computers are actually quite fast, these days!

How to run
==========

**C**

    sudo apt-get install libglib2.0-0
	sudo apt-get install libjansson-dev # or equivalent for your OS
	./compile.sh
	./kmeans

**Clojure**: `lein with-profile uberjar run`

**Erlang**:

    erl
    1> c(main).
    2> c(kmeans).
    3> main:run().

**Factor**:

    USE: kmeans.benchmark
    100 "../points.json" kmeans-benchmark

**Java**:

	mvn compile
	mvn exec:java

**Julia**:

    julia -e 'Pkg.add("JSON")'
    julia kmeans.jl

**Kotlin**:

    mvn compile exec:java

**Lua**: download [this JSON library](http://dkolf.de/src/dkjson-lua.fsl/home) and put it in the same folder as the main file. Then run

    lua kmeans.lua
    luajit kmeans.lua

**Nim**:

    nim c -d:release benchmark
    ./benchmark

**Node**:

    npm install
    node kmeans.js

**OCaml**:

    opam install core yojson
    corebuild -pkg yojson main.native
    ./main.native

**Parasail**: assume `pslc.csh` is on `$PATH`. Then

    pslc.csh -O3 point.psl kmeans.psl benchmark.psl -o benchmark
    ./benchmark

**Pharo3**: install `NeoJSON` and file-in `Kmeans.st`, then open a workspace and write something like

    | path points kmeans |

    path := '../points.json'.

    kmeans := KMeans new
      iterations: 15;
      clusters: 10;
      yourself.

    StandardFileStream readOnlyFileNamed: path
      do: [ :stream |
        points := (NeoJSONReader on: stream) next collect: [ :each |
          (each first) @ (each second)
        ].
      ].

    kmeans benchmark: points repeating: 100

**Python**:

    python kmeans.py
    pypy kmeans.py

**Rust**:

    cargo run --release

**Scala**: `sbt run`
