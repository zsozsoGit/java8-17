# Bevezető

istvan.viczian@training360.com   
_Effective Java_ és _Clean Code_ könyv, enélkül nem is lenne szabad java programozni.
_Java Concurrency in Practice_ brutál könyv

https://www.elegantobjects.org/

https://www.jtechlog.hu/2022/09/13/spring-boot-3.html

https://sdkman.io/jdks

Témák:
* Streamek
* Thread már egy "régi téma"
* Csinálj jegyzeteket
* Bátran kérdezni
* "wiki Java version history" - google sokat ad :)

# Streamek
## Java8 funkcionális programozás
Lambda kifejezés:

``` java
metódus_paraméter -> metódus_törzs
```
Metódus paraméter lehet üres is `()`
A metódus paraméter lokális változó valójában...
El lehet hagyni a paraméter nevet, helyette _method reference_ a jó... és akkor nem is kell metódus paramétert bevetni...

A metódus törzsbe lehet több minden is. De ezt ki lehet emelni az ideával egy metódus referenciává.

A függvény egy first-class citizen: válozó lehet egy függvény, mint visszatérési érték, vagy paraméter is lehet.
Mikor jó ez? Pl.: strategy design pattern, tehát egy nagy algoritmusban egy kicsi részét szeretnénk paraméterezhetővé tenni ennek. Pl. rendezésnél az összehasonlítást cserélhetővé teszem. Régen a comparable interfészt kellett implementálni.

Hogyan jönnek a képbe a _stream_-ek? Ezek nem az input és output streamek! (I/O részei!)
Ezt mint egy cső kell elképzelni, mert adatfolyam. A csövön van egy ablakocska... És mindig csak egy elemet látok itt. Nem tudok használni streamet, ha az elemek között kell valami összefüggést keresni, vagy használni az algoritmusomban. Lehet több ablak is, de csak mindig egy elemet látunk...

Ne csináljunk bonyolult lambdákat, hanem szervezzük ki a logikát az osztályba.

Short-circuiting: véges lesz végtelen esetén...
Vannak még a cső elején
* forrás
* lezáró művelet: count, min, max, forEach, _reduce_, _collect_ a vége mindig ez a kettő valójában, tehát a többi is ezt a kettőt implementálja.

Egy ilyen művelet (ablak) lehet "állapotos" és állapottal nem rendelkező.

* Stateless: Filter
* Stateful: map, flatMap, distinct, _sort_, mert be kell várni az összes elemet

## Coding
* psvm: main függvény generálás
* Lombok dependency - delombok és akkor minden láthatóvá lesz
* Lomboknak van ilyen annotációja `@slf4j`
* Lombok csinál `toString`-et is
* Lombok `@Value`?
* Miért kell IntStream? Az int-tel való műveletek gyorsabbak, mint az Integer-rel
* limit(), egy későbbi operáció, a stream-ben és ez behatárolja a dolgokat...
* **muszáj lezárót alkalmazni, "húzza" a dolgokat, elindítja az elemek generálását. Ez "lazy" kiértékelés.**
* "File beolvasás: `BufferedReader`-rel" helyett: 
```java
Files.lines(Path.of("file.csv"))
    .filter(s->s.startsWith("1"))
    .map(s-> s...)
    .mapToInt(...)
    .sum
```
A JPA repository támogatja a stream feldolgozást, `@Transactional`. 

`Stream<Employee> findEmployee().forEach...`  Az EmployeeService interface erre jó, az infrastruktúra megcsinálja helyettem. Érdemes a SpringData JPA doksiját átböngészni. DTO is hasznos.
Clean code: előbb legyen szép. A performancia bajok oka általában az adatbázis (lassú select) és a hálózati kapcsolat volt.

Érdemes ilyen Architecture Deceision List-et használni, hogy akkor most milyen megoldásokat, konvenciókat fogunk alkalmazni. Moduláris programozás vagy újabban az MSA nagyon hasznos és akkor nem kell ilyen giga monolitikus alkalmazásokat összehozni.

### Referencia implementációk
Érdekes, hogy van egy szabvány és akkor van egy referencia implementációja ennek. De lehet más implementáció is. Sokszor ezek jobban is teljesen teljesítenek.

"Java jó backend fejlesztésre, szkriptelésre shell vagy python."
"Ne nagyon használjunk öröklődést... Asszimetrikus, mert nem ismeri elvileg az ősosztály a leszármazottakat"
"Reflection, `instanceOf`, `Object` class használata -> _codesmell_
## Reduce
Hasonló a Hadoop-hoz (mapreduce), ami több számítógépre (node) osztja szét a feladatot.
Alapból mindig egy szálon megy a stream.

Három paramétere van a `reduce()`-nak
* identity - _new_ eredmény
* accumulator - bifunction, tehát bemegy két elem, és kijön egy - _egy szálon csinálja_. egyik az identity és utána jön amit adunk neki a streamből, mint a második
* combiner - Bináris operátor összegzi a szálak eredményeit

