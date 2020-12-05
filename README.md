# Useful tips to get started with Nextflow
**(Please report errors and suggestions in Issues. Thank you!)**

### Places to search for answers:
- [Nextflow patterns, official](https://nextflow-io.github.io/patterns/index.html)
- [Gitter chat room](https://gitter.im/nextflow-io/nextflow)
- [Google group](https://groups.google.com/forum/#!forum/nextflow)
- The old posts in these places are a treasure dump. I find 99% of my answers in there. In fact the last function `.collect{ it[1] }` in the cheatsheet came from a post in Gitter by (@Juke34)[https://github.com/Juke34]

### The working directory
- **Each execution of a process happens in its own temporary working directory.** 
- The working directory is the folder named like `/path_to_tmp/4d9c3b333734a5b63d66f0bc0cfcdc` that Nextflow points you to when there is an error in execution. This folder contains the error log that could be useful for debugging. One can find the folder path in the .nextflow.log or in the report.html. 
- This folder only contains files (usually in form of symlinks, see below) from the input channel, so it's isolated from the rest of the file system. 
- This folder will also contain all output files (unless specifically directed elsewhere), and only those specified in the output channels and `publishDir` will be moved or copied to the `publishDir`.
- Be mindful that if the `"""` script section `"""` involves changing folders, such as `cd` or `rmarkdown::render( knit_root_dir = "folder/" )`, Nextflow will still only search the working directory for output files. 
- Run `nextflow clean -f` in the excecution folder to clean up the working directories.

### Where am I?
- Throughout Nextflow scripts, one can use 
  - `${workflow.projectDir}` to refer to where the nextflow script (usually main.nf) locates. For example: `publishDir "${workflow.projectDir}/output", mode: 'copy'` or `Rscript ${workflow.projectDir}/bin/task.R`.
  - `${workflow.launchDir}` to refer to where the script is called from. 

### path("A.txt")
- `Channel.from( "A.txt" )` will put `A.txt` as is into the channel 
- `Channel.fromPath( "A.txt" )` will add a full path (usually current directory) and put `/path/A.txt` into the channel. 
- `Channel.fromPath( "folder/A.txt" )` will add a full path (usually current directory) and put `/path/folder/A.txt` into the channel. 
- `Channel.fromPath( "/path/A.txt" )` will put `/path/A.txt` into the channel. 
- In other words, `Channel.fromPath` will only add a full path if there isn't already one and ensure there is always a full path in the resulting channel.
- This goes hand in hand with `input: path("A.txt")` inside the process, where **Nextflow actually creates a symlink named `A.txt`** (note the path from first / to last / is stripped) **linking to `/path/A.txt` in the working directory**, so it can be accessed within the working directory by the script `cat A.txt` without specifying a path.

### `input: path("A.txt")` vs. `input: path(A)` in the process section 
- With `input: path("A.txt")` one can refer to the file in the script as `A.txt`. Side note `A.txt` doesn't have to be the same name as in channel creation, it can be anything, `input: path("B.txt")`, `input: path("n")` etc. 
- With `input: path(A)` one can refer to the file in the script as `$A`
- `input: path("A.txt")` and `input: path "A.txt"` generally both work. Occasionally had errors that required the following (tip from [@danielecook](https://github.com/danielecook)): 
  - if not in a tuple, use `input: path “A.txt”` 
  - if in a tuple, use `input: tuple path(“A.txt”), path(“B.txt”)`
- `path(A)` is the same as `file(A)`. `tuple` is the same as `set`. It's recommended to use `path` and `tuple` with newer versions.

### DLS2 vs DLS1
- In DSL1, each queue channel can only be used once. 
- In DSL2, a channel can be fed into multiple processes, but each process can only be called once. The solution is either `.concat()` the input channels so they run as parallel processes, or put the process in a module and import multiple times from the module.
- DSL2 also enforce that each process takes only 1 input channel, so all inputs needs to be combined into 1 channel. See **Nextflow_cheatsheet_channel**.

### Nextflow reports
- Having `report.enabled = true` and `timeline.enabled = true` in the config will let Nextflow write out report for the run. 
- They contain resource usage, status for each execution, path to working directory and time spent queueing and running for each step. 
- Extremely useful for troubleshooting and optimize resources.
- `dag.enabled = true` will draw a flowchart for the pipeline. Needs graphviz installed.

### File paths in Singularity with Docker container
- It's sometimes tricky to get Nextflow to find the files in a Docker container and below info is from trial and error.
- In config, use `singularity.autoMounts = true` OR `singularity.runOptions = '-B /projects/b1042'` to mount the local file paths into the container.
- Then:
  - run Nextflow in the folder where the main.nf is (so ${workflow.projectDir} points to the right place) 
  - specify absolute file paths in a parameter and append it into the file channels. 

