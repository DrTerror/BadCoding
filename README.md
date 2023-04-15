# Beispiele gutes/schlechtes Coding 

Es gibt einige generelle Programmierregeln (programming principles), die 
man sich zu Herzen nehmen sollte. Hier sind einige Merksprüche zu solchen
Prinzipien:

//**DRY** = **D**on't **R**epeat **Y**ourself//: auch bekannt unter 
//**DIE** = **D**uplication **I**s **E**vil// und soll dafür stehen, dass 
viele Codestellen, die das selbe bewirken, ausführen, ausgeben, ... schnell
zu Missverständnissen und schlechter Wartbarkeit führen können. 

//**KISS** = **K**eep **I**t **S**imple, **S**tupid//: 
Wie schon der alte Firmenslogan sagte: //"HEUFT, simply easy"// steht dieser
Merksatz für einfachen leserlichen Code, der schnell verstanden werden kann.

//**YAGNI** = **Y**ou **A**ren't **G**onna **N**eed **I**t//: 
Beim Schreiben von Code ist man oft gut bedient, wenn man viele Möglichkeiten
durchspielt und den Blick gen Zukunft richtet. Es macht aber wenig Sinn, kostet 
nur Zeit, macht Code schwer verständlich und führt oft zu unnötigen Fehlern, wenn
man Code einbaut, der dann "auf Halde liegt" und nicht gebraucht wird. Daher merkte: 
//"Wenn du es jetzt nicht brauchst, brauchst du es später erst recht nicht!"//

