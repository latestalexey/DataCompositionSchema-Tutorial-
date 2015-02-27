# DataCompositionSchema-Tutorial-
Dedicated to beginners in 1C programming who want to master the basics of "Data composition schema" object

Language: Russian

Данный туториал будет полезен тем, кто уже имеет навыки работы с СКД и хотел бы далее совершенствовать свои знания и навыки. Материал, который содержится в туториале ,  описывает процесс создания и настройки СКД посредством встроенного языка 1С. Если вы никогда не работали с СКД советую ознакомится с ее основами: Е.Ю. Хрусталева «Разработка сложных отчетов в 1С Предприятие 8»

Итак начнем.

Для начала работы создадим:

1.	Справочник «Номенклатура» (иерархия групп и элементов) и справочник «Единицы измерения». 

2.	Перечисление «Типы цен» со значениями «Оптовая» и «Розничная».

3.	Независимый периодический регистр сведений «Цены номенклатуры»,  периодичность: в пределах дня. Измерения: номенклатура (СправочникСсылка «Номенклатура»), тип цены (ПеречислениеСсылка «Типы цен»). Ресурсы: Цена (16/2). Реквизиты: единица измерения (СправочникСсылка «Единицы измерения»).

4.	Отчет «Прайс-лист».

Над отчетом поговорим подробнее. В новом отчете добавим новую схему компоновки данных. В макетах у нас появится «ОсновнаяСхемаКомпоновкиДаннах». Откроем ее двойным кликом. На закладке «Наборы данных» добавим новый набор типа «Запрос».
Текст запроса:

ВЫБРАТЬ

	ЦеныНоменклатурыСрезПоследних.Номенклатура,
	
	ЦеныНоменклатурыСрезПоследних.Цена,
	
	ЦеныНоменклатурыСрезПоследних.БазоваяЕдиницаИзмерения КАК ЕдИзм
	
ИЗ

РегистрСведений.ЦеныНоменклатуры.СрезПоследних(, ТипЦены = &ТипЦены) КАК ЦеныНоменклатурыСрезПоследних

В параметрах добавим  параметр «ТипЦены» .

Настройки:
Конструктор настроек, поля: номенклатура, цена, единица измерения;  группировка: номенклатура(только иерархия);

Параметр «ТипЦены» включим в пользовательские настройки.

   В итоге мы имеем готовый отчет без единой строчки кода. Как мы видим  создавать отчеты с помощью конструктора СКД  довольно просто и удобно.
  Но на практике часто бывают случаи когда код все таки требуется писать. Поэтому перейдем сразу к написанию кода.

   Предположим мы хотим отображать наш прайс из формы списка нашого справочника. Создаем форму списка. Создаем команду  «ПоказатьПрайс» и в модуле формы пишем следующий код:


&НаКлиенте

Процедура  ПоказатьПрайс (Команда)

   Результат  =  ВывестиПрайс();
   
   Результат.ОтображатьСетку      =  Ложь;
   
   Результат.ОтображатьЗаголовки  =  Ложь;
   
   Результат.Показать();

КонецПроцедуры


&НаСервереБезКонтекста

Функция  ВывестиПрайс ()

     СхемаКомпоновкиДанных  =  Отчеты.ПрайсЛист.ПолучитьМакет("ОсновнаяСхемаКомпоновкиДанных");
 
      Настройки  =  СхемаКомпоновкиДанных.НастройкиПоУмолчанию;
   
      КомпоновщикМакета  =  Новый КомпоновщикМакетаКомпоновкиДанных;
      
      МакетКомпоновки   = КомпоновщикМакета.Выполнить(СхемаКомпоновкиДанных, Настройки);
 
      ПроцессорКомпоновки  =  Новый ПроцессорКомпоновкиДанных;
      
      ПроцессорКомпоновки.Инициализировать(МакетКомпоновки);
 
      ТабДок  =  Новый  ТабличныйДокумент;

     ПроцессорВывода  =  Новый ПроцессорВыводаРезультатаКомпоновкиДанныхВТабличныйДокумент;
     
     ПроцессорВывода.УстановитьДокумент(ТабДок);
     
     ПроцессорВывода.Вывести(ПроцессорКомпоновки);
  
     Возврат  ТабДок; 
     
КонецФункции

  Как видно из кода, из обработчика команды вызывается функция которая программно создает отчет на основе СКД. Результат отчета выводится в табличный документ и возвращается обратно на клиент. За основу мы берем уже готовый макет СКД : «ОсновнаяСхемаКомпоновкиДанных».

# Отчет в СКД полностью на встроенном языке <Head/>

Теперь создадим полностью все с нуля.  Для этого нам понадобится создать новый отчет, а в нем создадим форму отчета.
  В событии формы «ПриСозданииНаСервере» создадим схему компоновки данных, добавим в нее набор данных и опишем в нем запрос и добавим поля набора. На закладке «Параметри» добавим параметр: «ТипЦены».
  Схему поместим во временное хранилище. Адресс хранилища запишем в реквизит формы типа: «Строка», который назовем «Адресс».

Примечание: 

   Добавляя в набор поля ресурсов мы не рассчитываем по ним итоги. Так как ресурсы в данном примере не сворачиваются агрегатной функцией «СУММА» мы будем добавлять их в поля таким же способом каким мы добавляли измерения.

![code1](https://cloud.githubusercontent.com/assets/11144999/6413751/29cf7056-be9b-11e4-85b0-8a8d68da1d20.jpg)
![code2](https://cloud.githubusercontent.com/assets/11144999/6413979/fa03d1e4-be9c-11e4-89de-63015a7bf8e6.jpg)

Опишем собственную команду формирования отчета:

![code3](https://cloud.githubusercontent.com/assets/11144999/6414181/3a346ea8-be9e-11e4-86f4-b95b8956fa04.jpg)

  При вызове команды будет вызыватся  функция «Вывести прайс». В ней  мы получаем макет СКД из временного хранилища, устанавливаем в нем настройки с помощью функции «УстановитьНастройки». Далее описан стандартный механизм компоновки макета и вывода результата в табличный документ:

![code4](https://cloud.githubusercontent.com/assets/11144999/6414330/30789b90-be9f-11e4-9355-870f7a7e7778.jpg)

Установить настройки мы можем с помощью настоек по умолчанию (НастройкиПоУмолчанию()) или компоновщика настроек (КомпоновщикНастроек.Настройки).

НастройкиПоУмолчанию  -  это настройки которые были сделаны при редактировании схемы, т.е. некий пресет с которым будет создаваться отчет.

КомпоновщикНастроек.Настройки - это текущие настройки с которыми формируется отчет.

То есть правильнее было бы оформить наши настройки в настройках по умолчанию.  Так и поступим:
