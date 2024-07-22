# Adatstruktúra 

## crons
Van sokféle automatizmus amit rendszeresen le szeretnénk futtatni, hogy minden rendben legyen a honlapunkon. És mivel jó sok féle van, ezért a crontab-ban egyetlen paranccsal mondjuk 4 percenként lehívjuk a miserend.hu/cron oldalt. A /cron az mindig megnézi, hogy az alábbi táblázatból melyik deadline_at jött már el, és azok közül egyet megcsinál, ami éppen következik. De azért valami /cron?cron_id=23 formával is meg lehet hívni egyet-egyet leginkább debug miatt. Mert amúgy a crontab-ban nem kérünk emailes értesítést mert sajnos túl sok lenne.

| mezőnév      | típus        | nullable | leírás  |
|--------------|--------------|----------|---------|
| class          | rövid szöveg       | Nem     | Amit futtatni akarunk az mindig a `class::function()` meghívásával történik.|
| function | rövid szöveg       | Nem     |  lásd: class |
| frequency          | rövid szöveg       | Nem     | Az hogy milyen gyakran fusson le valami. Pl. "1 hour" vagy "20 minutes". Úgy számolunk mindig, hogy  `deadline_at = strtotime(now() . " +".frequency)` |
| from          | rövid szöveg       | Nem     | Van, hogy mondjuk csak éjszaka szeretnénk valamit futtatni és sosem nappal, mert túl sok erőforrást eszik meg. Ilyenkor a from = 1am és az until = 6am. Hát ja, mindkét időpontnak azonos naphoz van sázmítva! Vagyis a from=10pm és until=6am hibára fut. Szövegesen van megadva, hogy "1 am". Olyan dolgokat lehet megadni, amit a phpnak a strotime() függvénye tud értelmezni. |
| until          | rövid szöveg       | Nem     | lásd: from |
| deadline_at | datetime vagy timestamp  | Nem kéne  | Hogy milyen időpont után kell(ene) újra futnia. Ha ennek az értéke kisebb mint a jelen ideje, akkor lefut, amint teheti. |
| attempts | szám | 0 ám ha minden jó | Hogy hányszor próbálkoztunk sikertelenül lefuttatni ezt a feladatot. Valahol van egy beállítás, hogy ha mondjuk 100 szor sikertelen, akkor légyszi adja fel és ne eméssze az erőforrásokat tovább.  |
| lastsuccess_at | datetime vagy timestamp  | Nem kéne  | Az időpont, hogy mikor sikerült ennek utoljára lefutnia. Hogy ki tudjuk számolni az új deadline_at -t.  |
| created_at és updated_at | datetime vagy timestamp  | Nem | Csak a laravel miatt került ide. |

