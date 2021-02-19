- Solr does not start after adding Ranger.

**Solution:**

It is recommended to add Solr and Ranger services together with the IBM Spectrum Scale service at the time of initial CDP Private Cloud Base cluster creation. However, if Ranger and Solr were added later, following workaround is needed for Solr to start properly.

1. Log in to the Cloudera Manager console.
2. While adding the Solr service, set the **ZooKeeper ZNode** parameter to solr-infra in the configuration wizard or if you had already added Solr but Solr does not come up properly, click **Solr** > **Configuration** and search for ZNode and set the value of the Solr configuration **ZooKeeper ZNode** parameter to solr-infra.
3. Ensure that Kerberos checkbox is enabled in the Solr configuration.
4. Continue to add the Solr and Ranger services. Skip if you have already added the services.
5. After adding Ranger, the Solr service changes its name to CDP-INFRA-SOLR.
6. Restart Solr and Ranger.
7. Ensure that Solr and Ranger have started successfully. Do not proceed unless Solr and Ranger appear healthy.





If solr is pre-installed, solr znode changes to /solr-infra when you add Ranger or Atlas on a cluster with Ranger.

**Solution:**

1. Rename znode back to /solr.
2. Renaming the znode causes an Atlas initialization issue. To address this issue, restart Atlas on correct znode as follows:
   1. Stop Atlas.
   2. Go to Atlas Service Actions, click **Initialize Atlas** > **Start Atlas**.