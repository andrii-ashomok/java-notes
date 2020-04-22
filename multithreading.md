# Многопоточность

## Основные тезисы


- При проектировании потокобезопасных классов **хорошие** объектно‐ориентированные технические решения: 
**инкапсуляция, немутируемость и четкая спецификация инвариантов** — будут вашими помощниками.
- Класс является потокобезопасным, если он ведет себя правильно во время доступа из многочисленных потоков, 
независимо от того, как выполнение этих потоков планируется или перемежается рабочей средой, 
и без дополнительной синхронизации или другой координации со стороны вызывающего кода.
- Для сохранения непротиворечивости состояний обновляйте родственные переменные состояния в **единой атомарной операции**.
- Операции **A и B** являются атомарными, если, с точки зрения потока, выполняющего операцию **A**, 
операция **B** либо была целиком выполнена другим потоком, либо не выполнена даже частично.
- Все обращения к **мутируемой переменной состояния** должны выполняться с удержанием одного и того же замка. 
Только тогда переменная защищена этим замком.
- Когда класс имеет инварианты, включающие более одной переменной состояния, каждая переменная, участвующая в инварианте,
должна быть защищена тем же замком. Это позволит обращаться к переменным или обновлять их в единой атомарной операции,
соблюдая инвариант.
- Хотя синхронизированные методы могут делать отдельные операции атомарными, 
дополнительная блокировка требуется, **когда многочисленные операции объединяются в составное действие**.
- При реализации политики синхронизации не поддавайтесь искушению пожертвовать простотой 
(что может поставить безопасность под угрозу) ради производительности.
- Избегайте удержания блокировки во время длительных вычислений или операций, таких как сетевой или консольный ввод‐вывод.


## Совместное использование объектов


- Однако существует распространенное заблуждение, что механизм **synchronized** затрагивает исключительно атомарность 
или демаркацию «критических секций». Синхронизация также имеет еще один важный аспект: **видимость памяти**.
- **Без синхронизации** компилятор, процессор и рабочая среда могут запутать порядок выполнения операций. 
Не стоит ожидать естественного порядка действий памяти в недостаточно синхронизированных многопоточных программах.
- Чтобы обеспечить видимость актуальных значений совместных волатильных переменных, 
**синхронизируйте читающие и пишущие потоки на общем замке**.
- Переменная **volatile** для компилятора и рабочей среды является совместной, то есть операции над ней не будут переупорядочены
с другими операциями в памяти. Волатильные переменные не кэшируются в регистрах или кэшах, 
где данные скрыты от других процессоров, поэтому их чтение всегда возвращает самый последний результат операций записи. 
Однако обращение к волатильной переменной не может побудить выполняющий поток к блокированию, 
что делает ее легковесным механизмом синхронизации.
- Используйте волатильные переменные **только** тогда, когда они **упрощают** реализацию и **проверку** политики синхронизации. 
**И избегайте их использования**, когда диагностика правильности требует утонченных рассуждений о видимости. 
Волатильные переменные должны обеспечивать видимость их собственного состояния, состояния объекта,
на который они ссылаются, или важного события жизненного цикла (например, инициализации или выключения).
- Блокировка может гарантировать как видимость, так и атомарность, а волатильные переменные гарантируют только видимость.

### Использование volatile переменных оправданно при следующих условиях:

1. записи в переменную не зависят от ее текущего значения, либо есть гарантия, 
что значения переменной обновляются только одним потоком;
2. переменная не участвует в инвариантах с другими переменными состояния;
3. при обращении к переменной заранее не требуется блокировка.

## Немутируемые, фактически немутируемые, мутируемые объекты.


Немутируемые объекты всегда являются потокобезопасными.
Объект является немутируемым, если:
- его состояние невозможно изменить после конструирования;
- все поля являются финальными;
- он надлежаще сконструирован (ссылка this не ускользает);
Немутируемые объекты могут безопасно использоваться потоками без дополнительной синхронизации, 
даже когда синхронизация для их публикации не используется. Однако если финальные поля ссылаются на мутируемые объекты, 
то синхронизация по-прежнему необходима для доступа к состоянию этих объектов.

Безопасную публикацию объекта, при которой ссылка на него и его состояние видна всем потокам в одно и то же время, 
можно провести с помощью: 
- инициализации объектной ссылки из статического инициализатора;
- сохранения ссылки на него в волатильном поле либо в AtomicReference;
- сохранения ссылки на него в финальном поле надлежаще сконструированного объекта;
- сохранения ссылки на него в поле, которое надлежаще защищается замком.

