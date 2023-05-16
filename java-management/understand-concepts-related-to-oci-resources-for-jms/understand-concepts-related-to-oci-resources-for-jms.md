# Understand concepts related to Oracle Cloud Infrastructure Resources and Services for Java Management Service

## Introduction

This lab walks you through key concepts that need to be understood before setting up your Oracle Cloud Infrastructure (OCI) environment for Java Management Service (JMS).

Estimated Time: 30 minutes

### Objectives

In this lab, you will:

- Learn about the important concepts regarding OCI resources in preparation for setting up the OCI environment for JMS to operate. These OCI resources include compartment, tag, user group, dynamic group, policies and fleet.
- Understand the relationships between these OCI resources and services, including logging, metrics and object storage, and how JMS leverages these relationships to allow you to observe and manage Java SE usage in your enterprise.

## Task 1: Understand concepts related to Oracle Cloud Infrastructure Resources and Services for Java Management Service

Before the set up of Oracle Cloud Infrastructure (OCI) resources and services for Java Management Service (JMS), it is important to understand the concepts behind them and how JMS taps on them to operate.

This diagram illustrates the purpose of OCI resources and services in JMS with details of each resource and service explained below:

![image of resources and services in jms](images/resources-and-services-in-jms.png)

1. User Group:

    - See [Managing Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/groups/managinggroups.htm) for its definition and details.
    - Certain policies are applied to the user group, such as "Fleet Managers", to allow these users to manage OCI resources and services required in JMS, such as creating a fleet and managing tag namespaces.

