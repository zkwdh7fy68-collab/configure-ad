# Preparing Active Directory in Azure

## Project overview

In this lab, I prepare an Azure environment for Active Directory by creating two virtual machines, configuring their network settings, and checking connectivity between them. One VM will become the domain controller, and the other will act as a client workstation.

This walkthrough is written for beginners in IT, Azure, and Active Directory. Each section explains both the steps and the reason behind them.

**Scope:** This project covers infrastructure and network preparation. Installing Active Directory Domain Services, promoting the server to a domain controller, and joining the client to the domain are later steps.

## What is Active Directory?

Active Directory is Microsoft's directory service for centrally managing users, computers, and access to network resources.

For example, a company with 500 employees can use Active Directory to manage employee accounts in one place. IT staff can reset passwords, organize users into groups, and control access to shared files and other resources.

## Lab environment

| Resource | Name or configuration | Purpose |
| --- | --- | --- |
| Resource group | `Active-Directory-Lab` | Groups the lab resources together |
| Virtual network | `Active-Directory-VNet` | Provides the network for both VMs |
| Subnet | `default` — `10.0.0.0/24` | Places both VMs on the same subnet |
| Server VM | `dc-1` — Windows Server | Will serve as the domain controller and DNS server |
| Client VM | `client-1` — Windows Pro | Acts as the client workstation |
| Administrator username | `labuser` | Example account used to sign in to the VMs |

The diagram below illustrates the planned environment. Its example address ranges differ from the `10.0.0.0/24` subnet used in this walkthrough, and the domain join shown is a later step.

![Azure resource hierarchy and planned domain controller and client relationship](images/step-01.png)

## Before you begin

- Have access to an Azure subscription and the Azure portal.
- Have a Remote Desktop client available to connect to the VMs.
- Use the same Azure region for the resource group, virtual network, and VMs in this lab.
- You can choose your own resource names and administrator username. Use them consistently throughout the walkthrough.
- Create your own strong password for each VM and keep it private.

**Version note:** The draft references Windows Server 2025, while the screenshots show an earlier Windows Server image. This guide uses the general label **Windows Server**; match the image to the version you use in your lab.

## 1. Create the resource group

A resource group keeps the resources for this lab organized in one place.

1. In the Azure portal, open **Resource groups** and select **Create**.
2. Select your subscription.
3. Enter `Active-Directory-Lab` as the resource group name.
4. Choose the region you will use throughout the lab.
5. Select **Review + create**, then **Create**.

<details>
<summary>View resource group screenshot</summary>

![Creating the Active Directory lab resource group](images/step-02.png)

</details>

## 2. Create the virtual network

Both VMs will use the same virtual network so they can communicate through their private IP addresses.

1. Open **Virtual networks** and select **Create**.
2. Select the `Active-Directory-Lab` resource group.
3. Name the network `Active-Directory-VNet`.
4. Select the same region as the resource group.
5. Confirm that the network includes the `default` subnet with the address range `10.0.0.0/24`, which will be used by both VMs.
6. Select **Review + create**, then **Create**.

<details>
<summary>View virtual network screenshot</summary>

![Creating the Active Directory virtual network](images/step-03.png)

</details>

## 3. Create the server VM

Create the Windows Server VM that will later become the domain controller.

1. Open **Virtual machines** and start creating a new VM.
2. Configure the following settings:

   | Setting | Value |
   | --- | --- |
   | Resource group | `Active-Directory-Lab` |
   | Virtual machine name | `dc-1` |
   | Region | Same region as the virtual network |
   | Image | Your selected Windows Server image |
   | Administrator username | `labuser`, or your chosen username |
   | Administrator password | Your own strong password |

3. Review the image's licensing requirements and select only the confirmations that apply to your subscription and licenses.
4. On the **Networking** tab, select `Active-Directory-VNet`.
5. Set the subnet to `default (10.0.0.0/24)`.
6. Review the configuration and select **Create**.

**Why this subnet?** The `/24` corresponds to a subnet mask of `255.255.255.0`. Keeping both VMs on the same subnet simplifies communication between them.

<details>
<summary>View server VM screenshots</summary>

![Configuring the dc-1 virtual machine](images/step-04.png)

![Selecting the Windows Server image and VM size](images/step-05.png)

![Selecting the virtual network and subnet for dc-1](images/step-06.png)

</details>

## 4. Create the client VM

Create a second VM to represent a user's workstation.

1. Open **Virtual machines** and start creating another VM.
2. Configure the following settings:

   | Setting | Value |
   | --- | --- |
   | Resource group | `Active-Directory-Lab` |
   | Virtual machine name | `client-1` |
   | Region | Same region as `dc-1` |
   | Image | Your selected Windows Pro image |
   | Administrator username | `labuser`, or your chosen username |
   | Administrator password | Your own strong password |
   | Virtual network | `Active-Directory-VNet` |
   | Subnet | `default (10.0.0.0/24)` |