Требования к публикации объекта зависят от его мутируемости:
- немутируемые объекты могут быть опубликованы любым механизмом;
- фактически немутируемые объекты должны быть безопасно опубликованы;
- мутируемые объекты должны быть безопасно опубликованы и быть либо потокобезопасными, либо защищенными замком.

Ниже приведены наиболее полезные политики для применения и совместного использования объектов в конкурентной программе:
1. **Ограничение одним потоком**. Объект, ограниченный одним потоком, принадлежит эксклюзивно владеющему потоку, 
который может его изменять.
2. **Совместный доступ только для чтения**. Потоки могут обращаться к объекту, предназначенному только для чтения, 
конкурентно, без дополнительной синхронизации и возможности его изменять. 
Совместные объекты только для чтения включают немутируемые и фактически немутируемые объекты.
3. **Совместная потокобезопасность**. Потокобезопасный объект выполняет синхронизацию внутренне, 
поэтому потоки могут свободно обращаться к нему через его публичный интерфейс без дополнительной синхронизации.
4. **Защищенность**. С удержанием конкретного замка можно обращаться к объекту, 
инкапсулированному в другие потокобезопасные объекты, а также к опубликованному объекту, защищенному замком.

## Компоновка объектов (паттерны для безопасного структурирования классов)


Проектирование потокобезопасного класса включает три этапа:
- идентификация переменных, формирующих состояние объекта;
- идентификация инвариантов, ограничивающих переменные состояния;
- создание политики для управления конкурентным доступом к состоянию объекта.

### Мини итог:

- Инкапсуляция данных в объекте ограничивает доступ к ним только методами объекта, 
что упрощает обеспечение постоянного доступа с удержанием замка.
- Ограничение одним экземпляром упрощает создание потокобезопасных классов, 
позволяя анализировать их потокобезопасность без проверки всей программы.
- Если класс составлен из независимых потокобезопасных переменных состояния 
и не имеет недопустимых переходов из состояния в состояние, 
то он может делегировать потокобезопасность базовым переменным состояния.
- Если переменная состояния является потокобезопасной, не участвует в инвариантах, ограничивающих ее значение, 
и не имеет запрещенных переходов из состояния в состояние для любой из ее операций, 
то она может быть безопасно опубликована.
- Документируйте гарантии потокобезопасности класса для клиентов и политику синхронизации — для сопроводителей.

## Строительные блоки


- **Класс ConcurrentHashMap** — это хешированный ассоциативный массив Map, аналогичный хеш-массиву HashMap, 
но использующий другую замковую стратегию. Расширяющее возможности совместного доступа к ассоциативному массиву, 
оно обеспечивает конкурентность между читающими потоками, между читателями и писателями и между писателями. 
Однако семантика методов, работающих на всем ассоциативном массиве Map, таких как size и isEmpty, 
была немного ослаблена: в конкурентных средах эти методы менее полезны, поскольку ищут движущиеся цели.
- **Класс CopyOnWriteArrayList** является конкурентной версией синхронизированного спискового класса List, 
который предлагает более высокую конкурентность и устраняет необходимость в блокировке или клонировании коллекции 
во время итеративного обхода;
- **BlockingQueue**. Ограниченные очереди делают вашу программу устойчивой к перегрузке. 
Встраивайте управление ресурсами в проект с помощью блокирующих очередей, причем гораздо проще делать это заранее, 
чем модернизировать код позже.
- **Очереди LinkedBlockingQueue и ArrayBlockingQueue** являются очередями с дисциплиной доступа FIFO, 
аналогичными связному списку LinkedList и массиву-списку ArrayList, но с лучшей конкурентной производительностью, 
чем синхронизированный список List.
- **Очередь PriorityBlockingQueue** является очередью с приоритетом, которая полезна при обработке элементов в порядке,
отличном от FIFO. Как и другие сортированные коллекции, PriorityBlockingQueue может сравнивать элементы в соответствии
с их естественным порядком с помощью Comparable или Comparator.
- **Реализация SynchronousQueue** не содержит места для хранения элементов, но поддерживает список потоков, 
ожидающих постановки элемента в очередь или его удаления. 
При отсутствии стеллажа и непосредственной передаче вымытой посуды явно экономится время. 
Прямая эстафетная передача также подает больше обратных сигналов производителю о состоянии отданной им задачи.


## Блокирующие и прерываемые методы