Ezeket metódusokat érdemes áttenni az osztályba magába.
Így jön létre egy immutable osztály (Lásd `Count` osztály a tananyagban)
Példányosítás nem nagyon költséges, ha nem csinálunk fura dolgokat a konstruktorban. Nem is túl ajánlott konstruktorban kódot írni, csakis értékadásokat. Itt fontos, mert mindig új objektum jön létre.


## Collectors `collect()`

Az `asList` létrehoz egy nem bővíthető listát... Helyette jobb a `List.of`, ami már nem is módosítható...
Helyette használd a copy-constructort :)
```java
new ArrayList(List.of(...))
```
Itt identity helyett supplier van. Itt a supplier minden szálnak létrehoz egy listát
Az accumulator másképpen működik.
Itt nem kell másik objektum, hanem annak az állapotát módosítja, ami a supplierben van, tehát nem kell annyi példány.
Itt eltérően a `reduce`-tól, szálanként hoz létre objektumokat.
Mutable osztály kell.
### Collector Characteristics
* FINISHED (?) Kell-e finisher, tehát egy másik típus jön-e létre
* CONCURRENT: lehet szálbiztos?
* UNORDERED: (?)
### "Standard" Collectors
* `sum()` összeg
* `summarize()` több infó, átlag, méret, összeg, stb.
* `toMap()` valami szerint `Map`-pé tudom alakítani, és utána egyszerűen kereshető lesz. Gyorsabb lesz az egész történet. `mergeFunction`, stb. opcionálisan megadható.
* `groupingBy()`
* `flatMap()` "kiterítés" több elem is kijöhet...
![](https://1.bp.blogspot.com/-RJseuNzmm7I/Vtb3pH7iPkI/AAAAAAAAE-s/ZJSxR4EnlSI/s1600/Java%2B8%2BflatMap%2Bexample%2B.jpg)

#### Optional mint pl. `findFirst()`! Ezt lehet `.get()`-tel "megoldani".

## Új feature-ök

* mulitline string
* `var` meghatározza a típust
```java 
var text = """
    fjkdjdkjkf
    fjdkjd
    """
```

## Párhuzamos programozás eszközei
### Java 1
Már a nyelv megjelenésekor nyelvi elemek voltak erre. Pl. `Thread` osztály, `Runnable` interface. Viszont a `Thread` alacsony szintű valami, nem ajánlott ma már. `synchronised` nyelvi elem volt kezdetben erre, a versenyhelyzetek kiküszöbölésére. De ma már ezt se használjuk.
### Java 5
#### Executor framework 
 `Executor` interface ->  `Executor service` -> `Scheduled Executor service` -> ...
 Thread pool került bevezetésre.
 Ha csak egy feladatot adok meg, akkor csak egy szálon futtatja a dolgokat.
 *  `execute()` részben kell megadni mit akarok futtatni. De itt nem lehet visszatérési érték.
* `submit()` függvényben van `Callable` lehetőség, tehát tudok visszatérni.
 kell a mindeség végén `shutdown()` a service-en.
 * `future.get()`-et mindenképpen egy timeout-tal kell meghívni.
 #### ForJoinPool és Task
 Felosztható feladatok megoldására
 #### atomicInteger
 Szálbiztos munka, versenyhelyzetek nem veszélyesek
 #### CountDownLatch
 A szálak addig várnak, míg a számláló nulla nem lesz.
 #### Cyclic Barrier
 Újrafelhasználható CountDownLatch, több szál is bevárja egymást.
 #### Phaser
 A feldolgozást fázisokra bontja, és megmondja, hány szálon fusson.
 #### Semaphor
 Egy erőforrás (I/O), amit csak korlátozott számú szálon lehet meghívni.
 ##### Best practices
 Timeout beállítása távoli rendszer elérése esetén.
 Limitálni a háttérhívások számát (távoli rendszer).
#### Lock interface
A synchronised működését csinálja.
De sokkal inkább fair, nincsen éheztetés. Le lehet kérdezni, hogy van-e szabad lock. Nem csak egy metóduson belül használható. A blokkolt szál is megszakítható.
### Java 8
#### Completable Future
Ez hasonlít a javascript async/await funkciójára. A callback használatát egyszerűsíti és támogatja a funkcionális programozást.

Ez egy párhuzamossági keretrendszer. Van, hogy hamarabb áll le az alkalmazás, mint ahogy a thread lefutna, ezért érdemes a `future.get()` használata. :)
* `runAsync` : void
* `supplyAsync`: van visszatérési érték.

Itt van még a `thenXXX..` és akkor ilyen stream-szerűen lehet továbbdolgozni... Párhuzamos programozás funkcionális interfészekkel :)
Lehet olyan, hogy két szolgáltatást hívok meg, és egyik szolgáltató gyorsabb, akkor az elsővel dolgozok tovább, a másikat eldobom.
Vagy hasonló az MSA _API Composition_ patternhez. Ekkor meg pont összekombinálom a kettőt. Ekkor lesz egy kombinált REST végpontom.Pa

