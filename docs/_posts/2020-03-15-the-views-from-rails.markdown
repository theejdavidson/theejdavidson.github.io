---
layout: post
title:  "Views From Rails"
date:   2020-03-03 16:50:53 -0600
categories: jekyll update
---

Like ActiveRecord, the backend ORM used with Rails, Action View makes the use of Macros for streamlining the process of writing view templates (The V in the MVC Pattern). The key to this streamlining is the generation of helper methods that are commonly used within view forms.

The standard Ruby view form both in and outside of rails is written in ERB (Embedded Ruby) which is a mix of HTML and Ruby code. Continuing our Pokémon examples from the last blog, a simple snippet for a Pokémon index page would look like this:

```erb
<h1>Pokémon:</h1>
<ul>
<% @pokemons.each do |pokemon| %>
    <li><%= pokemon.name %></li>
<% end %>
</ul>
```

This would give us a simple bulleted list with names of all Pokémon in our collection. Since our application would typically have a 'show' page as well for each Pokémon in the collection, we might want to use the above list to link to those pages. A refactor would look like this:

```erb
<h1>Pokémon:</h1>
<ul>
<% @pokemons.each do |pokemon| %>
    <li><a href="/pokemons/<%= pokemon.id %>">
    <%= pokemon.name %></a></li>
<% end %>
</ul>
```

The above anchor tag acts as a somewhat hardcoded link to the show page of a given Pokémon. With Rails, we can use the link_to helper method to clean this up:

```erb
<h1>Pokémon:</h1>
<ul>
<% @pokemons.each do |pokemon| %>
    <li><%= link_to pokemon.name, pokemon_path(pokemon) %></li>
<% end %>
</ul>
```

Another area we could use Rails helpers is in the creation of a new Pokémon. In standard ERB we would write a form as follows:

```erb
<h1> Create a New Pokémon:</h1>
<form method="post" action="/pokemons">
<label for="Name"></label>Name:
<input id="Name" type="text" name="pokemon[name]"><br><br>
<label for="Type"></label>Type:
<input id="Type" type="text" name="pokemon[type]"><br><br>
<input type="submit" value="Create">
</form>
```

With Rails, we can use the helper method form_with to streamline this form:

```erb
<h1> Create a New Pokémon:</h1>
<%= form_with model: @pokemon do |f| %>
<%= f.text_field :name %>
<%= f.text_field :type %>
<%= f.submit %>
<% end %>
```
As you can see, using these helper methods can speed up our workflow on Rails once we get a feel for using them.