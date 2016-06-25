---
title: Replication error 1753 There are no more endpoints available from the endpoint mapper
H1: na
ms.custom: 
  - x
ms.prod: windows-server-threshold
ms.reviewer: na
ms.service: active-directory
ms.suite: na
ms.technology: 
  - active-directory-domain-services
ms.tgt_pltfrm: na
ms.assetid: f7022f0d-0099-410c-8178-c654e624bc42
---
# Replication error 1753 There are no more endpoints available from the endpoint mapper
This topic explains symptoms, causes and how to resolve Active Directory replication error 8524 The DSA operation is unable to proceed because of a DNS lookup failure.  
  
-   [Symptoms](../../identity/ad-ds/Replication-error-1753-There-are-no-more-endpoints-available-from-the-endpoint-mapper.md#BKMK_Symptoms)  
  
-   [Cause](../../identity/ad-ds/Replication-error-1753-There-are-no-more-endpoints-available-from-the-endpoint-mapper.md#BKMK_Cause)  
  
-   [Resolutions](../../identity/ad-ds/Replication-error-1753-There-are-no-more-endpoints-available-from-the-endpoint-mapper.md#BKMK_Resolutions)  
  
-   [More information](../../identity/ad-ds/Replication-error-1753-There-are-no-more-endpoints-available-from-the-endpoint-mapper.md#BKMK_MoreInfo)  
  
##  <a name="BKMK_Symptoms"></a> Symptoms  
 This article describes symptoms, cause and resolution steps for Active Directory operations that fail with Win32 error 1753: "There are no more endpoints available from the endpoint mapper."  
  
1.  DCDIAG reports that the Connectivity test, Active Directory Replications test or KnowsOfRoleHolders test has failed with error 1753: “There are no more endpoints available from the endpoint mapper."  
  
    ```  
    Testing server: <site>\<DC Name>  
    Starting test: Connectivity  
    * Active Directory LDAP Services Check  
    * Active Directory RPC Services Check  
    [<DC Name>] DsBindWithSpnEx() failed with error 1753,  
    There are no more endpoints available from the endpoint mapper..  
    Printing RPC Extended Error Info:  
    Error Record 1, ProcessID is <process ID> (DcDiag)   
    System Time is: <date> <time>  
    Generating component is 2 (RPC runtime)  
    Status is 1753: There are no more endpoints available from the endpoint mapper. Detection location is 500  
    NumberOfParameters is 4  
    Unicode string: ncacn_ip_tcp  
    Unicode string: <source DC object GUID>._msdcs.contoso.com  
    Long val: -481213899  
    Long val: 65537  
    Error Record 2, ProcessID is 700 (DcDiag)   
    System Time is: <date> <time>  
    Generating component is 2 (RPC runtime)  
    Status is 1753: There are no more endpoints available from the endpoint mapper.  
    NumberOfParameters is 1  
    Unicode string: 1025  
    [Replications Check,<DC Name>] A recent replication attempt failed:  
    From <source DC> to <destination DC>  
    Naming Context: <DN path of directory partition>  
    The replication generated an error (1753):  
    There are no more endpoints available from the endpoint mapper.   
    The failure occurred at <date> <time>.  
    The last success occurred at <date> <time>.  
    3 failures have occurred since the last success.  
    The directory on <DC name> is in the process.  
    of starting up or shutting down, and is not available.  
    Verify machine is not hung during boot.  
  
    ```  
  
2.  REPADMIN.EXE reports that replication attempt has failed with status 1753.  
  
     REPADMIN commands that commonly cite the 1753 status include but are not limited to:  
  
    |||  
    |-|-|  
    |-   REPADMIN \/REPLSUM<br />-   REPADMIN \/SHOWREPL|-   REPADMIN \/SHOWREPS<br />-   REPADMIN \/SYNCALL|  
  
     Sample output from "REPADMIN \/SHOWREPS" depicting inbound replication from CONTOSO\-DC2 to CONTOSO\-DC1 failing with the "replication access was denied" error is shown below:  
  
    ```  
    Default-First-Site-Name\CONTOSO-DC1  
    DSA Options: IS_GC   
    Site Options: (none)  
    DSA object GUID: b6dc8589-7e00-4a5d-b688-045aef63ec01  
    DSA invocationID: b6dc8589-7e00-4a5d-b688-045aef63ec01  
    ==== INBOUND NEIGHBORS ==========  
    DC=contoso,DC=com  
    Default-First-Site-Name\CONTOSO-DC2 via RPC  
    DSA object GUID: 74fbe06c-932c-46b5-831b-af9e31f496b2  
    Last attempt @ <date> <time> failed, result 1753 (0x6d9):  
    There are no more endpoints available from the endpoint mapper.  
    <#> consecutive failure(s).  
    Last success @ <date> <time>.  
  
    ```  
  
3.  The **Check Replication Topology** command in Active Directory Sites and Services returns "There are no more endpoints available from the endpoint mapper."  
  
     Right\-clicking on the connection object from a source DC and choosing **Check Replication Topology** fails with “There are no more endpoints available from the endpoint mapper.” The on\-screen error message is shown below:  
  
     Dialog title text: Check Replication Topology  
  
     Dialog message text:  
  
     The following error occurred during the attempt to contact the domain controller: There are no more endpoints available from the endpoint mapper.  
  
4.  The **Replicate now** command in Active Directory Sites and Services returns "there are no more endpoints available from the endpoint mapper."  
  
     Right\-clicking on the connection object from a source DC and choosing **Replicate now** fails with "There are no more endpoints available from the endpoint mapper.” The on\-screen error message is shown below:  
  
     Dialog title text: Replicate Now  
  
     Dialog message text: The following error occurred during the attempt to synchronize naming context \<%directory partition name%\> from Domain Controller \<Source DC\> to Domain Controller \<Destination DC\>:  
  
     There are no more endpoints available from the endpoint mapper.  
  
     The operation will not continue  
  
5.  NTDS KCC, NTDS General or Microsoft\-Windows\-ActiveDirectory\_DomainService events with the \-2146893022 status are logged in the Directory Services log in Event Viewer.  
  
     Active Directory events that commonly cite the \-2146893022 status include but are not limited to:  
  
    |Event ID|Event Source|Event String|  
    |--------------|------------------|------------------|  
    |1655|NTDS General|Active Directory attempted to communicate with the following global catalog and the attempts were unsuccessful.|  
    |1925|NTDS KCC|The attempt to establish a replication link for the following writable directory partition failed.|  
    |1265|NTDS KCC|An attempt by the Knowledge Consistency Checker \(KCC\) to add a replication agreement for the following directory partition and source domain controller failed.|  
  
##  <a name="BKMK_Cause"></a> Cause  
 The diagram below shows the RPC workflow starting with the registration of the server application with the RPC Endpoint Mapper \(EPM\) in step 1 to the passing of data from the RPC client to the client application in step 7.  
  
 \<ADDS\_RPCWorkflow\>  
  
 Steps 1 through 7 map to the following operations:  
  
1.  Server app registers its endpoints with the RPC Endpoint Mapper \(EPM\)  
  
2.  Client makes an RPC call \(on behalf of a user, OS or application initiated operation\)  
  
3.  Client side RPC contacts the target computers EPM and ask for the endpoint to complete the client call  
  
4.  Server Machine's EPM responds with an endpoint  
  
5.  Client side RPC contacts the server app  
  
6.  Server app executes the call, returns the result to the client RPC  
  
7.  Client side RPC passes the result back to the client app  
  
 Failure 1753 is generated by a failure between steps \#3 and \#4. Specifically, error 1753 means that the RPC client \(destination DC\) was able to contact the RPC Server \(source DC\) over port 135 but the EPM on the RPC Server \(source DC\) was unable to locate the RPC application of interest and returned server side error 1753. The presence of the 1753 error indicates that the RPC client \(destination DC\) received the server side error response from the RPC Server \(AD replication source DC\) over the network.  
  
 Specific root causes for the 1753 error include:  
  
1.  The server app never started \(i.e. Step \#1 in the "more information" diagram located above was never attempted\).  
  
2.  The server app started but there was some failure during initialization that prevented it from registering with the RPC Endpoint Mapper \(i.e. Step \#1 in the "more information" diagram above was attempted but failed\).  
  
3.  The server app started but subsequently died. \(i.e. Step \#1 in the "more information" diagram above was completed successfully, but was undone later because the server died\).  
  
4.  The server app manually unregistered its endpoints \(similar to 3 but intentional. Not likely but included for completeness.\)  
  
5.  The RPC client \(destination DC\) contacted a different RPC server than the intended one due to a Name to IP mapping error in DNS, WINS or host\/Lmhosts file.  
  
 Error 1753 is NOT caused by:  
  
-   A lack of network connectivity between the RPC client \(destination DC\) and RPC Server \(source DC\) over port 135  
  
-   A lack of network connectivity between the RPC server \(source DC\) using port 135 and the RPC client \(destination DC\) over the ephemeral port.  
  
-   A password mismatch or the inability by the source DC to decrypt a Kerberos encrypted packet  
  
##  <a name="BKMK_Resolutions"></a> Resolutions  
  
1.  **Verify that the service registering its service with the endpoint mapper has started**  
  
     For Windows 2000 and Windows Server 2003 DCs: ensure that the source DC is booted into normal mode.  
  
     For Windows Server 2008 or Windows Server 2008 R2: from the console of the source DC, start Services Manager \(services.msc\) and verify that the **Active Directory Domain Services** service is running.  
  
2.  **Verify that RPC client \(destination DC\) connected to the intended RPC Server \(source DC\)**  
  
     All DCs in a common Active Directory forest register a domain controller CNAME record in the \_msdcs.\<forest root domain\> DNS zone regardless of what domain they reside in within the forest. The DC CNAME record is derived from the **objectGUID** attribute of the NTDS Settings object for each domain controller.  
  
     When performing replication\-based operations, a destination DC queries DNS for the source DCs CNAME record. The CNAME record contains the source DC fully qualified computer name which is used to derive the source DCs IP address via DNS client cache lookup, Host \/ LMHost file lookup, host A \/ AAAA record in DNS, or WINS.  
  
     Stale NTDS Settings objects and bad name\-to\-IP mappings in DNS, WINS, Host and LMHOST files may cause the RPC client \(destination DC\) to connect to the wrong RPC Server \(Source DC\). Furthermore, the bad name\-to\-IP mapping may cause the RPC client \(destination DC\) to connect to a computer that does not even have the RPC Server Application of interest \(the Active Directory role in this case\) installed. \(Example: a stale host record for DC2 contains the IP address of DC3 or a member computer\).  
  
     Verify that the objectGUID for the source DC that exists in the destination DCs copy of Active Directory matches the source DC objectGUID stored in the source DCs copy of Active Directory. If there is a discrepancy, use repadmin \/showobjmeta on the ntds settings object to see which one corresponds to last promotion of the source DC \(hint: compare date stamps for the NTDS Settings object create date from \/showobjmeta against the last promotion date in the source DCs dcpromo.log file. You may have to use the last modify \/ create date of the DCPROMO.LOG file itself\). If the object GUIDs are not identical, the destination DC likely has a stale NTDS Settings object for the source DC whose CNAME record refers to a host record with a bad name to IP mapping.  
  
     On the destination DC, run IPCONFIG \/ALL to determine which DNS Servers the destination DC is using for name resolution:  
  
    ```  
    c:\>ipconfig /all  
    ```  
  
     On the destination DC, run NSLOOKUP against the source DCs fully qualified DC CNAME record:  
  
    ```  
    c:\>nslookup -type=cname <fully qualified cname of source DC> <destination DCs primary DNS Server IP >  
    c:\>nslookup -type=cname <fully qualified cname of source DC> <destination DCs secondary DNS Server IP>  
    ```  
  
     Verify that the IP address returned by NSLOOKUP "owns" the host name \/ security identity of the source DC:  
  
    ```  
    C:\>NBTSTAT –A <IP address returned by NSLOOKUP in the step above>  
    ```  
  
     or  
  
     Log onto the console of the source DC, run "IPCONFIG" from the CMD prompt and verify that the source DC owns the IP address returned by the NSLOOKUP command above  
  
     Check for stale \/ duplicate host to IP mappings in DNS  
  
    ```  
    NSLOOKUP –type=hostname <single label hostname of source DC> <primary DNS Server IP on destination DC>  
    NSLOOKUP –type=hostname <single label hostname of source DC> <secondary DNS Server IP on destination DC>  
  
    NSLOOKUP –type=hostname <fully qualified computer name of source DC> <primary DNS Server IP on destination DC>  
    NSLOOKUP –type=hostname <fully qualified computer name of source DC> <secondary DNS Server IP on dest. DC>  
    ```  
  
     If invalid IP addresses exist in host records, investigate whether DNS scavenging is enabled and properly configured.  
  
     If the tests above or a network trace doesn’t show a name query returning an invalid IP address, consider stale entries in HOST files, LMHOSTS files and WINS Servers. Note that DNS Servers can also be configured to perform WINS fallback name resolution.  
  
3.  **Verify that the server application \(Active Directory et al\) has registered with the endpoint mapper on the RPC server \(source DC\)**  
  
     Active Directory uses a mix of well\-known and dynamically registered ports. This table lists well known ports and protocols used by Active Directory domain controllers.  
  
    |RPC Server Application|Port|TCP|UDP|  
    |----------------------------|----------|---------|---------|  
    |DNS server|53|X|X|  
    |Kerberos|88|X|X|  
    |LDAP server|389|X|X|  
    |Microsoft\-DS|445|X|X|  
    |LDAP SSL|636|X|X|  
    |Global Catalog Server|3268|X||  
    |Global Catalog Server|3269|X||  
  
     Well\-known ports are NOT registered with the endpoint mapper.  
  
     Active Directory and other applications also register services that receive dynamically assigned ports in the RPC ephemeral port range. Such RPC server applications are dynamically assigned TCP ports between 1024 and 5000 on Windows 2000 and Windows Server 2003 computers and ports between 49152 and 65535 range on Windows Server 2008 and Windows Server 2008 R2 computers. The RPC port used by replication can be hard\-coded in the registry using the steps documented in [KB article 224196](http://support.microsoft.com/kb/224196). Active Directory continues to register with the EPM when configured to use a hard coded port.  
  
     Verify that the RPC Server application of interest has registered itself with the RPC endpoint mapper on the RPC Server \(the source DC in the case of AD replication\).  
  
     There are a number of ways to accomplish this task but one is to install and run PORTQRY from an admin privileged CMD prompt on the console of the source DC using the syntax:  
  
    ```  
    c:\>portquery -n <source DC> -e 135 >file.txt  
    ```  
  
     In the portqry output, note the port numbers dynamically registered by the "MS NT Directory DRS Interface" \(UUID \= 351...\) for the **ncacn\_ip\_tcp protocol**. The snippet below shows sample portquery output from a Windows Server 2008 R2 DC and the UUID \/ protocol pair specifically used by Active Directory highlighted in **bold**:  
  
    ```  
    UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface  
    ncacn_np:CONTOSO-DC01[\\pipe\\lsass]   
    UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface  
    ncacn_np:CONTOSO-DC01[\\PIPE\\protected_storage]   
    UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface  
    ncacn_ip_tcp:CONTOSO-DC01[49156]   
    UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface  
    ncacn_http:CONTOSO-DC01[49157]   
    UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface  
    ncacn_http:CONTOSO-DC01[6004]  
    ```  
  
4.  Other possible ways to resolve this error:  
  
    1.  Verify that the source DC is booted in normal mode and that the OS and DC role on the source DC have fully started.  
  
    2.  Verify that the Active Directory Domain Service is running. If the service is currently stopped or was not configured with default startup values, reset the default startup values, reboot the modified DC then retry the operation.  
  
    3.  Verify that the startup value and service status for RPC service and RPC Locator is correct for OS version of the RPC Client \(destination DC\) and RPC Server \(source DC\). If the service is currently stopped or was not configured with default startup values, reset the default startup values, reboot the modified DC then retry the operation.  
  
         In addition, ensure that the service context matches default settings listed in the following table.  
  
        |Service|Default status \(Startup type\) in Windows Server 2003 and later|Default status \(Startup type\) in Windows Server 2000|  
        |-------------|----------------------------------------------------------------------|------------------------------------------------------------|  
        |Remote Procedure Call|Started \(Automatic\)|Started \(Automatic\)|  
        |Remote Procedure Call Locator|Null or Stopped \(Manual\)|Started \(Automatic\)|  
  
    4.  Verify that the size of the dynamic port range has not been constrained. The Windows Server 2008 and Windows Server 2008 R2 NETSH syntax to enumerate the RPC port range is shown below:  
  
        ```  
        >netsh int ipv4 show dynamicport tcp  
        >netsh int ipv4 show dynamicport udp  
        >netsh int ipv6 show dynamicport tcp  
        >netsh int ipv6 show dynamicport udp  
        ```  
  
    5.  Verify that hard coded port definitions defined in KB 224196 fall within the dynamic port range for source DCs OS version.  
  
         Review [KB article 224196](http://support.microsoft.com/kb/224196) and ensure that the hard coded port falls within the ephemeral port range for the source DC's operating system version.  
  
    6.  Verify that the ClientProtocols key exists under HKLM\\Software\\Microsoft\\Rpc and contains the following 5 default values:  
  
        ```  
        ncacn_http REG_SZ rpcrt4.dll  
        ncacn_ip_tcp REG_SZ rpcrt4.dll  
        ncacn_nb_tcp REG_SZ rpcrt4.dll  
        ncacn_np REG_SZ rpcrt4.dll  
        ncacn_ip_udp REG_SZ rpcrt4.dll  
        ```  
  
##  <a name="BKMK_MoreInfo"></a> More information  
 **Example of a bad name to IP mapping causing RPC error 1753 vs. \-2146893022: the target principal name is incorrect**  
  
 The contoso.com domain consists of \\\\DC1 and \\\\DC2 with IP addresses x.x.1.1 and x.x.1.2. The host "A" \/ "AAAA" records for \\\\DC2 are correctly registered on all of the DNS Servers configured for \\\\DC1. In addition, the HOSTS file on \\\\DC1 contains an entry mapping DC2s fully qualified hostname to IP address x.x.1.2. Later, DC2's IP address changes from X.X.1.2 to X.X.1.3 and a new member computer is joined to the domain with IP address x.x.1.2. AD Replication attempts triggered by the **Replicate now** command in Active Directory Sites and Services snap\-in fails with error 1753 as shown in the trace below:  
  
```  
F# SRC    DEST    Operation   
1 x.x.1.1 x.x.1.2 ARP:Request, x.x.1.1 asks for x.x.1.2  
2 x.x.1.2 x.x.1.1 ARP:Response, x.x.1.2 at 00-13-72-28-C8-5E  
3 x.x.1.1 x.x.1.2 TCP:Flags=......S., SrcPort=50206, DstPort=DCE endpoint resolution(135)  
4 x.x.1.2 x.x.1.1 ARP:Request, x.x.1.2 asks for x.x.1.1  
5 x.x.1.1 x.x.1.2 ARP:Response, x.x.1.1 at 00-15-5D-42-2E-00  
6 x.x.1.2 x.x.1.1 TCP:Flags=...A..S., SrcPort=DCE endpoint resolution(135)  
7 x.x.1.1 x.x.1.2 TCP:Flags=...A...., SrcPort=50206, DstPort=DCE endpoint resolution(135)  
8 x.x.1.1 x.x.1.2 MSRPC:c/o Bind: UUID{E1AF8308-5D1F-11C9-91A4-08002B14A0FA} EPT(EPMP)   
9 x.x.1.2 x.x.1.1 MSRPC:c/o Bind Ack: Call=0x2 Assoc Grp=0x5E68 Xmit=0x16D0 Recv=0x16D0   
10 x.x.1.1 x.x.1.2 EPM:Request: ept_map: NDR, DRSR(DRSR) {E3514235-4B06-11D1-AB04-00C04FC2DCD2} [DCE endpoint resolution(135)]  
11 x.x.1.2 x.x.1.1 EPM:Response: ept_map: 0x16C9A0D6 - EP_S_NOT_REGISTERED  
  
```  
  
 At frame **10**, the destination DC queries the source DCs end point mapper over port 135 for the Active Directory replication service class UUID E351...  
  
 In frame **11**, the source DC, in this case a member computer that does not yet host the DC role and therefore has not registered the E351... UUID for the Replication service with its local EPM responds with symbolic error EP\_S\_NOT\_REGISTERED which maps to decimal error 1753, hex error 0x6d9 and friendly error "there are no more endpoints available from the endpoint mapper".  
  
 Later, the member computer with IP address x.x.1.2 gets promoted as a replica "MayberryDC" in the contoso.com domain. Again, the **Replicate now** command is used to trigger replication but this time fails with the on\-screen error "The target principal name is incorrect." The computer whose network adapter is assigned the IP address x.x.1.2 *is* a domain controller, is currently booted into normal mode and has registered the E351... replication service UUID with its local EPM but it does not own the name or security identity of DC2 and cannot decrypt the Kerberos request from DC1 so the request now fails with error "The target principal name is incorrect." The error maps to decimal error \-2146893022 \/ hex error 0x80090322.  
  
 Such invalid host\-to\-IP mappings could be caused by stale entries in host \/ lmhost files, host A \/ AAAA registrations in DNS, or WINS.  
  
 Summary: This example failed because an invalid host\-to\-IP mapping \(in the HOST file in this case\) caused the destination DC to resolve to a "source" DC that did not have the Active Directory Domain Services service running \(or even installed for that matter\) so the replication SPN was not yet registered and the source DC returned error 1753. In the second case, an invalid host\-to\-IP mapping \(again in the HOST file\) caused the destination DC to connect to a DC that had registered the E351... replication SPN but that source had a different hostname and security identity than the intended source DC so the attempts failed with error \-2146893022: The target principal name is incorrect.  
  
## See Also  
 [Troubleshooting Active Directory operations that fail with error 1753: There are no more endpoints available from the endpoint mapper.](http://support.microsoft.com/kb/2089874)   
 [KB article 839880 Troubleshooting RPC Endpoint Mapper errors using the Windows Server 2003 Support Tools from the product CD](http://support.microsoft.com/kb/839880)   
 [KB article 832017 Service overview and network port requirements for the Windows Server system](http://support.microsoft.com/kb/832017/)   
 [KB article 224196 Restricting Active Directory replication traffic and client RPC traffic to a specific port](http://support.microsoft.com/kb/224196/)   
 [KB article 154596 How to configure RPC dynamic port allocation to work with firewalls](http://support.microsoft.com/kb/154596)   
 [How RPC Works](http://msdn.microsoft.com/library/aa373935\(VS.85\).aspx)   
 [How the Server Prepares for a Connection](http://msdn.microsoft.com/library/aa373938\(VS.85\).aspx)   
 [How the Client Establishes a Connection](http://msdn.microsoft.com/library/aa373937\(VS.85\).aspx)   
 [Registering the Interface](http://msdn.microsoft.com/library/aa375357\(VS.85\).aspx)   
 [Making the Server Available on the Network](http://msdn.microsoft.com/library/aa373974\(VS.85\).aspx)   
 [Registering Endpoints](http://msdn.microsoft.com/library/aa375255\(VS.85\).aspx)   
 [Listening for Client Calls](http://msdn.microsoft.com/library/aa373966\(VS.85\).aspx)   
 [How the Client Establishes a Connection](http://msdn.microsoft.com/library/aa373937\(VS.85\).aspx)   
 [Restricting Active Directory replication traffic and client RPC traffic to a specific port](http://support.microsoft.com/kb/224196)   
 [SPN for a Target DC in AD DS](http://msdn.microsoft.com/library/dd207688\(PROT.13\).aspx)