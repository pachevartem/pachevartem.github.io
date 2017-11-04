---
layout: post
title:  "Unity3D. Post"
date:   2017-11-04 17:25:11 +1000
categories: jekyll update
---
# Unity3D. Коротко о Системе событий. Event System.
____________


## Задача 1. Вывести на экран количество нажатий левой кнопки мыши. 
### Способ 1. Update ++
Для этого создадим скрипт MouseController.cs. В сцене создадим объект GameController (empty) на который добавим компонент MouseController.cs. В MouseController.cs запишем следующий код: 

>~~~
> public int CountClickLeftMouse = 0;
> 
>    void Update()
>    {
>       if (Input.GetMouseButtonDown(0))
>       {
>          CountClickLeftMouse++;
>          Debug.Log(CountClickLeftMouse);
>       }
>    }
>~~~


Каждый кадр в методе ***Update*** проверяется нажатие на левую кнопку мыши (**Input.GetMouseButtonDown(0)**). По нажатию на кнопку инкрементируется поле ***CountClickLeftMouse*** и в консоль выводится количесвто нажатий. 

Заменим вывод в консоль на вывод нажатий клавишь на экран. Создадим еще один скрипт **UIController.cs** и так же добавим его к объекту **GameController**, а так же в сцене объект Text. (menu create-UI-text) и пернесем его в компонент **UIController**. В скрипте создадим ссылку на объект типа UnityEngine.UI.Text. Изменим скрипт **MouseController.cs**. и **UIController.cs**. В итоге иммем:

> **MouseController.cs**
> ____________
> ~~~
> using UnityEngine;
> 
> public class MouseController : MonoBehaviour
> {
> 
>    public UIController UiController;
>    public int CountClickLeftMouse = 0;
> 
>    private void Start()
>    {
>       UiController = GetComponent<UIController>();
>    }
> 
>    void Update()
>    {
>       if (Input.GetMouseButtonDown(0))
>       {
>          CountClickLeftMouse++;
>           UiController.ClickMouseLeftButton.text = CountClickLeftMouse.ToString();
>       }
>    }
> 
> }
> ~~~

&nbsp;

> **UIController.cs**
> ___________________
> ~~~
> using UnityEngine;
> using UnityEngine.UI;
> 
> public class UIController : MonoBehaviour
> {
>     public Text ClickMouseLeftButton;
> }
> ~~~

Так как оба компонента добавлены к одному объекту. Мы сможем с помощью **GetComponent** в начале загрузки приложения получить ссылку на компонент **UIController**. Теперь мы видим, что по центру экрана отображается число, отвечающее за количество нажатий на левую кнопку мыши.


### Способ 2. Свойства объекта.
В данном способе мы можем изменять текст на экране через свойства поля **CountClickLeftMouse**
Изменим код в **MouseController.cs**

>~~~
> 	 public UIController UiController;
> 
>    private int _countClickLeftMouse;
>    
>    private int CountClickLeftMouse {
>       get { return _countClickLeftMouse; }
>       set
>       {
>          _countClickLeftMouse = value;
>          UiController.ClickMouseLeftButton.text = _countClickLeftMouse.ToString();
>       }
>    }
> 
>    private void Start()
>    {
>       UiController = GetComponent<UIController>();
>    }
> 
>    void Update()
>    {
>       if (Input.GetMouseButtonDown(0))
>       {
>          CountClickLeftMouse++;
>       }
>    }
> ~~~

Данный подход хорош тем, что мы можем писать любые ограничения в свойствах. В некоторых ситуацияю очень удобно. Пример ипользования - оповещения о малом количестве жизней или маны персонажа. 

### Способ 3. Создать событие. Delegate
Немного изменим код в обоих скриптах. Получим: 

> **MouseController.cs**
> ____________
> ~~~
> using UnityEngine;
> 
> public class MouseController : MonoBehaviour
> {
>    public delegate void OnMouse();
>    public static OnMouse LeftClickMouse;
> 
>    void Update()
>    {
>       if (Input.GetMouseButtonDown(0))
>       {
>          LeftClickMouse();
>       }
>    }
> }
> ~~~

