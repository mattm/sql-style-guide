
# Mazur's SQL Style Guide

Howdy! I'm [Matt Mazur](https://mattmazur.com/) and I'm a data analyst who has worked at several startups to help them use data to grow their businesses. This guide is an attempt to document my preferences for formatting SQL in the hope that it may be of some use to others. If you or your team do not already have a SQL style guide, this may serve as a good starting point which you can adopt and update based on your preferences. 

Also, I'm a strong believer in having [Strong Opinions, Weakly Held](https://medium.com/@ameet/strong-opinions-weakly-held-a-framework-for-thinking-6530d417e364) so if you disagree with any of this, [drop me a note](https://mattmazur.com/contact/), I'd love to discuss it.

If you're interested in this topic, you may also enjoy my [LookML Style Guide](https://github.com/mattm/lookml-style-guide) or my [blog](https://mattmazur.com/category/analytics/) where I write about analytics and data analysis.

Simplified Chinese version here: [中文版](https://github.com/huangxinping/sql-style-guide/blob/zh-cn/README.md)

## Example

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql
with hubspot_interest as (

    select
        email,
        timestamp_millis(property_beacon_interest) as expressed_interest_at
    from hubspot.contact
    where 
        property_beacon_interest is not null

), 

support_interest as (

    select 
        conversation.email,
        conversation.created_at as expressed_interest_at
    from helpscout.conversation
    inner join helpscout.conversation_tag on conversation.id = conversation_tag.conversation_id
    where 
        conversation_tag.tag = 'beacon-interest'

), 

combined_interest as (

    select * from hubspot_interest
    union all
    select * from support_interest

),

first_interest as (

    select 
        email,
        min(expressed_interest_at) as expressed_interest_at
    from combined_interest
    group by email

)

select * from first_interest
```
## Guidelines

### Use lowercase SQL

It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```sql
-- Good
select * from users

-- Bad
SELECT * FROM users

-- Bad
Select * From users
```

### Put each selected column on its own line

When selecting columns, always put each column name on its own line after the `select` keyword. This makes it easier to iterate on the query as you add more columns.

```sql
-- Good
select 
    id
from users 

-- Good
select 
    id,
    email
from users 

-- Bad
select id
from users 

-- Bad
select id, email
from users 
```

## Selecting *

When selecting `*` it's fine to include the `*` next to the `select` and also fine to include the `from` on the same line, assuming no additional complexity like where conditions:

```sql
-- Good
select * from users 

-- Good too
select *
from users

-- Bad
select * from users where email = 'name@example.com'
```

## Indenting where conditions

Similarly, conditions should always be spread across multiple lines to maximize readability and make them easier to add to. Operators should be placed at the end of each line:

```sql
-- Good
select *
from users
where 
    email = 'example@domain.com'

-- Good
select *
from users
where 
    email like '%@domain.com' and 
    created_at >= '2021-10-08'

-- Bad
select *
from users
where email = 'example@domain.com'

-- Bad
select *
from users
where 
    email like '%@domain.com' and created_at >= '2021-10-08'

-- Bad
select *
from users
where 
    email like '%@domain.com' 
    and created_at >= '2021-10-08'
```

### Left align SQL keywords

Some IDEs have the ability to automatically format SQL so that the spaces after the SQL keywords are vertically aligned. This is cumbersome to do by hand (and in my opinion harder to read anyway) so I recommend just left aligning all of the keywords:

```sql
-- Good
select 
    id,
    email
from users
where 
    email like '%@gmail.com'

-- Bad
select id, email
  from users
 where email like '%@gmail.com'
```

### Use single quotes

Some SQL dialects like BigQuery support using double quotes, but for most dialects double quotes will wind up referring to column names. For that reason, single quotes are preferable:

```sql
-- Good
select *
from users
where 
    email = 'example@domain.com'

-- Bad
select *
from users
where 
    email = "example@domain.com"
```

### Use `!=` over `<>`

Simply because `!=` reads like "not equal" which is closer to how we'd say it out loud.

```sql
-- Good
select 
    count(*) as paying_users_count
from users
where 
    plan_name != 'free'
```

### Commas should be at the the end of lines

```sql
-- Good
select
    id,
    email
from users

-- Bad
select
    id
    , email
from users
```

### Avoid spaces inside of parenthesis

```sql
-- Good
select *
from users
where 
    id in (1, 2)

-- Bad
select *
from users
where 
    id in ( 1, 2 )
```

### Break long lists of `in` values into multiple indented lines

```sql
-- Good
select *
from users
where 
    email in (
        'user-1@example.com',
        'user-2@example.com',
        'user-3@example.com',
        'user-4@example.com'
    )
```

### Table names should be a plural snake case of the noun

```sql
-- Good
select * 
from users

-- Good
select * 
from visit_logs

-- Bad
select * 
from user

-- Bad
select * 
from visitLog
```

### Column names should be snake_case

```sql
-- Good
select
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
from users

-- Bad
select
    id,
    email,
    timestamp_trunc(created_at, month) as SignupMonth
from users
```

### Column name conventions

* Boolean fields should be prefixed with `is_`, `has_`, or `does_`. For example, `is_customer`, `has_unsubscribed`, etc.
* Date-only fields should be suffixed with `_date`. For example, `report_date`.
* Date+time fields should be suffixed with `_at`. For example, `created_at`, `posted_at`, etc.

### Column order conventions

Put the primary key first, followed by foreign keys, then by all other columns. If the table has any system columns (`created_at`, `updated_at`, `is_deleted`, etc.), put those last.

```sql
-- Good
select
    id,
    name,
    created_at
from users

-- Bad
select
    created_at,
    name,
    id,
from users
```

### Include `inner` for inner joins

Better to be explicit so that the join type is crystal clear:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- Bad
select
    users.email,
    sum(charges.amount) as total_revenue
from users
join charges on users.id = charges.user_id
```

### For join conditions, put the table that was referenced first immediately after the `on`

By doing it this way it makes it easier to determine if your join is going to cause the results to fan out:

```sql
-- Good
select
    ...
from users
left join charges on users.id = charges.user_id
-- primary_key = foreign_key --> one-to-many --> fanout
  
select
    ...
from charges
left join users on charges.user_id = users.id
-- foreign_key = primary_key --> many-to-one --> no fanout

-- Bad
select
    ...
from users
left join charges on charges.user_id = users.id
```

### Single join conditions should be on the same line as the join

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id
group by email

-- Bad
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges
on users.id = charges.user_id
group by email
```

When you have mutliple join conditions, place each one on their own indented line:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on 
    users.id = charges.user_id and
    refunded = false
group by email
```

### Avoid aliasing table names most of the time

It can be tempting to abbreviate table names like `users` to `u` and `charges` to `c`, but it winds up making the SQL less readable:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- Bad
select
    u.email,
    sum(c.amount) as total_revenue
from users u
inner join charges c on u.id = c.user_id
```

Most of the time you'll want to type out the full table name.

There are two exceptions:

If you you need to join to a table more than once in the same query and need to distinguish each version of it, aliases are necessary.

Also, if you're working with long or ambiguous table names, it can be useful to alias them (but still use meaningful names):

```sql
-- Good: Meaningful table aliases
select
  companies.com_name,
  beacons.created_at
from stg_mysql_helpscout__helpscout_companies companies
inner join stg_mysql_helpscout__helpscout_beacons_v2 beacons on companies.com_id = beacons.com_id

-- OK: No table aliases
select
  stg_mysql_helpscout__helpscout_companies.com_name,
  stg_mysql_helpscout__helpscout_beacons_v2.created_at
from stg_mysql_helpscout__helpscout_companies
inner join stg_mysql_helpscout__helpscout_beacons_v2 on stg_mysql_helpscout__helpscout_companies.com_id = stg_mysql_helpscout__helpscout_beacons_v2.com_id

-- Bad: Unclear table aliases
select
  c.com_name,
  b.created_at
from stg_mysql_helpscout__helpscout_companies c
inner join stg_mysql_helpscout__helpscout_beacons_v2 b on c.com_id = b.com_id
```

### Include the table when there is a join, but omit it otherwise

When there are no join involved, there's no ambiguity around which table the columns came from so you can leave the table name out:

```sql
-- Good
select
    id,
    name
from companies

-- Bad
select
    companies.id,
    companies.name
from companies
```

But when there are joins involved, it's better to be explicit so it's clear where the columns originated:

```sql
-- Good
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- Bad
select
    email,
    sum(amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

```

### Always rename aggregates and function-wrapped arguments

```sql
-- Good
select 
    count(*) as total_users
from users

-- Bad
select 
    count(*)
from users

-- Good
select 
    timestamp_millis(property_beacon_interest) as expressed_interest_at
from hubspot.contact
where 
    property_beacon_interest is not null

-- Bad
select
    timestamp_millis(property_beacon_interest)
from hubspot.contact
where
    property_beacon_interest is not null
```

### Be explicit in boolean conditions

```sql
-- Good
select * 
from customers 
where 
    is_cancelled = true

-- Good
select * 
from customers 
where 
    is_cancelled = false

-- Bad
select * 
from customers 
where 
    is_cancelled

-- Bad
select * 
from customers 
where 
    not is_cancelled
```

### Use `as` to alias column names

```sql
-- Good
select
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
from users

-- Bad
select
    id,
    email,
    timestamp_trunc(created_at, month) signup_month
from users
```

### Group using column names or numbers, but not both

I prefer grouping by name, but grouping by numbers is [also fine](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/).

```sql
-- Good
select 
    user_id, 
    count(*) as total_charges
from charges
group by user_id

-- Good
select 
    user_id, 
    count(*) as total_charges
from charges
group by 1

-- Bad
select
    timestamp_trunc(created_at, month) as signup_month,
    vertical,
    count(*) as users_count
from users
group by 1, vertical
```

### Take advantage of lateral column aliasing when grouping by name

```sql
-- Good
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by signup_year

-- Bad
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by timestamp_trunc(com_created_at, year)
```

### Grouping columns should go first

```sql
-- Good
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by signup_year

-- Bad
select
  count(*) as total_companies,
  timestamp_trunc(com_created_at, year) as signup_year
from mysql_helpscout.helpscout_companies
group by signup_year
```

### Aligning case/when statements

Each `when` should be on its own line (nothing on the `case` line) and should be indented one level deeper than the `case` line. The `then` can be on the same line or on its own line below it, just aim to be consistent.

```sql
-- Good
select
    case
        when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
        else 'Other'
    end as page_name
from events

-- Good too
select
    case
        when event_name = 'viewed_homepage'
            then 'Homepage'
        when event_name = 'viewed_editor'
            then 'Editor'
        else 'Other'            
    end as page_name
from events

-- Bad 
select
    case when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
        else 'Other'        
    end as page_name
from events
```

### Use CTEs, not subqueries

Avoid subqueries; CTEs will make your queries easier to read and reason about.

When using CTEs, pad the query with new lines. 

If you use any CTEs, always `select *` from the last CTE at the end. That way you can quickly inspect the output of other CTEs used in the query to debug the results.

Closing CTE parentheses should use the same indentation level as `with` and the CTE names.

```sql
-- Good
with ordered_details as (

    select
        user_id,
        name,
        row_number() over (partition by user_id order by date_updated desc) as details_rank
    from billingdaddy.billing_stored_details

),

first_updates as (

    select 
        user_id, 
        name
    from ordered_details
    where 
        details_rank = 1

)

select * from first_updates

-- Bad
select 
    user_id, 
    name
from (
    select
        user_id,
        name,
        row_number() over (partition by user_id order by date_updated desc) as details_rank
    from billingdaddy.billing_stored_details
) ranked
where 
    details_rank = 1
```

### Use meaningful CTE names

```sql
-- Good
with ordered_details as (

-- Bad
with d1 as (
```

### Window functions

Leave it all on its own line:

```sql
-- Good
select
    user_id,
    name,
    row_number() over (partition by user_id order by date_updated desc) as details_rank
from billingdaddy.billing_stored_details

-- Okay
select
    user_id,
    name,
    row_number() over (
        partition by user_id
        order by date_updated desc
    ) as details_rank
from billingdaddy.billing_stored_details
```

## Credits

This style guide was inspired in part by:

* [Fishtown Analytics' dbt Style Guide](https://github.com/fishtown-analytics/corp/blob/master/dbt_coding_conventions.md#sql-style-guide)
* [KickStarter's SQL Style Guide](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
* [GitLab's SQL Style Guide](https://about.gitlab.com/handbook/business-ops/data-team/sql-style-guide/)

Hat-tip to Peter Butler, Dan Wyman, Simon Ouderkirk, Alex Cano, Adam Stone, Brian Kim, and Claire Carroll for providing feedback on this guide.
