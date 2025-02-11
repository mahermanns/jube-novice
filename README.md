# Reproducible HPC workflows using JUBE

When applying for computing time on HPC systems, applicants are often asked to provide measurements on different scales of parallelism.  Furthermore, preparing performance measurements often involves an application-specific workflow as well as platform-specific configurations.  The objective of this lesson is to enable users of HPC systems to run performance measurements with minimal intervention with high reproducibility, using the [Jülich Benchemarking Environment (JUBE)](https://apps.fz-juelich.de/jsc/jube/docu/index.html) [1].

## Contributing

This lesson was created as material for HPC.NRW workshops on this topics and are released under [CC-BY 4.0 license](https://creativecommons.org/licenses/by/4.0/).

Please check the file `CONTRIBUTING.md` for details on how to contribute to this lesson.

### Building Locally

This lesson is using the [Carpentries Workbench](https://carpentries.github.io/sandpaper-docs/). If you edit the lesson, it is important to verify that the changes are rendered properly in the online version. The best way to do this is to build the lesson locally. You will need an R environment to do this: as described in the {sandpaper} docs, the environment can be either your terminal or RStudio.

#### Setup

The `environment.yaml` file describes a Conda virtual environment that includes R, pandoc, and termplotlib: the tools you'll need to develop and run this lesson, as well as some depencencies. To prepare the environment, install Miniconda following the official instructions. Then open a shell application and create a new environment:

```sh
user@cluster:~$ cd path/to/local/jube-novice
user@cluster:jube-novice$ conda env create -f environment.yaml
```

N.B.: the environment will be named "jube-novice" by default. If you prefer another name, add `-n <alternate_name>` to the command.

#### {sandpaper}

{sandpaper} is the engine behind The Carpentries Workbench lesson layout and static website generator. It is an R package, and has not yet been installed. Paraphrasing the installation instructions, start R or radian, then install:

```sh
user@cluster:hpc-workflows$ R --no-restore --no-save
```
```r
install.packages(c("sandpaper", "varnish", "pegboard", "tinkr"),
 repos = c("https://carpentries.r-universe.dev/", getOption("repos")))
```

Now you can render the site! From your R session,

```r
library("sandpaper")
sandpaper::serve()
```

This should output something like the following:

```
Output created: jube-novice/site/docs/index.html
To stop the server, run servr::daemon_stop(1) or restart your R session
Serving the directory jube-novice/site/docs at http://127.0.0.1:4321
```

Click on the link to `http://127.0.0.1:4321` or copy and paste it in your browser. You should see any changes you've made to the lesson on the corresponding page(s). If it looks right, you're set to proceed!

## References

[1] JUBE Documentation: <https://apps.fz-juelich.de/jsc/jube/docu/index.html>