&nbsp;

> **UIController.cs**
> ___________________
> ~~~
> using UnityEngine;
> using UnityEngine.UI;
> 
> public class UIController : MonoBehaviour
> {
>     public Text ClickMouseLeftButton;
>     private int CountClickLeftMouse = 0;
> 
>     private void Start()
>     {
>         MouseController.LeftClickMouse += ChangeCountClickLeftMouse;
>     }
> 
>     private void ChangeCountClickLeftMouse()
>     {
>         CountClickLeftMouse++;
>         ClickMouseLeftButton.text = CountClickLeftMouse.ToString();
>     }
> }
> ~~~

&nbsp;

Что изменилось? **MouseController.cs** больше не имеет ссылок на **UIController.cs**. Так же в **MouseController.cs** появился **delegate** который будет оповещать всех о событии. Само событие выглядит как **public static OnMouse LeftClickMouse** именно этот экземпляр делегата с сигнатурой void будет хранить в себе методы, которые будут реагировать на событие. 

В классе **UIController.cs** в методе **Start** будет осуществляться <mark>подписка</mark> с помощью <mark>**+=**</mark> методов с одинаковой сигнатурой void() ***ChangeCountClickLeftMouse*** на **LeftClickMouse**.




## Задача 2. Разобраять глобальную систему получения сообщений. 



Смысл глобальной системы оповещений. Мягкая передача сообщений от любых объектов любым объектом. 

Для начала давайте определим базовый класс: 

Листинг BaseClass
>~~~
>     public class BaseMessage
>     {
>         public string name;
> 
>         public BaseMessage()
>         {
>             name = this.GetType().Name;
>         }
>     }
> ~~~

От данного класса буду наследоваться все остальные типы сообщений, которые вы захотите <mark>передавать</mark>

Теперь необходимо саму систему сообщений. За это будет отвечать скрипт MessagingSystem.cs. Определим его критерии:

* должен быть доступным глобально
* любой объект (MonoBehavior или нет) должен иметь возможность или анулировать подписку на сообщения определенного типа (<mark>то есть реализовать шаблон "Наблюдатель"</mark>)
* Подписываясь на сообщения, объект должен указать метод, вызовом которого ему будут передаваться сообщения
* система должна передавать сообщения всем получателям в разумные сроки, но не забрасывать их слишком быстро большим количеством одновременных запросов. 

Класс отвечающий за логику работы системы сообщений необходимо иметь в единственном лице, поэтому к нему будет применен патрен Singleton. 

Листинг кода универсального SingleTone.
>~~~
> using UnityEngine;
> 
> namespace myEventSystem
> {
>     public class SingletonAsComponent<T>: MonoBehaviour where T: SingletonAsComponent<T>
>     {
>         private static T __Instance;
> 
>         protected static SingletonAsComponent<T> _Instance
>         {
>             get
>             {
>                 if (!__Instance)
>                 {
>                     T[] managers = GameObject.FindObjectsOfType(typeof(T)) as T[];
>                     if (managers != null)
>                     {
>                         if (managers.Length ==1)
>                         {
>                             __Instance = managers[0];
>                             return __Instance;
>                         }
>                         else if (managers.Length>1)
>                         {
>                             Debug.LogError("You have more than one" + typeof(T).Name + " in the scene. You only need 1, it's a singleton");
>                             for (int i = 0; i < managers.Length; i++)
>                             {
>                                 T manager = managers[i];
>                                 Destroy(manager.gameObject);
>                             }
>                         }
>                     }
>                     GameObject go = new GameObject(typeof(T).Name, typeof(T));
>                     __Instance = go.GetComponent<T>();
>                     DontDestroyOnLoad(__Instance.gameObject);
>                 }
>                 return __Instance;
>             }
>             set
>             {
>                 __Instance = value as T;    
>             }
>         } 
>     }
> }
> ~~~

Определим сигнатуру делегата, который будет отвечать за передачу сообщений. 

>~~~
>public delegate bool MessageHandlerDelegate(BaseMessage message);
>~~~

