# EIS Archiver
This repo will describe the tools used to successfully Archive a given set of webpages. This is was developed for EPA's EIS documents serving catalouge, but it is generalized enough to be used for any webpage. The process involves 3 major steps:

1.	Generating a list of URLs to be archived.
2.	Running the [grab-site](https://github.com/slang800/grab-site) for the list of URLs as a warcfactory process that is constantly generating WARC format files for the supplied URLs.
3.	Verifying the quality of the generated WARCs using [warcat](https://github.com/chfoo/warcat)
4.	Optional wayback machine [pywb](https://github.com/ikreymer/pywb) that be used to playback the generated WARCs.

# Dependecies
Have the following dependecies installed in your machine before getting started with the harvesting proccess.

List of URLs
---
Supplied in the repo is a basic URL generator `csv2Urls.py` it just concatenates a base URL with a value from the supplied csv to form a valid URL. The output is a text file newline delimited that can be supplied to the grab-site docker app. See example files `eis-listing.csv` and `paths.txt` for reference.

**Usage**

Script takes in 3 arguments:
- Path to the csv
- Key/Column name in the csv
- Base

```bash
python csv2Urls.py [path/to/csv] [key name in the csv to concatenate] [base url]
```

OR can have your own way of generating URLs, the app just needs a text file of URLs newline delimited.

Docker
---
Install the Docker Engine on your machine, its available for Mac/Win/Linux, see [instructions](https://docs.docker.com/engine/installation/) for installation specific to your environment. Although this process has been only tested on Mac/Linux, it requires a bash shell. This process hasn't been tested for a Windows environment with or without bash shell.

# Usage
Once the dependecies are installed, specifically the docker environment then you are ready to use grab-site 

grab-site
---
[![Travis](https://img.shields.io/travis/rust-lang/rust.svg)](https://hub.docker.com/r/slang800/grab-site/) [![Docker](https://img.shields.io/badge/docker--repo-pull-blue.svg)](https://hub.docker.com/r/slang800/grab-site/)

Full install/reference [instructions](https://github.com/slang800/grab-site).

Get the pre-built docker container:
```bash
docker pull slang800/grab-site
```

Running
---
To run  grab-site, enter the following command in the terminal:
Can set the port to whatever you want after `-p` flag.
Set Volume path for data output (ie WARCs) after `-v` flag up to `:/data`

```bash
docker run --detach -p [port]:[port] -v [path-to-folder-to-store-data]/grab-site-date:/data --name warcfactory slang800/grab-site
```

Verify the container running using:

```bash
docker ps
```
![here](/docs/img/docker-ps.png)

The container `warcfactory` should appear on the output.

#Starting a crawl
For a single URL crawl use the following command:

```bash
docker exec warcfactory grab-site --no-offsite-links http://example.com
```

Supplied in this repo is a shell script `runWarcfactory.sh` that just reads in a text file of URLs and runs the above command for each and every URLs. 

To run the shell script in background and with no outputs run it as:

```bash
nohup sh runWarcfactory.sh [path_to_text_file_of_URLS] &
```

The crawling will be done automatically for each URLs supplied in the text file.
If your machine is setup or exposed to be a public ip, you can monitor it at `localhost:[port]`, the port being the port the grab-site is running on. The dashboard will show current progress and status of the crawl.

#Tools for verification of WARCS
Below are some tools to verify the validity of the WARC files generated by `grab-site`
TODO: Write usage instructions for `pywb`
warcat
---
[![Travis](https://img.shields.io/travis/rust-lang/rust.svg)](https://travis-ci.org/chfoo/warcat)

Install warcat using pip or source [here](https://github.com/chfoo/warcat).

warcat Quick Installation and usage
-----------------------------------------
- Only Python3 compatible, Install pip3 via ```sudo apt-get install python3-pip```
- Install warcat via ```pip3 install warcat```.
- To verify a valid WARC file - ```python3 -m warcat verify megawarc.warc.gz --progress```
- To extract files from a WARC file - ```python3 -m warcat extract megawarc.warc.gz --output-dir /tmp/megawarc/ --progress```

pywb
---
[![Travis](https://img.shields.io/travis/rust-lang/rust.svg)](https://travis-ci.org/ikreymer/pywb) [![Coverage Status](https://coveralls.io/repos/github/ikreymer/pywb/badge.svg?branch=master)](https://coveralls.io/github/ikreymer/pywb?branch=master)

This tool can playback WARC files on your localhost, this can be used to verify the generated WARCs, full installation instructions [here](https://github.com/ikreymer/pywb)
