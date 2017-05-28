---
layout: post
title:  "Updating bytea data in PostgreSQL"
date:   2016-11-28 11:48:48 -0500
categories: Drupal, PostgreSQL, Drupal 8, bytea
---
## Updating bytea data in PostgreSQL
#### 1.0

Drupal is a PHP CMS and as such is most often used in a LAMP stack. However, with many new offerings being presented and changes to existing products, many people are moving away from the standard LAMP stack.
As people who are familiar with Drupal know, it loves to put large swathes of configuration in database tables as [PHP serialized](http://php.net/manual/en/function.serialize.php) strings. In order to make sure that these strings aren't constrained by length, Drupal's designers use the BLOB type in MySQL to store this data.
While this proves to be fairly user-friendly in MySQL, it is not so friendly in other storage systems. Drupal 8 uses [PHP PDO](http://php.net/manual/en/book.pdo.php) as it's ORM, which makes a lot of sense because it's base to the language - giving it speed and availability advantages.
Where this creates a challenge is that somewhere between Drupal and PDO, the large storage field in PostgreSQL is [bytea](https://www.postgresql.org/docs/9.0/static/datatype-binary.html), a data type that stores information in binary.
For those devs that aren't familiar with bytea, this field can be awkward to not only update, but actually read from, as most PostgreSQL clients don't show binary data in a readable text format.


After much digging through documentation, I found the following to work for me. In order to read binary data, use the **CONVERT_FROM** function and tell it to change the data to UTF-8 format:

  SELECT name, CONVERT_FROM(data::bytea,'UTF8') FROM drupal_config WHERE name = '<field name>';

Now that you have the data that you are looking for, you can make the changes to it in any text editor. Then, in order to persist those changes in the database, the text needs to be converted back to hex - a fairly simple task which can be done with an online converter such as: http://string-functions.com/string-hex.aspx

At that point, it's a matter of a simple UPDATE statement, using the DECODE function, which tells PostgreSQL to take that hex code and convert it to binary, which can then be stored in the bytea column.

  UPDATE drupal_config SET data = DECODE('613a31363a7b733a343a2275756964223b733a33363a2234373837316131642d393065382d343335352d383066382d643430356333343230353031223b733a383a226c616e67636f6465223b733a323a22656e223b733a363a22737461747573223b623a313b733a31323a22646570656e64656e63696573223b613a313a7b733a363a226d6f64756c65223b613a323a7b693a303b733a343a2266696c65223b693a313b733a343a226e6f6465223b7d7d733a323a226964223b733a32343a226e6f64652e6669656c645f61747461636865645f66696c65223b733a31303a226669656c645f6e616d65223b733a31393a226669656c645f61747461636865645f66696c65223b733a31313a22656e746974795f74797065223b733a343a226e6f6465223b733a343a2274797065223b733a343a2266696c65223b733a383a2273657474696e6773223b613a343a7b733a31333a22646973706c61795f6669656c64223b623a303b733a31353a22646973706c61795f64656661756c74223b623a303b733a31303a227572695f736368656d65223b733a363a227075626c6963223b733a31313a227461726765745f74797065223b733a343a2266696c65223b7d733a363a226d6f64756c65223b733a343a2266696c65223b733a363a226c6f636b6564223b623a303b733a31313a2263617264696e616c697479223b693a313b733a31323a227472616e736c617461626c65223b623a313b733a373a22696e6465786573223b613a303a7b7d733a32323a22706572736973745f776974685f6e6f5f6669656c6473223b623a303b733a31343a22637573746f6d5f73746f72616765223b623a303b7d', 'hex') WHERE name = '<field name>';

I hope this helps prevent people from the headaches that I had been having.
