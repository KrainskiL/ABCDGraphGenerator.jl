# ABCDGraphGenerator.jl
Artificial Benchmark for Community Detection (ABCD) - A Fast Random Graph Model with Community Structure

Bogumił Kamiński, Paweł Prałat, François Théberge

---

### Julia API

The package does not export functions and types. The public API is the following:
* `ABCDGraphGenerator.ABCDParams`: type holding information about sampled degrees,
  sampled cluster sizes and required mode of ABCD graph generation
* `ABCDGraphGenerator.gen_graph`: ABCD graph generator that uses `ABCDParams`
  specification
* `ABCDGraphGenerator.sample_degrees`: sample degrees of vertices following power law
* `ABCDGraphGenerator.sample_communities`: sample community sizes following power law
* `ABCDGraphGenerator.get_ev`: get expected value of truncated discrete power law distribution
* `ABCDGraphGenerator.find_v_min`: find the lower truncation given expected value
  and upper truncation of truncated discrete power law distribution

The resason for such split of the functionality is that generation of vertex degrees
and community sizes is fast, while the generation of the final graph is the most expensive step.

### Using from R and Python

The functions provided in the package can be directly called from R and Python.

Instructions how to seamlessly integrate Julia into R session are given in the [JuliaCall](https://cran.r-project.org/web/packages/JuliaCall/index.html) package documentation.

An interface to call Julia directly from Python is provided by the [PyJulia](https://github.com/JuliaPy/pyjulia) package.

### Command Line Interface

The `utils/` folder contains command line interface utilities that are aimed at users that want
to the use package without using the API directly. A requirement for these
utilities to be run is to have the Julia language in version at least 1.0 installed on a computer.
It contains the following files:
* `install.jl`: installs all required packages
* `abcd_sampler.jl`: generates an ABCD graph folloging a configuration file
* `deg_sampler.jl`: samples degrees of vertices in the graph
* `com_sampler.jl`: samples communitiy sizes in the graph
* `graph_sampler.jl`: samples edges and community assignments in the graph

The main file intended to be used is `abcd_sampler.jl`.
Here is an example configuration file, named `example_config.toml`, in this guide:
```
n = "10000"                   # number of vertices in graph
t1 = "3"                      # power-law exponent for degree distribution
d_min = "5"                   # minimum degree
d_max = "50"                  # maximum degree
d_max_iter = "1000"           # maximum number of iterations for sampling degrees
t2 = "2"                      # power-law exponent for cluster size distribution
c_min = "50"                  # minimum cluster size
c_max = "1000"                # maximum cluster size
c_max_iter = "1000"           # maximum number of iterations for sampling cluster sizes
xi = "0.2"                    # fraction of edges to fall in background graph
isCL = "false"                # if "false" use configuration model, if "true" use Chung-Lu
degreefile = "deg.dat"        # name of file do generate that contains vertex degrees
communitysizesfile = "cs.dat" # name of file do generate that contains community sizes
communityfile = "com.dat"     # name of file do generate that contains assignments of vertices to communities
networkfile = "edge.dat"      # name of file do generate that contains edges of the generated graph
```
In this file all parameters required to generate an ABCD graph and store to on disk are passed.
Here is an output from an example session using CLI in the ABCD-generation mode using the above file:
```
$ julia abcd_sampler.jl example_config.toml
[ Info: Usage: julia abcd_sampler.jl config_filename
[ Info: For the syntax of config_filename see example_config.toml file
[ Info: Expected value of degree: 8.327743727955891
[ Info: Expected value of community size: 156.5613820733916
```
After the program terminates four files, `deg.dat`, `cs.dat`, `com.dat` and `edge.dat`
are created in the working directory.

`deg_sampler.jl`, `com_sampler.jl` and `graph_sampler.jl` files are provided
mainly to facilitate comparisons with LFR algorithm.
Here is an output from an example session using CLI in the LFR-comparison mode:
```
$ julia install.jl
  Updating registry at `~\.julia\registries\General`
  Updating git-repo `https://github.com/JuliaRegistries/General.git`
  Updating git-repo `https://github.com/bkamins/ABCDGraphGenerator.jl`
  Updating git-repo `https://github.com/bkamins/ABCDGraphGenerator.jl`
 Resolving package versions...
  Updating `~\.julia\environments\v1.3\Project.toml`
  [4c9194b5] ~ ABCDGraphGenerator v0.1.0 #master (https://github.com/bkamins/ABCDGraphGenerator.jl)
  Updating `~\.julia\environments\v1.3\Manifest.toml`
  [4c9194b5] ~ ABCDGraphGenerator v0.1.0 #master (https://github.com/bkamins/ABCDGraphGenerator.jl)

$ julia deg_sampler.jl degrees.dat 3 5 50 10000 1000
[ Info: Usage: julia deg_sampler.jl filename τ₁ d_min d_max n max_iter
[ Info: Example: julia deg_sampler.jl degrees.dat 3 5 50 10000 1000
[ Info: Expected value of degree: 8.327743727955891

$ julia com_sampler.jl community_sizes.dat 2 50 1000 10000 1000
[ Info: Usage: julia com_sampler.jl filename τ₂ c_min c_max n max_iter
[ Info: Example: julia com_sampler.jl community_sizes.dat 2 50 1000 10000 1000
[ Info: Expected value of community size: 156.5613820733916

$ julia graph_sampler.jl network.dat community.dat degrees.dat community_sizes.dat 0.2 true true
[ Info: Usage: julia graph_sampler.jl networkfile communityfile degreefile communitysizesfile μ isCL islocal
[ Info: Example: julia graph_sampler.jl network.dat community.dat degrees.dat community_sizes.dat 0.2 true true
```
After running these commands you will have the following files in your working directory (all data is 1-based)):
* `degrees.dat` a sequence of vertex degrees (in descending order)
* `community_sizes.dat` a sequence of cluster sizes (in descending order)
* `community.dat` a sequence of vertex number-community number pairs
* `network.dat` a sequence of generated edges sorted lexicographically as pairs of vertices (in increasing order)
