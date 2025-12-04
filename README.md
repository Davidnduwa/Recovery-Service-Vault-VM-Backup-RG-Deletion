 Recovery Services Vault, VM Backup & Resource Group Deletion Constraints

This lab provides hands-on experience with **Azure Recovery Services Vault (RSV)**, focusing on regional limitations for VM backups, the impact of VM power state on backup execution, and the strict procedure required to delete Resource Groups containing active backup items.

## 🎯 Lab Goals

* Deploy **Recovery Services Vaults (RSVs)** across different regions.
* Enable and trigger backups for Azure Virtual Machines (VMs).
* Understand the **strict regional requirement** for RSV and VM placement.
* Learn the multi-step process required to **purge backup data** before a Resource Group (RG) or Vault can be deleted.

---

## 🏗️ Architecture Overview

The lab requires creating mirrored infrastructure in two separate Azure regions to test the RSV's regional constraint.

| Resource Name | Region | Resource Group |
| :--- | :--- | :--- |
| **Resource Group** | East US | `rg-backup-eastus` |
| **Resource Group** | West US | `rg-backup-westus` |
| **VM** | East US | `vm-backup-east1` |
| **VM** | West US | `vm-backup-west1` |
| **Recovery Services Vault** | East US | `rsv-eastus` |
| **Recovery Services Vault** | West US | `rsv-westus` |


---

## 🧪 Step-by-Step Lab Guide

### 1. Initial Setup: Resource Groups and VMs

1.  Create the following **Resource Groups**:
    * `rg-backup-eastus` → **East US**
    * `rg-backup-westus` → **West US**
2.  Create one **Virtual Machine** in each Resource Group:
    * `vm-backup-east1` → **East US**
    * `vm-backup-west1` → **West US**

### 2. Testing the Regional Constraint

1.  Create a **Recovery Services Vault** in East US:
    * Name: `rsv-eastus`
    * Region: **East US**
    * RG: `rg-backup-eastus`
2.  **Enable Backup for the East US VM:**
    * Navigate to **`rsv-eastus` → Backup → Azure VM**.
    * Select `vm-backup-east1` → Apply policy → **Backup Now**.
3.  **Try enabling backup for the West US VM:**
    * Attempt to select `vm-backup-west1` in the **`rsv-eastus`** setup.
    * ➡️ **Expected Result:** **You will NOT see `vm-backup-west1` listed.**

> **Principle Verified:** An Azure VM can **only** be backed up by a Recovery Services Vault that resides in the **exact same region** as the VM.

### 3. Completing the Setup and Testing Power State

1.  Create a second RSV in West US: `rsv-westus` → Region: **West US**.
2.  Protect `vm-backup-west1` using `rsv-westus`.
3.  **Stop (Deallocate) `vm-backup-east1`** in the Azure Portal.
4.  Trigger a manual backup for the deallocated `vm-backup-east1` using `rsv-eastus`.

> **Principle Verified:** **Backups still work** even when the VM is **Stopped (Deallocated)**. This is because the backup process primarily takes a snapshot of the underlying disk (the storage account), which is independent of the VM's compute state.

### 4. Testing Resource Group Deletion

1.  Attempt to delete the Resource Group: `rg-backup-eastus`.
2.  **Expected Result:** **❌ Deletion Fails.**

> **Reason:** The Resource Group contains the active **`rsv-eastus`** vault, which in turn still holds **protected backup data** for `vm-backup-east1`. Azure prevents deletion to avoid accidental data loss.

### 5. Remediation Steps (The AZ-104 Procedure)

To successfully delete the RG, you must perform the following steps **in order**:

1.  Navigate to **`rsv-eastus` → Backup Items → Azure Virtual Machine**.
2.  For `vm-backup-east1`, click the ellipses (`...`) → **Stop Backup**.
3.  Crucially, on the next screen, choose **"Stop backup and Delete backup data."** You must explicitly confirm the deletion of the backup data.
4.  Wait for the deletion to complete (it can take time).
5.  Delete the `rsv-eastus` vault itself.
6.  Retry deleting the Resource Group: `rg-backup-eastus`.
7.  **Expected Result:** **✔️ Success.**

---

## 📌 Key Takeaways 

| Topic | Constraint or Requirement |
| :--- | :--- |
| **Regionality** | The **Recovery Services Vault** must be in the **same region** as the VM it is protecting. |
| **VM Power State** | Backups execute successfully even if the VM is **Stopped/Deallocated** because they operate on the underlying disk storage. |
| **Resource Deletion** | You **cannot delete a Vault or Resource Group** until: **1)** Backup protection is officially **Stopped**, and **2)** All associated **Backup Data is Purged** from the vault. |
| **Procedure** | Deleting an RG with a protected VM requires deleting **Backup Data → Vault → Resource Group**. |
