Лекция 12.02.2025. Управление нитями.

идея - есть ядро, которое умеет планировать процессы - те же самые нити, только нитей по одной. Почему бы не переиспользовать его. Но, поскольку нити внутри процессов, надо планировщик ядра переделать, чтобы он понимал, что в одном процессе может быть несколько нитей. И эти нити должны одновременно звать системные вызовы.
Когда мы зовем системный вызов, мы переключаемся на некий стек в ядре, который находится в user areа, и что-то в нем делаем. То есть мы должны иметь несколько стеков и несколько user area. Я предлагаю терминологию, что у нас в процессах могут находиться несколько user area и стеков.
Тут возникает вопрос, которым меня пытались доканывать еще в прошлом семестре. Такая терминология не совсем общепринята, но в солярисе логична. Есть нити ядра, которые обслуживают пользовательские процессы. И есть юзерлендовские нити. Критерий разделения - где находится стек нити - в ядре или в юзерспейсе, по которому нити делятся на ядерные и пользовательские. 
Однако есть нити ядра, у которых есть пользовательский стек. Такие нити называются LWP - light weight process. Системная нить - нить, у которой есть ядерный стек. LWP - нить, которая обслуживает системные вызовы. В солярисе можно считать, что это одно и то же.
Одному позикс треду соответствует один LWP в солярисе. В других системах это может быть не так. Планировщик мог бы разбрасывать пользовательские нити по LWP, смотря на то, какой LWP свободен.
Привязка пользовательской нити к системной происходит при переходе состояния нити от runnable к running.
При этом если нить заблокировалась на чем-то в системной нити, она может продолжать быть привязана. 
Это так называемыая Гибридная реализация. В языке GO она доведена до некоторой степени совершенства. Я экспериментировал с утилиткой которая может называться утилиткой ддос атаки и обнаружил, что у меня на тысячи открытых сокетов всего восемь системных нитей по числу ядер.
Если у вас нитей больше, чем процессорных ядер, надо задуматься, а зачем они нужны?
Пользовательские нити
- планировщик в пользовательском адресном пространстве
- не требует переделки ядра системы
- не требует дополнительных системных ресурсов
- легко реализовать
- как быть с блокирующимися вызовами? Блокироваться будет весь процесс!
- не может использовать несколько процессоров

Ну опять же граница... ну, собственно, ладно, ну и вот.

В winapi существует fiber - волокно. В рамках thread можно создавать fiber. При этом fiber-ы не могут звать блокирующиеся системные вызовы, поскольку они блокируют всю нить. Поэтому файберы бесполезны и ими никто не пользуется.

Системные нити - в ядре уже есть планировщик, причем вытесняющий и довольно хороший. Однако его надо переделывать, как минимум в него надо внедрить мысль, что в рамках одного процесса может быть несколько нитей. 
Первая попытка выгледела так, что они сделали некоторое обобщение fork, можно сделать clone, в котором сказать, какие сегменты оставить разделяемыми. То есть процессы разделяют адресное пространство, по сути являясь нитями. Однако воспроизвести семантику posix тредов на этом затруднительно.

почему в джаве и го пользовательские нити возможны, а в си не особо? Потому что сишная программа может напрямую дергать системные вызовы, в джаве и го такое невозможно. То есть в си вам надо обманывать каким-то образом пользователя.

Гибридная реализация MxN
- требует наличия как системного, так и пользовательского планировщика.
- системный планировщик поддерживает M нитей на процесс
- пользовательский планировщик поддерижвает N нитей.
- пользовательский планировщик распределяет пользовательские нити между системными нитями, подобно тому, как планировщик многозадачной ОС распределяет задания между процессами.

Гибридный планировщик.
Вот ЭЭЭЭЭЭЭЭЭЭЭЭЭЭЭЭЭ. Ну и вот э, тут опять про это все вспоминается, вводится термин LWP - это системная нить.
Системные нити подчинены процессы
пользователських нитей больше, чем системных, во всяком случае, в начале.
Ядерная нить - нить ядра, системная нить - это пользовательская нить, у которой есть соответствующая ядерная.
Когда все LWP садятся в блокирующиеся системные вызовы, система посылает сигнал SIGWAITING.

