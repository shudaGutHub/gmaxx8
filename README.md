# gmaxfeed

Python3 modules for downloading and maintaining files from gmax server to local machine.

## GmaxFeed.py
Old and and barely fit for purpose, I've only left it in in case someone's used the functions from it. Will probably remove in the near future.

## PostraceFeeds.py

### GmaxFeed class

Stores the path to each of the directories for the feeds as given from environment variables or passed at init, also checks for existence of GMAXLICENCE in environment variable, else stores in environment variables if a key is passed at init. Setting environment variables of the full path for all the directories is hands down the easiest way to use the GmaxFeed class from several different projects, as all files will be maintained just once, not duplicated in several places. Make sure you enter unique paths for each of the feeds as files are stored under the sharecode fname so you can't have points and sectionals files loose in the same directory. When environment variables are set, to use access the gmax urls all you have to do is run,

from gmaxfeed.PostraceFeeds import GmaxFeed;
gmax_feed = GmaxFeed()

and call methods from the gmax_feed instance.
Useful methods:

.get_racelist(date:str or datetime = None, new:bool = False, sharecode:str = None)

First check is the sharecode field, if given this overrides date. Pass a sharecode to get the metadata about this sharecode. 
Second check is the date field, if not given defaults to datetime.utcnow(), if the target racelist file age is within 6 days of the date of the races then the racelist is refreshed (for changes to Published field), else the local version is used. For all method the keyword parameter new=True can be passed to force new downloads.

.get_racelist_range(start_date:datetime or str = None, end_date:datetime or str = None, new:bool = False)

Calls all dates in the daterange start_date to end_date (inclusive), self.get_racelist(date = date, new = new) from a threadpool of max default 6 threads.

.get_points(sharecode:str, new:bool)

.get_sectionals(sharecode:str, new:bool = False)

.get_sectionals_history(sharecode:str, new:bool = False)

.get_obstacles(sharecode:str, new:bool = False)

.get_route(course_codes:str or int = None, new:bool = False)

For the above 5 methods, load the file for the sharecode from the directory, unless new is passed in which case a new file is downloaded from the gmax server. If the file doesn't exist in the directory a new one is downloaded anyway. For the benefit of threadpools and identifying what has been returned, the return format is a dict of for example, {'sc':'01202006151340', 'data':data}.
If the feed in the .py file isn't listed above or in the docs below then it's for internal use only.

.get_data(sharecodes:list, request:set = {'sectionals', 'sectionals-history', 'points', 'obstacles'}, new:bool = False, filter:RaceMetadata = None)

Get the data for the a passed list of sharecodes using the above functions. Returns the data for the options given, eg if only want points just pass request = {'points'}. Cached versions are default, else a new file is downloaded if a local one isn't available. This can be forced by passing new = True. A filter can be passed using the RaceMetadata class, default is None which is replaced with a Published = True filter so as not to request data from races which aren't published.

.update(start_date:datetime or str = None, end_date:datetime or str = None, request:set = {'sectionals', 'points'}, new:bool = False, filter:RaceMetadata = None)

Update the contents of the directories for the requested feeds in request, racelist always updated and doesn't have to be specified. Pass new=True to force new files for everything. As above a filter can be passed to specify only certain sharecodes to update. By default if the file exists a new one isn't downloaded (except racelist as per the age condition above in .get_racelist()), if wanting to replace a whole list of sharecodes ensure to pass new=True.

.load_all_sectionals(start_date:datetime = None, end_date:datetime = None, filter:RaceMetadata = None)

Load and dump the sectionals in the range into an xlsx for any manual viewing. To be improved.

### RaceMetadata class

Useful class for managing/filtering a large collection of race metadata.
Useful functions:

.set_filter(countries:list or set = None, courses:list or set = None, course_codes:list or set = None, published:bool = None, start_date:datetime or str = None, end_date:datetime or str = None)

Set the conditions for the instance filter. Calling with no parameters clears the filter.

.apply_filter(countries:list or set = None, courses:list or set = None, course_codes:list or set = None, published:bool = None, start_date:datetime or str = None, end_date:datetime or str = None)

You don't have to set_filter() before using apply filter, just pass the keywords to apply_filter() instead. The default behaviour is to use the details sotred using set_filter, but if this isn't set the value given to apply_filter is used, it's possible to mix the two if set_filter and apply_filter are called with different parameters which might produce unexpected results.

.import_data(data:list or dict = None, direc:str = None)

