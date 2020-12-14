# SV4dotNet
Broadcom SV for Dot Net

The SV4dotnet library provides a .NET developer with the CA Service
Virtualization's capabilities, as well as creating the newly available BlazeVSE
mock services. The .NET developer can create, update, and deploy the SV/BlazeVSE
virtual/mock services directly from your .NET code.

## Supported Virtual Services

The following are the supported virtual services.

*SV*:

-   RR pairs

-   Zip file

-   Swagger

-   WSDL

-   VSI/VSM

*BlazeVSE:*

-   MAR file

The SV4dotnet library integrates with the Nunit test framework that is part of
the MS Visual Studio IDE.

![Nunit Test Framework](https://github.com/CA-DevTest/SV-DOT-NET/blob/ea63fa08045210c56d6bc5150566a4ce07a25074/readme-images/Setup-nunit-tests.png)

## Architecture

The SV4dotnet library is designed to be a seamless integration in your MS Visual
Studio IDE infrastructure.

![SV4dotnet Architecture](https://github.com/CA-DevTest/SV-DOT-NET/blob/7bf51b18779eb3a4b3fc0b6bd4ff533239ee25fa/readme-images/architecture.png)

This architecture provides with the flexibility to create virtual services in
either SV or BlazeVSE. Each developer can then update the supplied config
(SV.config and BlazeVSE.config) files with their corresponding username/password
(key API in case of BlazeVSE), as well as the SV/blazemeter URLs that will be
used during the virtual services CRUD calls.

## SV4dotnet Runtime Configuration

The following steps are necessary for a successful integration of SV4dotnet into
your .NET solution development environment.

1.  Create Nunit tests project as part of your .NET solution

2.  Download and add dependencies for the SV4dotnet components

    1.  SV4dotNet.dll

    2.  SV.config

    3.  BlazeVSE.config

    4.  Log4net.config

3.  Install the following dependencies (you can download them via the nuget
    manager that is part of MS Visual Studio). You would need to work closely
    with your SCM team/dev management to make sure that you download and install
    the following packages:

    1.  Authenticated Encryption 2.0.0

    2.  Log4net 2.0.11

    3.  Newtonsoft.Json 12.0.3

    4.  SystemConfiguration.ConfigurationManager 4.7.0

4.  Add a setup method as part of your testcases. This method will load the
    config (SV or BlazeVSE)

5.  Add the necessary methods to create, update, and delete the virtual services
    (as needed)

Here is an example in how to setup the Nunit tests

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
namespace DemoTestProject
{
	public class DemoTest
	{
		// Below is the url used with the application under test, update this to the virtual 
		// service endpoint
		// once you define it in the configuration file.
		string url = http://demoserver.abc.com:8001
		
		// configure the below parameters
		Configuration config;
		string user;
		string password;
		AppSettingsSection section;
		
		// configure the below method to execute before the test methods
		[OneTimeSetup]
		
		public void Setup()
		{
			// The below method will load the config that you would like to use 
			// (Ex: for DevTest - SV.config and for Blazemeter - BlazeVSE.config)
			// Alternatively to read the below parameter during runtime, you can add it
			// to the runsettings file and read it from the TestContext.Parameters
			config = SVClient.SVLoadConfig(TestContext.Parameters["SV.config"]);
			section = (AppSettingsSection)config.GetSection("appSettings");
			user = section.Settings["username"].Value;
			password = SVScrambler.SVScramble(config);
			
			// Call method below to create the virtual service
			// functionality can be "Create", "Update", and "Delete" depending
			// on what you plan to do
			//the path1 and path2 arguements are associated with files 
			// for creation of the SV virtual service
			//the file arguement is associated with the files for the creation 
			// of Blazemeter VSE virtual service
			HttpResponseMessage response = SVClient.SVProcessRequests(config, user, 
				password, "Update", "wgdemoservice", null, 
				"Pet-onhold-req.txt", "Pet-onhold-rsp.txt", null).Result;
		}
		
		[Test, Order(1)]
        public void FindPetsbyStatusTest_newentry()
        {
            HttpResponseMessage response = PetStore.findPetsbyStatus(url, "newentry").Result;
            Assert.AreEqual(HttpStatusCode.OK, response.StatusCode);
        }
	}
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is an example of test.runsettings file that will would allow you to load
the desired config file dynamically.

\<?xml version="1.0" encoding="utf-8"?\>

\<RunSettings\>

\<!-- Parameters used by tests at runtime --\>

\<TestRunParameters\>

\<Parameter name="SVConfig" value="BlazeVSE.Config" /\>

\<!-- \<Parameter name="SVConfig" value="SV.Config" /\> --\>

\</TestRunParameters\>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
</RunSettings>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Special note

The SV4dotnet also includes support for secured SV portal (https). If you are
going to be working in said secured SV environment, you will need to obtain a
signed certificate from the SV admin team. The certificate will have a suffix of
“CER” or “CRT” and be X-509 complaint for SV4dotnet to use it and be able to
establish a trusted connection with the secured SV portal.