У меня была дискуссия на питоне с вашими коллегами, которые были на год старше. В ходе этой дискуссии они сделали неожиданное открытие, что линукс это не система. Когда вы пишете расширение на языке си для питона, вы можете использовать стандартную библиотеку, ту же, которую использует питон.
Самая веселая форма этой ситуации вас ждет под виндоус. Каждая версия Visual Studio имеет свою версию libc. Количество версий libc в системе должно быть не меньше количества версий Visual Studio. И перед вами в полный рост встает вопрос, как быть, когда у вас питон, и вы не знаете, какой Visual Studio он собран. В юниксе все хорошо, потому что в норме у вас libc одна на всю систему - системная библиотека. Питон использует эту библиотеку, если вы собирали его сами или его собирала система управления пакетами, то проблем нет. Правильно? Правильно. Я это сказал и успокоился. Мне студент начал возражать, если мы соберем модуль на Debian и подсунем его redhatщвскому питону, то у нас будет то же самое, как если у вас не совпадут версии Visual Studio. Значит линукс это не система, раз у redhat и debian разные версии системной библиотеки.

POSIX например, стандарт языка Си, говорит, как должна выглядеть программа на языке Си. Это API. А то, как должны быть разложены байты в структурках, это Application Binary Interface. Он один только у ядра линукса. Ядро линукса вроде бы как одинаковое (на самом деле тоже не одинаковое, поскольку у debian и у redhut разные сборки ядра) но системы разные.
Ядро и система - разные вещи, вот у линукса есть одно ядро на разные вещи. Поэтому термины "Ядро" и "система" надо разводить.

Возвращаемся к нашим потокам.
Unix с MxN реализациями до сих пор где-то ездят, хотя вы живьем их уже вряд ли увидите, но встретить реализации именно позикс тредов вы можете. Как и встретить в стандарте или манах какие-то понятия.

В linux в ядре 2.4 есть системные нити clone(2), они имеют собственный PID и собственную запись в таблице процессов.

Сборка многопоточных программ. 
насчет компиляции, компилятору надо сказать, что он компилирует многопоточную программу, и желательно сказать, какую библиотеку он использует. -lpthread. Дальше были всякие анекдоты, плюсовая библиотека линукса, template STL содержали вызовы posix thread примитивов. И чтобы их можно было скомпоновать в однопоточную версию, в плюсовую библиотеку были засунуты пустышки этих примитивов, чтобы они линковались. Вроде это починили в современных линуксах, тем не менее, довольно забавная грабля. В другую сторону это не грабля.
В десятом солярисе втащили posix thread в основную libc. А библиотека lpthread есть, но она пустышка, существует только чтобы старые makefile не ругались.
На самом деле еще с вами мягко поступили.
Вот вы все пишете makefile с использованием lsocket. Вот у кого макос? Не признаются. Тем не мнеее, если вы запишете lsocket там, вас обругают, что нет такой библиотеки, вам надо адаптировать makefile. Я за makefile с lpthread карать не буду, но под солярисом она не нужна. на самом деле это имеет несколько подводных камней, которые все сводятся к фразе, которую я говорил - философия языка си - вы не платите за то, чем не пользуетесь. Увы, многопточность это такая вещь, в которой вы платите за то, чем не пользуетесь.
Есть рекомендация, если вы используете pthread.h, вы должны его включать перед всеми остальными header файлами. Он может взять и заменить что-нибудь на что-то другое. На солярисе вам достаточно, чтобы у вас был определен символ REENTRANT, тогда у вас все будет работать как надо. lpthread в солярисе пустышка и не нужна, только для совместимости со старыми makefile.
На этой оптимистической ноте мы с вами реализацию заканичвамем.
Так, мне надо, мне надо этот.. чертов.. А, это то, что я вашим коллегам с первого курса показывал. Отстань.. Бздыньк. Отстань. Так, вот, то есть у нас следующая тема это, соответственно, вот лекцию два мы с вами изучили, так, ээээээ, где у меня презенташка-то? Да, у меня два дока и она у меня лежит в другом месте. Ха-ха. 

