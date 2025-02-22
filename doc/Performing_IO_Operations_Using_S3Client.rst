========================================
Performing IO Operations Using S3 Client
========================================

If you want to use the provided OVA to test CORTX and observe activity in its GUI, you should have already followed the `instructions <CORTX_on_Open_Virtual_Appliance.rst>`_ to install and configure the OVA.  If you have not yet done so, please do that before following the rest of the instructions on this page.


Prerequisites
=============

Verify that the S3 server endpoint is reachable using the following command:

    ::
  
        ping <s3 endpoint or public data IP>

- If yes then go ahead and configure the s3 client. See `Procedure <#Procedure>`__
- If not then recheck the client VM network configuration.  See `Troubleshoot Virtual Network </doc/troubleshoot_virtual_network.rst>`__

Procedure
=========

#. Install and configure the the AWS CLI Client. See `AWS CLI <https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html>`__.

    **Note:** You can install the AWS CLI either on the same OVA or you can create a new VM and install it. If you have installed the AWS CLI on the separate VMs, then the following entry must be added in the */etc/hosts* file of the new VM:

    ::

        <<Public Data IP>> s3.seagate.com sts.seagate.com iam.seagate.com sts.cloud.seagate.com   

#. To create an S3 user account:
    
    1. On the CORTX GUI, go to the **Manage > S3 Account > Add New Account**.
    
        .. image:: images/add_s3_user.png
    
    2. Enter the required details and click **Create**

#. After the new account is created, make sure to copy the *Access Key* and the *Secret Key*. The Access Key and Screte Key will also be available in the CSV file which is downloaded when you add the new account.

#. Create a bucket on CORTX GUI:
   
    1. Log-in to CORTX GUI  using S3 account credentials at https://<management IP>:28100/#/
    
    2. Go to the **Manage tab > Bucket > Create Bucket**.
    
    3. Enter the Bucket name and click **Create**.

        .. image:: images/Create-Bucket.png

#. Configure AWS credentials using following commands:
    
    1. Run the following command to configure the AWS configure: 
    
        ::
           
            aws configure

        .. image::  images/aws-configure.png

    2. Set the endpoint URL:
        
        ::

            aws configure set s3.endpoint_url https://s3.seagate.com

    3. Set the API endpoint URL: 
            
        ::
        
            aws configure set s3api.endpoint_url https://s3.seagate.com

    4. If you are using a separate VM for client, copy the S3 certificate from ova to client location:

        ::
        
            scp root@<ova-server-ip>:/opt/seagate/cortx/provisioner/srv/components/s3clients/files/ca.crt /etc/ssl/stx-s3-clients/s3/ca.crt

    5. Set the AWS certificate path:
        
        1. If you are using OVA VM as a client: 
    
            ::
        
                aws configure set default.ca_bundle /opt/seagate/cortx/provisioner/srv/components/s3clients/files/ca.crt

        2. If you are using a separate VM for client: 
    
            ::
            
                aws configure set default.ca_bundle /etc/ssl/stx-s3-clients/s3/ca.crt

#. Perform IO operation:

    1. Verify buckect created using CORTX GUI:

        ::
        
            aws s3 ls --endpoint-url=http://s3.seagate.com

        .. image::  images/verify-bkt.png

    2. Run the following command to create a new large file to upload:

        ::
        
            dd if=/dev/zero of=/tmp/1G bs=1G count=1

        .. image::  images/create-file.png

    3. Upload the file in the created bucket:

        ::
        
            aws s3 cp /tmp/1G s3://ova-bucket --endpoint-url https://s3.seagate.com

        .. image::  images/upload.png

    4. Downlonad same file from the bucket:

        ::
        
            aws s3api get-object --bucket ova-bucket --key 1G /tmp/read-1G
    
        .. image::  images/aws-download.png

        **Note:** For more information on CORTX supported S3 APIs, see `S3 Supported API <https://github.com/Seagate/cortx-s3server/blob/main/docs/s3-supported-api.md>`__.

#. Navigate to the Dashboard tab in your CORTX GUI, change 'Metric 1' to 'throughput_write' and you should see activity in the dashboard.
   
    .. image:: images/PG.PNG