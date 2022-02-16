---
title: Teradici CAS Cliff Notes
description: A guide deployed to quickly set-up CAS on a workstation without having to consult official documentation
author: chad-m-smith
tags: CAS, CAS-Plus, Windows
date_published: 2022-02-15
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

While Teradici's [official documentation](https://docs.teradici.com/find/product/cloud-access-software) is the recommended for deploying CAS, this “cliff noted” guide is a condensed version which highlighting recommendations, troubleshooting and provides insight 'in-line' as a part of this step-by-step deployment process. This removes the need to open several guides and ‘cuts the fat’ while illustrating the most common deployment scenarios seen out in the field over my past two years at Teradici.

**A brief description of deployment:**
1. Define network topology and ports for communcations between host and client systems
1. Review of system requirement to ensure compatibility
1. Administrator of workstation logs into Teradici portal to download CAS agent.
1. Install CAS agent on workstation
1. Configure CAS agent
1. End-user logs into Teradici portal to download CAS client on their remote system

# Defining Network topology and ports for communications

The assumption is that a remote worker outside a firewall will need access to a workstation internally. In this situation, Teradici CAS requires a publicly accessible IP/FQDN and port(s) opened from the location (home/corporate) that the a workstation resides in. A fallback is an VPN connection, but VPNs have been known to hamper the performance of the PCoIP protocol. PCoIP protocol encrypts all traffic between the client and host devices by default. For larger deployment, where multiple workers will be establishing CAS connections, Teradici does offer a connection gateway called [CAS Manager](https://www.teradici.com/web-help/cas_manager_as_a_service/?_ga=2.12859831.1699787421.1637180645-1894139970.1589168508). Its a fully featured connection broker that has a plethora of features but won't be configuring it as a part of this quick start guide.

+ Both Client and Host installs of CAS will configure OS level firewall rules
+ Current home firewalls have bi-directional policy that allows originating traffic back through IP/Port combinations.
+ Most firewall confiurations leave outbound ACLs open to all traffic
+ Inbound traffic to the workstation that has the CAS agent install need to have the following ports opened.
```
    TCP: 443, 4172, 60443
    UDP :4172
```
+ Many deployment will leverage a NAT or PAT rule mapping External IPs to Workstations Internal to specified port(s) as well

    ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Firewall%20rules.jpg)

# Decide on the version of CAS agent that is needed #
**An assumption on type of CAS agent and purchase has been made in pre-sales processes**

There are two CAS agent SKUs availble depending on the requirements of the end-user. Each of these SKUs have a difference price point that is assoicated to the purchase/ registration key at time of sale. It is important that right agent is selected, there have been situations where end-users paid for Graphics Agents but downloaded Standard Agent. Our licensing server would permit this senerio but it would NOT prevent the opposite (paying for Standard but installing Graphics) from happening.

**Overview of CAS Agents:**

|  CAS Version    |  Performance    |         Encoder        |       Hardware      |    Use Case    |
| --------------- | --------------- |------------------------|---------------------|----------------|
| CAS Graphics    |    60FPS -4K    | multi-core AVX2 -NVENC | CPU AVX2 /GPU-Nvidia|Content Creation|
| CAS Standard    |    30FPS -2K    |        CPU             |       x86           |  Task Worker   |

# Ensure the requirements each OS are met #
**The next step is to decide on the OS for the CAS agent**

Both CAS Graphics and Standard isn't locked to a particular OS. CAS Graphics has the option to be installed on various flavors of Windows, Linux and MAC. CAS Standard can be installed on Windows and Linux.  Each possible deployment seneraio has a system requirement page that should be verified before an installation begins. 

