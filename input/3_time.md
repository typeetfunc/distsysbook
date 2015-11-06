# %chapter_number%. Время и порядок

Что означет для нас порядок и почему он важен?

Что мы хотим сказать этим вопросом?

Почему мы так фокусируемся на порядке в первую очередь? Почему нам важно знать что A случилось раньше B? Почему мы не беспокоимся о другом свойстве, таком как цвет?

Хорошо, мои сумасшедшие друзья, давайте вернемся назад к определению распределенной системы чтобы ответить на этот вопрос.

Как мы можем помнить, распределенное программирование описывалось как искуство решать тех же задач что вы решаете на одном компьютере на многих компьютерах.

Этот факт - основа нашей одержимости идеей порядка. Любая система, которая может делать только одну операцию в один момент времени создает абсолютный порядок операций. Подобно людям проходящим через одну дверь, каждая операция имеет определенных предшественника и последователя. Это базовая модель для программирования и потрачено много усилий чтобы она была сохранена.

Традиционная модель это: одна программа, один процесс, одно пространство в памяти, один вычислительный процесс. Операционная система абстрагирует, пряча от нас, тот факт что может быть много CPU или много программ, и память на самом деле может быть разделена между программами. Мы не говорим, что несуществует программ использующих множество потоков или событийно-ориентированную парадигму; просто это все специальная абстракция поверх модели "one/one/one"(одна программа, одна память, один CPU). Программы написаны чтобы исполнятся в определенном порядке: исполнение начинается с верхней строчки и переходит вниз до самой нижней.

Порядок, как свойство получает так много внимания, потому что простейший способ доказать "корректность" работы это сказать "это работает также как и работало бы на одной машине". Ичто обычно означает что а) мы запускаем на всегда одни и теже операции и б) мы их запускаем в одинаковом порядке - даже запуская программу на нескольких машинах.

Хорошим свойством для распределенной системы, которая сохраняет порядок операций, является то что она универсальна. Вам можно не думать о том что за операции надо выполнять с ее помощью, потому что они будут выполнены как будто на одной машине. Это великолепно потому что вы можете использовать одну и ту же системы для различных операций.

В реальности, распределенные программы запускаются на распределенных узлах; с многими CPU и многими потоками операций вместе с тем. Вы попрежнему можете назначать абсолютный порядок операций, но это требует либо точных часов или особых форм сообщения. Вы могли бы присвоить каждой операции временную метку используя абсолютно точные часы, чтобы определить глобальный порядок. Или вы можете какуюто систему коммуникаций между узлами которая может сделать возможным назначать глобальные порядковые номера для операций.

## Абсолютный(полный) и частичный порядок

