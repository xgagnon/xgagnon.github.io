

# An Easier Way to Manage your Database

Do you ever wish database management was easier? During development there are always many changes made to your database as the project evolves. You need someone, or even multiple people sometimes to keep track of all of these changes. When they were made, what was changed and why, and to keep documentation of each iteration of your database. Managing this feels tedious and frustrating, and  you can wish there was an easier way to do it. What if I told you there was? In this blog I am going to introduce you to the wonderful database management tool called Liquibase. I’m going to go over topics like what is Liquibase? I’ll go over some of it’s basic concepts, and a more in depth look at ho it works. Hopefully by the end of this read, you will have a good understanding of what Liquibase is, and how you might be able to implement it to improve your tech stack at work.

### What is Liquibase?

For the reader who do have never heard of Liquibase, here is a brief introduction of what Liquibase is. Liquibase is an open-source database management tool meant to track database versions and deploy database changes made by any developer on the team. There are four different languages you can write your scripts in, including XML, YAML, JSON and SQL, and over fifty different database management systems, like Oracle, MySQL, or SQL Server, that Liquibase can be integrated into. It has several useful features as well including version control, the ability to rollback changes, and the use of change sets and change logs to control all of your database changes.

### Basic concepts

There are a handful of concepts you need to be familiar with to understand how Liquibase works and how you can use it. The first of which are change sets and change logs. Change sets are straight forward. A change set is the actual change you are making to the database. That could be altering a table, changing its name or a column constraint. Essentially any change you want to make is made in a change set. A change log is a collection of all your change sets listed in the order in which the changes need to be applied. It also has a connection to the database that you want the changes to be applied to. There can be multiple change sets in a log, and there can be multiple change logs. The last concept you need to be familiar with is the Database change log table. This is an automatically generated table that keeps track of every changeLog that you have applied to the database. Each row in the table consists of a unique combination of ‘id’, ‘author’, and ‘filename’. This helps ensure that change logs are only applied once, and to keep track of all of your previous database versions, in case you ever need to rollback to any of them.

### How it works

Now that I’ve gone over some of the basic concepts, we can dive a little deeper into how change sets are written and some of the best practices to organize your change logs.
Writing a change set is once again, simple. Most of the time they are written using XML, and always open with the <changeSet> tag. This lets Liquibase know that everything within this tag is only going to be one change made when the changeLog is applied. Let’s take an easy example to start with. We can write a change set to create a new table in a database.

```xml
  <changeSet id="set1" author="xgagnon">
		<createTable  tableName="user">
			<column  name="id"  type="number(10)">
				<constraints nullable="false" primaryKey="true" primaryKeyName="user_pkey"/>
			</column>
			<column  name="first_name"  type="varchar(20)"/>
			<column  name="last_name"  type="varchar(20)"/>
		</createTable> 	
	</changeSet>
```
 
Looking at the changeSet, it should be obvious what each tag in the change set does. We are creating a table called ‘user’ and adding three columns that represent user_id, first_name and last_name. We can even declare the  id column as the primary key. When this eventually gets applied in a changeLog, Liquibase will create this table. Now let’s say we forgot to add a column to the table, and the changes have already been applied to the database. We would then have to create another changeSet to add this column to the table.
```xml
  <changeSet  id="2"  author="xgagnon">  
		<addColumn  tableName="user">  
			<column  name="birthday"  type="date"/>  
		</addColumn>  
	</changeSet>
```
 
This changeSet again makes it obvious what each tag is doing. The benefit to this is that you don’t have to change the initial database schema when you make this change, making it easier to keep track of all the changes you have made without creating extra documentation.
Now we need to add the changeSets to a changelog. Doing this for one change is also easy. You just need to add the <databaseChangeLog> tag around all your changeSet.
```xml
 <databaseChangeLog
xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.9"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.9
http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.9.xsd">

	<changeSet id="set1" author="xgagnon">
		<createTable  tableName="user">
			<column  name="id"  type="number(10)">
				<constraints nullable="false" primaryKey="true" primaryKeyName="user_pkey"/>
			</column>
			<column  name="first_name"  type="varchar(20)"/>
			<column  name="last_name"  type="varchar(20)"/>
		</createTable> 	
	</changeSet>
</databaseChangeLog>
```
```xml
  <databaseChangeLog
xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.9"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.9
http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.9.xsd">

	<changeSet  id="2"  author="xgagnon">  
		<addColumn  tableName="user">  
			<column  name="birthday"  type="date"/>  
		</addColumn>  
	</changeSet>
</databaseChangeLog>
```
  
Every changeLog you create will need to be its own XML file. The way you can run multiple logs is to create a parent changeLog that instructs Liquibase to include all the child changeLogs.
```xml
<databaseChangeLog
xmlns="http://www.liquibase.org/xml/ns/dbchangelog/1.9"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog/1.9
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-1.9.xsd">

    <include file="set1.xml"/>
    <include file="set2.xml"/>
</databaseChangeLog>
```
 
This parent child relation ship can extend as many levels as you want if the parent changeLog has instruction to include the master changeLog of its next level of children.
As far as best practices go, it’s good to keep each changeLog to only one changeSet. This way it’s very easy to see what changes have been made, when they were made, and if there is an error when trying to apply the changeSets, which changeSet was the cause of the error. It also makes rearranging the order in which you want to apply the changeSets much easier. Instead of having to rearrange the whole block off code for each change set, you can simply rearrange the ‘include file’ table in your parent changeLog.
If you are working in an agile environment, it is also good practice to keep a task number associated with a group of changeLogs. This way you can have a folder for each task number and include the changeLog file of each task folder to another parent changeLog file. This once again makes it easier to keep track of what changes were made when, and why they were made.


### Why you should use Liquibase

If you spend to many hours manage and documenting you database, there is a very good chance that implementing Liquibase into your tech stack at work is going to make your life much easier. It lets you easily make database changes without loosing track of how your database has changed over a project. It lets you easily organize your changes so that they can be easier to trouble shoot, and to apply your changes in the correct order without rewriting a large portion of your database script. If you’re looking to make database management easier, looking to fill a database management role, or even to replace your current database manager with a more streamlined system the whole team can cooperate on, I think it is worth implementing Liquibase into your project.

---

### Resources

https://www.liquibase.org/
  
https://docs.liquibase.com/home.html 
