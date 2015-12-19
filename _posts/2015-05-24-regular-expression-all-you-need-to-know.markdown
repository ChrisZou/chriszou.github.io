---    
layout: post    
title:  "Regular expression: All you need to know"     
date:   2015-05-24 18:21  
categories: regular expression, ruby  
comments: true  
tags: [regex, ruby]
excerpt_separator: <!--more-->  
---  

Regular expressions are nuts. You spent lots of time studying them and later when you need to write one, you came up with nothing. That’s why you need a cheatsheet, like this one.  

## Abbreviations used in this post  

RE for regular expression.        
`/regular_expression_content/`   regular\_expression\_content is the text of a regular expression, The two "/" is there to indicate the this string is used as a regular expression, rather than a normal string.   
`<=>`   means “equals”, or “is the same as”.    
`=~`    means perform the match action. For example: `/\d+/ =~ “hello, 123!”` means match `“hello, 123!”` against the regular expression `\d+`.    
`#=>`   The content after `#=>` is the result of the matches before it.     

Yes, some of them are just Ruby syntax.    

## Normal text is RE  

Normal string like “hello” is valid regular expression that matches itself(“hello”).    
Example:    
`“Hello, my name is Chris” =~ /name/`   #=> “name”   

## Character class  

A character class is a set of characters inside a square bracket “[]”: `[abcdefg]`.    
A character class matches any **single** characters consists of it. So `/[abcdefg]/` matches any single characters among ‘a’, ‘b’, ‘c’, ..., ‘g’.    
You can use ‘-‘ to define a range for a  character class: [a-g], [a-z], [A-Z], [0-9], etc.    
Some  character classes are so commonly used that they made some *shortcuts* for them:    
\s <=> any space character, including ‘ ‘, tab and newline character.      
\S <=> any character expect space character(‘ ‘, tab and newline character)    
\w <=> any *word character*: a-z, A-Z, 0-9, and ‘\_’ Some programming languages limit you to define variable or method names using these characters, and this is why(or the result?).    
\W <=> any character except *word character*.    
\d <=> any number character, aka, [0123456789].    
\D <=> I guess you already know what it means: any character except number character.    

Example:    
`“Hello, 123!” =~ /\w/`     &nbsp; &nbsp; &nbsp; #=> “H”     
`“Hello, 123!” =~ /\W/`     &nbsp; &nbsp; &nbsp;  #=> “,”    
`“Hello, 123!” =~ /\s/`     &nbsp; &nbsp; &nbsp;  #=> “ ”    
`“Hello, 123!” =~ /\S/`     &nbsp; &nbsp; &nbsp;  #=> “H”    
`“Hello, 123!” =~ /\d/`    &nbsp; &nbsp; &nbsp;   #=> “1”    
`“Hello, 123!” =~ /\D/`     &nbsp; &nbsp; &nbsp;  #=> “H”    

## Anchors  

Anchors are characters that specify a position. Commonly known are “^” which specifies the start of a line, and “$” which specifies the end of a line. Other commonly used anchors are:    
\A&nbsp; &nbsp; &nbsp;  the start of a string    
\Z&nbsp; &nbsp; &nbsp;  the end of a string    
\b&nbsp; &nbsp; &nbsp;  the start or end of a word    
\B&nbsp; &nbsp; &nbsp;  some place other than the start or end of a word.  
The difference between \A(\Z) and ^($)? Well, a string may contain several lines.  


## Repetitions specifier  

\*  &nbsp; &nbsp; &nbsp; any times, zero times or more.  
\+  &nbsp; &nbsp; &nbsp; one time or more  
?   &nbsp; &nbsp; &nbsp; zero or more times.  
{m, n}   &nbsp; &nbsp; &nbsp;at lease m times and at least n times  
{m, }    &nbsp; &nbsp; &nbsp;at lease m times  
{,n}  &nbsp; &nbsp; &nbsp; at most n times.  

## Parenthesise(groups)  

Groups are used for two purpose.   
- Used to define a unit  
- Used to store matching results so that you can refer to them later in the same regular expression  

### Use group to define a unit.  

Normally, the repetition characters are applied to the single character in front of it.  
`/hello*/`  will match “hell”, “hello”, “helloo”, ....  
the \* is applied to the “o” only. How can you specify the repetition to be applied to the whole word “hello”? That where groups comes in:  
`/(hello)*/`  will match “hello”, “helloholle”, “hellohellohello”, ...  
Another example to use group to define a unit is when using the “|” operator.  
We can use “|” as “or” in regular expression. For example, I want to match the word “Captain” or “Natasha”. Then I can write the regular expression as:  
`/Captain|Natasha/`  
Yet the problem with `|` is that it is very greedy. It will regard all the left-side part as a unit and then all the right-side part as a unit. Say you want to match “Hello Captain” or “Hello Natasha”. You may be tempted to write the regular expression as:  
`/Hello Captain|Natasha/`  
Yet this will not work. Rather, this regular expression will match “Hello Captain” or “Natasha”.  If you want to achieve the expected result, you can use `()` in it:  
`/Hello (Captain|Natasha)/`  

### Used group as a storage that can be referenced later  

Say you want to match a *Palindromes* string consists of 5 characters, like “abcba”. You can write the regular expression as such:  
`/(\w)(\w)(\w)\2\1/`  

The `\1` in the regular expression means the matching result of the first part that was enclosed by `()` in the regular expression. And \2 means the second.   
Regular expression count the occurrence sequence of `()` and put the matching result in \1, \2, etc. So that you can refer to these matching result later in the same regular expression.   
Most programming languages also have ways to reference these parts of matching result **after** the match. In Ruby, the `match` method of regular expression will return a MatchData object.  

```Ruby  
md = /(\d\d):(\d\d)([aApP][mM])/.match('12:50am')  
md[1]   #=> 12  
md[2]   #=> 50  
md[3]   #=> am  
md[0]   #=> 12:50am  (the whole matching result)  
```  

In some *find and replace* tools, you can also use \1, \2 etc in the *replacing* regular expression to refer to the the matching group in the *find* process.  
Say you have a file containing a list of some names:  

```
Steve Rogers  
Natasha Romanova  
Thor Odinson  
Bruce Banner  
...  
```

And you want to change the display order to   

```
Rogers, Steve  
Romanova, Natasha  
Odinson, Thor  
Banner, Bruce  
...  
```

You can write the following regular expression:  
`%s/\v(\w+) (\w+)/\2, \1/g`  
The `\v` is there so that we don’t have to escape the `(` and `)` characters, otherwise you have to write as:  `%s/\(\w+\) \(\w+\)/\2, \1/g`. It just the way vim works in its `find and replace` tool, rather then regular expression’s behaviour.  
