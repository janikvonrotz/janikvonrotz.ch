---
title: "PL SQL - Declare an array of objects and loop insert statements"
date: 2018-07-24T15:49:48+02:00
slug: plsql-declare-an-array-of-objects-and-loop-insert-statements
categories:
 - SQL-Server
tags:
 - oracle
 - plsql
 - type declaration
image: /images/oracle pl sql.jpg
---

While gathering informations about how an array with objects can be declared in PlSQL and the processed by aa for loop, I ended up with very different results. Some people created temporary tables and other defined complex new types. It was difficult to see through. By prioritizing simpler and more common approaches, I ended up with a suitable solution.

Before we have a look at the the PlSQL script, let me introduce you to the scenario I had to resolve.
<!--more-->

In my case there is a profile table (T_PROFILE) with an id (ID) and name (NAME) attribute. This table has an n:m relationship with a profile table (T_PROFILE) containing the same attributes. Roles are assigned to profiles in an authorization (T_AUTHORIZATION) table. A test data set for the authorization table had to be defined and its entries had to be inserted into the authorization table.

My approach to resolve this scenario was simple.

First I declared the required types, then populated the array structure, looped the array and finally inserted the authorization entries.

```sql
DECLARE

    index INT;
	role_id INT;
	profile_id INT;

    -- Create an object type
	TYPE t_authorization IS RECORD(
        role_name VARCHAR2(50),
        profile_name VARCHAR2(50)
	);
    -- Create an array type consisting of the authorization type
	TYPE t_authorizations IS TABLE OF t_authorization INDEX BY pls_integer;

    -- Declare the array
	authorizations t_authorizations;

BEGIN

	-- Add new authorization entries
	index := 1;
	authorizations(index).role_name := 'Administrator';
	authorizations(index).profile_name := 'Janik';

	-- Every entry is process by the loop statement
	FOR i IN authorizations.FIRST .. authorizations.LAST
	LOOP

        -- Look up the id for profile and role
		SELECT ID into role_id FROM T_ROLE WHERE NAME = authorizations(i).role_name;
		SELECT ID into profile_id FROM T_PROFILE WHERE NAME = authorizations(i).profile_name;

        -- Add a new authorization
		INSERT INTO T_AUTHORIZATION (id,ctl_cre_dat,ctl_mod_dat,profile_id,role_id)
			VALUES (S_AUTHORIZATION.NEXTVAL,SYSDATE,SYSDATE,profile_id,role_id);

	END LOOP;
END;
```

Actually super simple.