Далее пишем класс ***MessagingSystem.cs***.
Листинг: 
>~~~
>  public class MessagingSystem: SingletonAsComponent<MessagingSystem>
>     {
>         
>         public static bool IsAlive
>         {
>             get
>             {
>                 if (Instance==null)
>                 {
>                     return false;
>                 }
>                 return Instance._alive;
>             }
>         }
>         private bool _alive = true;
> 
>         private void OnDestroy()
>         {
>             _alive = false;
>         }
> 
>         private void OnApplicationQuit()
>         {
>             _alive = false;
>         }
>         public static MessagingSystem Instance
>         {
>             get { return (MessagingSystem) _Instance; }
>             set { _Instance = value; }
>         }
>         
>         private Dictionary<string, List<MessageHandlerDelegate>> _listenerDict = new Dictionary<string, List<MessageHandlerDelegate>>();
> 
>         private float maxQueueProcessingTime = 0.166667f;
>         
>         public bool AttachListener(System.Type type, MessageHandlerDelegate handler)
>         {
>             if (type==null)
>             {
>                 Debug.Log("Messaging system: AttacjListener failed due to no messagy type specified");
>                 return false;
>             }
>             string msgName = type.Name;
>             if (!_listenerDict.ContainsKey(msgName))
>             {
>                 _listenerDict.Add(msgName,new List<MessageHandlerDelegate>());
>             }
>             List<MessageHandlerDelegate> listenerList = _listenerDict[msgName];
>             if (listenerList.Contains(handler))
>             {
>                 return false; 
>             }
>             listenerList.Add(handler);
>             return true;
>         }
>         
>         
>         private Queue<BaseMessage> _messegeQueue = new Queue<BaseMessage>();
> 
>         public bool QueueMessege(BaseMessage msg)
>         {
>             if (!_listenerDict.ContainsKey(msg.name))
>             {
>                 return false;
>             }
>             _messegeQueue.Enqueue(msg);
>             return true;
>         }
> 
>         public bool TriggerMessege(BaseMessage msg)
>         {
>             string msgName = msg.name;
>             if (!_listenerDict.ContainsKey(msgName))
>             {
>                 Debug.Log("MessagingSystem: Message \"" + msgName + "\" has no listeners!");
>                 return false;
>             }
>             List<MessageHandlerDelegate> listenerList = _listenerDict[msgName];
>             for (int i = 0; i < listenerList.Count; i++)
>             {
>                 if (listenerList[i](msg))
>                 {
>                     return true;
>                 }
>             }
>             return true;
> 
>         }
> 
>         public bool DetachListener(System.Type type, MessageHandlerDelegate handler)
>         {
>             if (type==null)
>             {
>                 Debug.Log("MessageSystem: DetachListener failed due to no message type specifield");
>                 return false;
>             }
>             string msgName = type.Name;
>             if (!_listenerDict.ContainsKey(type.Name))
>             {
>                 return false;
>             }
> 
>             List<MessageHandlerDelegate> listenerList = _listenerDict[msgName];
>             if (!listenerList.Contains(handler))
>             {
>                 return false;
>             }
>             listenerList.Remove(handler);
>             return true;
>         }
> 
>         private void Update()
>         {
>             float timer = 0.0f;
>             while (_messegeQueue.Count > 0)
>             {
>                 if (maxQueueProcessingTime > 0.0f)
>                 {
>                     if (timer > maxQueueProcessingTime)
>                     {
>                         return;
>                     }
>                 }
> 
>                 BaseMessage msg = _messegeQueue.Dequeue();
>                 if (!TriggerMessege(msg))
>                 {
>                     Debug.Log("Error when processing message: " + msg.name);
>                 }
>                 if (maxQueueProcessingTime > 0.01f)
>                 {
>                     timer += Time.deltaTime;
>                 }
>             }
>         }
>     } 
> }
> ~~~