A try-catch-re is van egy megoldás itt. `whenComplete((result, e)->...)`, ahol az `e` egy exception.

Van ilyen is, hogy `future.thenCombine`, ekkor összevárja a két CompletableFuture "szálat".
Hasonló a `thenAcceptBoth()`, viszont az `applyToEither()` az elsővel kezd el dolgozni.

_Tehát ez egy nagyon elegáns magas szintű eszköz._

### Steam párhuzamosítás
Ez a combiner-nél jött elő először. A szálak száma a logikai CPU-k száma -1.
Párhuzamos futtatásnál módosulhat az eredmény is! Pl. `forEach` esetén a feldolgozási sorrend. A `findFirst` az elsőt adja vissza, de `findAny` a legelőször elkészülőt.

Kis elemszám esetén nem mérhető a párhuzamosítás előnye, tehát csak magas erőforrásigényű műveletek esetén érdemes, pl. HTTP lekérések esetén vagy milliós elemszám esetén, mert a szálmenedzsment maga is rengeteg erőforrást igényel.

Globális válozók veszélyesek. Stream-ből ne nyúljunk ilyenhez. Helyette lehet synchronised. Vagy: `CopyOnWriteArrayList`. A `final` csak az értékadást tiltja le, de a módosítást nem... Érdemes egy saját thread poolt létrehozni a `ForkJoinPool`-lal.
Bizonyos műveletek, mint a `flatMap` elveszik a párhuzamosságot. `unordered` gyorsíthat. Az `...ordered` pedig lassíthat.
* `reduce()` eléggé bonyolult, de az immutabilitás miatt nem gond a párhuzamosság.
* Collector viszont különböző karakterisztikákat tartalmaz erre.

### Java 11
HttpClient párhuzamos és modern is. Illetve a többi ehhez tartozó osztály is. Pl. a HttpResponse streamet ad vissza. A `.parallel()` bárhol lehet, akkor is a megfelelő helyen kezdi el a párhuzamosítást. Tehát maga a stream lesz paralell. Ugyanígy a limit is már az elején lekorlátozza a dolgot.
# Java 9 újdonságok
* Vannak már privát metódusok interfészekben. (default és static után)
* try with resources _effective final_ változókon
* Visszafelé inkompatibilis: már _nem_(?) lehet `_` változónév
* `@SafeVarArgs` ("`...`" - varargs) Generikus típus és varargs kombinációja veszélyes és mostmár rátehető private változókra. Generikus és primitív típusok nem barátai egymásnak, és hasonló a tömbökkel is a generikus típusok helyzete.
Mert generikusnál van egy implicit kasztolás.
## Java Plattform Module System
* Új láthatóságok
* Jar-ok keresése induláskor és hiányzók, duplikátumok keresése. JAR-Hell handling. Hogyan? Meta-adatokkal. Hol? `module-info.java` file. Kulcsszavak: `requires`, `exports`; `provides`, `uses`... A hívó oldalnak nem kell ismernie az implementáló osztályt. `IF.load(IF.class).findFirst().orElseThrow(...)`
Ehhez hasonló az OSGI. Ez kicsivel többet tud, pl. futás közben másik JAR változatot betölteni.
* nincsen többé `rt.jar`, csak a JDK-ból azt kell odatenni, ami pont kell, így kisebb lesz a csomag. Java Linker.

* diamond operátor `<>` kivezetődik, mert egyszerűbben lehet létrehozni kollekciókat.
* `optional.stream()`
* Scanner metódusok kimenet: Stream. 
* `valueOf` preferáltabb a konstruktorok helyett.
* Reaktív programozás, reaktív kiáltvány. -> Reactive Streams API.

# Java 10

Itt jelenik meg a
* `var` syntatic sugar, de ettől még nem lesz dinamikus programozási nyelv. Olvashatóvá teszi a kódot. De pl. streameknél nehéz az IDE segítsége nélkül megmondani, hogy mi lett a változó.
* Intersection types, két interfészt is lehet implementálni
* ...

# Java 11

Oracle fizetőssé tette. Megjelentek a szabvány alapján írt alternatívák...
Az OpenJDK nem ajánlja magát enterprise környezetben használni. Van az eclipse Temurin, vagy az Amazon Corretto

Csomó mindent kiszednek ebben a verzióban, innét nem is alkalmas a Java már tulajdonképpen vastagkliens fejlesztésére.

Itt is folytatódik az, hogy "megirigyelnek" dolgokat a python-tól pl. és akkor átvesznek ötleteket más programozási nyelvekből.
* String metódusok, amiknek már réges-régen be kellett volna kerülniük
* Hasonlóan: fájl olvasás egyszerűen.

# Java 12...15
Nézd a GitHub slide-okat :)

# Java 16

## Unix Domain Socket channels
Pl. Docker Daemonnal való kommunikáció felgyorsul. Gyorsan lehet bizonyos konténereket provision-álni.

# Java 17
## Sealed classes
Bonyolítja a leszármazást valójában.