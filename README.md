# Active Directory & Windows Server Home Lab

**Author:** Michael Barisua Madu  
**Series:** IT/Cybersecurity Home Lab Series (Lab 1 of 7)

---

## Overview

Built a fully functional Active Directory domain environment from scratch in VirtualBox, covering domain controller setup, organizational structure, Group Policy enforcement, DNS/DHCP services, file share permissions, and a domain-joined Windows 11 client—the foundational infrastructure most enterprise environments run on.

* **Domain:** `corp.local`
* **Hypervisor:** Oracle VirtualBox, NAT Network (`LabNet`, `10.0.2.0/24`)
* **Host:** Acer Predator 16

---

## Architecture & Components

| Hostname | IP Address | OS / Roles | Key Details |
| :--- | :--- | :--- | :--- |
| **DC01** | `10.0.2.10` | Windows Server 2022 Standard (Eval) | Promoted new forest `corp.local`. Active Directory Domain Services, DNS Server, DHCP Server, File Server (`\\DC01\ITShare`) |
| **Client01** | `10.0.2.20` static / `10.0.2.100-200` via DHCP | Windows 11 Enterprise | Domain-joined to `corp.local`, subject to IT-Staff GPOs |

---

## What Was Built

* **Domain Controller:** Windows Server 2022 Standard (Eval), promoted new forest `corp.local`.
* **OU Structure:** Departments (`IT`, `Sales`, `Human Resource (HR)`), `Groups`, and `Service Accounts`.
* **Security Groups:** `IT-Staff`, `Sales-Staff`, `HR-Staff`, populated with test users per department.
* **Group Policy:**
  * **Domain Password Policy:** 8-character minimum, complexity requirements, 90-day max age, history 5.
  * **Drive Mapping:** IT-scoped drive mapping (`Z:` to `\\DC01\ITShare`).
  * **Control Panel Restriction:** IT-scoped Control Panel access blocked.
* **DNS:** Self-hosted on `DC01`, resolving `corp.local` internally and forwarding externally.
* **DHCP:** Scope `10.0.2.100-200`, gateway and DNS options configured, authorized in AD.
* **File Sharing:** `ITShare` folder share permissions for `Authenticated Users`, NTFS permissions scoped to `IT-Staff`.
* **Client:** Windows 11 Enterprise VM, domain-joined, tested login as a real `IT-Staff` user.

---

## Verification Performed

* **Drive Mapping & Share Access:** Logged into `Client01` as a domain user (`IT-Staff` member); confirmed automatic drive mapping to `Z:` and successful connection to `\\DC01\ITShare`.
* **GPO Restrictions:** Confirmed Control Panel access is blocked for the same user with the expected Windows restriction message.
* **Scope Isolation:** Confirmed Command Prompt remains accessible (GPO scoped correctly, no unintended over-restriction).
* **DHCP Verification:** Released/renewed `Client01`'s IP and confirmed it received a DHCP-leased address (`10.0.2.100`) with correct subnet mask, gateway, and DNS suffix.

---

## Problems Encountered and Fixed

* **Domain accounts couldn't log into the DC directly:** By design, Windows Server restricts interactive logon on domain controllers to admin-tier accounts. Fixed by testing GPO effects on the domain-joined client instead, which is also the correct real-world approach.
* **GPO "Edit" option greyed out:** Was logged in with a local account rather than `CORP\Administrator`, which has no rights over domain GPOs. Fixed by signing into the domain admin account via the VM's Ctrl-Alt-Del menu.
* **DHCP authorization failed (Error 20070):** The post-install wizard defaulted to the currently logged-in account (`CORP\Madu`), which lacked Enterprise Admins rights required to authorize a DHCP server in AD. Fixed using the "alternate credentials" option to authorize with `CORP\Administrator` directly, without needing to log out.
* **VirtualBox NAT Network dropdown appeared to only show two options:** The Settings dialog was in "Basic" mode, which hides advanced network types like NAT Network. Switching to "Expert" mode revealed the full list.
* **`ipconfig /release` failing:** Client adapter was still statically configured; the automatic-IP setting hadn't actually been applied yet. Fixed by re-confirming and applying the "Obtain automatically" radio buttons before retrying.

---

## What I'd Do Differently in Production

* **Redundancy:** A single DC is a single point of failure; production environments would run at least two DCs for high availability.
* **Network Segmentation:** This lab uses a flat NAT Network with no VLAN segmentation; a real deployment would separate server, client, and management traffic across VLANs with firewall rules between them.
* **Granular Password Policies:** Password/account policies here are domain-wide defaults; production environments often use Fine-Grained Password Policies (FGPP) to apply stricter rules to privileged accounts specifically.
* **Hybrid Authentication:** Remote/hybrid access wasn't addressed; this setup assumes devices are on the same local network as the DC, which doesn't reflect how a real remote workforce authenticates. That gap is the direct motivation for the Microsoft Entra ID lab that follows this one.

---

## Skills Demonstrated

Active Directory Domain Services, Group Policy Management, DNS, DHCP, NTFS/share permissions, Windows Server administration, VirtualBox networking, systematic troubleshooting.
