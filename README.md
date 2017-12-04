# Ruby Object Remembrance 

## Objectives

1. Explain the concept of remembrance in object-oriented programming. 
2. Use class variables to remember, or store, instances of a class that are produced. 

## Introduction

Let's say we're building a program to organize and explore our music library, or something to help us track and store the passwords for all of our internet accounts.

In all of these situations, our application will instantiate instances of the classes we design. These instances are what bring the code we write into existence. It would do us little good to write our program, bring it into existence and have to no way to refer to it, no way to use the instances we created. Let's take a look at the following example:

```ruby
class Song
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def play
    "Playing #{self.name}"
  end
end

Song.new("Thriller") 
```

We now have an instance of the "Thriller" song and lucky for us the instance knows how to play itself. How can we play it? Sadly we can't. When we instantiated the "Thriller" song we did not save a reference to it. We have no way of telling the song to play itself (calling the `#play` method or sending the `play` message to the song instance in Object Oriented parlance).

What is a first step towards solving this problem? Let's now save a reference to the song we instantiate.

```ruby
class Song
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def play
    "Playing #{self.name}"
  end
end

thriller = Song.new("Thriller")

thriller.play
```

By saving a reference to the instance of the "Thriller" song to a local variable we now have a way to play the song.

What happens if during our Flatiron graduation party we want to play all our songs? We would need to **manually gather** all the references to our song instances and cycle through them to ask each song to play itself. Wouldn't it make sense to automatically create a collection that records a reference to each song as we instantiate them? Anytime we want to act on the entirety of our music library we could refer to the collection.

Whether we are writing code to manage Songs or Passwords, our program should know how to keep track of the instances it creates. Luckily for us, Ruby allows us to do so by using class variables to store new instances as soon as they are created. Let's take a look together.


## Using class variables to store instances of a class

Let's rethink our music library application. We want to automatically keep track of all song instances. This feature would enable us to easily add fun features such as a method to play all of our songs.

### Creating the Class Variable

Let's take a step back and think about the concept of responsibility. Whose job is it to know about *every instance of the `Song` class*? We have two choices right now: an instance of the `Song` class or the `Song` class itself.

It is not the responsibility of an individual song to know about all of the other songs. Keeping track of all of the songs that it creates, however, fits right into the purview of the Song class. 

So, how can we tell the `Song` class to keep track of every instance that it creates? We can use a class variable. 

Let's create a class variable, `@@all`, that will store every instance of the `Song` class. Recall that `@@` before a variable name is how we define a class variable.

```ruby
class Song
	
  @@all = []
	
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def play
    "Playing #{self.name}"
  end
end
```

Notice that we set our class variable equal to an empty array. Arrays are perfect for storing lists of data, so we'll use an array to store our lists of `Song` instances. 

Now that our class is *set up* to store the instances that it produces, we have to ask: *how* does it store these instances?

### Adding Instances to the `@@all` Array

Before we can answer this question, we should ask another. *When* should the `Song` class become aware of, or store, an instance of itself? 

This should happen at the time on instantiation––when a new song gets created, it should be immediately stored by our `Song` class' `@@all`class variable. 

We can implement this by simply adding the new instance that gets created into the array stored in `@@all` *inside our `#initialize` method.*

Let's take a look:

```ruby
class Song
	
  @@all = []
	
  attr_accessor :name

  def initialize(name)
    @name = name
    @@all << self
  end

  def play
    "Playing #{self.name}"
  end
end
```

In `#initialize` we use the `self` keyword to refer to the new song instance that has just been created by `#new`. Remember that when `#new` is called, it creates a new instance of the class and then calls `#initialize` on that new instance. So, `#initialize` is technically an instance method. Inside an instance method we are in what is called **method scope** and `self` will refer to whichever instance the method is being called on. 

We push `self` into the array that is stored in `@@all`. In this way, the `@@all` class variable will point to an ever-growing array that contains every instance of the `Song` class that gets created. 

### Our Code in Action

Let's see what happens when we actually execute the code we've written:

```ruby
class Song
  
  @@all = []
  
  attr_accessor :name

  def initialize(name)
    @name = name
    @@all << self
  end

  def play
    "Playing #{self.name}"
  end
end

Song.new("99 Problems")
Song.new("Thriller")
```

Now that we've created some songs, let's ask our `Song` class to show us all of the instances that we just created:

```ruby
class Song
  
  @@all = []
  
  attr_accessor :name

  def initialize(name)
    @name = name
    @@all << self
  end

  def play
    "Playing #{self.name}"
  end
end

Song.new("99 Problems")
Song.new("Thriller")

Song.all # => NoMethodError: undefined method `all' for Song:Class
```

Looks like we don't have a class method to access the contents of the `@@all` array. That sounds right, we did not define a class method to access the contents of the `@@all`. Just like how we've built reader methods that expose the value of instance variables, we need to build a method that will expose, or make accessible outside of the class, the value of a class variable. 

Let's build one now.

### Building a Class Method to Access a Class Variable

Let's define a class method named `.all` to expose our `@@all` class variable.

```ruby
class Song

  @@all = []
	
  attr_accessor :name
	
  def initialize(name)
    @name = name
    @@all << self
  end
	
  def self.all
    @@all
  end

  def play
    "Playing #{self.name}"
  end
end
```

Recall that to define a class method we use the `def self.method_name` syntax. In this case, `self` refers to the class on which the class method is being defined. This is because we are in the **class scope** right now, not inside the method scope, i.e. in between the `def`/`end` method definition keywords. 

Now we can try again to call our class method on the `Song` class:

```ruby
class Song

  @@all = []
  
  attr_accessor :name
  
  def initialize(name)
    @name = name
    @@all << self
  end
  
  def self.all
    @@all
  end

  def play
    "Playing #{self.name}"
  end
end

Song.new("99 Problems")
Song.new("Thriller")

Song.all # Should output something like [#<Song:0x00007fc41902c738 @name="99 Problems">, #<Song:0x00007fc4189bc9e8 @name="Thriller">]
```

We did it! We used a class variable to store a collection of song instances. We added new instances to this storage container every time a new instance was created with the help of the `self` keyword in our `#initialize` method. Lastly, we wrote a class method to expose our song instances collection.

Let say we want a way to play all our songs. We could use the `.all` to obtain a reference to the songs collection and then call the `#play` method on each instance of a song. See the code below:


```ruby
class Song

  @@all = []
  
  attr_accessor :name
  
  def initialize(name)
    @name = name
    @@all << self
  end
  
  def self.all
    @@all
  end

  def play
    puts "Playing #{self.name}"
  end

  def self.play_all_songs
    self.all.each {|song|song.play}
  end
end

Song.new("99 Problems")
Song.new("Thriller")

Song.play_all_songs # Will output:
"Playing 99 Problems"
"Playing Thriller"
```

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/ruby-remembering-objects-readme' title='Ruby Object Remembrance'>Ruby Object Remembrance</a> on Learn.co and start learning to code for free.</p>
