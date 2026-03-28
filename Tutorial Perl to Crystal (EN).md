# Crystal for Perl programmers

Crystal is a compiled, statically typed language with a Ruby-inspired syntax. For a Perl programmer, many concepts will feel familiar — hashes, regex, blocks, STDIN — but the execution model is radically different: Crystal produces a native binary, not an interpreter.

---

## Table of Contents

1. [Compilation and execution](#1-compilation-and-execution)
2. [Variables and types](#2-variables-and-types)
3. [Strings and interpolation](#3-strings-and-interpolation)
4. [Numbers and operators](#4-numbers-and-operators)
5. [Arrays](#5-arrays)
6. [Hashes](#6-hashes)
7. [Conditionals](#7-conditionals)
8. [Loops and iteration](#8-loops-and-iteration)
9. [Blocks — `do |var|`](#9-blocks--do-var)
10. [Regular expressions](#10-regular-expressions)
11. [Functions](#11-functions)
12. [I/O: files and STDIN](#12-io-files-and-stdin)
13. [Encoding](#13-encoding)
14. [Nil and handling the absence of value](#14-nil-and-handling-the-absence-of-value)
15. [Classes and objects](#15-classes-and-objects)
16. [Key differences to keep in mind](#16-key-differences-to-keep-in-mind)

---

## 1. Compilation and execution

In Perl you run directly:

```perl
perl my_script.pl
```

In Crystal you compile first, then run:

```sh
crystal build my_script.cr   # produces the binary my_script
./my_script
```

Or in a single step (slower, useful during development):

```sh
crystal run my_script.cr
```

The compiler performs type inference: in most cases you do not need to declare types explicitly, but the compiler still checks them at compile-time. If there is a type error, the program will not compile.

---

## 2. Variables and types

### Perl
```perl
my $name = "Mario";
my $age  = 42;
my $pi   = 3.14;
```

### Crystal
```crystal
name = "Mario"
age  = 42
pi   = 3.14
```

- No `my`, `our`, `local` — scope is lexical by default
- No `$`, `@`, `%` sigils — a variable's type is inferred from context
- The type is inferred from the assignment: `name` is `String`, `age` is `Int32`, `pi` is `Float64`

To declare the type explicitly:

```crystal
age : Int32 = 42
```

Main primitive types:

| Perl | Crystal |
|------|---------|
| string | `String` |
| integer | `Int32`, `Int64` |
| float | `Float64` |
| undef | `nil` |
| array `@a` | `Array(T)` |
| hash `%h` | `Hash(K, V)` |

---

## 3. Strings and interpolation

### Perl
```perl
my $name = "Mario";
print "Hello, $name!\n";
print "Hello, ${name}!\n";
```

### Crystal
```crystal
name = "Mario"
puts "Hello, #{name}!"
puts "Hello, #{"mario".upcase}!"  # any expression can go inside
```

- Interpolation uses `#{ }` instead of `${ }`
- Any Crystal expression can be written inside `#{ }`
- `puts` automatically appends a newline (like `say` in Perl 5.10+)
- `print` does not append a newline

Single-quoted strings do not interpolate, just like in Perl:

```crystal
puts 'Hello, #{name}'  # prints literally: Hello, #{name}
```

Useful string methods:

```crystal
s = "  Hello World  "
s.upcase        # "  HELLO WORLD  "
s.downcase      # "  hello world  "
s.strip         # "Hello World"      (equivalent to s =~ s/^\s+|\s+$//g)
s.size          # 14
s.includes?("World")  # true
s.split(" ")    # ["", "", "Hello", "World", "", ""]
s.split         # ["Hello", "World"]  (split on whitespace, discards empty strings)
```

---

## 4. Numbers and operators

Arithmetic operators are identical to Perl: `+`, `-`, `*`, `/`, `%`, `**`.

Differences:

```crystal
7 / 2    # => 3      (integer division if both operands are Int)
7.0 / 2  # => 3.5    (float if at least one operand is float)
7 // 2   # does not exist; use (7 / 2).to_i or simply 7 / 2
```

In Perl `7 / 2` gives `3.5`. In Crystal the result depends on the types of the operands.

---

## 5. Arrays

### Perl
```perl
my @fruits = ("apple", "pear", "banana");
push    @fruits, "kiwi";    # append to end
my $u = pop     @fruits;    # remove and return the last element
my $p = shift   @fruits;    # remove and return the first element
unshift @fruits, "strawberry"; # insert at front
my $first = $fruits[0];
my $n     = scalar @fruits;

foreach my $f (@fruits) {
  print "$f\n";
}
```

### Crystal
```crystal
fruits = ["apple", "pear", "banana"]
fruits << "kiwi"           # push: append to end
fruits.push("kiwi")        # equivalent to <<
fruits.pop                  # remove and return the last element
fruits.shift                # remove and return the first element
fruits.unshift("strawberry") # insert at front
first = fruits[0]
n     = fruits.size

fruits.each do |f|
  puts f
end
```

- `<<` is the append operator (equivalent to `push`)
- `fruits[-1]` works like in Perl: accesses the last element
- The array is typed: `["apple", "pear"]` is `Array(String)` and cannot contain integers

Mixed-type arrays must be declared explicitly:

```crystal
mixed = ["hello", 42, 3.14] of String | Int32 | Float64
```

Common methods:

```crystal
a = [3, 1, 4, 1, 5]
a.sort         # [1, 1, 3, 4, 5]  (does not modify a)
a.sort!        # modifies a in-place  (! indicates methods that modify the object)
a.reverse      # [5, 1, 4, 1, 3]
a.uniq         # [3, 1, 4, 5]
a.size         # 5
a.first        # 3
a.last         # 5
a.join(", ")   # "3, 1, 4, 1, 5"
a.map { |x| x * 2 }    # [6, 2, 8, 2, 10]
a.select { |x| x > 2 } # [3, 4, 5]
a.reject { |x| x > 2 } # [1, 1]
a.sum          # 14
```

---

## 6. Hashes

### Perl
```perl
my %age = ("Mario" => 42, "Luigi" => 35);
$age{"Mario"} = 43;
print $age{"Mario"}, "\n";

foreach my $name (keys %age) {
  print "$name: $age{$name}\n";
}
```

### Crystal
```crystal
age = {"Mario" => 42, "Luigi" => 35}
age["Mario"] = 43
puts age["Mario"]

age.each do |name, value|
  puts "#{name}: #{value}"
end
```

- The `=>` syntax is identical to Perl
- Access with `[]` instead of `{}`
- `age["Unknown"]` raises an exception if the key does not exist (unlike Perl, which returns `undef`)
- For safe access: `age["Unknown"]?` returns `nil` instead of raising an exception

Hash with a default value:

```crystal
count = Hash(String, Int32).new(0)
count["word"] += 1  # works even on first occurrence
```

Equivalent to the Perl pattern:
```perl
$count{$word} //= 0;
$count{$word}++;
```

Common methods:

```crystal
h = {"a" => 1, "b" => 2, "c" => 3}
h.keys          # ["a", "b", "c"]
h.values        # [1, 2, 3]
h.size          # 3
h.has_key?("a") # true
h.delete("b")   # removes the key
h.merge({"d" => 4})  # new merged hash
```

---

## 7. Conditionals

### Perl
```perl
if ($x > 0) {
  print "positive\n";
} elsif ($x < 0) {
  print "negative\n";
} else {
  print "zero\n";
}

print "ok\n" if $x > 0;  # postfix form
```

### Crystal
```crystal
if x > 0
  puts "positive"
elsif x < 0
  puts "negative"
else
  puts "zero"
end

puts "ok" if x > 0  # postfix form identical to Perl
```

- No mandatory parentheses around the condition
- `elsif` (not `elseif` nor `else if`)
- `end` instead of `}`
- The postfix form `statement if condition` works just like in Perl

`unless` exists as in Perl:

```crystal
puts "non-zero" unless x == 0
```

`if` is an expression and returns a value:

```crystal
message = if x > 0
  "positive"
else
  "not positive"
end
```

---

## 8. Loops and iteration

### Perl
```perl
# loop with index
for my $i (0..9) {
  print "$i\n";
}

# while
while ($line = <>) {
  chomp $line;
  print $line, "\n";
}

# foreach
foreach my $elem (@array) {
  print "$elem\n";
}
```

### Crystal
```crystal
# loop with index
(0..9).each do |i|
  puts i
end

# or alternatively
10.times do |i|
  puts i
end

# while
while line = gets
  puts line
end

# each on array
array.each do |elem|
  puts elem
end
```

The range `(0..9)` includes 9. `(0...9)` excludes 9 (three dots = exclusive):

```crystal
(0..9).to_a   # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
(0...9).to_a  # [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

Infinite loop:

```crystal
loop do
  # ...
  break if condition
end
```

`next` and `break` work like in Perl (`next` = skip iteration, `break` = exit the loop).

---

## 9. Blocks — `do |var|`

Blocks are the most pervasive feature of Crystal (inherited from Ruby). A block is a piece of anonymous code passed to a function.

```crystal
array.each do |element|
  puts element
end
```

The `each` function calls the block once for each element, passing the element as the `element` parameter.

The short form with curly braces is equivalent:

```crystal
array.each { |element| puts element }
```

Convention: use `do...end` for multi-line blocks, `{ }` for single-line blocks.

A block can receive multiple parameters:

```crystal
hash.each do |key, value|
  puts "#{key} => #{value}"
end
```

Blocks are also the foundation of `map`, `select`, and `reject`:

```crystal
squares = [1, 2, 3, 4].map { |x| x ** 2 }    # [1, 4, 9, 16]
evens   = [1, 2, 3, 4].select { |x| x.even? } # [2, 4]
```

The closest Perl equivalents are `map` and `grep`:

```perl
my @squares = map { $_ ** 2 } (1, 2, 3, 4);
my @evens   = grep { $_ % 2 == 0 } (1, 2, 3, 4);
```

---

## 10. Regular expressions

Regex in Crystal uses the same PCRE syntax as Perl.

### Match

```perl
# Perl
if ($s =~ /(\d+)/) {
  print "found: $1\n";
}
```

```crystal
# Crystal
if m = s.match(/(\d+)/)
  puts "found: #{m[1]}"
end
```

- `match` returns a `Regex::MatchData` object or `nil`
- Groups are accessed with `m[1]`, `m[2]`, etc. (like `$1`, `$2` in Perl)
- `m[0]` is the entire match (like `$&` in Perl)

### Global match (all occurrences)

```perl
# Perl
while ($s =~ /(\w+)/g) {
  print "$1\n";
}
```

```crystal
# Crystal
s.scan(/(\w+)/) do |m|
  puts m[1]
end
```

### Substitution

```perl
# Perl
$s =~ s/foo/bar/;       # first occurrence
$s =~ s/foo/bar/g;      # all occurrences
```

```crystal
# Crystal
s = s.sub(/foo/, "bar")   # first occurrence  (returns a new string)
s = s.gsub(/foo/, "bar")  # all occurrences
```

In Crystal strings are immutable: `sub` and `gsub` return a new string rather than modifying the original.

### Simple check

```crystal
s.matches?(/^\d+$/)   # true if s consists only of digits
```

---

## 11. Functions

### Perl
```perl
sub sum {
  my ($a, $b) = @_;
  return $a + $b;
}

my $result = sum(3, 4);
```

### Crystal
```crystal
def sum(a, b)
  a + b  # the last expression is the implicit return value
end

result = sum(3, 4)
```

- `def` instead of `sub`
- Parameters are declared directly, not via `@_`
- `return` is optional: the last expression is the returned value
- Parameter and return types are optional but can be declared:

```crystal
def sum(a : Int32, b : Int32) : Int32
  a + b
end
```

Parameters with default values:

```crystal
def greet(name, greeting = "Hello")
  puts "#{greeting}, #{name}!"
end

greet("Mario")                # Hello, Mario!
greet("Mario", "Good morning") # Good morning, Mario!
```

---

## 12. I/O: files and STDIN

### Reading from STDIN line by line

```perl
while ($line = <STDIN>) {
  chomp $line;
  print $line;
}
```

```crystal
STDIN.each_line do |line|  # newline is already stripped
  print line
end
```

In Crystal `each_line` automatically strips the newline (equivalent to automatic `chomp`).

### Reading a file

```perl
open(my $fh, "<", "file.txt") or die $!;
while (my $line = <$fh>) {
  chomp $line;
  print $line;
}
close($fh);
```

```crystal
File.each_line("file.txt") do |line|
  print line
end
```

Or read everything into memory:

```crystal
content = File.read("file.txt")       # string
lines   = File.read_lines("file.txt") # array of strings
```

### Writing to a file

```perl
open(my $fh, ">", "output.txt") or die $!;
print $fh "line 1\n";
close($fh);
```

```crystal
File.write("output.txt", "line 1\n")

# or for incremental writing:
File.open("output.txt", "w") do |f|
  f.puts "line 1"
  f.puts "line 2"
end  # file is closed automatically
```

### STDERR

```perl
print STDERR "error!\n";
```

```crystal
STDERR.puts "error!"
```

---

## 13. Encoding

Perl handles raw bytes by default and leaves encoding responsibility to the programmer. Crystal, on the other hand, requires all strings to be valid UTF-8.

To read ANSI (Windows-1252) files:

### Perl
```perl
use Encode;
binmode(STDIN, ":encoding(Windows-1252)");
while (my $line = <STDIN>) {
  chomp $line;
  # $line is now a Perl internal string (logical UTF-8)
  print $line, "\n";
}
```

### Crystal
```crystal
STDIN.set_encoding("Windows-1252")
STDIN.each_line do |line|
  # line has already been converted to UTF-8
end
```

In Perl the conversion is activated via `binmode` with an encoding layer (requires `use Encode`). Crystal instead uses the `set_encoding` method on the I/O channel.

To detect encoding at runtime, a dedicated routine must be implemented in both languages.

---

## 14. Nil and handling the absence of value

In Perl the absent value is `undef`, which propagates silently. In Crystal it is `nil`, but the compiler forces you to handle it explicitly.

```crystal
h = {"a" => 1}
value = h["b"]?   # returns Int32 | Nil

# the compiler will not let you use value as Int32 without checking first
if value
  puts value + 1  # here the compiler knows value is not nil
end

# or with a default value
v = h["b"]? || 0
```

This eliminates entire classes of bugs present in Perl caused by unchecked `undef`.

---

## 15. Classes and objects

Crystal is object-oriented. Even primitive types are objects with methods:

```crystal
42.to_s       # "42"
3.14.round    # 3
"hello".size  # 5
[1,2,3].sum   # 6
```

Defining a class:

```crystal
class Person
  getter name : String
  getter age  : Int32

  def initialize(@name, @age)
  end

  def greet
    puts "Hi, I'm #{@name} and I'm #{@age} years old"
  end
end

p = Person.new("Mario", 42)
p.greet
puts p.name
```

- `@name` is an instance variable (as in Ruby)
- `getter` automatically generates a read accessor method
- `initialize` is the constructor

---

## 16. Key differences to keep in mind

| Concept | Perl | Crystal |
|---------|------|---------|
| Execution | Interpreted | Compiled (native binary) |
| Types | Dynamic, implicit | Static, inferred at compile-time |
| Variable sigils | `$`, `@`, `%` required | None |
| Hash access | `$h{key}` | `h[key]` |
| Regex match groups | `$1`, `$2` | `m[1]`, `m[2]` |
| Absent value | `undef` | `nil` (enforced by compiler) |
| String mutability | Mutable | Immutable (`sub`/`gsub` return a new string) |
| Integer division | `int(7/2)` = 3 | `7 / 2` = 3 (automatic if Int) |
| Chomp | Explicit | Automatic in `each_line` |
| Encoding | Raw bytes by default | UTF-8 required (explicit conversion) |
| Block end | `}` | `end` |
| Modulo | `%` | `%` |
| Print with newline | `say` (5.10+) | `puts` |
| Concatenation | `.` | `+` or interpolation |
