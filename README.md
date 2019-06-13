
# Mazur's SQL Style Guide

Howdy! I'm [Matt Mazur](https://mattmazur.com/) and I'm a professional data analyst who has worked at several startups to help them use data to grow their businesses. This guide is an attempt to document my preferences for formatting SQL in the hope that it may be of some use to others. If you or your team do not already have a SQL style guide, this may serve as a good starting point which you can adopt and update based on your preferences. 

Also, I'm a strong believer in having [Strong Opinions, Weakly Held](https://medium.com/@ameet/strong-opinions-weakly-held-a-framework-for-thinking-6530d417e364) so if you disagree with any of this, [drop me a note](https://mattmazur.com/contact/), I'd love to discuss it.

If you're interested in this topic, you may also enjoy my [Matt On Analytics](http://eepurl.com/dITJS9) newsletter and [my blog](https://mattmazur.com/category/analytics/) where I write about analytics and data analysis.

## Example

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql
with hubspot_interest as (

    select
        email,
        timestamp_millis(property_beacon_interest) as expressed_interest_at
    from hubspot.contact
    where property_beacon_interest is not null

), 

support_interest as (

    select 
        email,
        created_at as expressed_interest_at
    from helpscout.conversation
    join helpscout.conversation_tag on conversation_tag.conversation_id = conversation.id
    where tag = 'beacon-interest'

), 

combined_interest as (

    select * from hubspot_interest
    union all
    select * from support_interest

),

final as (

    select 
        email,
        min(expressed_interest_at) as expressed_interest_at
    from combined_interest
    group by email

)

select * from final
```

## Guidelines

### Use lowercase SQL

```sql
-- Good
select * from users

-- Bad
SELECT * FROM users

-- Bad
Select * From users
```

### Single line vs multiple line queries

The only time to put all of your SQL on one line is when you're selecting:

* All columns (*) or selecting 1 or 2 columns
* _And_ there's no additional complexity in your query

```sql
-- Good
select * from users

-- Good
select id from users

-- Good
select id, email from users

-- Good
select count(*) from users
```

The reason for this is simply that it's still easy to read this when everything is on one line. But once you start adding more columns or more complexity, it's easier to read if it's on multiple lines:

```sql
-- Good
select
    id,
    email,
    created_at
from users

-- Good
select *
from users
where email = 'example@domain.com'
```

For queries that have 1 or 2 columns, you can place the columns on the same line. For 3+ columns, place each column name on its own line, including the first item:

```sql
-- Good
select id, email
from users
where email like '%@gmail.com'

-- Good
select user_id, count(*) as total_charges
from charges
group by user_id

-- Good
select
    id,
    email,
    created_at
from users

-- Bad
select id, email, created_at
from users

-- Bad
select id,
    email
from users
```

### Left align everything

```sql
-- Good
select id, email
from users
where email like '%@gmail.com'

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
where email = 'example@domain.com'

-- Bad
select *
from users
where email = "example@domain.com"
```

### Use `!=` over `<>`

Simply because `!=` reads like "not equal" which is closer to how we'd say it out loud.

```sql
-- Good
select count(*) as paying_users_count
from users
where plan_name != 'free'
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

### Naming conventions

* Boolean fields should be prefixed with `is_`, `has_`, or `does_`. For example, `is_customer`, `has_unsubscribed`, etc.
* Date-only fields should be suffixed with `_date`. For example, `report_date`.
* Date+time fields should be suffixed with `_at`. For example, `created_at`, `posted_at`, etc.

### Always rename aggregates and function-wrapped arguments

```sql
-- Good
select count(*) as total_users
from users

-- Bad
select count(*)
from users

-- Good
select timestamp_millis(property_beacon_interest) as expressed_interest_at
from hubspot.contact
where property_beacon_interest is not null

-- Bad
select timestamp_millis(property_beacon_interest)
from hubspot.contact
where property_beacon_interest is not null
```

### Indenting where conditions

When there's only one where condition, leave it on the same line as `where`:

```sql
select email
from users
where id = 1234
```

When there are multiple, indent each one one level deeper than the `where`. Put logical operators at the end of the previous condition:

```sql
select id, email
from users
where 
    created_at >= '2019-03-01' and 
    vertical = 'work'
```

### Be explicit in boolean conditions

```sql
-- Good
select * from customers where is_cancelled = true
select * from customers where is_cancelled = false

-- Fine too
select * from customers where not is_cancelled

-- Bad
select * from customers where is_cancelled
```

### Use `as` to alias column names

```sql
-- Good
select
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
from users

-- Bad: Omitting `as`
select
    id,
    email,
    timestamp_trunc(created_at, month) signup_month
from users
```

### Group by column name, not number

```sql
-- Good
select user_id, count(*) as total_charges
from charges
group by user_id

-- Bad
select
    user_id,
    count(*) as total_charges
from charges
group by 1
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
where id in (1, 2)

-- Bad
select *
from users
where id in ( 1, 2 )
```

### Break long lists of `in` values into multiple indented lines

```sql
-- Good
select *
from users
where email in (
    'user-1@example.com',
    'user-2@example.com',
    'user-3@example.com',
    'user-4@example.com'
)
```

### Aligning case/when statements

Each `when` should be on its own line (nothing on the `case` line) and should be indented one level deeper than the `case` line. The `then` part should be on its own line, indented one level deeper than `when`.

```sql
-- Good
select
    case
        when event_name = 'viewed_homepage'
            then 'Homepage'
        when event_name = 'viewed_editor'
            then 'Editor'
    end as page_name
from events

-- Bad 
select
    case when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
    end as page_name
from events
```

### CTEs > subqueries

Avoid subqueries; CTEs will make your queries easier to read and reason about.

When using CTEs, pad the query with new lines. 

If you use any CTEs, always have a CTE named `final` and `select * from final` at the end. That way you can quickly inspect the output of other CTEs used in the query to debug the results.

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

final as (

    select user_id, name
    from ordered_details
    where details_rank = 1

)

select * from final
```

### Use meaningful CTE names

```sql
-- Good
with ordered_details as (

-- Bad
with d1 as (
```

### Omit `inner` for inner joins

```sql
-- Good
select
    email,
    sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id

-- Bad
select
    email,
    sum(amount) as total_revenue
from users
inner join charges on charges.user_id = users.id

```

### For join conditions, be consistent in which table comes first in the `on` condition

My preference when writing a join is to place the joined table's column _first_ in the `on` condition:

```sql
-- Good
select
    email,
    sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id
```

However, the counterargument is that the `users` table is listed first so it should come first in the `on` condition. And also, by doing it this way it makes it easier to determine if your join is going to cause the results to fan out:

```sql
-- Good too
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
```

Whichever style you adopt, just try to be consistent.

### Join conditions should be on the same line as the join

```sql
-- Good
select
    email,
    sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id

-- Bad
select
    email,
    sum(amount) as total_revenue
from users
join charges
on charges.user_id = users.id
```

For multiple join conditions, place the second, third, etc ones on their own line.

### Avoid aliasing tables

```sql
-- Good
select
    email,
    sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id

-- Bad
select
    email,
    sum(amount) as total_revenue
from users u
join charges c on c.user_id = u.id
```

The only exception is when you need to join onto a table more than once and need to distinguish them.

### Window functions

You can leave it all on its own line or break it up into multiple depending on its length:

```sql
-- Good
select
    user_id,
    name,
    row_number() over (partition by user_id order by date_updated desc) as details_rank
from billingdaddy.billing_stored_details

-- Good
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

Hat-tip to Peter Butler, Dan Wyman, Simon Ouderkirk, and Claire Herther for providing feedback on this guide.