Переменная ***_listenerDict*** – это словарь строк, отображаемых в списки ***MessageHandlerDelegates***. Этот словарь объединяет делегатов в списки по типам обрабатываемых ими сообщений. То есть по типу сообщения можно быстро получить список всех делегатов объектов, подписавшихся на этот тип сообщений, затем обойти список и послать сообщение каждому получателю. Метод ***AttachListener()*** требует передачи двух параметров, типа сообщения в форме ***System.Type*** и делегата ***MessageHandlerDelegate***, который будет вызываться для отправки сообщения.

**Очередь и обработка сообщений**. Для обработки сообщений система MessagingSystem должна поддерживать очередь входящих сообщений, чтобы их можно было обрабатывать в порядке поступления.

Метод ***public bool QueueMessage(BaseMessage msg)*** предварительно проверяет наличие такого типа сообщения в словаре. Дополнительно, для большей эффективности, определяется наличие получателей, подписавшихся на сообщение этого типа, и только потом сообщение добавляется в очередь для последующей обработки. Роль очереди играет закрытое свойство ***_messageQueue***.

Определение метода ***Update()***. Этот метод будет регулярно вызываться движком Unity. Он будет выполнять последовательный обход сообщений в очереди и для каждого будет определять,не слишком ли много времени прошло с момента начала его обработки. Защита на основе подсчета времени состоит в том, чтобы проверить превышение порогового значения. Это позволяет исключить приостановку игры системой обмена сообщениями, если в нее поступит слишком много сообщений за короткий промежуток времени. По истечении интервала времени, отведенного на обработку сообщений, она будет приостановлена и отложена до следующего кадра.

Метод **TriggerMessage()**, передающий сообщения получателям. **TriggerEvent()** – главная рабочая лошадка системы обмена сообщениями. Он извлекает список получателей сообщений данного типа и дает каждому возможность его обработать. Если один из делегатов вернет значение true, обработка текущего сообщения прекращается, и метод завершает работу, что позволит методу ***Update()*** перейти к обработке следующего сообщения. Обычно для рассылки сообщений используется метод ***QueueEvent()***, но с той же целью можно использовать метод ***TriggerEvent()***. Он позволяет отправителям заставить получателей обработать сообщение немедленно, не дожидаясь следующего события ***Update()***, действуя в обход механизма регулирования, и потому должен использоваться для отправки только критически важных сообщений, когда ожидание следующего кадра нарушит порядок игрового процесса.

Реализация пользовательского сообщения. Система обмена сообщениями готова, но теперь необходимо реализовать пример, помогающий понять, как ею пользоваться. Начнем с определения класса простого сообщения для передачи данных ***class MyCustomMessage : BaseMessage***. Объявление свойств сообщений как readonly гарантирует невозможность изменения данных после создания объекта. Это защитит содержимое сообщений от изменения при передаче от одного получателя к другому.
листинг:
> ~~~
> public class MyCustomMessage : BaseMessage {
>			public readonly int _intValue;
>			public readonly float _floatValue;
>
>			public MyCustomMessage(int intVal, float floatVal) 
>			{
>				_intValue = intVal;
>				_floatValue = floatVal;
>			}
>		}
> ~~~