Потоки могут блокировать продвижение, если ожидают завершения ввода-вывода, приобретения замка, 
пробуждения ото сна Thread.sleep или результата вычисления в другом потоке. 
Помещаясь в состояние **BLOCKED, WAITING или TIMED_WAITING**, поток не контролирует задачу, 
завершения которой ожидает, и возвращается в состояние работоспособности **RUNNABLE** только после возникновения 
внешнего события.
**InterruptedException** данное исключение отличает блокирующие методы, которые активно сопротивляются собственному
прерыванию. Прерывание представляет собой кооперативный механизм. 
Когда поток **A** пытается прерывать поток **B**, то просто просит поток **В** прекратить это делать. 
Отменой длительных действий занимаются не потоки, а блокирующие методы.


## Синхронизаторы


**Синхронизатор** (synchronizer) — это любой объект, координирующий поток управления в остальных потоках, 
основываясь на их состоянии. В качестве синхронизаторов могут выступать **блокирующие очереди, семафоры, барьеры и защелки**. 
Все синхронизаторы инкапсулируют состояние, которое определяет, пропускать или отправлять в ожидание поступающие потоки.
1. **Защелка (latch)** представляет собой синхронизатор, который может задерживать продвижение потоков до достижения 
своего конечного состояния. Он действует как закрытый шлюз, который в определенный момент разрешает всем потокам пройти, 
и навсегда остается открытым. Класс CountDownLatch представляет собой гибкую реализацию защелки. 
Состояние защелки **состоит из счетчика**, инициализируемого положительным числом ожидаемых событий. 
Метод **countDown** уменьшает счетчик, сигнализируя о том, что произошло событие, 
и методы **await** ожидают до тех пор, пока счетчик не достигнет нуля, т. е. все события завершатся.
2. **FutureTask** действует как защелка: он реализует **Future**, описывающий абстрактные вычисления, 
приносящие результат, которые реализуются с помощью интерфейса **Callable**, эквивалентного **Runnable**. 
Они могут находиться в состоянии ожидания выполнения, самого выполнения либо завершения 
(нормального завершения, отмены или прерывания). **FutureTask**, приняв завершенное состояние, остается в нем **навсегда**. 
Спецификация FutureTask гарантирует безопасную публикацию результата.
3. **Счетные семафоры (сounting semaphores)** регулируют число действий, способных обращаться к определенному ресурсу 
или выполнять одну и ту же задачу в одно и то же время. Класс Semaphore управляет набором виртуальных разрешений (permits). 
Семафоры полезны для реализации **ресурсных пулов**.
4. **Барьеры (barriers)** подобны защелкам в том, что они блокируют группу потоков до наступления какого-то события. 
Но в отличие от защелки барьер заставляет потоки вместе проходить барьерную точку в одно и то же время, 
чтобы продолжить работу. Защелки предназначены для ожидания событий, а барьеры — для ожидания других потоков. 
Барьер реализует протокол, который похож на назначение групповой встречи: 
«Подходите к “Макдоналдсу” в 18:00, оставайтесь там, пока все не соберутся, потом решим, что будем делать дальше». 
5. **Класс CyclicBarrier** позволяет фиксированному числу сторон неоднократно назначать встречу в барьерной точке.
6. Еще одной формой барьера является **обменник Exchanger** — двухсторонний барьер, в котором стороны обмениваются данными 
в барьерной точке. Обменники полезны, когда стороны выполняют асимметричные действия, например, 
когда один поток заполняет буфер данными, а другой потребляет данные из буфера. 
Потоки могут использовать Exchanger для встречи и обмена полного буфера на пустой.


### Итог:

- Все вопросы конкурентности сводятся к координированию доступа к мутируемому состоянию. 
Чем менее мутируемо состояние, тем легче обеспечить потокобезопасность.
- Объявляйте поля финальными, если нет необходимости в том, чтобы они были мутируемыми. 
- Немутируемые объекты автоматически являются потокобезопасными.
- Немутируемые объекты упрощают конкурентное программирование. 
Они могут свободно использоваться без блокировки или защитного копирования.
- Инкапсуляция позволяет управлять сложностью. Можно написать потокобезопасную программу, 
в которой все данные хранятся в глобальных переменных, но зачем? 
Инкапсулирование данных внутри объектов упрощает соблюдение их инвариантов, 
а инкапсулирование синхронизации внутри объектов упрощает соблюдение их политики синхронизации.
- Защищайте каждую мутируемую переменную с помощью замка.
- Защищайте все переменные в инварианте с помощью одинаковых замков.
- Удерживайте замки в течение всего времени составных действий.
- Программа, которая обращается к мутируемой переменной из многочисленных потоков без синхронизации, неисправна.
- Не верьте доводам, что обойдетесь без синхронизации.
- Включайте потокобезопасность в процесс проектирования либо явным образом документируйте то, 
что класс не является потокобезопасным.
- [Документируйте политику синхронизации. ](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/concurrent/package-summary.html)


