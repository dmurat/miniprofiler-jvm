MiniProfiler-JVM
================

[![Build Status](https://travis-ci.org/tomdcc/miniprofiler-jvm.png)][1]

This library provides (some of) the functionality of the StackExchange [.NET MiniProfiler][2] for JVM-based applications.

(Screenshot)

Installation
------------
The JVM MiniProfiler is deployed to Maven Central and can be added to your project as a dependency using the following coordinates:

    groupId: io.jdev.miniprofiler
    artifactId: miniprofiler-core
    version: 0.3

Or just download the jar from http://search.maven.org/ if your build system is a bit less connected.

Basic Usage
-----------
Once installed, start using the profiler thusly:

    Profiler profiler = MiniProfiler.start("/url/my web request");
    try {
        // do some stuff
    } finally {
        profiler.stop();
    }

To profile individual parts of your program, create and stop individual timings:

    Timing t = MiniProfiler.getCurrentProfiler().step("My complicated step");
    try {
        // stuff
    } finally {
        timing.stop();
    }

If you are using Java 7, you can use Java 7 auto-closable resource blocks for even cleaner code:

    try (Timing t = MiniProfiler.getCurrentProfiler().step("My complicated step")) {
        // stuff
    }

Steps are nestable, and will appear nested in profiler output:

    try (Timing t = MiniProfiler.getCurrentProfiler().step("My complicated step")) {
        // stuff
	    try (Timing t1 = MiniProfiler.getCurrentProfiler().step("A sub-part of the complicated step")) {
	        // sub-stuff 1
	    }
	    try (Timing t2 = MiniProfiler.getCurrentProfiler().step("A second, sibling sub-part of the complicated step")) {
	        // sub-stuff 2
	    }
    }

Usage with Dependency Injection
-------------------------------
The default code above uses a static reference to a global ProfilerProvider object. If you are using a dependency injection framework such as Spring or Guice, or if you just like more testable code, then create a ProfilerProvider and inject it into your code. The included DefaultProfilerProvider should be enough for most purposes. Then use it in your code like this:

    // at start of request
    Profiler profiler = profilerProvider.start("/url/my web request");
    try {
        // do some stuff
    } finally {
        profiler.stop();
    }


    // further in where stuff is happening
    Timing t = profilerProvider.getCurrentProfiler().step("My complicated step");
    try {
        // stuff
    } finally {
        timing.stop();
    }


Profiling JDBC Queries
----------------------
To see your SQL queries in your profiling output, just wrap your JDBC DataSource in the ProfilingDataSource, and call
getConnection() on that datasource as normal.

Seeing the output
-----------------
To see the output of the profiled request on your web page, add a script tag to the bottom of your HTML page, just inside the body tag. The library comes with a `ScriptTagWriter` class to help with this:

    <%= new ScriptTagWriter().printScriptTag(MiniProfiler.getCurrentProfiler(), request.getContextPath() + "/miniprofiler")%>

This will output a javascript script tag which will load the necessary javascript, css and data from under `/miniprofiler` in your web app.

Inclusion in Servlet Web App
----------------------------
The core library includes a `ProfilingFilter` which does several jobs:

 - Start and stop profiling for all web requests, excluding anything that looks like a file to be served up directly
 - Serve up front-end javascript, CSS and templates used for displaying the profiling info in a web page
 - Serve up profiling data as JSON

The filter can be included in your web.xml like this:

	<filter>
		<filter-name>miniprofiler</filter-name>
		<filter-class>io.jdev.miniprofiler.servlet.ProfilingFilter</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>miniprofiler</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

Currently the filter is not very configurable - it is hardcoded to expect requests for the static resources and data under `/miniprofiler` in your web app. This and other behaviour will be made configurable in the future.

Feedback / Contributions
------------------------
Please raise issues in the [GitHub issue tracker][3]

[1]:https://travis-ci.org/tomdcc/miniprofiler-jvm
[2]:http://miniprofiler.com/
[3]:https://github.com/tomdcc/miniprofiler-jvm/issues

Fork notes
----------
- This `miniprofiler-jvm` version (0.3.1) is my fork of main development stream with few enhancements (see changelog). I have use it in production as underlying implementation for
  `grails-miniprofiler` v0.3.1 (also my fork) plugin (available at https://github.com/dmurat/grails-miniprofiler/tree/myModifications).
- This fork is intended to work with grails 2.3.x. For later grails versions (i.e. 2.4.x) please look at latest version at original project github page - https://github.com/tomdcc/miniprofiler-jvm
- Source code for this fork is available at https://github.com/dmurat/miniprofiler-jvm/tree/myModifications (all changes are in myModification branch).
- Released version for this fork is available in maven compatible repository at https://github.com/dmurat/mvn-repo/raw/master/releases/, so make sure that you have it included in yours build
  configuration.
- Most probably, this is last version which will be released. However, there might be a version which will be a fork for 0.4.1 version (or later) of original project. Currently I don't have any plans
  to maintain it further.
- I have submitted pull requests of all my changes to the original project. However, author ignored them and did not included them in original source tree. So I was somewhat forced to publish this
  fork with primary intention to make my life somewhat easier.

Changelog
---------
### 0.3.1 - forked
  - exposing various GUI configuration options to be able to configure them from miniprofiler's GSP tag
  - increasing max width of miniprofiler's popup to be able to see longer urls
  - few JavaScript fixes to improve stability
