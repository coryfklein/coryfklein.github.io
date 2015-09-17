# Using Scala's RegexParsers to create a grammar for interpreting a REST data query

## Introduction

Say we want to provide a way for users to find all brown bunny rabbits under the age of 3 whose name contains the letter c. This is one way we could make such a request via a RESTful api.

    GET bunnies.com/bunnies?filter=age < 3, name ~ 'c', color == brown
    

In this post, I will explain how to use Scala to create a grammar representing this query that could be used to produce the specific data requested.

## Specify The Grammar

First we need to formalize the grammar. We are really only concerned with this part of the string.

    age < 3, name ~ 'c', color == brown
    

We need to break this string down into smaller and smaller parts. We'll start out with the string itself, and break it down until we get to atomic pieces.

### Expression List

Let's call the entire string an `expression-list`, and each specific comma-separated item an `expression`. For our example string, that would give us three `expressions` making up a single `expression-list`.

For now we'll just use a pseudo-language to define the grammar and help us visually layout the structure of the grammar. Later on we will convert it to functional Scala code.

    expression-list = expression (, expression)*
    

### Expression

Here are the three expressions from our example

    age < 3
    name ~ 'c'
    color == brown    
    

Each is composed of three parts. A name, comparator, and a value. The name always appears on the left of the comparator, and the value on the right.

    expression = name comparator value
    

### Name

An `name` is a symbol. Let's follow standard convention and say that these must start with a letter or an underscore, which can then be followed by any number of letters, numbers, or underscores.

    name = [_a-zA-Z]+[_a-zA-Z0-9]*
    

If you are unfamiliar with this syntax, we've borrowed the grouping characters `[` and `]` from regular expressions and used them to indicate a group of valid characters. The `+` and `*` characters also are reminiscent of regular expressions and indicate "one or more" and "zero or more" respectively.

### Comparator

A `comparator` is really just a keyword or static phrase to reference which specific operation we use to compare the contents of a `name` to a `value`. In this language, let's just support the ones from the example. Namely, `<` (less than), `~` (contains), and `==` (equals).

    comparator = < | ~ | ==
    

The `|` character is called a pipe, and is used as an alternator, which means that a `comparator` can be a `<` OR A `~` OR A `==`.

### Value

A `value` represents something that can be compared against. A `name` only refers to a symbol or variable, but a `value` can be anything that could possibly be contained inside that variable, so we need to be more liberal with our definition.

