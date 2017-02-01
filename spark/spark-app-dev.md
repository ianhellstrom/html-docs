name: inverse
layout: true
class: center, middle, inverse
---
# Application Development with Apache Spark
A brief tutorial for beginners

[Ian Hellstrom](http://databaseline.wordpress.com)
---
## Where do I start?
---
layout: false
.left-column[
  ## What is Spark?
]
.right-column[
Apache Spark is an

- open source

- cluster computing framework

- written in Scala

- that can run independently or on

    - [Hadoop YARN](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)
    - [Apache Mesos](http://mesos.apache.org)

- and supports *multi-stage in-memory* computations
]
---
.left-column[
  ## What is Spark?
  ## Cool, but so what?!
]
.right-column[
[MapReduce](http://research.google.com/archive/mapreduce.html) is a *two-stage disk-based* programming model
- batch processing does not suit all use cases
- absence of caching is an issue for iterative algorithms
- it's extremely tedious to write MapReduce programs

Spark.alert[*] is (often) faster, more flexible and comes with a nice collection of libraries

.footnote[.alert[*] Spark can sort 100 TB [3x faster and with 10x fewer machines](http://databricks.com/blog/2014/11/05/spark-officially-sets-a-new-record-in-large-scale-sorting.html) than MapReduce]
]

???

For streaming, Apache Flink is sometimes mentioned as an alternative to Spark, especially with regard to streaming.
Both are very similar yet there are [some differences](http://stackoverflow.com/a/28083781/5193203).

For instance, Spark does most of the heavy lifting in memory and creates *micro-batches* when streaming data.
Flink does true streaming in the sense that data is processed as soon as it arrives.
However, Flink is not (yet) as widely adopted as Spark, it seems.
---
.left-column[
  ## What is Spark?
  ## Cool, but so what?!
  ## OK. Now what?!
]
.right-column[
Here's what you need to get going:

- Basic knowledge of [Scala](http://www.horstmann.com/scala/index.html)
- Basic knowledge of Spark (more on that later)
- Access to Hadoop cluster with Spark or a (free) [VM](http://www.virtualbox.org/wiki/Downloads):
  - [Cloudera QuickStart](http://www.cloudera.com/content/support/en/downloads/quickstart_vms.html)
  - [Hortonworks Sandbox](http://hortonworks.com/products/hortonworks-sandbox)
  - [IBM Infosphere BigInsights Quick Start Edition](http://www.ibm.com/developerworks/downloads/im/biginsightsquick)
  - [MapR Sandbox](http://www.mapr.com/products/mapr-sandbox-hadoop)
  - [Oracle Big Data Lite](http://www.oracle.com/technetwork/community/developer-vm/index.html#bdl)
]

???

There are alternatives to VirtualBox (e.g. VMware), but VirtualBox is completely free.
Docker is also an option for some VMs, e.g. [Cloudera](http://blog.cloudera.com/blog/2015/12/docker-is-the-new-quickstart-option-for-apache-hadoop-and-cloudera).
---
template: inverse

## A Crash Course in Spark
---
layout: false
.left-colum[
  ## Basics
]
.right-column[
The `SparkContext`:
  - is the [main entry point](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkContext) for Spark functionality
  - tells Spark how to access cluster
  - is limited to one *active* instance per JVM
  - is automatically created in interactive shells: `sc`

Interactive shell:
  - Scala: `spark-shell`
  - Python: `pyspark`

API languages:
  - Scala
  - Java
  - Python

Applications are *isolated*
]

???

`SparkContext`: driver sends assembled code (with `spark-submit`) to each worker node's executor.

Scala vs Java vs Python:
- Java does not have REPL (i.e. no interactive shell).

Scala vs Python:
- Scala is faster: compiled instead of interpreted.
- Scala is statically typed.
- Spark is written in Scala; Python wraps Scala code.
- New features are always immediately available in Scala; must be ported to Python.
---
.left-column[
  ## Basics
  ## RDDs
]
.right-column[
RDDs are:
  - **resilient**
  - **distributed** (i.e. partitioned across the nodes of a cluster)
  - **data sets**
  - that are immutable
  - can be cached or temporarily stored on disk
  - and may depend on zero or more RDDs

Operations on RDDs:
- transformations
    - create/extend DAG
    - are evaluated lazily
    - do *not* return values
- actions
    - perform transformations *and* subsequent actions
    - return value(s)

*Lineage* refers to a directed acyclic graph (DAG) from the root or any persisted (intermediate) RDD, from which any partition is reconstructed in case of failure

Spark automatically reschedules tasks to recover lost partitions
]

???

The DAG lineage can be shown (in the shell) with `toDebugString`.

---
.left-column[
  ## Basics
  ## RDDs
  ## Operations
]
.right-column[
Examples of *narrow* (i.e. co-partitioned data) operations:
  - `map`, `flatMap` or `mapValues`
  - `filter`
  - `sample`
  - `union`

Examples of *wide* (i.e. non-co-partitioned data) operations:
  - `groupByKey`
  - `reduceByKey`
  - `sortByKey`
  - `distinct`
  - `join`

*Shuffle* combines data from all partitions to generate new key-value pairs: expensive operation

*Job* is a sequence of transformations initiated by an action
  - A job consists of stages; stages consist of tasks
  - Stage boundaries are defined by shuffle dependencies
  - Tasks are serialized and distributed to executors
]

???

There are many more transformations and of course actions but they are too numerous to [list](http://spark.apache.org/docs/latest/programming-guide.html#transformations).
---
.left-column[
  ## Basics
  ## RDDs
  ## Operations
  ## Caching
]
.right-column[
RDDs can be cached/persisted with:

  - `cache()`: deserialized in memory
  - `persist()`:
      - `MEMORY_ONLY`: deserialized in JVM
      - `MEMORY_ONLY_SER`: serialized; space efficient but more CPU-intensive to read
      - `MEMORY_AND_DISK`: deserialized in JVM unless partition(s) do not fit in memory
      - `MEMORY_AND_DISK_SER`: serialized version of previous
      - `DISK_ONLY`: stored only on disk
      - `OFF_HEAP`: serialized in Tachyon with a shared pool of memory and less overhead from garbage collection

Objects are aged out of cache in a least-recently-used (LRU) fashion; manual removal can be done with `unpersist()`

Spark performs best when data fits in memory; best not to spill to disk unless absolutely necessary

Replicated `persist()` modes perform better after faults
]
---
.left-column[
  ## Basics
  ## RDDs
  ## Operations
  ## Caching
  ## Variables
]
.right-column[
Variables are copied from driver to workers when functions are passed to operations (i.e. they are not shared)

Options for *shared variables*:
  - Broadcast variables:
    - *Single* read-only copy on each machine
    - Ideal when data is needed across multiple stages

  - Accumulators:
    - Aggregated through *associative* operation (e.g. sums or counters)
    - Only driver can read accumulator's value
]
---
.left-column[
  ## Basics
  ## RDDs
  ## Operations
  ## Caching
  ## Variables
  ## Serialization
]
.right-column[
Serialization is critical for network performance and memory use:
  - Java (default): slow but flexible
  - Kryo: fast but not all data types are supported

    ```scala
    conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    conf.registerKryoClasses( Array(classOf[MyClass1], classOf[MyClass2]) )
    ```

Tasks must be serializable (typically < 20 kB on master)

Variables and methods in local vs cluster mode behave differently due to [closures](http://spark.apache.org/docs/latest/programming-guide.html#understanding-closures-a-nameclosureslinka) and serialization
  - classes must be serializable (i.e. extend `Serializable`)
  - methods can be made functions (i.e. transferred from `class`es to `object`s)
  - [hygienic closures](http://erikerlandson.github.io/blog/2015/03/31/hygienic-closures-for-scala-function-serialization) can be created with shim functions
  - classes that depend on non-serializable classes can use the `@transient` [annotation](http://www.artima.com/pins1ed/annotations.html)
]

???

- Making entire classes serializable may not be an option when there are many of large members in the closure of, say, a method.
- Making functions out of methods may not be desirable.
- The `@transient` annotation causes a non-serializable class to be instantiated locally inside each task.

In general, it's best to have local references to objects so as to avoid having to serialize the entire closure.

---
.left-column[
  ## Basics
  ## RDDs
  ## Operations
  ## Caching
  ## Variables
  ## Serialization
  ## Performance
]
.right-column[
The following are guidelines, not the Gospel of St. Spark:
- Cache/persist RDDs after pruning, filtering, parsing and joining, or to avoid recomputation when accessed repeatedly
- Broadcast large variables
- Use local variables when passing objects by reference to avoid sending the entire object
- Use arrays of primitives in favour of nested structures to reduce serialized storage and overhead of garbage collection
- Distribute data evenly across partitions
- Align the number of partitions to the number of CPU cores; schedule 2-3 tasks per core
- Avoid shuffling whenever possible as it causes out-of-memory (OOM) errors (e.g. by doing aggregations by partition and shuffling the results rather than shuffling the complete data):

      ```scala
      // For averages, use...
      val res = rdd.aggregateByKey((0,0))
      (
        (acc, val) => (acc._1 + value, acc._2 + 1),            # (sum, count)
        (acc1, acc2) => acc1._1 + acc2._1, acc1._2 + acc2._2)  # totals
      )
      res.mapValues(i => i._1/i._2)

      // instead of ...
      rdd.groupByKey().mapValues(l => l.sum / l.size)
      ```
]

???

- Actions are blocking operations: the driver has to wait.
- Why broadcast? Local variables in closures are serialized to every node, which causes start-up delays and is a waste of space.
- `mapValues` is often to be preferred: the result of `map` and a subsequent `reducyByKey` may be shuffled ; `mapValues` informs Spark that the hashed keys will remain in their partitions, thereby avoiding a shuffle.
- Spark has hash (default) and range (for sorted data) partitioners, and allows custom partitioners to be defined.
  Tasks that immediately precede and follow a shuffle are called map and reduce tasks, respectively. 
  When using a custom partitioner, shuffling *always* occurs.
  What is more, shuffling also occurs when a hash partitioner with a different number of partitions than before is used.
  Intermediate files (e.g. the file system's cache) are used for these tasks, and the data is sent over the network.
- Spark has two schedulers
    - FIFO (default): each job gets as many resources as it needs
    - Fair: resources are allocated equally across jobs, which is more appropriate for multi-user environments. Pools can each have their own scheduler.

Speculative execution (disabled by default):
- Slow running tasks in each stage are automatically re-launched.
- 3 retrials are default.
- Slow is defined as a multiplier of the median.
- Side effects: output may be written multiple times on retrials!

Asynchronous actions (implemented as `Future`s):
- `collectAsync`
- `countAsync`
- `foreachAsync`
- `foreachPartitionAsync`
- `taskAsync`

FIFO scheduler will still pretty much execute asynchronous actions serially; must use Fair scheduler instead.
---
template: inverse

## Basic Setup
---
layout: false
.left-column[
  ## Installation
]
.right-column[
1. Install [Scala](http://www.scala-lang.org/download)
2. Install [sbt](http://www.scala-sbt.org/download.html)
3. Add `scala` and `sbt` to `$PATH`
4. Install [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
5. Install IDE: [IntelliJ IDEA](http://www.jetbrains.com/idea/download).alert[*],
   [Scala IDE](http://scala-ide.org) (Eclipse),
   or stick with favourite text editor (optional)

.footnote[.alert[*] Standard Scala plugin already includes sbt plugin]
]

???

To add path to `PATH` environment variable:
* Linux: add path to binaries to either `$HOME/.bashrc` or `$HOME/.profile`: `PATH=$PATH:/path/to/binaries`
* Mac OS/X: add path to `/etc/paths`
* Windows:
    * In command line, execute `SETX "%PATH%;C:\path\to\binaries"`
    * Start &rarr; Control Panel,
      search for 'variables',
      click on 'Edit the system environment variables',
      and add the path to the binaries

You may also need to create the `SBT_HOME` variable, which points to the binaries.

---
.left-column[
  ## Installation
  ## Proxies (optional)
]
.right-column[
Deal with proxies only if necessary:

1. Add proxy settings to sbt in `$SBT_HOME/conf/sbtconfig.txt`:

    ```
    -Dhttp.proxyHost=your-proxy-server.com
    -Dhttp.proxyPort=1234
    -Dhttp.proxyUser=userName
    -Dhttp.proxyPassword=thisIsSecret!

    -Dhttps.proxyHost=your-proxy-server.com
    -Dhttps.proxyPort=1234
    -Dhttps.proxyUser=userName
    -Dhttps.proxyPassword=thisIsSecret!
    ```

2. Add custom [repositories](http://stackoverflow.com/a/18511228) to `$HOME\.sbt\repositories`:

    ```
    [repositories]
      local
      sbt-releases-repo: http://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
      sbt-plugins-repo: http://repo.scala-sbt.org/scalasbt/sbt-plugin-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
      maven-central: http://repo1.maven.org/maven2/
      concurrent-maven: http://conjars.org/repo/
    ```
]

???

Only [basic authentication](http://stackoverflow.com/a/13804254/5193203) is supported by sbt.

`$JAVA_OPTS` would be universal as almost all tools (e.g. Maven) use the Java options variable.
Gradle, however, [does not](http://docs.gradle.org/current/userguide/build_environment.html).

The last repository listed is only needed when you want to use Scalding (i.e. Map/Reduce).
---
.left-column[
  ## Installation
  ## Proxies
  ## Setup
]
.right-column[
1. Check the Scala version: `scala -version`
2. Create a project folder with build file `build.sbt`:

    ```
    name := "SparkSandbox"           // Your app's name
    version := "0.0.1"               // Your app's version
    scalaVersion := "2.11.7"         // Use output from "scala -version"
    organization := "com.your-org"   // Your organization's reverse domain name

    libraryDependencies ++= Seq(
      "org.apache.spark" %% "spark-core" % "1.5.2",     // Version must match Scala's
      "org.apache.hadoop" % "hadoop-client" % "2.3.0",
      "org.scalatest" %% "scalatest" % "2.2.5" % "test" // Version must match Scala's
    )
    ```

3. Add a [`plugins.sbt`](http://github.com/sbt/sbt-assembly) file:

    ```bash
    mkdir project
    echo 'addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.1")' > project/plugins.sbt
    ```

5. Create the directory structure
   (or [let IDE handle it](http://scalatutorials.com/beginner/2013/07/18/getting-started-with-sbt))

    ```bash
    mkdir -p src/{main,test}/{java,resources,scala}
    ```
]

???

The difference between [`%%` and `%`](http://www.scala-sbt.org/release/tutorial/Library-Dependencies.html) is that
the former appends the scala version, whereas the latter does not.
One of the aforementioned libraries is equivalent to `"org.apache.spark" % "spark-core_2.11" % "1.5.1"`.
Note the single percentage sign after the groupId `org.apache.spark`.

How do you know which one (`%%` or `%`) to use?
Check [Maven](http://search.maven.org) for the artifact.
There you'll also find the necessary code for most build tools.

The `java` and `resources` folders are optional.

**Important**: in case you run into problems when running `sbt assembly`,
especially errors such as `deduplicate: different file contents found in the following`,
there are two main options:
- Add `% "provided"` to both the Spark and Hadoop client library dependencies;
- Manually exclude the [transitive dependencies](http://github.com/sbt/sbt-assembly#excluding-jars-and-files).

Technically you can also try the [workaround](http://stackoverflow.com/questions/32859367/sbt-assembly-merge-errors-deduplicate) with the `sbt-native-packager` plugin,
or define an assembly merge strategy, but that is a pain in the rear.
---
.left-column[
  ## Installation
  ## Proxies
  ## Setup
  ## Code
]
.right-column[
4. Create a basic application file `SparkSandbox.scala`:

    ```scala
    import org.apache.spark.{SparkConf, SparkContext}

    object SparkSandbox extends App {
      val conf = new SparkConf()
        .setAppName("Spark Sandbox")
        .setMaster("local[*]")

      val sc = new SparkContext(conf)

      sc.stop()
    }
    ```

5. Run `sbt compile`

6. Grab some coffee or tea
]
---
## ScalaTest Setup
1. Create class `src/main/scala/YourClass.scala` that should be unit-tested

2. Create `src/test/scala/UnitSpec.scala` with common mixins:

    ```scala
    import org.scalatest._

    abstract class UnitSpec extends FlatSpec with Matchers with OptionValues with BeforeAndAfterAll
    ```

3. Bundle common behaviour across cases in `src/test/scala/YourClassBehaviour.scala` (if applicable)

4. Create unit tests in `src/test/scala/YourClassSpec.scala` that

  - inherits (i.e. extends) `UnitSpec`
  - and mixes in common behaviour from `YourClassBehaviour`

5. Run `sbt test`

???

The abstract class `UnitSpec` mainly contains the traits you intend to use often,
so that you don't have to repeat yourself in each [unit test specification](http://www.scalatest.org/quick_start).

* `FlatSpec` facilitates behaviour-driven development (BDD),
  in which tests are accompanied by text that specifies the desired behaviour.
* `Matchers` provides a DSL to write assertions that look like normal English.
* `OptionValues` enables you to deal with `Option`s using the `value` method.
* `BeforeAndAfterAll` is ScalaTest's equivalent of setup and teardown operations for suites of unit tests.
  There is also `BeforeAndAfterEach`, which can be used for code that needs to be run before and/or after *each* unit test.
  `BeforeAndAfter` is a tad simpler than `BeforeAndAfterEach`, but it does not allow traits to be stacked.

When using Gradle, `JUnitRunner` [must be imported](http://stackoverflow.com/a/18824251):

  ````scala
  import org.junit.runner.RunWith
  import org.scalatest._
  import org.scalatest.junit.JUnitRunner

  @RunWith(classOf[JUnitRunner])
  abstract class UnitSpec extends FlatSpec with Matchers with OptionValues with BeforeAndAfter
  ```

---
template: inverse

## An Example
---
layout: false
.left-column[
  ## App code
]
.right-column[
Create an auxiliary class `src/main/scala/Car.scala`:

  ```scala
  case class Car(model: String, mph: Integer)
  ```

Create the application code `src/main/scala/CarAnalysis.scala`:

  ```scala
  import org.apache.spark.SparkContext
  import org.apache.spark.rdd.RDD

  class CarsAnalysis(sc: SparkContext, cars: RDD[Car]) {

    def fastestCar() = {
      cars.sortBy(-_.mph).first
    }

    def aboveMPHLimit(mphLimit: Integer) = {
      val above = cars.filter(_.mph >= mphLimit)
      above.sortBy(-_.mph).collect.toList
    }
  }
  ```
]
---
.left-column[
  ## App code
  ## Unit tests
]
.right-column[
Create unit tests based on `UnitSpec`:

  ```scala
  import org.apache.spark.{SparkContext, SparkConf}

  class CarAnalysisSpec extends UnitSpec {
    // Setup
    val MPH_LIMIT = 140

    val cars = List(Car("Family Sedan", 125),
                    Car("Canyonero", 120),
                    Car("Lamborgotti Fasterossa", 200),
                    Car("Rocket Car", 240))

    val fastCars = cars.filter(_.mph >= MPH_LIMIT).sortBy(-_.mph)
    val fastestCar = cars.sortBy(-_.mph).head

    // Local Spark setup
    val conf = new SparkConf()
      .setAppName("Spark Sandbox: CarAnalysis Suite")
      .setMaster("local[*]")
    val sc = new SparkContext(conf)
    val carsRDD = sc.makeRDD(cars)
    val analysis = new CarsAnalysis(sc, carsRDD)

    // Unit tests
    "A CarAnalysis" should "have exactly 1 fastest car" in {
      val fastest = analysis.fastestCar
      fastest should be(fastestCar)
    }

    it should "list all cars above a given MPH limit in descending order" in {
      val fast = analysis.aboveMPHLimit(MPH_LIMIT)
      fast.size should be(fastCars.size)
      fast should be(fastCars)
    }

    // Teardown
    override def afterAll() {
      sc.stop()
    }
  }
  ```
]

???

Here, we don't have any common behaviour traits that we could mix in.
---
template: inverse

## Deployment
---
## Deployment
1. Ensure master node URL is *not* set with `setMaster()` in main program

2. Create package:

    ```bash
    sbt assembly
    ```

3. Copy jar from `target/scala_2.11/SparkSandbox-assembly-*.jar` to your hadoop cluster (e.g. with ftp)

    ```bash
    sftp your-user@some-hadoop-cluster.com

    put /local/path/*.jar /remote/path/
    ```

4. Deploy and [run](http://spark.apache.org/docs/1.1.0/submitting-applications.html#master-urls):

    ```bash
    spark-submit \
      --class SparkSandbox \
      --master yarn-cluster \
      SparkSandbox-assembly-0.0.1.jar
    ```

5. Check [YARN logs](http://spark.apache.org/docs/latest/running-on-yarn.html) (e.g. to see output from `println` when debugging):

    ```bash
    yarn logs -applicationId <applicationId>
    ```

    Application ID is shown in lines with `tracking URL` and `Application report for`

???

Master node URL is set by `spark-submit`, hence it must *not* be set.

When you package you application with `package com.my-org`, it's best to place files in a folder structure that mimics that: `src/main/scala/com/my-org/`.
---
template: inverse

## What's next?
---
## Spark Libraries and Beyond
Check out the various libraries:

- Spark SQL

    ```
    "org.apache.spark" %% "spark-sql" % "1.5.2"
    ```

- Spark Streaming

    ```
    val sparkVersion = "1.5.2"
    "org.apache.spark" %% "spark-streaming" % sparkVersion
    "org.apache.spark" %% "spark-streaming-kafka" % sparkVersion // Optional
    "org.apache.spark" %% "spark-streaming-flume" % sparkVersion // Optional
    ```

- Graph analysis with Spark GraphX

    ```
    "org.apache.spark" %% "spark-graphx" % "1.5.2"
    ```

- Machine learning with MLlib

    ```
    "org.apache.spark" %% "spark-mllib" % "1.5.2"
    ```

Perform unit tests [on a cluster](http://eugenezhulenev.com/blog/2014/10/18/run-tests-in-standalone-spark-cluster) instead of locally, or try [Spark Testing Base](http://github.com/holdenk/spark-testing-base)

Check out continuous integration with [Jenkins and sbt](http://www.whiteboardcoder.com/2014/01/jenkins-and-sbt.html)

???

A `DataFrame` is an RDD with a schema; Spark SQL infers the schema when it loads the data.
---
## Resources
* [Apache Spark](http://spark.apache.org/docs/latest)
* [Typesafe activator](http://www.typesafe.com/activator/download) examples
* [Databricks reference applications](http://github.com/databricks/reference-apps)
* Big Data University's [Spark Fundamentals I](http://bigdatauniversity.com/courses/spark-fundamentals) and [II](http://bigdatauniversity.com/courses/spark-fundamentals-ii)
* [ScalaTest's user guide](http://www.scalatest.org/user_guide)
* Pat McDonough's [2013 Spark Summit tutorial](http://spark-summit.org/wp-content/uploads/2013/10/McDonough-spark-tutorial_spark-summit-2013.pptx)
* [Willem Meints'](http://fizzylogic.nl/2015/11/10/spark-101-writing-your-first-spark-app) [Spark 101](http://fizzylogic.nl/2015/11/12/spark-101-integration-with-data-sources)
* Ian Hellström's [sparkSetup.sh and updateAppConfig.sh](http://bitbucket.org/hell316/dbline/src/) shell scripts (see: `bash`)
* SequenceIQ's [posts on Spark](http://blog.sequenceiq.com/blog/categories/spark)
* [Spark in Action](http://www.manning.com/books/spark-in-action)
