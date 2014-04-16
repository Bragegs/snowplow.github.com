---
layout: post
shortenedlink: Spark Example Project released
title: Spark Example Project released for running Spark jobs on EMR
tags: [snowplow, spark, emr, example, tutorial]
author: Alex
category: Releases
---

On Saturday I attended [Hack the Tower] [hack-the-tower], the monthly collaborative hackday for the London Java and Scala user groups hosted at the Salesforce offices here. It's an opportunity to catch up with others in the Scala community, and to work collaboratively on non-urgent projects which may have longer-term value for us here at Snowplow. It also means I can hack against the backdrop of some of the best views in London (see below)! Many thanks as always to [John Stevenson] [xxx] of Salesforce for hosting us. 

Over the last few months I have been teaming up with other Scala heads to try out [Apache Spark] [apache-spark], a cluster computing framework and potential challenger to Hadoop. The particular challenge I set myself this month was to complete our [Spark Example Project] [spark-example-project], which is a clone of our popular [Scalding Example Project] [scalding-example-project]. Most tutorials introducing data processing frameworks like Scalding or Spark assume that you are working with a local cluster in an interactive (e.g. REPL-based) fashion. At Snowplow, we are much more interested in creating self-contained jobs which can be run on Amazon's Elastic MapReduce with a minimum of supervision, so that is what I set out to template in the Spark Example Project.

In the rest of this blog post I'll talk about:

* Challenges of running Spark on EMR
* How to use Spark Example Project
* Getting help
* Thoughts on Spark
* Spark and Snowplow

