# BS_07_Users_Teachers_Delete

Powerschool &rarr; BrightSpace CSV Delete inactive Teachers and Staff Export for 07-Users

**PROVIDES FIELDS:**

- `org_defined_id` used in [08-Enrollments Students](../BS_08_Enrollments_Students/8-Enrollments_students_README.md) as `child_code`
- `org_defined_id` used in [08-Enrollments Teachers](../BS_08_Enrollments_Teachers/8-Enrollments_Teachers_README.md) as `child_code`

|Field |Format |example |
|:-|:-|:-|
|`org_defined_id`| `T_`_`TEACHERS.ID`_ | T_123456

**USES FIELDS:**

- none

## Data Export Manager

- **Category:** Show All
- **Export Form:**  com.txoof.brightspace.teachers.inactive

### Lables Used on Export

| Label |
|-|
|type|
|action|
|username|
|org_define_id|
|first_name|
|last_name|
|password|
|role_name|
|relationships|
|pref_frist_name |
|fref_last_name |

### Export Summary and Output Options

#### Export Format

- *Export File Name:* `7-Users_teachers_inactive-%d.csv`
- *Line Delimiter:* `CR-LF`
- *Field Delimiter:* `,`
- *Character Set:* UTF-8

#### Export Options

- *Include Column Headers:* `True`
- *Surround "field values" in Quotes:* TBD

## Query Setup for `named_query.xml`

### PowerQuery Output Columns

| header | table.field | value | NOTE |
|-|-|-|-|
|type| TEACHERS.ID | user | N1 |
|action| TEACHERS.ID | UPDATE | N1 |
|username| TEACHERS.EMAIL_ADDR |_foo@ash.nl_ |
|org_define_id| TEACHERS.ID | _T\_234_ |
|first_name| TEACHERS.FIRST_NAME | _John_ |
|last_name| TEACHERS.LAST_NAME |_Doe_ | 
|password| TEACHERS.ID | '' | N1 |
|role_name| TEACHER.ID | _Learner_ | N1 |
|relationships| TEACHERS.ID | TBD | N1 |
|pref_frist_name| TEACHERS.ID |TBD | N1 |
|pref_last_name| TEACHERS.ID |TBD | N1 |

#### Notes

**N1:** Field does not appear in database; use a known field such as `<column column=STUDENT.ID>header<\column>` to prevent an "unknown column error"

### Tables Used

| Table |  |
|-|-|
|STUDENTS| |
|U_STUDENTSUSERFIELDS| |

### SQL

Delete staff that are not "active" (STATUS != 1) 
```
select distinct
    'user' as "type",
    'UPDATE' as "action",
    REGEXP_REPLACE(TEACHERS.EMAIL_ADDR, '(^.*)(@.*)', '\1') as "username",
    /* prepend a 'T' to make sure there are no studentid/teacherid colisions */
    'T_'||TEACHERS.ID as "org_defined_id",
    TEACHERS.FIRST_NAME as "first_name",
    TEACHERS.LAST_NAME as "last_name",
    '' as "password",
    0 as "is_active",
    'Instructor' as "role_name",
    TEACHERS.EMAIL_ADDR as "email",
    '' as "relationships",
    '' as "pref_first_name",
    '' as "pref_last_name"
 from 
     TEACHERS TEACHERS,
     LOGINS LOGINS
 where TEACHERS.HOMESCHOOLID = TEACHERS.SCHOOLID
    AND LOGINS.USERID=TEACHERS.ID
    /* Ignore all users with no email address */
    AND LENGTH(TEACHERS.EMAIL_ADDR) > 0
    AND TEACHERS.STATUS != 1
    /* deactivate anyone that is status !=1 and has not logged in in 5 months (150 days) */
    AND trunc(sysdate) - trunc(LOGINS.LOGINDATE) > 150
    ORDER BY "org_defined_id" asc
```