Ultimately, we would like anything to qualify as a `value`, but since these will be transmitted in a URL and also need to parseable, we will have to make some restrictions. Namely, we will disallow the ampersand and the comma.

    value = [^&',]+ | '[^&,]*'
    

Let's break this down. There are two possibilities, unquoted `[^&,]+` and quoted `'[^&,]*'` values. Adding in quotes to our grammar allows us the ability to create empty expressions like this.

    middleName == ''
    

The inside part `[^&',]` introduces a new character, the caret `^`. It is used inside the square bracket grouping to indicate that the characters in the grouping are *not* allowed.

So `[^&',]` says that any characters that are *not* an ampersand, single quote, or comma *are* allowed as part of a `value`. Let's take a look at each disallowed character.

*   ampersand `&` - Disallowed because it is used in URLs to separate query parameters, so it is reserved.
*   single quote `'` - The single quote is reserved to designate values themselves, a la `name ~ 'c'`. If we didn't have this rule, then the single quotes would be interpreted as part of the value rather than defining it.
*   comma `,` - Commas are already used to separate `expressions` in an `expression-list`, so they are reserved.

### Complete Grammar

If you put all those rules together, they look like this.

    comparator = < | ~ | ==
    value = [^&',]+ | '[^&,]*'
    name = [_a-zA-Z]+[_a-zA-Z0-9]*
    expression = name comparator value
    expression-list = expression (, expression)*
    

## (Interlude) Set Up

Now, before we actually translate the grammar we just defined into a working Scala program, we need to set up our Scala development environment and [RegexParser][1].

First, let's set up some boilerplate code to compile, run, and test this. We need to create three files: the main app, the parser, and a short compile script. We also create a directory named `classes` to keep our compiled code tidily separated from our source code.

    ~/BunnyParser $ ls
    App.scala BunnyParser.scala classes/ compile-and-run.sh
    

Here is our main Scala object

    // App.scala
    object App {
      def main(args: Array[String]) {
        println(BunnyParser.parse("Hello World!"))
      }
    }
    

Here is the parser. For now, it just uses a single grammar expression that matches everything (`matchesEverything = .*`).

    // BunnyParser.scala
    import scala.util.parsing.combinator.RegexParsers
    import scala.util.parsing.input.CharSequenceReader
    
    object BunnyParser extends RegexParsers {
    
      def matchesEverything = """.*""".r
    
      def parse(s: String) = {
        parseAll(matchesEverything, new CharSequenceReader(s))
      }
    }
    

And a simple script to compile and run the code.

    #!/bin/bash
    # compile-and-run.sh
    scalac -d classes *.scala && scala -cp classes App
    

You can download a this starting setup here.

Once you have it ready, just do `chmod u+x compile-and-run.sh`, and then compile and run it like this.

    ~/BunnyParser $ ./compile-and-run.sh
    [1.13] parsed: Hello World!    
    

That just took the string "Hello World!" and interpreted it with the extremely simple catch-all grammar. Now it's time to swap in our own grammar.

## Code It

Here is the summarized grammar specification that we created earlier.

    comparator = < | ~ | ==
    value = [^&',]+ | '[^&,]*'
    name = [_a-zA-Z]+[_a-zA-Z0-9]*
    expression = name comparator value
    expression-list = expression (, expression)*
    

Let's take each grammar rule and translate it into a valid RegexParser expression in our BunnyParser.

### Comparator

There are three possible comparators. We could just refer to them directly by their associated symbols `<`, `~`, and `==`, but it is cleaner if we define a grammar for each specific comparator and then refer to those grammars.

    def lessthan = "<"
    def contains = "~"
    def equals = "=="
    def comparator = lessthan | contains | equals
    

Now if we ever want to amend this specification to include synonyms for the comparators, we could do it by just changing these definitions.

    def lessthan = "<" | "lt"
    def contains = "~" | "ct"
    def equals = "==" | "eq"
    def comparator = lessthan | contains | equals
    

### Value

For `value`, we need to use regular expressions. Fortunately, Scala makes this really easy. To compile the string `'sometext'` into a regular expression, we just type it like this: `"""sometext""".r`.

Using this information, we can now define our `value` grammar.

    def quotedValue = """'[^'&,]*'""".r
    def unquotedValue = """[^&',]+""".r
    def value = unquotedValue | quotedValue
    

### Name

`name` can be compiled similarly.

    def name = """[_a-zA-Z]+[_a-zA-Z0-9]*""".r
    

### Expression

An `expression` is a combination of `name`, `comparator`, and `value`. These can be [sequenced][2] using the `~` character - not to be confused with the character we are defining in our grammar to mean "contains".

    def expression = name ~ comparator ~ value
    

### Expression List

In order to keep the rules of our grammar easy to read, we will split the expression list into two parts: head and tail. The head is just an expression, and the tail is a list of zero or more comma-separated expressions.

    def expressionTail = "," ~ expression
    def expressionList = expression ~ (expressionTail*)
    

### Try It

Here is our complete parser, with the `parse` method updated to take our newly minted `expressionList` grammar.

    import scala.util.parsing.combinator.RegexParsers
    import scala.util.parsing.input.CharSequenceReader
    
    object BunnyParser extends RegexParsers {
    
      def lessthan = "<"
      def contains = "~"
      def equals = "=="
      def comparator = lessthan | contains | equals
    
      def quotedValue = """'[^'&,]*'""".r
      def unquotedValue = """[^&',]+""".r
      def value = unquotedValue | quotedValue
    
      def name = """[_a-zA-Z]+[_a-zA-Z0-9]*""".r
    
      def expression = name ~ comparator ~ value
    
        def expressionTail = "," ~ expression
        def expressionList = expression ~ (expressionTail*)
    
      def parse(s: String) = {
        parseAll(expressionList, new CharSequenceReader(s))
      }
    }
    

Then we can test our parser with some expressions like this.

    object App {
      def main(args: Array[String]) {
        println(BunnyParser.parse("age < 3, name ~ 'c', color == brown"))
      }
    }
    

Run it and everything looks great!

    $ ./compile-and-run.sh 
    [1.28] parsed: (((age~<)~3)~List((,~((name~~)~'c')), (,~((color~==)~brown))))
    

## Abstract Syntax Tree (AST)

Parsing a string into a grammar is useless without actually doing something with it. You may not have noticed, but our `BunnyParser` actually returned an object that is vaguely tree-like. The result `(((age~<)~3)~List((,~((name~~)~'c')), (,~((color~==)~brown))))` could be diagrammed something like this.

    age
      \
       <
        \
         3
          \
           List
     ,        ,
      \        \
       name     color
        \        \
         ~        ==
          \        \
           'c'      brown
    

In order to intelligently interpret this data, we might want to associate some types to the parts of the tree.

    age (Name)
      \
       < (Comparator)
        \
         3 (Value)
          \
           List (ExpressionList)
     ,        ,
      \        \
       name     color (Name)
        \        \
         ~        == (Comparator)
          \        \
           'c'      brown (Value)
    

In addition, we would really like the hierarchy of the objects to better match the hierarchy of the domain.

                          List
                           |
          -----------------------------------
          |                |                |
         Exp              Exp              Exp
        / | \            / | \            / | \
       /  |  \          /  |  \          /  |  \
      /   |   \        /   |   \        /   |   \
    Name Comp Val    Name Comp Val    Name Comp Val
     |    |    |      |    |    |      |    |    |
    age   <    3     name  ~   'c'   color  ==  brown
    

Our grammar was driven by the design of the language, but we need another structure driven by the semantic meaning of the symbols. This is called an Abstract Syntax Tree, or AST.

The AST will be easily traversable, and thus easier to actually apply to our data as a "query", which was our original goal with creating this grammar and interpreter.

We can model the AST exactly off of the above diagram. Let's create a new file `AST.scala`.

    // AST.scala
    object Comparator extends Enumeration {
        type Comparator = Value
        val LessThan, Contains, Equal = Value
    }
    import Comparator._
    
    case class Name(name: String)
    case class Value(value: String)
    case class Expression(name: Name, comparator: Comparator, value: Value)
    case class Filter(expressions: List[Expression])
    

Now that we have defined the AST, we need to modify our grammar to create an instance of it when we interpret a string. Before, all of our grammar definitions actually returned a `Parser` object by default. We want them to instead return the appropriate object from our AST.

This is done using the `^^` and `^^^` operators. The double caret `^^` can be used to access the string value that was parsed, while the triple caret `^^^` just returns a static value.

      def lessthan = "<"
      def contains = "~"
      def equals = "=="
      def comparator = (lessthan ^^^ Comparator.LessThan
        | contains ^^^ Comparator.Contains
        | equals ^^^ Comparator.Equal)
    
      def quotedValue = """'[^'&,]*'""".r ^^ { case v => {
        Value(v.substring(1, v.size-1))
      } }
      def unquotedValue = """[^&',]+""".r ^^ { case v => Value(v) }
      def value = (unquotedValue | quotedValue) ^^ { case v => v }
    
      def name = """[_a-zA-Z]+[_a-zA-Z0-9]*""".r ^^ { case n => Name(n) }
    
      def expression = name ~ comparator ~ value ^^ { case n ~ c ~ v => Expression(n, c, v) }
    
      def expressionTail = "," ~ expression ^^ { case _ ~ e => e }
      def expressionList = expression ~ (expressionTail*) ^^ { case e ~ l => e :: l }
    

Finally, to verify that our AST was created correctly, we add `toString` methods to our AST. Once we have parsed the AST, we can just print it out to see if it matches our expectations.

    case class Name(name: String) {
        override def toString = name
    }
    case class Value(value: String) {
        override def toString = value
    }
    
    case class Expression(name: Name, comparator: Comparator, value: Value) {
        override def toString = name.toString + " " + comparator.toString + " " + value.toString
    }
    
    case class Filter(expressions: List[Expression]) {
        override def toString = expressions.mkString(", ")
    }
    

Compile and run, and voila! We have now created an AST representing the query we started with.

    $ ./compile-and-run.sh 
    [1.36] parsed: List(age LessThan 3, name Contains c, color Equal brown)
    

At this point, how you use the AST will be dependent on your situation, but you now have a fully typed and instantiated structure that represents a query that originally existed just as a stream of characters. Traverse it with whatever tree-traversal algorithm is appropriate and apply it to your data query.

 [1]: http://www.scala-lang.org/api/2.10.4/index.html#scala.util.parsing.combinator.RegexParsers
 [2]: http://www.scala-lang.org/api/2.10.4/index.html#scala.util.parsing.combinator.Parsers