# Структурирование конкурентных приложений


## Выполнение задач

### Основные тезисы

Большинство конкурентных приложений организованы вокруг выполнения задач (tasks) — абстрактных дискретных единиц работы. 
Разделение работы приложения на задачи упрощает организацию программы, облегчает обнаружение и исправление ошибок, 
предоставляя естественные транзакционные границы, и способствует конкурентности, 
обеспечивая естественную структуру для параллелизации работы.

- Первым шагом в организации программы вокруг выполнения задачи является определение разумных границ задач (task boundaries). 
Независимые друг от друга задачи могут выполняться параллельно при наличии достаточных обрабатывающих ресурсов.
- Серверные приложения должны демонстрировать хорошую пропускную способность, высокую отзывчивость, 
а также плавную деградацию по мере перегрузки.
- Задачи — это логические единицы работы, а потоки — это механизм их асинхронного выполнения.
- Интерфейс Executor обеспечивает отделение предоставления задачи от ее выполнения с помощью интерфейса Runnable 
и поддерживает жизненный цикл программы и перехватчиков для добавления функционала сбора статистики, 
управления приложением и мониторинга.


Политика выполнения — это инструмент управления ресурсами, выбор которого зависит от имеющихся вычислительных ресурсов 
и требований к качеству обслуживания. 


Спецификация политики выполнения отвечает на вопросы:
- В каком потоке будут выполняться задачи?
- Каков порядок выполнения задач (FIFO, LIFO, приоритетный порядок)?
- Сколько задач может выполняться конкурентно?
- Сколько задач может быть поставлено в очередь?
- Какую задачу удалить из-за перегрузки системы и как уведомить приложение?
- Какие действия предпринимать до и после выполнения задачи?


Статических фабричных методы **Executor**:
- **newFixedThreadPool**. Пул потоков фиксированного размера создает определенное число потоков по мере предоставления задач,
а затем старается держать размер пула постоянным.
- **newCachedThreadPool**. Более гибкий кэшированный пул потоков убирает простаивающие потоки 
и при необходимости добавляет новые, не накладывая лимит на размер пула.
- **newSingleThreadExecutor**. Исполнитель создает один поток для последовательной обработки задач, 
который при необходимости можно заменить.
- **newScheduledThreadPool**. Пул потоков фиксированного размера, который поддерживает отложенное 
и периодическое выполнение задач аналогично классу Timer.


Исполнитель Executor отключается, когда все (не являющиеся демонами) потоки терминированы, 
после чего следует выход JVM. Интерфейс ExecutorService расширяет интерфейс Executor, 
добавляя ряд методов управления жизненным циклом.
Жизненный цикл имеет три состояния: **работает, выключается и терминирован**. 
Службы ExecutorService создаются в рабочем состоянии, после чего метод **shutdown** инициирует выключение.
**Интерфейсы Runnable и Callable** описывают абстрактные вычислительные задачи, 
у которых есть четкая отправная точка, чтобы в итоге терминироваться. 
Жизненный цикл задачи, выполняемой исполнителем, состоит из четырех этапов: **создана, предоставлена, запущена и завершена**.
Поскольку выполнение задач может занимать много времени, мы хотим иметь возможность отменить задачу. 
В структуре Executor задачи, которые были предоставлены, но не были запущены, всегда могут быть отменены, 
а задачи, которые были запущены, могут быть отменены, если они откликаются на прерывание. 
Отмена задачи, которая была уже завершена, не имеет эффекта.
В спецификации Future подразумевается, что жизненный цикл задачи может двигаться только вперед, 
а не назад — как и жизненный цикл службы ExecutorService. Как только задача завершена, 
она остается в этом состоянии навсегда.

Реальная отдача от разделения рабочей нагрузки программы на задачи достигается при наличии большого числа независимых
однородных задач, которые могут обрабатываться конкурентно.

**CompletionService**: исполнитель Executor встречается с очередью **BlockingQueue**.

