
## Setup

Pulse is a [Flask](http://flask.pocoo.org/) app written in **Python 3**. We recommend [pyenv](https://github.com/yyuu/pyenv) for easy Python version management.

* Install dependencies:

```bash
pip install -r requirements.txt
```

* If developing the stylesheets, you will also need [Sass](http://sass-lang.com/), [Bourbon](http://bourbon.io/), [Neat](http://neat.bourbon.io/), and [Bitters](http://bitters.bourbon.io/).

```bash
gem install sass bourbon neat bitters
```

* If editing styles during development, keep the Sass auto-compiling with:

```bash
make watch
```

* And to run the app in development, use:

```bash
make debug
```

This will run the app with `DEBUG` mode on, showing full error messages in-browser when they occur.

### Initializing dataset

To initialize the dataset with the last production scan data and database, there's a convenience function:

```
make data_init
```

This will download (using `curl`) the current live production database and scan data to the local `data/` directory.

## Deploying the site

The site can be easily deployed (by someone with credentials to the right server) through [Fabric](https://github.com/fabric/fabric), which requires Python 2.

The Fabric script will expect a defined `ssh` configuration called `pulse`, which you should already have defined in your SSH configuration with the right hostname and key.

To deploy to staging, switch to a Python 2 virtualenv with `fabric` installed, and run:

```
make staging
```

This will `cd` into `deploy/` and run `fab deploy`.

To deploy to production, activate Python 2 and `fabric` and run:

```
make production
```

This will run the fabric command to deploy to production.

## Updating the data in Pulse

The command to update the data in Pulse and publish it to production is simple:

```
python -m data.update
```

**But you will need to do some setup first.**

### Install domain-scan and dependencies

Download and set up `domain-scan` [from GitHub](https://github.com/18F/domain-scan).

`domain-scan` in turn requires [`pshtt`](https://github.com/dhs-ncats/pshtt) and [`ssllabs-scan`](https://github.com/ssllabs/ssllabs-scan). These currently both need to be cloned from GitHub and set up individually.

Pulse requires you to set one environment variable:

* `DOMAIN_SCAN_PATH`: A path to `domain-scan`'s `scan` binary.

However, `domain-scan` may need you to set a couple others if the binaries it uses aren't on your path:

* `PSHTT_PATH`: Path to the `pshtt_cli` binary.
* `SSLLABS_PATH`: Path to the `ssllabs-scan` binary.

### Configure the AWS CLI

To publish the resulting data to the production S3 bucket, install the official AWS CLI:

```
pip install awscli
```

And link it to AWS credentials that allow authorized write access to the `pulse.cio.gov` S3 bucket.

### Then run it

From the Pulse root directory:

```
python -m data.update
```

This will kick off the `domain-scan` scanning process for HTTP/HTTPS, using the `.gov.jm` domain list as specified in `meta.yml` for the base set of domains to scan.

Then it will run the scan data through post-processing to produce some JSON and CSV files the Pulse front-end uses to render data.

Finally, this data will be uploaded to the production S3 bucket.



