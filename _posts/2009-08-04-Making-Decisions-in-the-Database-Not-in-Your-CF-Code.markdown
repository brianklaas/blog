---
layout: post
title:  "Making Decisions in the Database, Not in Your CF Code"
date:   2009-10-15 15:28:00 -0400
categories: SQL
---

Like many people who develop Web applications, I don't have a formal background in computer science. I've taught myself most of what I know, and I've been very fortunate to be the recipient of a lot of great knowledge from people much, much smarter than I. One of the areas in which my skills can be improved is with utilizing the full power of SQL. I'm no SQL expert, and while I've certainly read up on stored procedures, the variances of T-SQL (as I use MS SQL Server at work), and fun stuff like cursors (and how you should use them judiciously, at best), most of my ColdFusion queries revolve around four simple SQL statements: SELECT, INSERT, UPDATE and DELETE.

As I've been refactoring some existing code as part of the development of a new, modern codebase for one of our most important applications, I've been trying to ask myself "OK, if I'm making a series of queries, can't I just do that all in the database instead?" The answer, often, is yes.

Here's an example, and one that I think is quite common: You need to query the database to check on the existence or state or value in a particular record. Then, depending on the value returned from that first query, you either do an update or an insert in to the database, or nothing at all. I've covered the basics of the upsert (a conditional INSERT or UPDATE depending on the existence of a matching record in the database) in a previous post, but I'd like to take it a step further in this example.

To be more specific in this example, I track all logins to the site. However, if the user logs in more than once in 10 minutes, I don't need to track that, as it's considered to be part of the same session. If I were going old-school with my code, I'd probably do something like this:

- Query the database to get the last login time of the user.
- If there's a record, compare the last login time to the current time, and if it is greater than 10 minutes ago, perform an update in the database.
- If there's no record in the database, perform an insert in the database.

So that will work out to three different <cfquery> calls, and a bunch of conditional <cfif>s.

There's a simpler way to do all of this in the database. It involves using (gasp!) an [IF tag in your SQL](http://doc.ddart.net/mssql/sql70/ia-iz_4.htm). Not too hard to do, really, and it takes care of everything in one query, rather than a series.

The code is as follows. I've included a "result flag" so you know whether any data update (insert or update) was performed.

{% highlight sql %}
// You have to first "initialize" your variables in T-SQL
DECLARE @lastVisitDate datetime
DECLARE @recordedLogin bit
set @recordedLogin = 0
// We get the last visit date as the result of a query. It will return NULL if no match is found (that is, a login that is occurred in the last 30 minutes).
// We use T-SQL's DateDiff function to compare the lastVisitDate value for this user to the current time, just like in ColdFusion.
set @lastVisitDate =
	( 
	SELECT lastVisitDate
	FROM loginTrackingTable
	WHERE user=<cfqueryparam cfsqltype="cf_sql_integer" value="#userID#">
	AND DateDiff(n, lastVisitDate, getDate()) < 10
	)
// If there is no login for this person in the last 10 minutes, we perform an upsert
IF @lastVisitDate IS NULL
BEGIN
	// Set our flag to let us know that we did record the login
	set @recordedLogin = 1
	// This is the upsert: we try an update first, and if no records were affected by the update, we do an insert.
	UPDATE loginTrackingTable
	SET lastVisitDate = <cfqueryparam cfsqltype="cf_sql_timestamp" value="#CreateODBCDateTime(Now())#" />
	WHERE userID=<cfqueryparam cfsqltype="cf_sql_integer" value="#.userID#">
	// If no records were affected, the value of the @@rowcount variable will be zero
	IF @@rowcount = 0
	BEGIN
		INSERT INTO loginTrackingTable (
			userID, lastVisitDate
		)
		VALUES (
			<cfqueryparam cfsqltype="cf_sql_integer" value="#userID#">,
			<cfqueryparam cfsqltype="cf_sql_timestamp" value="#CreateODBCDate(Now())#">,
		)
	END
END
// This returns the recordedLogin flag back to CF
SELECT @recordedLogin as resultFlag
{% endhighlight %}

[IF statements in T-SQL](http://doc.ddart.net/mssql/sql70/ia-iz_4.htm") work a whole lot like <cfif>s, the major difference being the IF...BEGIN...END structure. It's really easy to figure out and, yes, Virginia, you can do ELSEs as well.

So instead of making two or three <cfqueries> in ColdFusion, we've done the decision making inside of the database itself. This cuts down on traffic from the CF server to the database, which is always a good thing. Are you going to get a huge performance increase by doing this? Not necessarily, but it helps to keep the logic together, and simple when possible.