# Hi five, {$userName}, and low latency!
![oul3gz1bvwwy](https://user-images.githubusercontent.com/10584911/114997036-1fcaa280-9ea8-11eb-8df9-a426e94bc17a.jpg)

Итак, ты PHP-разработчик. В широких кругах ты возможно немного стесняешься об этом говорить, в узких ты стараешься говорить, что ты исключительно Backend PHP-разработчик.
Справедливости ради, PHP-разработчик в 2001 и в 2021 абсолютно разные люди с абсолютно разным технологическим стеком. В твоем портфеле навыков, кроме основного стека LNMP (LAMP, WAMP...), обязательно есть JavaScript, HTML, CSS.

И это не предел - Docker, Kubernetes (шучу, просто docker swarm), AWS и другие облачные сервисы, Gitlab CI/Github Action (Jenkins, Teamcity, CircleCI). А также Symfony, Laravel, Yii, Phalcon, Laminas, Spiral, CakePHP и масса менее известных фреймворков для разработки, несколько фреймворков для тестирования кода, библиотеки для статического анализа кода и сотни других готовых и не очень решений.

Но. Твое приложение в итоге тяжелое, медленное и кривое. Не то что шустрые красивые решения на ASP.NET или Java. Коробочные продукты требуют специфического окружения, плохо дружат с Windows экосистемой и вообще крайне нишевые. Зайдя в обсуждение любой статьи про PHP ты увидишь как минимум один комментарий про то, что ему пора уже на покой, а тебе - сменить свой стек. Рандомный школьник, что пишет свой велосипед по практикам нулевых на Java может смело бездоказательно утверждать, что его калькулятор лучше, чем твоя CRM.

А так ли плох язык? Релизы PHP 7 и PHP 8 стали важной вехой в истории языка, так как внесли не просто новые фичи, а кардинально изменили подходы написания приложений на нем. Улучшения в структуре типов, в самой системе типов, в синтаксисе. Буст производительности языка и новые возможности его применения помимо той старой первой задачи, о которой часто пишут любители набросать на вентилятор.

Помимо этого, ты уже умеешь горизонтально масштабировать свое приложение, обвязываешь его RoadRunner, Swoole, Workerman, AmpPHP, ReactPHP, NodeJS, а некоторые даже используют Nginx Unit или пишут свои велосипеды для многопоточности. Добавляешь туда оптимизированные реестры данных (Clickhouse, Sphinx, Elasticseatch, MongoDB), разного рода кеши (Redis, etcd, Cassandra, Memcached, Tarantool), интегрируешь с кучей cdn для статики.

И все равно медленно! Тлен, безнадега, пора опускать руки и принять судьбу "формошлёпа"?

Не обязательно. Ведь ты можешь последовать советам комментаторов и уйти в тот же JavaScript (меняя шило на мыло), или переучиться на Go, Python, Java, C# (и все это применять в рамках ASP.NET, штампуя ужасные soap решения). Можешь сделать монстра, совместив все эти подходы, чтобы бизнес нуждался в широком спектре специалистов разного стека.

А можешь упасть на дно нативного кода со своей вершины абстрактной разработки! Я предлагаю тебе писать модули для PHP, сразу исключив из цепочки твоего рантайма огромный пласт интерпретации твоего кода, не считая еще и чтения сотен файлов твоего проекта и еще большего количества файлов `vendor-а` с файловой системы. Можно отвертеться от этой идеи, ответив - я же не знаю ни C, ни zend engine. А если надо знать только PHP? В таком случае есть Zephir.

# Знакомься, Zephir!
![EU0zN4zXQAAxW-D](https://user-images.githubusercontent.com/10584911/114997186-412b8e80-9ea8-11eb-9809-bcf6f5f38160.jpg)

Что это такое? Это промежуточный язык, позволяющий писать расширения/модули для PHP, скомпилированные и оптимизированные, не имея многолетнего опыта в разработке на языке C или навыков в Zend Engine, вызвающих болезненные сновидения.

На самом деле этому языку больше 8 лет. Первые упоминания о нем были аж в 2013 году, а последние, увы, в 2016. Последние 5 лет в океане Zephir был штиль и только релиз PHP 8 вдохнул свежий бриз в паруса языка. На данный момент core-team активно занимается исправлением некробагов и добавлением новых фич, язык уже совместим с последней версией PHP, имея при этом свои уникальные фичи, которые были в нем задолго до того, как появились RFC по ним в PHP.

Давай разберем кусочек кода, написанного на Zephir:
```
    public function get(string! key, var defaultValue = null) -> var
    {
        string content, filepath;
        array payload, reversedPayload;

        if this->has(key: key) == false {
            return defaultValue;
        }

        let filepath = this->getFilepath(key);

        if !file_exists(filepath) {
            return defaultValue;
        }

        let reversedPayload = this->getPayload(filepath);
        let payload = reversedPayload->flip();

        if unlikely empty payload {
            return defaultValue;
        }

        if this->isExpired(payload) {
            return defaultValue;
        }

        let content = Arr::get(array: payload, key: "content", defaultValue: null);

        return this->getUnserializedData(content);
    }
```
Первое, что бросается в глаза - это typehint var в конструкторе метода и возвращаемом типе - висящий на момент публикации RFC PHP с mixed typehint все еще не принят.

Можно увидеть конструкции инициализации переменных на третьей и четвертой строке. Да, в Zephir тебе необходимо объявлять все переменные в начале логического блока, чтобы дальше использовать. И как ты заметил, в языке реализована статическая типизация (!).

В первой же контрольной структуре встречается именованный параметр в вызове `this->has(key: key)`. Да, в Zephir ты можешь использовать именованные параметры. Там же видна разница между контрольными структурами в PHP и Zephir - не требуется оборачивать условия в `()`.

Как ты заметил, в языке есть встроенные методы в скалярных типах, например: `let payload = reversedPayload->flip();` где reversedPayload - обычный массив и эта конструкция обычно выглядит как `let payload = array_flip(reversedPayload);`

В третьей контрольной конструкции можно заметить оператор `unlikely` - это оператор прогнозирования ветвлений, позволяющий не выполнять условие проверки, если это возможно.

В языке множество других очень вкусных фич и сахара, которые позволят тебе писать более эффективный код, но при этом твоих навыков в PHP будет достаточно, так как синтаксические различия минимальные.

Если тебя вдруг заинтересовал этот язык, ниже будут ссылочки на документацию и репозиторий проекта.

https://docs.zephir-lang.com/0.12/en/types

https://github.com/zephir-lang/zephir
