# Baseball Analytics
&#x1F534; Under Construction - may be ready for use by 12/21/19  

Scripts to download, parse, and prepare baseball data for analysis with Pandas.

Additional scripts to optionally load data into Postgres for analysis with SQL.

## Summary

Scripts have been created to download and wrangle the Lahman and Retrosheet MLB datasets to make analysis with Pandas or SQL using a database such as Postgres, easier.  It is not necessary to use both Pandas and SQL.

Examples of data analysis will be provide later in the form of Jupyter Notebooks.

### MLB Data

Retrosheet has play-by-play data for almost every Major League Baseball (MLB) game.  Lahman has MLB data summarized per year.

The Lahman data is tidy and has several csv files.  The latest available description of each csv file has been copied to the `data/lahman` directory.

Retrosheet is not distributed as csv files, but as text files which have play-by-play information.  This data must be parsed to create csv files.  The parsing will be done with open source parsers from Dr. T. L. Turocy described below.  The description of the generated csv files can itself be generated from the parsers.  These descriptions have been generated and copied to the `data/retrosheet` directory.

As of December 2019, Lahman has data through the 2018 season whereas Retrosheet has data through the 2019 season.

### Field Names

The field names in both datasets are based on standard baseball statistic abbreviations.  See for example: https://en.wikipedia.org/wiki/Baseball_statistics

The field names in both datasets will be converted from upper case to lower case.  Also Lahman's "gidp" will be renamed to "gdp" to match the abbreviation used in Retrosheet.

Although the use of Postgres is optional, field names which require double quotes in Postgres will be renamed so that double quotes are not required.  Some Lahman fields which will be renamed include: 2b, 3b, year, last, first, name, rank, start and end.

### Data Wrangling

Non-standard parsing of dates and times will be performed.  

Pandas datatypes will be optimized to save space and more accurately describe the attribute.  For example, the number of hits in a game is always between 0 and 255, so a uint8 can be used rather than an int64.  Likewise, for integer fields with missing values, the new Pandas Int64 can be used instead of requiring a float datatype.  A similar discussion applies to Postgres column data types.

Datatype optimizations are persisted to disk for each corresponding csv file with the suffix `_types.csv`.

### Data Consistency

Much of the Lahman data can be derived from the Retrosheet data by summing it per year.  The results are not an exact match in part because Retrosheet is missing some games.  No games are missing from Retrosheet since 1974 and less than 1% of all games are missing since 1955.

Data consistency unit tests are provided to validate that the Retrosheet data aggregated per year is very close to the corresponding Lahman data.  Most data unit tests are for the period 1955 through 2018 (the last available data from Lahman).  The year 1955 was chosen as this is the first year in which both the National and American League recorded statistics for sacrifice flies, sacrifice bunts, and intentional walks.

### Statistics per Player Role

A baseball player may have several roles during the course of a game, such as batter, fielder, and pitcher.  Most attributes are unique to the role, but some are not.  The same player can hit a home run as a batter and allow a home run as a pitcher.

There are nine fielding roles.  For example, the same player could make a put-out while playing second base and later make a put-out while playing first base.

### Script Summary

All scripts have help.  For example: `python lahman_download.py --help`

* **lahman_download.py** -- downloads the Lahman data
* **retrosheet_download.py** -- downloads the Retrosheet data
* **lahman_wrangle.py** -- convert to lowercase fieldnames with underscores, custom parse dates, etc.
* **retrosheet_datadictionary.py** -- generate a description of the generated csv files from the parsers
* **retrosheet_parse.py** -- generate the csv files
* **retrosheet_wrangle.py** -- convert to lowercase fieldnames with underscores, custom parse time, etc.
* and more ...



## MLB Data Details

### Lahman

The most recent data will be downloaded from:  https://github.com/chadwickbureau/baseballdatabank/archive/master.zip

As per https://github.com/chadwickbureau/baseballdatabank readme.txt, regarding the **Lahman data license**:

```
This work is licensed under a Creative Commons Attribution-ShareAlike
3.0 Unported License.  For details see:
http://creativecommons.org/licenses/by-sa/3.0/
```

#### Lahman Data Dictionary

The most recent data dictionary is:  http://www.seanlahman.com/files/database/readme2017.txt  

This file is also copied to this repo at: `data/lahman/readme2017.txt`

#### Unmodified Lahman CSV Data Files

The csv files for Lahman are tidy.  They are in my repo at: `data/lahman/raw`

* **per player per year**
  * Batting.csv
  * Fielding.csv
  * Pitching.csv
* **per team per year**
  * Teams.csv
* **reference tables**
  * People.csv
  * and more ...

### Retrosheet

The play-by-play data will be downloaded from: http://www.retrosheet.org/events/{year}eve.zip

As per: https://www.retrosheet.org/notice.txt regarding the **Retrosheet data license**:

```
     The information used here was obtained free of
     charge from and is copyrighted by Retrosheet.  Interested
     parties may contact Retrosheet at "www.retrosheet.org".
```

#### Generated CSV Data Files

The csv files must be generated by parsing the play-by-play data files.  The parsed csv files are in my repo at: `data/retrosheet/parsed`.

The **cwdaily** parser generates statistics per player per game.  Attributes are prefixed by b for batter, p for pitcher and f_{pos} for fielder where pos is one of P, C, 1B, 2B, 3B, SS, LF, CF, RF.

The **cwgame** parser generates statistics per team per game.

#### Retrosheet Data Dictionary

The retrosheet_datadictionary.py script will generate a description of the output of the cwdaily and cwgame parsers.

This script has been run and the descriptions have been saved to:  `data/retrosheet/cwdaily_datadictionary.txt` and `data/retrosheet/cwgame_datadictionary.txt`

#### My Wrangled CSV Data Files

* **per player per game**
  * player_game.csv.gz
* **per team per game**
  * team_game.csv.gz
* **reference tables**
  * people.csv
  * and more ...

### Parsers for Retrosheet

The open source parsers created by Dr. T. L. Turocy will be used.

As per https://github.com/chadwickbureau/chadwick README, regarding the parser license:

```
This is Chadwick, a library and toolset for baseball play-by-play
and statistics.

Chadwick is Open Source software, distributed under the terms of the 
GNU General Public License (GPL).
```

Parser Description: http://chadwick.sourceforge.net/doc/cwtools.html  
Parser Executables and Source: https://sourceforge.net/projects/chadwick/  

At the time of this writing, version 0.7.2 is the latest version.  Executable versions of the parsers are available for Windows.  Source code is available for Linux and MacOS.

#### How to Build Parsers on Linux

If you do not already have a build environment:

1. sudo apt install gcc
2. sudo apt install build-essential

cd to the source directory:

1. ./configure
2. make
3. sudo make install

Result

1. The cw command line tools will be installed in /usr/local/bin.
2. The cw library will be installed in /usr/local/lib.

To allow the command line tools to find the shared libraries, add the following to your .bashrc and then: source .bashrc
`export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib`