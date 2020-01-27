[Back to Home](/my-notes)

# Performance Testing in Azure / JMeter

It looks like using JMeter in an Azure pipeline is definitely possible.
It looks like JMeter is pretty similar to the other tool I have used before
called WebSurge. Both allow you to create and store tests to load test endpoints.
WebSurge is probably a bit more simplistic than JMeter though.

JMeter does require a bit of work to get going in Azure. It requires the installation
of the Java JDK and adding some other Java specific environment variables.

This
[Medium article](https://medium.com/@ganeshsirsi/configuring-jmeter-tests-in-vsts-tfs-azure-devops-and-publishing-results-dcdd7b451cb9)
has a decent step by step on how to get JMeter running in Azure DevOps build pipeline.

Azure had load testing built into it but it has now been deprecated.

## What is it?

JMeter is a tool that will fire requests and get responses in HTML/JSON/XML/etc.
It does not execute any JS in your UI. It only fires off requests to the server.

It has tools to record user sessions and run load tests based on that.
You can also specify a URL and amounts of users/requests/timespans.
It's highly configurable with all kinds of variables. It can also be extended using plugins like Custom Thread Groups, 3 Basic Graphs, Throughput Shaping Timer, and Dummy Sampler.

It can test different types of servers: web, LDAP, SMTP, etc.

## What we're doing with it

We're going to be using JMeter for load testing.

One use case scenario is that we might test a large/long running SQL Script
in that is in progress and while the script is executing use JMeter to fire off
a bunch of requests to the dev instance of our API that would make DB changes as well and see the results.

JMeter says that in order to do load testing, your tests should be run via the CLI
(which is what you would use in your pipeline as well).
You will still create and can run your test suite in the GUI though.

We want to measure response times, throughput, reliability, and scalability of our API.

## Types of tests

There are multiple tests that you may want to configure and run against your API. (I hope my skill in MS Paint is appreciated)

-   Smoke Tests - a simple test with one user at normal load - just verifies that your tests work
-   Load Tests - used to validate the performance of an application at a specific load / duration.
-   Stress Tests - run a very high load (lots of users and requests) - you want to see where your application finally breaks down - ![stress test](./images/stress-test.png)
-   Spike Tests - test for huge spikes in users/requests - peaks and valleys - ![spike test](./images/spike-test.png)
-   Endurance Tests - test at normal load for an extended period of time (hours/days) - good for seing if there are memory leaks for instance - ![endurance test](./images/endurance-test.png)