Néhány sorról a táblázatban
* \Api\Sqlite::cron() -> Naponta legyártjuk a miserend_v{number}.sqlite fájlokat. Ezeket töltik be a miserend alkalmazások, ezért elég fontosak.
* \ExternalApi\OverpassApi::clearOldCache() -> Az externelapi hívások respone értékeit nyersen fájlban tároljuk, és újra is használjuk. De ha valami már úgyis túl van a cache idején, akkor szépen törlődjenek.
* \KeywordShortcut::updateAll() -> Ez az osm varázslás átalakítása révén érvényét veszíti
* \Distance::updateSome() -> Nagyon sok templom nagyon sok szomszédjának az útvonaltervezős számításait fokozatosan végeztük.
* \ExternalApi\OverpassApi::updaeUrlMiserend() -> Megnézi az OSM-ből az összes olyan izét (node,way,relation) amiben szerepel az url:miserend tag. És szépen hozzárendelni a megfelelő templomhoz az adatbázisunkban, ha még nem történt meg. 
* \OSM::checkUrlMiserend() -> Azt hiszem az előző kiegészítése: megnézni, hogy minden rendben van-e az OSM és miserend.hu összefűzésével
* \OSM:checkBoundaries() -> Juj, elfelejtettem. De az osm világ miatt úgyis átalakul. Ennek van köze ahhoz, hogy egy egy OSM alapú templom melyik egyházmegyéhez tartozik a világban.
* \Token::cleanOut() -> Tokenek a belépéses világhoz kellettek és ott a régiek takarítása. Az új auth rendszerre felejtős.
* \Message::clean() -> Üzeneteket és akár hibaüzeneteket `new \Message()` formában adhatunk meg és az azonos token-el rendelkező legközelebbi oldalbetöltésnél kiírja nekünk. De jól jön némi takarítás pl. a soha meg nem jelentetett üzenetek miatt
* \Photos::cron() -> A fotók kapcsán volt pár dolog amit rendszeresen ellenőriztünk: vannak-e kicsi előképek, tudjuk-e a képnek a méreteit pixelben hogy könnyű legyen elemezni, meg egyáltalán megvan-e minden kép. Egy jól működő rendszerben ez is kidobható. 
* \Crons::gorogkatolizalas() -> A magyar nyelvtan új szabályai szerint helyesen "görögkatolikus" és "római katolikus" igen... Szóval nézzük át, hogy valaki csinált-e butaságot ... De végülis ez sem kell, csak ha már egyszer volt egy naaagy takarítás és rendszeresen régi adatokat töltöttem be, akkor jól jött egy ilyen takarító script.
* \User::sendActivationNotification() -> Ha valaki regisztrált, de nagyon régen nem lépett be, akkor küldünk neki emailt meg aztán töröljük... Avagy ez is egy fontos automatizmus. Egyszerre cska vagy 20 emailt mert elküldeni és azt is csak éjjel, mert azért eszi ez az erőforrásokat
* \User::sendUpdateNotification() -> Kedves felahsználó, légyszi nézz rá a templomodra, hogy minden adat jó-e. Tök sokat segít egy ilyen automata email. Ez is csak éjszaka és egyszerre keveset.
* \User::deleteNonActivatedUsers() -> Aki regisztrált, de sosem lépett be az kap némi értesítőt (sendActivationNotification()) és végül töröljük. Ez is csak éjszaka és egyszerre keveset.

## emails
Jó pár emailt küldünk ki. Van amit rögtön azonnal, más emailek meg ráérnek. Meg hát a terhelés csökkentése érdekében sem kéne mindig azonnal küldeni a mailt.

| mezőnév      | típus        | nullable | leírás  |
|--------------|--------------|----------|---------|
| to, header, subject, body | szöveg | Nem     | Ezek amik mennek a PHPMailer-be. Persze dev környezetben a mailcatcher-nek kéne küldeni mindent. |
| type | rövid szöveg       | Nem     | Mégis legyen valami jelölő, hogy ez milyen email. Pl. hogy felhasználó jelszavának újraküldése az, vagy értesítés egy észrevételről. Gyakorlatilag itt az a string van, ami a twig-ben az email sablonjának a neve. |
| updated_at, created_at | időpont |  | laravel miatt van csak. nekem nem is kell. Na jó, kell azért az nekünk, hogy mikor is ment ki a levél. |
| status | enum('queued','sending', 'error', 'sent') | Nem     | Hogy mi is történt a levéllel. Be lehet állítani a sorba (queued) és akkor legközelebb kimegy amikor eljön az ideje. (Kéne hozzá egy cron job a crons táblába.). 'Sending' esetén elkezdődött és aztán elvileg megváltozik, ha nem dől össze minden: 'sent' = minden ok. 'error' = hát ja, nem mindig minden ok. |

A `to` mező az email cím, nem pedig felhasználó név, mert küldünk bőven emaileket olyanoknak is, akik nem felhasználók. Pl. regisztráláskor. De ha valaki észrevételt küldött be, akkor ő is kaphat (!) különféle válaszleveleket.

FIGYELEM: Szenzitív adatok! Egyrészt annak akinek küldjük itt a címe. De a szöveg tartalma is lehet...



## misek
Egyik legfontosabb tábla jelenleg. Ebben van, hogy hol milyen misék vannak jelenleg. Nem szeretem a struktráját. Lásd #144

Alap struktúra, hogy vannak időszakok pl. téli vagy nyári időszámítás. Először mindig ki kell deríteni, hogy melyik időszak van. Aztán az időszakon belül összegyűjtjük valamennyi misét. Mindegyikhez megvan, hogy melyik nap (vasárnaptól szombatig), de olykor lehet hogy csak minden második, harmadik, vagy páros héten. És persze meg van, hogy hány órakor kezdődik. De azárt vanank még tulajdonságai egy misének mint a nyelve, a rítus (görög, római, tridentist, stb.), a zeneisége (gitáros, csöndes, orgonás, stb.), meg úgy egyébként lehet még bármilyen kommentár hozzá.