Add the races in data to the instance, takes either list of dicts, or dict mapping each sharecode -> race_metadata. If data is None and a directory is passed instead (path to racelist folder) this contents of the folder are iterated and imported can be called multiple times, for instance if you run it in the morning to gather all metadata in one place and then want to add metadata for new races that have appeared in the gmax racelist later that day.

.get_set(countries:bool = True, courses:bool = True, course_codes:bool = True)

Get a set of all possible values within the data attribute for the given fields, and return as dictionary of sets. Useful for passing "everything except" conditions to filter, eg, for all courses except Ascot Newcastle and Bath, obj.filter(courses = obj.get_set().get('courses') - {'Ascot', 'Newcastle', 'Bath'})

### TPDFeed class

Defines functions and houses directories/licence keys to access the TPD derivatives, par times for a race type in the ground/class/age, par attribute timelines, and other cool developments.
To be completed.

### Anecdotes and Specs

Any problems please let me know via email or comments. Currently when run as main it downloads all sectionals and gps points files, but if you're not set up for one then remove it from the requests set or you'll end up with a folder full of empty braces. Daterange should also be set to your permissable daterange fo the same reason as RaceLists will be stored for each day which will just be an empty list if you're not activated for the date in query.

Multi threaded to max of 6 threads for high speed - if you're going to download a back history please do so overnight or in the morning so as not to potentially add to the latency of the live races on the day.

Requires an active licence key obtained by purchase from Total Performance Data, http://www.totalperformancedata.com/ .

Feed Specifications below:
- GPS Points: https://www.gmaxequine.com/downloads/GX-RP-00038%202018-10-12%20Gmax%20Race%20Point%20Data%20Feed%20Specification.pdf
- Sectionals: https://www.gmaxequine.com/downloads/GX-RP-00019%202019-02-15%20Gmax%20Race%20Post-Race%20Protocol%20Specification.pdf
- Daily Race List: https://www.gmaxequine.com/downloads/GX-RP-00054%202020-06-03%20Gmax%20Race%20List%20Feed%20Specification.pdf
- Sectionals History: https://www.gmaxequine.com/downloads/GX-RP-00047%202018-12-14%20Gmax%20Race%20Sectionals%20History%20Feed%20Specification.pdf
- Racecourse Surveys: https://www.gmaxequine.com/downloads/GX-RP-00039%202018-02-12%20Gmax%20Race%20Route%20Data%20Feed%20Specification.pdf
- Jumps Locations (Hurdles coming Soon): https://www.gmaxequine.com/downloads/GX-RP-00055%202019-06-05%20Gmax%20Race%20Jumps%20Feed%20Specification.pdf

Live data streams also available are:
- Live GPS Points: https://www.gmaxequine.com/downloads/GX-RP-00007%202018-08-08%20Gmax%20Race%20Live%20Data%20Feed%20Specification.pdf
- Live Progress: https://www.gmaxequine.com/downloads/GX-RP-00020%202019-02-15%20Gmax%20Race%20Live%20Progress%20Feed%20Specification.pdf

Limitations:
- Runners which don't finish are usually not included due to current limitations in the software setup,
- For some courses/distances the 'P' field doesn't decrease, this is a problem in the historic feed mapping to the route files but is usually not an issue in the live feed. We're working on fixing the issue.
- Whip strikes can cause sudden spikes in the data, velocities hitting near 50m/s and skewing the X,Y way off the track, there's nothing we can do about this as the trackers are padded as much as the weight/size guidelines will allow.
- We cannot distribute horse names due to licence limitations, we can only supply the sharecode with saddle cloth number appended.
- Occaisionally the trackers suffer errors, usually due to very hard and direct whip strikes/kicks which disable them for some period of time and sometimes due to operator error. If the data cannot be recovered to an acceptable standard then the race is not published and there are no sectionals or points available for any of the runners. 

## RecordLiveTPD.py

RecordLiveTPD is a simple program to listen for incoming live data and save them to a file system based on the sharecodes within the packets. Change directory to gmaxfeed, then run "python3 RecordLiveTPD.py", this can be tested using Packet Sender app by sending packets to 127.0.0.1:4629, then exited by input of 't' (for terminate). An example test packet from the Progress feed to use in packet sender is below:
{"K":5,"T":"2020-03-15T21:02:03.0Z","I":"00202003151552","G":"4.5f","S":5.35,"C":16.49,"R":17.96,"V":18.5,"P":877.9,"O":["5","3","6","2","4","7"],"F":["4","7","5","3","2","6"],"B":[0,6.1,13.7,14.6,16.8,19.5]}