Before continuing consult the requirement page for the applicable deployment:
- [Graphics Agent for Windows](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/windows/22.01/admin-guide/requirements/system-requirements/)
- [Graphics Agent for Linux](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/linux/22.01/admin-guide/requirements/system-requirements/)
- [Graphics Agent for MAC](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/macos/22.01/admin-guide/requirements/system-requirements/)
- [Standard Agent for Windows](https://www.teradici.com/web-help/pcoip_agent/standard_agent/windows/22.01/admin-guide/requirements/system-requirements/)
- [Standard Agent for Linux](https://www.teradici.com/web-help/pcoip_agent/standard_agent/linux/22.01/admin-guide/requirements/system-requirements/)

# CAS Agent installation on host #
**This deployment guide we will focus on CAS Graphics on Windows, which will be the most common use case for this group.**

1. From the [Teradici portal](https://docs.teradici.com/find/product/cloud-access-software), select the CAS version and type of host systems OS. ) Then select the **‘Log in to download’** button.

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Select%20Download.png)

1. You must have a registered Teradici account in order to download the software, You can “Create an account”, if you don’t have an account. This account isn’t tied to a purchase or sale. HP or personal email address are allowed.  Note: Teradici Sales and Support employees are tied our old **@teradici.com** email addresses (not) **@HP.com**
     
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/LogintoPortal.png)

 1. &#x1F536;**(Optional)** &#x1F536; the Teradici portal will default to the latest GA build. By selecting the drop-down menu, you have access to the previous quarters build, as well has the next to upcoming beta and development builds as well. 

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Access2OptionalBuilds.png?raw=true)
      
 1. Next Scroll through the EULA until the “Agree button ”  isn't greyed button is available to “click on”

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/EULA_Agree.png?raw=true)     
 
 1. Once EULA is agreed you have two options a download (or) a download using a script via curl. (Download via script does include as user session token to programmatically authenticate to download page and download. Note: curl is primarly used for Linux, windows server does have curl pre-installed, windows 10)
 
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/DownLoadOptions.png?raw=true)
     
 1. Once the agent installer is downloaded, run installer under a local /AD administrator account. Administrator permission will be required for USB passthrough driver, audio driver and a PowerShell script to modify Windows Defender Firewall. Many times the installer launches and near the end receives a **PowerShell Execution Policy** error, the user at a minimum must set **AllSigned** when running outside of administrator account. 

      ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/RUNASADMIN.png?raw=true)
 
1. Select a language, English by default

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Selectlanguage.png?raw=true)
      
