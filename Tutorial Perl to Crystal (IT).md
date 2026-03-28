# Crystal per programmatori Perl

Crystal è un linguaggio compilato, staticamente tipizzato, con una sintassi ispirata a Ruby. Per un programmatore Perl, molti concetti sono familiari — hash, regex, blocchi, STDIN — ma il modello di esecuzione è radicalmente diverso: Crystal produce un binario nativo, non un interprete.

---

## Indice

1. [Compilazione ed esecuzione](#1-compilazione-ed-esecuzione)
2. [Variabili e tipi](#2-variabili-e-tipi)
3. [Stringhe e interpolazione](#3-stringhe-e-interpolazione)
4. [Numeri e operatori](#4-numeri-e-operatori)
5. [Array](#5-array)
6. [Hash](#6-hash)
7. [Condizionali](#7-condizionali)
8. [Loop e iterazione](#8-loop-e-iterazione)
9. [Blocchi — `do |var|`](#9-blocchi--do-var)
10. [Espressioni regolari](#10-espressioni-regolari)
11. [Funzioni](#11-funzioni)
12. [I/O: file e STDIN](#12-io-file-e-stdin)
13. [Encoding](#13-encoding)
14. [Nil e gestione dell'assenza di valore](#14-nil-e-gestione-dellassenza-di-valore)
15. [Classi e oggetti](#15-classi-e-oggetti)
16. [Differenze chiave da tenere a mente](#16-differenze-chiave-da-tenere-a-mente)

---

## 1. Compilazione ed esecuzione

In Perl si esegue direttamente:

```perl
perl mio_script.pl
```

In Crystal si compila prima, poi si esegue:

```sh
crystal build mio_script.cr   # produce il binario mio_script
./mio_script
```

Oppure in un solo passo (più lento, utile in sviluppo):

```sh
crystal run mio_script.cr
```

Il compilatore esegue l'inferenza di tipo: nella maggior parte dei casi non è necessario dichiarare i tipi esplicitamente, ma il compilatore li verifica comunque a compile-time. Se c'è un errore di tipo, il programma non compila.

---

## 2. Variabili e tipi

### Perl
```perl
my $nome = "Mario";
my $eta  = 42;
my $pi   = 3.14;
```

### Crystal
```crystal
nome = "Mario"
eta  = 42
pi   = 3.14
```

- Niente `my`, `our`, `local` — lo scope è lessicale per default
- Niente `$`, `@`, `%` come sigilli — il tipo di una variabile si deduce dal contesto
- Il tipo viene inferito dall'assegnazione: `nome` è `String`, `eta` è `Int32`, `pi` è `Float64`

Per dichiarare il tipo esplicitamente:

```crystal
eta : Int32 = 42
```

I tipi primitivi principali:

| Perl | Crystal |
|------|---------|
| stringa | `String` |
| intero | `Int32`, `Int64` |
| float | `Float64` |
| undef | `nil` |
| array `@a` | `Array(T)` |
| hash `%h` | `Hash(K, V)` |

---

## 3. Stringhe e interpolazione

### Perl
```perl
my $nome = "Mario";
print "Ciao, $nome!\n";
print "Ciao, ${nome}!\n";
```

### Crystal
```crystal
nome = "Mario"
puts "Ciao, #{nome}!"
puts "Ciao, #{"mario".upcase}!"  # si può mettere qualsiasi espressione
```

- L'interpolazione usa `#{ }` invece di `${ }`
- All'interno di `#{ }` si può scrivere qualsiasi espressione Crystal
- `puts` aggiunge automaticamente il newline (come `say` in Perl 5.10+)
- `print` non aggiunge il newline

Le stringhe con apici singoli non interpolano, come in Perl:

```crystal
puts 'Ciao, #{nome}'  # stampa letteralmente: Ciao, #{nome}
```

Metodi utili sulle stringhe:

```crystal
s = "  Ciao Mondo  "
s.upcase        # "  CIAO MONDO  "
s.downcase      # "  ciao mondo  "
s.strip         # "Ciao Mondo"      (equivale a s =~ s/^\s+|\s+$//g)
s.size          # 14
s.includes?("Mondo")  # true
s.split(" ")    # ["", "", "Ciao", "Mondo", "", ""]
s.split         # ["Ciao", "Mondo"]  (split su whitespace, scarta vuoti)
```

---

## 4. Numeri e operatori

Gli operatori aritmetici sono identici a Perl: `+`, `-`, `*`, `/`, `%`, `**`.

Differenze:

```crystal
7 / 2    # => 3      (divisione intera se entrambi Int)
7.0 / 2  # => 3.5    (float se almeno uno è float)
7 // 2   # non esiste, usare (7 / 2).to_i oppure semplicemente 7 / 2
```

In Perl `7 / 2` dà `3.5`. In Crystal dipende dal tipo degli operandi.

---

## 5. Array

### Perl
```perl
my @frutti = ("mela", "pera", "banana");
push    @frutti, "kiwi";   # aggiunge in fondo
my $u = pop     @frutti;   # rimuove e restituisce l'ultimo
my $p = shift   @frutti;   # rimuove e restituisce il primo
unshift @frutti, "fragola"; # inserisce in testa
my $primo = $frutti[0];
my $n     = scalar @frutti;

foreach my $f (@frutti) {
  print "$f\n";
}
```

### Crystal
```crystal
frutti = ["mela", "pera", "banana"]
frutti << "kiwi"          # push: aggiunge in fondo
frutti.push("kiwi")       # equivalente a <<
frutti.pop                 # rimuove e restituisce l'ultimo elemento
frutti.shift               # rimuove e restituisce il primo elemento
frutti.unshift("fragola")  # inserisce in testa
primo = frutti[0]
n     = frutti.size

frutti.each do |f|
  puts f
end
```

- `<<` è l'operatore di append (equivale a `push`)
- `frutti[-1]` funziona come in Perl: accede all'ultimo elemento
- L'array è tipizzato: `["mela", "pera"]` è `Array(String)` e non può contenere interi

Array con tipi misti si dichiarano esplicitamente:

```crystal
misto = ["ciao", 42, 3.14] of String | Int32 | Float64
```

Metodi comuni:

```crystal
a = [3, 1, 4, 1, 5]
a.sort         # [1, 1, 3, 4, 5]  (non modifica a)
a.sort!        # modifica a in-place  (il ! indica metodi che modificano l'oggetto)
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

## 6. Hash

### Perl
```perl
my %eta = ("Mario" => 42, "Luigi" => 35);
$eta{"Mario"} = 43;
print $eta{"Mario"}, "\n";

foreach my $nome (keys %eta) {
  print "$nome: $eta{$nome}\n";
}
```

### Crystal
```crystal
eta = {"Mario" => 42, "Luigi" => 35}
eta["Mario"] = 43
puts eta["Mario"]

eta.each do |nome, valore|
  puts "#{nome}: #{valore}"
end
```

- La sintassi `=>` è identica a Perl
- Accesso con `[]` invece di `{}`
- `eta["Pippo"]` lancia un'eccezione se la chiave non esiste (diverso da Perl che restituisce `undef`)
- Per accesso sicuro: `eta["Pippo"]?` restituisce `nil` invece di eccezione

Hash con valore di default:

```crystal
conteggio = Hash(String, Int32).new(0)
conteggio["parola"] += 1  # funziona anche alla prima occorrenza
```

Equivale al pattern Perl:
```perl
$conteggio{$parola} //= 0;
$conteggio{$parola}++;
```

Metodi comuni:

```crystal
h = {"a" => 1, "b" => 2, "c" => 3}
h.keys          # ["a", "b", "c"]
h.values        # [1, 2, 3]
h.size          # 3
h.has_key?("a") # true
h.delete("b")   # rimuove la chiave
h.merge({"d" => 4})  # nuovo hash unito
```

---

## 7. Condizionali

### Perl
```perl
if ($x > 0) {
  print "positivo\n";
} elsif ($x < 0) {
  print "negativo\n";
} else {
  print "zero\n";
}

print "ok\n" if $x > 0;  # postfisso
```

### Crystal
```crystal
if x > 0
  puts "positivo"
elsif x < 0
  puts "negativo"
else
  puts "zero"
end

puts "ok" if x > 0  # postfisso identico a Perl
```

- Niente parentesi obbligatorie attorno alla condizione
- `elsif` (non `elseif` né `else if`)
- `end` al posto di `}`
- La forma postfissa `istruzione if condizione` funziona come in Perl

`unless` esiste come in Perl:

```crystal
puts "non zero" unless x == 0
```

L'`if` è un'espressione e restituisce un valore:

```crystal
messaggio = if x > 0
  "positivo"
else
  "non positivo"
end
```

---

## 8. Loop e iterazione

### Perl
```perl
# loop con indice
for my $i (0..9) {
  print "$i\n";
}

# while
while ($riga = <>) {
  chomp $riga;
  print $riga, "\n";
}

# foreach
foreach my $elem (@array) {
  print "$elem\n";
}
```

### Crystal
```crystal
# loop con indice
(0..9).each do |i|
  puts i
end

# oppure
10.times do |i|
  puts i
end

# while
while line = gets
  puts line
end

# each su array
array.each do |elem|
  puts elem
end
```

Il range `(0..9)` include 9. `(0...9)` esclude 9 (tre punti = esclusivo):

```crystal
(0..9).to_a   # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
(0...9).to_a  # [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

Loop infinito:

```crystal
loop do
  # ...
  break if condizione
end
```

`next` e `break` funzionano come in Perl (`next` = salta iterazione, `break` = esci dal loop).

---

## 9. Blocchi — `do |var|`

I blocchi sono la caratteristica più pervasiva di Crystal (ereditata da Ruby). Un blocco è un pezzo di codice anonimo che viene passato a una funzione.

```crystal
array.each do |elemento|
  puts elemento
end
```

La funzione `each` chiama il blocco una volta per ogni elemento, passando l'elemento come parametro `elemento`.

La forma breve con graffe è equivalente:

```crystal
array.each { |elemento| puts elemento }
```

Convenzione: `do...end` per blocchi multiriga, `{ }` per blocchi su una riga.

Un blocco può ricevere più parametri:

```crystal
hash.each do |chiave, valore|
  puts "#{chiave} => #{valore}"
end
```

I blocchi sono anche alla base di `map`, `select`, `reject`:

```crystal
quadrati = [1, 2, 3, 4].map { |x| x ** 2 }   # [1, 4, 9, 16]
pari     = [1, 2, 3, 4].select { |x| x.even? } # [2, 4]
```

In Perl l'equivalente più vicino sono `map` e `grep`:

```perl
my @quadrati = map { $_ ** 2 } (1, 2, 3, 4);
my @pari     = grep { $_ % 2 == 0 } (1, 2, 3, 4);
```

---

## 10. Espressioni regolari

Le regex in Crystal usano la stessa sintassi PCRE di Perl.

### Match

```perl
# Perl
if ($s =~ /(\d+)/) {
  print "trovato: $1\n";
}
```

```crystal
# Crystal
if m = s.match(/(\d+)/)
  puts "trovato: #{m[1]}"
end
```

- `match` restituisce un oggetto `Regex::MatchData` oppure `nil`
- I gruppi si accedono con `m[1]`, `m[2]`, ecc. (come `$1`, `$2` in Perl)
- `m[0]` è l'intero match (come `$&` in Perl)

### Match globale (tutte le occorrenze)

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

### Sostituzione

```perl
# Perl
$s =~ s/foo/bar/;       # prima occorrenza
$s =~ s/foo/bar/g;      # tutte le occorrenze
```

```crystal
# Crystal
s = s.sub(/foo/, "bar")   # prima occorrenza  (restituisce nuova stringa)
s = s.gsub(/foo/, "bar")  # tutte le occorrenze
```

In Crystal le stringhe sono immutabili: `sub` e `gsub` restituiscono una nuova stringa invece di modificare quella originale.

### Verifica semplice

```crystal
s.matches?(/^\d+$/)   # true se s è composta solo da cifre
```

---

## 11. Funzioni

### Perl
```perl
sub somma {
  my ($a, $b) = @_;
  return $a + $b;
}

my $risultato = somma(3, 4);
```

### Crystal
```crystal
def somma(a, b)
  a + b  # l'ultima espressione è il valore di ritorno implicito
end

risultato = somma(3, 4)
```

- `def` al posto di `sub`
- I parametri si dichiarano direttamente, non tramite `@_`
- `return` è opzionale: l'ultima espressione è il valore restituito
- I tipi dei parametri e del ritorno sono opzionali ma possono essere dichiarati:

```crystal
def somma(a : Int32, b : Int32) : Int32
  a + b
end
```

Parametri con valore di default:

```crystal
def saluta(nome, saluto = "Ciao")
  puts "#{saluto}, #{nome}!"
end

saluta("Mario")           # Ciao, Mario!
saluta("Mario", "Buongiorno")  # Buongiorno, Mario!
```

---

## 12. I/O: file e STDIN

### Leggere da STDIN riga per riga

```perl
while ($riga = <STDIN>) {
  chomp $riga;
  print $riga;
}
```

```crystal
STDIN.each_line do |line|  # il newline è già rimosso
  print line
end
```

In Crystal `each_line` rimuove automaticamente il newline (equivale a `chomp` automatico).

### Leggere un file

```perl
open(my $fh, "<", "file.txt") or die $!;
while (my $riga = <$fh>) {
  chomp $riga;
  print $riga;
}
close($fh);
```

```crystal
File.each_line("file.txt") do |line|
  print line
end
```

Oppure leggere tutto in memoria:

```crystal
contenuto = File.read("file.txt")       # stringa
righe     = File.read_lines("file.txt") # array di stringhe
```

### Scrivere su file

```perl
open(my $fh, ">", "output.txt") or die $!;
print $fh "riga 1\n";
close($fh);
```

```crystal
File.write("output.txt", "riga 1\n")

# oppure per scrittura incrementale:
File.open("output.txt", "w") do |f|
  f.puts "riga 1"
  f.puts "riga 2"
end  # il file viene chiuso automaticamente
```

### STDERR

```perl
print STDERR "errore!\n";
```

```crystal
STDERR.puts "errore!"
```

---

## 13. Encoding

Perl gestisce i byte grezzi per default e lascia al programmatore la responsabilità dell'encoding. Crystal invece richiede che tutte le stringhe siano UTF-8 valido.

Per leggere file ANSI (Windows-1252):

### Perl
```perl
use Encode;
binmode(STDIN, ":encoding(Windows-1252)");
while (my $riga = <STDIN>) {
  chomp $riga;
  # $riga è ora una stringa Perl interna (UTF-8 logico)
  print $riga, "\n";
}
```

### Crystal
```crystal
STDIN.set_encoding("Windows-1252")
STDIN.each_line do |line|
  # line è già convertita in UTF-8
end
```

In Perl la conversione si attiva tramite `binmode` con un livello di encoding (richiede `use Encode`). Crystal usa invece il metodo `set_encoding` sul canale di I/O.

Per rilevare l'encoding a runtime, in entrambi i linguaggi si deve implementare un'apposita routine.

---

## 14. Nil e gestione dell'assenza di valore

In Perl il valore assente è `undef`, che si propaga silenziosamente. In Crystal è `nil`, ma il compilatore impone di gestirlo esplicitamente.

```crystal
h = {"a" => 1}
valore = h["b"]?   # restituisce Int32 | Nil

# il compilatore non permette di usare valore come Int32 senza prima controllare
if valore
  puts valore + 1  # qui il compilatore sa che valore non è nil
end

# oppure con valore di default
v = h["b"]? || 0
```

Questo elimina interi classi di bug presenti in Perl dovuti a `undef` non controllato.

---

## 15. Classi e oggetti

Crystal è orientato agli oggetti. Anche i tipi primitivi sono oggetti con metodi:

```crystal
42.to_s       # "42"
3.14.round    # 3
"ciao".size   # 4
[1,2,3].sum   # 6
```

Definire una classe:

```crystal
class Persona
  getter nome : String
  getter eta  : Int32

  def initialize(@nome, @eta)
  end

  def saluta
    puts "Ciao, sono #{@nome} e ho #{@eta} anni"
  end
end

p = Persona.new("Mario", 42)
p.saluta
puts p.nome
```

- `@nome` è una variabile di istanza (come in Ruby)
- `getter` genera automaticamente il metodo di accesso in lettura
- `initialize` è il costruttore

---

## 16. Differenze chiave da tenere a mente

| Concetto | Perl | Crystal |
|----------|------|---------|
| Esecuzione | Interpretato | Compilato (binario nativo) |
| Tipi | Dinamici, impliciti | Statici, inferiti a compile-time |
| Sigilli variabili | `$`, `@`, `%` obbligatori | Nessuno |
| Hash access | `$h{chiave}` | `h[chiave]` |
| Regex match groups | `$1`, `$2` | `m[1]`, `m[2]` |
| Stringa assente | `undef` | `nil` (controllato dal compilatore) |
| Mutabilità stringhe | Mutabili | Immutabili (`sub`/`gsub` restituiscono nuova stringa) |
| Divisione intera | `int(7/2)` = 3 | `7 / 2` = 3 (automatica se Int) |
| Chomp | Esplicito | Automatico in `each_line` |
| Encoding | Byte grezzi per default | UTF-8 obbligatorio (conversione esplicita) |
| Fine blocco | `}` | `end` |
| Modulo | `%` | `%` |
| Stampa con newline | `say` (5.10+) | `puts` |
| Concatenazione | `.` | `+` oppure interpolazione |
