# peermaps jan 2019 roadmap

This document describes the current iteration of the peermaps roadmap.

This roadmap document will be replaced and archived in this repository as newer
roadmaps are published.

* [eyros][] - multidimensional interval database
* [eyros-swarm][] - kappa-core swarm optimized for spatial queries
* [swarmhead][] - collective batch processing
* [osmhive][] - planet.osm batch processor
* [georender][] - webgl-based map rendering
* [peermaps][] - web-based map viewer/editor

The target platform for the user-facing tools is the web and web-like
environments such as [web extensions][libdweb] and electron. Some components will be
written in javascript and some will be written in rust. Many of the rust
components will be compiled into web assembly to achieve higher performance and
lower memory use for parts of the web stack that will process a high volume of
data.

The batch processing tools are targeted at a linux environment using a
combination of rust and node.js.

# eyros: multidimensional interval database

Eyros is a database for multidimensional intervals and scalar points. The data
format is designed so that queries can be performed over p2p networks without
downloading the entire database. Internally, eyros is based on interval trees
which allow querying regions with features that might span the search window
completely with none of the feature's points contained inside. These properties
will (hopefully) let us skip additional processing for polygon clipping or
geometry duplication at the rendering stage that is often required with
tile-based approaches so that we may instead directly render features directly
from the database.

# [eyros-swarm][]: kappa-core swarm optimized for spatial queries

Set up a swarm for a topic (such as "planet.osm") to process log data through
kappa-core with eyros materialized views and a spatially-aware hypercore
extension.

The larger eyros materialized views such as planet.osm are derived from primary
log data but distributed as hypercores themselves.

The spatially-aware hypercore extension will reduce the number of round-trips
required for knowing which blocks to fetch when loading from a cold cache. For
example, if a peer supports the extension, a client can share the spatial
bounding box that they are interested in and the peer can perform the query
locally to get a list of blocks. This avoids `log(n)` round-trips down each
level of the tree to find the appropriate set of blocks, but in an optimistic
backwards-compatible way if the other peer doesn't support the extension.

# [swarmhead][]: collective batch processing

Find other peers under a swarm topic and coordinate running batch jobs and
archiving the results. The results are written to hypercores managed by
kappa-core to synthesize the results.

A big design goal here is to make tools in the style of seti@home which can
accept donated resources while also not depending on fixed infrastructure. With
these features, people can build networks on top of volunteer resources that
remain useful so long as some people run and care for running swarm nodes.
Ideally, this will make it possible to run cooperative computing projects which
can outlast the interest of the initial participants. 

# [osmhive][]: planet.osm batch processor

Process planet.osm (including diffs) using swarmhead to generate archives and
diffs of eyros database releases. Clients should use heuristics to determine
whether it makes more sense to continue to use an older cache and download the
diff or invalidate their old sparse replica and start sparsely replicating a new
one. This is a consideration that users should have input into through the user
interface because invalidating an old cache can be very disruptive to some kinds
of work.

# [georender][]: webgl-based map rendering

Render map geometry from the interval database as a [mixmap][] layer.

It might make sense to cache some aspects of the geometry engine on clients in
durable storage or if the costs are high enough and the transfer size justifies
it, to also distribute some artifacts in the interval database directly.

One important goal is to be able to edit the geospatial data stored in the
interval tree and directly render the updated view without having those changes
cascade into updating multiple large tile artifacts for each batch of edits.

Another goal is to be able to share the same well-seeded p2p data for many
different styles of mapping using custom shaders and visualizations. Storing
separate tile formats for different graphical customizations is wasteful and
should be avoided where possible.

This layer will also need an iterative label placement engine.

# [peermaps][]: web-based viewer/editor

Peermaps provides a standalone and embeddable web application for map browsing
and editing on top of the p2p network, data products, and rendering layers.

In addition to graphical mapping tools, the peermaps application will provide a
layering system for organizing databases and network tools to manage data
storage and network use profiles to know when it's permissable to send and
receive large files.

A [cryptographic capability system][hyperlog-capability] on top of
[kappa-core][] will encrypt maps to allow offsite hosting and granular
permissions for multi-user collaboration.

[eyros]: https://github.com/peermaps/eyros
[swarmhead]: https://github.com/peermaps/swarmhead
[osmhive]: https://github.com/peermaps/osmhive
[georender]: https://github.com/peermaps/georender
[peermaps]: https://github.com/peermaps/peermaps
[mixmap]: https://github.com/peermaps/mixmap
[kappa-core]: https://github.com/kappa-db/kappa-core
[libdweb]: https://github.com/mozilla/libdweb
[hyperlog-capability]: https://github.com/substack/hyperlog-capability
