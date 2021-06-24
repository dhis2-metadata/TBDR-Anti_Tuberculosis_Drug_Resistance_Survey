# WHO Anti-Tuberculosis Drugs Resistance Survey (DRS) - Tracker Installation Guide

Package Version 1.0.0

DHIS2 Version compatibility 2.33 and above

## Overview

The Anti-Tuberculosis Drugs Resistance Survey tracker package was developed in DHIS2.33. If you are setting this module in a new instance, it is recommended to use later versions of DHIS2 Please refer to the [DHIS2 installation guide](https://docs.dhis2.org/en/manage/performing-system-administration/dhis-core-version-master/installation.html).

This document covers the installation of the following packages:

1. WHO Anti-Tuberculosis Drugs Resistance Survey (DRS)

You will have to follow the instructions to ensure that the package is installed and configured correctly.

## Installation

Installation of the module consists of several steps:

1. [Preparing](#preparing-the-metadata-file) the metadata file.
2. [Importing](#importing-metadata) the metadata file into DHIS2.
3. [Configuring](#additional-configuration) the imported metadata.
4. [Adapting](#adapting-the-tracker-program) the program after being imported

It is recommended to first read through each section before starting the installation and configuration process in DHIS2. Sections that are not applicable have been identified, depending on if you are importing into a new instance of DHIS2 or a DHIS2 instance with metadata already present. The procedure outlined in this document should be tested in a test/staging environment before either being repeated or transferred to a production instance of DHIS2.

## Requirements

In order to install the module, an administrator user account on DHIS2 is required. The procedure outlined in this document should be tested in a test/staging environment before being performed on a production instance of DHIS2.

Great care should be taken to ensure that the server itself and the DHIS2 application is well secured, to restrict access to the data being collected. Details on securing a DHIS2 system is outside the scope of this document, and we refer to the [DHIS2 documentation](https://docs.dhis2.org/).

## Preparing the metadata file

**NOTE:** If you are installing the package on a new instance of DHIS2, you can skip the “Preparing the metadata file” section and move immediately to the section [Importing a metadata file into DHIS2](#importing-metadata)

While not always necessary, it can often be advantageous to make certain modifications to the metadata file before importing it into DHIS2.

### Default data dimension (if applicable)

In early versions of DHIS2, the UID of the default data dimension was auto-generated. Thus, while all DHIS2 instances have a default category option, data element category, category combination and category option combination, the UIDs of these defaults can be different. Later versions of DHIS2 have hardcoded UIDs for the default dimension, and these UIDs are used in the configuration packages.

To avoid conflicts when importing the metadata, it is advisable to search and replace the entire .json file for all occurrences of these default objects, replacing UIDs of the .json file with the UIDs of the database in which the file will be imported. Table 1 shows the UIDs which should be replaced, as well as the API endpoints to identify the existing UIDs.

|Object|UID|API endpoint|
|:--|:--|:--|
|Category|GLevLNI9wkl|`../api/categories.json?filter=name:eq:default`|
|Category option|xYerKDKCefk|`../api/categoryOptions.json?filter=name:eq:default`|
|Category combination|bjDvmb4bfuf|`../api/categoryCombos.json?filter=name:eq:default`|
|Category option combination|HllvX50cXC0|`../api/categoryOptionCombos.json?filter=name:eq:default`|

For example, if importing a configuration package into <https://play.dhis2.org/demo>, the UID of the default category option combination could be identified through <https://play.dhis2.org/demo/api/categoryOptionCombos.json?filter=name:eq:default> as bRowv6yZOF2.

You could then search and replace all occurrences of HllvX50cXC0 with bRowv6yZOF2 in the .json file, as that is the ID of default in the system you are importing into. **_Note that this search and replace operation must be done with a plain text editor_**, not a word processor like Microsoft Word.

### Indicator types

Indicator type is another type of object that can create import conflict because certain names are used in different DHIS2 databases (.e.g "Percentage"). Since Indicator types are defined simply by their factor and whether or not they are simple numbers without a denominator, they are unambiguous and can be replaced through a search and replace of the UIDs. This avoids potential import conflicts, and avoids creating duplicate indicator types. Table 2 shows the UIDs which could be replaced, as well as the API endpoints to identify the existing UIDs

|Object|UID|API endpoint|
|:--|:--|:--|
|Percentage|hmSnCXmLYwt|`../api/indicatorTypes.json?filter=number:eq:false&filter=factor:eq:100`|

### Tracked Entity Type

Like indicator types, you may have already existing tracked entity types in your DHIS2 database. The references to the tracked entity type should be changed to reflect what is in your system so you do not create duplicates. Table 3 shows the UIDs which could be replaced, as well as the API endpoints to identify the existing UIDs

|Object|UID|API endpoint|
|:--|:--|:--|
|Person|MCPQUTHX1Ze|`../api/trackedEntityTypes.json?filter=name:eq:Person`|

## Importing metadata

The .json metadata file is imported through the [Import/Export](https://docs.dhis2.org/en/use/user-guides/dhis-core-version-master/maintaining-the-system/importexport-app.html) app of DHIS2. It is advisable to use the "dry run" feature to identify issues before attempting to do an actual import of the metadata. If "dry run" reports any issues or conflicts, see the [import conflicts](#handling-import-conflicts) section below.

If the "dry run"/"validate" import works without error, attempt to import the metadata. If the import succeeds without any errors, you can proceed to [configure](#additional-configuration) the module. In some cases, import conflicts or issues are not shown during the "dry run", but appear when the actual import is attempted. In this case, the import summary will list any errors that need to be resolved.

### Handling import conflicts

**NOTE:** If you are importing into a new DHIS2 instance, you will not have to worry about import conflicts, as there is nothing in the database you are importing to to conflict with. Follow the instructions to import the metadata then please proceed to the “[Additional configuration](#additional-configuration)” section.

There are a number of different conflicts that may occur, though the most common is that there are metadata objects in the configuration package with a name, shortname and/or code that already exists in the target database. There are a couple of alternative solutions to these problems, with different advantages and disadvantages. Which one is more appropriate will depend, for example, on the type of object for which a conflict occurs.

**_Alternative 1_**

Rename the existing object in your DHIS2 database for which there is a conflict. The advantage of this approach is that there is no need to modify the .json file, as changes are instead done through the user interface of DHIS2. This is likely to be less error prone. It also means that the configuration package is left as is, which can be an advantage for example when training material and documentation based on the configuration package will be used.

**_Alternative 2_**

Rename the object for which there is a conflict in the .json file. The advantage of this approach is that the existing DHIS2 metadata is left as-is. This can be a factor when there is training material or documentation such as SOPs of data dictionaries linked to the object in question, and it does not involve any risk of confusing users by modifying the metadata they are familiar with.

Note that for both alternative 1 and 2, the modification can be as simple as adding a small pre/post-fix to the name, to minimise the risk of confusion.

**_Alternative 3_**

A third and more complicated approach is to modify the .json file to re-use existing metadata. For example, in cases where an option set already exists for a certain concept (e.g. "sex"), that option set could be removed from the .json file and all references to its UID replaced with the corresponding option set already in the database. The big advantage of this (which is not limited to the cases where there is a direct import conflict) is to avoid creating duplicate metadata in the database. There are some key considerations to make when performing this type of modification:

* it requires expert knowledge of the detailed metadata structure of DHIS2
* the approach does not work for all types of objects. In particular, certain types of objects have dependencies which are complicated to solve in this way, for example related to disaggregations.
* future updates to the configuration package will be complicated.

### Additional configuration

Once all metadata has been successfully imported, there are a few steps that need to be taken before the module is functional.

### Sharing

First, you will have to use the _Sharing_ functionality of DHIS2 to configure which users (user groups) should see the metadata and data associated with the programme as well as who can register/enter data into the program. By default, sharing has been configured for the following:

* Tracked entity type
* Program
* Program stages
* Dashboards

There are four user groups that come with the package:

* DRS - admin
* DRS - access
* DRS - data capture

By default the following is assigned to these user groups

|Object|User Groups| | |
|:--|:--|:--|:--|
| |TB access|TB admin|TB data capture|
|Tracked entity type|**Metadata:** can view <br> **Data:** can view|**Metadata:** can edit and view <br> **Data:** can view|**Metadata:** can view <br> **Data:** can capture and view|
|Program|**Metadata:** can view <br> **Data:** can view|**Metadata:** can edit and view <br> **Data:** can view|**Metadata:** can view <br> **Data:** can capture and view|
|Program Stages|**Metadata:** can view <br> **Data:** can view|**Metadata:** can edit and view <br> **Data:** can view|**Metadata:** can view <br> **Data:** can capture and view|
|Dashboards|**Metadata:** can view <br> **Data:** can view|**Metadata:** can edit and view <br> **Data:** can view|**Metadata:** can view <br> **Data:** can view|

You will want to assign your users to the appropriate user group based on their role within the system. You may want to enable sharing for other objects in the package depending on your set up. Refer to the [DHIS2 Documentation](https://docs.dhis2.org/en/use/user-guides/dhis-core-version-master/configuring-the-system/about-sharing-of-objects.html) for more information on configuring sharing.

### User Roles

Users will need user roles in order to engage with the various applications within DHIS2. The following minimum roles are recommended:

1. Tracker data analysis : Can see event analytics and access dashboards, event reports, event visualizer, data visualizer, pivot tables, reports and maps.
2. Tracker data capture : Can add data values, update tracked entities, search tracked entities across org units and access tracker capture

Refer to the [DHIS2 Documentation](https://docs.dhis2.org/) for more information on configuring user roles.

### Organisation Units

You must assign the program to organisation units within your own hierarchy in order to be able to see the program in tracker capture.

### Duplicated metadata

**NOTE:** This section only applies if you are importing into a DHIS2 database in which there is already meta-data present. If you are working with a new DHIS2 instance, you may skip this section.

Even when metadata has been successfully imported without any import conflicts, there can be duplicates in the metadata - data elements, tracked entity attributes or option sets that already exist. As was noted in the section above on resolving conflict, an important issue to keep in mind is that decisions on making changes to the metadata in DHIS2 also needs to take into account other documents and resources that are in different ways associated with both the existing metadata, and the metadata that has been imported through the configuration package. Resolving duplicates is thus not only a matter of "cleaning up the database", but also making sure that this is done without, for example, breaking potential integrating with other systems, the possibility to use training material, breaking SOPs etc. This will very much be context-dependent.

One important thing to keep in mind is that DHIS2 has tools that can hide some of the complexities of potential duplications in metadata. For example, where duplicate option sets exist, they can be hidden for groups of users through [sharing](https://docs.dhis2.org/en/use/user-guides/dhis-core-version-master/configuring-the-system/about-sharing-of-objects.html).

### Constants

WHO Anti-Tuberculosis Drugs Resistance Survey (DRS) Tracker package includes a set of tests and a list of drugs that can be modified by the implementing country according to national context (e.g. which drugs and tests are used/available in country). The use of constants and corresponding program rules enables a system admin in an implementing country to easily ‘turn on’ or ‘turn off’ types of drugs and tests depending on availability. When the complete package is installed into a DHIS2 instance, all data elements for all tests and drugs included in this package are included in the system. All constant values are initially set to **1** (enabling the related data elements for data entry) and can be set to **0** by an implementer or system admin according to country context if not needed (disabling the related data elements for data entry). If a test or drug later becomes available, the admin can simply re-enable the data elements by changing the constant from a value of **0** to a value of **1**.

#### Cluster Sampling / Exhaustive Sampling of all Health Facilities

Configure whether the country is using Cluster Sampling or Exhaustive Sampling of all Health Facilities

| Constant | Settings | UID |
|---|---|---|
| Cluster Sampling | Value "1" = Include Cluster Sampling for DRS. <br> Value "0" = Exclude Cluster Sampling from DRS. | aYPQMlxAZsz |

#### Number of DRS Samples sent to NRL

Configure the number of samples are sent to NRL processing

| Constant | Settings | UID |
|---|---|---|
| Sample 2 for NRL Processing | Value "1" = Add DRS Sample 2 collected for NRL processing to program configuration <br> Value "0" = Remove DRS Sample 2 from program configuration | rSS2dY6gwLJ |
| Sample 3 for NRL Processing | Value "1" = Add DRS Sample 3 collected for NRL processing to program configuration <br> Value "0" = Remove DRS Sample 3 from program configuration | sqgp1l0s4wp |
| Sample 4 for NRL Processing | Value "1" = Add DRS Sample 4 collected for NRL processing to program configuration <br> Value "0" = Remove DRS Sample 4 from program configuration | NXr7RH56Q1R |

#### Initial Screening Tests

Configure which initial screening tests are used for bacteriological confirmation of TB: **Sputum Smear Microscopy, Xpert MTB/RIF** or **both**.

| Constant | Settings | UID |
|---|---|---|
| Bacteriological Confirmation of TB | Value "1" = Include both "Sputum Smear Microscopy" and "Xpert MTB/RIF" for Bacteriological Confirmation of TB. <br> Value "2" = Include only "Sputum smear microscopy" for Bacteriological Confirmation of TB. <br> Value "3" = Include only "Xpert MTB/RIF" for Bacteriological Confirmation of TB. | gYj2CUoep4O |

#### SRL Sample Processing

Configure whether the country has an option of sending samples to SRL (supranational lab).

| Constant | Settings | UID |
|---|---|---|
| SRL | Value "1" = include an option of sending Samples to SRL <br> Value "0" = exclude an option of sending Samples to SRL | AUltNkQXzdm |

#### Tests at NRL

| Constant | Settings | UID |
|---|---|---|
| Sputum Smear Microscopy | Value "1" = include Sputum Smear Microscopy in the list of tests. <br> Value "0" = exclude Sputum Smear Microscopy from the list of tests. | q1ah12sKfG3 |
| Culture in Solid Media (e.g. LJ) | Value "1" = include Culture in Solid Media (e.g. LG) in the list of tests. <br> Value "0" = exclude Culture in Solid Media (e.g. LG) from the list of tests. | iSKEqcuVAui |
| Culture in Liquid Media (e.g. MGIT) | Value "1" = include Culture in Liquid Media (e.g. MGIT) in the list of tests. <br> Value "0" = exclude Culture in Liquid Media (e.g. MGIT) from the list of tests. | qpAseG5vJyS |
| Xpert MTB/RIF | Value "1" = include Xpert MTB/RIF in the list of tests. <br> Value "0" = exclude Xpert MTB/RIF from the list of tests. | H4ObQDbhnTA |
| Xpert MTB/RIF Ultra | Value "1" = include Xpert MTB/RIF Ultra in the list of tests. <br> Value "0" = exclude Xpert MTB/RIF Ultra from the list of tests. | cFoFDkXKcXC |
| Initial Phenotypic DST in Solid Media (eg. LJ) | Value "1" = include Initial Phenotypic DST in Solid Media (eg. LJ) in the list of tests. <br> Value "0" = exclude Initial Phenotypic DST in Solid Media (eg. LJ) from the list of tests. | HjN2Bgnusyy |
| Initial Phenotypic DST in Liquid Media (eg. MGIT) | Value "1" = include Initial Phenotypic DST in Liquid Media (eg. MGIT) in the list of tests. <br> Value "0" = exclude Initial Phenotypic DST in Liquid Media (eg. MGIT) from the list of tests. | OQxeAIyQUeB |
| Subsequent Phenotypic DST in Solid Media (eg. LJ) | Value "1" = include Subsequent Phenotypic DST in Solid Media (eg. LJ) in the list of tests. <br> Value "0" = exclude Subsequent Phenotypic DST in Solid Media (eg. LJ) from the list of tests. | BpRfvWQcvTo |
| LPA (Rif/Inh) | Value "1" = include LPA (Rif/Inh) in the list of tests. <br> Value "0" = exclude LPA (Rif/Inh) from the list of tests. | ESUffSPwmju |
| LPA (Fq/2LI) | Value "1" = include LPA (Fq/2LI) in the list of tests. <br> Value "0" = exclude LPA (Fq/2LI) from the list of tests. | govArZqiFzY |
| Subsequent Phenotypic DST in Liquid Media (eg. MGIT) | Value "1" = include Subsequent Phenotypic DST in Liquid Media (eg. MGIT) in the list of tests. <br> Value "0" = exclude Subsequent Phenotypic DST in Liquid Media (eg. MGIT) from the list of tests. | W8Fm1pJuJPL |
| Targeted Gene Sequencing | Value "1" = include Targeted Gene Sequencing in the list of tests. <br> Value "0" = exclude Targeted Gene Sequencing from the list of tests. | KkypNdLNW1f |
| Whole Genome Sequencing | Value "1" = include WGS in the list of tests. <br> Value "0" = exclude WGS from the list of tests. | MC7QUDr9YKC |

#### List of Available Drugs

| Initial Phenotypic DST in Solid Media (eg. LJ) |||
|---|---|---|
| Constant | Settings | UID |
| Initial Phenotypic DST in Solid Media - Rifampicin | Value "1" = include Rifampicin in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Rifampicin from the list of drugs available for Initial Phenotypic DST testing in solid media. | Q67DDsupq7v |
| Initial Phenotypic DST in Solid Media - Isoniazid (CC) | Value "1" = include Isoniazid (CC) in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Isoniazid (CC) from the list of drugs available for Initial Phenotypic DST testing in solid media. | dAPAScvFPfX |
| Initial Phenotypic DST in Solid Media - Isoniazid (CB) | Value "1" = include Isoniazid (CB) in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Isoniazid (CB) from the list of drugs available for Initial Phenotypic DST testing in solid media. | anpSc1tmlft |
| Initial Phenotypic DST in Solid Media - Pyrazinamide | Value "1" = include Pyrazinamide in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Pyrazinamide from the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Pyrazinamide can only be tested using liquid media. | aCaNdqUIDZl |
| Initial Phenotypic DST in Solid Media - Ethambutol | Value "1" = include Ethambutol in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Ethambutol from the list of drugs available for Initial Phenotypic DST testing in solid media. | yxPHZFwMTN6 |
| Initial Phenotypic DST in Solid Media - Levofloxacin | Value "1" = include Levofloxacin in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Levofloxacin from the list of drugs available for Initial Phenotypic DST testing in solid media. | NlOX3oV4gWe |
| Initial Phenotypic DST in Solid Media - Moxifloxacin (CC) | Value "1" = include Moxifloxacin (CC) in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Moxifloxacin (CC) from the list of drugs available for Initial Phenotypic DST testing in solid media. | RJNE1o9cw7Y |
| Initial Phenotypic DST in Solid Media - Moxifloxacin (CB) | Value "1" = include Moxifloxacin (CB) in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Moxifloxacin (CB) from the list of drugs available for Initial Phenotypic DST testing in solid media. | m4c79OHEKCG |
| Initial Phenotypic DST in Solid Media - Amikacin | Value "1" = include Amikacin in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Amikacin from the list of drugs available for Initial Phenotypic DST testing in solid media. | XO1V9o5C95J |
| Initial Phenotypic DST in Solid Media - Bedaquiline | Value "1" = include Bedaquiline in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Bedaquiline from the list of drugs available for Initial Phenotypic DST testing in solid media. | UxcTzMRQsfa |
| Initial Phenotypic DST in Solid Media - Delamanid | Value "1" = include Delamanid in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Delamanid from the list of drugs available for Initial Phenotypic DST testing in solid media. | VUlsMrm4D8k |
| Initial Phenotypic DST in Solid Media - Linezolid | Value "1" = include Linezolid in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Linezolid from the list of drugs available for Initial Phenotypic DST testing in solid media. | CVQR2ZtZWxk |
| Initial Phenotypic DST in Solid Media - Clofazimine | Value "1" = include Clofazimine in the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Value "0" = exclude Clofazimine from the list of drugs available for Initial Phenotypic DST testing in solid media. <br> Clofazimine can only be tested using liquid media. | ZDpLleSK08x |

| Initial Phenotypic DST in Liquid Media (eg. MGIT) |||
|---|---|---|
| Initial Phenotypic DST in Liquid Media - Rifampicin | Value "1" = include Rifampicin in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Rifampicin from the list of drugs available for Initial Phenotypic DST testing in liquid media. | uOJYEwV7XfN |
| Initial Phenotypic DST in Liquid Media - Isoniazid (CC) | Value "1" = include Isoniazid (CC) in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Isoniazid (CC) from the list of drugs available for Initial Phenotypic DST testing in liquid media. | cRldkiBh5xv |
| Initial Phenotypic DST in Liquid Media - Isoniazid (CB) | Value "1" = include Isoniazid (CB) in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Isoniazid (CB) from the list of drugs available for Initial Phenotypic DST testing in liquid media. | lqaf0KfpHGd |
| Initial Phenotypic DST in Liquid Media - Pyrazinamide | Value "1" = include Pyrazinamide in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Pyrazinamide from the list of drugs available for Initial Phenotypic DST testing in liquid media. | FDtk4otxDse |
| Initial Phenotypic DST in Liquid Media - Ethambutol | Value "1" = include Ethambutol in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Ethambutol from the list of drugs available for Initial Phenotypic DST testing in liquid media. | n9zOOsLO0QP |
| Initial Phenotypic DST in Liquid Media - Levofloxacin | Value "1" = include Levofloxacin in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Levofloxacin from the list of drugs available for Initial Phenotypic DST testing in liquid media. | t7Okdm8VgDV |
| Initial Phenotypic DST in Liquid Media - Moxifloxacin (CC) | Value "1" = include Moxifloxacin (CC) in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Moxifloxacin (CC) from the list of drugs available for Initial Phenotypic DST testing in liquid media. | dgjpdQO2Iva |
| Initial Phenotypic DST in Liquid Media - Moxifloxacin (CB) | Value "1" = include Moxifloxacin (CB) in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Moxifloxacin (CB) from the list of drugs available for Initial Phenotypic DST testing in liquid media. | UL61U78GWCg |
| Initial Phenotypic DST in Liquid Media - Amikacin | Value "1" = include Amikacin in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Amikacin from the list of drugs available for Initial Phenotypic DST testing in liquid media. | x6v8O6EZo0U |
| Initial Phenotypic DST in Liquid Media - Bedaquiline | Value "1" = include Bedaquiline in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Bedaquiline from the list of drugs available for Initial Phenotypic DST testing in liquid media. | KjvwSybuWJU |
| Initial Phenotypic DST in Liquid Media - Delamanid | Value "1" = include Delamanid in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Delamanid from the list of drugs available for Initial Phenotypic DST testing in liquid media. | BJTzi3WWjhT |
| Initial Phenotypic DST in Liquid Media - Linezolid | Value "1" = include Linezolid in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Linezolid from the list of drugs available for Initial Phenotypic DST testing in liquid media. | aZq11UuXPP6 |
| Initial Phenotypic DST in Liquid Media - Clofazimine | Value "1" = include Clofazimine in the list of drugs available for Initial Phenotypic DST testing in liquid media. <br> Value "0" = exclude Clofazimine from the list of drugs available for Initial Phenotypic DST testing in liquid media. | GL5JTtBvbEC |

| Subsequent Phenotypic DST in Solid Media (eg. LJ) |||
|---|---|---|
| Constant | Settings | UID |
| Subsequent Phenotypic DST in Solid Media - Rifampicin | Value "1" = include Rifampicin in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Rifampicin from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | zJoEjvstp2d |
| Subsequent Phenotypic DST in Solid Media - Isoniazid (CC) | Value "1" = include Isoniazid (CC) in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Isoniazid (CC) from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | eM3UfUO7W8O |
| Subsequent Phenotypic DST in Solid Media - Isoniazid (CB) | Value "1" = include Isoniazid (CB) in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Isoniazid (CB) from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | s8WPR943gGS |
| Subsequent Phenotypic DST in Solid Media - Pyrazinamide | Value "1" = include Pyrazinamide in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Pyrazinamide from the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Pyrazinamide can only be tested using liquid media. | bHPYGfrFNV2 |
| Subsequent Phenotypic DST in Solid Media - Ethambutol | Value "1" = include Ethambutol in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Ethambutol from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | wNxsAieLWYc |
| Subsequent Phenotypic DST in Solid Media - Levofloxacin | Value "1" = include Levofloxacin in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Levofloxacin from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | Fw8wOlTETPt |
| Subsequent Phenotypic DST in Solid Media - Moxifloxacin (CC) | Value "1" = include Moxifloxacin (CC) in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Moxifloxacin (CC) from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | Czx5FuOqF9i |
| Subsequent Phenotypic DST in Solid Media - Moxifloxacin (CB) | Value "1" = include Moxifloxacin (CB) in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Moxifloxacin (CB) from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | NvA1K4hFJbc |
| Subsequent Phenotypic DST in Solid Media - Amikacin | Value "1" = include Amikacin in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Amikacin from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | dTwKm1u2VhY |
| Subsequent Phenotypic DST in Solid Media - Bedaquiline | Value "1" = include Bedaquiline in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Bedaquiline from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | r70ONQRhaHY |
| Subsequent Phenotypic DST in Solid Media - Delamanid | Value "1" = include Delamanid in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Delamanid from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | RFqmLP5W5di |
| Subsequent Phenotypic DST in Solid Media - Linezolid | Value "1" = include Linezolid in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Linezolid from the list of drugs available for Subsequent Phenotypic DST testing in solid media. | KIeI2d8xalG |
| Subsequent Phenotypic DST in Solid Media - Clofazimine | Value "1" = include Clofazimine in the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Value "0" = exclude Clofazimine from the list of drugs available for Subsequent Phenotypic DST testing in solid media. <br> Clofazimine can only be tested using liquid media. | bywBn5DxVdi |

| Subsequent Phenotypic DST in Liquid Media (eg. MGIT) |||
|---|---|---|
| Subsequent Phenotypic DST in Liquid Media - Rifampicin | Value "1" = include Rifampicin in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Rifampicin from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | lelJEADYpeI |
| Subsequent Phenotypic DST in Liquid Media - Isoniazid (CC) | Value "1" = include Isoniazid (CC) in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Isoniazid (CC) from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | ZBpqH7xJLBO |
| Subsequent Phenotypic DST in Liquid Media - Isoniazid (CB) | Value "1" = include Isoniazid (CB) in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Isoniazid (CB) from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | F9DTb5zl8rS |
| Subsequent Phenotypic DST in Liquid Media - Pyrazinamide | Value "1" = include Pyrazinamide in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Pyrazinamide from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | VY15auehssY |
| Subsequent Phenotypic DST in Liquid Media - Ethambutol | Value "1" = include Ethambutol in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Ethambutol from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | Ic5asMdbpH8 |
| Subsequent Phenotypic DST in Liquid Media - Levofloxacin | Value "1" = include Levofloxacin in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Levofloxacin from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | b1KgVOel21P |
| Subsequent Phenotypic DST in Liquid Media - Moxifloxacin (CC) | Value "1" = include Moxifloxacin (CC) in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Moxifloxacin (CC) from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | HhKitGif8RV |
| Subsequent Phenotypic DST in Liquid Media - Moxifloxacin (CB) | Value "1" = include Moxifloxacin (CB) in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Moxifloxacin (CB) from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | xfltmyqC3p0 |
| Subsequent Phenotypic DST in Liquid Media - Amikacin | Value "1" = include Amikacin in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Amikacin from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | a4PpEfSutcV |
| Subsequent Phenotypic DST in Liquid Media - Bedaquiline | Value "1" = include Bedaquiline in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Bedaquiline from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | QaioqMbO0TX |
| Subsequent Phenotypic DST in Liquid Media - Delamanid | Value "1" = include Delamanid in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Delamanid from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | Xt5GU9kBD7q |
| Subsequent Phenotypic DST in Liquid Media - Linezolid | Value "1" = include Linezolid in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Linezolid from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | mU5iEABeF2K |
| Subsequent Phenotypic DST in Liquid Media - Clofazimine | Value "1" = include Clofazimine in the list of drugs available for Subsequent Phenotypic DST testing in liquid media. <br> Value "0" = exclude Clofazimine from the list of drugs available for Subsequent Phenotypic DST testing in liquid media. | Nb6oHqjCCZq |

| LPA (Fluoroquinolones / Second-line Injectables) |||
|---|---|---|
| LPA (Fq/2Li) - Ethambutol | Value "1" = include Ethambutol Result in LPA (Fq/2LI) Test. <br> Value "0" = exclude Ethambutol Result from LPA (Fq/2LI) Test. | AiyTLOJHMkl |

Please note, that the constants help to hide certain sections and data element from the data entry without the need to delete them from the system/metadata.json file. However, the elements are still assigned to the program and will be included when exporting all data into  eg. csv files.

### Dashboards and visualizations

The dashboards and visualizations in each dashboard cannot be hidden/unhidden by constants. Therefore the unapplicable dashboards/visualizations have to be manually removed during package installation based on the selection of tests/drugs.

Some report tables are using Organisation Unit Levels in the configuration. These are dependent on the ROOT Organisation Unit.
The following report tables contain a placeholder <OU_ROOT_UID>
This placeholder has to be replaced by the ROOT Organisation unit UID in the metadata.json file before the package can be imported.

```json
    ../api/reportTables/CrxqCiQyhay

    ../api/reportTables/PazB6GSITwK

    ../api/reportTables/GinRhTtdKnI
```

### Configuring tracker capture interface, widgets and top bar

You must configure tracker capture dashboard after the package has been installed. This configuration includes data entry forms, widgets and top bar.

#### Data entry forms

* After registering the first (test) case, access the **Settings** menu in the tracker capture form and select **Show/Hide Widgets**
* Switch from **Timeline Data Entry** to **Tabular Data Entry**
* Make sure that **Enrollment**, **Feedback** and **Profile** widgets are selected. Click **Close**.
* Adjust the widgets on the screen.

#### Top Bar

* Access the **Settings** menu and select **Top bar settings**
* Select **Activate top bar**
* Select required information fields and assign their **Sort order**

| Recommended fields | Sort order |
|:--|:--|
| **Attributes** |  |
| Sex | 5 |
| **Indicators** |  |
| DRS ID (Attribute) | 1 |
| DRS ID (concatenated) | 2 |
|Patient's age (months)| 3 |
|Patient's age (years) | 4 |
| Treatment History | 6 |
|Resistance | 7 |

* Click **Save**
* Return to the **Settings** menu. Click **Saved dashboard layout as default**. Lock layout for all users.

#### Adapting the tracker program

Once the programme has been imported, you might want to make certain modifications to the programme. Examples of local adaptations that _could_ be made include:

* Adding additional variables to the form.
* Adapting data element/option names according to national conventions.
* Adding translations to variables and/or the data entry form.
* Modifying program indicators based on local case definitions.

However, it is strongly recommended to take great caution if you decide to change or remove any of the included form/metadata. There is a danger that modifications could break functionality, for example program rules and program indicators.
