## Basics

Ruby is a dynamically typed interpreted highly object oriented language.

The language is quite liberal and there are several different ways to do many things. This sometimes makes things tricky for a beginner

The code is written in rb files and executed (in the case of normal Ruby code) from the command line using the Ruby command.

A Ruby programmer's best friend is the interactive interpreter IRB, which is started with the command irb. The Rails console is equivalent to irb, but is executed in the context of a Rails application.

There are plenty of Ruby tutorials on the web, see http://www.ruby-lang.org/en/

You should read right from the beginning
* http://www.ruby-lang.org/en/documentation/quickstart/

A little more time-consuming and comprehensive, but very useful Rubykoans can be found in the following
* http://rubykoans.com/

Tutorials found to be good or at least tolerable include:
* https://www.railstutorial.org/book/rails_flavored_ruby#sec-strings_and_methods
* http://rubylearning.com/satishtalim/tutorial.html
* http://ruby.learncodethehardway.org/book/
* http://www.techotopia.com/index.php/Ruby_Essentials

Online version of the first edition of The Ruby book "Programming Ruby":
* http://ruby-doc.com/docs/ProgrammingRuby/

If you're going to buy one book on Ruby, I highly recommend Russ Olsen's __Eloquent Ruby__, see http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104 

The book is not a tutorial but is good for those with a programming background for those who have

The following lists some of the most relevant things about Ruby for Rails

## Hash and symbols

Next is the irb session that creates a hash, or associative table, and adds data there with two different keys. Ruby symbols are used as keys, i.e. strings that start with "behave like a standard":

```ruby
irb(main):001:0> h = {}
=> {}
irb(main):002:0> h[:name] = "Pekka"
=> "Pekka"
irb(main):003:0> h[:age] = 25
=> 25
irb(main):004:0> h
=> {:name=>"Pekka", :age=>25}
irb(main):005:0> h[:age]
=> 25
irb(main):006:0> h[:address]
=> nil
irb(main):007:0>
```

In the following, a hash is created which receives its content upon creation:

```ruby
irb(main):010:0> h2 = { name:"Arto", age:36 }
=> {:name=>"Arto", :age=>36}
irb(main):011:0> h2[:name]
=> "Arto"
irb(main):012:0> h2.keys
=> [:name, :age]
irb(main):013:0> h2.values
=> ["Arto", 36]
irb(main):014:0>
```

In the previous example, a hash was created using the new syntax used since Ruby 1.9 for creating symbol key hashes. Here's an example using the old "hashrocket" syntax:

```ruby
irb(main):014:0> h3 = { :name => "Esko", :address => "Westend" }
=> {:name=>"Esko", :address=>"Westend"}
```

read more at the following link:
* http://nicholasjohnson.com/ruby/ruby-course/exercises/hashes-and-symbols/

## Array

In the following IRB session, where a table is created, data is added to the table in a couple of different ways, the data is accessed and the contents of the table are printed in two different ways with a (very rarely used in Ruby) for statement and in a Ruby-like way with the each iterator:

```ruby
irb(main):015:0> t = []
=> []
irb(main):016:0> t[0] = "eka"
=> "eka"
irb(main):017:0> t << "toka"
=> ["eka", "toka"]
irb(main):018:0> t << 23
=> ["eka", "toka", 23]
irb(main):019:0> t
=> ["eka", "toka", 23]
irb(main):020:0> t.first
=> "eka"
irb(main):021:0> t.last
=> 23
irb(main):022:0> for element in t do
irb(main):023:1* puts element
irb(main):024:1> end
eka
toka
23
=> ["eka", "toka", 23]
irb(main):025:0> t.each { |element| puts element }
eka
toka
23
=> ["eka", "toka", 23]
irb(main):026:0>
```

read more from
* http://www.techotopia.com/index.php/Understanding_Ruby_Arrays

## Each

Although the familiar for and while statements found in Ruby, they are used very rarely. In particular, the traversal of different collections, e.g. tables, is practically always handled using the each iterator.

the working principle of the each iterator is as follows. When calling each on an array, the iterator returns the elements of the array one at a time and gives them to the block of code that follows the iterator using a named variable. For example, in the following block defined by {}, the variable <code>element</code> gets its value one at a time from the <code>t</code> element of each table:

```ruby
irb(main):027:0> t = [1,2,3,4,5]
 => [1, 2, 3, 4, 5]
irb(main):028:0> t.each { |element| puts element }
1
2
3
4
5
```

In Ruby, an alternative way to define a block of code is as follows

```ruby
t.each do |element|
 puts element
end
```

## Block

Generally, the practice is to define a block with curly braces if it is a block that fits on one line, while longer blocks are usually defined with do and end.

Blocks are core feature of Ruby and allow us to do much more also. Read more at the following link:
* https://www.tutorialspoint.com/ruby/ruby_blocks.htm

## Strings

Read more at the following link:
* https://www.rubyguides.com/2019/07/ruby-string-concatenation/

## Modules

Read more at the following link:
* http://juixe.com/techknow/index.php/2006/06/15/mixins-in-ruby/