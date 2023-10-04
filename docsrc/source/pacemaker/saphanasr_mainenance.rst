.. _saphanasr_maint:

#####################
SAPHanaSR Mainenance 
#####################

man SAPHanaSR_maintenance_example

Scale UP Administrativer TakeOver
*********************************

 .. code-block:: 

    # crm_mon -1r
    # SAPHanaSR-showAttr
    # crm configure show | grep cli-
    # cs_clusterstate -i
    # crm resource move msl_SAPHana_SLE_HDB10 force
    # cs_clusterstate -i
    # SAPHanaSR-showAttr
    # crm resource clear msl_SAPHana_SLE_HDB10
    # SAPHanaSR-showAttr
    # cs_clusterstate -i



SAP HANA takeover by using SAP tools (ScaleOut / ScaleUp)
**********************************************************

 .. code-block:: 
         
    1. On either node
      # crm_mon -1r
      # SAPHanaSR-showAttr
      # crm configure show | grep cli-
      # cs_clusterstate -i

    If everything looks fine, proceed.
      # crm resource maintenance msl_SAPHana_SLE_HDB10
      # crm_mon -1r

    2. On the HANA primary master nameserver (e.g. node11)
      # su - sleadm
      ~> sapcontrol -nr 10 -function StopSystem HDB
      ~> sapcontrol -nr 10 -function GetSystemInstanceList

    Only proceed after you made sure the HANA primary is down!

    3. On the HANA secondary master nameserver (e.g. node21)
      # su - sleadm
      ~> hdbnsutil -sr_takeover
      ~> HDBsettings.sh systemReplicationStatus.py; echo RC:$?
      ~> HDBsettings.sh landscapeHostConfiguration.py; echo RC:$?
    
    If everything looks fine, proceed.
    
    4. On the former HANA primary master nameserver, now future secondary master nameserver (e.g. node11)
      ~> hdbnsutil -sr_register --remoteHost=node21 --remoteInstance=10 --replicationMode=sync --name=site2 --operationMode=logreplay
      ~> sapcontrol -nr 10 -function StartSystem HDB
      ~> exit
    
    5. On the new HANA primary master nameserver (e.g. node21)
      ~> HDBsettings.sh systemReplicationStatus.py; echo RC:$?
      ~> HDBsettings.sh ./landscapeConfigurationStatus.py; echo RC:$?
      ~> exit
    
    If everything looks fine, proceed.
    6. On either node
      # cs_clusterstate -i
      # crm resource refresh msl_SAPHana_SLE_HDB10
      # crm resource maintenance msl_SAPHana_SLE_HDB10 off
      # SAPHanaSR-showAttr
      # crm_mon -1r
      # cs_clusterstate -i


Overview on maintenance procedure for Linux, HANA remains running, on pacemaker-2.0
*************************************************************************************

 .. code-block::

   1. Check status of Linux cluster and HANA.
   2. Set the Linux cluster into maintenance mode, on either node.
      # crm maintenance on
   3. Stop Linux Cluster on all nodes. Make sure to do that on all nodes.
      # crm cluster stop
   4. Perform Linux maintenance.
   5. Start Linux cluster on all nodes. Make sure to do that on all nodes.
      # crm cluster start
   6. Let Linux cluster detect status of HANA resource, on either node.
      # crm resource refresh cln_...
      # crm resource refresh msl_...
   7. Set cluster ready for operations, on either node.
      # crm maintenance off
   8. Check status of Linux cluster and HANA.

Overview on update procedure for the SAPHanaSR and SAPHanaSR-ScaleOut package.
*******************************************************************************
This procedure can be used to update RAs, HANA HADR provider hook scripts and related tools while HANA and Linux cluster stay online. 

 .. code-block:: 
    
     1. Check status of Linux cluster and HANA
     2. Set resources SAPHana (or SAPHanaController) and SAPHanaTopology to maintenance.
     3. Update RPM on all cluster nodes.
     4. Reload HANA HADR provider hook script on both sites.
     5. Refresh resources SAPHana (or SAPHanaController) and SAPHanaTopology.
     6. Set resources SAPHana (or SAPHanaController) and SAPHanaTopology from maintenance to managed.
     7. Check status of Linux cluster and HANA

