# polyendtracker-formats

Collaborative documentation of the file formats used by the Polyend Tracker

*NOTE: This project is not associated with Polyend. This (mis)information is public domain and the authors can't be held responsible for anything, etc...*

## Status

This document is an incomplete understanding of the `pti`/`mt`/`mtp` file formats as of firmware version `1.6.0`.

Questions are marked inline with `QUESTION`. Please submit PRs to resolve these if you know the answer!

All contributions are welcome.

## Related work

Big thanks to [@jaap3](https://github.com/jaap3) and [@DataGreed](https://github.com/DataGreed) for their awesome work!

* [pti-file-format](https://github.com/jaap3/pti-file-format)
* [polyendtracker-midi-export](https://github.com/DataGreed/polyendtracker-midi-export)
* scrapti - [coming back soon]

## Overview

The Tracker stores projects in the `Projects` folder at the root of the SD card. Each subdirectory there represents a single project. (QUESTION: Is there a limit to the number of projects?) The Tracker ignores folder hierarchies here. These project directories can be named according to the regex `[[:alnum:]\-+@ ]+`. (QUESTION: These are the keyboard keys - is actually more permissive?)

Inside a project folder there is a song file `project.mt` and two directories, `instruments` and `patterns`.

The `instruments` directory contains `pti` files that each specify instrument parameters and embed a PCM waveform. These instrument files are named `[[:digit:]+ [[:alnum:]\-+@ ]+\.pti`, where the digits denote the instrument number.

The `patterns` directory contains `mtp` files that specify pattern contents. These pattern files are named `pattern_0?[[:digit:]]+.mtp`, where the digits denote the pattern number.

The words in these formats are in little-endian order. Each format is framed with a file-type code and a 4 byte CRC32 code. The CRC is at the end of any fixed-length content (which means that it appears between the header and the sample data in a `pti` file).

## `*.mt` song files

* Code `MT` at byte 0
* Fixed length of 1796 bytes
* CRC32 code of all preceding data at byte 1792

TODO - Summarize information from `polyendtracker-midi-export` project

DataGreed has an excellent [diagram](https://github.com/DataGreed/polyendtracker-midi-export/blob/main/reverse-engineering/patterns-reverse-engineering.md).
His [parser](https://github.com/DataGreed/polyendtracker-midi-export/blob/main/polytrackermidi/parsers/patterns.py) is also a good resource.

## `*.mtp` pattern files

* Code `KS` as byte 0
* Fixed length of 6184 bytes
* CRC32 code of all preceding data at byte 6180

TODO - Summarize information from `polyendtracker-midi-export` project

Again, DataGreed has a [parser](https://github.com/DataGreed/polyendtracker-midi-export/blob/main/polytrackermidi/parsers/project.py) for this.

## `*.pti` instrument files

* Code `TI` as byte 0
* Dynamic length - header followed by sample data
* Fixed length header of 392 bytes
* CRC32 code of all preceding header data at byte 388
* Sample data as 16-bit Mono PCM waveform (sequence of `Int16`s) after CRC until end of file

NOTE In some cases the sample count header field is 0, but sample data is still present.

TODO - Summarize information from `pti-file-format` project

jaap3 has a great [table](https://github.com/jaap3/pti-file-format/blob/main/pti.rst) of all of the fields.

## Known changes from firmware version 1.5 to 1.6

* `mt` files increased in size
* Some unknown preamble changed in the `pti` header (probably version).
