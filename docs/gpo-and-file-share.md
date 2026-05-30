# GPO and File Share

This document describes the Group Policy Objects and file share configuration used in the lab.

## File Share Overview

A shared folder was created on SRV-AD01 to simulate a common company network share.

- Server: SRV-AD01
- Local path: C:\Shares\Partage-Lab
- Network path: \\SRV-AD01\Partage-Lab
- Purpose: shared storage for domain users

## Security Group

Access to the share is controlled with an Active Directory security group.

- Group name: GG_Lab_Share_RW
- Group scope: Global
- Group type: Security
- OU: Groups
- Member: test.user

This group is used to grant read/write access to the lab share.

## Share and NTFS Permissions

Share-level permissions:

- GG_Lab_Share_RW: Change / Full Control for lab purposes

NTFS permissions:

- GG_Lab_Share_RW: Modify

This follows the common Windows file server practice of using AD groups to manage access instead of assigning permissions directly to individual users.

## Drive Mapping GPO

A Group Policy Object was created to map the network share automatically for users.

- GPO name: GPO-Map-Drive-Lab
- Linked OU: Lab-Users
- Type: User Configuration
- Drive letter: P:
- Label: Partage Lab
- Target path: \\SRV-AD01\Partage-Lab

After logging in with test.user, the P: drive is automatically mapped.

Validation command:

- net use

Expected result:

- P: is mapped to \\SRV-AD01\Partage-Lab

## Security GPO

A user security GPO was created for lab users.

- GPO name: GPO-Security-Users
- Linked OU: Lab-Users
- Type: User Configuration

Configured settings:

- Enable screen saver
- Password-protect the screen saver
- Screen saver timeout: 300 seconds
- Restrict access to Control Panel and Windows Settings

This validates basic user environment hardening through Group Policy.

## Workstation GPO

A workstation GPO was created and linked to the Workstations OU.

- GPO name: GPO-Test-Workstations
- Linked OU: Workstations
- Type: Computer Configuration

Configured setting:

- Interactive logon banner

This was used to validate that computer-side GPOs apply correctly to the Windows 11 domain client.

## Validation

Validation performed from WIN11-LAB:

- gpupdate /force
  - Confirms that Group Policy updates successfully.

- gpresult /scope user /r
  - Confirms that user GPOs are applied.

- gpresult /scope computer /r
  - Confirms that computer GPOs are applied.

- net use
  - Confirms that the P: drive is mapped.

- Manual access to \\SRV-AD01\Partage-Lab
  - Confirms that the share is reachable.

- File creation test inside the share
  - Confirms write permissions through the AD security group.

## Design Reasoning

The file share and GPO configuration simulates common enterprise administration tasks.

Using AD groups for permissions makes access easier to manage and more scalable than assigning rights directly to users.

Mapping the share through GPO provides a centralized way to configure user workstations.

User and computer GPOs were separated to demonstrate the difference between user-side and computer-side policy application.