2. Dynamic Group:

    - See [Managing Dynamic Groups](https://docs.oracle.com/en-us/iaas/Content/Identity/dynamicgroups/managingdynamicgroups.htm) for its definition and details.
    - The creation of a dynamic group is important as it allows for policies to be applied to the group of compute instances and management agents, to allow them to communicate information back to the fleet through OCI service endpoints.

3. Policies:

    - See [Managing Policies](https://docs.oracle.com/en-us/iaas/Content/Identity/policieshow/how-policies-work.htm) for its definition and details.
    - To enable basic features, policies are applied to three categories:

        A. User group "FLEET\_MANAGERS":
        ```
        <copy>
        ALLOW GROUP FLEET_MANAGERS TO MANAGE fleet IN COMPARTMENT Fleet_Compartment
        ALLOW GROUP FLEET_MANAGERS TO MANAGE management-agents IN COMPARTMENT Fleet_Compartment
        ALLOW GROUP FLEET_MANAGERS TO MANAGE management-agent-install-keys IN COMPARTMENT Fleet_Compartment
        ALLOW GROUP FLEET_MANAGERS TO MANAGE tag-namespaces IN TENANCY
        ALLOW GROUP FLEET_MANAGERS TO MANAGE instance-family IN COMPARTMENT Fleet_Compartment
        ALLOW GROUP FLEET_MANAGERS TO MANAGE log-groups IN COMPARTMENT Fleet_Compartment
        ALLOW GROUP FLEET_MANAGERS TO MANAGE log-content IN COMPARTMENT Fleet_Compartment
        ALLOW GROUP FLEET_MANAGERS TO READ METRICS IN COMPARTMENT Fleet_Compartment
        ALLOW GROUP FLEET_MANAGERS TO READ instance-agent-plugins IN COMPARTMENT Fleet_Compartment
        </copy>
        ```

        These policy statements allow users in this user group to work with JMS such as creating a fleet, creating required tag, reading information about fleet and installing management agent on the instance.

        B. Dynamic group "JMS\_DYNAMIC\_GROUP":
        ```
        <copy>
        ALLOW DYNAMIC-GROUP JMS_DYNAMIC_GROUP TO MANAGE management-agents IN COMPARTMENT Fleet_Compartment
        ALLOW DYNAMIC-GROUP JMS_DYNAMIC_GROUP TO MANAGE instances IN COMPARTMENT Fleet_Compartment
        ALLOW DYNAMIC-GROUP JMS_DYNAMIC_GROUP TO MANAGE log-content IN COMPARTMENT Fleet_Compartment
        ALLOW DYNAMIC-GROUP JMS_DYNAMIC_GROUP TO USE tag-namespaces IN TENANCY
        ALLOW DYNAMIC-GROUP JMS_DYNAMIC_GROUP TO USE METRICS IN COMPARTMENT Fleet_Compartment
        </copy>
        ```

        These policy statements allow compute instances and management agents in this dynamic group to communicate information to OCI.

        C. Service "javamanagementservice":
        ```
        <copy>
        ALLOW SERVICE javamanagementservice TO MANAGE log-groups IN COMPARTMENT Fleet_Compartment
        ALLOW SERVICE javamanagementservice TO MANAGE log-content IN COMPARTMENT Fleet_Compartment
        ALLOW SERVICE javamanagementservice TO MANAGE metrics IN COMPARTMENT Fleet_Compartment WHERE target.metrics namespace='java_management_service'
        ALLOW SERVICE javamanagementservice TO READ instances IN tenancy
        ALLOW SERVICE javamanagementservice TO INSPECT instance-agent-plugins IN tenancy
        ALLOW SERVICE javamanagementservice TO USE management-agent-install-keys IN COMPARTMENT Fleet_Compartment
        </copy>
        ```

        These policy statements allow JMS to work with services including monitoring metrics, logs and object storage in the fleet.

    When advanced features are enabled during the fleet creation, additional dynamic groups and policies would be automatically created in OCI.
    This allows JMS, the management agents and the compute instances to access object storage in the fleet for uploading of information such as crypto event analysis report and JDK Flight Recording data.

4. Compartment:

    - See [Managing Compartments](https://docs.oracle.com/en-us/iaas/Content/Identity/compartments/managingcompartments.htm) for its definition and details.

    - A compartment can contain many JMS related resources such as fleets, managed instances, management agents, jms tag namespace with tag definition, logs, object storage and monitoring metrics.

5. Fleet:

    - A fleet is the primary collection with which you interact when using JMS. It contains Managed Instances that share rules and policies. See [Managed Instance](https://docs.oracle.com/en-us/iaas/jms/doc/getting-started-java-management-service.html#GUID-141F2F39-8078-481A-ACE7-65792E314ABB) for the definition of a Managed Instance.

    - It is created in a compartment and contain information about the Managed Instances such as logs, object storage and monitoring metrics.

6. Tag and tag namespace:

    - See [Managing tags and tag namespaces](https://docs.oracle.com/en-us/iaas/Content/Tagging/Tasks/managingtagsandtagnamespaces.htm) for its definition and details.

    - For JMS to operate, a specific tag key with the tag key definition '**fleet\_ocid**' has to be created in a specific tag namespace '**jms**' in the **root compartment**. This allows each management agent to be tagged with defined tags jms.fleet_ocid and tag value "ocid1.jmsfleet....." of the specific fleet it is associated with, after the management agent is installed on the Managed Instances.

    - With the tag link, these agents can send data back to the fleet in OCI, which JMS can process and manage. Users can then observe and monitor the Java related data e.g. usage tracking associated with each Managed Instance.

JMS also tap on the following OCI services to generate logs, object storage information and monitoring metrics of the fleet for the users to view. These are the details of these services:

7. Logging Service:

    - These logs are event logs contributed by Java Management Service and by the service plugins deployed on the management agent of the host machine.

    - All the different log objects will belong to the fleet log group.

    - Each log object will contain its own category of logs e.g. Inventory log will contain logs of the scanning of Java installations in the Managed Instance.

    - Note that the Inventory log is crucial as a fleet cannot be successfully created without it.

    To access the fleet logs, you may click on the respective log object displayed on the fleet main page.

    ![image of log configuration in fleet overview page](images/fleet-log-configuration.png)

    To view the logs in detail, click on the drop down arrow.

    ![image of fleet inventory log page](images/fleet-inventory-log.png)

    You should be able to see the details of the log.

    ![image of jvm installation log](images/jvm-installation-log.png)

    - See [Logging in JMS](https://docs.oracle.com/en-us/iaas/jms/doc/appendix.html#GUID-559AECF8-4FAD-45CC-AE3B-69CA0DC9BDDD) to learn more details about the logs managed by JMS.

8. Object Storage Service

    - This is required for JMS advanced features, such as JDK Flight Recording (JFR) and Crypto event analysis, where the recordings are uploaded to object storage before JMS retrieves the recording to perform crypto event analysis. The resulting analysis report is then uploaded to object storage.

    - The object storage bucket serves as the storage for resources such as JFR files and analysis reports.

    To access the object storage associated with the fleet, click on the object storage bucket link.

    ![image of object storage in fleet main page](images/fleet-object-storage.png)

    Scroll down the object storage bucket details page.

    ![image object storage bucket details](images/object-storage-bucket-details.png)

    You should be able to see the objects created by JMS advanced features, such as analysis reports.

    ![image of analysis report object](images/analysis-report-object.png)

9. Monitoring Metrics Service

    - This service processes the information generated by JMS and displays it as graphs of Managed Instances, Java runtimes and applications.

    - You may create your own alarms for notifications based on these metrics.

    You may view the fleet metrics on the fleet overview page, by clicking **Metrics** under Resources.

    ![image of metrics in fleet main page](images/fleet-metrics.png)

    - See [Java Management Metrics](https://docs.oracle.com/en-us/iaas/jms/doc/appendix.html#GUID-E7908768-AE97-4BB9-85CB-17A1BD87A271) to learn more about metrics in JMS.

You may now **proceed to the next lab.**

## Learn More

* Refer to the [Getting Started with Java Management Service](https://docs.oracle.com/en-us/iaas/jms/doc/getting-started-java-management-service.html) for more details.

## Acknowledgements

- **Author** - Sherlin Yeo, Java Management Service
- **Last Updated By** - Sherlin Yeo, May 2023
