# D2L BrightSpace IPSIS exports from PowerSchool SIS

Feb/March 2022, Aaron Ciuffo

## Implementation Notes

Exports are managed through PowerSchool PowerQuery Plugins. Plugins follow the structure outlined below. Each CSV Export for BrightSpace is managed through an individual plugin. Each plugin contains an SQL query that matches the required fields for the CSV.

## Setup and Installation

### SIS Installation

1. Download plugins from this GIT Repo; each plugin is stored as a .zip file
   - See [list below](#list-of-plugins-and-functions) for all plugins and their fuctions
2. Install plugins throught the _Plugin Management Dashboard_: 
   - **Start Page > System Administrator > System Settings > Plugin Management Dashboard**
   - If the plugin is already installed either choose to _Update_ or _Delete_ and reinstall using the .zip files
3. Configure a template for automated export in [_Data Export Manager_](#data-export-manager-configuration)
   - **Start Page > System Administrator > Page and Data Management > Data Export Manager**

### Data Export Manager Configuration

**Start Page > System Administrator > Page and Data Management > Data Export Manager**

Each plugin needs to be configured to produce CSV files with the appropriate data, characterset and column headers. Each plugin documents the structure and settings under the **Data Export Manager Setup** heading. See the `README.md` in each plugin directory for more details.

The basic settings are as follows:

1. Select Columns to Export:
   - **Category:** _Show All_
   - **Export From:** _NQ com.txoof.brightspace.table.area_ (see the DEM section in each readme)
   - ![DEM Screen 00](./documentation/DEM_00.png)
2. Select all of the fields:
   - ![DEM Screen 01 - Fields](./documentation/DEM_01.png)
3.  Remove the `TABLE.` portion in the _Labels used on Export_ for every column (highlighted in blue/red) and click _Next_
   - ![DEM Screen 02 - labels](./documentation/DEM_02.png)
4. On the following _Select/Edit Records from NQ - com.txoof.brightspace.table.area_ screen click _Next_
    - No filtering should be needed
5. On the following _Export Summary and Output Options_ screen set:
   - **Export File Name**: _See README.md_ for filename format (e.g. `1-Other.csv`)
   - **Line Delimiter**: `CR-LF`
   - **Field Delimiter**: `Comma`
   - **Character Set**: `UTF-8`
6. Click _Save Template_; see the `README.md` for each plugin for specifics
   - **Name**: `1-Other-%d.csv Export`
   - **Description**: `Updated: YYYY.MM.DD`
7. Click _Save as New_

## Automated Exports from PSL to BrightSpace

### In Progress Notes

BrightSpace accepts imports via IPSIS. IPSIS expects flat zip files with at minimum 8 CSV files (1-Other, 2-Departments, 3-Semesters, 4-Templates, 5-Offerings, 6-Sections, 7-Users, 8-Enrollments) and a `manifest.json`. 

`manifest.json`

```JSON
{
  "version":"2.0"
}
```

Files are sent to IPSIS via SFTP. Find SFTP details within the platform [here](https://lms.ash.nl/d2l/im/ipsis/admin/console/integration/3/dashboard)


## List of Plugins and Functions

- [BS_Organization](./BS_Organization/README_Organization.md): Export BrightSpace CSVs 1-6
  - 1: Other; 2: Departments; 3: Semesters; 4: Templates, 5: Offerings; 6: Sections
- [BS_Users-Enrollments](./BS_Users_Enrollments/README_Users_Enrollments.md): Export BrightSpace CSVs 7-8
  - 7: Users, 8: Enrollments

## Plugin Documentation

### Updating a Plugin

The Named Queries (NQ) within each plugin can be updated by editing the associated `xxx.named_query.xml` file. No changes to the files in the `permissions_root` should be necessary unless a new NQ is added. See the [Basic PowerQuery Plugin Structure](#basic-powerquery-plugin-structure) section below for more information.

After updating the NQ, it is necessary to update the version number in the `plugin.xml` file. PowerSchool will complain during the upgrade process if the version number remains the same or regresses. Always increment the version number.

PowerSchool SIS is very particular about the package structure of the plugin zip file. Use the `./package` script included in this repository to repackage the script. The package script will generate a new plugin with the current version number and the current time specified in the zip filename. 

Usage: `package.sh PLUGIN_DIR`

Example:
```SHELL
$ ./package.sh BS_Organization
```

### Reference Documentation

- [BrightSpace CSV v2.0](https://community.brightspace.com/s/article/D2L-Standard-CSV-Version-2-0-Administrator-Guide)
- [Plugins](https://support.powerschool.com/developer/#/page/plugins)
- [Plugin XML](https://support.powerschool.com/developer/#/page/plugin-xml)
- [Plugin Setup](https://support.powerschool.com/developer/#/page/plugin-zip)
- [Permission Mapping](https://support.powerschool.com/developer/#/page/permission-mapping)
- [PowerQueries](https://support.powerschool.com/developer/#/page/powerqueries)

### Basic PowerQuery Plugin Structure

Plugins must be zipped such that the `plugin.xml` file is at the root of the structure with no top level folder. Multiple Named Queries (NQ) can be specified within one plugin. Each NQ needs to be placed in the `queries_root` directory with an associated `permission_mappings.xml` placed in the `permissions_root` directory.

The `MessageKeys` and `web_root` directories are not used in these plugins and can be ignored.

```TEXT
|
+-- plugin.xml
|
+-- queries_root
    |
    *-- partner.module1.named_queries.xml
    *-- partner.module2.named_queries.xml
+-- permissions_root
    |
    *-- partner.module1.permission_mappings.xml
    *-- partner.module2.permission_mappings.xml
+-- MessageKeys
    |
    *-- example-plugin-message-keys.US_en.properties
+-- web_root
    |
    *-- admin
        |
        *-- home.partner.content.footer.txt
```

## `plugin.xml`

Defined in `./plugin.xml`. See [Plugin XML Article](https://support.powerschool.com/developer/#/page/plugin-xml). This file defines the name, version number, description and contact information for the plugin.

### `plugin`

#### `xmlns`: 
defines the namespace of the tags that are acceptible in this plugin

example: `xmlns="http://plugin.powerschool.pearson.com"`

#### `description`: 
Human readable description that will appear in plugin manager screen

example: `description="example plugin, does xyz -- tld.domain.app.area.fields <-- this helps users locate in the DEM manu"`

#### `name`: 
Name of plugin that appears in plugin menu (`https://<powerschool instance>/admin/pluginconsole/plugInConsole.action`). **NOTE** the name may only be 40 characters or less. Longer names will result in various errors when installing plugins.

example: `name="PowerQuery Example (Birthdays)"`

#### `xmlns:xsi`: 
scheme to validate this XML document agains

example: `xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"`

#### `version`: 
Version number of plugin. Must consist entirely of charset [0-9.].

example: `version="1.0.2.22`

#### `xsi:schemaLocation`:  
Schema document, plugin.xsd, against which to validate the XML document for the target namespace http://plugin.powerschool.pearson.com.

example: `xsi:schemaLocation="http://plugin.powerschool.pearson.com plugin.xsd"`

### publisher

Plublisher contact information displayed in plugin manager screen.

#### `name`: 
Publisher or Plublishing organization name

example: `name="Monty Python`

#### `contact email`: 
email contact for publisher

example: `<contact email=mpython@host.tld`/> 

### sample:
```XML
<plugin xmlns="http://plugin.powerschool.pearson.com"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        description="Powerquery Example"
        name="PowerQuery Example (Birthdays)"
        version="1.0"
        xsi:schemaLocation="http://plugin.powerschool.pearson.com plugin.xsd">
        
    <publisher name="Your/Org Name">
        <contact email="contact@host.tld"/>
    </publisher>
    
</plugin>
```

## `[name].named_queries.xml`

Defined in `./queries_root/[name].named_queries.xml`. See [PowerSchool Article](https://support.powerschool.com/developer/#/page/powerqueries). This file defines the query and resultant returned columns displayed in Data Export Manager.

### `query`

Basic format:

```XML
    <query name="[tld].[domain].[product name].[general data area].[name of data]" coreTable="table" flattened="false"> ... </query>
```

`name`: Used for identifying plugin in DEM and other menues

[tld].[domain].[product name].[general data area].[name of data]

example: `com.txoof.brightspace.students.userinfo`

`coreTable`: STUDENTS, CC, etc.

`flattened`: The flattened does not group fields by tables and displays all the fields in the same level.

### description

Human readable description of query

Basic format:

```XML
    <description>Description here</description>
```

### `columns`

This is the table and column name that the argument will be compared against. The `<column name=>` attribute must match a known table.field for all columns. The number of `<column>` tags must exactly match the number of columns returned by the SQL query. For calculated columns that do not match a known table.field, use any known column such as `STUDENTS.ID`. The column order must match the order of the columns returned by the SQL query.

Basic format:

```XML
    <columns>
        <column name="TABLE.Field">dem_name</column>
    </columns>
```

`column`: one column tag for each column returned by SQL query below

`name`: known TABLE.FIELD

`dem_name`: Data Export Manager column header. This will be displayed in the DEM configuration screen.

### sql

SQL query to run against the database. Always include an `ORDER BY`. Calculated fields are acceptable, but must be aliased. Each column returned by the query must match a `<column>` tag above.

Basic format:

```XML
        <![CDATA[
            SELECT
                student_number,
                lastfirst,
                grade_level
                lastfirst||grade_level as name_grade
            FROM
                students
            WHERE
                enroll_status = 0
            ORDER BY lastfirst
        ]]>
    </sql>
```

### Sample

```XML
<queries>
	<!--set name here (also applies to permissions_root-->
    <query name="com.txoof.brightspace.students.user" coreTable="students" flattened="false">
		<!--add description here-->
        <description>active students</description>
		<!--number of columns here must match number sql returns-->
        <columns>
			<column column="STUDENTS.ID">bar</column> <!-- for columns not in database use a core filed-->
			<column column="STUDENTS.ID">id</column>
			<column column="STUDENTS.LAST_NAME">LAST_NAME</column>
 		</columns>
		<!--SQL query in format <![CDATA[QUERY]]>-->
        <sql>
			<![CDATA[
			select 
			    'foo' as "bar",
			    STUDENTS.ID as ID,
				STUDENTS.LAST_NAME as LAST_NAME 
			from STUDENTS STUDENTS 
			where STUDENTS.ENROLL_STATUS =0
			]]>
        </sql>
    </query>

</queries>
```

## `[name].permission_mappings.xml`

Sets access permissions for this query plugin. See this [blog post](https://cookbrianj.wordpress.com/2017/01/23/plugin-export-with-ps-dem/).

```XML
<permission_mappings>
    <!--Anyone that has access to the following page can run this query-->
    <permission name='/admin/home.html'>
        <!--.../query/BASE_PLUGIN should be identical to `name` in named_queries.xml-->
        <implies allow="post">/ws/schema/query/BASE_PLUGIN</implies>
    </permission>
</permission_mappings>
```
