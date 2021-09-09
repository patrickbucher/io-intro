# Io

[Io](https://iolanguage.org) ist eine auf Prototypen basierte, objektorientierte
Programmiersprache.  Objekte basieren nicht auf einer Klasse, sondern sind
Kopien anderer Objekte.  Man interagiert mit Objekten, indem man ihnen
Nachrichten schickt (_Messaging_).

Für Alan Kay, der die objektorientierte Programmierung erfand, war Messaging die
zentrale Idee, nicht Klassen, Vererbung, Interfaces usw.

## Erste Schritte

Io kann mit dem Package-Manager des Betriebssystems oder via Download von der
[Webseite (binaries)](https://iolanguage.org/binaries.html) installiert werden.

Mit dem Befehl `io` startet man eine interaktive Sitzung:

    $ io
    Io 2017.09.06
    Io> 

In Io ist alles ein Objekt, und jedem Objekt können bestimmte Nachrichten
geschickt werden.

Hier wird dem String `"Hello, World!"` die Nachricht `print` geschickt:

    Io> "Hello, World!" print
    Hello, World!==> Hello, World!

Der _Empfänger_ der Nachricht steht links, die Nachricht steht rechts.

Speichert man folgendes Programm in einer Datei namens `hello.io` ab:

    "Hello, World!" println

Kann man es folgendermassen ausführen:

    $ io hello.io
    Hello, World!

## Objekte

Jedes Objekt wird vom ursprünglichen `Object` geklont:

    > Person := Object clone
    ==>  Person_0x559cbf4c9250:
      type             = "Person"

Mit dem Operator `:=` findet die initiale Zuweisung statt.

Alle Objekte werden in einem Namensraum namens `Lobby` abgelegt:

    > Lobby
    ==>  Object_0x559cbf19c770:
      Lobby            = Object_0x559cbf19c770
      Person           = Person_0x559cbf4c9250
      ...

### Slots

Statt Eigenschaften gibt es _Slots_, welche folgendermassen definiert werden
können:

    > Person name := "Patrick"
    ==> Patrick

Für die initiale Zuweisung muss `:=` verwendet werden, für spätere
Überschreibungen genügt `=`:

    > Person name = "Werner"
    ==> Werner

Die Slots können mit der Message `slotNames` angezeigt werden:

    > Person slotNames
    list(type, name)

### Typen

Objekte, die mit einem Grossbuchstaben anfangen, sind _Typen_ und haben darum
einen `type`-Slot.

Slots können abgefragt werden, indem man ihren Namen dem Objekt als Message
schickt:

    > Person type
    ==> Person

    > Person name
    ==> Werner

Objekte, die mit einem Kleinbuchstaben anfangen, sind einfache Objekte ohne
`type`-Slot:

    > peter := Person clone
    > peter name := "Peter"
    > peter slotNames
    ==> list(name)

Fragt man den `type`-Slot dennoch ab, wird die Nachricht an das Ursprungsobjekt
weiterdelegiert:

    > peter type
    ==> Person

Diese Kette kann beliebig lange sein:

    > Vehicle := Object clone
    > Ferrari := Vehicle clone
    > Ferrari country := "Italy"
    > FerrariEnzo := Ferrari clone
    > FerrariEnzo horsepower := 660
    > myFerrariEnzo := FerrariEnzo clone
    > myFerrariEnzo owner := "Patrick"
    > myFerrariEnzo owner
    ==> Patrick
    > myFerrariEnzo horsepower
    ==> 660
    > myFerrariEnzo country
    ==> Italy
    > myFerrariEnzo type
    ==> FerrariEnzo

Mit der `proto`-Message kann man anschauen, auf welchem Prototyp ein Objekt
basiert:

    > myFerrariEnzo proto
    ==>  FerrariEnzo_0x559cbf4278c0:
      horsepower       = 660
      type             = "FerrariEnzo"
    > FerrariEnzo proto
    ==>  Ferrari_0x559cbf44b1e0:
      country          = "Italy"
      type             = "Ferrari"
    > Ferrari proto
    ==>  Vehicle_0x559cbf3e64f0:
      drive            = method(...)
      type             = "Vehicle"

Man sieht auch, auf welcher Ebene welche Slots definiert sind.

### Methoden

Eine Methode wird mit der `method`-Message definiert:

    > Vehicle drive := method("Brum, Brum!" print)

Sie wird aufgerufen, indem man dem Objekt den Methodennamen als Message schickt:

    > myFerrariEnzo drive
    Brum, Brum!==> Brum, Brum!

Die Definition kann man sich anschauen, indem man die `getSlot`-Message mit
dem Slotnamen verwendet:

    > myFerrariEnzo getSlot("drive")
    ==> method(
        "Brum, Brum!" print
    )

### Singleton

Ein Objekt, das einmalig ist, bezeichnet man als Singleton. Mit Java oder C#
muss man bei der Implementierung auf einige Sachen achten. Mit Io ist diese ganz
einfach:

    > TheOneAndOnly := Object clone
    > TheOneAndOnly clone := TheOneAndOnly

Die `clone`-Message des Objekts `TheOneAndOnly` wird überschrieben, sodass sie
immer wieder das gleiche Objekt zurückgibt:

    > TheOneAndOnly name := "me"
    ==> me
    > alice := TheOneAndOnly clone
    > bob := TheOneAndOnly clone
    > alice == bob
    ==> true

## Collections

Neben Objekten hat Io zwei wichtige Datenstrukturen: Listen und Maps.

### Listen

Eine Liste kann erzeugt werden, indem man `List` klont:

    > names := List clone
    ==> list()

Man kann auch die `list`-Message verwenden und initiale Elemente definieren:

    > names := list("Alice", "Bob", "Mallory")
    ==> list(Alice, Bob, Mallory)

Elemente können mittels `append` am Ende angefügt werden:

    > names append("Zorro")
    ==> list(Alice, Bob, Mallory, Zorro)

Mit `prepend` kann ein Element hinten angefügt werden:

    > names prepend("Aaron")
    list(Aaron, Alice, Bob, Mallory, Zorro)

Die Anzahl der Elemente findet man mittels `size` heraus:

    > names size
    ==> 4

Mit `isEmpty` findet man heraus, ob eine Liste leer ist:

    > names isEmpty
    false
    > list() isEmpty
    true

Mit einem null-basierten Index kann man auf bestimmte Elemente zugreifen:

    > names at(0)
    ==> Alice
    > names at(3)
    ==> Zorro

Mittels `pop` kann das letzte Element entfernt werden:

    > names pop
    ==> Zorro
    > names
    ==> list(Alice, Bob, Mallory)

Listen, die Zahlen enthalten, unterstützen verschiedene arithmetische Methoden:

    > list(1, 1, 2, 3, 5, 8) sum
    ==> 20
    > list(1, 1, 2, 3, 5, 8) average
    ==> 3.3333333333333335

### Maps

Eine neue Map wird erstellt, indem man sie von der ursprünglichen `Map` klont:

    > dog := Map clone

Elemente können mit `at` abgefragt und mit `atPut` gesetzt werden:

    > dog atPut("name", "Woofie")
    > dog atPut("age", 3)
    > dog at("name")
    ==> Woofie
    > dog at("age")
    ==> 3

Die Grösse und Keys der Map können mit den Messages `size` und `keys` abgefragt
werden:

    > dog size
    2
    > dog keys
    ==> list(name, age)

Maps können in Objekte und (verschachtelte) Listen umgewandelt werden:

    > dog asObject
    ==>  Object_0x559cc01a2f60:
      age              = 3
      name             = "Woofie"
    > dog asList
    ==> list(list(name, Woofie), list(age, 3))

## Logik und Vergleiche

Für Vergleiche stehen verschiedene Operatoren zur Verfügung:

    > 3 > 5
    ==> false
    > 2 < 7
    ==> true
    > 3 >= 4
    ==> false
    > 5 <= 4
    ==> false

Logische Verknüpfungen macht man folgendermassen:

    > true and false
    ==> false
    > false or true
    ==> true
    > true and not false
    ==> false
    > true and 0
    ==> true
    > "" and 1
    ==> true 
