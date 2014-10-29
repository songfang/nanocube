# Nanocubes: an in-memory data structure for spatiotemporal data cubes

Nanocubes are a fast data structure for in-memory data cubes developed
at the
[Information Visualization department](http://www.research.att.com/infovis)
at [AT&T Labs – Research](http://www.research.att.com). Visualizations
powered by nanocubes can be used to explore datasets with billions of
elements at interactive rates in a web browser, and in some cases nanocubes
uses sufficiently little memory that you can run a nanocube in a
modern-day laptop.

## Releases

| Number | Description |
|:------:|-------------|
| 3.0.2 | Renamed existing tools, added new tools, standarized tool names, bug fixes |
| 3.0.1 | Fixed a bug that was causing memory inneficiencies |
| 3.0 | [New API](https://github.com/laurolins/nanocube/blob/api3/API.md) to support more general nanocubes |
| 2.1.3 | Minor fixes, improved csv2Nanocube.py script  |
| 2.1.2 | Minor fixes, better documentation, shutdown service |
| 2.1.1 | Fixed csv2Nanocube.py to work with pandas 0.14.0 |
| 2.1 | Javascript front-end, CSV Loading, Bug Fixes  |
| 2.0 | New feature-rich querying API                  |
| 1.0 | Original release with a simple querying API   |

## Installing prerequisites

At the moment, the list of prerequisites for nanocubes is this one

1. The nanocubes server is 64-bit only.  There is NO support on 32-bit operating systems.
2. The nanocubes server is written in C++ 11.  You must use a recent version of gcc (>= 4.8).
3. The nanocubes server uses [Boost](http://www.boost.org).  You must use version 1.48 or later.
4. To build the nanocubes server, you must have the [GNU build system](http://www.gnu.org/software/autoconf/) installed.

#### Linux (Ubuntu)

On a newly installed 64-bit Ubuntu 14.04 system, gcc/g++ is already 4.8.2, but you may have to install the following packages:

    sudo apt-get install build-essential
    sudo apt-get install automake
    sudo apt-get install libtool
    sudo apt-get install zlib1g-dev
    sudo apt-get install libboost-all-dev

#### Mac OS X (10.9)

Example installation on Mac OS 10.9 Maverick with a local homebrew:

	git clone https://github.com/mxcl/homebrew.git

Set your path to use this local homebrew				

	export PATH=${PWD}/homebrew/bin:${PATH}

Install the packages (This assumes your g++ has been installed by [XCode](https://developer.apple.com/xcode/))

	brew install boost libtool autoconf automake

Set path to the boost directory

	export BOOST_ROOT=${PWD}/homebrew

## Compiling the latest release

To compile the nanocube toolkit, tun the following commands on your
linux/mac system (you can replace `3.0.2` with other valid release
numbers, e.g. 3.0.1, 3.0, 2.1.3, 2.1, 2.0).

    wget https://github.com/laurolins/nanocube/archive/3.0.2.zip
    unzip 3.0.2.zip
    cd nanocube-3.0.2
    export NANOCUBE_SRC=`pwd`
    ./bootstrap
    mkdir build
    cd build
    ../configure --prefix=$NANOCUBE_SRC
    make -j
    make install

After these commands you should have directory `nanocube-3.0.2/bin` with the
nanocube toolkit inside. To make these tools available you can export them
to environment variables

    export NANOCUBE_BIN=$NANOCUBE_SRC/bin
    export PATH=$NANOCUBE_BIN:$PATH

If a recent version of gcc is not the default, you can run `configure`
with the specfic recent version of gcc in your system. For example
`CXX=g++-4.8 ../configure --prefix=$NANOCUBE_ROOT`. **(Hint)** For better
performance you might configure nanocubes with the option a tcmalloc
option: `../configure --prefix=$NANOCUBE_ROOT --with-tcmalloc`.

## Running a nanocube

With the nanocube toolkit installed, we are ready to start a first
nanocube with Chicago Crimes example dataset file included in the
distribution: `$NANOCUBE_ROOT/data/crime50k.dmp`. Here is the command:

    cat $NANOCUBE_SRC/data/crime50k.dmp | nanocube-leaf -q 29512

The command above simply says to start a nanocube back-end process
from the `crime50k.dmp` data file

    VERSION: 3.0.2
    query-port: 29512
    (stdin:done) count:      50000 mem. res:         17MB. time(s):          0

## Simple queries

With a nanocube process running we are able to query this nanocube
using [API](https://github.com/laurolins/nanocube/blob/api3/API.md). For
example by putting the URL `http://localhost:29512/count` we get the
following json object with the total count of records:

    { "layers":[  ], "root":{ "val":50000 } }

Or we can get the schema of the nanocube by querying the URL `http://localhost:29512/schema`

    { "fields":[ { "name":"src", "type":"nc_dim_quadtree_25", "valnames":{  } }, { "name":"crime", "type":"nc_dim_cat_1", "valnames":{ "CRIM_SEXUAL_ASSAULT":7, "WEAPONS_VIOLATION":30, "KIDNAPPING":13, "OFFENSE_INVOLVING_CHILDREN":20, "CONCEALED_CARRY_LICENSE_VIOLATION":4, "SEX_OFFENSE":27, "INTIMIDATION":12, "PROSTITUTION":23, "ARSON":0, "BURGLARY":3, "ROBBERY":26, "CRIMINAL_TRESPASS":6, "THEFT":29, "HOMICIDE":10, "OBSCENITY":19, "OTHER_NARCOTIC_VIOLATION":21, "MOTOR_VEHICLE_THEFT":15, "GAMBLING":9, "NARCOTICS":16, "NON-CRIMINAL_(SUBJECT_SPECIFIED)":18, "DECEPTIVE_PRACTICE":8, "STALKING":28, "CRIMINAL_DAMAGE":5, "PUBLIC_PEACE_VIOLATION":25, "BATTERY":2, "ASSAULT":1, "PUBLIC_INDECENCY":24, "NON-CRIMINAL":17, "LIQUOR_LAW_VIOLATION":14, "OTHER_OFFENSE":22, "INTERFERENCE_WITH_PUBLIC_OFFICER":11 } }, { "name":"time", "type":"nc_dim_time_2", "valnames":{  } }, { "name":"count", "type":"nc_var_uint_4", "valnames":{  } } ], "metadata":[ { "key":"tbin", "value":"2013-12-01_00:00:00_3600s" }, { "key":"src__origin", "value":"degrees_mercator_quadtree25" }, { "key":"name", "value":"crime50k.csv" } ] }

If we want to breakdown the counts per `crime` type we can simply query the URL: `http://localhost:29512/count.a("crime",dive([],1))`

    { "layers":[ "anchor:crime" ], "root":{ "children":[ { "path":[0], "val":63 }, { "path":[1], "val":2629 }, { "path":[2], "val":8990 }, { "path":[3], "val":2933 }, { "path":[4], "val":1 }, { "path":[5], "val":4660 }, { "path":[6], "val":1429 }, { "path":[7], "val":181 }, { "path":[8], "val":2190 }, { "path":[9], "val":2 }, { "path":[10], "val":69 }, { "path":[11], "val":229 }, { "path":[12], "val":21 }, { "path":[13], "val":46 }, { "path":[14], "val":69 }, { "path":[15], "val":2226 }, { "path":[16], "val":5742 }, { "path":[17], "val":1 }, { "path":[18], "val":1 }, { "path":[19], "val":1 }, { "path":[20], "val":456 }, { "path":[21], "val":2 }, { "path":[22], "val":3278 }, { "path":[23], "val":211 }, { "path":[24], "val":2 }, { "path":[25], "val":441 }, { "path":[26], "val":2132 }, { "path":[27], "val":119 }, { "path":[28], "val":20 }, { "path":[29], "val":11367 }, { "path":[30], "val":489 } ] } }


## Auxiliar Tools in Python

Nanocube's distribution comes with a set of tools that are Python
scripts:

| Tool Name | Description |
|:------:|-------------|
| nanocube-binning-csv | Convert a `.csv` file into a `.dmp` file readable by the nanocube-leaf program |
| nanocube-benchmark | From a text file with one query per row, generate a report of latency and sizes  <br> by running those queries on a given nanocube server |
| nanocube-view-dmp | Show records of a `.dmp` file on the command line |
| nanocube-monitor | Feedback of latency and size distribution for nanocube profiling |

These python scripts depend on some python modules that are not
standard `pandas`, `numpy`, `argparse`. Here is a recipe to install
a local python environment (in case you don't have these modules already)
and have the python toolkit of nanocubes ready to go.

1. Go to the `$NANOCUBE_SRC`

        cd `$NANOCUBE_SRC`

2. For compiling our python helper code, you will need the following packages:

        sudo apt-get install python-dev

3. Install the python data analysis library (pandas) in a separate python environment (Recommended)

        wget http://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.11.6.tar.gz
        tar xfz virtualenv-1.11.6.tar.gz
        python virtualenv-1.11.6/virtualenv.py  myPy
        
        # Make sure PYTHONHOME and PYTHONPATH are unset
        unset PYTHONHOME
        unset PYTHONPATH

        # activate the virtualenv, type "deactivate" to disable the env when done
        source myPy/bin/activate
        pip install pandas numpy argparse

Now you should have a local python environment compatible with the
nanocube's python toolkit.

## How to prepare a .csv file to be loaded into a nanocube?

Nanocube is able to ingest only a special kind of file. From a high
level perspective these files are simply tables. The file starts with
a header describing the `columns` of the table followed by the
records.  We call this file format `.dmp` files. To make things even
more complicated, not all tables in `.dmp` format are nanocube ready
data files. To be a nanocube ready `.dmp` file the column types of the
records need to be of a special kind. We discuss these issues later,
but for now, let's see how to go from a `.csv` file to a `.dmp` one
that is nanocube-ready.

    cd $NANOCUBE_SRC/data
    nanocube-binning-csv --sep=',' --timecol='time' --latcol='Latitude' --loncol='Longitude' --catcol='crime' --port=29512 $NANOCUBE/src/crime50k.csv > crime50k_from_csv.dmp

The file generated with the command above is the same file
`$NANOCUBE_SRC/data/crime50k.dmp` that goes in the distribution of
nanocubes.

## (extra) `nc_web_viewer`, a nanocube viewer in js, d3, and html5

To visualize a nanocube that has one spatial dimension, zero or more
categorical dimensions and one temporal dimension you can use a
javascrit/d3/html5 viewer that sits on
`$NANOCUBE_SRC/extra/nc_web_viewer`. We can start the `nc_web_viewer`
by running

    cd $NANOCUBE_SRC/extra/nc_web_viewer
    python -m SimpleHTTPServer 8000

So by pointing a browser to the URL `http://localhost:8000` we can
get to the `nc_web_viewer`, but there is one catch. How does the
`nc_web_viewer` knows where a nanocube server sits? To specify where
one or more nanocube processes are hosted we need to genereate
one or more `nc_web_viewer` specific `.json` configuration files and
copy those to the `$NANOCUBE_SRC/extra/nc_web_viewer` folder.

# TODO: cleanup comments below

### tcmalloc

We strongly suggest linking nanocubes with
[Thread-Caching Malloc](http://goog-perftools.sourceforge.net/doc/tcmalloc.html),
or tcmalloc for short.  It is faster than the default system malloc,
and in some cases, we found that the amount of memory used by
nanocubes was reduced by over 50% when using libtcmalloc.  To install
on a Ubuntu 14.04 machine, install the following package and all of
its dependencies.

    sudo apt-get install libgoogle-perftools-dev

You must then re-run the configure script, indicating support for tcmalloc.

    ../configure --prefix=`pwd`/..
    make clean
    make
    make install


## Loading a CSV file into a nanocube

1. For compiling our python helper code, you will need the following packages:

        sudo apt-get install python-dev

2. Install the python data analysis library (pandas) in a separate python environment (Recommended)

        wget http://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.11.6.tar.gz
        tar xfz virtualenv-1.11.6.tar.gz
        python virtualenv-1.11.6/virtualenv.py  myPy
        
        # Make sure PYTHONHOME and PYTHONPATH are unset
        unset PYTHONHOME
        unset PYTHONPATH

        # activate the virtualenv, type "deactivate" to disable the env when done
        source myPy/bin/activate
        pip install pandas numpy argparse

3. Start a web server in the "web" directory and send it to background.  If port 8000 is already being used
on your system, please choose another port.

        cd web
        python -m SimpleHTTPServer 8000 &

4. Run the script and pipe it to the nanocubes server using the
   included example dataset
   ([Chicago Crime](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-present/ijzp-q8t2)).
   If port 29512 is already being used on your system, please choose
   another port.  Note that the port is specified for both the python
   script and for the nanocubes server (nanocube-leaf). If these are
   not the same, you'll run into problems.

        cd ../scripts
        nanocube-binning-csv --sep=',' --timecol='time' --latcol='Latitude' --loncol='Longitude' --catcol='crime' --port=29512 crime50k.csv | nanocube-leaf --query-port 29512 --report-frequency 10000 --threads 100

   Please note: We modified the original dataset slightly in this
   example by changing the names of two columns in the header.  In
   previous versions, there were columns called 'Date' and 'Primary
   Type'.  We have renamed these 'time' and 'crime' to avoid any
   confusion and make the subsequent examples easier to read.

   The first few lines of the example dataset are shown below. The
   first line is a header, which describes each of the columns in this
   table of data.  You should notice that there are columns called:
   time, crime, Latitude, Longitude.  These are the columns used for
   this visualization.

        ID,Case Number,time,Block,IUCR,crime,Description,Location Description,Arrest,Domestic,Beat,District,Ward,Community Area,FBI Code,X Coordinate,Y Coordinate,Year,Updated On,Latitude,Longitude,Location
        9418031,HW561348,12/06/2013 06:25:00 PM,040XX W WILCOX ST,2024,NARCOTICS,POSS: HEROIN(WHITE),SIDEWALK,true,false,1115,011,28,26,18,1149444,1899069,2013,12/11/2013 12:40:36 AM,41.8789661034259,-87.72673345412568,"(41.8789661034259, -87.72673345412568)"
        9418090,HW561429,12/06/2013 06:26:00 PM,026XX W LITHUANIAN PLAZA CT,1310,CRIMINAL DAMAGE,TO PROPERTY,GROCERY FOOD STORE,false,false,0831,008,15,66,14,1160196,1858843,2013,12/10/2013 12:39:15 AM,41.76836587673295,-87.68836274472295,"(41.76836587673295, -87.68836277 4472295)"
        9418107,HW561399,12/06/2013 06:29:00 PM,045XX S DAMEN AVE,0860,THEFT,RETAIL THEFT,DEPARTMENT STORE,true,false,0924,009,12,61,06,1163636,1874247,2013,12/10/2013 12:39:15 AM,41.810564946613454,-87.6753212816967,"(41.810564946613454, -87.6753212816967)"
        9418222,HW561455,12/06/2013 06:30:00 PM,008XX W ALDINE AVE,0820,THEFT,$500 AND UNDER,RESIDENCE,true,false,1924,019,44,6,06,1169898,1922154,2013,12/14/2013 12:39:48 AM,41.94189139041086,-87.65095594008946,"(41.94189139041086, -87.65095594008946)"

   The parameters for csv2Nanocube.py are listed below.  Note that
   when we called the script above, we specified the categorical
   dimension (crime) and the time dimension (time), as well as
   Latitude and Longitude.  However, the script is smart enough to
   identify the Latitude and Longitude columns automatically if they
   have these names.  If they were named differently (e.g. lat, long),
   we would have to use the other parameters for the script (--latcol,
   --loncol) to identify them for the script.  If your data is also
   separated by a character other than a comma, you can indicate this
   when you run the script using the '--sep' parameter.

        --catcol='Column names of categorical variable'
        --latcol='Column names of latitude'
        --loncol='Column names of longitude'
        --countcol='Column names of the count'
        --timecol='Column names of the time variable'
        --timebinsize='time bin size in seconds(s) minutes(m) hours(h) days(D)'
        --port='Port of the nanocubes server'
        --sep='Delimiter of Columns'
        e.g. 1D/30m/60s'

   The output generated by running the csv2Nanocube.py script should
   look like the following.  You can see that 50,000 points were
   inserted into the nanocube, which is using 18MB of RAM.

        VERSION: 2014.10.11_16:40
        parent process is waiting...
        started redirecting stdin content to write-channel of parent-child pipe
        query-port:  29512
        insert-port: 0
        (stdin     ) count:      10000 mem. res:          6MB. time(s):          0
        (stdin     ) count:      20000 mem. res:          9MB. time(s):          1
        (stdin     ) count:      30000 mem. res:         12MB. time(s):          2
        (stdin     ) count:      40000 mem. res:         15MB. time(s):          2
        (stdin     ) count:      50000 mem. res:         18MB. time(s):          3
        (stdin:done) count:      50000 mem. res:         18MB. time(s):          3

5. That's it.  Point your browser (Firefox, Chrome, Safari) to http://localhost:8000 for the viewer. If you needed to change the port number in Step 3 above, make sure that you specify the same number here.

6. If you believe there may be a problem, try running 'nctest.sh' in the scripts subdirectory.  It will make some queries of the nanocube (change the script if you are not using port 29512) and compare the results to known results that we gathered ourselves.  If the results match, it will report 'SUCCESS'.

7. When finished, terminate the nanocube (e.g. Control-C) and then type 'deactivate' on the command-line to shut the virtual python environment down.

For this example we assume you are running everything on your
localhost. Modify `config.json` accordingly in the `web` folder for
different setups.

### Subsequent Runs

Running this example again later, you do not need to reinstall the linux or python packages.

        cd nanocube-3.0.2
        source myPy/bin/activate
        cd web
        python -m SimpleHTTPServer 8000 &
        cd ../data
        nanocube-binning-csv --sep=',' --timecol='time' --latcol='Latitude' --loncol='Longitude' --catcol='crime' --port=29512 crime50k.csv | nanocube-leaf --batch-size 1 --query-port 29512 --report-frequency 10000 --threads 100


## Further Details

For a better understanding on how to ingest data into nanocubes and
how to query nanocubes follow this
[link](https://github.com/laurolins/nanocube/wiki). For larger
datasets or if you want more flexibility on ingesting/querying data
using nanocubes the CSV loading method illustrated above might not be
the most efficient way to go.

## Asking for help

Our mailing list is the best and fastest way to ask questions and make suggestions related to nanocubes.
If you are having a problem, please search the archives before creating new topics to see if
your question has already been answered.  If you have other ideas for how we can improve
nanocubes, please let us know.

A nice front-end for our mailing list is now being served through [Nabble](http://nanocubes-discuss.64146.x6.nabble.com).
You should be able to post messages, search the archives, and even register as a new user from here.

The actual mailing list can be found [here](http://mailman.nanocubes.net/mailman/listinfo/nanocubes-discuss_mailman.nanocubes.net).
