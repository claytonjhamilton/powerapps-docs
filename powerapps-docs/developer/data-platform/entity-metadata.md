---
title: Table definitions in Microsoft Dataverse | Microsoft Docs
description: Learn how to programmatically work with table definitions in Microsoft Dataverse.
author: NHelgren
ms.author: nhelgren
ms.topic: article
ms.date: 03/24/2021
ms.subservice: dataverse-developer
ms.reviewer: jdaly
search.audienceType: 
  - developer
search.app: 
  - PowerApps
  - D365CE
contributors:
 - JimDaly
---

# Table definitions in Microsoft Dataverse

Each table provides the capability to store structured data. For developers, tables correspond to the classes you will use when working with data in Microsoft Dataverse.

[!INCLUDE[cc-terminology](includes/cc-terminology.md)]

## Table names

Each table has a unique name defined when it is created. This name is presented in several ways:

|Name Property |Description|
|---------|---------|
|`SchemaName`|Typically, a Pascal cased version of the logical name. i.e. Account|
|`CollectionSchemaName`|A plural form of the Schema name. i.e. Accounts|
|`LogicalName`|All lower-case version of the schema name. i.e. account|
|`LogicalCollectionName`|All lower-case version of the collection schema name. i.e. accounts|
|`EntitySetName`|Used to identify collections with the Web API. By default, it is the same as the logical collection name.<br />It is possible to change the Entity Set name by programmatically updating the definitions. But this should only be done before any Web API code is written for the table. More information: [Web API types and operations > Change the name of an entity set](/dynamics365/customer-engagement/developer/webapi/web-api-types-operations#change-the-name-of-an-entity-set)|
>[!NOTE]
>`EntitySetName` is automatically generated by changing the `EntityName` to the plural of that name. Example: EntityName: Car EntitySetName: Cars. When manually creating a new table the `EntitySetName` can be changed. System created `EntitySetName`(s) will always be the plural of the `EntityName` and these cannot be changed through the client interface or APIs. This includes system tables as well as tables created for many to many relationship intersects.<p/>
> When you create a custom table, the name you choose will be prepended with the customization prefix value of the solution publisher associated with the solution that the table was created within. Other than the entity set name, you cannot change the names of a table after it is created. If you want consistent names for definition items in your solution, you should create them in the context of a solution you create associated with a solution publisher that contains the customization prefix you want to use. More information : [Solution publisher](/power-platform/alm/solution-concepts-alm#solution-publisher)

Each table also has three properties that can display localized values:

|Name  |Description  |
|---------|---------|
|`DisplayName`|Typically, the same as the schema name, but can include spaces. i.e. Account|
|`DisplayCollectionName`|A plural form of the display name. i.e. Accounts|
|`Description`|A short sentence describing the table i.e. *Business that represents a customer or potential customer. The company that is billed in business transactions.*|

These are the localizable values that are used to refer to the tables in an app. These values can be changed at any time. To add or edit localized values see  [Dataverse Customization Guide: Translate customized table and column text into other languages](/dynamics365/customer-engagement/customize/export-customized-entity-field-text-translation).


## Primary key

The `PrimaryIdAttribute` property value is the logical name of the column that is the primary key for the table.

By default, all tables have a single GUID unique identifier column. This is usually named *&lt; table logical name &gt;*+ `Id`.

## Primary name

The `PrimaryNameAttribute` property value is the logical name of the column that stores the string value that identifies the table record. This is the value that will be displayed in a link to open the record in a UI.

**Example**: The Contact table uses the `fullname` column as the primary name column.

> [!NOTE]
> Not every table will have a primary name. Some tables are not intended to be displayed in a UI.

## Entity images

The `PrimaryImageAttribute` property value is the logical name of the column that stores the image data for the table record. Each table can have only one image column and the logical name of that column is always `entityimage`.

**Example**: The [Contact table](reference/entities/contact.md) [EntityImage](reference/entities/contact.md#BKMK_EntityImage) column can store a picture of the contact.

For performance reasons, entity images are not included in retrieve operations unless explicitly requested.

Each table that supports entity images will have three supporting columns.

|SchemaName|Type|Description|
|--|--|--|
|`EntityImage_Timestamp` |`BigIntType`|The value represents when the image was last updated and is used to help make sure that the latest version of the image is downloaded and cached on the client.|
|`EntityImage_URL`|`StringType`|An absolute URL to display the entity image in a client.|
|`EntityImageId`|`UniqueIdentifierType`|The unique identifier of the image.|

More information: [Image columns](image-attributes.md), [Sample: Set and retrieve entity images](org-service/samples/set-retrieve-entity-images.md)

> [!NOTE]
> This is different from the icon displayed for a table in model-driven apps. The `IconVectorName` property contains the name of the SVG web resource that sets this.

## Types of tables

The capabilities and behavior of tables depends on several table properties. Most of these properties are relatively simple and have descriptive names. Among four properties that require some additional explanation are: *Ownership*, *Activity tables*, *Activityparty table* and *Child tables*.

### Table ownership

Tables can be categorized by how the data within them is owned. This is an important concept related to how security is applied to tables. This information is in the `OwnershipType` property. The following table describes the different ownership types:

|Ownership Type  |Description  |
|---------|---------|
|Business|Data belongs to the Business unit. Access to the data can be controlled at the business unit level.|
|None|Data not owned by another table.|
|Organization|Data belongs to the organization. Access to the data is controlled at the organization level.|
|User or Team|Data belongs to a user or a team. Actions that can be performed on these records can be controlled on a user level.|

When you create new tables, the only ownership options are: **Organization** or **User or Team**. Once you make a choice for this option, you cannot change it. Choose **User or Team** for the most granular control over who can view or perform actions on the records. Choose **Organization** when this level of control is not necessary. 

### Activity tables

An activity is a task performed, or to be performed, by a user. An activity is any action for which an entry can be made on a calendar. 

Activities are modeled differently from other kinds of tables that store business data. Data about activities is frequently displayed together in a list, yet each kind of activity requires unique properties. Rather than have a single Activity table with every possible property, there are separate kinds of activity tables and each of them inherits properties from a base [ActivityPointer table](reference/entities/activitypointer.md). These tables will have the `IsActivity` property set to true.


|Table |Description  |
|---------|---------|
|[Appointment](reference/entities/appointment.md)|Commitment representing a time interval with start/end times and duration.|
|[Email](reference/entities/email.md)|Activity that is delivered using email protocols.|
|[Fax](reference/entities/fax.md)|Activity that tracks call outcome and number of pages for a fax and optionally stores an electronic copy of the document.|
|[Letter](reference/entities/letter.md)|Activity that tracks the delivery of a letter. The activity can contain the electronic copy of the letter.|
|[PhoneCall](reference/entities/phonecall.md)|Activity to track a telephone call.|
|[RecurringAppointmentMaster](reference/entities/recurringappointmentmaster.md)|The Master appointment of a recurring appointment series.|
|[SocialActivity](reference/entities/socialactivity.md)|For internal use only.|
|[Task](reference/entities/task.md)|Generic activity representing work needed to be done.|

Whenever anyone creates one of these kinds of activity table records, a corresponding `ActivityPointer` table record will be created with the same `ActivityId` unique identifier column value. You cannot create, update, or delete instances of the `ActivityPointer` table, but you can retrieve them. This is what allows all types of activities to be presented together in a list.

You can create custom activity tables that behave the same way.
<!-- TODO: Add link to topic about creating an activity table -->

### ActivityParty table

This table is used to add structure to activity table `PartyListType` columns that include references to other tables. You will use this table when setting values for activity table columns like `Email.to` or `PhoneCall.from`. Within the [ActivityParty table](reference/entities/activityparty.md), you set the [ParticipationTypeMask](reference/entities/activityparty.md#BKMK_ParticipationTypeMask) column to define the role that the reference is playing. Roles include `Sender`, `ToRecipient`, `Organizer` and more.

You can query the `ActivityParty` table, but you cannot create, retrieve, update, or delete an activity party outside of the activity that it is related to.
More information: 
- [ActivityParty entity](/dynamics365/customer-engagement/developer/activityparty-entity).


### Child tables

Tables where the `IsChildEntity` property is true will never have any privileges defined and will never be User or Team owned. Operations that can be performed on a child table are bound to a table that they are associated to via a Many-to-one relationship. Users can only perform operations on child tables if they can perform the same operation on the related table. Child tables get deleted automatically when the table record they depend on is deleted.

For example, `PostComment`, `PostLike`, and `PostRole` are each children of the `Post` table.

## Keys

Each alternate key definition describes one or more columns in combination that will uniquely identify a table. Alternate keys are typically only applied for integration with external systems. You can define alternate keys to uniquely identify a record. This is valuable if you are integrating data with a system that doesn't support GUID unique identifier keys. You can define a single column value or combination of column values to uniquely identify a table. Adding an alternate key will enforce a uniqueness constraint on these columns. You will not be able to create or update another table record to have the same values.

More information: 
 - [Dataverse Customization Guide: Define alternate keys to reference Dataverse records](/dynamics365/customer-engagement/customize/define-alternate-keys-reference-records)
 - [Define alternate keys for a table and Developer Guide: Synchronize Dataverse data with external systems](/dynamics365/customer-engagement/developer/synchronize-dynamics-365-data-with-external-systems)

## Table states

Most tables have two properties to track the state of a record. These are `StateCode`, which is called **Status** in model-driven apps and `StatusCode`, which is called **Status Reason** in model-driven apps. 

Both columns contain a set of choices that display the valid values. The `StateCode` and `StatusCode` column values are linked so that only certain `StatusCode` choices are valid for a given `StateCode`.

This can vary by table but the common pattern for many tables, and the default for custom tables is this:

|StateCode Choices  |StatusCode Choices  |
|---------|---------|
|0 : Active|1 : Active|
|1: Inactive|2: Inactive|

Some tables will have different sets of choices. 

**Example**: `PhoneCall` table `StateCode` and `StatusCode` choices


|StateCode|StatusCode|
|---------|---------|
|0 : Open|1: Open|
|1 : Completed|2: Made <br />4: Received|
|2: Canceled|3: Canceled|

The set of valid state codes for a table is not customizable, but the status codes are customizable. You can add additional `StatusCode` choices for a corresponding `StateCode`.

For custom tables, you can define additional criteria for valid transitions between statuses. 
More information: 
- [Customize column definitions](/dynamics365/customer-engagement/developer/customize-entity-attribute-metadata)
- [Define custom state model transitions](/dynamics365/customer-engagement/developer/define-custom-state-model-transitions).

### See also

[Dataverse tables](entities.md)


[!INCLUDE[footer-include](../../includes/footer-banner.md)]