Van, hogy egyetlen napnak szeretnénk egyedi miserendet adni. Pl. dec 31. Ilyenkor ez is egy időszak, aminek az eleje és a vége is ugyan az a nap.

Az church_id-időszak-nap-idő sem kell unique legyen, mert a "milyen"-ség és "nyelv" is lehet különböző. Pl. téli időszakban minden első hétfőn  a hatos mise latinul, van más hetekben epdig orgonás az a hatos mise.

| mezőnév      | típus        | nullable | leírás  |
|--------------|--------------|----------|---------|
| tid          | int(5)       | Igen     |  a templom azonosítója, foreign_key |
| nap          | int(1)       | Nem      |  0 = vasárnap, 1,2,3,4,5,6 |
| ido          | time         | Nem      |  Amikor kezdődik a szertartás. Óra perc elég lenne nekünk. |
| nap2         | varchar(4)   | Igen     |  Ha nem minden héten van az adott napon mise, csak bizonyos hetekben, akkor ide kerül valami. Ha egy szám, akkor a hónapnak abban a hetében csak. Ha -1, akkor a hónap utolsó hetében. Ha ps vagy pt, akkor csak páratlan vagy páros hetekben. A páros/páratlan kezdete sajnos nicns jól meghatározva. Mert ha csak a hónapra nézzük, akkor két hónap találkozásakor lehet két páratlan együtt. inkább az év hetének párossága számítson. Van hogy az értéke '0' pedig az NULL kéne legyen. |
| idoszamitas  | varchar(255) | Igen     | Az időszaknak a megnevezése. Például, hogy téli időszámítás, vagy őszi időszámítás. De lehet ez kicsinyugszika is, nincs meghatározva és nem foreign key. Pedig az alábbi hat mező az mindig ugyan az kell legyen azonos időszámítás érték esetén azonos templomnál. De két különböző templomnál lehet tök külön jelentése az "őszi időszámítás"-nak. Mert ott nem óraátállítással számol, hanem iskolai szünettel. De ez nem okos dolog. Szóval egész külön tábla lehetne valami összekötő azonosítóval. De jobban tetszik a #144-ben leírt xml struktúra.  |
| weight       | int(11)      | Igen     | Az időszakokat súlyozni kell, mert lehetnek átfedések. Pl. csinálunk egy "egész évben" időszakot és aztán egy "karácsonykort".  |
| tol          | varchar(100) | Igen     | Az időszak kezdete. Ez lehet konkrét dátum: 2024-12-01, de azt ritkán szeretjük mert másik évben máskor lesz. Lehet ez hónap-nap: 12-25, azaz karcsonykor mindig ez legyen. De lehet ez egy évenként változó valami is, például: "Őszi óraátállítás", vagy "advent I. vasárnapja". Ezek nem meghatározott szövegek jelenleg, hanem bármit be lehet írni. De hogy jól működjön utána az `events` táblába be kell írni az adott évre a helyes feloldását ennek a szövegnek. Kis varázslat az, hogy lehet olyat írni, hogy "Őszi óraátállítás +4" azaz az óraátállítás után 4 nappal lépjen ez életben. -- Megjegyzés: a miserend jogú adminok látják az összes ilyen beírt szöveget és azt hogy melyiket hányszor használják, és neki kell a dátumokat beírni előző, adott, és következő évre. Ezeket az értékeket elemezve jutottam oda, hogy a #144-ben definiáltam, hogy véges ilyen szöveg legyen |
| ig           | varchar(100) | Igen     | Az időszak vége. minden kommentár mint a "tol" részben |
| tmp_datumtol | varchar(5)   | Igen     | A "tol" mező nem egy konkrét dátum, ezért minden szerkesztés után, ill. cron_jobban egyébként is, rendszeresen kiszámoljuk hogy adott naptári évre mit jelent pontosan a "tol" és az "ig" értéke. Az "adott naptári évben" nem mindig egyszerű, mert ha a "télen" időszak/időszámítás kezdetek az első tanítási nap, akkor dec 31-ig ez az adott év szeptember 1, de januárban mar az előző év szeptember 1, ám júliusban már az adott év szeptember elseje (ami már a következő tanév). Ó jaj. Szóval végül ide egy hónap-nap kerül mindig. És aztán a tmp_relation segít. |
| tmp_relation | char(1)      | Igen     | Az előző problematikára reagál a kacsacspr, hogy adott időszaknak a határai a naptári éven átnyúlás okán most hogyan viszonyulnak egymáshoz. Fura egy izé, próbálom megérteni újra. :D  |
| tmp_datumig  | varchar(5)   | Igen     | A tmp_datumtól párja. Megjegyzéseket lásd ott.  |
| nyelv        | varchar(100) | Nem      | Ide jöhet arról információ, hogy milyen nyelvű a szentmise. Az alapértelmezett - ha nincs ide írva semmi - az az hogy Magyarországon (orszag.id=12) a nyelv magyar, idegen országban a nyelv a helyi nyelv. De olykor túlbuzgóságból be voltak írva kézzel is magyarországi misére, hogy a mise nyelve magyar. Ami felesleges és vannak legacy scriptek amik ezeket újra és újra rendbetették. Szóval két betűs nyelvi kód jön ide. Ám lehet olyat, hogy egyik héten ilyen és a másik héten olyan a nyelv. Ezért ki lehet egészíteni a "nap2" logikájával. Például "va2" azt jelenti, hogy a hónap 2. hetében latinul van a mise (egyéb hetekben pedig a helyi nyelven). (Hát igen, va mint latin... gratulálok...). De olyat is lehet, hogy "hups,dept" azaz páros héten magyarul van a mise, páratlan héten meg németül. Persze ennek csak akkor van értelme így megírni, hogy szlovákiában van templom. Magyarországi templonál a "dept" ugyan ezt eredményezi. Szóval lényeg, hogy vesszővel elválasztva több szabály is megadható. Ez alehetőség igazából nem jó ötlet szerintem. Most ha első héten magyar mise van szlovákiában és különben német, akkor lehet olyat, hogy nap=0;nap2=Null;nyelv=hu1,de2,de3,de4,de5 VAGY lehet hogy több sor misét hozunk létre: nap=0;nap2=1;nyelv=hu majd nap=0;nap2=2;nyelv=de stb. Ugyan az előbbivel kevesebb sorban meg lehet adni az azonos misét, de ettől az egésztől sokat bonyolódik az egész kereső rendszer. Tovább lehet hibás dolgokat megadni: nap=0;nap2=ps;nyelv=hupt -> Hiába adom meg, hogy páratlan héten magyar nyelvű a mise, ha cska páros héten van egyáltalán mise. Juj     |
| milyen       | varchar(50)  | Nem      | Itt különféle tulajdonságait lehet megadni a misének. Mint pl, hogy görögkatolikus a mise, vagy hogy orgonás, vagy hogy diákmise. Ezeket vesszővel (space nélkül) kell elválasztani egymástól, és csak bizonyos dolgok lehetnek itt. Ám mindegyikhez mehet egy nap2 típusú kiegészítő (1,2,3,4,5,-1,ps,pt). Ez hozza magával a "nyelv"-hez írt problematikát. Továbbá vannak tulajdonságok amik ütik egymást tehát nem lehetnek egyszerre (görögkatolikus, római katolikus, tridenti), de vannak amik megférnek egymás mellett. Eredete ennek az, hogy ezek mind szövegként voltak tárolva, hol ilyen hol olyan helyesírással. A "milyen" mező révén egységesek lettek és kerehetőek. Fontosak ezek, de a "nap2" típusú részt kiírtanám belőle és inkább több mise sor legyen. Azt hiszem.   |
| megjegyzes   | text         | Nem  |  Full szöveges kommentár. Gyakran örökletesen a "milyen" rész megfogalmazása, de máskor olyanok, hogy "azé' nem minden héten" vagy "a kápolnában van" vagy "szentmise vagy igeliturgia valamilyen rendszer szerint" vagy "ez az Egyetemi lelksézség miséje is" -- Amikor valami nagyon gyakran előfordul, akkor az átmehet a "milyen" részbe. Pl. a "szentmise vagy igeliturgia" az át kéne menjen valamilyen formában. |
| modositotta  | varchar(20)  | Nem      | Általában az a felhasnzáló azonosító (user.login) aki a legutóbbi változtatást véghezvitte. No nem ebben az egy sorban, hanem az ehhez a templomhoz tartozó misék közül bármelyikben. Vagyis, ha a templomi felületen a miserendek oldalon átírt bármit! Ezért is kéne egyben tartani minden miseadatot és nem sok sorban. Vigyázat, nem foreign-key ez, mert lehet "AUTO" is az érték ami azt jelenti, hogy valamelyik cron_job tette a dolgát és módosított egyet. (Azt nem tudom, hogy a cron_job soronként módosít vagy templomonként) |
| moddatum     | datetime     | Nem      | Az előbb emlegetett módosítás dátuma. jaaaj, itt is ott van #174 avagy 0000000-00-00 00:§00 helyett NULL kéne  |
| torles       | datetime     | Nem      |  Ha törölték ezt a konkrét misét, akkor sehol nem jelenik meg többé. Azért jött létre mert így lehetne követni a miserend változásokat. De igazából nem releváns, mert csak akkor tudnánk tényleg követni ha időpont változás esetén törlődne a régi sor és létrejönne egy új, de nem. Amúgy meg nem egyes miséket szeretnénk összehasonlítani hanem egyes templomok teljes miserendjét. |
| torolte      | varchar(20)  | Nem      | A törlő felhasználó azonosítója. Azért sem foreign_key mert szreintem felhasnzáló törlésénél is megmarad ez itten emlékbe. Azt hiszem.  |

