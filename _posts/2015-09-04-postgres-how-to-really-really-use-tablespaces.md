---
layout: post
title: "Postgres: How to Really, Really use Tablespaces"
date:       2015-09-04 14:00:44
description: ""
author: "T.J. Alumbaugh"
header-img: "img/cd-background-img.jpg"
---
<h2 class="section-heading">Postgres: How to Really, Really use Tablespaces</h2>

I like PostgresDB, but recently I had a very opaque problem. The documentation (eventually) saved the day, but not before filling up the disk holding `/var/lib/pgsql/`. Oops. Read on to learn from my terrible, terrible mistakes.

Sometimes, when using an RDMS, you end up with more data that you thought you were going to have. You've already committed to having your database on some disk, but that disk is not big enough to hold the next chunk of data you need to import. There are a number of options for how to handle this, but we opted to use <i>tablespaces</i> in Postgres, which allow you to keep the 'home' for your data in $PGDATA, but put all of the data for a new table in a new directory (presumably attached to some bigger disk) specified for your tablespace.

As long as this new directory is owned by user `postgres`, you should be good to go. So I happily attempted to create my new tablespace as follows:

<code>
CREATE TABLESPACE extraspace LOCATION '/big_disk/data';
</code>

Not so fast! Total and utter failure. No matter how many times you `chown` that directory to `postgres`, if it doesn't work the first time, it's not going to work. How? Why? Why does Linux hate you so much? Why doesn't this just work so we go back to reading Hacker News while the data imports? Because SELinux.

Security Enhanced Linux, if turned on for your system, will prevent Postgres from using the filesystem in that way unless you explicitly allow it. Assuming you want to create the tablespace in `/big_disk/data`, execute this command as root:

<code>
chcon system_u:object_r:postgresql_db_t:s0 /big_disk/data
</code>

Some additional background information on SELinux issues with Postgres is available <a href="http://www.mmtek.com/dp20090929/node/39">here</a>. Anyway, now you can create a tablespace with the command above. So, now it's time to make a table using your brand new tablespace! We had a smaller test table that had a sample of the full dataset, so we just need to create a new table just like the old one:

<code>
CREATE TABLE IF NOT EXISTS data_all (LIKE data_some INCLUDING CONSTRAINTS INCLUDING DEFAULTS) TABLESPACE extraspace;
</code>

OK, now we can `pgimport` to our heart's content right? NO. This is where I went wrong and filled up our production server. That was bad. The key is that, although creating the table this way will indeed start filling up the directory designated for your tablespace, the <i>index</i> (or indices if you have more than one) will still be added somewhere in $PGDATA. So your disk won't fill up as fast, but if you're importing a terabyte of data with a lot of rows, you'll still end up putting a lot of data on your main disk. The solution: at table creation time, you must ALSO specify that each index you create uses the new tablespace:

<code>
CREATE TABLE IF NOT EXISTS data_all (PRIMARY KEY (id) USING INDEX TABLESPACE extraspace, LIKE some_data INCLUDING CONSTRAINTS INCLUDING DEFAULTS) TABLESPACE extraspace;
</code>

<code>
CREATE INDEX an_index ON data_all USING btree (col1, col2) TABLESPACE extraspace;
</code> 

And with <i>that</i>, your data, your primary key index, and an additional index, will all be stored on your new disk. You'll only get a few kilobytes in PGDATA per 1 GB block file as Postgres ingests the data. Now, start your massive `pgimport` and go back to reading Hacker News.
