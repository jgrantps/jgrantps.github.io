---
layout: post
title:      "Nested Classes in Object Oriented Javascript"
date:       2020-02-05 12:53:50 -0500
permalink:  nested_classes_in_object_oriented_javascript
---


In Ruby On Rails we are used to the Active Record conventions of an object that belongs to, or has many of, other objects.  Active Record takes care of all the heavy lifting in linking objects together and providing us suites of methods for manipulating the relationships.  In JavaScript, this functionality is a bit trickier to pull off.  In this post I'm going to walk through a couple of my 'Rails with JS' app objects and explain how I was able to achieve some Active Record-like functionality in Java Script.

The back-end of my app holds a `user` model, which `has_many: selections` and then `has_many :characters, through: :selections`.  Typical Ruby On Rails would allow for functionality like the following:

```
class User
  has_many :selections
	has_many :characters, through: selections
end

user.selections.build
# => #<Selection id: nil, character_id: nil, user_id: 1>

user.selections.build(character_id:12)
# => #<Selection id: nil, character_id: 12, user_id: 1>
user.selections.save

user.selections.first 
#=>#<Selection id: 1, character_id: 12, user_id: 1>
user.selections.first.character
#=>#<Character id: 12 name: character_name, description: character_description>

```



To reflect these AR relationships accurately in Javascript, I looked at the functionality for each of these AR/Ruby method calls on a granular level and worked towards rebuilding them in JS.

Before I could realistically accomplish this I realized I needed to make sure that the nesting relationship between `user` and `selection` was accurate in the incoming JSON object.  (note how the below code takes in just a single argument — `input` —into the `User` constructor method)This was achieved by having an instance method on the `User` model that built out a specific JSON string for all selections belonging to any particular `User` instance.  This method, when called by the JSON serializer, provided consistently formatted `selection` JSON data as a part of my `User` JSON package.


I first started by building out my User class object to include a method that would return all the selections belonging to that User:

```
var user = new User(jsonInput)
user.selections 
// => [Selection {0}, Selection {1}, Selection {2}, ]
```

Either a constructor method or a getter method would be required for this ability.  I ultimately decided that a constructor method would be best suited, as it removed the need for extra `user.selections` calls elsewhere in the code, and would allow for the building of the `selections` components in the DOM to be automatic.  This 'selections auto-load' feature of the User class was achieved through an IIFE:

```
class User {
    constructor(input) {
       var  includedSelections = [];
        this.input = input
        this.id = input.data.id
        this.name = input.data.attributes.name
        
        this.selections = (function() {
            includedSelections = [];
            var selectionSet = input.data.attributes.selections;
            
            for (let element of selectionSet) {
               let selection = new Selection(element);
                includedSelections.push(selection)
            }            
            return includedSelections
        })(input);
				//...
```

Now, once a `user` is loaded in JS, all of the `user.selections` components *as well as the `user.selections[id].attributes`* are available to the program, just like in Rails!

This approach can be done on the other side of the `selections` class, with the auto-loading of a selection's character.  Lets take a look at the `Selection` class object that is being called in the User constructor in the code above:


```
class Selection {
    constructor(input) {
        let existingCharacter;
    
        this.id = input.id;
        this.user_id = input.user_id;
        this.character_id = input.character_id;
        this.character = (existingCharacter = includedCharacters.find(element => element.id === input.character_id)) ? existingCharacter : new Character(input.character);
        this.save();
    }     
```

You can see here that `this.character` fixes the entire `Character` object to the attribute, allowing a further extension of functionality vis a vis:

```
user.selections[id].character.name
//=> "characterName"
```

The final Active Record relationship that I wanted to achieve in JS is that characters `belongs_to :series`, and a `Series` instance `has_many :characters`.  This is done in the same way as how selections `belongs_to :character`:

```
class Character {
    constructor(characterData) {
        let existingSeries;
      this.id = characterData.id;
      this.name = characterData.name;
      this.description = characterData.description;
      this.image_URL = characterData.image_URL;
      this.series_id = characterData.series_id;
      this.series = (existingSeries = includedSeries.find(element => element.id === this.series_id)) ? existingSeries : new Series(characterData.series);
      this.save();
    }
```

The `this.series=...` constructor function allows for the same full-function reference ability to the corresponding `series` of a `character` as enjoyed by a `selection` to its corresponding `character` object, all while keeping the attributes of each object in the chain available in the code:

```
user.selections[id].character.series.title
//=> "seriesTitle"
```

This essentially completes the rebuild of the Active Record Functionality enjoyed by Rails in my JavaScript front end, *from the perspective of the `user`* There is still more work to be done if one wanted to, for example, call the User of a given selection:

```
var selection1 = user.selections[id]

selection1.user
//=> reference error!... we don't have a 'this.user=` constructor or getter method in the Selection class...
```

I don't think adding this further functionality would be difficult... I just didn't have any used for it in my particular app. 

Once you understand the approach of instantiating nested JS objects by using constructor methods, and can couple that approach with properly configured JSON data from your back end, you can enjoy much of the convenience and functionality of Active Record inside your Java Script code.