Вопрос: на солярисе, когда мы собираем нашу программу, вообще можно ключи не указывать?
Ответ: нет, ему надо указать ключ -mt, чтоб он знал, что собирает многопоточную программу.

Так, чтобы с нитями работать, их надо во-первых научиться создавать и завершать.
Часть проблемы в том, что в джаве за вас часть проблем решает хреновина под названием сборщик мусора. В си сборщика мусора нет, никто за вами сопли вытирать не будет, а в многопоточной программе вы этих соплей генерируете довольно много.

Ну и вот, создается нить функцией pthread_create - это не системный вызов. Она внутри содержит какой-то системный вызов или даже несоклько, в солярисе и линуксе они даже разные и не стандартные.
Особенности
1. имя начинается с pthread, на момент, когда принимался стандарт, было несколько проприетарных реализаций многопточности.
2. не следует соглашениям, которым следовали наши системные вызовы, которые мы проходили в прошлом семестре.

Вопрос, что такое errno и как с ним быть, встает перед нами в полный рост. Если его делать глобальной переменной, будет плохо, а если нет, еще хуже. Куды крестьянину податься? Откажемся от errno. Код ошибки будет прямо в коде возврата функции. Код нельзя напрямую использовать с perror. Рекомендованный способ - звать strerror и получать ошибку. У pthread есть коды ошибок, которые нормальный системный вызов вам не вернет. Все коды в мане описаны. Больше, чем в стандарте. Но эти коды из мана только для текущей версии соляриса, в других системах могут быть другие коды, и новые коды. 
Почему может не создасться нить? Может не хватать памяти под стек или не хватать системных ресурсов на LWP.
Есть довольно странное ограничение - вы не можете иметь больше восьми тысяч нитей на процесс. Почему - вам надо объяснять детали про процессор x86. Интеловские юниксы хранят ID текущей нити в регистре то ли GS то ли FS. Эти регистры мало кто использует с одной стороны, с другой стороны это должен быть валидный селектор сегмента. В общем-то вряд ли вы столько нитей в ваших программах создадите. Третья особенность следует из предыдущего.
Системный вызов create что-нибудь мог бы вернуть что-нибудь, но не может, поскольку возвращаемое значение занято под код возврата. pthread_t - это непрозрачный тип. То есть вы можете посмотреть, что это такое в конкретной реализации, но вам не следует полагаться на это знание, потому что в следующих реализациях или в других это может быть иначе. В солярисе ID нити это небольшое целое число, локальное для процесса. Причем первая нить, которую вы создаете, имеет идентификатор 4. Куда делись предыдущие - я в общем-то сам не знаю. Вы не можете тем не менее полагаться на то, что это число, эта штука может быть даже указателем. Это не очень хорошо, ведь если там лежит какой-то мусор и это число, вас просто обругают. А если это указатель и там лежит какой-то мусор, ваша программа упадет по ошибке защиты памяти. Не рекомендуется лихо манипулировать неиницилазированынми переменными типа pthread_t. Вы должны быть уверены, что там валидный идентификатор.
3. Еще один параметр, который называется pthread_att. На тот момент, когда стандарт принимался, было ясно, что разные реалзиации заходят напихать туда своих, зависящих от реализации, опций. Если вы эти опции хотите передавать как отденльные параметры, вы должны знать, сколько их будет. Конечно, вы можете использовть ..., то есть передавать их как список, но дальше у вас начинаются всякие ужасы на тему того, в каком порядке их передавать и прочее. В общем использовали такую идиому, у вас есть еще один непрозрачный тип, pthread_att, которым вы манипулируете при помощи set/get методов, в случае pthreadов функций, которые вы проходили в джаве. Соответственно структура непрозрачная, поэтому её реализации могут расширять как бог на душу положит. С одной стороны в си нет синтаксического сахара для такого, с другой стороны это объекто-ориентированная идиома, которая имеет некоторый смысл, но порождает некоторую писанину. Если вы вместо pthread_att передадите NULL, вы получите аттрибуты по умолчанию, которые библиотека считает правильными. Если вы хотите атрибуты не по умолчанию, можете этой структурой манипулировать, но только тем, что знаете.
4. Тело функции, которые вы хотите позвать. В отличие от джавы, у нити нет метода start(). Функция вызывается не сразу же после pthread_create, но, тем не менее, она вызывается как можно скорее, как она создалась. Этой функции вы можете передать параметр типа void*, но runtime с этим параметром никогда не манипулирует как с указателем. Поэтому туда можно передавать NULL и небольшие целые числа, которые могут быть приведены к типу указателю.

