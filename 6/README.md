***DISCLAIMER: THIS IS A WORK IN PROGRESS***
```
shortname: 6/INVOKE
name: Invoke compute and storage services
type: Standard
status: Raw
editor: Ahmed Abdullah <ahmed@oceanprotocol.com>
```
***Lightweight On-Chain Workflow Management System (salpchain)***

<!--ts-->
### Table of Contents

   * [Lightweight on-chain WMS](#salpchain)
      * [Change Process](#change_process)
      * [Language](#language)
      * [Motivation](#motivation)
      * [Introduction to WMS](#wms)
      * [DAG based WMSs](#dag)
      * [Architecture](#architecture)
      * [Implementation](#implementation)
      * [Use Cases](#usecases)
      * [Reference](#reference)
      * [Copyright Waiver](#copyright-waiver)
      
## Lightweight On-chain WMS (salpchain) <a name="salpchain"></a>

Ocean protocol works as a substrate for data consumption on blockchain, in which enables users to consume multiple 
services that could be on-chain or off-chain. Moreover theses services SHOULD preserve the privacy models such as zero-knowledge and 
at the same time provide some sort of service integrity such as verifiable computation. In this OEP, we are going to 
introduce a proposal for how to implement lightweight on-chain workflow management system [salpchain](https://en.wikipedia.org/wiki/Salp). 
Salpchain SHOULD enable users to execute workflows, orchestrate services, and manage tasks with minimal amount 
of gas on Ethereum blockchain.


## 1. Change Process <a name="change_process"></a>

This document is governed by the [2/COSS](../2/README.md) (COSS).

## 2. Language <a name="language"></a>
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", 
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in 
[BCP 14](https://tools.ietf.org/html/bcp14) \[[RFC2119](https://tools.ietf.org/html/rfc2119)\] 
\[[RFC8174](https://tools.ietf.org/html/rfc8174)\] when, and only when, they appear in all capitals, 
as shown here.

## 3. Motivation <a name="motivation"></a>
As a data scientist, you have complex pipelines. They maybe just consuming dataset
but also they might include different computation, storage and networking services. As a result, 
you will need to schedule these services in terms of workflow. Salpchain is an on-chain workflow 
management system in Ocean protocol that manages the scheduling and execution of workflows on Ethereum blockchain.

## 4. Introduction to WMS <a name="wms"></a>

Workflows are a sequence of tasks where each task scheduled and triggered by an event. Mainly we use workflows to handle complex 
pipelines and big data analysis. For instance as shown below we have a some sort of simple workflow which start with data acquisition, then
processing this dataset, monitor processing progress and finally report results. 

![Typical Workflow](imgs/typicalWorkflow.png)

Examples for workflows such as Extract, Transform, Load (ETL), also in testing we can use a 
workflow in order to orchestrate automated testing, and one of the most complex workflows coming from Bioinformatics and genomics workflows. The following
table shows different workflow management systems (WMS) and summarize main points for each workflow:

***Open Source WMS***

WMS    |                                         Language                    | Deployed By | Web UI     
--------------|--------------------------------------------------------------|-------------|---------
[Luigi](https://luigi.readthedocs.io/en/stable/)| Python                     | Spotify     |   Yes
[Apache Airflow](http://airflow.incubator.apache.org/) | Python              | Airbnb      |    Yes
[Apache Oozie](http://oozie.apache.org/)               | Java                | Cloudera, Yahoo, Microsoft | Yes
[Taskflow](https://wiki.openstack.org/wiki/TaskFlow)   | Python              | OpenStack   | No

***Bioinformatics WMS***

WMS    |                                         Language                    | Deployed By | Web UI     
--------------|--------------------------------------------------------------|-------------|---------
[Galaxy Engine](https://galaxyproject.org/) | Python                     | Johns Hopkins    |   Yes
[Cosmos](https://cosmos.hms.harvard.edu/) | Python              | Harvard Medical School    |   No
[Apache Taverna](https://taverna.incubator.apache.org/)| Java                | University of Manchester | No
[Cromwell Engine](http://cromwell.readthedocs.io/en/develop/)   |       Scala        | Broad Institute   | No



## 5. DAG based WMSs <a name="dag"></a>

***TODO***

## 6. Architecture <a name="architecture"></a>

***TODO***

## 7. Implementation <a name="implementation"></a>

***TODO***

## 8. Use Cases <a name="usecases"></a>

***TODO***

## 9. Reference <a name="reference"></a>

***TODO***

## 10. Copyright Waiver  <a name="copyright-waiver"></a>
To the extent possible under law, the person who associated CC0 with this work has waived all copyright and related or neighboring rights to this work.