Естественное состояние для распредленной системы это [частичный порядок](http://en.wikipedia.org/wiki/Partially_ordered_set). Ни сеть ни независимость различных узлов не позволяют намм говорит о каких либо гарантиях относительно порядка; но для каждого узла вы можете соблюдать локальный порядок.

[Абсолютный(полный) порядок](http://en.wikipedia.org/wiki/Total_order) это бинарное отношение, которое определяет порядок для каждого элемента в некотром множестве.

Два уникальных элемента являются **сравниваемыми** когда один из них больше другого. В частично упорядоченном множестве, некотрые пары элементов не являются сравниваемыми и следовательно частичный порядок не указывает точный порядок следования для каждого элемента.

И абсолютный и частичный порядок [транзитивны](http://en.wikipedia.org/wiki/Transitive_relation) и [антисиметричны](http://en.wikipedia.org/wiki/Antisymmetric_relation). Следующие утверждения верны и для частичного и абсолютного порядка для всех a, b, c из X:

    Если a ≤ b и b ≤ a тогда a = b (антисиметричность);
    Если a ≤ b и b ≤ c тогда a ≤ c (транзитивность);

Однако, абсолютный порядок также является [полным отношением](http://en.wikipedia.org/wiki/Total_relation):

    a ≤ b или b ≤ a (полнота) для всех a, b в X

в то время как частичный порядок обладает только [свойством рефлексивности](http://en.wikipedia.org/wiki/Reflexive_relation):

    a ≤ a (рефлексивность) для всех a в X

Заметим что полнота включает в себя рефлексивность; то есть частичный порядок более слабый вариант полного порядка.
Для некотрых элементов в частичном порядке, свойство полноты не будет выполнено - иными словами, некотрые элементы оказываются не сравнимыми.

Ветки Git это пример частично упорядоченного порядка. Как вы возможно знаете, система контроля ревизий git позволяет создавать множество ветвей из одной базовой ветки - например из мастер ветки. Каждая ветвь представляет собой отдельную историю изменений исходного кода основанную на общем предке:

    [ branch A (1,2,0)]  [ master (3,0,0) ]  [ branch B (1,0,2) ]
    [ branch A (1,1,0)]  [ master (2,0,0) ]  [ branch B (1,0,1) ]
                      \  [ master (1,0,0) ]  /

Ветви A и B были получены из общего предка, но нигде неопределенн порядок между ними: они представляют разные истории и не могут быть сведены к одной последовательной истории без дополнительной работы(мерджа). Конечно можно попытатся расположить коммиты в определенном порядке (скажем сортирую сначала по порядку в предке а затем расположить все измения из А и только потом из Б - или наоброт) - но это будет сопровождатся потерей информации так как вы пытаетесь имитировать порядок там где его нету.

В системе, состоящей из одного узла, абсолютный порядок возникает неизбежно: так как инструкции исполняются а сообщения отправляются в определенном порядке. Мы можем положится на абсолютный порядок вычислений - он делает выполнение программ предсказуемым. Такой порядок может поддерживаться и в распределенной системе, но это будет иметь свою цену: коммуникации крайне дороги, а синхронизация времени крайне сложна и хрупка.

# Что такое время?

Время это источник порядка - оно позволяет нам опрделить порядок выполнения операций - который по случайному совпадению может пониматься людьми (секунды, минуты и.т.д).

В некотром смысле, время похоже на любой другой счетчик. Достаточно важно, что большинство компьютеров имеет встроенный сенсор времени, известный как часы. Это настолько важно что человечество научилось синтезировать такой счетчик с некотрой погрешностью используя некотрые физические обьекты (от восковой свечи до атома цезия). Под "синтезировать" понимается что мы можем получить приблизительное значение счетчика в двух физически отдаленных местах используя некотрое физическое свойство без прямой коммуникации.

Временная метка это сокращенное значение для представления состояния мира с появления вселенной до текущего момента - если чтото происходит в определленый момент времени, тогда на это могло повлиять все что произощло до этого момента. Эта идея может быть обобщена до часов на основе причинных связей, которые явно отслеживают зависимости событиями, а не просто предпологают что все произошедшее до определнного события связано с ним. Конечно обычно предпологают, что мы должны беспокоится только о состоянии конкретной системы а не о состоянии всего мира.

Если предположить, что время движется везде с одинаковой скоростью - и это крайне большое предположение к оторому мы еще вернемся -  время и временные метки могут интерпритироватся несколькими полезными нам способами, как:

- Порядок
- Продолжительность
- Интерпритация

*Порядок*. Когда мы говорили, что время это источник порядка имелось ввиду что:

- мы можем присваивать временные метки на неупорядоченные события и тем самым упорядочить их
- мы можем использовать временные метки для обеспечения конкретного порядка операций или доставки сообщений(например путем задержки операции если она поступила не в свою очередь)
- мы можем использовать значение времени чтобы определить, что предшествовало в хронологическом порядке какому либо событию

*Интерпритация* - время это универсальное сравниваемое значение. Абсолютное значение времени может интерпретироватся как дата, что может полезно для людей. Учитывая временную метку начала даунтайма из логов мы можете сказать, что он произошел в прошлую субботу во время  [thunderstorm](https://twitter.com/AWSFail/statuses/218915147060752384).

*Продолжительность* - продолжительность измеряемая во времени имеет отношение к реальному миру. Алгоритмы как правило не беспокоятся о абсолютном значении часов или интерпритации их как даты, но продолжительность может помочь сделать программе несколько полезных выводов. К примеру, по времени потраченому на ожидание ответа можно предположить является система разделенной или просто велики задержки коммуникации.

Природа компонентов распределенных систем такова что они не могут вести себя предсказуемым способом. Они не гарантируют какой либо определенный порядок, скорость или отсутсвие задержек. Каждый узел имеет некотрый локальный порядок - приблизительно последовательное выолнение - но этот локальный порядок независим от порядка других узлов.

Установление (или допущение) порядка это единственный способ сократить пространство возможных путей исполнения и путей возникновения. Людям тяжело рассуждать о вещах который могут происходить в любом порядке - так как в таком случае число возможных перстановок крайне велико - больше чем может вместить человеческий мозг.

## Везде ли одинаково течет время?

Все мы имеем интуитивное понимание времени, основывающееся на нашем личном опыте. К сожалению, это интуитивное понимание облегчает представление абсолютного(глобального) порядка, но не частичного. Гораздо проще представить последовательность, в которой события происходят одно за другим, а не одновременно. Гораздо проще рассуждать о сообщениях, поступаемых в единой последовательности, чем о сообщениях, поступающих в разном порядке и с разными задержками.

Однако, разрабатывая распределенную систему, хочется обойтись без сильных допущений касательно времени и порядка, потому что, чем сильнее допущения, тем более уязвима становится система к проблемам с временем. Более того, наложение порядка влечет за собой дополнительные расходы. Чем больше непоследовательных действий мы можем обрабатывать, тем больше возможность мы сможем получить от распределенной стистемы.

Есть три основных ответа на вопрос "везде ли время течет одинаково?":

- "Глобальные часы": да
- "Локальные часы": нет, но...
- "В отсутствии часов": нет!

Они примерно соответствуют трем временным моделям, о которых говорилось во второй главе: модель синхронной системы работает с глобальными часами, частчно синхронная система - с локальными часами, ассинхронная система вообще не может использовать часы. Давайте разберем эти предположения подробнее.

### Время в модели "глобальных часов"

Модель глобальных часов подразумевает существование абсолютно точных мировых часов, к которым есть доступ у каждого. Это то, как мы привыкли думать о времени, потому что в повседневной жизни небольшие различия во времени не имеют значения.

![Глобальные часы](images/global-clock.png)

Глобальные часы по факту являются источником общего порядка(точный порядок на каждом узле, не смотря на то, что они даже не взаимодействуют друг с другом).

Однако, это идеализированное предствление мира: на самом деле синхронизация часов возможно только с ограниченной точностью. Ограничение обусловлено отсутствием точности в часах обычных компьютеров, а так же задержкой, если используется протокол синхронизации, например такой как [NTP](http://en.wikipedia.org/wiki/Network_Time_Protocol) и кроме того принципиально ограничено [природой пространства-времени](http://en.wikipedia.org/wiki/Time_dilation).

Предположение, что часы в распределенной системе идеально синхронизированы, подразумевает собой предположение, что отсчет времени начался одновременни, а так же, что часы никогда не смогут разойтись. Это полезное допущение, потому что, с ним появляется возможность использовать временные метки без ограничений, чтобы определять глобальный общий порядок, будучи ограниченным только сдвигами времени, но не задержками - но это [не тривиальная](http://queue.acm.org/detail.cfm?id=1773943) задача и потенциальное место для аномалий. Существует множество различных сценариев, когда небольшая неполадка - например, случайное изменение пользователем локального времени на компьютере, или добавление в кластер машины с опозданием, или синхронизированные часы слегка разъехались, таким образом приводя к сложно обнаружимым аномалиям.

Тем не менее, есть некоторые системы, в которые принято такое допущение. В их числе [Cassandra](http://en.wikipedia.org/wiki/Apache_Cassandra), созданная в Facebook - это пример системы, которая допускает, что часы полностью синхронизирована. Она использует метки времени для разрешения конфликтов между конкурируещими источниками информации - актуальной считается информация, с самым поздним временем создания. Это значит, что если часы все же разъехались, по-настоящему новые данные могут быть проигнорированы или затерты; повторюсь, это задача администрирования (и насколько я слышал, люди остро понимают это). Другой интересный пример это Spanner, разработанный в Google. Данная [статья](http://research.google.com/archive/spanner.html) описывает TrueTime API, которое синхронизирует время, а так же оценивает максимально возможные сдвиги часов.

### Время в модели "локальных часов"

Второе и, возможно, наиболее убедительное допущение - это тот случай, когда каждый узел имеет свои собственные часы, которые не являются глобальными. Это означает, что вы не можете использовать локальные часы для того, чтобы определить, когда была получена отметка времени на другом узле, до или после локальной; проще говоря вы не можете сравнивать временные метки с разных узлов.

![Локальные часы](images/local-clock.png)

Допущение о локальных часах более точно соответствует реальному миру. Оно устанавливает частичный порядок: события каждого узла упорядочены между собой, но события между различными узлами не могут быть упорядочены, полагаясь только на часы.

Однако, вы можете использовать временные метки для определения порядка событий на одиночном узле; а также можете использовать таймауты(то есть опиратся на свойство "длительности времени") на одиночном узле до тех пор, пока вы контролируете часы и не позволяете им сбиться. Безусловно, для машины, контролируемой конечным пользователем, это, возможно слишком сложная задача: например, пользователь мог случайно изменить ее дату, пока просматривал ее с помощью встроенных в ОС средств.


### Время в модели "без-часов"

Наконец, существует понятие логического времени. В данном случае часы вообще не используются и вместо этого причнинно-следственная связь отслеживается каким-то другим способом. Запомните, временная метка - это просто некоторое сокращенное обозначение для состояния мира к данному моменту - поэтому мы можем использовать счетчики и средство связи, чтобы определить, что какие-то действия произошли раньше или одновременно с другими.

В этом случае мы можем определить порядок событий между различными узлами, но не можем работать с интервалами и не можем использовать временные задержки(у нас все еще нет "датчика времени"). Тут присутствует частичный порядок: события могут быть упорядочены внутри единой системы, поддержание же порядка между системами требует обмена сообщениями.

Одна из наиболее цитируемых статей, касательно распределенных систем - это статья Лампорта о [времени, часах и упорядочении событий](http://research.microsoft.com/users/lamport/pubs/time-clocks.pdf). Векторные часты - обобщение этой концепции (которая будет мной рассмотрена позже более детально). Это способ отслеживать причинно-следственные связи, не используя часы. Родственные Cassandra системы Riak (Basho) и Voldemort (Linkedin) используют векторные чаты вместо допущения о том, что все узлы имеют доступ к едиными абсолютно точным часам. Это позволяет данным системам не сталкиваться с проблемами точности часов, которые были рассмотрены ранее.

Когда часы не используются, максимальная точность, с которой могут быть упорядочены события на удаленных узлах зависит от задержки передачи сообщений.

## Как время используется в распределенных системах?

Что может дать нам время?

1. Время может определять сквозной порядок(без коммуникаций)
2. Время может определять граничные условия алгоритмов

Порядок событий важен в распределенных системах, потому что многие свойства распределенных систем определенны в терминах порядка операций/событий:

- когда корректность зависит от соглашения о корректном порядке событий, для примера свойство сериализуемости в распределенной базе данных
- порядок может быть использован, для связи переключения ресурсов, для примера если есть два обращения к виджету, тогда будет выполнен первое и отменено второе

Глобальные часы позволяют упорядочить операции на двух разных машинах без прямой коммуникации. Без глобальных часов, нам необходимо сообщатся, чтобы определить порядок.

Время также может быть использовано, чтобы определять граничные условия алгоритмов - в частности, различать "высокие задержки" и "сетевое разделение". Это очень важный вариант использования времени; в большинсве реальных систем таймауты используются для определения отказал ли удаленный сервер или в данный момент в сети просто довольно большие задержки. Алгоритмы которые могут определять это называются детекторами отказов; и вскоре о них будет вестись речь.

## Векторные часы (время для причинного порядка)

Ранее, мы обсуждали разные допущения о скорости течения времени в распределенной системе. Если предположить, что мы не можем достичь точной синхронизации времени - или мы ставим себе целью создать систему не чувствительную к рассинхронизации, как мы можем упорядочивать события?

Лэмпортовы часы и векторные часы это замена физических часов - они полагаются на счетчики и коммуникации для определения сквозного порядка событий в распределенной системе. Эти часы предоставляют счетчик, который можно сравнивать на различных узлах

*Лампортовы часы* довольно просты. Каждый процесс обладает счетчиком, который использует следующие правила:

- Всякий раз когда процесс совершает некотрую работу, он увеличивает счетчик
- Посылая сообщения процесс включает в него этот счетчик
- Когда процесс получает сообщение, счетчик устанавливается согласно формуле `максимум(локальный_счетчик, полученный_счетчик) + 1`

Выразим, это следующим кодом:

    function LamportClock() {
      this.value = 1;
    }

    LamportClock.prototype.get = function() {
      return this.value;
    }

    LamportClock.prototype.increment = function() {
      this.value++;
    }

    LamportClock.prototype.merge = function(other) {
      this.value = Math.max(this.value, other.value) + 1;
    }

[Лэмпортовы часы](http://en.wikipedia.org/wiki/Lamport_timestamps) позволяют счетчикам быть сравниваемы между узлами в системе, с оговоркой: Лэмпоротовы часы определяют частичный порядок. Если `время(a) < время(b)`:

- `a` могло произойти перед `b` или
- `a` может быть несравниваемо с`b`

Это известно как условие согласованности часов: если одно событие произошло перед другим, тогда логическое время этого событие должно быть меньше времени другого события. Если `a` и `b` пришли из одной цепочки причинно-следственной связи, например оба значения времени были получены из одного процесса; или `b` это ответ на сообщение посланное в результате `a` тогда мы знаем что `a` случилось перед `b`.

Можно догадатся, что причина этого в том что Лэмпортовы часы  могут нести в себе только информацию о одной цепочке событий / истории; следовательно, сравнение Лэмпортовых меток времени от систем, которые никогда не сообщаются друг с другом может приводить к видимости порядка между конкурентными событиями, хотя порядка между ними нету.

Представим систему, которая после инициализации разделяется на две независимые подсистемы которые никогда не сообщаются друг с другом.

Для всех событий происходящих в каждой независимой системе, если a случилось раньше b, тогда `ts(a) < ts(b)`; но если взять два события с разных независимых систем (то есть события которые не связаны причинно - следственной связью) тогда мы не можем сказать что-либо осмысленное о их порядке относительно друг друга. Каждая часть системы присваивает событиям временные метки не связанные друг с другом. Два события могут казатся упорядоченными даже когда они совершенно не связаны.

Однако - и это остается полезным свойством - на одной машине, на любое сообщение отправленное с временной меткой `ts(a)` будет получен ответ с временной меткой `ts(b)` которая будет `> ts(a)`.

*Векторные часы* это расширение часов Лэмпорта, которое представляет собой массив `[ t1, t2, ... ]` логических часов - по одному экземпляру часов для каждого узла. Вместо увеличения общего счетчика, каждый узел увеличивает его логический счетчик когда происходит какое либо внутреннее событие. Следовательно правила изменения часов таковы:

- Когда процесс выполняет работу он увеличивает значение логических часов узла в векторе
- Когда процесс отправляет сообщение, он включет в него весь вектор логических часов
- Когда сообщение получено:
  - обновляем каждый элемент в векторе, присваивая ему `максимум(локальное_значение, полученное_значение)`
  - увеличиваем значение логических часов представляющих конкретный узел в векторе

Опять таки выразим в коде:

    function VectorClock(value) {
      // представляем в виде хеша, в качестве ключей используем id узла: то есть { node1: 1, node2: 3 }
      this.value = value || {};
    }

    VectorClock.prototype.get = function() {
      return this.value;
    };

    VectorClock.prototype.increment = function(nodeId) {
      if(typeof this.value[nodeId] == 'undefined') {
        this.value[nodeId] = 1;
      } else {
        this.value[nodeId]++;
      }
    };

    VectorClock.prototype.merge = function(other) {
      var result = {}, last,
          a = this.value,
          b = other.value;
      // Фильтруем дубликаты ключей в хеши
      (Object.keys(a)
        .concat(b))
        .sort()
        .filter(function(key) {
          var isDuplicate = (key == last);
          last = key;
          return !isDuplicate;
        }).forEach(function(key) {
          result[key] = Math.max(a[key] || 0, b[key] || 0);
        });
      this.value = result;
    };

Эта иллюстрация ([источник](http://en.wikipedia.org/wiki/Vector_clock)) показывает векторные часы:

![from http://en.wikipedia.org/wiki/Vector_clock](images/vector_clock.svg.png)

Каждый из трех узлов (A, B, C) ведет векторные часы. Присвоенные к событиям значения векторных часов показывают как происходили события в системе. Рассматривая значение векторных часов такие как `{ A: 2, B: 4, C: 1 }` мы можем точно определить сообщения которые (потенциально) привели к этому событию.

TПроблемы с векторными часами возникают в основном изза того что для их реализации необходимо одно значение на один узел, это означает, чтоони могут потенциально разрастись для очень больших систем. Для сокращения размера векторных часов могут применятся различные техники - выполнение периодической сборки мусора или уменьшая размер вектора за счет снижения точности.

Мы увидели как порядок и причинно-следственные связи могут отслеживатся без физических часов. Сейчас мы увидим как длительность времени может быть использована для ограничений.

## Детекторы отказов (время для отсечки)

Как заявлялось ранее, количество времени потраченого на ожидание, может быть ключом к ответу на вопрос является ли система разделенной или просто испытывает высокие задержки. В этом случае, мы не нуждаемся в допущении идеально точных глобальных часов - это настолько просто, что надежно работает даже с локальными часами.

Предположим что программа запущена на одном узле, когда мы можем сказать что удаленный узел отказал? В отсутсвии точной информации, мы можем сделать вывод, что узлы которые не отвечают на сообщения по проществии некотрого разумного количества времени отказали.

But what is a "reasonable amount"? This depends on the latency between the local and remote nodes. Rather than explicitly specifying algorithms with specific values (which would inevitably be wrong in some cases), it would be nicer to deal with a suitable abstraction.

A failure detector is a way to abstract away the exact timing assumptions. Failure detectors are implemented using heartbeat messages and timers. Processes exchange heartbeat messages. If a message response is not received before the timeout occurs, then the process suspects the other process.

A failure detector based on a timeout will carry the risk of being either overly aggressive (declaring a node to have failed) or being overly conservative (taking a long time to detect a crash). How accurate do failure detectors need to be for them to be usable?

[Chandra et al.](http://www.google.com/search?q=Unreliable%20Failure%20Detectors%20for%20Reliable%20Distributed%20Systems) (1996) discuss failure detectors in the context of solving consensus - a problem that is particularly relevant since it underlies most replication problems where the replicas need to agree in environments with latency and network partitions.

They characterize failure detectors using two properties, completeness and accuracy:

<dl>
  <dt>Strong completeness.</dt>
  <dd>Every crashed process is eventually suspected by every correct process.</dd>
  <dt>Weak completeness.</dt>
  <dd>Every crashed process is eventually suspected by some correct process.</dd>
  <dt>Strong accuracy.</dt>
  <dd>No correct process is suspected ever.</dd>
  <dt>Weak accuracy.</dt>
  <dd>Some correct process is never suspected.</dd>
</dl>

Completeness is easier to achieve than accuracy; indeed, all failure detectors of importance achieve it - all you need to do is not to wait forever to suspect someone. Chandra et al. note that a failure detector with weak completeness can be transformed to one with strong completeness (by broadcasting information about suspected processes), allowing us to concentrate on the spectrum of accuracy properties.

Avoiding incorrectly suspecting non-faulty processes is hard unless you are able to assume that there is a hard maximum on the message delay. That assumption can be made in a synchronous system model - and hence failure detectors can be strongly accurate in such a system. Under system models that do not impose hard bounds on message delay, failure detection can at best be eventually accurate.

Chandra et al. show that even a very weak failure detector - the eventually weak failure detector ⋄W (eventually weak accuracy + weak completeness) - can be used to solve the consensus problem. The diagram below (from the paper) illustrates the relationship between system models and problem solvability:

![From Chandra and Toueg. Unreliable failure detectors for reliable distributed systems. JACM 43(2):225–267, 1996.](images/chandra_failure_detectors.png)

As you can see above, certain problems are not solvable without a failure detector in asynchronous systems. This is because without a failure detector (or strong assumptions about time bounds e.g. the synchronous system model), it is not possible to tell whether a remote node has crashed, or is simply experiencing high latency. That distinction is important for any system that aims for single-copy consistency: failed nodes can be ignored because they cannot cause divergence, but partitioned nodes cannot be safely ignored.

How can one implement a failure detector? Conceptually, there isn't much to a simple failure detector, which simply detects failure when a timeout expires. The most interesting part relates to how the judgments are made about whether a remote node has failed.

Ideally, we'd prefer the failure detector to be able to adjust to changing network conditions and to avoid hardcoding timeout values into it. For example, Cassandra uses an [accrual failure detector](https://www.google.com/search?q=The+Phi+accrual+failure+detector), which is a failure detector that outputs a suspicion level (a value between 0 and 1) rather than a binary "up" or "down" judgment. This allows the application using the failure detector to make its own decisions about the tradeoff between accurate detection and early detection.

## Time, order and performance

Earlier, I alluded to having to pay the cost for order. What did I mean?

If you're writing a distributed system, you presumably own more than one computer. The natural (and realistic) view of the world is a partial order, not a total order. You can transform a partial order into a total order, but this requires communication, waiting and imposes restrictions that limit how many computers can do work at any particular point in time.

All clocks are mere approximations bound by either network latency (logical time) or by physics. Even keeping a simple integer counter in sync across multiple nodes is a challenge.

While time and order are often discussed together, time itself is not such a useful property. Algorithms don't really care about time as much as they care about more abstract properties:

- the causal ordering of events
- failure detection (e.g. approximations of upper bounds on message delivery)
- consistent snapshots (e.g. the ability to examine the state of a system at some point in time; not discussed here)

Imposing a total order is possible, but expensive. It requires you to proceed at the common (lowest) speed. Often the easiest way to ensure that events are delivered in some defined order is to nominate a single (bottleneck) node through which all operations are passed.

Is time / order / synchronicity really necessary? It depends. In some use cases, we want each intermediate operation to move the system from one consistent state to another. For example, in many cases we want the responses from a database to represent all of the available information, and we want to avoid dealing with the issues that might occur if the system could return an inconsistent result.

But in other cases, we might not need that much time / order / synchronization. For example, if you are running a long running computation, and don't really care about what the system does until the very end - then you don't really need much synchronization as long as you can guarantee that the answer is correct.

Synchronization is often applied as a blunt tool across all operations, when only a subset of cases actually matter for the final outcome. When is order needed to guarantee correctness? The CALM theorem - which I will discuss in the last chapter - provides one answer.

In other cases, it is acceptable to give an answer that only represents the best known estimate - that is, is based on only a subset of the total information contained in the system. In particular, during a network partition one may need to answer queries with only a part of the system being accessible. In other use cases, the end user cannot really distinguish between a relatively recent answer that can be obtained cheaply and one that is guaranteed to be correct and is expensive to calculate. For example, is the Twitter follower count for some user X, or X+1? Or are movies A, B and C the absolutely best answers for some query? Doing a cheaper, mostly correct "best effort" can be acceptable.

In the next two chapters we'll examine replication for fault-tolerant strongly consistent systems - systems which provide strong guarantees while being increasingly resilient to failures. These systems provide solutions for the first case: when you need to guarantee correctness and are willing to pay for it. Then, we'll discuss systems with weak consistency guarantees, which can remain available in the face of partitions, but that can only give you a "best effort" answer.

---

## Further reading

### Lamport clocks, vector clocks

- [Time, Clocks and Ordering of Events in a Distributed System](http://research.microsoft.com/users/lamport/pubs/time-clocks.pdf) - Leslie Lamport, 1978

### Failure detection

- [Unreliable failure detectors and reliable distributed systems](http://scholar.google.com/scholar?q=Unreliable+Failure+Detectors+for+Reliable+Distributed+Systems) - Chandra and Toueg
- [Latency- and Bandwidth-Minimizing Optimal Failure Detectors](http://www.cs.cornell.edu/people/egs/sqrt-s/doc/TR2006-2025.pdf) - So & Sirer, 2007
- [The failure detector abstraction](http://scholar.google.com/scholar?q=The+failure+detector+abstraction), Freiling, Guerraoui & Kuznetsov, 2011

### Snapshots

- [Consistent global states of distributed systems: Fundamental concepts and mechanisms](http://scholar.google.com/scholar?q=Consistent+global+states+of+distributed+systems%3A+Fundamental+concepts+and+mechanisms), Ozalp Babaogly and Keith Marzullo, 1993
- [Distributed snapshots: Determining global states of distributed systems](http://scholar.google.com/scholar?q=Distributed+snapshots%3A+Determining+global+states+of+distributed+systems), K. Mani Chandy and Leslie Lamport, 1985

### Causality

- [Detecting Causal Relationships in Distributed Computations: In Search of the Holy Grail](http://www.vs.inf.ethz.ch/publ/papers/holygrail.pdf) - Schwarz & Mattern, 1994
- [Understanding the Limitations of Causally and Totally Ordered Communication](http://scholar.google.com/scholar?q=Understanding+the+limitations+of+causally+and+totally+ordered+communication) - Cheriton & Skeen, 1993