***Регистрация сообщений.*** Следующий простой класс регистрируется в системе обмена сообщениями, передавая ей ссылку на метод HandleMyCustomMessage(),который должен получать сообщения типа MyCustomMessage:
>~~~ 
> public class TestMessageListener : MonoBehaviour {
>  void Start() {
>  		MessagingSystem.Instance.AttachListener( typeof(MyCustomMessage), this.HandleMyCustomMessage);
>  }
> 
>  bool HandleMyCustomMessage(BaseMessage msg) { 
> 		MyCustomMessage castMsg = msg as MyCustomMessage;
>  		Debug.Log (string.Format("Got the message! {0}, {1}", castMsg._intValue,castMsg._floatValue));
>  		return true;
>  }
> ~~~

При передаче объекта MyCustomMessage (откуда угодно!) этот объект будет извлекать сообщение с помощью своего метода HandleMyCustomMessage().Обработчик может выполнить приведение типа сообщения и обработать его своим уникальным способом. На это же сообщение могут подписаться другие классы, выполняющие другие действия в своем собственном методе (предполагается, что делегаты, обработавшие сообщение ранее, не вернули значения true).
Тип сообщения в аргументе msg метода HandleMyCustomMessage() известен, поскольку он был определен при регистрации в вызове метода AttachListener(). Благодаря этому можно быть уверенным в безопасности приведения типов и сэкономить время, отказавшись от проверки ссылки на значение null, хотя технически ничто не мешает использовать один и тот же делегат для обработки нескольких типов сообщений! В таких случаях придется реализовать определение типа полученного объекта сообщения и обработать его соответственно. Но лучше определить отдельный метод для каждого типа сообщений,чтобы не смешивать их обработку.Обратите внимание, что определение метода HandleMyCustomMessage соответствует сигнатуре функции MessageHandlerDelegate и передается в вызов AttachListener(). Именно так мы сообщаем системе, какой метод должен вызываться в случае появления сообщения данного типа, и гарантируем безопасность преобразования типов в делегатах. Если сигнатура функции имеет другое возвращаемое значение или другой список аргументов, она не может служить делегатом для передачи в метод AttachListener(), и это должно вызвать ошибку компиляции.Самое замечательное, что делегату можно дать любое имя. На практике чаще выбираются имена, соответствующие именам обрабатываемых сообщений. Тогда любой, кто возьмется читать код, легко поймет, для чего используется метод и какой тип сообщений он обрабатывает.
	***Отправка сообщений*** Наконец, реализуем пример отправки сообщения для тестирования системы! Следующий компонент передает экземпляр класса ***MyCustomMessage*** через систему обмена сообщениями после нажатия
клавиши пробел: 
Листинг
> ~~~
> public class TestMessageSender : MonoBehaviour {
> 		public void Update() {
>  			if (Input.GetKeyDown (KeyCode.Space)) {
>  				MessagingSystem.Instance.QueueMessage( new MyCustomMessage(5,13.355f));
>  				}
>  		}
> }
> ~~~

Если добавить в сцену объекты TestMessageSender и TestMessageListener и нажать клавишу пробела, в консоли должно появиться сообщение, информирующее об успехе тестирования.

Объект-одиночка MessagingSystem будет создан сразу после инициализации сцены, в вызове метода Start() объекта TestMessageListener,который зарегистрирует делегата HandleMyCustomMessage. Для создания
объекта-одиночки не требуется никаких дополнительных усилий.

Очистка сообщений
Поскольку сообщения являются объектами, они будут создаваться в куче динамически и уничтожаться вскоре после обработки получателями. Однако, как будет описано в главе 7 «Мастерство управления памятью», это ведет к необходимости уборки мусора из-за увеличения объема занятой памяти в куче с течением времени. Если приложение проработает достаточно долго, сборка мусора будет производиться в случайные моменты времени. Поэтому систему обмена сообщениями следует использовать экономно и избегать частой рассылки ненужных сообщений, сответствующих каждому обновлению. Еще более важной задачей является реализация своевременного исключения делегатов, объекты которых должны быть уничтожены или заново воссозданы. Если этого не сделать, в системе обмена сообщениями зависнут ссылки на делегатов, которые будут мешать сборщику мусора освобождать память, занятую уже уничтоженными объектами. По существу, каждому вызову AttachListener() должен соответствовать парный вызов DetachListener() в момент уничтожения объекта или если принято решение, что он больше не нуждается в получении сообщений. Следующий метод класса MessagingSystem отключает получателя определенного сообщения.

Обратите внимание на использование свойства IsAlive, объявленного в классе SingletonAsComponent. Это защитит от упомянутых выше проблем при завершении работы приложения, когда нет гарантии, что объект-одиночка будет уничтожен последним.

__________________
__________________
__________________
__________________
__________________
__________________
__________________
__________________
__________________
__________________
__________________
__________________
## Практика. Показать разницу на примерах.

* Создать проект. 
* Скачать 4 звука. ([пример сайта](https://wav-library.net/))

### Задача1. Создать SoundController.cs и хранить в нем лист звуков.



