## orszagok
Mezők: id, nev, telkod, ok, kiemelt

Leginkább a `templomok` tábla használja, arra, hogy a `templomok.orszag` helyére beillessze e megfelelő `orszagok.nev` részt. Valamint a misézőhely szerkesztésénél legördülőből ki lehet választani az országot. Ám ez nem lesz helyes mert OSM adatból érjük majd el ezt is, ugyebár.
Nagyon sok helyen teszünk különbséget magyarországi és külföldi templomok között. Például másképp jelenhet meg a név. Vagy statisztikáknál is a fókusz a hazai templomokon van. Magyarország a 12-es. 

Szívem szerint leépíteném és kiiktatnám ezt.


## remarks
Ezek a befutott szöveges javítási javaslatok, észrevételek amik az egyes templomkhoz befutnak.

| name | type | description|
| :---         |     :---:      |          :--- |
| nev | rövid szöveg, akár üres | az észrevételt beküldő teljes neve. A honlapon keresztül küldött észrevételkor kerül ide. Vagy ha regisztrált felhasználóként küldi be. ha mobilról küld észrevételt akkor üres.  |
| login | rövid szöveg, akár üres | ha az észrevételt regisztrált felhasználó küldi be akkor neki az azonosítója (felhasználóknak a felhasználónév az elsődleges azonosító nem valamifél user_id. sajnos?) De jelenleg nem foreign_key, mert a felhasználó törlésekor is megmarad itten |
| email | rövid szöveg, akár üres | az észrevételt beküldő teljes neve. A honlapon keresztül küldött észrevételkor kerül ide. Vagy ha regisztrált felhasználóként küldi be. ha mobilról küld észrevételt akkor üres. Az észrevételekre való reakció szempontjából fontos, hogy tudjuk értesíteni, hogy tökre köszi  |
| mebízható |  enum('?', 'i', 'n', 'e') | Arra lett kitalálva, hogy a szerkesztő kapjon visszajelzést arról, hogy egy észrevételt beküldő személy mennyire tűnik megbízhatóank. Pl. hogy ha sokszor ír hülyeséget, akkor jobban utána kell neki nézni. A szerkesztő tud kattintani, hogy ő megbízható (i), vagy hogy ő nem megbízható (n). Email cím alapján akkor mások is megkapják ezt a flag-et. De ez a flag nem kapcsolódik az userekhez, csak itt. Igazából sosem használtuk igazán. Lehet törölni és egyszerább lesz minden |
| church_id | integer, nem üres, foreign_key(templomok.id) | hogy melyik misézőhelyhez tartozik ez az észrevétel |
| allapot |  enum('u', 'f', 'j') | Az észrevételekkel foglalkozunk. Ha "ú(j)", akkor még senki nem kezdett vele semmit. Ha "f(olyamatban)" akkor már tettünk vele valamit, de még nincs befejezve. ha "j(avítva)" akkor lezárva a dolog, több dolog nincs vele. Ezek fontos különbségek, mert így keresünk rá újakra meg ilyenek. Lehet olyat, hogy javítva-ból visszaállítjuk újra.  |
| admin | rövid_szöveg, foreign_kkey(user.login), akár üres | a felhasználó azonosítója aki a legutóbbi módosítást végrehajtotta ezen az észrevételen |
| admindatum | datetime vagy hasonló, akár üres | a legutóbbi módosítás időpontja az észrevételen. talán (?!) már nem használjuk, hanem az updated_at mezőt |
| leiras | hosszú szöveg, sosem üres | no ez maga az észrevétel, egy jó kis szöveg. lehet benne néhány html tag is. a mobil appok amikor észrevételt küldenek, akkor ennek a szövegnek a végére illesztenek be információkat |
| adminmegj | text, lehet üres | amikor egy adminisztárot változtat az észrevétel állapotán például, akkor be tudja írni, hgoy "áh semmi fontosat nem kért". sokszor nem mutatjuk a befejezet észrevételeket az adminoknak csak röviden azt amit az adminmegj-be írtak. illetve amikor válasz emailt ír egy admin ilyen olyan sablon alapján, akkor ennek a mezőnek a végébe kerül bele a gép által egy kis kiegészítés, hogy email ment, ezen sablon szeinr, és ez az email azonosítója. egész sok ilyen email lehet egyetlen észrevételhet |
| log | hosszú szöveg | upsz, nem tudom. vigyázat! lehet hogy az adminmegj-jez írt dolgok ide tartoznak, de lehet kuka|
| created_at | hosszú szöveg | laravel szereti az ilyet, de amúgy hasznos |
| updated_at | hosszú szöveg | laravel szereti az ilyet, talán mi is használjuk ezt |




