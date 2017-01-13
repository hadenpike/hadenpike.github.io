---
layout: post
title: Send Books to Kindle From the Command Line
categories: Programming Projects Utilities
---

# Introduction

I'm a firm believer that reading is one of the great joys of life. From Science Fiction to biographies, there's something for everyone. Whenever possible, I purchase books in the EPUB format. As Amazon's Kindle doesn't support that format, I need to convert them to a format the Kindle understands (MOBI usually), and then upload them to my device. This can be done by hand, but I wanted an automated way to do it.

Fortunately, Amazon provides the [Kindle Personal Document Service](https://www.amazon.com/gp/help/customer/display.html/ref=hp_pdoc_main_short_us?nodeId=200767340). Briefly, this service allows you to send documents to an email address and have them appear on your Kindle. Thus, I created s2k. This utility allows you to send files to your Kindle email from the command line.

## Installation

First, download and install [the Python programming language](https://www.python.org/). Then, to install use pip:

    $ pip install s2k


Or clone the repo:

    $ git clone https://github.com/hadenpike/s2k.git
    $ python setup.py install

### Configuration

First, follow the instructions [here](https://www.amazon.com/gp/help/customer/display.html?nodeId=201974220) to view your Send to Kindle email address. The default location of the configuration file is ~/.s2krc. Create it if it does not exist. The configuration file is written in the standard INI format, and supports the following values. Any key without a default value is required.

| Option | Type | Description | Default |
| ----- | ---- | ----------- | ------- |
| from | string | The email address to send from | The $EMAIL environment variable |
| to | string | Your Kindle Email address | |
| server | string | The address of the SMTP server | |
| port | integer | The port the SMTP server is listening on | 587 |
| username | string | The username to log into the SMTP server | |
| password | string | The password to log into the SMTP server | |

## Usage

$ s2k -h
usage: s2k [-h] [-cf CONFIGFILE] [-c] f [f ...]

Send files to your Kindle email

positional arguments:
  f                     Files to send to Kindle.

optional arguments:
  -h, --help            show this help message and exit
  -cf CONFIGFILE, --configfile CONFIGFILE
                        Location of configuration file.
  -c, --convert         Request the Kindle service to convert files to
                        internal format.

