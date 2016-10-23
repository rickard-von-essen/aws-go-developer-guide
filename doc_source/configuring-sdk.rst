.. Copyright 2010-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.


#################
SDK Configuration
#################


.. meta::
   :description: Configure the |sdk-go| to specify which credentials to use and to which region to send requests.
   :keywords: configuration, specify region, region, credentials, proxy

In the |sdk-go|, you can configure settings for service clients,
such as the log level and maximum number of retries. Most settings are
optional; however, for each service client, you must specify a region
and your credentials. The SDK uses these values to send requests to the
correct AWS region and sign requests with the correct credentials. You
can specify these values as part of a session or as environment
variables.

.. contents::
   :local:
   :depth: 1

.. _specifying-the-region:

Specifying the Region
=====================

When you specify the region, you specify where to send requests, such as
``us-west-2`` or ``us-east-1.`` The SDK does not select a default
region. For a list of regions for each service, see |regions-and-endpoints|_ 
in the |AWS-gr|.

To specify the region, set the ``AWS_REGION`` environment variable or
specify it in a session. If you do both, the SDK will always use the
region you specified in the session.

The following examples show you how to configure the environment
variable.

**Linux, OS X, or Unix**

.. code-block:: bash

    $ export AWS_REGION=us-west-2

**Windows**

.. code-block:: sh

    > set AWS_REGION=us-west-2

The following snippet specifies the region in a session:

.. code-block:: go

    sess, err := session.NewSession(&aws.Config{Region: aws.String("us-west-2")})

    
.. _specifying-credentials:

Specifying Credentials
======================

The |sdk-go| requires credentials (an access key and secret access
key) to sign requests to AWS. You can specify your credentials in
several different locations, depending on your particular use case. For
information about obtaining credentials, see :doc:`Setting Up <setting-up>`.

When you initialize a new service client without providing any
credential arguments, the SDK uses the :sdk-go-api-deep:`default credential provider
chain <aws/defaults/#CredChain>` to find AWS credentials. The SDK uses the first provider 
in the chain that returns credentials without an error. The default provider chain
looks for credentials in the following order:

1. Environment variables.
2. Shared credentials file.
3. If your application is running on an |EC2| instance, |IAM| role for |EC2|.

The SDK detects and uses the built-in providers automatically, without
requiring manual configurations. For example, if you use |IAM| roles for
|EC2| instances, your applications will automatically use the
instance's credentials. You don't need to manually configure credentials
in your application.

As a best practice, AWS recommends that you specify credentials in the
following order:

1. Use |IAM| roles for |EC2| (if your application is running on an
   |EC2| instance).

   |IAM| roles provide applications on the instance temporary security
   credentials to make AWS calls. |IAM| roles provide an easy way to
   distribute and manage credentials on multiple |EC2| instances.

2. Use a shared credentials file.

   This credentials file is the same one used by other SDKs and the |CLI|
   If you're already using a shared credentials file, you can use
   it for this purpose, too.

3. Use environment variables.

   Setting environment variables is useful if you're doing development
   work on a machine other than an |EC2| instance.

4. Hard-code credentials (not recommended).

   Hard-coding credentials in your application can make it difficult to
   manage and rotate those credentials. Use this method for small
   personal scripts or testing purposes only. Do not submit code with
   credentials to source control.

|IAM| Roles for |EC2| Instances
-------------------------------

If you are running your application on an |EC2| instance, you can
use the instance's :ec2-ug:`IAM role <iam-roles-for-amazon-ec2>`
to get temporary security credentials to make calls to AWS.

If you have configured your instance to use |IAM| roles, the SDK will use
these credentials for your application automatically. You don't need to
manually specify these credentials.

Shared Credentials File
-----------------------

A credential file is a plaintext file that contains your access keys.
The file must be on the same machine on which you're running your
application. The file must be named :file:`credentials` and located in the
:file:`.aws/` folder in your home directory. The home directory can vary by
operating system. In Windows, you can refer to your home directory by
using the environment variable :code:`%UserProfile%`. In Unix-like systems, you
can use the environment variable :code:`$HOME` or :code:`~` (tilde).

If you already use this file for other SDKs and tools (like the |CLI|), 
you don't need to change anything to use the files in this SDK. If
you use different credentials for different tools or applications, you
can use *profiles* to configure multiple access keys in the same
configuration file.

Creating the Credentials File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you do not have a shared credentials file (:file:`.aws/credentials`), you
can use any text editor to create one in your home directory. Add the
following content to your credentials file, replacing
:code:`<YOUR_ACCESS_KEY_ID>` and :code:`<YOUR_SECRET_ACCESS_KEY>` with your
credentials:

.. code-block:: ini

    [default]
    aws_access_key_id = <YOUR_ACCESS_KEY_ID>
    aws_secret_access_key = <YOUR_SECRET_ACCESS_KEY>

