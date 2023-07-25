

# Building your first Database

In my last blog post I went over some of the basic concepts of Liquibase. I talked about a change set, change log and the basic structure of how you might right and organize them. Now I’m going to help you build a database using the previous knowledge of Liquibase.

### Getting Set Up

There is only one package you will have to install to get Liquibase set up. 

https://www.liquibase.com/download?_ga=2.161084413.15932716.1690298390-542266862.1690298390

For the example I am going through today you are going to install the CLI Liquibase package. It can also be installed for different dependency technologies like Spring, Maven or Gradle, but to keep it simple I will only be using the command line interface in this blog.
Once you have this installed you should see a folder under C:/Program Files called “Liquibase”. In this folder you should see a sub folder called examples, which contains a script called start-h2. This will be the command we use to bring up a database management tool in your browser to help you see the changes you’ve made with Liquibase. We won’t be using any commands in this UI, but it is a helpful tool to see our changes.
Once you have confirmed you have the package installed, we can start writing our first changelog.


### First Change Log

I’m going to demonstrate this changelog using xml. There is a blank changelog template under the example/xml folder you can use to get started.
You will need to create a folder for your new database. For this example, I am going to copy the xml folder under examples and rename it to “blog”. The name and location are not important, if all the configuration files are in this folder, and you are able to find the folder location in the command line. I am only copying this folder and renaming it to use the configuration that has been made.
Once you have created your folder, you can open the blank template. For this demonstration we are going to build the database from the third semester web apps final project. This has 4 tables called user, role, category, and item. The first change log will only be the script to create these 4 tables. Here is the script:

```xml
  	<changeSet id="1" author="Xavier">
		<createTable tableName="user">
			<column name="email" type="varchar(40)">
				<constraints primaryKey="true" nullable="false"/>
			</column>
			<column name="active" type="int"/>
			<column name="first_name" type="varchar(20)"/>
			<column name="last_name" type="varchar(20)"/>
			<column name="password" type="varchar(20)"/>
			<column name="role" type="int"/>
		</createTable>
	</changeSet>

	<changeSet id="2" author="Xavier">
		<createTable tableName="role">
			<column name="role_id" type="int">
				<constraints primaryKey="true" nullable="false"/>
			</column>
			<column name="role_name" type="varchar(25)"/>
		</createTable>
	</changeSet>

	<changeSet id="3" author="Xavier">
		<createTable tableName="category">
			<column name="category_id" type="int" autoIncrement="true">
				<constraints primaryKey="true" nullable="false"/>
			</column>
			<column name="category_name" type="varchar(45)"/>
		</createTable>
	</changeSet>

	<changeSet id="4" author="Xavier">
		<createTable tableName="item">
			<column name="item_id" type="int" autoIncrement="true">
				<constraints primaryKey="true" nullable="false"/>
			</column>
			<column name="category" type="int"/>
			<column name="item_name" type="varchar(45)"/>
			<column name="price" type="double"/>
			<column name="owner" type="varchar(40)"/>
		</createTable>
	</changeSet>

```

Each table we are creating will have its own changeset, and each change set will need a unique combination of ids and authors. It is also a good idea to rename the template file to be more descriptive of what the changelog is doing. In this case I have renamed the file to “create_tables.xml”
We will also need to create a master change log to include all our smaller change log files. We can use the example script for this. First delete all the change sets within the databaseChangeLog tags. Then you will add one line that looks like this:

```xml
  <include file="create_tables.xml"/>
```
 
Make sure the file name within the include tag matches the file name of your change log. You can also rename this file. I have named it changelogs since it will include every change log we’ll be making.
The last step will be to reconfigure where Liquibase is looking for your change logs. To do this you will have to open the liquibase.properties file you should have in the folder with your xml scripts. Find the line that says changeLogFile. I found this on line 23, it may be different for you. Now set that value to equal the name of the file that will contain all your other change log files.


### Running your first changes

To apply your changes, you will need to open your command line, and navigate to the examples folder you downloaded previously, or to the folder where the start-h2 script is stored. Now run the command “start-h2”. This should bring up a UI in your browser. There should be no tables on the left side of your screen right now.
Now open a second terminal window and navigate to the folder where you have stored your xml scripts. Once you are in that directory on the command line, type “liquibase update”. You should see a message that says “Liquibase command 'update' was executed successfully”. If you see this message, go back to your browser and refresh the database view. You should now see 6 tables. The 4 you created, as well as 2 called DATABASECHANGELOG and DATABASECHANGELOGLOCK. These tables are created automatically to keep track of all the changes you have made to your database.

### Adding constraints

The last change we are going to make to this database in this blog is to add constraints to the tables. Create a new change log file and add the following code:

```xml
  <changeSet id="1" author="Xavier">
		<addForeignKeyConstraint constraintName="fk_user_role"
				 baseTableName="user" 
				 baseColumnNames="role" 
				 referencedTableName="role" 
				 referencedColumnNames="role_id"/>
	</changeSet>

	<changeSet id="2" author="Xavier">
		<addForeignKeyConstraint constraintName="fk_item_categories"
				 baseTableName="item" 
				 baseColumnNames="category" 
				 referencedTableName="category" 
				 referencedColumnNames="category_id"/>
	</changeSet>
	
	<changeSet id="3" author="Xavier">
		<addForeignKeyConstraint constraintName="fk_item_owner"
				 baseTableName="item" 
				 baseColumnNames="owner" 
				 referencedTableName="user" 
				 referencedColumnNames="email"/>
	</changeSet>
```

### Conclusion
There you have it. A working database that you created using only Liquibase. If you want to see all the changes that were made, you can perform a select statement on the DATABASECHANGELOG in the browser application. This should show all the changes you made, all with a unique combination of ids, authors, and filenames. Hopefully this demonstration was helpful and will help you integrate liquibase into your own applications. Liquibase is a very strong production tool, and I’m hoping to see more of it out in the world.

---

### Resources
https://www.liquibase.org/get-started/using-the-liquibase-installer)https://www.liquibase.org/get-started/using-the-liquibase-installer
