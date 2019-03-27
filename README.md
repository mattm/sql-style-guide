
# Mazur's SQL Formatting Guide

## Use two spaces to indent, not tabs

## Use Lowercase SQL
```
# Good
select * from users

# Bad
SELECT * FROM users

# Bad
Select * From users
```

## Use Multiple Lines
The only time to put all of your SQL on one line is when you're selecting:

* Selecting all columns (*) or selecting 1 or 2 columns
* There's no additional complexity in your query

```
# Good
select * from users

# Good
select id, email from users

# Good
select count(*) from users

# Good
select id from users
```

For everything else, break it into multiple lines:

```
# Good
select
	id,
	email,
	created_at
from users

# Good
select *
from users
where email = "example@domain.com"
```

For results with 1 or 2 columns, place them on the same line. For 3+, place each item on its own line, including the first item:

```
# Good
select id, email
from users
where email like "%@gmail.com"

# Good
select user_id, count(*) as total_charges
from charges
group by user_id

# Good
select
	id,
	email,
	created_at
from users

# Bad
select id, email, created_at
from users

# Bad
select id,
	email
from users
```

##  Left align everything

```
# Good
select id, email
from users
where email like "%@gmail.com"

# Bad
select id, email
  from users
 where email like "%@gmail.com"
```

## Use Double Quotes
Avoid single quotes unless your SQL dialect requires single quotes:

```
## Good
select *
from users
where email = "example@domain.com"

# Bad
select *
from users
where email = 'example@domain.com'
```

## Column names should be snake_case
```
# Good
select
	id,
	email,
	timestamp_trunc(created_at, month) as signup_month
from users

# Bad: No alias
select
	id,
	email,
	timestamp_trunc(created_at, month)
from users

# Bad: Other casing
select
	id,
	email,
	timestamp_trunc(created_at, month) as SignupMonth
from users
```

## Use `as` to alias column names

```
# Good
select
	id,
	email,
	timestamp_trunc(created_at, month) as signup_month
from users

# Bad: Omitting `as`
select
	id,
	email,
	timestamp_trunc(created_at, month) signup_month
from users
```

## Group by column name
```
# Good
select user_id, count(*) as total_charges
from charges
group by user_id

# Bad
select
	user_id,
	count(*) as total_charges
from charges
group by 1
```

## Commas should be at the the end of lines

```
# Good
select
	id,
	email
from users

# Bad
select
	id
	, email
from users
```

## Avoid spaces inside of parenthesis
```
# Good
select *
from users
where id in (1, 2)

# Bad
select *
from users
where id in ( 1, 2 )
```

## Aligning case/when statements
Each `when`  should be on its own line (nothing on the `case` line) and should be indented one level deeper than the `case` line. Then `then` part should be on its own line, indented one level deeper.

```
# Good
select
	case
		when event_name = "viewed_homepage"
			then "Homepage"
		when event_name = "viewed_editor"
			then "Editor"
	end as page_name
from events

# Bad 
select
	case when event_name = "viewed_homepage" then "Homepage"
		when event_name = "viewed_editor" then "Editor"
	end as page_name
from events
```

## Use CTEs Instead of Subqueries
In short, never use subqueries. CTEs will make your queries easier to read and reason about.

When using CTEs, pad the inner query with new lines. 

If you use any CTEs, then always have a CTE named `final` and `select * from final` at the end. That way you can quickly inspect the output of other CTEs used in the query to debug the results.

Closing CTE parentheses should use the same indentation level as `with` and the CTE names.

```
# Good
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

## Use Meaningful CTE names

```
# Good
with ordered_details as (

# Bad
with d1 as (
```

## Indenting where conditions
When there's only one where condition, leave it on the same line as `where`:

```
select email
from users
where id = 1234
```

When there are multiple, indent each one one level deeper than the `where`. Put logical operators at the end of the previous condition:

```
select id, email
from users
where 
	created_at >= "2019-03-01" and 
	vertical = "work"
```

## Omit `inner` from joins

```
# Good
select
	email,
	sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id

# Bad
select
	email,
	sum(amount) as total_revenue
from users
inner join charges on charges.user_id = users.id
```

## For join conditions, put the joined table column first

```
# Good
select
	email,
	sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id

# Bad
select
	email,
	sum(amount) as total_revenue
from users
join charges on users.id = charges.user_id
```

## Join conditions should be on the same line as the join
```
# Good
select
	email,
	sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id

# Bad
# Good
select
	email,
	sum(amount) as total_revenue
from users
join charges
on charges.user_id = users.id
```


## Avoid aliasing tables

```
# Good
select
	email,
	sum(amount) as total_revenue
from users
join charges on charges.user_id = users.id

# Bad
select
	email,
	sum(amount) as total_revenue
from users u
join charges c on c.user_id = u.id
```

The only exception is when you need to join onto a table more than once and need to distinguish them.

## Partitions
You can leave it all on its own line or break it up into multiple depending on its length:

```
# Good
select
	user_id,
	name,
	row_number() over (partition by user_id order by date_updated desc) as details_rank
from billingdaddy.billing_stored_details

# Good
select
	user_id,
	name,
	row_number() over (
		partition by user_id
		order by date_updated desc
	) as details_rank
from billingdaddy.billing_stored_details
```



