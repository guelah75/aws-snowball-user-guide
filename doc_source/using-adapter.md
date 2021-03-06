--------

This guide is for the Snowball \(50 TB or 80 TB of storage space\)\. If you are looking for documentation for the Snowball Edge, see the [AWS Snowball Edge Developer Guide](http://docs.aws.amazon.com/snowball/latest/developer-guide/whatisedge.html)\.

--------

# Using the Amazon S3 Adapter for Snowball<a name="using-adapter"></a>

Following, you can find an overview of the Amazon S3 Adapter for Snowball, which allows you to programmatically transfer data between your on\-premises data center and the Snowball using Amazon S3 REST API actions\. This Amazon S3 REST API support is limited to a subset of actions, meaning that you can use the subset of supported Amazon S3 AWS CLI commands or one of the AWS SDKs to transfer data\.

If your solution uses the AWS SDK for Java version 1\.11\.0 or newer, you must use the following `S3ClientOptions`:

+ `disableChunkedEncoding()` – Indicates that chunked encoding is not supported with the adapter\.

+ `setPathStyleAccess(true)` – Configures the adapter to use path\-style access for all requests\.

For more information, see [Class S3ClientOptions\.Builder](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/s3/S3ClientOptions.Builder.html) in the Amazon AppStream SDK for Java\.


+ [Starting the Amazon S3 Adapter for Snowball](#start-adapter)
+ [Getting the Status of a Snowball Using the Adapter](#get-status-using-adapter)
+ [Options for the Amazon S3 Adapter for Snowball](using-adapter-options.md)
+ [Using the Adapter with Amazon S3 Commands for the AWS CLI](using-adapter-cli.md)
+ [Supported REST API Actions](using-adapter-supported-api.md)

## Starting the Amazon S3 Adapter for Snowball<a name="start-adapter"></a>

To use the Amazon S3 Adapter for Snowball, start it in a terminal on your workstation and leave it running while transferring data\.

**Note**  
Because the computer workstation from which or to which you make the data transfer is considered to be the bottleneck for transferring data, we highly recommend that your workstation be a powerful computer\. It should be able to meet high demands in terms of processing, memory, and networking\. For more information, see [Workstation Specifications](specifications.md#workstationspecs)\.

Before you start the adapter, you need the following information:

+ **The Snowball's IP address** – Providing the IP address of the Snowball when you start the adapter tells the adapter where to send your transferred data\. You can get this IP address from the E Ink display on the Snowball itself\.

+ **The job's manifest file** – The manifest file contains important information about the job and permissions associated with it\. Without it, you won't be able to access the Snowball\. It's an encrypted file that you can download after your job enters the `WithCustomer` status\. The manifest is decrypted by the unlock code\. You can get the manifest file from the console, or programmatically from calling a job management API action\.

+ **The job's unlock code** – The unlock code a string of 29 characters, including 4 dashes\. It's used to decrypt the manifest\. You can get the unlock code from the [AWS Snowball Management Console](transfer-data.md#unlockappliance), or programmatically from the job management API\.

+ **Your AWS credentials** – Every interaction with the Snowball is signed with the AWS Signature Version 4 algorithm\. For more information, see [Signature Version 4 Signing Process](http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)\. When you start the Amazon S3 Adapter for Snowball, you specify the AWS credentials that you want to use to sign this communication\. By default, the adapter uses the credentials specified in the *home directory*/\.aws/credentials file, under the \[default\] profile\. For more information on how this Signature Version 4 algorithm works locally with the Amazon S3 Adapter for Snowball, see [Authorization with the Amazon S3 API Adapter for Snowball](auth-adapter.md)\.

Once you have the preceding information, you're ready to start the adapter on your workstation\. The following procedure outlines this process\.

**To start the adapter**

1. Open a terminal window on the workstation with the installed adapter\.

1. Navigate to the directory where you installed the snowball\-adapter\-*operating\_system* directory\.

1. Navigate to the bin subdirectory\.

1. Type the following command to start the adapter: `./snowball-adapter -i Snowball IP address -m path to manifest file -u 29 character unlock code`\.

**Note**  
If you don't specify any AWS credentials when starting the adapter, the default profile in the `home directory/.aws/credentials` file is used\.

The Amazon S3 Adapter for Snowball is now started on your workstation\. Leave this terminal window open while the adapter runs\. If you're going to use the AWS CLI to transfer your data to the Snowball, open another terminal window and run your AWS CLI commands from there\.

## Getting the Status of a Snowball Using the Adapter<a name="get-status-using-adapter"></a>

You can get a Snowball’s status by initiating a `HEAD` request to the Amazon S3 Adapter for Snowball\. You receive the status response in the form of an XML document\. The XML document includes storage information, latency information, version numbers, and more\. 

You can't use the AWS CLI or any of the AWS SDKs to retrieve status in this\. However, you can easily test a `HEAD` request over the wire by running a `curl` command on the adapter, as in the following example\.

```
curl -H "Authorization Header" -X HEAD http://192.0.2.0:8080
```

**Note**  
When requesting the status of a Snowball, you must add the authorization header\. For more information, see [Signing AWS Requests with Signature Version 4](http://docs.aws.amazon.com/general/latest/gr/sigv4_signing.html)\.

An example of the XML document that this request returns follows\.

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Status xsi:schemaLocation="http://s3.amazonaws.com/doc/2006-03-01/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <snowballIp>192.0.2.0</snowballIp>
    <snowballPort>8080</snowballPort>
    <snowballId>012EXAMPLE01</snowballId>
    <totalSpaceInBytes>77223428091904</totalSpaceInBytes>
    <freeSpaceInBytes>77223428091904</freeSpaceInBytes>
    <jobId>JID850f06EXAMPLE-4EXA-MPLE-2EXAMPLEab00</jobId>
    <snowballServerVersion>1.0.1</snowballServerVersion>
    <snowballServerBuild>2016-08-22.5729552357</snowballServerBuild>
    <snowballAdapterVersion>Version 1.0</snowballAdapterVersion>
    <snowballRoundTripLatencyInMillis>1</snowballRoundTripLatencyInMillis>
</Status>
```