The :code:`[default]` heading defines credentials for the default profile,
which the SDK will use unless you configure it to use another profile.

You can also use temporary security credentials by adding the session
tokens to your profile, as shown in the following example:

.. code-block:: ini

    [temp]
    aws_access_key_id = <YOUR_TEMP_ACCESS_KEY_ID>
    aws_secret_access_key = <YOUR_TEMP_SECRET_ACCESS_KEY>
    aws_session_token = <YOUR_SESSION_TOKEN>

Specifying Profiles
~~~~~~~~~~~~~~~~~~~

You can include multiple access keys in the same configuration file by
associating each set of access keys with a profile. For example, in your
credentials file, you can declare multiple profiles:

.. code-block:: ini

    [default]
    aws_access_key_id = <YOUR_DEFAULT_ACCESS_KEY_ID>
    aws_secret_access_key = <YOUR_DEFAULT_SECRET_ACCESS_KEY>
    
    [test-account]
    aws_access_key_id = <YOUR_TEST_ACCESS_KEY_ID>
    aws_secret_access_key = <YOUR_TEST_SECRET_ACCESS_KEY>
    
    [prod-account] 
    ; work profile
    aws_access_key_id = <YOUR_PROD_ACCESS_KEY_ID>
    aws_secret_access_key = <YOUR_PROD_SECRET_ACCESS_KEY>

By default, the SDK checks the :code:`AWS_PROFILE` environment variable to
determine which profile to use. If no :code:`AWS_PROFILE` variable is set,
the SDK uses the default profile.

If you have an application named ``myapp`` that uses the SDK, you can
run it with the test credentials by setting the variable to
``test-account myapp``, as shown in the following command:

.. code-block:: sh

    $ AWS_PROFILE=test-account myapp

You can also use the SDK to select a profile by specifying
:code:`os.Setenv("AWS_PROFILE", test-account)` before constructing any
service clients or by manually setting the credential provider, as shown
in the following example:

.. code-block:: go

    sess := session.New(&aws.Config{
        Region:      aws.String("us-west-2"),
        Credentials: credentials.NewSharedCredentials("", "test-account"),
    })

In addition, checking if your credentials have been found is fairly easy.

.. code-block:: go

    _, err := sess.Config.Credentials.Get()

If :code:`ChainProvider` is being used, set :code:`CredentialsChainVerboseErrors` to 
:code:`true` in the session config.

.. note::
   If you specify credentials in environment variables, the SDK
   will always use those credentials, no matter which profile you specify.

Environment Variables
---------------------

By default, the SDK detects AWS credentials set in your environment and
uses them to sign requests to AWS. That way you don't need to manage
credentials in your applications.

The SDK looks for credentials in the following environment variables:

-  :code:`AWS_ACCESS_KEY_ID`
-  :code:`AWS_SECRET_ACCESS_KEY`
-  :code:`AWS_SESSION_TOKEN` (optional)

The following examples show how you configure the environment variables.

**Linux, OS X, or Unix**

.. code-block:: bash

    $ export AWS_ACCESS_KEY_ID=YOUR_AKID
    $ export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY
    $ export AWS_SESSION_TOKEN=TOKEN

**Windows**

.. code-block:: sh

    > set AWS_ACCESS_KEY_ID=YOUR_AKID
    > set AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY
    > set AWS_SESSION_TOKEN=TOKEN

Hard-Coded Credentials in an Application (not recommended)
----------------------------------------------------------

.. warning::
   Do not embed credentials inside an application. Use this
   method only for testing purposes.

You can hard-code credentials in your application by passing the access
keys to a configuration instance, as shown in the following snippet:

.. code-block:: go

    sess := session.New(&aws.Config{
        Region:      aws.String("us-west-2"),
        Credentials: credentials.NewStaticCredentials("AKID", "SECRET_KEY", "TOKEN"),
    })

Other Credentials Providers
---------------------------

The SDK provides other methods for retrieving credentials in the
:code:`aws/credentials` package. For example, you can retrieve temporary
security credentials from AWS Security Token Service or credentials from
encrypted storage. For more information, see :sdk-go-api-deep:`Credentials 
<aws/credentials/>`.

.. _configuring-a-proxy:

Configuring a Proxy
===================

If you cannot directly connect to the Internet, you can use Go-supported
environment variables (``HTTP_PROXY``) or create a custom HTTP client to
configure your proxy. Use the
:sdk-go-api-deep:`Config.HTTPClient <aws/#Config.WithHTTPClient>` 
struct to specify a custom HTTP client. For more information about how
to create an HTTP client to use a proxy, see the
`Transport <http://golang.org/pkg/net/http/#Transport>`_ struct in
the Go ``http`` package.


