∙ [English](README.md) ∙ [简体中文](README-zh.md)

# Mazur's 的SQL风格指南

- [Mazur's 的SQL风格指南](#mazur-s--sql----)
    * [示例](##示例)
    * [指南](##指南)
        + [SQL语句采用小写](##SQL语句采用小写)
        + [单行查询与多行查询](#---------)
        + [SQL关键字左对齐](#sql------)
        + [使用单引号](#-----)
        + [使用 `!=` 而不是 `<>`](#----------------)
        + [逗号应该在行尾](#-------)
        + [缩进Where条件](#--where--)
        + [避免括号里面的空格](#---------)
        + [将in值的长列表分成多个缩进行](#-in------------)
        + [表名应该是名词的复数蛇形`例如：file_names、 line_numbers`形式](#----------------file-names--line-numbers---)
        + [列名应该是蛇形命名](#---------)
        + [列名的约定](#-----)
        + [列顺序约定](#-----)
        + [内联接中使用`inner`](#-------inner-)
        + [对于连接条件，将其直接放在首先引用的表`on`之后](#--------------------on---)
        + [单个连接条件应与连接在同一行上](#---------------)
        + [在大多数情况下避免表名的别名](#--------------)
        + [当存在连接时包含表，否则省略表](#---------------)
        + [聚合和函数包装的参数需要重新命名](#----------------)
        + [让布尔条件更加易于理解](#-----------)
        + [使用`as`来指定列的别名](#---as--------)
        + [使用列名或列号进行分组，但不能同时使用两种](#---------------------)
        + [在按名称分组时，利用列的别名](#--------------)
        + [分组的列放在最前](#--------)
        + [对齐 `case/when` 语句](#----case-when----)
        + [使用CTES而不是自查询](#--ctes------)
        + [使用有意义的CTE的名字](#------cte---)
        + [开窗函数](#----)
    * [鸣谢](#--)
    

你好!我是[Matt Mazur](https://mattmazur.com/) ，是一名数据分析师，曾在几家初创公司工作过，帮助他们利用数据发展业务。本指南试图记录我对格式化SQL的偏好，希望它对其他人可能有一些用处。如果您或您的团队还没有SQL风格指南，这可以作为一个很好的起点，您可以根据自己的偏好采用和更新它。

另外，我在[Strong Opinions, Weakly Held](https://medium.com/@ameet/strong-opinions-weakly-held-a-framework-for-thinking-6530d417e364) 是一个坚定的信徒，所以如果你不同意我的观点，请给我留言，我很乐意和你讨论。
Also, I'm a strong believer in having o if you disagree with any of this, [drop me a note](https://mattmazur.com/contact/), I'd love to discuss it.

如果你对这个主题感兴趣，你可能也会喜欢我LookML的风格指南[LookML Style Guide](https://github.com/mattm/lookml-style-guide) 和我关于分析和数据分析的博客 [blog](https://mattmazur.com/category/analytics/) 。

## 示例

下面是一个非常重要的查询，可以让您了解这个风格指南在实际中是什么样的：

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
        conversation.email,
        conversation.created_at as expressed_interest_at
    from helpscout.conversation
    inner join helpscout.conversation_tag on conversation.id = conversation_tag.conversation_id
    where conversation_tag.tag = 'beacon-interest'

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
## 指南

### SQL语句采用小写

它就像大写SQL一样易读，而且你不必总是按住shift键。
It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```sql
-- Good
select * from users

-- Bad
SELECT * FROM users

-- Bad
Select * From users
```

### 单行查询与多行查询

只有当查询结果集很醒目且查询条件并不复杂的时候，才应该将你的SQL放在一行中。

```sql
-- Good
select * from users

-- Good
select id from users

-- Good
select count(*) from users
```

当你开始查询更多列或更复杂的内容时，如果查询分布在多行上，阅读起来会更容易。

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

-- Good
select
    user_id,
    count(*) as total_charges
from charges
group by user_id

-- Bad
select id, email, created_at
from users

-- Bad
select id,
    email
from users
```

### SQL关键字左对齐

一些IDE能够自动格式化SQL，以便SQL关键字之后的空格垂直对齐。手工做这个很麻烦(而且在我看来很难阅读)，所以我建议所有的关键字都左对齐

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

### 使用单引号

一些SQL方言，比如BigQuery，支持使用双引号，但是对于大多数方言，双引号最终会指向列名。出于这个原因，单引号更好。

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

### 使用 `!=` 而不是 `<>`

很简单的原因，因为!=读起来像“不相等”，更接近我们大声说出来的方式。

```sql
-- Good
select count(*) as paying_users_count
from users
where plan_name != 'free'
```

### 逗号应该在行尾

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

### 缩进Where条件

当只有一个where条件时，让它和where在同一行

```sql
select email
from users
where id = 1234
```

当有多个时，每一个都要比where缩进一层。将逻辑运算符放在前一个条件的末尾

```sql
select id, email
from users
where 
    created_at >= '2019-03-01' and 
    vertical = 'work'
```

### 避免括号里面的空格

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

### 将in值的长列表分成多个缩进行

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

### 表名应该是名词的复数蛇形`例如：file_names、 line_numbers`形式

```sql
-- Good
select * from users
select * from visit_logs

-- Bad
select * from user
select * from visitLog
```

### 列名应该是蛇形命名

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

### 列名的约定

* Boolean 字段应该加前缀 `is_`, `has_`, or `does_`. 示例, `is_customer`, `has_unsubscribed`, 等.
* Date-only 字段应该加后缀 `_date`. 示例, `report_date`.
* Date+time 字段应该加后缀 `_at`. 示例, `created_at`, `posted_at`, 等.

### 列顺序约定

将主键放在前面，然后是外键，然后是所有其他列。如果表中有任何系统列(创建于，更新于，删除，等等)，将它们放在最后。
Put the primary key first, 
followed by foreign keys, 

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

### 内联接中使用`inner`

最好是显式的，这样连接类型就非常清楚了

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

### 对于连接条件，将其直接放在首先引用的表`on`之后

通过这样做，可以更容易地确定您的连接是否会导致结果呈扇形分布

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

### 单个连接条件应与连接在同一行上

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

当您有多个连接条件时，请将每个条件放在它们自己的缩进行中：

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

### 在大多数情况下避免表名的别名

很容易将表名(如`users`)缩写为`u`，将`charges` 缩写为`c`，但是这样会降低SQL的可读性

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

大多数情况下，您需要键入完整的表名。

有两个例外:

如果您需要在同一个查询中多次连接到一个表，并且需要区分该表的每个版本，那么就需要别名。

另外，如果您使用的表名很长或有歧义，可以使用别名(但仍然使用有意义的名称)。

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

### 当存在连接时包含表，否则省略表

当没有涉及到连接时，就不会对列来自哪个表产生歧义，因此可以省略表名。

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

但是，当涉及到连接时，最好是显式的，这样就可以清楚地知道列起源于何处

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

### 聚合和函数包装的参数需要重新命名

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

### 让布尔条件更加易于理解

```sql
-- Good
select * from customers where is_cancelled = true
select * from customers where is_cancelled = false

-- Bad
select * from customers where is_cancelled
select * from customers where not is_cancelled
```

### 使用`as`来指定列的别名

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

### 使用列名或列号进行分组，但不能同时使用两种

我更喜欢按名称分组，但按数字分组也不错[also fine](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/).

```sql
-- Good
select user_id, count(*) as total_charges
from charges
group by user_id

-- Good
select user_id, count(*) as total_charges
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

### 在按名称分组时，利用列的别名

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

### 分组的列放在最前

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

### 对齐 `case/when` 语句

每个`when`都应该在自己的行上( `case`行上没有任何内容)，并且应该缩进比 `case`行深一层。`then`可以在同一条线上，也可以在它的下方，目的是保持一致。

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

### 使用CTES而不是自查询

避免子查询; CTEs将使您的查询更容易阅读和推理。

使用CTEs时，用新行填充查询。

如果使用任何CTE，请始终使用名为`final`的CTE，并在 `select * from final` 结尾。通过这种方式， 您可以快速检查查询中使用的其他cte的输出，以调试结果。

结束的CTE括号应该使用与`with`和CTE名称相同的缩进级别 。

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

-- Bad
select user_id, name
from (
    select
        user_id,
        name,
        row_number() over (partition by user_id order by date_updated desc) as details_rank
    from billingdaddy.billing_stored_details
) ranked
where details_rank = 1
```

### 使用有意义的CTE的名字

```sql
-- Good
with ordered_details as (

-- Bad
with d1 as (
```

### 开窗函数

你可以把它单独放在一行上，或者根据它的长度把它分成多个排列

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

## 鸣谢

这个风格指南的灵感部分来自于：

* [Fishtown Analytics' dbt Style Guide](https://github.com/fishtown-analytics/corp/blob/master/dbt_coding_conventions.md#sql-style-guide)
* [KickStarter's SQL Style Guide](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
* [GitLab's SQL Style Guide](https://about.gitlab.com/handbook/business-ops/data-team/sql-style-guide/)

感谢 Peter Butler, Dan Wyman, Simon Ouderkirk, Alex Cano, Adam Stone, Brian Kim, and Claire Carroll 提供对本指南的反馈。.
