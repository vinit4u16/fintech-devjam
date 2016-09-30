![](./media/image14.png)

*Lab 4 - Creating Custom Reports*

![](./media/image16.png)

**Overview**

Let's say your API has gained wide adoption. It's popular. You have
attracted a number of talented, creative app developers and people are
downloading and installing their apps. Obviously, the API team is very
interested in how the API is performing, how it's being used, and how to
plan for improvements. Apigee Edge Analytics Services collects and
analyzes a wealth of information that flows through APIs. This
information is gathered, analyzed, and provided to you immediately, in
real time. In this lab we will see how you can extend the Edge analytics
services by create dimension and metrics and use them in Custom reports.
We will have to use Statistics collector policy to create custom
dimensions and metrics.

![](./media/image15.png)

**Statistics Collector policy**
Enables you to collect statistics for data in a message, such as
product ID, price, REST action, client and target URL, and message
length. The data can come from flow variables predefined by Apigee or
custom variables that you define. The statistics data is passed to the
analytics server, which analyzes the statistics and generates reports.
You can view the reports by using the Edge management UI or Edge API.

**Custom Reports**
There are several Out-of-the-box, “Standard” reports that are
automatically provided for every Edge organization. They track several
critical operational metrics, such as proxy response time, target
response time, cache performance, error rates, and others. An API
Publisher can create custom reports to augment the standard reports. By
adding custom reports, you can create a set of charts that provide
insight into the exact aspects of your API program that you wish to
analyze.


**Objectives**

The objective of this lesson is to get you familiar with Apigee’s
Statistics collector policy and learn how to create custom reports in
Edge.

**Prerequisites**

- Lab 2 is completed

**Estimated Time: 45 mins**

Now we will enhance the proxy to start collecting some statistics from
the response payload. As part of this we will add a couple of policies
including a Statistics Collector to the existing proxy that we have just
created. Let's add a policy **-**

1.  Go to the Apigee Edge Management UI browser tab

2.  Go to the ‘{your_initials}\_payment’ proxy’s ‘Develop’ tab

3.  Click on ‘Get Payment by uuid’

4.  Click on “**+ Step**” on the Response Flow

  > ![](./media/image21.png)

5.  Select the ‘Extract Variables’ policy with the following properties:

  > Policy Display Name: **Extract Variables**<br/>
  > Policy Name: **Extract-Variables**

  > ![](./media/image27.png)

6.  For the ‘Extract Variables’ policy, change the XML configuration of
    the policy as follows :

  ```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ExtractVariables async="false" continueOnError="false" enabled="true" name="Extract-Variables">
    <DisplayName>Extract Variables</DisplayName>
    <JSONPayload>
        <Variable name="sourceType">
            <JSONPath>$.entities[0].sourceType</JSONPath>
        </Variable>
        <Variable name="transactionStatus">
            <JSONPath>$.entities[0].transactionstatus</JSONPath>
        </Variable>
        <Variable name="amount">
            <JSONPath>$.entities[0].amount</JSONPath>
        </Variable>
    </JSONPayload>
    <Source clearPayload="false">response</Source>
</ExtractVariables>
  ```


  The JSONPath expression above extracts source type & status information from
  the response returned the payments API and assigns it to variables
  “sourceType” and “transactionStatus” respectively.

7.  Select the ‘Statistics Collector’ policy with the following
    properties:

  > Policy Display Name: **Statistics Collector**<br/>
  > Policy Name: **Statistics Collector**

  **Note :** Make sure you have Statistics collector policy after
Extract Variable policy on the response path (as shown below).

  > ![](./media/report-policies.png)

8.  For the ‘Statistics Collector’ policy, change the XML configuration
    of the policy as follows :

  ```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<StatisticsCollector async="false" continueOnError="false" enabled="true" name="Statistics-Collector">
    <DisplayName>Statistics Collector</DisplayName>
    <Properties/>
    <Statistics>
        <Statistic name="devjam_{your-initials}_type" ref="sourceType" type="String">NO_SOURCE_TYPE</Statistic>
        <Statistic name="devjam_{your-initials}_status" ref="transactionStatus" type="String">NO_STATUS</Statistic>
        <Statistic name="devjam_{your-initials}_amount" ref="amount" type="Float">0.00</Statistic>
    </Statistics>
</StatisticsCollector>
  ```

  Replace {your\_initials} with your initials. Note that you will see devjam\_{your\_initials}\_type, as a dimension in a
