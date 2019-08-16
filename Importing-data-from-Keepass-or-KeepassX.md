## Introduction
Bitwarden can import your data from a large number of [applications](https://help.bitwarden.com/article/import-data/).

The current importers let you only choose the format, not HOW the data is translated to Bitwarden.

## Different import results for Keepass and KeepassX
Importing from Keepass or KeepassX gives complete different results, although they use the same Keepass 2.x kbdx database:
* Keepass CSV files are imported at the **Organization** level (owner of each entry) and translates the Keepass Groups into Bitwarden **Collections**.
* Keepass XML files are imported at the **User** level (owner of each entry) and translates the Keepass Groups into Bitwarden **Folders** with as main folder the name of the Keepass database.

It is a lot of work in Bitwarden itself to change Collections to Folders or to transfer ownership of all the entries.
So depending on what you want, choose the appropriate method!

## Example
### Keepass database with name 'MyVault'

**Groups:**
* Group1
  *   Group1Sub1
  *   Group2Sub2
* Group2

### Import via Keepass:

**Owner** = Organization

**Collections:**
* Group1
  *   Group1Sub1
  *   Group2Sub2
* Group2

### Import via KeepassX:
**Owner** = Logged in User

**Folders:**
* MyVault
  * Group1
    * Group1Sub1
    * Group2Sub2
  * Group2

Note: you might have to create the main folder manually, as the import shows MyVault/Group1 as a Folder. Creating the folder MyVault shows the subfolders in the MMI.

Note2: you can edit the folders to remove the main folder 'MyVault', or edit the exported CSV file and remove the 'MyVault/' string in each entry before importing into Bitwarden.