У вас есть задачка про число пи, и у вас есть соблазн ввернуть и передать туда double. И загвоздка в том, что хотя на 64-битной платформе дабл туда влезает, на 32-битной не влезет и вас ждут чудеса. Еще одно чудо вас ждет, у нити есть статус завершения, который тоже имеет тип void*. Стандарт с одной стороны явно разрешает туда передавать скаляры. С другой стороны это выводит нас на концепцию, которая называется ownership. Она довольно простая, для языка Си и Си++ довольно важна. В джаве за вами подтирают сопли, когда вы куда-то передаете объект, вы можете не париться насчет его дальнейшей судьбы. Пока вы пишете на чистой джаве. А когда вы зовете какую-нибудь JNI - java native interface, позволяющие звать написанные на языке Си функции, перед вами в полный рост встает вопрос, каков же срок жизни того указателя, который вы отдали. Джава не знает, что эта JNI штука этим объектом пользоваться. И она его может собрать. А та штука, которую вы зовете, если написана на простом Си, то у вас нету способа сообщить джаве, что вам этот объект больше не нужен. В общем начинаются приколы. А в языке си они перед вами встают в полный рост, кто должен освобождать те объекты, которые вы создаете в языке си. А простой объект - объекты должен освобождать тот же, кто их создавал, и функции, которым вы эти объекты передаете, не должны знать, как эти объекты создавались. В многопоточных программах эта идиома приобретает несколько страныне оттенки, и куды крестьянину податься, в конечном итоге науке неизвестно. Однако стандартная практика - кто создает объект, тот его и уничтожает. 
Это в джаве вы можете объект создавать только new. А в Си и Си++ вы можете создавать их не только маллоком. Даже оператор new в плюсах может создавать черт-те что. В однопоточной программе логично, вы создали кого-то, отдали, он вернулся, вы его убили. А в многопоточной программе вы можете даже не дожить до момента, когда надо убить того, кого вы создали. 
Что может пойти не так - когда мы дойдем до pthread join и detach нитей, мы поймем.

Вопрос: что значит restrict
Ответ: антоним volatile. Это значит, кроме этой функции никто по этому указателю не будет ходить. Это позволяет компилятору более вольно с этой памятью обходиться, однако если вы эту часть контракта нарушите, то вас ждут чудеса, причем вплоть до краха. Слово restricted было придумано в рамках той же программы, в которой придумали volatile. Без неё невозможно писать lockless программы. Но мы это не проходим.

pthread_t - в старых линуксах был просто тип процесса.
Передача параметров нити - возникают две проблемы: утечка памяти и висячие ссылки. Утечка памяти образуется если вы выделяете память и не освобождаете, а висячие ссылки образуются, если вы слишком агрессивно освобождаете.

Выделение памяти под параметры в стеке. Следует проявлять осторожность при передаче параметров, размещенных в стеке. Если вы их выделяете и зовете функцию нормальным образом, то ваши объекты доживают до возвращения этой функции гарантированно. Если вы это делаете многопоточно, то вы либо должны дожидаться завершения нити, либо вы получаете висячую ссылку, ведь если вы выходите за пределы того блока, где вы размещали переменную на стеке, она уничтожается, но ваша нить может пытаться использовать эту память, что будет ужасно.

