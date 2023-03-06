# SQL CRUD Assignment

## Restaurant Reviews

### 1) Table Design

Firstly, I created the tables in SQLite to store the data for ```restaurants``` and ```reviews```. I did not set the ```id``` field as the primary key for this question since it was giving me issues with importing the Mockaroo data, but for the Social Media App exercise I corrected this issue.
```
create table restaurants (
id integer,
name text,
neighborhood text,
cuisine_type text,
price_tier text,
opening_time time,
closing_time time
);

create table reviews (
id integer,
restaurant_id integer,
rating integer,
kid_friendly text
);
```

I then created the mock data on Mockaroo. Here are the links for the [restaurants](/data/restaurants.csv) and [reviews](/data/reviews.csv) text files, respectively. I imported the mock data into the SQLite tables using the code below:
```
.mode csv

.import /Users/odin.loehr/Desktop/db_design/sql-crud-ohding11/data/restaurants.csv restaurants

.import /Users/odin.loehr/Desktop/db_design/sql-crud-ohding11/data/reviews.csv reviews
```

Before proceeding to writing the queries, I decided to add a column to the 'restaurants' table that showed the average rating for each restaurant:
```
alter table restaurants add column avg_rating real;

update restaurants set avg_rating = (
select round(avg(rating),2) from reviews
where reviews.restaurant_id = restaurants.id
);
```

### 2) Queries

a. Find all cheap restaurants in a particular neighborhood (example neighborhood: 'Flushing')
```
select * from restaurants where
price_tier = "Low" and neighborhood = "Flushing";
```

b. Find all restaurants in a particular genre (example: "Mexican") with 3 stars or more, ordered by the number of stars in descending order.
```
select * from restaurants where
cuisine_type = "Mexican" and avg_rating > 2.99
order by avg_rating desc;
```

c. Find all restaurants that are open now (see hint below).
```
select * from restaurants where
strftime('%H:%M','now','localtime') >= opening_time
and strftime('%H:%M','now','localtime') <= closing_time;
```

d. Leave a review for a restaurant (example: "The Happy Hotcake"). Note: Since I did not set the ```id``` field as the primary key, I have to insert a new value for ```id``` when I add a review.
```
insert into reviews (
id, restaurant_id, rating, kid_friendly)
values (
(select max(id)+1 from reviews),
(select id from restaurants where name = "The Happy Hotcake"),
4, "true");
```

e. Delete all restaurants that are not good for kids. Note: Since I decided to include ```kid_friendly``` as a field in ```reviews```, this query returns any restaurant that has been left a review that said it was not kid friendly.
```
delete from restaurants where exists (
select 1 from reviews where 
reviews.restaurant_id=restaurants.id
and reviews.kid_friendly = "false");
```

f. Find the number of restaurants in each NYC neighborhood.
```
select neighborhood, count(neighborhood) from restaurants
group by neighborhood;
```

## Social Media App

### 1) Table Design
Below is the code I used to create the ```users``` and ```posts``` tables:
```
create table users (
id integer primary key,
username text,
email text,
password text
);

create table posts (
id integer primary key,
post_type text,
content text,
poster_id integer,
recipient_id integer,
posted integer,
is_visible text
);
```
I then created the [users](/data/users.csv) and [posts](/data/posts.csv) text files using Mockaroo. For the 'posts.csv' file, I formatted the ```posted``` field to be the UNIX timestamp to make it easier to compare dates and times - these values were given in milliseconds by Mockaroo by default. I also made sure the ```recipient_id``` field was null if the ```post_type``` was 'story', as stories are not sent to a specific user. I imported the mock data using the code below:
```
.mode csv

.import /Users/odin.loehr/Desktop/db_design/sql-crud-ohding11/data/users.csv users

.import /Users/odin.loehr/Desktop/db_design/sql-crud-ohding11/data/posts.csv posts
```

### 2) Queries

a. Register a new User.
```
insert into users (username, email, password)
values ('ohding11','ol544@nyu.edu','asdfghjk');
```

b. Create a new Message sent by a particular User to a particular User (example: User 38 wants to know what User 99 is doing). Note, I had to concatenate '000' to the end of the value returned by the ```strftime``` function as it returns the time in seconds, while the times in my table are stored in milliseconds.
```
insert into posts (post_type, content, 
poster_id, recipient_id, posted, is_visible)
values ('message','wyd',38,99,
strftime('%s','now') || '000','true');
```

c. Create a new Story by a particular User (example: User 38 is facing legal trouble).
```
insert into posts (post_type, content, 
poster_id, recipient_id, posted, is_visible)
values ('story','i am wanted for murder',38,null,
strftime('%s','now') || '000','true');
```


d. Show the 10 most recent visible Messages and Stories, in order of recency.
```
select * from posts where is_visible = "true"
order by posted desc limit 10;
```

e. Show the 10 most recent visible Messages sent by a particular User to a particular User in order of recency (Example: User 150 to User 124).
```
select * from posts where is_visible = "true"
and post_type = 'message'
and poster_id = 150
and recipient_id = 124
order by posted desc limit 10;
```

f. Make all Stories that are more than 24 hours old invisible. Note: '86400000' = 24 hours in Unix time represented in milliseconds.
```
update posts set is_visible = "false"
where cast(strftime('%s','now') || '000' as integer) - posts.posted > 86400000;
```

g. Show all invisible Messages and Stories, in order of recency.
```
select * from posts where is_visible = "false"
order by posted desc;
```

h. Show the number of posts by each User.
```
select users.username, count(posts.poster_id) as posts_made
from users left join posts on users.id=posts.poster_id
group by users.id;
```
i. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```
select posts.content, users.email from
posts inner join users on posts.poster_id=users.id
where cast(strftime('%s','now') || '000' as integer) - posts.posted < 86400000;
```

j. Show the email addresses of all Users who have not posted anything yet.
```
select users.email, count(posts.poster_id) as posts_made
from users left join posts on users.id=posts.poster_id
group by users.id having posts_made = 0;
```