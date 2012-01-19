
.. _plugins:

Разработка  расширений на Python
================================


Для разработки расширений можно использовать язык программирования Python.
По сравнению с классическими расширениями, написанными на C++, их легче
разрабатывать, понимать, поддерживать и распространять в силу динамической
природы самого Python.

Расширения на Python перечисляются в Менеджере модулей QGIS наравне с
расширениями на C++. Поиск расширений выполняется в следующих каталогах:

    * UNIX/Mac: :file:`~/.qgis/python/plugins` и :file:`(qgis_prefix)/share/qgis/python/plugins`
    * Windows: :file:`~/.qgis/python/plugins` и :file:`(qgis_prefix)/python/plugins`

В Windows домашний каталог (обозначенный выше как :file:`~`) обычно выглядит
как :file:`C:\\Documents and Settings\\(user)`. Вложенные каталоги в этих
папках рассматриваются как пакеты Python, которые можно загружать в QGIS
как расширения.

Шаги:

1. *Идея*: Прежде всего нужна идея для нового расширения QGIS.
   Зачем это нужно?
   Какую задачу необходимо решить?
   Может, есть готовое расширения для решения этой задачи?

2. *Создание файлов*: Подробнее этот шаг описан ниже.
   Точка входа (:file:`__init.py__`).
   Тело расширения (:file:`plugin.py`).
   Форма QT-Designer (:file:`form.ui`), со своим :file:`resources.qrc`.

3. *Реализация*: Пишем код в :file:`plugin.py`

4. *Тестирование*: Закройте и снова откройте QGIS, загрузите своё расширение.
   Проверьте, что всё работает как надо.

5. *Публикация*: опубликуйте своё расширение в репозитории QGIS или настройте
   свой собственный репозиторий в качестве "арсенала" личного "ГИС вооружения"


Разработка расширения
---------------------

С момента введения поддержки Python в QGIS появилось множество расширений ---
на странице `Plugin Repositories <http://www.qgis.org/wiki/Python_Plugin_Repositories>`_
можно найти некоторые из них. Исходный код этих расширений можно использовать,
чтобы узнать больше о программировании с PyQGIS, а также для того, чтобы
удостовериться, что разработка не дублируется. Готовы к созданию расширения,
но отсутствует идея? На странице `Python Plugin Ideas <http://www.qgis.org/wiki/Python_Plugin_Ideas>`_
собрано много идей и пожеланий!


Создание необходимых файлов
---------------------------

Ниже показано содержимое каталога нашего демонстрационного расширения::

  PYTHON_PLUGINS_PATH/
    testplug/
      __init__.py
      plugin.py
      resources.qrc
      resources.py
      form.ui
      form.py

Для чего используются файлы:

* __init__.py = Точка входа расширения. Содержит общую информацию, версию
  расширения, его название и основной класс.
* plugin.py = Основной код расширения. Содержит информацию обо всех действиях,
  доступных в расширении, а также основной код.
* resources.qrc = XML-документ, созданный QT-Designer. Здесь хранятся
  относительные пути к ресурсам форм.
* resources.py = Понятная Python версия вышеописанного файла.
* form.ui = Интерфейс пользователя (GUI), созданный в QT-Designer.
* form.py = Конвертированная в код Python версия вышеописанного файла.

`Здесь <http://pyqgis.org/builder/plugin_builder.py>`_ и `вот здесь <http://www.dimitrisk.gr/qgis/creator/>`_
можно найти два способа автоматической генерации базовых файлов (скелета)
типового Python расширения для QGIS. Это упростит работу и поможет быстрее
начать разработку типового расширения.

Написание кода
--------------

__init__.py
^^^^^^^^^^^

Прежде всего, Менеджер модулей должен получить основные сведения о расширении,
такие как его название, описание и т.д. Файл :file:`__init__.py` именно то
место, где должна быть эта информация::

  def name():
    return "My testing plugin"

  def description():
    return "This plugin has no real use."

  def version():
    return "Version 0.1"

  def qgisMinimumVersion():
    return "1.0"

  def authorName():
    return "Developer"

  def classFactory(iface):
    # загружаем класс TestPlugin из файла testplugin.py
    from testplugin import TestPlugin
    return TestPlugin(iface)

plugin.py
^^^^^^^^^

