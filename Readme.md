# Bevezető

istvan.viczian@training360.com   
_Effective Java_ és _Clean Code_ könyv, enélkül nem is lenne szabad java programozni.
_Java Concurrency in Practice_ brutál könyv

https://www.elegantobjects.org/

https://www.jtechlog.hu/2022/09/13/spring-boot-3.html

Témák:
* Streamek
* Thread már egy "régi téma"
* Csinálj jegyzeteket
* Bátran kérdezni

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

#