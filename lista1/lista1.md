### Technologie Sieciowe
## Lista 1
##### Mateusz Trzeciak, 2021

---


1. 
    Przetestuj działanie programów:
    a) Ping: Sprawdź za jego pomocą ile jest węzłów na trasie do (i od) wybranego, odległego geograficznie,serwera. Uwaga: trasy tam i z powrotem mogą być różne. Zbadaj jaki wpływ ma na to wielkość pakietu. Zbadaj jak wielkość pakietu wpływa na obserwowane czasy propagacji. Zbadaj jaki wpływ na powyższe ma konieczność fragmentacji pakietów. Jaki największy niefragmentowany pakiet uda się przesłac. Przeanalizuj te same zagadnienia dla krótkich tras (do serwerów bliskich geograficznie). Określ "średnicę" internetu (najdłuższą sćieżkę którą uda sie wyszukać). Czy potraficz wyszukać trasy przebiegające przez sieci wirtualne (zdalne platformy "cloud computing"). Ile węzłów mają scieżki w tym przypadku.
    b) Traceroute,
    c) WireShark.
2.    Napisz sprawozdanie zawierające: opis programów, wywołania dla powyższych zagadnień z analizą wyników, wnioski dotyczące przydatności tych programów.


---

#### 1. Ping

###### 1.1 Opis programu

Ping jest narzędziem dostępnym w większości systemów operacyjnych, służącym do diagnozowania połączeń sieciowych **TCP/IP** za pomocą protokołu **ICMP**, a konkretnie wysyłanych sygnałów *Echo Request* i odbieraniu sygnałów *Echo Reply*. Za ich pomocą mierzone są także opóźnienie połączenia i liczby zgubionych pakietów.

Warto zaznaczyć, że nieudane próby połączenia z serwerami tą metodą nie są jednoznaczne z tym, że nie możemy połączyć się z danym serwerem w ogóle. W trakcie nauki odkryłem, że wyszukiwarka pod adresem *duckduckgo.com* nie pozwala na testowanie połączenia tą metodą. Rzekomo jest to zabezpieczenie przed atakami DDOS.


###### 1.2 Przykład użycia

![logo](/img_1.png)

Próba połączenia z serwerem strony Politechniki Wrocławskiej. Po wysłaniu pięciu pakietów, nie otrzymano żadnego. Może to oznaczać bład sieci, serwera DNS, obecność firewalla blokującego połączenie lub przeciążenie serwera na którym hostowana jest strona politechniki.

![logo](/img_2.png)

Próba połączenia z serwerem hostujący portal miasta Wrocław. Poniżej opis poszczególnych elementów komunikatu.

- **icmp_seq** - Sequence number, czyli ID wysłanego pakietu
- **ttl** - Time to live/hop limit. Określa limit "skoków" które może wykonać pakiet pomiędzy poszczególnymi urządzeniami. Idea jest prosta, bez takiego limitu pakiet mógłby krążyć po sieci w nieskończoność. Jest elementem protokołu IP. Przy każdym skoku, wartość ta zmniejsza się o 1. W tym przykładzie pakiety po dotarciu do nas mogą wykonać jeszcze 54 skoki.
- **time** - określa czas w milisekundach który minał od wysłania do otrzymania wysłanego pakietu.

###### 1.3 Test połączenia z wybranymi serwerami

Ilość skoków do serwera obliczana jest, korzystając z flagi -t i wybieraniu najmniejszego TTL zdolnego połączyć się z serwerem. Skoki pakietu powracającego obliczane są wg TTL które posiadał pakiet (należy założyć, jaką wartość bazową posiadał, dla poniższych przykładów przyjąłem 64).

| Adres         |lokalizacja |  ilość skoków do | ilość skoków z |
| ------------- |:---------:|:-------------:| :-----:|
|www.wroclaw.pl |Gdańsk     | 12 | 10 |
| lukesmith.xyz |New York   | 18 |   17             |
| en.stju.edu.cn |Shanghai           | 32      |    32 |

---

Następna tabela pokazuje jak trasa, ttl oraz lag zmienia się w zależności od wielkości pakietu.

| Adres         |pakiet|lokalizacja |  ilość skoków do | ilość skoków z |czas (ms) |
| ------------- |:-:   |:---------: |:-------------:   | :-----:        |:--: |
|www.wroclaw.pl |  64  |Gdańsk      | 12               | 10             |27.2 |
|www.wroclaw.pl |2048  |Gdańsk      | 12               | 10             |29.6 |
|www.wroclaw.pl |32768 |Gdańsk      | 12               | 10             |36.6 |
| lukesmith.xyz |  64  |New York    | 18               |  17            |112  |
| lukesmith.xyz | 2048 |New York    | 18               |  17            |123  |
| lukesmith.xyz | 32768|New York    | 18               |  17            |134  |
| en.stju.edu.cn|64    |Shanghai    | 32               |    32          |365  |
| en.stju.edu.cn| 2048 |Shanghai    | 32               |    32          |373  |
| en.stju.edu.cn| 32768|Shanghai    | 32               |    32          |377  |

Okazuje się, że wielkość znaczenia nie ma wpływu na trasę. Ma natomiast nieznaczy wpływ na opóźnienie połączenia. 

---

Poniżej przedstawiony jest największy pakiet, który udało mi się przesłać:

![logo](/img_5.png)

---

Przetestujmy teraz czas propagacji pakietów z flagą DF. 

Testując połączenie z serwerem www.wroclaw.pl największy pakiet który udało mi się przesłać z flagą DF wynosił 1472 bajty.

Poniższa tabela prezentuje różnice w średnim opóźnieniu oraz ilości skoków w przypadku włączonej i wyłączonej flagi DF

|Adres|avg|avg (DF)|TTL|TTL (DF)|
|:---:|:-:|:------:|:-:|:------:|
|www.wroclaw.pl| 26| 25.9| 10|10|
|lukesmith.xyz|114|114|17|17|
|en.stju.edu.cn|367|376|32|32|

Okazuje się, że fragmentacja pakietów nie wpływa na ich trasę, a różnice w prędkości mieszczą się w granicach błędu pomiarowego.




#### 2. Traceroute

Narzędzie służące do prześledzenia trasy pakietu do konkretnego serwera. Działa na zasadzie inkrementacji wartości TTL o 1 i odbieraniu komunikatu ICMP "time exceded".

##### 2.1 Przykład użycia

![logo](/img_3.png)

Pierwsza linijka opisuje detale przeprowadzanego połączenia. Kolejne opisują połączenia z konkretnymi serwerami, oraz czas otrzymania odpowiedzi od trzech pakietów wysłanych do tego serwera, z tym samym TTL. Brak odpowiedzi na zadany pakiet jest oznaczany gwiazdką (*). W tym przykładzie, nic takiego jednak nie wystąpiło.


#### 3. Wireshark

Wireshark to popularne narzędzie do przechwytywania, monitorowania i dekodowania pakietów danych przesyłanych w sieci. Zapewnia ogrom narządzi do realizacji wymienionych celów, z których najważniejszym jest możliwość filtowania pakietów tak, aby wychwycić tylko te, które nas interesują. Osobiście, pozwoliło mi to kiedyś prześledzenie pakietów wysyłanych przez portal internetowy, który w tle korzystał z zasobów mojego komputera do kopania kryptowalut.

![logo](/img_4.png)