Много задачек и решений построены по такому признаку, что вы создаете все нити в main, а в конце main делаете на них на всех join. Это получается довольно жесткое ограничение, что вы нити ни откуда кроме main создавать не можете. Другая программа, если вы зовете malloc, то кто должен делать free? Я вам предлагаю идиому, когда вы просто этот же блок параметров, который передаете, возвращаете из нити. 
В рамках вашего процесса любая нить может ждать любую другую вашу нить. Эта идиома тоже не всегда подходит, поскольку теоретически вы не обязаны ждать всегда завершения нитей. И тут снова начинается история, куда крестьянину податься, и самая темная сторона истории в том, что если создатель не дожидается завершения нити, вы должны внутри нити освобождать параметры. Однако в этом случае вы должны внутри нити знать, как была выделена память под этим парамтеры.

В общем у вас есть несколько идиом, среди которых вы должны будете выбирать и думать.

В прошлом семестре я не слишком карал за шаблонные решения, я был против только того, что мне пытаются сдать откровенное дерьмо. Почему-то те, кто борется с шаблонными решениями, единственным способом бороться с ними считают вставлять в код, не побоюсь этого слова, дерьмецо.

Опять же, заниматься многопоточным программированием с невыставленным стилем, с непоставленной рукой, это путь либо к вывиху мозга, либо к выпуску людей, которые будут писать сбертройку. Причем писать её ежедневно и ежечасно.

Ну и вот.. Аа. Вот, ну и вот. На самом деле у меня еще дофига времени.

Завершение нити - это функция pthread_exit, она ничего не возвращает, то есть она не может завершиться ошибкой. И в качестве кода возврата этой нити возвращает то, что вы передали как параметр. 

то есть pthread_exit уничтожает все ресурсы, созданные при создании нити, в том числе стек нити и так называемый thread local data, который мы с вами изучить не сумеем. И никакие указатели в стек нити не могут быть валидными.

В некоторых комбинациях libthread c++ компилятора не ывзывает деструкторы всех локальных переменных.

Трагизм в том, что разные плюсовые компиляторы между собой несовместимы. У gcc и сановского компилятора разная структура стекового кадра и по-разному надо звать деструкторы. Если вы пишете на плюсах на gcc на нашем солярисе, вы рискуете потерять деструкторы.

Тем не мене pthread_exit сама по себе концептуально простая. 
Тема на самом деле веселая. 
Утверждение первое. В позикс тредах и в солярисе нити подчинены процессу. То есть когда вы делаете exit, завершаются все нити процесса, причем завершаются по-жесткому. Обычно если вы зовете exit откуда-то из недр вашей программы, вы уверены, что все плохо, в том числе это означает, что и другие нити у вас продолжать смысла нет, их нужно как можно скорее завершить. Есть такое понятие, как каскадные ошибки - ошибки работы с памятью. Когда вы такую ошибку видите, вам надо программу как можно скорее завершить, потому что ваша программа начинает что-то делать на основе ошибочных данных - может записать мусор в файлы, наплодить нитей, убивать другие процессы, причем все это со скоростью, пропорциональной количеству ядер в системе. Когда вы понимаете, что все плохо, вы хотите, чтобы ваши хуки звались. У exit есть родственник _exit(), который завершает ваш процесс и никаких atexit хуков не зовет, не говоря уже о деструкторах глобальных переменных и локальных переменных в нитях.

Многие программы на заре многопоточного программирования отличались таким свойством, что вы нажимаете exit, а программа начинает что-то делать, и делает это что-то полчаса. Целое поколение программ, написанных в конце 90-х начале 00-х, страдали этим страшно. Однако, чего это стоило под капотом, мы не знаем, исходников не видим.

Собственно, в джаве та же сама проблема, если джава начнет перед выходом собирать мусор, она его может собирать долго.