Стоит сказать несколько слов о функции ``classFactory()``, которая вызывается
когда расширение загружается в QGIS. Она получает ссылку на экземпляр класса
:class:`QgisInterface` и должна вернуть экземпляр класса вашего расширения ---
в нашем случае этот класс называется``TestPlugin``. Ниже показано он должен
выглядеть (например, :file:`testplugin.py`)::

  from PyQt4.QtCore import *
  from PyQt4.QtGui import *
  from qgis.core import *

  # загружаем ресурсы Qt из файла resouces.py
  import resources

  class TestPlugin:

    def __init__(self, iface):
      # сохраняем ссылку на интерфейс QGIS
      self.iface = iface

    def initGui(self):
      # создаем действия для запуска расширения или его настройки
      self.action = QAction(QIcon(":/plugins/testplug/icon.png"), "Test plugin", self.iface.mainWindow())
      self.action.setWhatsThis("Configuration for test plugin")
      self.action.setStatusTip("This is status tip")
      QObject.connect(self.action, SIGNAL("triggered()"), self.run)

      # добавляем кнопку на панель и пункт в меню
      self.iface.addToolBarIcon(self.action)
      self.iface.addPluginToMenu("&Test plugins", self.action)

      # подключаемся к сигналу renderComplete, который посылается по завершению отрисовки карты
      QObject.connect(self.iface.mapCanvas(), SIGNAL("renderComplete(QPainter *)"), self.renderTest)

    def unload(self):
      # удаляем пункт меню и кнопку на панели
      self.iface.removePluginMenu("&Test plugins",self.action)
      self.iface.removeToolBarIcon(self.action)

      # отключаемся от сигнала карты
      QObject.disconnect(self.iface.MapCanvas(), SIGNAL("renderComplete(QPainter *)"), self.renderTest)

    def run(self):
      # создаем и показываем диалог настройки или выполняем что-то еще
      print "TestPlugin: run called!"

    def renderTest(self, painter):
      # рисуем на карте, используя painter
      print "TestPlugin: renderTest called!"


В расширении обязательно должны присутствовать функции ``initGui()`` и
``unload()``. Эти функции вызываются когда расширение загружается и
выгружается.

Файл ресурсов
^^^^^^^^^^^^^

Как видно в примере выше, в ``initGui()`` мы использовали иконку из файла
ресурсов (в нашем случае он называется :file:`resources.qrc`)::

  <RCC>
    <qresource prefix="/plugins/testplug" >
       <file>icon.png</file>
    </qresource>
  </RCC>

Хорошим тоном считается использование префикса, это позволит избежать
конфликтов с другими расширениями или с частями QGIS. Если префикс не задан,
можно получить не те ресурсы, которые нужны. Теперь сгенерируем файл
ресурсов на Python. Это делается командой :command:`pyrcc4`::

  pyrcc4 -o resources.py resources.qrc

Вот и все... ничего сложного :)
Если всё сделано правильно, то расширение должно отобразиться в Менеджере
модулей и загружаться в QGIS без ошибок. После его загрузки на панели появится
кнопка, а в меню --- новый пункт, нажатие на которые приведет к выводу
сообщения на терминал.

При работе над реальным расширением удобно вести разработку в другом (рабочем)
каталоге и создать makefile, который будет генерировать файлы интерфейса
и ресурсов, а также выполнять копирование расширения в каталог QGIS.

Документация
------------

*Этот способ создания документации требует наличия Qgis версии 1.5.*

Документацию к расширению можно готовить в виде файлов HTML. Модуль :mod:`qgis.utils`
предоставляет функцию :func:`showPluginHelp`, которая откроет файл справки
в браузере, точно так же как другие файлы справки QGIS.