custom report, whereas devjam_\{your\_initials}\_status as a
metric.

  As we will want to send in a few test messages then bump up the “rate” of Spike Arrest to some arbitrarily high number so
you can run a few requests through.

  ```
  <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
  <SpikeArrest async="false" continueOnError="false" enabled="true"name="Spike-Arrest-10pm">
  <DisplayName>Spike Arrest 10pm</DisplayName>
  <Properties/>
  <Identifier ref="request.header.some-header-name"/>
  <MessageWeight ref="request.header.weight"/>
  <Rate>1000pm</Rate>
  </SpikeArrest>
  ```
9. Save your proxy by hitting “Save”, and now you can test the Statistics collector policy by generating some load

**Test the Statistics Collector Policy**

1.  Start a Trace session for the ‘{your_initials}_payments’ proxy

2.  Send a ‘/GET payments’ request from Postman. This will return the list
    of payments from BaaS collection.

3.  Copy the UUID of any payment entity

4.  Append that the copied UUID to the URL of ‘/GET payments’ request and
    send another request from Postman. 

  NOTE : replace your org, env and {your_initials}_payments with
  your Edge Org and Environment and proxy names. If you completed lab 3 you will also need to add your apikey as a query parameter.

5.  Review the Trace for the proxy and notice that “Extract Variables”
    and “Statistics Collector” policies get executed only for the GET
    request with UUID.

6.  Invoke the ‘/GET payments’ with different UUIDs to generate some test data.

**Creating a Custom Report**

1.  Login to the Apigee Edge Management User Interface (Management UI).
    On the top menu, click on the Analytics item and then click
    on “Reports”. When on that page click on the '+ Custom Report'
    button on the top right.

  > ![](./media/image13.png)

2.  Define the custom report - Enter the values as indicated below and
    click on the blue “Save” button.

  a.  Report Name: {your initials} Payment Type and Status

  b.  Report Description: This report shows the most popular destination queried.

  c.  Chart Type: Column

  d.  Add two Metrics: Traffic - Sum, devjam_{your_intials}_amount - Average

  > This will create a multidimensional report.

  f.  Dimensions: devjam_{your_initials}_type, devjam_{your_initials}_status
    These are the variables which we created in earlier using
    Statistics Collector to capture the payment amount, type and status from the
    response payload. Using the Statistics Collector policy, you can
    capture variables from the request or response, including headers,
    body, and other attributes and have the analytics engine start
    harvesting these values for you. Generally these variables will
    show up under the “custom dimensions” category in the
    Drilldown drop-down. Notice that there are many “standard”
    dimensions which Edge collects for you, for every request,
    including response time, payload size, and so on.

  h.  Hit the save button to save and open the report - use the environment dropdown to switch to the test environment. Edge Analytics performs aggregation on a regular interval, asynchronously with respect to incoming API requests. Therefore, you may have to wait a bit to see the data appear in the chart. After a cycle of aggregation occurs, you will see:

  > ![](./media/report.png)

  The final configuration of the custom report will look like the
following :

  > ![](./media/report-metrics2.png)

**Summary**

In this exercise, you learned about the statistics collector policy &
custom reports in Apigee Edge. You also added the custom report along
with other reports to a custom dashboard. Please visit the
[*documentation*](http://apigee.com/docs/api-services/content/analytics-dashboards)
to see the different kinds of operational reports and dashboards that
are available to you.

**Bonus Section**

-   Review out of the box reports provided by Apigee Edge on
    -   Proxy performance
    -   Cache performance
    -   Backend performance
    -   Latency reports
    -   Traffic reports on the Geo-map
    -   Developer Engagement
    -   Error Analysis reports
