# Higgs-Charm documentation website

This repository holds the markdown files which are used by [MKDocs](https://www.mkdocs.org/) to produce the [Higgs-Charm documentation website](https://higgs-charm.docs.cern.ch/). The website is hosted by CERN on Nordin Breugelmans' EOS storage on lxplus.

## Making changes to the repo

First, you need to have a local [MKDocs](https://www.mkdocs.org/) installation. One way to do this is by executing the following command in your terminal:

```shell
pip install mkdocs
```

Also, make sure you have cloned this repository:

```shell
git clone https://github.com/nbreugel/higgs-charm-docs.git
```

At the moment, the file-structure of this MKDocs project is the following:

```text
mkdocs.yml
cinder/
site/
docs/
	index.md
	javescript/
	stylesheets/
	higgsjet/
	simulation/
```

The most important file is `mkdocs.yml`. It contains the top-level information required by MKDocs to build the website. It looks somewhat like this:

```yaml
site_name: Higgs-Charm
site_url: https://higgs-charm.docs.cern.ch/
site_author: Nordin Breugelmans
theme:
	name: null
	custom_dir: 'cinder'
nav:
	- Home: 'index.md'
	- Simulation:
		'Using the repository': 'simulation/repository.md'
		etc...
	- 'Higgs + Jet':
		- 'Leading order': 'higgsjet/leading-order.md'
		etc...
	- etc...

and some other, less important stuff here
```

Most likely, the only changes you will ever need to make will be to the `nav:` field. This reflects the structure of the pages of the website and contains the page name and the paths relative (to `docs/`) to the Markdown files which contains the body of the page.

The other directories have the following purpose:

* `cinder/`: This contains the stylesheets etc. for the `cinder` [custom theme](https://github.com/chrissimpkins/cinder) for MKDocs.
* `site/`: This contains the files which produce the website. This directory is created and updated automatically after running `mkdocs build`.
* `docs/`: This is the main working directory for this project. It contains the Markdown files from which the website is built.

If you only make changes to an already existing page (i.e. a Markdown file), all you need to do is simply save the file and then run `mkdocs build`. Afterwards, you can push your changes (there will also be changes in the `site/` directory which contains the latest build, make sure you push these as well!).

If you have created a new Markdown file and therefore a new page, you need to make sure that the `nav:` field above reflects this change as well by adding an entry `- 'Page Name': 'path/to/file.md'` at the appropriate level.
