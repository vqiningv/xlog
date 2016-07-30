## xlog - Enhanced glog for Go ##

This project is an enhanced/adapted version of the google [glog](https://github.com/golang/glog), which is the Go implementation of leveled logs in the manner of the open source C++ package.

Since the glog repo is only for export and is not itself under development, this repo doesn't fork glog.

Feel free to contact me and any feedback is welcome: **vqiningv@yeah.net**

###Features Comparison

| Feature | xlog | glog |
| --- | --- | --- |
| log severities (low->high) | DEBUG/INFO/WARNING/ERROR/FATAL | INFO/WARNING/ERROR/FATAL |
| log rotation support | by size & date | only by size |
| log level setting support(to filter out low level logs) | yes | no |
| high severity logs also go to low severity logs| no | yes |


###IMPORTANT NOTICE

- xlog adopts a rotation policy which combines both size and date based rotation, i.e. each time before a log message is "actually" written, xlog will check the current log file size and the current date(the day when you write log message), if log file size exceeds a predefined threshhold, or no log file is created yet on the current day, xlog will trigger log rotation(flush all the possible previous log msg to disk, create new log file, initialize it with some header content and append the real user log msg).

- Log rotation checking is based on logging "action". This means that xlog will check if it needs to trigger rotation only when each time you write a log message, not when the clock "ticks". So, on some circumstances, if you haven't written any log messages for a pretty long time, you would only see log files created that "pretty long time" before(rotation won't be checked hence new log files won't be created).

- xlog defines 5 logging severities("DEBUG" is added to the original glog setup to aid debugging), from low to high severity: DEBUG/INFO/WARNING/ERROR/FATAL. Note that the "DEBUG" severity is only for debugging and should not be used in production environment. Usually, "INFO" and "ERROR" log would be heavily used, but of course this depends on your logging policy.

- xlog provides support for log level setting by the "log_above" flag, e.g. with `-log_above=INFO`, which is also the default setting, xlog will only output log messages that have severities equal or higher than "INFO", and "DEBUG" severity messages will be abandoned. This is useful when you want to switch log settings between development and production environments(through startup script etc.) while still keeping your source code unchanged.

- When your app starts to log, xlog will only create log files with severities equal or higher than the log level(specified by the "log_above" flag, and if absent, defaults to "INFO"), from the current log severity "down" to the threshhold. For instance, with `-log_above=INFO`, `xlog.Infoln("App starts...")` will only create "INFO" log file while `xlog.Errorln("Crap...")` will create "ERROR" log file and "WARNING" log file in order("INFO" log file will not be created since it's already created earlier, given all the logging actions happen on the same day).


###Usage

Using xlog won't be much different than doing glog, just write `xlog.xxx()` instead of `glog.xxx()` and remember to provide the necessary flag values like `-log_dir="/var/logs"` (this specifies the directory to output logs to).

Here is an example flag value setup: 

	-log_dir="/var/logs" -log_above="INFO"

(as mentioned earlier, `-log_above="INFO"` is not necessary since it is default)

Then, in your code:
	
	package main
	
	import (
	  "flag"
	  "github.com/vqiningv/xlog"
	)
	
	func main() {
		//make sure all logs are flushed to disk before exit
		defer xlog.Flush()
	
		//parse command params first(xlog needs this)
		flag.Parse()
	
		xlog.Infoln("App starts...")
		xlog.Errorln("My icecream fell on the ground, crap...")
	

###Other

All the other features not mentioned are just preserved from glog, for example: 

By binding methods to booleans it is possible to use the log package without paying the expense of evaluating the arguments to the log.

Through the -vmodule flag, the package also provides fine-grained control over logging at the file level.
	
See the documentation for the V function for an explanation	of these examples:
	
		if xlog.V(2) {
			xlog.Info("Starting transaction...")
		}
	
		xlog.V(2).Infoln("Processed", nItems, "elements")


###TODO

- Appending logs to previously created file when app restarts(we are still on the same day)?  Since frequent app restart is only a common situation in development/debug phase, this requirement seems not that urgent or even necessary...

- Leaving users the freedom to define their own log format(just like other logging tools such as logback, log4j)? The current setup is already good and clear enough, but I'll see what I can do...