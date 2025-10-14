---
author: James Richardson
pubDatetime: 2021-11-14T15:22:00Z
modDatetime: 2025-06-13T16:52:45.934Z
title: Monitoring 'things' with AWS IoT
slug: aws-iot-asset-condition-monitoring
featured: false
draft: false
tags:
  - aws
  - IoT
description: "Building an asset condition monitoring system using AWS IoT services."


---


Developing IoT applications is hard! ... Let's break down the process and steps to create a working solution.

Internet of Things (IoT) presents an unparalleled opportunity for every industry to address core business challenges, such as reducing downtime, improving safety, increasing system output, reducing operating costs, and creating innovative services and business models.

During my time at AWS, I authored the following blog post detailing how you can leverage AWS IoT services to build an asset condition monitoring solution to capture data from physical assets so that you can understand their status and performance and take appropriate action.

## Why monitor your assets?

When you monitor the status of industrial assets like machinery, remote infrastructure, and vehicles, you can use the data collected to quickly respond to changing conditions, identify operational trends, and improve forecasting to maximize the performance and efficiency of the larger system. Data can be collected in near real-time (sub-second), hourly, or daily, depending on your business requirements, connectivity, and budget. You can use [AWS IoT Greengrass](https://aws.amazon.com/greengrass/) to process, store, and analyze this data in the cloud or at the edge. AWS IoT Greengrass is software that brings memory and compute closer to the physical asset. It provides local compute and machine learning inference capabilities at the device level. As you connect more assets, the data set becomes richer, providing a high fidelity view of your operations. This gives you deeper insight and the ability to identify larger trends for data-driven decision making.

Many industrial organizations look to a future where distributed systems continuously analyze data from assets using machine learning (ML) models and autonomously adapting inputs and outputs to ensure maximum performance. The first step is to connect assets and begin collecting, processing, and analyzing data at the edge and in the cloud.

## Define business objectives and outcomes

I recommend that you start small. Clearly define the business problem and identify measurable outcomes (for example, improving equipment up-time by 60% or reducing food spoilage across a cold chain distribution network by 85%). You might want to use user stories, which are a feature of agile development. For example,” As an airport operations manager, I want to see the real-time status of the baggage conveyor system so that I can quickly resolve faults and ensure the system is 99.9% operational.” You can iterate on and scale the solution using AWS IoT services.

## Case Study: Maximizing Wind Turbine performance

Consider a wind farm with turbines distributed over hundreds of square miles. Energy operators must generate consistent, profitable power output by maximizing turbine performance. Although routine scheduled maintenance is key to continuous operations, this does not prevent turbines from going offline before the next maintenance window. Energy operators want to maximize wind farm output by providing operations staff near real-time data into turbine mechanical performance. Access to this real-time data makes it possible to schedule maintenance based on current conditions and initiate immediate action when a fault is detected. Then alerts can be sent automatically to onsite engineers through a mobile device.

## Building a solution

You can break down these business requirements into the following solution objectives:

*   Obtain continuous, near real-time sensor data from turbines at the edge.
*   Securely transport and ingest data into the AWS cloud.
*   Process the data and automatically trigger mobile alerts on anomalous values (for example, if vibration (Hz) becomes too high, then investigate a potential fault).
*   Visualize the turbines’ metrics for operational staff and provide a permanent historical data set for ad-hoc analysis, research, and future input for a machine learning model.

1. Collecting data from industrial equipment at the Edge

Data can be collected from industrial equipment (in this example, wind turbines) through sensors. Sensor data is transmitted from equipment to an [AWS IoT Greengrass core](https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-core.html) running on a [qualified gateway device](https://devices.amazonaws.com/search?page=1&sv=gg). AWS IoT Greengrass provides local compute using [AWS Lambda](https://aws.amazon.com/lambda/) functions, a local message broker to support local device-to-device and device-to-cloud communication, device shadows for managing equipment state, and machine learning inference using deep-learning models trained in [Amazon SageMaker](https://aws.amazon.com/sagemaker/). You can use the AWS Management Console or [AWS CLI](https://aws.amazon.com/cli/) to create Lambda functions and configure AWS IoT Greengrass, which simplifies the development and deployment process. Wind turbines can communicate their state to AWS IoT Greengrass using [MQTT](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html), a lightweight publish-subscribe messaging protocol. Messages are routed between publishers and subscribers by the local message broker through [topics](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html), which serve as bi-directional communication channels between devices, AWS IoT Greengrass, and AWS IoT Core.

AWS IoT Greengrass also provides support for [OPC-UA](https://docs.aws.amazon.com/greengrass/latest/developerguide/opcua.html) and local [protocol conversion](https://docs.aws.amazon.com/greengrass/latest/developerguide/modbus-protocol-adapter-connector.html) using prebuilt connectors or Lambda functions. This makes it possible for you to interface with equipment that uses proprietary protocols like Modbus RTU without deploying a new sensing overlay. There is an advantage to adding more sensors, however. You create a new acquisition layer separate from your existing operational technology systems, which can improve security and make it possible for your development team to innovate more quickly. If you are using microcontrollers to build new sensing devices, you can use [Amazon FreeRTOS](https://aws.amazon.com/freertos/), an open-source real-time operating system (RTOS) that supports many hardware architectures and provides libraries to simplify network connectivity to AWS IoT, manage security, and execute over-the-air (OTA) updates. Amazon FreeRTOS is free to download under the MIT license and can be modified without permission of AWS. You can find a list of qualified microcontrollers [here](https://devices.amazonaws.com/).

2. Securely transporting and ingesting data to AWS

MQTT messages are transmitted securely from AWS IoT Greengrass to [AWS IoT Core](https://aws.amazon.com/iot-core/), the entry point to the AWS Cloud. AWS IoT Core provides several key features:

*   An identity service for authentication and authorization.
*   A device gateway to connect devices.
*   A message broker to route messages.
*   A rules engine to trigger actions.
*   A device shadow to enable applications to interact with devices when they are offline.
*   A registry that enables automatic device registration.

You use AWS IoT Core to define a turbine as a thing and generate an [X.509 device certificate](https://docs.aws.amazon.com/iot/latest/developerguide/managing-device-certs.html), private key, and root CA certificate that is placed on the device for authentication. You then use [AWS IoT Device Management](https://docs.aws.amazon.com/iot/latest/developerguide/thing-groups.html) to define a thing group and add metadata for each turbine, such as model number, GPS coordinates, manufacturer. This metadata is useful for data analysis, remote device management, and implementation of future predictive maintenance. You also configure a rule in the [AWS IoT rules engine](https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules.html) to route all messages received from site1 (turbines/site1/#) in the MQTT topic stream to [AWS IoT Analytics](https://aws.amazon.com/iot-analytics/) for processing.

This diagram shows wind turbine publishing state information to a topic. AWS IoT Greengrass is subscribed to the topic and forwards it to AWS IoT Core when connectivity is available. AWS IoT Greengrass can communicate with turbines locally even if connectivity to the cloud is lost. To reduce latency and data transmission costs, you could also process and analyze data locally at the edge using a Lambda function running on AWS IoT Greengrass.

![](https://jamesprichardson.com/content/images/2021/12/image.png)

3. Processing data, detecting anomalies and triggering action

There is now a steady stream of data being ingested from the turbines. You need to process and “clean” this data before you analyze it and make it available to upstream applications. AWS IoT Analytics can be used for the collection, processing, and storage of unstructured data, or alternatively, this data can be analyzed using SQL or Jupyter Notebooks. The turbine data received from AWS IoT Core is processed in stages as follows:

*   A **channel** receives data from AWS IoT Core, filtered by the MQTT topic **turbines/site1/#**
*   A **pipeline** processes messages from this channel. You can enrich the data through filters and transformations. It is here that you can trigger action on the incoming data by invoking a Lambda function that contains logic to send a message to Amazon PinPoint. If vibration readings exceed 100 Hz, which indicates an impending failure, [Amazon PinPoint](https://aws.amazon.com/pinpoint/) generates an SMS notification to onsite field engineers. You can enrich the stream with data from other sources (for example, weather data) for context and clean any noisy or corrupted messages or false readings that might hinder analysis.
*   Finally, the data is stored in a **time-series** data set where it is ready for visualization or analysis using SQL and Jupyter Notebooks. You can also schedule an automatic refresh of the query every 15 minutes to ensure that the latest data is available to upstream applications.

![](https://jamesprichardson.com/content/images/2021/12/image-1.png)

4. Analyzing and Visualizing data to enable insight and action

With a processed data set, you can use [Amazon QuickSight](https://aws.amazon.com/quicksight/), a business intelligence (BI) service, to directly connect to the data set in AWS IoT Analytics and create a dashboard that displays turbine speed, vibration, and current power output for a specific turbine model. This allows you to analyze asset performance over time, gain new insights, and then identify actions to improve efficiency and maximize wind farm performance. These insights increase in value as more devices are connected by providing more fidelity to the data. For example, you might overlay weather, GPS location, and vibration data to discover that turbine models of the exact same type perform quite differently due to local weather conditions or temperature. This insight can then be used to modify turbine mechanical attributes as needed and also serve as a continuous feedback loop into product development of new turbines, with the goal of maximizing energy output and overall turbine performance.

![](https://jamesprichardson.com/content/images/2021/12/image-2.png)

You can also create ad-hoc queries on the AWS IoT Analytics data store using SQL. For example, to locate turbines with potential mechanical issues, you can use the following query to search for vibration readings of more than 100 Hz:

```sql
SELECT thingid, vibration, datetime FROM turbine_datastore WHERE vibration > 100
```

The output is displayed in the AWS IoT console and is provided as a downloadable CSV file.

For more sophisticated analysis and prediction, you can also use the integrated Jupyter Notebooks with ML analytics and Python. There are several sample notebooks available for use cases. For example, the Contextual Anomaly Detection notebook can be used to locate sensor readings outside the normal operating range. You can also download Jupyter notebooks locally onto your system.

## System Architecture

Here is the high-level architecture of the solution. It shows how AWS services are used for asset condition monitoring:

![](https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/03/06/iot-asset-condition-architecture.png)

## Cost Optimization

In this solution, data is continuously streamed to AWS for processing and analysis. Because excessive data transport from remote locations could increase costs, you can use Lambda functions deployed onto an AWS IoT Greengrass core and send only the data required by your use case to the cloud. For example, in this scenario, you could implement logic in a Lambda function that analyzes data streaming from turbines locally (at the edge) and send a message to AWS IoT Core only if an anomalous value is detected.

## The value of Partners for IoT projects

IoT projects span many devices and technology domains—from sensors and actuators to sophisticated machine learning algorithms. The AWS network of consulting partners specializes in guiding you through the development of production scale IoT solutions. Your business outcome improvements are limited only by your team’s imagination. To learn more, visit the [AWS Partner Network](https://aws.amazon.com/partners/).

## Next Steps

In this post, I have shown how you can use Amazon FreeRTOS and AWS IoT Greengrass to sense physical attributes at the edge, transport data to AWS using AWS IoT Core, and perform analysis using AWS IoT Analytics. Asset condition monitoring is the first step toward a larger digital transformation strategy. AWS IoT services make continuous innovation possible through its integration with other services. Now that data is flowing from your equipment, you can modify this architecture to use machine learning and enable predictive maintenance. This helps you maximize asset performance and predict failures ahead of time.

[](/re-invent-a-builders-guide/)

[Share](https://www.facebook.com/sharer.php?u=https://jamesprichardson.com/aws-iot-asset-condition-monitoring/) [Tweet](https://twitter.com/intent/tweet?url=https://jamesprichardson.com/aws-iot-asset-condition-monitoring/&text=Monitoring%20'things'%20with%20AWS%20IoT)

[](/a-framework-for-architecting-iot-solutions/)

