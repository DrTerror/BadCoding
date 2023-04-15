# Beispiele gutes/schlechtes Coding

## Verwirrende Zuweisungen/Vergleiche
```c
if (a == b == c) { ... }
```
--> 
```c
if ( (a == b) (a == c) ) { ... }
```

Kompakter (und damit auch komplizierter Code) ist nicht immer mit schnellem/
optimiertem Code gleich zu setzen! Dies hängt viel von den Compiler-Einstellungen
ab.
 
## Dummy-Variable bei Pointer-Übergaben
Wird bei einer Funktionsparameterübergabe eine Referenz (Pointer) übergeben,
sollte diese Adresse stets auf Gültigkeit geprüft werden. Ist diese ungülitg,
darf sie folglich nicht zum Arbeiten verwenden werden. Eine Abhilfe stellt eine
lokale temporäre Variable des entsprechenden Typs dar.
```c
func(int* ptr_p)
{
    int tmp;
    
    if (!PTR_CHECK_RAM(ptr_p))
    { /* ungültig */
        ptr_p = &tmp;
    }
    ...
}
```

Somit ist gewährleistet, dass es validen Speicher hinter einer Referenz gibt.

Der Bereich 0x00--0xff
ist bei unserer Software nichtdefinierter Speicher und somit darf mit diesen 
Daten weder lesend noch schreiben gearbeitet werden. Es gibt daher mehrere 
Prüf-Makros (z. B. PTR_CHECK_RAM, ...), die prüfen, ob eine Adresse in einem
bestimmten Bereich liegt.

## Sichere Vergleiche
Schnell hat man beim Tippen ein Zeichen vergessen, was nicht immer zu einem 
Compile-Fehler führt. Damit ein Vergleichen mit einer Konstanten so nicht zu 
einer Zuweisung wird, kann man die Konstante an erste Position setzen. So würde
beim Vergessen eines ''='' mittels Compile-Fehler gemeldet, dass hier etwas 
nicht stimmt.
```c
if (var == NULL) /* Es soll geprüft werden, ob var gleich 0 ist */

if (var = NULL)  /* Da ein ''=' vergessen wurde, wird aus dem Vergleich eine 
                  * Zuweisung. Dies bewirkt vorerst keinen Compile-Fehler, ist
                  * aber inhaltlich falsch. */

if (NULL = var)  /* Einer Konstanten kann nichts zugewiesen werden, daher wird
                  * hier frühzeitig beim Compilieren auf einen Fehler 
                  * hingewiesen. */

if (NULL == var) /* Dies ist eine sicherere Art zu vergleichen */
```

## Beschränkungen / Restriktionen 

FIXME 

diverse konstrukte bzw vorgehensweisen, gilt es nicht zu nutzen, da sie als gefährlich und undurchsichtig gelten.
''GOTO'', ''...''

## Klammersetzung 

### Fehlende Klammern bei Konditionalen 
Über die Einrückung kann man zwar annehmen zu welchem if das else gehört, 
aber sicher sein, kann man nicht.
```c
if (TRUE == a)
    for (i = 0 ; i < MAX_VAL; i++)
        if (FALSE == b[i])
            retVal = i;
        else
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
```c
for (i = MAX_VAL; i; i--)
    if (var[i - 1].member)
        if (TRUE == a)
            return i;
return 0;
```

### Fehlende Klammern bei Makros 
Makros mit arithmetischem Anteil sollten stehts geklammter werden.
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
übergibt, daher sind Makrovariablen __immer__ zu klammern!
```c
#define MASK_MACRO_VAR(a)   ( (u32) (a) * 42)
```

### Operatorrangfolge 

Hierzu bitte die die <link_todo> beachten.
