# Data Processing (ETL) Pipeline for 2020 100&Change Competition

The 2020 [Lever for Change 100&Change
Competition](https://www.leverforchange.org/what-we-do/explore-competitions/100change/)
receives many proposals, which come to us in a giant CSV file.  This
code processes that CSV, together with various side inputs, and loads
the processed data into a
[Torque](https://github.com/opentechstrategies/torque/) site where
internal evaluation of the proposals is conducted.

## Dependencies

You will need Python 3 and the following tools:

* [Beautiful Soup 4](https://www.crummy.com/software/BeautifulSoup/)
* [MWClient](https://github.com/mwclient/mwclient)
* [Unidecode](https://pypi.python.org/pypi/Unidecode)
* [csv2wiki](https://github.com/OpenTechStrategies/csv2wiki/)

The first three can be installed with `pip3 install bs4 mwclient
unidecode`.  For `csv2wiki`, fetch the sources and install it on your
system:

```
$ get clone https://github.com/OpenTechStrategies/csv2wiki
$ cd csv2wiki
$ pip3 install -e . # From csv2wiki README
```

This code is designed to work with a properly-configured
[Torque](https://github.com/OpenTechStrategies/torque) instance.
See the Torque documentation for details.

### Note about csv2wiki

`csv2wiki` is a substantial program that we started out using to
populate the whole wiki.  After moving to Torque, we no longer needed
most of the `csv2wiki` features having to do with converting CSV rows
to wiki pages, and we use just the features that make adding pages to
a wiki convenient.

### Configuration

Copy `macfound-torquedata-csv2wiki-config.tmpl` to
`macfound-torquedata-csv2wiki-config` then edit the latter in the
obvious ways.  The wiki credentials should match those specified in
the [ansible](https://www.ansible.com/) configuration for
the [Torque](https://github.com/OpenTechStrategies/torque) server.

## Running

The entry point is the `torque-refresh` script.  Run it like this:

```
$ ./torque-refresh /path/to/data
```

(If you typically store your data encrypted, as we do, then unencrypt
it first.)

## Components

* torque-refresh

  Start from the [torque-refresh](torque-refresh) script and see how it
  drives first [compose-csvs](compose-csvs), which performs some one-time
  transformations to a specific CSV file from the MacArthur Foundation,
  as well as joining them with some supplemental data, and then uploads
  the final CSV to a running torque server.  After which is creates
  proposal pages using
  [csv2wiki](https://github.com/OpenTechStrategies/csv2wiki), that
  each have a single line that calls into the torque server.

* torque

  This system depends on a running
  [Torque](https://github.com/OpenTechStrategies/torque) system.
  It uses the TorqueDataConnect extensionsion to upload the
  spreadsheet and TOCs after generating them.

* compose-csvs

  This is a script that torque-refresh uses to sanitize and combine csvs
  for use.  You need to have access to the files listed in it to create
  the final proposal csv, as well as all the TOC json data to upload
  to the torque server.

  Look at the file for more information, especially the usage section
  and how torque-refresh uses it.

* upload-csvs

  After compose-csvs has generated all the files, upload-csvs is
  responsible for taking the generated files and sending them to
  the running torque/mediawiki server.

  This also uploads all the attachments specific to the MacArthur
  foundation.
