---
title: Application Console
---

# Visualizing and monitoring your application
The Quarks application console is a web application that enables you to visualize your application topology and monitor the tuples flowing through your application.

The application console includes two sections.

## Topology graph
The topology graph section enables you to visualize the following aspects of your topology:

  * Oplet types
  * Stream tags
  * Tuple flows

However, what you can visualize depends on how your application is composed:

  * You can always visualize oplet types and a static view of the topology
  * Your application must include tags to view stream tags
  * Your application must include counters or rate meters to visualize tuple flows

## Metrics
The metric section enables you to closely monitor the flow rate of tuples through the oplets in your environment. Specifically, you can see charts that enable you to monitor:

  * Tuple counts for all of the oplets that have either a counter or a rate metric.
  * The change rate in the tuple count for an oplet that has a rate metric. You can view the following change rates for a single oplet at a time:
    * 1 minute change rate
    * 5 minute change rate
    * 15 minute change rate
    * Mean change rate

# Adding the console to your application
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


# Accessing the console
To access the console from your web browser, enter the URL that is printed to the command line. The URL has the following format:

http://host_name:port_number/console

If you cannot access the console at this URL, ensure your running application has a `console.war` file in the `webapps` directory.