3. Review the image's licensing requirements and applicable confirmations.
4. Select **Review + create**, then **Create**.

<details>
<summary>View client VM screenshots</summary>

![Configuring the client-1 virtual machine](images/step-07.png)

![Selecting the client operating system and administrator settings](images/step-08.png)

</details>

## 5. Assign a static private IP address to dc-1

The client will use `dc-1`'s private IP address as its DNS server address. Assigning a static private IP keeps that address consistent.

1. Open **Virtual machines** and select **dc-1**.
2. Go to **Networking → Network settings**.
3. Open the VM's **network interface**.
4. Select **IP configurations**, then **ipconfig1**.
5. Under **Private IP address settings**, change the allocation from **Dynamic** to **Static**.
6. Save the change and record the private IP address.

The later screenshots use `10.0.0.4` for `dc-1`. Use the actual private IP assigned to your server if it differs.

<details>
<summary>View static IP screenshots</summary>

![Opening the network interface for dc-1](images/step-09.png)

![Viewing the private IP configuration](images/step-10.png)

![Changing private IP allocation to static and saving](images/step-11.png)

</details>

## 6. Prepare dc-1 for the connectivity test

Connect to `dc-1` using Remote Desktop. Use its **public IP address** and the administrator credentials you created during VM setup.

For the connectivity test in this lab, Windows Defender Firewall was temporarily disabled:

1. Right-click **Start** and select **Run**.
2. Enter `wf.msc` to open Windows Defender Firewall with Advanced Security.
3. Open **Windows Defender Firewall Properties**.
4. Set **Firewall state** to **Off** for the **Domain**, **Private**, and **Public** profiles.
5. Apply the changes.

**Lab-only configuration:** Disabling all firewall profiles is a temporary testing step. Re-enable the firewall after testing; do not treat this as a production configuration.

<details>
<summary>View firewall screenshot</summary>

![Windows Defender Firewall profile settings on dc-1](images/step-12.png)

</details>

## 7. Configure client-1 to use dc-1 for DNS

Configure the client's network interface to use the server's private IP address. This prepares the client to use `dc-1` once the DNS service is configured there.

1. Copy `dc-1`'s **private IP address**.
2. In the Azure portal, open **client-1**.
3. Go to **Networking → Network settings** and open the client's **network interface**.
4. Under **Settings**, select **DNS servers**.
5. Select **Custom**.
6. Enter `dc-1`'s private IP address and select **Save**.
7. Restart `client-1` so it picks up the new settings.

**Note:** Setting the DNS address does not install or configure a DNS service on `dc-1`. Until that service is available, DNS lookups from the client may fail.

<details>
<summary>View DNS configuration screenshots</summary>

![Locating the private IP address of dc-1](images/step-13.png)

![Opening the client-1 network interface](images/step-14.png)

![Setting a custom DNS server address on client-1](images/step-15.png)

![Restarting client-1 after changing its DNS settings](images/step-16.png)

</details>

## 8. Test connectivity from client-1

Connect to `client-1` through Remote Desktop and open PowerShell. Ping the private IP address of `dc-1`:

```powershell
ping 10.0.0.4
```

Replace `10.0.0.4` with your server's private IP if necessary.

**Expected result:** Replies from `dc-1` confirm that the client can reach the server over the private network. The example below shows four replies and 0% packet loss.

![Successful ping from client-1 to dc-1 at 10.0.0.4](images/step-17.png)

## 9. Verify the client's DNS setting

On `client-1`, run:

```powershell
ipconfig /all
```

Find **DNS Servers** in the output and confirm that it lists `dc-1`'s private IP address. In this lab, the expected value is `10.0.0.4`.

This checks the client's configured DNS address. It does not yet verify DNS resolution or an Active Directory domain.

![Client network configuration showing 10.0.0.4 as the DNS server](images/step-18.png)

## Results

The screenshots document:

- A Windows Server VM and a Windows client VM in the Azure lab environment.
- A static private IP configuration for the future domain controller.
- Client DNS settings pointing to the server's private IP address.
- Successful private-network connectivity from `client-1` to `dc-1`.

## Skills practiced

- Organizing Azure resources with resource groups.
- Creating virtual networks, subnets, and Windows VMs.
- Connecting to VMs through Remote Desktop.
- Configuring a static private IP address and custom DNS settings.
- Checking network connectivity with `ping` and reviewing configuration with `ipconfig /all`.

## Next steps

The environment is prepared for the next stage: installing Active Directory Domain Services, promoting `dc-1` to a domain controller, and joining `client-1` to the domain.