## service_hours
Az egyes misézőhelyekhez tartozó miserendnek az általános leírása és verzió követése. Ebből lehet levezetni majd, hogy egy konkrét napon mikor vannak szentmisék és mikor nincsenek.



| name | type | description|
| :---         |     :---:      |          :--- |
| church_id   | integer, non_null   | a misézőhely azonosítója (mint az url-ben)    |
| service_hours     | valami hosszú szöveg, lehet üres       | Egy megfelelő XML formátumban megadva, hogy milyen időszakban milyen misék vannak. A szintakszis leírását lást: [#144](https://github.com/borazslo/miserend.hu/issues/144)    |
| ical   | string/varchar, lehet üres  | Egy hosszú url ami egy ical formátumú calendar fáljra mutat. Ha ez ki van töltve (és működik), akkor a konkrét nap konkrét miséit ebből generáljuk és a `service_hours`-t csak szépen megjenítjük|
| notes vagy hasonló  | nem túl hosszú szöveg   | kiegészítő szöveges megjegyzés. pl.: "péntekenként az alagsoron át lehet bejutni amisére. Korábban ez volt a `templom.misemegj` |
| user_id | integer, non_null | A felhasználó azonosítója, aki ezt a miserendet létrehozta (módosítván az előző verziót). Javítási javaslatnál is a jóváhagyó azonosítója és nem a beküldőé (aki vagy van, vagy nincs) |
| last_confirmation / confirmed / valami hasonló | datetype vagy timestamp vagy hasonló, nem lehet üres | Legtöbb helyen csak az év-hónap-nap formában írjuk ki. Ez az nap amikor létrejött ez a miserend, vagy amikor később egy illetékes kijelentette, hogy ez még mindig a helyes időpont |
| change_notes vagy hasonló | nem túl hosszú szöveg, lehet üres | Belső megjegyzés, ami a szerkesztői felületen látszik csak. Itt lehet a változásról mondani valamit. Pl.: "minden esti mise megszűnt" vagy "egy mebízható betelefonáló kérésére módosítottuk". Ennek az API-ban talán ki sem kell menjen, mert az csak archiv szempontjából izgalmas. |

Egyetlen church_id -hez több sor is tartozik, az idők során egész sok. Mert minden változás a service_hours / ical / notes mezőkben az magával rántja, hogy új sor jöjjön létre a táblába. Egy misézőhely aktuálisan érényes mierend leírását úgy kapjuk meg hogy a táblát időrendbe helyezzük és az adott church_id-t lekérdezve az ELSŐ eredményt vesszük figyelembe.

Kapcsolódások: a kintről beérkezett miserend javaslatokat, amikben már konkért xml-t küldenek át átnézésre, azokat külön táblába érkeztetjük és kezeljük.

## user
A felhasználók adatait tartalmazó tábla. jé.

|               |              |  null    |   |
|---------------|--------------|------|---|
| login         | varchar(20)  | Nem  |  felhasználó név, ez az elsődleges id, ez alapján azonosítunk mindenkit. természetesen egyedi. Soha, de soha nem módosítjuk, mert sok helyen kéne módosítani sok mindent. Ráadásul nem csak mezőkben, hanem csetben és talán máshol szöveges mezőbe is bekerül ez a név, és ezért elröthetnek linkek. De cseten kívül nem tudom hogy valójában van-e valahol gond. |
| jelszo        | varchar(255) | Nem  |  Naná hogy kódolva: sózott md5 hash. A user joggal rendelkező admin meg tud adni új jelszót, de régit ő sem látja. Jelszó emlékeztető email esetén kap egy random új jelszót amit elküldünk emailben. (jaj) Aztán reméljük hamar megváltoztatja. |
| jogok         | varchar(200) | Nem  |  a különféle jogai a felhasználónak amiket egy-egy rövid karaktersorozat, ezeket egymástól kötőjelekkel választjuk el. miserend: minden templom minden adatát módosíthatja, user: felhasználók adatait is módosíthatja, meg törölhet. Más most már talán nincs. Egyes templomokat még a templomfelelősök módosíthatnak. Valamint az egyházmegyék felelőse hozzáfér az egyházmegyéje minden templomához. Templomot létre jeleneg senki nem tud hozni. A felhasználók lemondhatnak user / miserend jogukról, de akkor nem tudják visszavenni ugyebár.  |
| regdatum      | datetime     | Nem  | Regisztráció dátuma. Hát vannak naaagyon régiek  |
| lastlogin     | datetime     | Nem  | Legutóbbi belépése. vigyázat, mert igazából fontos hogy lehessen null. regisztráció után például ha nem lépett be egyből, akkor ez null. Meg akkor is, ha 2015 óta nem lépett be... Elvileg az egyik cron job éjszakánként emlékeztetőket küld a régen vagy soha be nem lépetteknek és adott idő után (90 nap?) törlni is azokat   |
| lastactive    | datetime     | Igen | Amikor legutóbb az emberünk bármit csinált. Kattintott, tovább lépett, módosított, bármit. Ezt igazából csak a chat használja. ott segít, hogy jé, éppen mostanság itt van ő is. |
| email         | varchar(100) | Nem  | Valid email cím. Regisztrációkor erre megy egy email és abban van az első random jelszó amivel be tud lépni. Vagyis egy ideje csak validált email címmel tudnak belépni. Sajna textként elküldjük emailben a jelszót és reméljük hogy majd módosítja. Sok-sok régi felhasználónál nincs ellenőrizve az email cím. De szépen lassan kihalnak ők.  |
| notifications | 1 vagy 0  | Igen | Felhasználó a profilján állíthatja, hogy kapjon-e értesítő emaileket vagy nem. A templomok gondnokai (megy egyházmegye felelősei, meg miserend jogosultsággal rendelkezők) kapnak emaileket, ha jön észrevétel vagy jön kép. Később lehetne, hogy a kedvencelt templomok változásairól is menjen automatikus üzenet, meg ilyenek. Fél évente megy még üzenet, hogy légyszi nézz rá a templomodra. Az inaktivitás kapcsán mindig megy email notification 0 esetén is.   |
| becenev       | varchar(50)  | Nem  |  Regisztárciókor meg kell adni. De minek? Vonjuk össze a 'nev' mezővel és halál erre. |
| nev           | varchar(100) | Nem  | Regisztrációkor meg kell adni, általában teljen nevet használnak az emberek. Ezzel szólítjuk meg az emailekben például. |
| volunteer     | int(1)       | Nem  | Van egy évek óta hasnzálatlan és már elavult script ami kiválogat régen frissült templomokat és azokat átküldi hetente egyszer azoknak az önkénteseknek, akik vállalják hogy utána néznek pár templomnak. Ügyes a script mert figyel az átfedésekre és egyebekre is. Egy adventben működött és kb. 30 emberrel 500+ templom adatát frissítettük. Ennek a flag-je ez. A felhasználó tudja kapcsolgatni. Ha a "volunteer kampány" nincs bekapcsolva, akkor ez ne is jelenjen meg a profil oldalon szerintem.  |

