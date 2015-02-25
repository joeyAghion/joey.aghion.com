---
title: Instance variables, class variables, and class instance variables in Ruby
date: 2015-02-25 08:16 -05:00
tags: programming, ruby
---

Trying to understand how instance variables, class variables, and _class_ instance variables interacted in Ruby, I wrote this quick demo:

    class Foo
      def get_instance_v
        @v ||= '@v'.tap { puts 'Setting @v from instance method!' }
      end
    
      def get_class_v
        @@v ||= '@@v'.tap { puts 'Setting @@v from instance method!' }
      end
    
      def self.get_instance_v
        @v ||= 'class @v'.tap { puts 'Setting @v from class method!' }
      end
    
      def self.get_class_v
        @@v ||= 'class @@v'.tap { puts 'Setting @@v from class method!' }
      end
    end

Try to guess what will happen...

    > foo = Foo.new
    > foo.get_instance_v
    Setting @v from instance method!
    => "@v"

That's pretty predictable. Let's set the class variable `@@v` from an instance method:

    > foo.get_class_v
    Setting @@v from instance method!
    => "@@v"

All good. Now, let's set an instance variable `@v` from within a class method (usually called a _class instance variable_).

    > foo.class.get_instance_v
    Setting @v from class method!
    => "class @v"

Equivalently, we could have called `Foo.get_instance_v`. Now, let's again refer to the class variable `@@v`, but from a class method:

    > foo.class.get_class_v
    => "@@v"

(It was already set by the instance method.) The `@@v` reference resolves to the same variable from an instance context _or_ a class context.

Now we'll extend the `Foo` class and see how these variables work with inheritance.

    class Bar < Foo
      def get_instance_v
        @v ||= '@v in Bar'.tap { puts 'Setting @v from Bar instance method!' }
      end
    
      def get_class_v
        @@v ||= '@@v in Bar'.tap { puts 'Setting @@v from Bar instance method!' }
      end
    
      def self.get_instance_v
        @v ||= 'class @v in Bar'.tap { puts 'Setting @v from Bar class method!' }
      end
    
      def self.get_class_v
        @@v ||= 'class @@v in Bar'.tap { puts 'Setting @@v from Bar class method!' }
      end
    end

As expected, instance variables are scoped to our instance:

    > bar = Bar.new
    > bar.get_instance_v
    Setting @v from Bar instance method!
    => "@v in Bar"

What about class variables?

    > bar.get_class_v
    => "@@v"

Apparently, class variable references in the child class refer to the base class. But is that always the case? We'll reopen both classes to see what happens if the class variable is first set in the child class:

    class Foo
      def get_class_x
        @@x ||= "Foo's @@x".tap { puts 'Setting @@x in Foo' }
      end
    end

    class Bar < Foo
      def get_class_x
        @@x ||= "Bar's @@x".tap { puts 'Setting @@x in Bar' }
      end
    end

    > bar.get_class_x
    Setting @@x in Bar
    => "Bar's @@x"
    > foo.get_class_x
    Setting @@x in Foo
    => "Foo's @@x"

That was a surprise. The class variable isn't exactly _shared_ between the parent and child classes. The child class looks up the class variable in the base class, but sets it on itself. The base class keeps its own copy.

Something strange happens if the child class again references the class variable:

    > bar.get_class_x
    "Foo's @@x"

Huh? Apparently the child class _prefers_ the class variable set on its base class to its own.

There are further wrinkles when these class variables [and class instance variables] are used from modules, but that's a topic for another post.
