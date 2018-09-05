---
title: "Бенчмарк, как основной критерий принятия решения о изменении существующего кода"
date: 2018-09-05T20:00:57+05:00
draft: false
tags: ["go"]
---
Билл Кеннеди в одной из лекций своего курса "Ultimate Go programming" на safarybook ([https://www.safaribooksonline.com/library/view/ultimate-go-programming/9780134757476/](https://www.safaribooksonline.com/library/view/ultimate-go-programming/9780134757476/))  сказал следующее:

**"****_Многие разработчики стремятся оптимизировать свой код. Они берут строчку и переписывают ее, говоря, что так станет быстрее. Им нужно остановиться. Говорить, что тот или иной код быстрее можно только после того, как вы его отпрофилировали или сделали бенчмарки. Если вы гадаете, то это не является правильным подходом к написанию кода._****"**

Мне захотелось написать заметку, в которой бы на практическом примере показывалось, как это может работать. Давайте попробует доказать, следующее утверждение: 

*Для разработчиков бенчмарки должны стать главной инстанцией при совершении суждения о том какой из алгоритмов: существующий или предложенный ему на замену является эффективнее. Информация из интернета, суждения коллег и других людей, какой бы авторитет они для вас не представляли, могут служить вспомогательными элементами на пути к поиску правильного решения, но роль главного судьи, решающего быть или нет новому алгоритму должна оставаться за бенчмарками.*

На днях я переписывал один пакет на Go и мое внимание привлек следующий код, который как раз можно было бы использовать в качестве такого практического примера:

'''
builds := store.Instance.GetBuildsFromNumberToNumber(stageBuild.BuildNumber, lastBuild.BuildNumber)

tempList := model.GameBuildsList{}

for i := len(builds) - 1; i >= 0; i-- {

  b := builds[i]
  
  b.PatchURLs = b.ReversePatchURLs
  
  b.ExtractedSize = b.RPatchExtractedSize
  
  tempList = append(tempList, b)
  
}
'''

Этот код во всех элементах слайса **_builds,_** возвращаемого из базы данных,* заменяет PatchURLs на ReversePatchURLs, ExtractedSize на RPatchExtractedSize и делает реверс - меняет порядок элементов, так чтобы последний элемент стал первым, а первый последним.

На мой взгляд, исходный код немного сложен для восприятия и может быть упрощен. Этот код выполняет простой алгоритм состоящий из двух логических частей: изменение элементов слайса и сортировка, но чтобы читателю вычленить эти две части понадобится время. 

Не смотря на то, что обе части важны, в коде реверс не так акцентируется, как хотелось бы. Он разбросан по трем оторванным друг от друга строкам: инициализация нового слайса, итерация существующего слайса в обратном порядке, добавление нового элемента в новый слайс. Тем не менее, нельзя игнорировать и несомненные преимущества данного кода: код рабочий и протестирован, и если говорить объективно, то он вполне адекватный. 

Субъективное восприятие кода отдельным разработчиком не может служить оправданием для его переписывания. К сожалению или к счастью, мы не переписываем код только потому что он нам просто не нравится (или, как чаще бывает, просто потому что он не наш, см. "Фатальный недостаток").  Но что, если нам удастся не только улучшить восприятие данного кода, но и существенно ускорить его? Это же совсем другое дело. Можно предложить несколько альтернативных алгоритмов выполняющих заложенный в коде функционал. 

Первый вариант: перебор всех элементов в цикле range; для реверса первоначального слайса в каждой итерации добавляем элемент в начало итогового массива. Так мы избавимся от: громоздкого *for*, переменной  *i*, использования функции *len*, трудно воспринимаемого перебора элементов с конца, а также сократим количество кода на две строки с 7 строк до  5, а чем меньше кода, тем меньше вероятность допустить в нем ошибку.

**_var _****_tempList _****_[]*_****_store_****_._****_GameBuild_**

**_for _****___****_, _****_build _****_:= _****_range _****_builds _****_{_**

**_  _****_build_****_._****_PatchUrls_****_, _****_build_****_._****_ExtractedSize _****_= _****_build_****_._****_ReversePatchUrls_****_, _****_build_****_._****_RPatchExtractedSize_**

**_  _****_tempList _****_= _****_append_****_([]*_****_store_****_._****_GameBuild_****_{_****_build_****_}_****_, _****_tempList_****_...)_**

**_}_**

Убрав перебор слайса с конца,  мы четко разграничили операции изменения элементов (3 строка)  и реверса исходного массива (4 строка).

Основной замысел второго варианта состоит в еще большем разнесении изменения элементов и сортировки. Сперва перебираем элементы и меняем их, а потом сортируем слайс отдельной операцией. Данный метод потребует дополнительной реализации интерфейса сортировки для структуры-элемента слайса,  но повысит читабельность и позволит полностью отделить и изолировать реверс от изменения элементов, а слово **_Reverse _**однозначно укажет читателю на желаемый результат.

**_var _****_tempList _****_[]*_****_store_****_._****_GameBuild_**

**_for _****___****_, _****_build _****_:= _****_range _****_builds _****_{_**

**_  _****_build_****_._****_PatchUrls_****_, _****_build_****_._****_ExtractedSize _****_= _****_build_****_._****_ReversePatchUrls_****_, _****_build_****_._****_RPatchExtractedSize_**

**_ _****_}_**

**_sort._****_Sort_****_(sort._****_Reverse_****_(sort._****_IntSlice_****_(keys)))_**

Третий вариант - является практически повторением второго, но для сортировки используется **_sort_****_._****_Slice_**, что добавляет лишнюю строчку, но избавляет нас от необходимости дополнительно реализовывать интерфейс сортировки. 

**_     _****_for _****___****_, _****_build _****_:= _****_range _****_builds _****_{_**

**_  		_****_build_****_._****_PatchUrls_****_, _****_build_****_._****_ExtractedSize _****_= _****_build_****_._****_ReversePatchUrls_****_,          _****_build_****_._****_RPatchExtractedSize_**

**_     _****_}_**

**_     _****_sort_****_._****_Slice_****_(_****_builds_****_, func_****_(_****_i _****_int_****_, _****_j _****_int_****_) _****_bool _****_{_**

**_        _****_return _****_builds_****_[_****_i_****_]._****_Id _****_> _****_builds_****_[_****_j_****_]._****_Id_**

**_     _****_})_**

 

На первый взгляд, по внутренней сложности, количеству итераций, используемым функциям изначальный код и первый алгоритм близки; второй  и третий варианты могут быть чуть сложнее, но за счет использования встроенных функций однозначно сказать какой из четырех вариантов будет самым оптимальным все же нельзя. 

Итак, после слов Билла, мы запретили себе принимать решения, основываясь на голословных предположениях не подкрепленных доказательствами, но очевидно, что здесь наибольший интерес представляет то, как поведет себя функция **_append_** при добавлении элемента в конец и в начало слайса. Ведь на самом деле эта функция не так проста, как кажется.

Append работает следующим образом (blog.golang.org/slices): она либо добавляет к существующему в памяти слайсу новый элемент, если его capacity больше итоговой длины, либо  резервирует в памяти место под новый слайс, копируя в него данные из первого аргумента и добавляя данные второго, возвращая в качестве результата уже новый слайс. 

Самый интересный нюанс в работе этой функции состоит в алгоритме используемом для резервирования памяти под новый массив. Так как самой дорогой операцией являться выделение нового участка памяти, то чтобы сделать операцию append дешевле, разработчики Go пошли на небольшую хитрость. По началу при каждом резервировании памяти, чтобы не резервировать память заново при каждом добавлении элемента, объем резервируемого участка выделяется с некоторым запасом - в два раза больше первоначального, но после определенного количества элементов объем вновь резервируемого участка памяти становится больше не в два раза, а на 25%.

С учетом нового понимания работы **_append_** ответ на вопрос: "*Что будет быстрее: добавлять один элемент в конец существующего слайса, или добавлять к слайсу из одного элемента существующий слайс?*" уже более прозрачен. Можно предположить, что во втором случае по сравнению с первым будет больше аллокаций памяти, что может прямым образом отразиться на скорости работы.

Так плавно мы подошли к бенчмаркам. С помощью бенчмарков можно оценить нагрузку алгоритма на самые критичные ресурсы такие как время выполнения и оперативная память.

Давайте приступим и напишем бенчмарк для оценки всех четырех наших алгоритмов, и также оценим какой прирост может нам дать отказ от сортировки (чтобы понимать какая часть всего времени тратится именно на сортировку).

Код бенчмарка:

**package **services

**import **(

  **"testing"**

**  "sort"**

)

**type **GameBuild **struct **{

  Id                  int

  ExtractedSize       int64

  PatchUrls           string

  ReversePatchUrls    string

  RPatchExtractedSize int64

}

**type **GameBuilds []*GameBuild

**func **(a GameBuilds) Len() int           { **return **len(a) }

**func **(a GameBuilds) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

**func **(a GameBuilds) Less(i, j int) bool { **return **a[i].Id < a[j].Id }

**func **prepare() GameBuilds {

  **var **builds GameBuilds

  **for **i := 0; i < 100000; i++ {

     builds = append(builds, &GameBuild{Id: i})

  }

  **return **builds

}

**func **BenchmarkF1(b *testing.B) {

  data := prepare()

  builds := make(GameBuilds, len(data))

  b.ResetTimer()

  **for **i := 0; i < b.N; i++ {

     **var **tempList GameBuilds

     copy(builds, data)

     **for **i := len(builds) - 1; i >= 0; i-- {

        b := builds[i]

        b.PatchUrls, b.ExtractedSize = b.ReversePatchUrls, b.RPatchExtractedSize

        tempList = append(tempList, b)

     }

  }

}

**func **BenchmarkF2(b *testing.B) {

  data := prepare()

  builds := make(GameBuilds, len(data))

  b.ResetTimer()

  **for **i := 0; i < b.N; i++ {

     **var **tempList GameBuilds

     copy(builds, data)

     **for **_, build := **range **builds {

        build.PatchUrls, build.ExtractedSize = build.ReversePatchUrls, build.RPatchExtractedSize

        tempList = append([]*GameBuild{build}, tempList...)

     }

  }

}

**func **BenchmarkF3(b *testing.B) {

  data := prepare()

  builds := make(GameBuilds, len(data))

  b.ResetTimer()

  **for **i := 0; i < b.N; i++ {

     copy(builds, data)

     **for **_, build := **range **builds {

        build.PatchUrls, build.ExtractedSize = build.ReversePatchUrls, build.RPatchExtractedSize

     }

     sort.Sort(sort.Reverse(builds))

  }

}

**func **BenchmarkF4(b *testing.B) {

  data := prepare()

  builds := make(GameBuilds, len(data))

  b.ResetTimer()

  **for **i := 0; i < b.N; i++ {

     copy(builds, data)

     **for **_, build := **range **builds {

        build.PatchUrls, build.ExtractedSize = build.ReversePatchUrls, build.RPatchExtractedSize

     }

     sort.Slice(builds, **func**(i int, j int) bool {

        **return **builds[i].Id > builds[j].Id

     })

  }

}

**func **BenchmarkF5(b *testing.B) {

  data := prepare()

  builds := make(GameBuilds, len(data))

  b.ResetTimer()

  **for **i := 0; i < b.N; i++ {

     copy(builds, data)

     **for **_, build := **range **builds {

        build.PatchUrls, build.ExtractedSize = build.ReversePatchUrls, build.RPatchExtractedSize

     }

  }

}

Запустим бенчмарк командой:

**_go test -bench=. -benchmem_**

Результаты вычислений для слайсов размеров: 10, 100, 1000, 10000, 100000 приведены на графике ниже. По оси абсцисс отложен размер итерируемого слайса, по оси ординат время вычисления в секундах.

<span style="display:block;text-align:center">
<img src="/static/bench_1.png" width="450px">
<br>Рис. 1 Время выполнения операции
</span>


<span style="display:block;text-align:center">
<img src="/static/bench_2.png" width="450px">
<br>Рис. 2  Количество аллокаций памяти на одну операцию
</span>



Как видно из графика, можно увеличивать массив, но конечный результат уже в принципе различим на слайсе из 10 элементов. Исходя из этого результата,  ни один из альтернативных алгоритмов: F2, F3, F4 не смог превзойти по скорости работы изначальный алгоритм (F1). Даже не смотря на то, что у всех кроме первого количество аллокаций памяти незначительно меньше, чем у изначального. Первый алгоритм (F2) с добавлением элемента в начало слайса оказался самым неэффективным - как и предполагалось в нем слишком много аллокаций памяти. настолько, что использовать его в продуктовой разработке категорически нельзя. Алгоритм с использованием встроенной функции сортировки реверсом (F3) существенно медленнее изначального. И только алгоритм с сортировкой  **_sort.Slice_** сравним по скорости с изначальным алгоритмом, но при этом тоже слегка уступает ему. 

Также можно заметить, что отказ от сортировки (F5) дает существенное ускорение. Поэтому, если очень хочется переписать код, то можно пойти в этом направлении, например, поднимая изначальный слайс builds из базы данных отсортировать его по id не ASC, а DESC средствами базы. Но при этом мы вынуждены выйти за границы рассматриваемого участка кода, что влечет за собой риск привнести множественные изменения. 

**Заключение**

Нет большого смысла тратить свое время на рассуждения о том будет тот или иной код быстрее. Пишите бенчмарки. Написать бенчмарк в Go практически ничего не стоит. 

Go только на первый взгляд довольно простой язык. Вездесущее правило 80/20  применимо и здесь. Как  раз эти 20% - это те тонкости внутреннего устройства языка, знание которых отделяют новичка от опытного разработчика.  Практика написания бенчмарков для разрешения своих вопросов - это один их самых дешевых и эффективных способов получить как ответ на вопрос                   так и более глубокое понимание внутреннего устройства и принципов работы языка Go.
