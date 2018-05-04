---
layout: post
title:  "Why @@Identity May Not Return the Actual New Row ID Value in SQL Server"
date:   2011-01-14 09:54:00 -0500
categories: SQL
---

While I was reading the excellent topic "[Database Development Mistakes Made by Application Developers](http://stackoverflow.com/questions/621884/database-development-mistakes-made-by-application-developers)" over on Stack Overflow, I followed a link to "[What are the most common SQL anti-patterns?](http://stackoverflow.com/questions/346659/what-are-the-most-common-sql-anti-patterns)" and came across this tip that made me wonder if I've been handling the creation of new identity values all wrong:

- @@IDENTITY returns the last identity value generated for any table in the current session, across all scopes. You need to be careful here, since it's across scopes. You could get a value from a trigger, instead of your current statement.

- SCOPE_IDENTITY returns the last identity value generated for any table in the current session and the current scope. Generally what you want to use.

[This article](http://blog.sqlauthority.com/2007/03/25/sql-server-identity-vs-scope_identity-vs-ident_current-retrieve-last-inserted-identity-of-record/) from a SQL Server MVP pretty much repeats that information.

Now, I don't use triggers in my application building. I prefer to have the business logic in the application layer, not the database layer. I've not run across any identity scope issues before, but that doesn't necessarily mean I'm safe, or that one of my team members won't use triggers in their application development.

If you do an insert via &lt;cfquery&gt; in ColdFusion 8 and 9, it returns the ID of the inserted row (or rows) in the result attribute of the &lt;cfquery&gt; tag. For example, if you're using SQL Server, result_name.IDENTITYCOL is the reference to the ID of the inserted row. If you're using MySQL, it's result_name.GENERATED_KEY.

Does anyone know if ColdFusion uses @@IDENTITY or SCOPE_IDENTITY()?