Она сочетает в себе функциональность исполнителя Executor и блокирующей очереди BlockingQueue. 
Вы можете предоставлять ей задачи Callable на выполнение и использовать методы **take и poll**, 
аналогичные тем, которые используются в очереди, для получения завершенных результатов. 
**Класс ExecutorCompletionService реализует CompletionService, делегируя вычисление исполнителю Executor.**
Реализация службы ExecutorCompletionService довольно проста. 
Конструктор создает очередь BlockingQueue для хранения завершенных результатов. 
Класс FutureTask содержит метод done, который вызывается по завершении вычислений. 
Когда задача предоставлена, она обертывается в QueueingFuture, подкласс класса FutureTask, 
который переопределяет метод done, помещая результат в очередь BlockingQueue. 
**Методы take и poll** делегируют очереди BlockingQueue, блокируя продвижение, если результаты еще отсутствуют.

### Мини итог:

Структурирование приложений вокруг выполнения задач упрощает разработку и способствует конкурентности. 
Структура Executor позволяет отстыковывать предоставление задач от их выполнения 
и поддерживает широкий спектр политик выполнения. Всякий раз, когда вы создаете потоки для выполнения задач,
рассмотрите возможность использования исполнителя Executor. 
Для того чтобы извлечь максимальную пользу из разбиения приложения на задачи, 
необходимо определить разумные границы задач. В некоторых приложениях хорошо работают очевидные границы задач, 
в то время как в других может потребоваться более детальный анализ.


## Отмена и выключение


### Основные тезисы


- В спецификации API или языка прерывание не привязано к какой-либо конкретной семантике отмены, 
но на практике использование прерывания не для отмены является хрупким и трудно сопроводимым механизмом. 
**Каждый поток имеет булев статус прерванности (interrupted status): true или false.**
- Вызов **метода interrupt не обязательно побуждает целевой поток прекратить действия** — он только доставляет сообщение о том,
что прерывание было запрошено.
- Потокам необходима политика прерывания, определяющая, как они интерпретируют запрос на прерывание: 
что и с какой скоростью они делают при обнаружении запроса и какие единицы работы считают атомарными по отношению 
к прерыванию. Наиболее разумной политикой является отмена на уровне потока или службы: 
максимально быстрый выход потока или службы, очистка при необходимости и уведомление владеющей сущности о выходе.
- Задачи не выполняются в потоках, которыми они владеют, — они заимствуют потоки, принадлежащие службе, 
такой как пул потоков. Код, который не является владельцем потока (любой код вне реализации пула), 
должен следить за сохранением статуса прерванности, чтобы владеющий код мог предпринять по нему какие-либо действия. 
(Присматривая за чьим-то домом, **вы не выбрасываете приходящую почту**, пока хозяева в отъезде — **вы ее сохраняете**, 
даже если читаете чужие журналы.)
- Поток должен быть прерван только его владельцем, который может инкапсулировать осведомленность 
о политике прерывания потока в соответствующем механизме отмены, таком как метод выключения.

Существуют две практические стратегии для обработки исключения InterruptedException: 
- распространение исключения (возможно, после очистки, специфичной для задачи), после которого метод станет прерываемым 
и блокирующим; 
- восстановление статуса прерванности, чтобы код выше в стеке вызовов мог с ним работать.

Когда метод Future.get выдает исключение InterruptedException (или TimeoutException), и вы знаете, 
что результат больше не нужен, то отмените задачу с помощью метода Future.cancel.

**Прерывание потока**, заблокированного во время выполнения синхронного сокетного ввода-вывода 
или ожидания приобретения внутреннего замка, не состоится (будет только установлен статус прерванности потока). 
Чтобы убедить поток, блокированный в непрерываемом действии, остановиться с помощью средств, подобных прерыванию, 
нужно понимать механизм его блокирования.
**Приобретение замка**. Нельзя остановить поток, заблокированный во время ожидания внутреннего замка. 
Но можно помочь ему приобрести замок и продвинуться. А явные замковые **классы Lock предлагают метод lockInterruptibly**, 
который позволяет ожидать замок и при этом **отзываться на прерывания**.

API потока не содержит формального понятия о владении: поток представлен объектом Thread, 
который может использоваться совместно, как любой другой объект. Однако фактически потоки имеют владельца, 
и обычно он представляет собой класс, который их создал. Пул потоков владеет своими рабочими потоками и должен при необходимости
позаботиться об их остановке. Приложение может владеть службой, а служба — рабочими потоками, 
но приложение не владеет рабочими потоками и не должно пытаться останавливать их напрямую. 
Вместо этого служба должна предоставлять методы жизненного цикла для самовыключения, 
которые также выключат принадлежащие ей потоки. **Например, служба ExecutorService предоставляет методы shutdown и shutdownNow.**