Es gibt noch viele weitere (//**SOILD**//, ...) aber die stehen eher für die OOP 
(objektorientierte Programmierung). Wer sich hierfür interessiert, kann [hier](https://www.google.com/search?q=programming+principles)
weiterlesen.


## C-Code-Beispiele 

### Verwirrende Zuweisungen/Vergleiche 

Kompakter (und damit auch komplizierter Code) ist nicht immer mit schnellem/
optimiertem Code gleich zu setzen! Dies hängt viel von den Compiler-Einstellungen
ab.

Nehmen wir diese beiden Vergleiche:

```c
if (a == b == c) { ... }
```
und
```c
if ( (a == b) && (a == c) ) { ... }
```

Sie sind nich identisch! Der erste Vergleich prüft ob ''a==b'' und liefert einen boolschen Wert zurück (''0'' oder ''1'' bzw. 
''true''/''false''). Dann wird dieser mit ''c'' verglichen. 
Erst der zweite Vergleich prüft auf die Gleichheit aller Variablen.

 
### Sichere Vergleiche 

Schnell hat man beim Tippen ein Zeichen vergessen, was nicht immer zu einem 
Compiler-Fehler führt. Damit ein Vergleichen mit einer Konstanten so nicht zu 
einer Zuweisung wird, kann man die Konstante an erste Position setzen. So würde
beim Vergessen eines ''='' mittels Compiler-Fehler gemeldet, dass hier etwas 
nicht stimmt.
```c
if (var == NULL) /* Es soll geprüft werden, ob var gleich 0 ist */

if (var = NULL)  /* Da ein ''=' vergessen wurde, wird aus dem Vergleich eine 
                  * Zuweisung. Dies bewirkt vorerst keinen Compiler-Fehler, ist
                  * aber inhaltlich falsch. */

if (NULL = var)  /* Einer Konstanten kann nichts zugewiesen werden, daher wird
                  * hier frühzeitig beim Compilieren auf einen Fehler 
                  * hingewiesen. */

if (NULL == var) /* Dies ist eine sicherere Art zu vergleichen */
```


### Beschränkungen / Restriktionen 

Diverse Konstrukte bzw. Vorgehensweisen, gilt es nicht zu nutzen, da 
sie als gefährlich und undurchsichtig gelten.

## Schleifen

Schleifen sind nicht per se schlecht oder gefährlich, aber dennoch
sollte hier auf die Problematik von Endlosschleifen (gewollt oder ungewollt) 
hingewiesen werden. \\
Jede unserer Tasks ist von einer Endlosschleife umgeben, aber das muss
so sein und ich auch mit diversen Mechaniken abgesichert. Problematisch
sind eher diese Schleifen, die z. B. beim Pollen verwendet werden und keine
Abbruchbedingung (z. B. timeout) im Fehlerfall haben.


### Klammersetzung

## Fehlende Klammern bei Konditionalen
Über die Einrückung kann man zwar annehmen zu welchem ''if'' das ''else'' gehört, 
aber sicher sein, kann man nicht.
```c
if (TRUE == a)                       /* if-a */
    for (i = 0 ; i < MAX_VAL; i++)
        if (FALSE == b[i])           /* if-b */
            retVal = i;
        else                         /* Annahme, dass es zu if-b gehört */
            break;
```
Dies bewirkt (zu recht) beim GCC eine Warning.

Eindeutig wäre es mit entsprechenden Klammern:
```c
if (TRUE == a)
{
    for (i = 0 ; i < MAX_VAL; i++)
    {
        if (FALSE == b[i])
        {
            retVal = i;
        }
        else
        {
            break;
        }
    } /* for */
}
```

Ähnliches Beispiel:
Gehört das ''return 0'' hinter das ''for'' oder unter das erste ''if''?
```c
for (i = MAX_VAL; i; i--)
    if (var[i - 1].member)
        if (TRUE == a)
            return i;
return 0;
```

### Fehlende Klammern bei Makros

## Bei Arithmetik

Makros mit arithmetischem Anteil sollten stets geklammert werden.
```c
#define BUFFER_SIZE 0x8000
#define MASK_MACRO  BUFFER_SIZE - 1
...
buffer[i * MASK_MACRO] = value;
```

Mit aufgelöstem Makro sieht dies wie folgt aus:
```c
buffer[i * BUFFER_SIZE - 1] = value;
```
Da das ''*'' stärker bindet als das ''-'' (//Punkt-vor-Strich//), wäre die logische Auflösung
```c
buffer[(i * BUFFER_SIZE) - 1] = value;
```
und nicht wie gewollt
```c
buffer[i * (BUFFER_SIZE - 1)] = value;
```

Daher sollten Makros, die keine reinen Konstanten sind, __immer__ geklammert werden.
```c
#define MASK_MACRO  (BUFFER_SIZE - 1)
```

Es sollte bei selbstdefinierten Makros auch immer davon ausgegangen werden, dass der User keine Konstante
übergibt, daher sind Makrovariablen, in diesem Falle ''a'', __immer__ zu klammern!
```c
#define MASK_MACRO_VAR(a)   ( (u32) (a) * 42 )
```

## Bei Mehrzeiligen Makros

Mehrzeilige Makros sind praktisch, wenn es darum geht einen zusammengehörigen 
Codeteil an mehreren Stellen zu verwenden. Es ist meist übersichtlicher und 
vor allem wartbarer, als wenn man immer den Code kopiert. \\
Aber man sollte Obacht walten lassen!

Nutzt man dieses Makro in folgendem Code

```c
#define MULTI_LINE_MACRO(a,text)   printf("line%d\r\n", a); \
                                   printf("%s\r\n", text)
                              
void fnPrint(void)
{
    MULTI_LINE_MACRO(__LINE__, "test");
}
```

stellt dies noch kein Problem dar. Aber wenn man nun den Code
erweitert, sieht er im ersten Moment richtig aus und es gibt 
auch keinen Compileerror.

```c
void fnPrintWithIf(void)
{
    int l = __LINE__;
    if (l% 2)
        MULTI_LINE_MACRO(l, "is odd");
    //else
    //    MULTI_LINE_MACRO(l, "is even");
}
```

Aufgelöst sieht es aber so aus:

```c
void fnPrintWithIf(void)
{
    int l = __LINE__;
    if (l % 2)
        printf("line%d\r\n", l);
    printf("%s\r\n", "is odd");
}
```

Es wird die zweite Zeile IMMER ausgegeben, was natürlich falsch ist.
Mit dem auskommentierten ''else''-Zweig wäre es wegen eines Compilefehlers
aufgefallen.

Jetzt könnte man denken: "Ich mache einfach Klammern drum!"
```c
#define MULTI_LINE_MACRO(a,text)   { printf("line%d\r\n", a); \
                                     printf("%s\r\n", text); }
```

Dann muss man aber das letzte '';'' hinzufügen und es könnte sein,
dass man im Code dennoch ''MULTI_LINE_MACRO(1,"test");'' schreibt,
was bei einem ''if/else'' wiederum zu einem Compilefehler führt.

```c
void fnNewPrintWithIf(void)
{
    int l = __LINE__;
    if (l% 2)
    {
        printf("line%d\r\n", l);
        printf("%s\r\n", "is odd");
    }
    ;
    else
    {
        printf("line%d\r\n", l);
        printf("%s\r\n", "is even");
    }
    ;
}
```

Die Lösung sieht im ersten Moment etwas kryptisch aus, funktioniert aber!

```c
#define MULTI_LINE_MACRO(a,text)   do { printf("line%d\r\n", a);  \
                                        printf("%s\r\n", text); } \
                                   while ((0))
```

Es steht nun immer nur eine Anweisung jeweils bei ''if'' und ''else''.
```c
void fnNewPrintWithIf(void)
{
    int l = __LINE__;
    if (l% 2)
    do {
        printf("line%d\r\n", l);
        printf("%s\r\n", "is odd");
    } while ((0));
    else
    do {
        printf("line%d\r\n", l);
        printf("%s\r\n", "is even");
    } while ((0));
}
```

**Wichtig ist die doppelte Klammer, da es sonst eine Compilerwarning gibt!** \\
Ihr könnt aber auch ''do {...} while (FALSE)'' benutzen, da ''FALSE'' schon ''(0)'' ist.


### Zugriffsbeschränkungen

Nicht jede Funktion und jede Variable ist für Jedermann gedacht, sowohl lesend, 
als auch schreibend. Daher sollten lokale Funktionen und Variablen statisch 
(''static'') definiert werden. Dies spart (je nach Compiler-Einstellung) Platz 
in der späteren Object-Datei und der Compiler weist einen auch auf nicht genutzten
Code (Funktionen und Variablen) hin. 


### "Toter" Code

Nicht mehr verwendete Code-Stellen sollten im Sinne der Code-Pflege entfernt
werden. Es gibt aber auch Code, der je nach Compiler-Optionen nicht verwendet 
wird. Diese Stellen sollten z. B. mittels Define-Schalter konfigurierbar hinzu-
bzw. abgeschaltet werden. Wann dieser Code aktiv ist und wie er aktiviert wird
sollte dokumentiert werden. Dies gilt auch für Test-Code.
Auskommentierter, nicht extern zuschaltbarer Code (z. B. mittels ''%%//%%'' oder 
''/*  */'') sollte für die spätere Kundensoftware entfernt werden.
Auch nicht verwendete Variablen (hier wirft der Compiler eine Warning) sollten 
entfernt werden.

FIXME
Für eine Prüfung auf ungenutze Codestellen gibt es sogenannte Code-Coverage-Tools,
die auf mehrere Arten und Weisen prüfen, ob auch der ganze Code verwendet wird.
Hierzu prüfe ich gerade die Nutzbarkeit für unsere Embedded Systeme, da hier 
der Einsatz nicht trivial ist.


### Sprungbefehle

Darunter fallen
''goto''s, ''switch/case''-Anweisungen, ''break'', ''continue''s und sogar 
''return''s wenn sie mitten in einer Funktion verwendet werden.

Bekannt und nicht zuletzt verrufen sind ''goto''s. Sie sollten 
nur bei der Abhandlung von Fehlerfällen oder Ausnahmebehandlungen genutzt 
werden. Hierbei sollte das Sprung-Label (z. B.''ERROR_xyz'') hinter dem 
normalen Austrittspunkt (''return FALSE;'') liegen.
Auch wenn //Labels// lokal beschränkt sind, sollten sprechende und eindeutige 
Namen (gerade bei mehrfacher Fehlerbehandlung) genutzt werden.
Sprünge in Schleifen oder Konditionalblöcke (''if'', ''else'', ''switch'', 
''while'', ...) sind stets zu vermieden.

Bsp.
```c
...
if (NULL == pointer)
{
    goto ERROR_NULL_PTR;
}
...
if (100 < value)
{
    goto ERROR_VALUE;
}

...

return FALSE;

ERROR_NULL_PTR:
/* Fehlerbehandlung a */
...
return TRUE;

ERROR_VALUE:
/* Fehlerbehandlung b */
...
return TRUE;
```


### Nutzen von Konstanten (const)

Sollte ein Strings mehrfach im Code verwendet werden, sollte man dies nicht 
mittels ''define'', sondern über eine Konstante lösen. Dies spart Speicherplatz 
und erleichtert das Refactoring.

```c
#define TEXT_STR     "TEXT"
const char* text_p = TEXT_STR;

printf("1. Textausgabe = %s", TEXT_STR); /* Hier wird jedes Mal */
printf("2. Textausgabe = %s", TEXT_STR); /* der Text ''TEXT_STR'' */
printf("3. Textausgabe = %s", TEXT_STR); /* im Object */
printf("4. Textausgabe = %s", TEXT_STR); /* hinterlegt. */

printf("1. Textausgabe = %s", text_p); /* Hier wird jedes Mal */
printf("2. Textausgabe = %s", text_p); /* der Text hinter der */
printf("3. Textausgabe = %s", text_p); /* Konstanten ''text_p'' */
printf("4. Textausgabe = %s", text_p); /* verwendet. */
```


### Fallauswahl (switch-case)

In einem ''switch-case''-Konstrukt sollte immer ein Standardfall (''default'')
angelegt werden (auch wenn dieser nichts tut). Sollte dieser absichtlich 
weggelassen werden, sollte hier ein entsprechender Kommentar angelegt werden, 
warum es keinen Standardfall gibt.


#### Fallstricke

Es gibt für jede Programmiersprache diverse fehlerträchtige Sprachkonstrukte. 
Diese sollten vermieden werden, soweit man dies kann. Wenn nicht, sollte man 
sich im Klaren sein, was es für Auswirkungen habe könnte und wie der Compiler 
mit solchen Konstrukten umgeht. Hier sind folgende zu nennen:
  * Gotos ([siehe oben](#Sprungbefehle))
  * Gleitkommazahlen (''floats'', ''doubles'')
  * Rekursionen
  * dynamische Speicherreservierung zur Laufzeit
  * Parallelverarbeitung oder -speicherzugriff auf geteilten Speicher

FIXME

## Typenumwandung

Auch bei ''cast''s (Typenumwandung) sollte man vorsichtig sein! Wenn wir einen
''s8'' (''signed char'') zu einem ''int'' (ebenfalls ''signed'') casten,
kann unter Umständen so etwas geschehen:
```c
s8 sc = 0xfa;
int si = (int) sc;
printf("si=0x%x", si);
>>> si=0xfffffffa
```
Wer hätte hier ''0xfffffffa'' erwartet? Die meisten wohl ''0xfa''.

Wusstet ihr auch, dass sowohl unser PPC-, als auch der ZYNQ-Compiler 
''char'' als ''unsigned'' ansehen? Sonst kennt man ''char'' als ''signed''-Typen.
Daher sollte man immer die Eigenheiten und Einstellungen des Compilers
kennen. Schaut euch hierzu die Handbücher der jeweiligen Compiler an.

** Rechnen mit oder vergleichen von unterschiedlichen Typen (signed<->unsigned) **

Ein wichtiger Faktor ist auch, wie implizit gecastet wird und welcher Typ vorrang hat.
Nehmen wir folgendes Beispiel:

```c
unsigned int a = 1000;
         int b = -1;

if (a > b) 
{
    printf("a (%d) is greater than b (%d)\n", a, b);
}
else 
{
    printf("a (%d) is less than or equal to b (%d)\n", a, b); 
}
```

Die Ausgabe ist: \\
  >> a  (1000) is less than or equal to b (-1)
Und dies ist nicht in erster Linie ersichtlich und das Ergebinis ist 
immer von der Implementierung abhängig.

## Schreibweisen beim Stellenwertsystem

```c
int dez =   37; /* dezimal */
int oct =  037; /* oktal */
int hex = 0x37; /* hexadezimal */
int HEX = 0X37; /* hexadezimal */
```

Eine führende ''0'' macht aus einer Dezimalzahl eine auf 
Oktalbasis! Wird das ''x'' bei einer hexadezimalen Zahl 
vergessen, erhalten wir ebenfalls wieder eine Oktalzahl!

## Portabilität

In 32-Bit-Systemen gibt es keinen Unterschied zwischen -1 
und 0xffffffff. Wechselt man aber auf ein 64-Bit-System
kann es zu Problemen kommen, da hier
```
-1 != 0xffffffff ? 0x00000000ffffffff
```

Um so etwas portabel zu lösen sollten die zur Verfügung stehenden 
Möglichkeiten genutzt werden. So gibt es z. B. in der ''limits.h''
das Define ''UINT_MAX'', das hier aushelfen könnte.


#### Präprozessor-Tipps

### IN-OUT-Makros bei Funktionen nutzen

Eine interessante Technik: ob ein Funktionsparameter als Übergabe __in__ 
eine Funktion gilt oder später über ihn etwas zurück (__out__) gegeben wird (oder 
beides), kann man mit drei simplen (leeren) Defines anzeigen.
```c
#define _IN_
#define _OUT_
#define _IN_OUT_
```

So könnte z. B. ein Funktionsprototyp in einem API-Header folgendermaßen aussehen:
```c
void func(_IN_ u32 inVar, _IN_OUT_ u8* pBufferInOut)
```
Da diese Defines nach dem Durchlauf des Präprozessors "verschwinden", ändern 
sie nichts am Code, geben aber dennoch nützliche Informationen über die
Parameter. \\
Dies erleichtert auch das Erkennen von Call-By-Value- und Call-By-Reference-Parametern.


### Sprechende Defines

Hartkodierte Werte, (//magic numbers//) sollten vermieden werden, da nicht 
genau ersichtlich ist, was damit gemeint ist. Dies gilt besonders bei Adressen, 
die auf Hardwareadressen zeigen! Hier sollen entweder Linker-Variablen oder
sprechende Defines oder Konstanten genutzt werden. Dem Compiler ist es egal, ob man
```c
const int c = 4711;
```
oder
```c
const int cologneWaterTheRealOne = 4711;
const int contactTaxiMayen= 4711;
```
schreibt.


Bspe.:
```c
float umfang = 2 * 3.14 * radius;
```
=>
```c
#define PI 3.14
float umfang = 2 * PI * radius;
```

''defines'' helfen hier auch bei der Wartbarkeit des Codes. Hätte man 
100 Stellen mit einem hartkodierten Wert und merkt, dass man ihn anpassen
muss, sagen wir, wir brauchen einen genaueren Wert von PI 3.14 -> 3.141592653, 
muss man 100 Stellen händisch anpassen. Hat man stattdessen ein
''define'' genutzt, muss man lediglich eine Stelle korrigieren und der 
Präprozessor erledigt den Rest. Auch Zahlendreher (3.141592**6**53 != 3.141592**5**63)
gibt es wenn nur einmal und sind schneller gefunden und behoben als sonst.

**Ein Refactoring bei Magic Numbers ist nahezu unmöglich!**

Magic Numbers in String können bei Änderungen auch zu Verwirrung führen:
```c
if (a<100)
{
    printf("a is less than 100\r\n");
}
else
{
    printf("a is great than or equal to 100\r\n");
}
```

Eine Magic Number, aber drei Verwendungen. Eine Lösung wäre ein Formatstring 
(wenn dies möglich ist)
```c
printf("a is less than %d\r\n", ONE_HUNDRED);
```

### Prüfen von enummerierten Listen mit ct_assert

```c
typedef enum BSP_Enum
{
    BSP_FIRST, /**<   [ 0 ]  */
    BSP_SECOND, /**<  [ 1 ]  */
    BSP_THIRD, /**<   [ 2 ]  */
    BSP_FOURTH, /**<  [ 3 ]  */
    BSP_MAX /**<      [   ]  */
} BSP_Enum;

static BSP_Struct BSP_list[] = { {BSP_FIRST , "FIRST"},
                                 {BSP_SECOND, "SECOND"},
                                 {BSP_THIRD , "THIRD"},
                                 {BSP_FOURTH, "FOURTH"},
                               };
ct_assert(ARR_ELEMS(BSP_list) == (u32) BSP_MAX );
```

## Gleitkomma-Rechnungen

Nur um dies wieder aufzufrischen: \\
  int a / int b, wenn b > a ergibt 0!
Sobald ein Faktor in einer Rechnung eine Gleitkommazahl (float, double) ist, werden auch die Faktoren impliziet dorthin gecastet.

Gute weiterführende Links findet ihr hier:
  * [Why 0.1 does not exist in floating point](https://www.exploringbinary.com/why-0-point-1-does-not-exist-in-floating-point)
  * [C-FloatingPoint](https://www.cs.yale.edu/homes/aspnes/pinewiki/C(2f)FloatingPoint.html)
  * [What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)
  * [Binary numbers floating point conversion](https://blog.penjee.com/binary-numbers-floating-point-conversion)

## Globals

FIXME
achtung bei globalen defines!!!
wenn in einer api ein define abgeändert wird (wissentlich oder unwissentlich) wirkt sich das auf alle benutzen stellen aus. das bekommen die nutzer ggf nicht mit.


#### Literale nutzen

FIXME
Bei defines wird oft der "Typ" vergessen.

|  **Datentyp**  |                                  **Suffix**                                                 |           **Bedeutung**           |
|:--------------:|:-------------------------------------------------------------------------------------------:|:---------------------------------:|
|    ```int```   |                               ```u``` oder ```U```                                          |         ```unsigned int```        |
|    ```int```   |                               ```l``` oder ```L```                                          |             ```long```            |
|    ```int```   |     ```ul```, ```uL```, ```Ul```, ```UL```, ```lu```, ```lU```, ```Lu```, oder ```LU```     |        ```unsigned long```        |
|    ```int```   |                              ```ll``` oder ```LL```                                         |          ```long long```          |
|    ```int```   | ```ull```, ```uLL```, ```Ull```, ```ULL```, ```llu```, ```llU```, ```LLu```, oder ```LLU``` |      ```unsigned long long```     |
|  ```double```  |                               ```f``` oder ```F```                                          | ```float``` (**KEIN double !!!**) |
|  ```double```  |                               ```l``` oder ```L```                                          |         ```long double```         |
 	
Links
  * [Constant suffixes and prefixes in ANSI-C](https://dystopiancode.blogspot.com/2012/08/constant-suffixes-and-prefixes-in-ansi-c.html)
  * [Integer literals](https://en.cppreference.com/w/cpp/language/integer_literal)


#### Abstraktionsebenen einführen

Zu Software Dritter (z. B. Libs, Open-Source-SW, ...) und Hardware- 
Kernel-Treibern (auch unseren!) sollte eine Abstraktionsebene (//abstraction layer//) 
geschaffen werden, der ungewünschte Abhängigkeiten zu sich ggf. ändernden 
Schnittstellen (APIs) hat. Als Beispiel sei hier ein Prozessorwechsel zu nennen.
Die API-Funktionen, die von der Applikationsschicht genutzt werden, sollten so
geschrieben sein, dass die unterliegende Schicht getauscht werden kann, ohne dass
alle Applikationsmodule angepasst werden müssen. Die Änderung geschieht 
ausschließlich im abstraction layer.

Bsp. Die schon länger eingesetzte Lib1 wird durch Lib2 ersetzt
```
   Lib1              Abstract-Layer       Application
                                         +---------------+
                                         | AppFunc()     |
                    +----------------+   | {             |
  +-------------+   | AbsLyrA() ------------> AbsLyrA(); |
  | Lib1FuncA() ----> {Lib1FuncA();} |   |    ...        |
  | {...}       |   | AbsLyrB() ------------> AbsLyrB(); |
  | Lib1FuncB() ----> {Lib1FuncB();} |   |    ...        |
  | {...}       |   | AbsLyrC() ------------> AbsLyrC(); |
  | Lib1FuncC() ----> {Lib1FuncC();} |   | }             |
  | {...}       |   | ...            |   +---------------+
  | ...         |   |                | 
  |             |   +----------------+ 
  +-------------+
```

```
   Lib1->Lib2        Abstract-Layer       Application
                                         +---------------+
                                         | AppFunc()     |
                    +----------------+   | {             |
  +-------------+   | AbsLyrA() ------------> AbsLyrA(); |
  | Lib2FuncA() ----> {Lib2FuncA();} |   |    ...        |
  | {...}       |   | AbsLyrB() ------------> AbsLyrB(); |
  | Lib2FuncB() ----> {Lib2FuncB();} |   |    ...        |
  | {...}       |   | AbsLyrC() ------------> AbsLyrC(); |
  | Lib2FuncC() ----> {Lib2FuncC();} |   | }             |
  | {...}       |   | ...            |   +---------------+
  | ...         |   |                | 
  |             |   +----------------+ 
  +-------------+
```

Dank der Abstraktionsschicht müssten bei der Applikation keine 
Änderungen vorgenommen werden. Dennoch müssten Tests durchgeführt
werden, ob die Arbeitsweise der neuen Lib-Funktionen gleich den
alten ist.


#### Übersichtlichkeit

Gemeinsam genutzte Daten (Kontextdaten, globale Daten, ...) bestehen oft aus 
einer großen Anzahl an Members (sehr viele ''struct''-Einträge). Hierbei kann man
schnell die Übersicht verlieren und mehrere Entwickler können aus Versehen 
die selben Daten mehrfach in verschiedenen Members ablegen. Dies verschenkt
Speicherplatz und auch Rechenressourcen. Auch die Übersichtlichkeit leidet
sehr an der schieren Masse an Einträgen, sodass z. B. modulfremde Entwickler
oder Neulinge Einträge einfach nicht finden. \\
Daher sollten große Strukturen gut strukturiert, die Member eindeutig und
passend beschreibend benannt und alles gut dokumentiert und beschrieben sein.
Da das Umsortieren von ''structs'' fehleranfällig ist, rate ich von einer 
namentlichen Sortierung bei einer Erweiterung ab (-> Speicherverschiebungen, ...)
Als Lösung gäbe es hier eine granularere Gruppierung (Modularisierung) von 
Daten. So sollten z. B. Messwerte, Bilddaten, Gerätedaten, ... in einzelnen
Modulen und somit in einzelnen Unter-Structs abgelegt werden, deren Pointer
dann später nur in die Oberstruktur eingetragen wird.

Statt alles in einem Header zu definieren
```c 
//app_bsp.h
typedef struct BspGlobalDataStruc
{
    /* Teil A */
    u32 varA01;
    u32 varA02;
    u32 varA03;
    ...
    struct PartA01 sA01;
    struct PartA02 sA02;
    struct PartA03 sA03;
    ...
    
    /* Teil B */
    ...
    
    /* Teil C */
    ...
    
}BspGlobalDataStruc;
```

sollten Gruppen gebildet werden, 

```c
//app_bsp_suba.h
typedef struct BspSubDataStrucA
{
    /* Teil A */
    u32 varA01;
    u32 varA02;
    u32 varA03;
    ...
    struct PartA01 sA01;
    struct PartA02 sA02;
    struct PartA03 sA03;
    ...
    
}BspSubDataStrucA;
```

```c
//app_bsp_subb.h
typedef struct BspSubDataStrucB
{
    /* Teil B */
    ...
    
}BspSubDataStrucB;
```

```c
//app_bsp_subc.h
typedef struct BspSubDataStrucC
{
    /* Teil C */
    ...
    
}BspSubDataStrucC;
```

die dann nur Referenzen auf die Unterdatenstrukturen enthalten
```c
//app_bsp.h
typedef struct BspGlobalDataStruc
{
    /* Teil A */
    BspSubDataStrucA* pGDa;
    
    /* Teil B */
    BspSubDataStrucB* pGDb;
    
    /* Teil C */
    BspSubDataStrucC* pGDc;
    
}BspGlobalDataStruc;
```


#### Operatorrangfolge

Hierzu bitte die [Tabelle](https://en.cppreference.com/w/c/language/operator_precedence) beachten.
