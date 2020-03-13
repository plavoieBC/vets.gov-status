# VA.gov Performance Dashboard

## A performance dashboard for va.gov

The performance dashboard is meant to provide a simple overview of the va.gov project to external audiences. It is often used in briefings but should be able to stand-alone with enough context offered for a visitor unfamiliar with va.gov to navigate it and understand it.

## Jekyll Structure

This is a static site generated by [Jekyll](https://jekyllrb.com/docs/). Jekyll uses the [Liquid](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) templating.
 
File and directory names starting with an underscore (`_`) are used by Jekyll. The pages of the website map to `.html` files at the top level and `.md` files in `_boards/`.

Each feature or function gets its own page in [boards](src/_boards/) directory, with the same [layout](src/_layouts/board.html). Each has a [project overview section](src/_includes/header.html), followed by up to three data [tiles](src/_includes/tiles/) of key facts, a set of [charts](src/_includes/tiles/chart.html), and before/after [screenshots](src/_includes/ux_compare.html).

Datapoints for tiles are hard-coded in the YAML front matter under 'tiles' in each HTML/Markdown file. Data for the charts comes from `.csv` files in `_data/`, which are generated by python scripts found in `scripts/`. The scripts fetch data from Google Analytics and ID.me.

> Note: file and folder naming is important for Jekyll. Jekyll looks for folders with specific names (ex: `_includes/`) when building the site. The names of files correspond to parameters in the YAML front matter in `.md` files and `.yml` files.

## Design Context and Details

The performance dashboard has a main landing page `index.html` that provides the va.gov vision statement as context for the project.

The six most important data points on the impact of va.gov are presented in a set of "tiles." These are rotated and may be customized for ahead of briefings to key stakeholders.

A set of summary metrics over time are conveyed in series of charts in a tab group. The metrics are presented as weekly data to smooth out some of the day-to-day variance and provide a long-term trend view. The tabs are used to conserve visual space. The data for the charts is pulled in at build time from csv files in the `_data` directory. Those csv files are updated using Python scripts in `scripts` directory.

Each significant feature or function of va.gov gets its own "tile" in the project section. They are grouped by which of the parts of vision statement they fulfill. Completed features are links to detailed scorecard boards. "Coming soon" features can also be displayed. This section is constructed at build time from the contents of the `_board` directory.

There are special call-out sections for the human-centered design work and progress of migrations.

### Feature/Function Board Fields

Required fields below are marked with the asterisk (*) character. Other fields are optional.
- *title: The name of the feature/function
- date_added: When the feature/function launched 
- vetsdotgov_url: URL on production site 
- *status: `normal` for launched items, `progress` for 'coming soon' items
- *category: Which of the vision statement categories it belongs in: `Discover`, `Apply`, `Track`, `Manage`
- *description: One sentence description, will display on the landing page
- extended_description: Will replace description on detailed page if supplied 
- *screenshot: The file base name for screenshots. If `basename` is entered here, there should be `assets\img\basename.png` for the after and `assets\img\basename_old.png` provided. `placeholder` can be used if no screenshots are ready.
- *tiles: A list of up to three data `tiles` to display. Some examples are in `showcase.html` or check the actual html for each in `_includes\tiles`.
- clicks: Will display an "Outbound links" chart for tracking traffic sent to other sites
- charts: The file base name for the data charts in `_data` and should match what is used in `scripts\config.json`

### Chart data

The charts are powered by the va.gov Google Analytics account. The Python scripts in `scripts` pull data from the Google Analytics account and create a set of updated CSV files in `_data` that are then used by Jekyll to build the actual charts.

Once deployed the data is static until the next deploy. Because the executive scorecard is meant for external audiences, this ensures that the data is available and can be quality controlled prior to putting it in front of an audience. Once deployed, we do not have to worry about data abnormalities or failures appearing.

## Getting Started

### Install yarn

[Yarn](https://yarnpkg.com/) is used to manage javascript dependencies. You can install it with:

`$ brew install yarn`

Then install dependencies with:

`$ yarn install`

### Jekyll

In order to mimic the CI environment, we run Jekyll out of a docker container, instead of locally. We have
built helper scripts as follows:

Build jekyll site:

`$ ./jekyll-build.sh`

Start jekyll site and serve it on http://localhost:4000/scorecard/:

`$ ./jekyll-serve.sh`

Alternatively, you can run

`yarn serve`

### Run Python Scripts

Make sure you have the correct version of python: `pyenv install 3.6.8` (use version in `.python-version`)

Run `./python-install.sh` to install a virtual environment and install the dependencies.

Activate the virtual environment with `source ENV/bin/activate`

Go into the scripts directory and run the scripts with `./fetch-data-local.sh`

### Connecting to AWS

The scripts get secrets out of [credstash](https://github.com/fugue/credstash), which connects to a running
AWS instance using your local credentials.

For interim ThoughtWorks development, your `~/.aws/config` must contain an entry like:

```
[profile va-pension-automation-admin]
role_arn=<arn of the admin role>
source_profile=va-pension-automation-readonly
region=us-east-2
```

1. `saml2aws login`

2. `saml2aws exec --exec-profile va-pension-automation-admin $SHELL` (you may need to reactivate your virtualenv)

### Adding new packages to python scripts

Add the package names to `requirements.in` then run `./upgrade-requirements.sh` to get them synced to `requirements.txt`. 

Then run `./python-install.sh` to update your virtual environment with the new packages.

### Running Tests

Run tests with pytest and generate a coverage report with  `./run-tests.sh`.

### Running with Docker

You can run the scripts in a Docker image using the following commands:

```
$ cd scripts
$ docker build -t scorecard-fetch .
$ docker run --env AWS_SECRET_ACCESS_KEY --env AWS_SESSION_TOKEN --env AWS_ACCESS_KEY_ID scorecard-fetch
```

Note that we pass in the host machine's AWS credentials.

## Deploying

Our deployments are handled by Jenkins using the `Jenkinsfile`. We deploy by committing to the `production` branch. We use the `demo` branch to deploy to our development server to internally demo new boards or tile updates without blocking the data update path from `master` to `production`.

## Developer Onboarding

More useful developer onboarding documentation can be [found here](dev/onboarding.md).

## Previous repo

This repo previously held a now defunct dashboard. The prior work is archived as a release on this repo in case that work needs to be revisited.

## Browsers supported

The current list of supported browsers for scorecard redesign include Chrome 61, Firefox 60, iOS 11, Edge 16, ChromeAndroid 67, Safari 11. This list aligns with the [vets-website list](https://github.com/department-of-veterans-affairs/vets-website/blob/master/.babelrc#L16).
