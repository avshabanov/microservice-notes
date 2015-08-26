
## Garbage Collector Types

### Serial

OK for single-threaded applications, CLI tools and small heaps.

JVM switch: ``-XX:+UseSerialGC``.

### Parallel

Uses multiple threads to scan and compact the heap. Good for applications that can tolerate pauses and should have small CPU overhead caused by GC (preferrably stop before GC will trigger).

### CMS or Concurrent Mark-and-Sweep

Good choice as server GC for heaps less than 4 Gigabytes.

JVM switch: ``-XX:+UseConcMarkSweepGC``.

### G1 or Garbage-First

Available in Java 7 and Java 8.

Splits heaps into multiple regions and focuses on the most 'polluted' regions or regions with the most garbage objects.

Compacts heap on the go as opposed to CMS and Parallel GCs.

Java 8 version comes with optional ability to deduplicate strings. It can be turned on by providing ``-XX:+UseStringDeduplication``.

Good for large heaps more than 6 Gigabytes.

Quote from Oracle documentation:

```
The first focus of G1 is to provide a solution for users running applications that require large heaps with limited GC latency. This means heap sizes of around 6GB or larger, and stable and predictable pause time below 0.5 seconds.

Applications running today with either the CMS or the ParallelOldGC garbage collector would benefit switching to G1 if the application has one or more of the following traits.

Full GC durations are too long or too frequent.
The rate of object allocation rate or promotion varies significantly.
Undesired long garbage collection or compaction pauses (longer than 0.5 to 1 second)
Note: If you are using CMS or ParallelOldGC and your application is not experiencing long garbage collection pauses, it is fine to stay with your current collector. Changing to the G1 collector is not a requirement for using the latest JDK.
```

## Most used heap keys

See also http://www.oracle.com/technetwork/articles/java/vmoptions-jsp-140102.html and http://www.cubrid.org/blog/dev-platform/how-to-tune-java-garbage-collection/

Heap Size Control:

* -Xms500m - sets up initial heap size to 500 MB
* -Xmx2g - sets up maximum heap size to 2 GB

New Area Size:

* -XX:NewRatio=4 - sets ratio between the old and young generation to 1:4
* -XX:NewSize=250m - sets size of young generation to 250 MB
* -XX:MaxNewSize=500m - sets maximum size of young generation to 500 MB
* -XX:SurvivorRatio - sets ratio of eden area/survivor area

Stack Size:

* -Xss=1m - set stack size to 1MB

note, that you should not tune stack size unless you have *VERY* good reason to do so.

GC logging:

* -verbose:gc - enables garbage collection logging
* -Xloggc:/path/to/your/gc.log - redirects logging to the file ``/path/to/your/gc.log``
* -XX:+PrintGCDetails - enables detailed GC output.
* -XX:+PrintGCTimeStamps or -XX:+PrintGCDateStamps - adds time-/date- stamps to each log record.
* -XX:+PrintGCApplicationStoppedTime - enables printing GC timings when stop-the-world garbage collection occurs.
* -XX:+PrintGCApplicationConcurrentTime - enables printing application time.
* -XX:+PrintTenuringDistribution - show ages of objects in survivor spaces.
* And even more finer grained (usually not needed):
** -XX:+PrintSafepointStatistics - prints all safepoints details
** -XX:+PrintAdaptiveSizePolicy - shows statistics to calculate heap generations sizes

Sample application parameters:

```
java
-XX:+UseConcMarkSweepGC
-Xms100m -Xmx800m 
-verbose:gc -Xloggc:/opt/HelloWorldGC.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintTenuringDistribution
-classpath deployment/target/classes
com.mycompany.HelloWorldApp
```

## How to tune GC for JVM?

When your application is written in a way so that it does not have leaking memory you can think of tuning your JVM.

At first, you can try simplest measures:

* Make sure you have enough free space. GC performance starts degrading when free/total heap memory ratio is low, say less than 40%. It starts dramatically degrading at around 10% and becomes worse and worse when application has less memory.
* Make sure you're utilizing aggressive cleanups of young generation. GC collects objects in young generation first and understanding why it promotes objects from young space to survivor space and then to old generation may help to understand what needs to be fixed first to improve GC performance by utilizing young generation at its maximum.
* Know maximum load your application can handle while staying stable and not consuming too much memory. Once you know this number start throttling excessive load so that spillover usage won't affect your service too much.
* When it comes to finer tuning understand what GC goal you want to target:
** Throughput
** Stop-the-world pause length
** Memory footprint