Функция :func:`showPluginHelp`` ищет файлы справки в том же каталоге, где
находится вызвавший её модуль. Она по очереди будет искать файлы
:file:`index-ll_cc.html`, :file:`index-ll.html`, :file:`index-en.html`,
:file:`index-en_us.html` и :file:`index.html`, и отобразит первый найденный.
Здесь ``ll_cc`` --- язык интерфейса QGIS. Это позволяет включать в состав
расширения документацию на разных языках.

Кроме того, функция :func:`showPluginHelp` может принимать параметр packageName,
идентифицирующий расширение, справку которого нужно отобразить;
filename, который используется для переопределения имени файла с документацией;
и section, для передачи имени якоря (закладки) в документе, на который браузер
должен перейти.

Фрагменты кода
--------------

Здесь собраны фрагменты кода, полезные при разработке расширений.

Как вызвать метод по нажатию комбинации клавиш
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Добавьте в ``initGui()``::

  self.keyAction = QAction("Test Plugin", self.iface.mainWindow())
  self.iface.registerMainWindowAction(self.keyAction, "F7") # action1 is triggered by the F7 key
  self.iface.addPluginToMenu("&Test plugins", self.keyAction)
  QObject.connect(self.keyAction, SIGNAL("triggered()"),self.keyActionF7)

И в ``unload()``::

  self.iface.unregisterMainWindowAction(self.keyAction)

Метод, вызываемый по нажатию F7::

  def keyActionF7(self):
    QMessageBox.information(self.iface.mainWindow(),"Ok", "You pressed F7")

Как управлять видимостью слоя (временное решение)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

*Примечание:* в QGIS 1.5 появился класс :class:`QgsLegendInterface`, позволяющий
управлять списком слоёв легенды.

Так как в настоящее время методы прямого доступа к слоям легенды отсутствуют,
в качестве временно решения для управления видимостью слоёв можно использовать
решение на основе изменения прозрачности слоя::

  def toggleLayer(self, lyrNr):
    lyr = self.iface.mapCanvas().layer(lyrNr)
    if lyr:
      cTran = lyr.getTransparency()
      lyr.setTransparency(0 if cTran > 100 else 255)
      self.iface.mapCanvas().refresh()

Метод принимает номер слоя в качестве параметры (0 соответствует самому
верхнему) и вызывается так::

  self.toggleLayer(3)

Как получить доступ к таблице атрибутов выделенных объектов
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  def changeValue(self, value):
    layer = self.iface.activeLayer()
    if(layer):
      nF = layer.selectedFeatureCount()
      if (nF > 0):
      layer.startEditing()
      ob = layer.selectedFeaturesIds()
      b = QVariant(value)
      if (nF > 1):
        for i in ob:
        layer.changeAttributeValue(int(i),1,b) # 1 соответствует второй колонке
      else:
        layer.changeAttributeValue(int(ob[0]),1,b) # 1 соответствует второй колонке
      layer.commitChanges()
      else:
        QMessageBox.critical(self.iface.mainWindow(),"Error", "Please select at least one feature from current layer")
    else:
      QMessageBox.critical(self.iface.mainWindow(),"Error","Please select a layer")


Метод принимает один параметр (новое значения атрибута выделенного объекта(ов))
и вызывается как::

  self.changeValue(50)


Как выполнять отладку при помощи PDB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Сначала добавьте следующий код в место, которое будет отлаживаться::

 # для отладки используем pdb
 import pdb
 # устанавливаем точку останова
 pyqtRemoveInputHook()
 pdb.set_trace()

Затем запускаем QGIS из командной строки.

В Linux:

:command:`$ ./Qgis`

В Mac OS X:

:command:`$ /Applications/Qgis.app/Contents/MacOS/Qgis`

Когда приложение достигнет точки останова, консоль станет доступной и можно
будет вводить команды!

Тестирование
------------

Публикация расширения
---------------------

Если после создания расширения вы решите, что оно может быть полезно
и другим пользователям --- не бойтесь загрузить его в репозиторий
`PyQGIS plugin repository <http://pyqgis.org/>`_. На этой же странице
можно найти инструкции по подготовке пакета, следование которым избавит
от проблем с установкой расширения через Установщик модулей.
В случае, когда нужно настроить собственный репозиторий, создайте простой
XML документ, описывающий все расширения и их метаданные. Пример файла
можно найти на странице `Python plugin repositories <http://www.qgis.org/wiki/Python_Plugin_Repositories>`_.

Примечание: настройка IDE в Windows
-----------------------------------

При использовании Linux разработка расширений не требует дополнительных
настроек. Но в при использовании Windows необходимо убедиться, что
и QGIS, и интерпретатор используют одни и те же переменные окружения и
библиотеки. Наиболее простой и быстрый способ сделать это --- модифицировать
командный файл для запуска QGIS.

Если используется установщик OSGeo4W, командный файл можно найти в каталоге
bin папки, куда выполнена установка OSGeo4W. Ищите что-то похожее на
:file:`C:\\OSGeo4W\\bin\\qgis-unstable.bat`.

Далее будет описана настройка `Pyscripter IDE <http://code.google.com/p/pyscripter>`_.
Настройка других сред разработки может несколько отличаться:

* Сделайте копию qgis-unstable.bat и переименуйте её в pyscripter.bat.
* Откройте это файл в редакторе. Удалите последнюю строку, которая отвечает
  за запуск QGIS.
* Добавьте строку для запуска pyscripter с параметром, указывающим на
  используемую версию Python. QGIS 1.5 использует Python 2.5.
* Добавьте еще один аргумент, указывающий на каталог, где pyscripter должен
  искать библиотеки Python, используемые qgis. Обычно это каталог bin
  папки, куда установлен OSGeo4W::

    @echo off
    SET OSGEO4W_ROOT=C:\OSGeo4W
    call "%OSGEO4W_ROOT%"\bin\o4w_env.bat
    call "%OSGEO4W_ROOT%"\bin\gdal16.bat
    @echo off
    path %PATH%;%GISBASE%\bin
    start C:\pyscripter\pyscripter.exe --python25 --pythondllpath=C:\OSGeo4W\bin

Теперь при запуске этого командного файла установятся необходимые переменные
окружения и будет запущен pyscripter.