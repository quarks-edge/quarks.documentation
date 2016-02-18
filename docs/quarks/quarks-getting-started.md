---
layout: docs
title: Getting Started
description:  Quarks Getting Started
weight: 10
---

# Getting Started With Quarks
Quarks is an open source programming model and runtime for edge devices that enables you to analyze streaming data on your edge devices. When you analyze on the edge, you can:

* Reduce the amount of data that you transmit to your analytics server

* Reduce the amount of data that you store

For more information, see the [Quarks overview](../overview)

## Quarks and Streaming Analytics
The fundamental building block of a Quarks application is a **stream**: a continuous sequence of tuples (messages, events, sensor readings, and so on).

The Quarks API provides the ability to process or analyze each tuple as it appears on a stream, resulting in a derived stream.

Source streams are streams that originate data for analysis, such as readings from a device's temperature sensor.

Streams are terminated using sink functions that can perform local device control or send information to centralized analytic systems through a message hub.

Quarks' primary API is functional where streams are sourced, transformed, analyzed or sinked though functions, typically represented as lambda expressions, such as `reading -> reading < 50 || reading > 80` to filter temperature readings in Fahrenheit.


## Downloading Quarks
To access Quarks, download a release from GitHub. We recommend downloading the [latest release](https://github.com/quarks-edge/quarks/releases/latest).

After you download and extract the Quarks package, you can set up your environment.

## Setting up your environment
Ensure that you are running a supported environment. For more information, see the [Quarks overview](../overview). This guide assumes you're running Java 8.

The Quarks Java 8 JAR files are located in the `quarks/java8/lib` directory.


1. Create a new Java project in Eclipse, and specify Java 8 as the execution environment JRE:

    <img src="../images/New_Java_Project.JPG" style="width:462px;height:582px;">


2. Modify the Java build path to include all of the JAR files in the `quarks\java8\lib` directory:

    <img src="../images/Build_Path_Jars.JPG" style="width:661px;height:444px;">

<br/>
Your environment is set up! You you can start writing your first Quarks application.


# Creating a simple application
If you're new to Quarks or to writing streaming applications, the best way to get started is to write a simple program.

Quarks is a framework that pushes data analytics and machine learning to *edge devices*. (Edge devices include things like routers, gateways, machines, equipment, sensors, appliances, or vehicles that are connected to a network.) Quarks enables you to process data locally---such as, in a car engine, on an Android phone, or Raspberry Pi---before you send data over a network.

For example, if your device takes temperature readings from a sensor 1,000 times per second, it is more efficient to process the data locally and send only interesting or unexpected results over the network. To simulate this, let's define a (simulated) TempSensor class:


  {% highlight java %}

  	import java.util.Random;

  	import quarks.function.Supplier;

  	/**
     * Every time get() is called, TempSensor generates a temperature reading.
     */
    public class TempSensor implements Supplier<Double> {
  		double currentTemp = 65.0;
  		Random rand;

  		TempSensor(){
  			rand = new Random();
  		}

  		@Override
  		public Double get() {
  			// Change the current temperature some random amount
  			double newTemp = rand.nextGaussian() + currentTemp;
  			currentTemp = newTemp;
  			return currentTemp;
  		}
  	}
  {% endhighlight %}


Every time you call `TempSensor.get()`, it returns a new temperature reading. The continuous temperature readings are a stream of data that a Quarks application can process.

Our sample Quarks application processes this stream by filtering the data and printing the results:

  {% highlight java %}

  public static void main(String[] args) throws Exception {
    TempSensor sensor = new TempSensor();
    DirectProvider dp = new DirectProvider();      
    Topology topology = dp.newTopology();
    TStream<Double> tempReadings = topology.poll(sensor, 1, TimeUnit.MILLISECONDS);
    TStream<Double> filteredReadings = tempReadings.filter(reading -> reading < 50 || reading > 80);

    filteredReadings.print();
    dp.submit(topology);
  }
  {% endhighlight %}

To understand how the application processes the stream, let's review each line.

## Specifying a Provider
Your first step when you write a Quarks application is to create a
[`DirectProvider`](http://quarks-edge.github.io/quarks/docs/javadoc/index.html?quarks/providers/direct/DirectProvider.html) :

    DirectProvider dp = new DirectProvider();

A **Provider** is an object that contains information on how and where your Quarks application will run. A **DirectProvider** is a type of Provider that runs your application directly within the current virtual machine when its `submit()` method is called.

## Creating a Topology
Additionally a Provider is used to create a
[`Topology`](http://quarks-edge.github.io/quarks/docs/javadoc/index.html?quarks/topology/Topology.html) instance :

    Topology topology = dp.newTopology();

In Quarks, **Topology** is a container that describes the structure of your application:

* Where the streams in the application come from

* How the data in the stream is modified

In the TempSensor application above, we have exactly one data source: the `TempSensor` object. We define the source stream by calling `topology.poll()`, which takes both a Supplier function and a time parameter to indicate how frequently readings should be taken. In our case, we read from the sensor every millisecond:

    TStream<Double> tempReadings = topology.poll(sensor, 1, TimeUnit.MILLISECONDS);

## Defining The TStream Object
Calling `topology.poll()` to define a source stream creates a `TStream<Double>` instance, which represents the series of readings taken from the temperature sensor.

A streaming application can run indefinitely, so the TStream might see an arbitrarily large number of readings pass through it. Because a TStream represents the flow of your data, it supports a number of operations which allow you to modify your data.

## Filtering a TStream
In our example, we want to filter the stream of temperature readings, and remove any "uninteresting" or expected readings---specifically readings which are above 50 degrees and below 80 degrees. To do this, we call the TStream's `filter` method and pass in a function that returns *true* if the data is interesting and *false* if the data is uninteresting:

    TStream<Double> filteredReadings = tempReadings.filter(reading -> reading < 50 || reading > 80);

As you can see, the function that is passed to `filter` operates on each tuple individually. Unlike data streaming frameworks like [Apache Spark](https://spark.apache.org/), which operate on a collection of data in batch mode, Quarks achieves low latency processing by manipulating each piece of data as soon as it becomes available. Filtering a TStream produces another TStream that contains only the filtered tuples; for example, the `filteredReadings` stream.

## Printing to Output
When our application detects interesting data (data outside of the expected parameters), we want to print results. You can do this by calling the `TStream.print()` method, which prints using  `.toString()` on each tuple that passes through the stream:

    filteredReadings.print();

Unlike `TStream.filter()`, `TStream.print()` does not produce another TStream. This is because `TStream.print()` is a **sink**, which represents the terminus of a stream.

In addition to `TStream.print()` there are other sink operations that send tuples to an MQTT server, JDBC connection, file, or Kafka cluster. Additionally, you can define your own sink by invoking `TStream.sink()` and passing in your own function.


## Submitting Your Application
Now that your application has been completely declared, the final step is to run your application.

`DirectProvider` contains a `submit()` method, which runs your application directly within the current virtual machine:

    dp.submit(topology);

After you run your program, you should see output containing only "interesting" data coming from your sensor:

	49.904032311772596
	47.97837504039084
	46.59272336309031
	46.681544551652934
	47.400819234155236
	...

As you can see, all temperatures are outside the 50-80 degree range. In terms of a real-world application, this would prevent a device from sending superfluous data over a network, thereby reducing communication costs.

## Further Examples
This example demonstrates a small piece of Quarks' functionality. Quarks supports more complicated topologies, such as topologies that require merging and splitting data streams, or perform operations which aggregate the last *N* seconds of data (for example, calculating a moving average).

For more complex examples, see:

* [Quarks sample programs](../samples)
* [Common Quarks operations](../common-quarks-operations)

# Accessing the Quarks application console
The Quarks application console is a web application that enables you to visualize your application topology and monitor the tuples flowing through your application.

The application console includes two sections:

* A topology graph section, which enables you to visualize the following aspects of your topology:
  * Oplet types
  * Stream tags
  * Tuple flows

  However, what you can visualize depends on how your application is composed:
    * You can always visualize oplet types and a static view of the topology
    * Your application must include tags to view stream tags
    * Your application must include counters or rate meters to visualize tuple flows

*	A metric section, which enables you to closely monitor the flow rate of tuples through the oplets in your environment. Specifically, you can see charts that enable you to monitor:

  * Tuple counts for all of the oplets that have either a counter or a rate metric.
  * The change rate in the tuple count for an oplet that has a rate metric. You can view the following change rates for a single oplet at a time:
    * 1 minute change rate
    * 5 minute change rate
    * 15 minute change rate
    * Mean change rate

## Adding the console to your application
If you want to use the console, you must embed a web server in your application that configures a web application for the console.

To include the console in your application, your application must use the DevelopmentProvider instead of the DirectProvider. You can get the url for the console from the development provider using the getService method as shown in the modified application below:

  {% highlight java %}

	import java.util.concurrent.TimeUnit;

	import quarks.console.server.HttpServer;
	import quarks.providers.development.DevelopmentProvider;
	import quarks.topology.TStream;
	import quarks.topology.Topology;

	public class TempSensorApplication {
		public static void main(String[] args) throws Exception {
		    TempSensor sensor = new TempSensor();
		    DevelopmentProvider dp = new DevelopmentProvider(); 
		    Topology topology = dp.newTopology();
		    TStream<Double> tempReadings = topology.poll(sensor, 1, TimeUnit.MILLISECONDS);
		    TStream<Double> filteredReadings = tempReadings.filter(reading -> reading < 50 || reading > 80);
		    filteredReadings.print();
	    
		    // print the console URL and wait for 10 seconds before submitting the topology
		    System.out.println(dp.getServices().getService(HttpServer.class).getConsoleUrl());
		    try {
		        TimeUnit.SECONDS.sleep(10);
		    } catch (InterruptedException e) {
		        //do nothing
		    }
		    dp.submit(topology);
		  }
	}

  {% endhighlight %}


## Accessing the console
To access the console from your web browser, enter the URL that is printed to the command line. The URL has the following format:

http://localhost:port_number/console

Important: You can access the console only from the host where you run your application.

If you cannot access the console at this URL, ensure your running application has a `console.war` file in the `webapps` directory.