1. &#x1F536;**(Caution)** &#x1F536; You may receive an “unsupported graphics adapter cannot be found”, many times the GPU is supported, although the driver is not.
     - **If you have a supported GPU, but not the correct driver** - You have the option to select "No" to continue the CAS installation and fix afterwards.
     - **If you don't have a supported GPU, thus wrong driver as well** - You can still select "No", but will get a "installation aborted", in essence as selecting "Yes"

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Unsupported%20Graphics.png?raw=true)
 
 1. If you have a supported GPU and driver, you will go straight to the installation wizard screen.
     
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/WelcomeScreen.png)
     
 1. There is a CAS Agent EULA section, that can just be "clicked" **I Agree** to continue.

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/CAS_EULA.png)
     
 1. Choose an Installation path, Next to continue.
 
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/InstallPath.png)
     
 1. The installation process should take under a minute to complete. Rarely, if there any installation errors will show up as &#x1F536; (RED) &#x1F536; text by the installer. You can scroll back to error messages can copy directly from installer. 
 
     Also installation logs are kept in *%Appdata%/roaming/Teradici/*

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/InstallerProgress.png?raw=true)
 
 1. When purchasing Teradici CAS, you will receive a registration code that will have to be entered in the registration code window. All registration code follow the same format of a having a *ABCDEFGH12@AB12-C345-D67E-89FG* with @ in the middle of the code. (A registation code is tied to a pool of floating seats)
  
     You can select the “not now” and run a registration script after installation. Generally, this is for building “golden images”. Typically, images and will be deployed and have a start-up script for registration. For more information see; [PowerShell Licensing Script](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/windows/22.01/admin-guide/licensing/licensing/). The licensing portion of the installation will take, machine name, CPU ID, MAC and other machine metadata IDs) to create a series of secure files on the local drive. Everytime, PCoIP service is started it checks if these file match with the workstation. If is why you can't clone a registered workstation/VM. 

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/LicensingStep.png)

1. &#x1F536; (Optional) &#x1F536; The license registration step in CAS installer assumes that the workstation has outbound internet access to registration the workstation to our cloud Licensing Server. If a customer is working in a dark site, a [local licensing server](https://www.teradici.com/web-help/pcoip_license_server/22.01/offline/?_ga=2.209723026.607431230.1644860675-1630610697.1641343076) has to be deployed in their enviroment.

     There is a field to enter a proxy server for Internet Connections with port, if needed. When the **Next** button is pressed. The CAS agent will try and register itself to the cloud licensing server. It may take a couple of minutes for the registration  to complete.
     
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Final_reg.png)

 
1. It’s a good idea to reboot the workstation before starting a CAS session, the Teradici virtual audio and the USB virtual driver will not be available until the system is rebooted.

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/RebootNow.png)

# Post installation configuration for Windows Graphics Agent # 
CAS has a dynamic adaption system that will adjust session parameters automatically as network congestion arises. By default, there are few “knobs” we  have to adjust, I do set two parameters  after installation on Graphics Agents for Windows. I do recommend the same settings on Graphics of Linux as well, but it is done in a [configuration file](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/linux/22.01/admin-guide/configuring/configuring/) instead. 

1. Start by typing **gpedit.msc** in run command to open
     
      ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/gpedit.png)
      
1. Navigate to **Computer Configuration > Administrative Templates > PCoIP Session Variables**. Without knowing the specifics of the end-users requirements, I will only set two variables and let the dynamic nature of the PCoIP protocol do the rest of the adjustments on the fly. 

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/GPO_Navigate.png)
     
1. On the Left-hand side, you can choose either Overridable (or) Non-Overridable, on a basic configuration. The difference being that an AD GPO could override a overridable locally configured GPO.
 
1.  The two setting I manually adjust is **PCoIP Ultra** and **Log Verbosity**.

- By Default, PCoIP Ultra protocol dynamic encoder isn’t enabled. (In the future it will). **Enable** and select a **Ultra** encoding policy 

     **Decision criteria for PCoIP Ultra Configuration**
     -  If the workstation has a NVIDIA GPU and is Pascal generation or higher, then chances are that it has a NVENC encoder chip on the GPU (enable Auto-Offload)
     -  If the workstation has an AMD GPU --No NVENC-- (enable CPU Offload)
     -  The best experience is to turn on “Automatic Offload” for the typical content creator and leave PCoIP Ultra Offload MPPs at 10K (default)

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/PCoIPUltraSettings.png)
     
- Depending the Teradici Engineer you speak too, you will get varied recommendations on what logging level to set. My philosophy is that I don’t want to ask the customer to try and reproduce an error and re-generate a support bundle just because verbose logging wasn’t originally set. I set event log verbosity to **3** 
     
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/LoggingLevel.png)    
 
# Installing Teradici CAS client and connect to a host workstation #
In this section, you will obtain the software, install, and establish a connection to your CAS host. Depending on your network topology, use will either connect to the local IP,  Public IP (or) Fully Qualified Domain Names (FQDN) with the CAS client.

1. [Download the client installer](https://docs.teradici.com/find/product/software-and-mobile-clients) based on your client OS. You don't need a login credentials to download client software and can have as many copys of various client OS as you need.

1. Install the PCoIP client software per the OSs Administration Guides installation instructions.

1. Locate the **IP address** or **FQDN** of the Host workstation.

1. Under the **Details** tab you will see **Public IPv4 Address** (or) **Private IPv4 Address** (or) **Private IPv4 DNS** (or) **Public IPv4 DNS**

1. From the client system, start your PCoIP client per OS. Typically the PCoIP client will have a icon:

    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP_icon.jpg)

1. When the PCoIP client starts, it will ask for a **Host Address or Code**. Enter in your **IP address or FQDN** previously identified in previous section. (optionally) enter a name to **Connection Name** field then **SAVE**, if you want to save connection.

    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Client.jpg)
    
1. Next, you will get a Cannot verify your connection to IP warning. This error is becuase a 3rd party trusted certificate has not been install on the host. You can select the **Connect Insecurely** option. For your convenience you can remove this error message by adding the **security_mode = 0** to the **%appdata%/teradici/Teradici PCoIP Client** file. The PCoIP Client login screen will show a red opened lock, when enabled.
    
    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Trusted.jpg)
    
 1. Finally, enter in the login credentials. 

    ![image](https://github.com/ChadSmithTeradici/TeradiciPCoIPonMACinAWS/blob/main/images/PCoIP-Auth.jpg)
    
    From a troubleshooting perspective, if you can authenticated then traffic on port 443 is working correctly. When a PCoIP session takes over it will try and send the CAS client encrypted pixels over UDP 4172. If that portion fails, then you can bet that a firewall / security group is blocking traffic. Most often it will lead to a generic **[6405 Error code](https://help.teradici.com/s/article/1356)**
