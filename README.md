# VM Migration Between Standalone Esxi Hosts (without V Center) i.e. One Esxi to Another Esxi

## 📄 Title

**Standard Operating Procedure (SOP): Migrating a Virtual Machine Between Two Standalone ESXi Hosts Without vCenter**

---

## 🎯 1. Purpose

This document provides a clear, professional, and repeatable procedure for migrating a virtual machine (VM) between two standalone VMware ESXi hosts without using VMware vCenter Server. The objective is to enable IT administrators to safely move virtual machines in environments where vCenter is not available, such as labs, branch offices, or standalone production hosts.

---

## 📌 2. Scope

This procedure applies to:

* Standalone VMware ESXi hosts
* Environments without VMware vCenter
* IT/System Administrators responsible for VMware infrastructure

This SOP covers offline VM migration using datastore file transfer and command-line disk conversion.

---

## 🧪 3. Environment Overview (Lab Setup)

This demonstration is performed in a lab environment consisting of two standalone ESXi hosts:

* **ESXi Host 1 (Source Host):** 192.168.100.1

  * Contains one virtual machine: **ABC_LIVE_SRV**

* **ESXi Host 2 (Destination Host):** 192.168.100.2

  * No existing virtual machines

The virtual machine **ABC_LIVE_SRV** will be migrated from ESXi Host 1 to ESXi Host 2.

---

## 🖧 4. Source Host Verification – ESXi Host 1 (192.168.100.1)

Log in to ESXi Host 1 using the web interface. The host is currently running a Windows Server virtual machine.

Verify the datastore configuration:

* The virtual machine files are stored on the **DS1 datastore**.

Using the datastore browser:

* Locate the folder named **ABC_LIVE_SRV**.
* This folder contains all VM-related files, including:

  * `.vmx` (virtual machine configuration file)
  * `.vmdk` (virtual disk descriptor file)
  * `flat.vmdk` (virtual disk data file)

Note: In the ESXi web interface, the `.vmdk` and `flat.vmdk` files appear as a single VMDK entry.

---

## 🔐 5. Enabling SSH and Verifying VM Files

To confirm the presence of all VM files, SSH access must be enabled.

### 5.1 Enable SSH on ESXi Host 1

1. Log in to the ESXi host
2. Right-click **Host**
3. Navigate to **Services**
4. Enable **Secure Shell (SSH)**

### 5.2 Connect Using PuTTY

* **Hostname:** 192.168.100.1
* **Port:** 22
* **Username:** admin
* **Password:** passwordfg@12345!@

Navigate to the VM directory:

```
/vmfs/volumes/ds1/ABC_LIVE_SRV
```

Run the command:

```
ls -al
```

This confirms the presence of `.vmx`, `.vmdk`, and `flat.vmdk` files.

---

## ⏬ 6. Powering Off and Downloading VM Files

Before copying any files, the virtual machine **must be powered off**.

### 6.1 File Download Options

You may download VM files using:

* ESXi web interface (Datastore Browser), or
* **WinSCP** (recommended for large files)

### 6.2 Download Using WinSCP

Login details:

* **Host Name:** 192.168.100.1
* **Username:** admin
* **Password:** passwordfg@12345!@

Navigate to:

```
/vmfs/volumes/ds1/ABC_LIVE_SRV
```

Create a local folder on your desktop named:

```
ABC_LIVE_SRV copy
```

Copy the following files to the local folder:

* `.vmx`
* `.vmdk`
* `flat.vmdk`

**Important Note:**

* The `flat.vmdk` file is thick-provisioned and equals the full virtual disk size (e.g., 50 GB).
* If the VM has multiple disks, repeat this process for each disk.

---

## 📦 7. Destination Host Preparation – ESXi Host 2 (192.168.100.2)

Log in to ESXi Host 2 using the web interface.

Login credentials:

* **Username:** admin
* **Password:** pass@1234567@#@$

### 7.1 Create Destination Folder

On the **DS2 datastore**, create a new folder:

```
ABC_LIVE_SRV_Copy
```

### 7.2 Upload Files Using WinSCP

Login to ESXi Host 2 via WinSCP:

* **Host Name:** 192.168.100.2
* **Username:** admin
* **Password:** pass@1234567@#@$

Navigate to:

```
/vmfs/volumes/ds2/ABC_LIVE_SRV_Copy
```

Upload all previously downloaded VM files from the local desktop folder.

After upload completion, verify the files using the datastore browser.

---

## 🖥️ 8. SSH Verification on Destination Host

Connect to ESXi Host 2 using PuTTY.

Navigate to the VM directory:

```
cd /vmfs/volumes/ds2/ABC_LIVE_SRV_Copy
```

Run the command:

```
ls -al
```

Confirm the presence of `.vmx`, `.vmdk`, and `flat.vmdk` files.

---

## 💾 9. Converting Thick Disk to Thin Provisioning

Since the uploaded virtual disk is thick-provisioned, convert it to thin provisioning.

Run the command:

```
vmkfstools -i ABC_Server.vmdk -d thin ABC_Server-thin.vmdk
```

**Note:** If the file or folder name contains spaces, use quotation marks:

```
vmkfstools -i "ABC Server.vmdk" -d thin "ABC Server-thin.vmdk"
```

This process may take time depending on disk size.

---

## 🧹 10. Renaming and Cleanup

Rename the original thick-provisioned disk to avoid conflicts:

```
vmkfstools -E ABC_Server.vmdk ABC_Server-thick.vmdk
```

Rename the thin-provisioned disk to match the VM configuration:

```
vmkfstools -E ABC_Server-thin.vmdk ABC_Server.vmdk
```

At this stage, the directory will contain:

* One `.vmx` file
* Thick-provisioned VMDK and flat.vmdk (optional for deletion)
* Thin-provisioned VMDK and flat.vmdk (active disk)

You may safely delete the thick-provisioned files if no longer required.

---

## ▶️ 11. Registering and Powering On the Virtual Machine

1. In the ESXi web interface, browse to the VM folder
2. Right-click the `.vmx` file
3. Select **Register VM**
4. Power on the virtual machine
5. When prompted, select **“I copied it”**

The virtual machine will now boot successfully on ESXi Host 2.

---

## ✅ 12. Conclusion

This procedure demonstrates how to migrate a virtual machine between two standalone ESXi hosts without VMware vCenter. By following this SOP, IT administrators can safely perform offline VM migrations while maintaining data integrity and operational consistency.

---
