
# Hologram's SQL Style Guide

Adapted from [Matt Mazur](https://mattmazur.com/)'s excellent [SQL Style Guide](https://github.com/mattm/sql-style-guide), with the minor modification of capitalizing SQL keywords for better clarity for novice SQL writers.

## Example

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql
WITH hubspot_interest AS (

    SELECT
        email,
        timestamp_millis(property_beacon_interest) AS expressed_interest_at
    FROM hubspot.contact
    WHERE property_beacon_interest IS NOT NULL

),

support_interest AS (

    SELECT
        conversation.email,
        conversation.created_at AS expressed_interest_at
    FROM helpscout.conversation
    INNER JOIN helpscout.conversation_tag ON conversation.id = conversation_tag.conversation_id
    WHERE conversation_tag.tag = 'beacon-interest'

),

combined_interest AS (

    SELECT * FROM hubspot_interest
    UNION ALL
    SELECT * FROM support_interest

),

final AS (

    SELECT
        email,
        min(expressed_interest_at) AS expressed_interest_at
    FROM combined_interest
    GROUP BY email

)

SELECT * FROM final
```
## Guidelines

### Use uppercase SQL Keywords

It makes it clearer to novice SQL users what's a keyword and what's an identifier. Optimize for readability over convenience while typing.

```sql
-- Good
SELECT * FROM users

-- Bad
select * from users

-- Bad
Select * From users
```

### Single line vs multiple line queries

The only time you should place all of your SQL on a single line is when you're selecting one thing and there's no additional complexity in the query:

```sql
-- Good
SELECT * FROM users

-- Good
SELECT id FROM users

-- Good
SELECT count(*) FROM users
```

Once you start adding more columns or more complexity, the query becomes easier to read if it's spread out on multiple lines:

```sql
-- Good
SELECT
    id,
    email,
    created_at
FROM users

-- Good
SELECT *
FROM users
WHERE email = 'example@domain.com'

-- Good
SELECT
    user_id,
    count(*) as total_charges
FROM charges
GROUP BY user_id

-- Bad
SELECT id, email, created_at
FROM users

-- Bad
SELECT id,
    email
FROM users
```

### Left align SQL keywords

Some IDEs have the ability to automatically format SQL so that the spaces after the SQL keywords are vertically aligned. This is cumbersome to do by hand (and in my opinion harder to read anyway) so I recommend just left aligning all of the keywords:

```sql
-- Good
SELECT id, email
FROM users
WHERE email LIKE '%@gmail.com'

-- Bad
SELECT id, email
  FROM users
 WHERE email LIKE '%@gmail.com'
```

### Use single quotes

Some SQL dialects like BigQuery support using double quotes, but for most dialects double quotes will wind up referring to column names. For that reason, single quotes are preferable:

```sql
-- Good
SELECT *
FROM users
WHERE email = 'example@domain.com'

-- Bad
SELECT *
FROM users
WHERE email = "example@domain.com"
```

### Use `!=` over `<>`

Simply because `!=` reads like "not equal" which is closer to how we'd say it out loud.

```sql
-- Good
SELECT count(*) AS paying_users_count
FROM users
WHERE plan_name != 'free'
```

### Commas should be at the the end of lines

```sql
-- Good
SELECT
    id,
    email
FROM users

-- Bad
SELECT
    id
    , email
FROM users
```

### Indenting where conditions

When there's only one where condition, leave it on the same line as `where`:

```sql
SELECT email
FROM users
WHERE id = 1234
```

When there are multiple, indent each one one level deeper than the `where`. Put logical operators at the end of the previous condition:

```sql
SELECT id, email
FROM users
WHERE
    created_at >= '2019-03-01' AND
    vertical = 'work'
```

### Avoid spaces inside of parenthesis

```sql
-- Good
SELECT *
FROM users
WHERE id IN (1, 2)

-- Bad
SELECT *
FROM users
WHERE id IN ( 1, 2 )
```

### Break long lists of `in` values into multiple indented lines

```sql
-- Good
SELECT *
FROM users
WHERE email IN (
    'user-1@example.com',
    'user-2@example.com',
    'user-3@example.com',
    'user-4@example.com'
)
```

### Table names should be a plural snake case of the noun

```sql
-- Good
SELECT * FROM users
SELECT * FROM visit_logs

-- Bad
SELECT * FROM user
SELECT * FROM visitLog
```

### Column names should be snake_case

```sql
-- Good
SELECT
    id,
    email,
    timestamp_trunc(created_at, month) AS signup_month
FROM users

-- Bad
SELECT
    id,
    email,
    timestamp_trunc(created_at, month) AS SignupMonth
FROM users
```

### Column name conventions

* Boolean fields should be prefixed with `is_`, `has_`, or `does_`. For example, `is_customer`, `has_unsubscribed`, etc.
* Date-only fields should be suffixed with `_date`. For example, `report_date`.
* Date+time fields should be suffixed with `_at`. For example, `created_at`, `posted_at`, etc.

### Column order conventions

Put the primary key first, followed by foreign keys, then by all other columns. If the table has any system columns (`created_at`, `updated_at`, `is_deleted`, etc.), put those last.

```sql
-- Good
SELECT
    id,
    name,
    created_at
FROM users

-- Bad
SELECT
    created_at,
    name,
    id,
FROM users
```

### Include `inner` for inner joins

Better to be explicit so that the join type is crystal clear:

```sql
-- Good
SELECT
    users.email,
    sum(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id

-- Bad
SELECT
    users.email,
    sum(charges.amount) AS total_revenue
FROM users
JOIN charges ON users.id = charges.user_id
```

### For join conditions, put the table that was referenced first immediately after the `on`

By doing it this way it makes it easier to determine if your join is going to cause the results to fan out:

```sql
-- Good
SELECT
    ...
FROM users
LEFT JOIN charges ON users.id = charges.user_id
-- primary_key = foreign_key --> one-to-many --> fanout

SELECT
    ...
FROM charges
LEFT JOIN users ON charges.user_id = users.id
-- foreign_key = primary_key --> many-to-one --> no fanout

-- Bad
SELECT
    ...
FROM users
LEFT JOIN charges ON charges.user_id = users.id
```

### Single join conditions should be on the same line as the join

```sql
-- Good
SELECT
    users.email,
    sum(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id
GROUP BY email

-- Bad
SELECT
    users.email,
    sum(charges.amount) AS total_revenue
FROM users
INNER JOIN charges
ON users.id = charges.user_id
GROUP BY email
```

When you have multiple join conditions, place each one on their own indented line:

```sql
-- Good
SELECT
    users.email,
    sum(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON
    users.id = charges.user_id AND
    refunded = false
GROUP BY email
```

### Avoid aliasing table names most of the time

It can be tempting to abbreviate table names like `users` to `u` and `charges` to `c`, but it winds up making the SQL less readable:

```sql
-- Good
SELECT
    users.email,
    sum(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id

-- Bad
SELECT
    u.email,
    sum(c.amount) AS total_revenue
FROM users u
INNER JOIN charges c ON u.id = c.user_id
```

Most of the time you'll want to type out the full table name.

There are two exceptions:

If you you need to join to a table more than once in the same query and need to distinguish each version of it, aliases are necessary.

Also, if you're working with long or ambiguous table names, it can be useful to alias them (but still use meaningful names):

```sql
-- Good: Meaningful table aliases
SELECT
  companies.com_name,
  beacons.created_at
FROM stg_mysql_helpscout__helpscout_companies companies
INNER JOIN stg_mysql_helpscout__helpscout_beacons_v2 beacons ON companies.com_id = beacons.com_id

-- OK: No table aliases
SELECT
  stg_mysql_helpscout__helpscout_companies.com_name,
  stg_mysql_helpscout__helpscout_beacons_v2.created_at
FROM stg_mysql_helpscout__helpscout_companies
INNER JOIN stg_mysql_helpscout__helpscout_beacons_v2 ON stg_mysql_helpscout__helpscout_companies.com_id = stg_mysql_helpscout__helpscout_beacons_v2.com_id

-- Bad: Unclear table aliases
SELECT
  c.com_name,
  b.created_at
FROM stg_mysql_helpscout__helpscout_companies c
INNER JOIN stg_mysql_helpscout__helpscout_beacons_v2 b ON c.com_id = b.com_id
```

### Include the table when there is a join, but omit it otherwise

When there are no join involved, there's no ambiguity around which table the columns came from so you can leave the table name out:

```sql
-- Good
SELECT
    id,
    name
FROM companies

-- Bad
SELECT
    companies.id,
    companies.name
FROM companies
```

But when there are joins involved, it's better to be explicit so it's clear where the columns originated:

```sql
-- Good
SELECT
    users.email,
    sum(charges.amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id

-- Bad
SELECT
    email,
    sum(amount) AS total_revenue
FROM users
INNER JOIN charges ON users.id = charges.user_id

```

### Always rename aggregates and function-wrapped arguments

```sql
-- Good
SELECT count(*) AS total_users
FROM users

-- Bad
SELECT count(*)
FROM users

-- Good
SELECT timestamp_millis(property_beacon_interest) AS expressed_interest_at
FROM hubspot.contact
WHERE property_beacon_interest IS NOT NULL

-- Bad
SELECT timestamp_millis(property_beacon_interest)
FROM hubspot.contact
WHERE property_beacon_interest IS NOT NULL
```

### Be explicit in boolean conditions

```sql
-- Good
SELECT * FROM customers WHERE is_cancelled = true
SELECT * FROM customers WHERE is_cancelled = false

-- Bad
SELECT * FROM customers WHERE is_cancelled
SELECT * FROM customers WHERE NOt is_cancelled
```

### Use `as` to alias column names

```sql
-- Good
SELECT
    id,
    email,
    timestamp_trunc(created_at, month) AS signup_month
FROM users

-- Bad
SELECT
    id,
    email,
    timestamp_trunc(created_at, month) signup_month
FROM users
```

### Group using column names or numbers, but not both

I prefer grouping by name, but grouping by numbers is [also fine](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/).

```sql
-- Good
SELECT user_id, count(*) AS total_charges
FROM charges
GROUP by user_id

-- Good
SELECT user_id, count(*) AS total_charges
FROM charges
GROUP BY 1

-- Bad
SELECT
    timestamp_trunc(created_at, month) AS signup_month,
    vertical,
    count(*) AS users_count
FROM users
GROUP BY 1, vertical
```

### Take advantage of lateral column aliasing when grouping by name

```sql
-- Good
SELECT
  timestamp_trunc(com_created_at, year) AS signup_year,
  count(*) AS total_companies
FROM companies
GROUP BY signup_year

-- Bad
SELECT
  timestamp_trunc(com_created_at, year) AS signup_year,
  count(*) AS total_companies
FROM companies
GROUP BY timestamp_trunc(com_created_at, year)
```

### Grouping columns should go first

```sql
-- Good
SELECT
  timestamp_trunc(com_created_at, year) AS signup_year,
  count(*) AS total_companies
FROM companies
GROUP BY signup_year

-- Bad
SELECT
  count(*) AS total_companies,
  timestamp_trunc(com_created_at, year) AS signup_year
FROM mysql_helpscout.helpscout_companies
GROUP BY signup_year
```

### Aligning case/when statements

Each `when` should be on its own line (nothing on the `case` line) and should be indented one level deeper than the `case` line. The `then` can be on the same line or on its own line below it, just aim to be consistent.

```sql
-- Good
SELECT
    CASE
        WHEN event_name = 'viewed_homepage' THEN 'Homepage'
        WHEN event_name = 'viewed_editor' THEN 'Editor'
        ELSE 'Other'
    END AS page_name
FROM events

-- Good too
SELECT
    CASE
        WHEN event_name = 'viewed_homepage'
            THEN 'Homepage'
        WHEN event_name = 'viewed_editor'
            THEN 'Editor'
        ELSE 'Other'            
    END AS page_name
FROM events

-- Bad
SELECT
    CASE WHEN event_name = 'viewed_homepage' THEN 'Homepage'
        WHEN event_name = 'viewed_editor' THEN 'Editor'
        ELSE 'Other'        
    END AS page_name
FROM events
```

### Use CTEs, not subqueries

Avoid subqueries; CTEs will make your queries easier to read and reason about.

When using CTEs, pad the query with new lines.

If you use any CTEs, always have a CTE named `final` and `select * from final` at the end. That way you can quickly inspect the output of other CTEs used in the query to debug the results.

Closing CTE parentheses should use the same indentation level as `with` and the CTE names.

```sql
-- Good
WITH ordered_details AS (

    SELECT
        user_id,
        name,
        row_number() over (PARTITION BY user_id ORDER BY date_updated DESC) AS details_rank
    FROM billingdaddy.billing_stored_details

),

final AS (

    SELECT user_id, name
    FROM ordered_details
    WHERE details_rank = 1

)

SELECT * FROM final

-- Bad
SELECT user_id, name
FROM (
    SELECT
        user_id,
        name,
        row_number() over (PARTITION BY user_id ORDER BY date_updated DESC) AS details_rank
    FROM billingdaddy.billing_stored_details
) ranked
WHERE details_rank = 1
```

### Use meaningful CTE names

```sql
-- Good
WITH ordered_details AS (

-- Bad
WITH d1 AS (
```

### Window functions

You can leave it all on its own line or break it up into multiple depending on its length:

```sql
-- Good
SELECT
    user_id,
    name,
    row_number() over (PARTITION BY user_id ORDER BY date_updated DESC) AS details_rank
FROM billingdaddy.billing_stored_details

-- Good
SELECT
    user_id,
    name,
    row_number() over (
        PARTITION BY user_id
        ORDER BY date_updated DESC
    ) AS details_rank
FROM billingdaddy.billing_stored_details
```

## Credits

Adapted from:

* [Mazur's SQL Style Guide](https://github.com/mattm/sql-style-guide)

That style guide was inspired in part by:

* [Fishtown Analytics' dbt Style Guide](https://github.com/fishtown-analytics/corp/blob/master/dbt_coding_conventions.md#sql-style-guide)
* [KickStarter's SQL Style Guide](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
* [GitLab's SQL Style Guide](https://about.gitlab.com/handbook/business-ops/data-team/sql-style-guide/)

Hat-tip to Peter Butler, Dan Wyman, Simon Ouderkirk, Alex Cano, Adam Stone, Brian Kim, and Claire Carroll for providing feedback on this guide.
