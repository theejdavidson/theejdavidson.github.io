---
layout: post
title:  "Gotta Map 'Em All"
date:   2020-03-03 16:50:53 -0600
categories: jekyll update
---
Three weeks into [Flatiron School][flatiron-school] and we've hit the ground running with our first project. Using [Ruby's][ruby-lang] ActiveRecord to send GET requests to [PokéAPI][poke-api], an API containing extensive pokémon-related data, we were able to seed our database for tests.

For a little more info on how all these tools work together, I'll explain them piece by piece. To set up a database that we can interract with manually (sans ActiveRecord), the following can be placed in the '../config/environment.rb' ruby file: 

```ruby
DB = {:conn => SQLite3::Database.new("db/students.db")}
```

This bit of code allows us to call DB[:conn] to set up a database connection elsewhere in our application.

Since the standard way to perform these maps is with a CRUD (Create, Read, Update, Delete) pattern, I'll go through what each step might have looked like in our project if we didn't get to use ActiveRecord to map our Objects to the Relational Database in the case of our Pokemon Class. 

![image]({{ site.baseurl }}/assets/images/grimer.png)

Inside of our Pokemon Class we need to consider the data that would apply to all wild Pokémon. They each have a name corresponding to their species, a primary ID key to preserve data integrity, and a type_id foreign key to access the type of pokémon that they are (fire type, water type, ghost type, ect.). To create this table we would insert the following method:

```ruby
  def self.create_table
    sql = <<-SQL
    CREATE TABLE IF NOT EXISTS pokemons(
      id INTEGER PRIMARY KEY,
      name TEXT,
      type_id INTEGER
    );
    SQL
    # this lets us talk to our SQL database
    DB[:conn].execute(sql) 
  end
```
With Pokemon.create_table, we can now create the table in our database (we pluralize tables with an s and pokémon are no exception). However, this method simply creates columns for the table itself and leaves us with no way to make an entry with a name. Fear not, Ruby has an answer to this! Since we are defining a class, we can write an initialize function and a special helper macro called :attr_reader. Within our Pokemon class we can define the following:

```ruby
# this allows us to access a name that has been set
# on an instance of pokemon as well as the primary key id and type foreign key
attr_reader :name, :id, :type_id
# we set the default value of id to nil and let SQL fill it in
def initialize(name, type_id, id = nil)
  @id = id
  @name = name
  @type_id = type_id
end
```

Now when we create an instance of Pokemon it will have a name and a space for a primary key id. Our creation process isn't done yet though, we still need to take the instance of our Ruby object and save it as a database record. We can write a Save method like this:

```ruby
def save
    sql = <<-SQL
    INSERT INTO pokemons (name, type_id) VALUES (?, ?);
    SQL

    DB[:conn].execute(sql, self.name, self.type_id)

    @id = DB[:conn].execute("SELECT last_insert_rowid() FROM pokemons;")[0][0]
  end
```

Now, our Pokémon instances can be saved to the database! Our C in crud is complete!

![image]({{ site.baseurl }}/assets/images/party.gif)

Don't celebrate just yet, we've got plenty more to write.

The next step is reading them, so let's write a simple query function that accepts the pokémon name as a string and returns the matching row from the database:

```ruby
 def self.find_by_name(name)
    sql = <<-SQL
    SELECT *
    FROM pokemons
    WHERE name = ?
    LIMIT 1;
    SQL

    DB[:conn].execute(sql, name).map do |row|
      self.new_from_db(row)
    end.first 
    # by chaining .first we return the single row instance from an array
  end
```

Next up is Update. For our use case, let's say we accidentally entered a Bikachu instead of Pikachu into our database. With an 'attr_writer :name' macro we could change the name of our ruby object instance, but we would prefer to update the database to correct it first.

```ruby
def update_name(new_name)
  sql = <<-SQL
  UPDATE pokemons
  SET name = ?
  WHERE name = ?
  SQL

  DB[:conn].execute(sql, new_name, self.name)
end
```
Finally, we need a method to Delete a row from our database altogether. We can call this on an instance of pokémon to delete it from the database and let the garbage collector handle it's existence as a Ruby Object. The delete method would simply look like this:

```ruby
def delete
  sql = <<-SQL
  DELETE FROM pokemons
  WHERE name = ?
  SQL

  DB[:conn].execute(sql, self.name)
end
```

With these methods, we have a simple database CRUD functionality in place. However, this work needs to be done for every object/table relationship within our app and more specific queries and updates all need their own methods as well. All that typing can get tiring pretty quickly!

![image]({{ site.baseurl }}/assets/images/tired_pika.gif)

Lucky for us, Ruby has a tool called ActiveRecord that can save us all this work and a lot more.

ActiveRecord is a framework that abstracts away a large amount of code that comes from Object Relational Mapping by giving us inherited methods. With 

```ruby 
gem "activerecord"
```
in our Gemfile and 

```ruby
require 'sinatra/activerecord/rake'
```
in our Rakefile, we can allow our Pokémon class to inherit the ActiveRecord base object:

```ruby
class Pokemon < ActiveRecord::Base
end
```

Out of the box, this gives us the ability to call the following methods:
```ruby
# both creates the object and saves the relational mapping to the database
Pokemon.create(name: "Bikachu")
# returns the entire pokémon collection
Pokemon.all
# finds a pokémon by name or any other specified keyword
pika_correction = Pokemon.find_by(name: "Bikachu")
# updates an incorrect field
pika_correction.update(name: "Pikachu")
# deletes a row
pika_correction.destroy
# deletes all rows in the table
Pokemon.destroy_all
```
ActiveRecord also greatly simplifies table relations with the macros for associations such as has_one and belongs_to that generate primary/foreign key relationships as well as has-many :through and has-one :through to generate linking tables. More reading on that can be found in the [ActiveRecord Guide][activerecord-guide].

Thanks for reading, I'll keep posting as our project grows!

[poke-api]: https://pokeapi.co/
[ruby-lang]: https://www.ruby-lang.org/en/
[flatiron-school]: https://flatironschool.com/
[activerecord-guide]: https://guides.rubyonrails.org/association_basics.html