1. [Changes to the API](/blog/2014/04/15/snowplow-python-tracker-0.2.0-released/#api)
2. [Python 2.7](/blog/2014/04/15/snowplow-python-tracker-0.2.0-released/#compatibility)
3. [Integration tests](/blog/2014/04/15/snowplow-python-tracker-0.2.0-released/#tests)
4. [Other improvements](/blog/2014/04/15/snowplow-python-tracker-0.2.0-released/#other)
5. [Upgrading](/blog/2014/04/15/snowplow-python-tracker-0.2.0-released/#upgrading)
6. [Support](/blog/2014/04/15/snowplow-python-tracker-0.2.0-released/#support)

<!--more-->

<div class="html">
<h2><a name="challenges">1. Challenges of running Spark on EMR</a></h2>
</div>

We actually built an initial version of the Spark Example Project in 2013 when we first heard about Spark through the AWS tutorial, XXX by [XXX] [xxx]. We spent some time trying to get the project working on Elastic MapReduce: we wanted to be able to assemble a "fat jar" which we could deploy to S3 and then run on Elastic MapReduce via the API in a non-interactive way. This wasn't possible at the time, despite the valiant efforts of [XXX] [xxx] ([XXX] [xxx]) and [XXX] [xxx] ([XXX] [xxx]), and [our own questioning] [forum-post]. And so I paused on the project, to revisit when EMR's support for Spark progressed.

Happily on Saturday I noticed that the same AWS tutorial had been updated in early March, with scripts to deploy Spark 0.8.1 to an EMR cluster. We saw that the vanilla EMR Spark install included a script called `xxx`, designed to run one of Amazon's example Spark jobs, which had been pre-assembled into a fat jar.

It wasn't a lot of work to take the `xxx` script, and adapt it so that it could be used to run any pre-assembled Spark fat jar: that script is now available for anyone to invoke on Elastic MapReduce here:

xxx

Once this was working, it was just a matter of reverting our Spark Example Project to Spark 0.8.1, testing it thoroughly and updating the documentation!

<div class="html">
<h2><a name="usage">2. How to use Spark Example Project</a></h2>
</div>

Getting up-and-running with the Spark Example Project should be relatively straightforward:

<div class="html">
<h3>2.1 Building</h3>
</div>

Assuming you already have [SBT] [sbt] installed:

$ git clone git://github.com/snowplow/spark-example-project.git
$ cd spark-example-project
$ sbt assembly

The 'fat jar' is now available as:

target/spark-example-project-0.2.0.jar

<div class="html">
<h3>2.2 Deploying</h3>
</div>

Now upload the jar to an Amazon S3 bucket and make the file publically accessible.

Next, upload the data file [`data/hello.txt`] [hello-txt] to S3.

<div class="html">
<h3>2.3 Running</h3>
</div>

Finally, you are ready to run this job using the [Amazon Ruby EMR client] [emr-client]:

```
$ elastic-mapreduce --create --name "Spark Example Project" --instance-type m1.xlarge --instance-count 3 \
  --bootstrap-action s3://elasticmapreduce/samples/spark/0.8.1/install-spark-shark.sh --bootstrap-name "Install Spark/Shark" \
  --jar s3://elasticmapreduce/libs/script-runner/script-runner.jar --step-name "Run Spark Example Project" \
  --step-action TERMINATE_JOB_FLOW \
  --arg s3://snowplow-hosted-assets/common/spark/run-spark-job-0.1.0.sh \
  --arg s3://{{JAR_BUCKET}}/spark-example-project-0.2.0.jar \
  --arg com.snowplowanalytics.spark.WordCountJob \
  --arg s3n://{{IN_BUCKET}}/hello.txt \
  --arg s3n://{{OUT_BUCKET}}/results
```

Replace `{{JAR_BUCKET}}`, `{{IN_BUCKET}}` and `{{OUT_BUCKET}}` with the appropriate paths.

<div class="html">
<h3>2.4 Verifying</h3>
</div>

Once the output has completed, you should see a folder structure like this in your output bucket:

 results
 |
 +- _SUCCESS
 +- part-00000
 +- part-00001

Download the files and check that `part-00000` contains:

(hello,1)
(world,2)

while `part-00001` contains:

(goodbye,1)

And that's it!

<div class="html">
<h2><a name="help">3. Getting help</a></h2>
</div>

<div class="html">
<h2><a name="spark-thoughts">4. Thoughts on Spark</a></h2>
</div>

Although it is early days, I was impressed with Spark, and very pleased to have it running on Elastic MapReduce in exactly the same fashion as our existing Scalding jobs.

In particular, I like Spark's use of in-memory processing, where in contrast Hadoop can be rather disk-intensive. I also like Spark's tight focus: where Hadoop is an entire data ecosystem (file system, cluster management, job scheduling etc), Spark is much more manageable, being designed to work with other great technology such as [Apache Mesos] [mesos], [Typesafe Akka] [akka], [HDFS] [hdfs] et al.

Separately, at Snowplow are also closely following the Spark Streaming project, and excited about Amazon's [work adding Kinesis support] [kinesis-spark-streaming] there.

<div class="html">
<h2><a name="snowplow-spark">5. Snowplow and Spark</a></h2>
</div>

We are excited at Snowplow about the long-term potential for Apache Spark as a data processing framework to use alongside or potentially in places instead of Hadoop.

As a first step, we plan to pilot writing bespoke data processing jobs for Professional Services clients in Spark, where previously we would have used Scalding. If this goes well, we may experiment with running the Snowplow Enrichment process (scala-common-enrich) from inside Spark.

Separately, we will look into integrating Spark Streaming into our [Snowplow Kinesis flow] [snowplow-kinesis]; this could be a great way of implementing real-time decisioning flows and feedback loops for our users.

Stay tuned for more from Snowplow about Spark and Spark Streaming in the future!

[hack-the-tower]: http://www.hackthetower.co.uk/
[hack-the-tower-apr]: http://www.meetup.com/london-scala/events/173280452/
[]

[forum-post]: xxx
[emr]: http://aws.amazon.com/elasticmapreduce/
[spark-example-project]: https://github.com/snowplow/spark-example-project
[scalding-example-project]: https://github.com/snowplow/scalding-example-project

[mesos]: 
[akka]: 
[hdfs]: 

[kinesis-spark-streaming]: xxx


[snowplow-kinesis]: xxx

[hello-txt]: https://github.com/snowplow/spark-example-project/raw/master/data/hello.txt
[emr-client]: http://aws.amazon.com/developertools/2264
