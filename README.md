# Overview

Data processing for metabolomics data. Pipeline overview:

![alt text](https://github.com/GalaxyDream/metabolomics_data_processing/blob/master/figs/pipeline.png)

---

# Licence

This program is released as open source software under the terms of [GNU GPL-v3.0 License](https://github.com/GalaxyDream/metabolomics_data_processing/blob/master/LICENSE).

# Installing

You can install bionitio directly from the source code or build and run it from within Docker container.

Clone this repository: 
```
$ git clone https://github.com/GalaxyDream/metabolomics_data_processing.git
```

Move into the repository directory:
```
$ cd metabolomics_data_processing
```

[Nextflow](https://www.nextflow.io/) and [Docker](https://www.docker.com/) (or [Singularity](https://singularity.lbl.gov/) if using high-performance computing) are required for this software

# General Behavior

UMPIRE accepts `.mzXML` and `.mzXL` files. Files are processed in parallel using [MZmine-2.53](http://mzmine.github.io/); several statists are calculated using [Python3](https://www.python.org/download/releases/3.0/) codes; interactive report is generated with [MultiQC](https://multiqc.info/); and pathway analysis are done with [mummichog](http://mummichog.org/).

### Default parameter settings for MZmine-2.53:

- Mass detection (detector: Centoid; noise level: 1,000; mass list name: masses; MS level: 1)
- Chromatogram buider (Mass list: masses; Min time span: 0.06; Min height: 1.0E5; m/z tolerance: 0.002 m/z or 5.0 ppm; Suffix: "chromatograms-1E5")
- Smoothing (Filename suffix: "smoothed"; Filter width: 5; Remove original peak list: False)
- Chromatogram deconvolution (Suffix: "deconvolutedTG-dd"; Algorithm: Local minimum search; Chromatographic threshold: 0.95; Search minimum in RT range (min): 0.05; Minimum relative height: 0.05; Minimum absolute height: 30000.0; Min ratio of peak top/edge: 3.0; Minimum peak duration range (min): 0.06; Maximum peak duration range (min): 1.0; Remove original peak list: True)

### Following statistics are currently included in UMPIRE

* *Student t-test*: Test if there is a significant statistical difference of certain peak intensities between the two groups of samples.
* *Venn diagram*: Report the number of peaks that are significantly enriched in one of the groups, and the number of peaks that have no significant difference between two groups.
* *Principal component analysis*: Dimensional reduction using the peak intensities of the two group samples, and visualize the difference.
* *hierarchical clustering*: Cluster all samples and plot a heatmap to show the difference between samples and peaks.
* *bar plot*: plot the metabolites with top-10 and bottom-10 fold-change for the comparison between two groups. (note: the figure will display abnormally if there is an infinite fold change value)

### Process your own data

- Save your positive data files to `data/POS/` and negative data to `data/NEG/`
- Create design files for positve data and negative data, indicating the group of each file. Sample design file can be found in `data/sample_data/pos_design.csv` and `data/sample_data/neg_design.csv`
- Process your data with default parameters using local machine
```
Nextflow run_all.nf -with-docker galaxydream/metabolomics_pipeline
```
- Process your data with default parameters using high-performance computing
```
mkdir -p work/singularity &&
cd work/singularity &&
singularity pull --name galaxydream-metabolomics_pipeline.img docker://galaxydream/metabolomics_pipeline &&
cd ../../ &&
Nextflow run_all.nf -with-singularity docker://galaxydream/metabolomics_pipeline
```

### Process dataframe generatd by MZmine-2.53

- Save the dataframe for positive data and negative data to `data/pos_data.csv` and `data/neg_data.csv`
- Create design files describing the group of each column of positive/negative data, save them to `data/pos_design.csv` and `data/neg_design.csv`
- Get statistical analysis and pathway analysis
```
Nextflow run_aftermzmine.nf -with-docker galaxydream/metabolomics_pipeline
```

### Help message

UMPIRE can display usage information on the command line:
```
Nextflow run_all.nf --help true
```

### Logging

Log file `nextflow_report.html` will be output to current folder.

### Clean repository

Run the following command to clean all the files generated by previous run (except the pulled singularity)
```
bash clear.sh
```

