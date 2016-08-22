---
layout: post
title:  "Making PostgreSQL Work"
date:   2016-08-22 16:37:48 -0500
categories: Drupal postgresql
---
## Making PostgreSQL Work
#### 0.04

### Creating a materialized view in PostgreSQL: ######

    CREATE MATERIALIZED VIEW public.<NAME OF VIEW> AS 

    SELECT <FIELDS YOU WANT> as <ALIAS FOR FIELDS>

    FROM <FIRST TABLE> <FIRST TABLE ALIAS> 

    JOIN <SECOND TABLE> <SECOND TABLE ALIAS> ON <FIRST TABLE ALIAS>.<FIRST TABLE FOREIGN KEY> = <SECOND TABLE ALIAS>.<FOREIGN KEY FIELD>

    ORDER BY <TABLE>.<FIELD NAME> <ASC|DESC>

    WITH DATA;

ALTER TABLE public.drupal_sbx_custom_admin_customer_org_v
  OWNER TO drupal;

### Creating triggers for that (specific) view: ######

    create or replace function refresh_admin_cust_org_view()

    returns trigger language plpgsql

    as $$

    begin

    refresh materialized view drupal_sbx_custom_admin_customer_org_v;

    return null;

    end $$;

***

    create trigger refresh_mat_view

    after insert or update or delete or truncate

    on drupal_users for each statement

    execute procedure refresh_admin_cust_org_view();

***

    create trigger refresh_mat_view

    after insert or update or delete or truncate

    on drupal_user__field_user_accounts for each statement

    execute procedure refresh_admin_cust_org_view();

***

    create trigger refresh_mat_view

    after insert or update or delete or truncate

    on drupal_user__field_user_organization for each statement

    execute procedure refresh_admin_cust_org_view();

### List all tables in the database: ######


