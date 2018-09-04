### Preface
Этот аддон предоставляет возможность создавать вспомогательные туры для экранов в приложениях
[CUBA platform](https://www.cuba-platform.com).

### Dependencies
 - [Vaadin Addon for creating Product Tours](https://github.com/Juchar/product-tour)

### Main features
Компоненты, используемые для создания и запуска тура:
- Tour
- Step
- StepButton
- TourStartAction
- TourParser

### Usage
Select a version of the add-on which is compatible with the platform version used in your project:

| Platform Version| Add-on Version|
|:----------------|:--------------|
| 6.9.x           | 1.0.0         |


Add custom application component to your project (change the version part if needed):
```
com.haulmont.addon.tour:tour-global:0.1-SNAPSHOT
```

### Description
Этот аддон содержит набор компонентов, позволяющих создавать вспомогательные туры по пользовательским экранам. 

Для создания тура необходим компонент, реализующий интерфейс Tour. С помощью метода `addStep` к туру можно привязать
компоненты, реализующие интерфейс Step. Для взаимодействия между шагами тура с помощью метода `addStepButton` можно
привязать к шагам кнопки, реализующие интерфейс StepButton. Действия для кнопок можно использовать готовые из
`TourActionType` и `StepActionType` или собственные.

Также создать тур можно при помощи `TourParser` из json вызовом метода `parseTour`, который на вход принимает json, 
messagePack и window to extend.

Запуск тура осуществляется вызовом действия `TourStartAction`. Задать, будет ли тур запускаться каждый раз при переходе
на экран или только при первом переходе, можно с помощью метода `setSettingsEnabled`.

### Quick start
To use this add-on in a CUBA Studio project, in the project properties add add-on like custom application component and
save settings.
```
com.haulmont.addon.tour:tour-global:0.1-SNAPSHOT
```

#### Sample task
Давайте попробуем создать небольшое приложение, содержащее два экрана с турами.

В этом приложении мы реализуем следующие возможности:
- создание туров на экранах приложения
- задание разных опций для шагов тура
- задание параметра запуска туров

#### Step-by-step guide
Create a new CUBA project and add the given add-on to it.

![](screenshots/1-adding-addon.png "Adding add-on")

In the DATA MODEL tab create a new entity "Product" with two attributes.

![](screenshots/2-create-new-entity.png "Creating a new entity")

Create entity browser and entity editor screens via Generic UI screen.

Мы добавим кнопку для запуска тура в `ProductBrowse`, а также реализуем запуск туров при открытии экранов.

Для создания кнопки добавьте следующий блок к `buttonsPanel` в product-browse.xml:

```xml
<button id="tourButton"
        caption="Tutorial" 
        invoke="startTour"/>
```

При инициализации страницы `ProductBrowse` мы будем парсить тур из json файла.

productBrowseTour.json:
```JSON
[
  {
    "id": "step1",
    "text": "tour.tutorialStartedText",
    "title": "tour.tutorialStartedTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "cancellable": "true",
    "buttons": [
      {
        "caption": "tour.cancel",
        "style": "danger",
        "action": "tour:cancel",
        "enabled": "true"
      },
      {
        "caption": "tour.next",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  },
  {
    "id": "step2",
    "text": "tour.createButtonText",
    "title": "tour.createButtonTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "attachTo": "createBtn",
    "anchor": "right",
    "buttons": [
      {
        "caption": "tour.back",
        "style": "primary",
        "action": "tour:back",
        "enabled": "true"
      },
      {
        "caption": "tour.next",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  },
  {
    "id": "step3",
    "text": "tour.editButtonText",
    "title": "tour.editButtonTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "attachTo": "editBtn",
    "anchor": "right",
    "buttons": [
      {
        "caption": "tour.back",
        "style": "primary",
        "action": "tour:back",
        "enabled": "true"
      },
      {
        "caption": "tour.next",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  },
  {
    "id": "step4",
    "text": "tour.removeButtonText",
    "title": "tour.removeButtonTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "attachTo": "removeBtn",
    "anchor": "bottom",
    "buttons": [
      {
        "caption": "tour.back",
        "style": "primary",
        "action": "tour:back",
        "enabled": "true"
      },
      {
        "caption": "tour.next",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  },
  {
    "id": "step5",
    "text": "tour.filterPanelText",
    "title": "tour.filterPanelTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "attachTo": "filter",
    "anchor": "bottom",
    "modal": "true",
    "buttons": [
      {
        "caption": "tour.back",
        "style": "primary",
        "action": "tour:back",
        "enabled": "true"
      },
      {
        "caption": "tour.finish",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  }
]
```

ProductBrowse.java:
```Java
@Inject
protected Resources resources;

protected Tour tour;
    
@Override
public void init(Map<String, Object> params) {
    super.init(params);
    
    createTour();
}
    
protected void createTour() {
    String file = resources.getResourceAsString("com/company/touraddondemo/web/product/productBrowseTour.json");
    TourParser tourParser = AppBeans.get(TourParser.class);
    tour = tourParser.parseTour(file, getMessagesPack(), this);
}
```

Далее напишем метод создания действия для запуска тура. Отключим настройки, задающие запуск тура только при первом 
переходе на экран.

```Java
public void startTour() {
    TourStartAction tourStartAction = TourStartAction.create(tour);
    tourStartAction.setSettingsEnabled(false);
    tourStartAction.actionPerform(this);
}
```

Этот метод будет вызываться при нажатии кнопки `tourButton`. Также добавим вызов этого метода при инициализации экрана.

Для `ProductEdit` будем парсить тур из json таким же образом, только укажем путь к файлу productEditTour.json.
И в методе инициализации создадим действие для запуска тура без отключения дополнительных настроек.

productEditTour.json:
```JSON
[
  {
    "id": "step1",
    "text": "tour.editStartedText",
    "title": "tour.editStartedTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "cancellable": "true",
    "buttons": [
      {
        "caption": "tour.cancel",
        "style": "danger",
        "action": "tour:cancel",
        "enabled": "true"
      },
      {
        "caption": "tour.next",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  },
  {
    "id": "step2",
    "text": "tour.fieldGroupText",
    "title": "tour.fieldGroupTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "attachTo": "fieldGroup",
    "anchor": "right",
    "buttons": [
      {
        "caption": "tour.back",
        "style": "primary",
        "action": "tour:back",
        "enabled": "true"
      },
      {
        "caption": "tour.next",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  },
  {
    "id": "step3",
    "text": "tour.windowActionsText",
    "title": "tour.windowActionsTitle",
    "width": "400",
    "textContentMode": "html",
    "titleContentMode": "html",
    "attachTo": "windowClose",
    "anchor": "right",
    "buttons": [
      {
        "caption": "tour.back",
        "style": "primary",
        "action": "tour:back",
        "enabled": "true"
      },
      {
        "caption": "tour.finish",
        "style": "friendly",
        "action": "tour:next",
        "enabled": "true"
      }
    ]
  }
]
```

Для отображения локализованных сообщений необходимо добавить их в messages.properties:

```properties
tour.createButtonText = <p>This is a <b>create button</b>.</p> <p>Press the button to open an editor screen to \
  <b>create</b> a new entity.</p>
tour.editButtonText = <p>This is an <b>edit button</b>.</p> <p>Select an entity and press the button to open an \
  editor screen to <b>edit</b> the entity.</p>
tour.removeButtonText = <p>This is a <b>remove button</b>.</p> <p>Select an entity and press the button to remove \
  the entity.</p>
tour.filterPanelText = <p>This is a <b>filter panel</b>.</p> <p>You may choose conditions to filter entities.</p>
tour.fieldGroupText = <p>This is a <b>field group</b> consisting of text fields.</p> \
  <p>You may fill the fields by your own data.</p>
tour.windowActionsText = <p>These are window <b>actions</b>.</p> <p>You are able to <b>confirm</b> or <b>decline</b> \
  changes.</p>
tour.tutorialStartedText = <p>This tutorial will show you and describe some UI components.</p> \
  <p>This tour starts every time.</p>
tour.editStartedText = <p>This is an <b>editor screen</b>.</p> <p>Here you may <b>create</b> or <b>edit</b>\
   entities.</p> <p>This tour starts only once.</p>
tour.editStartedTitle = <b>Editor</b> screen
tour.tutorialStartedTitle = Tutorial has <b>started</b>!
tour.createButtonTitle = <b>Create button</b>
tour.editButtonTitle = <b>Edit button</b>
tour.removeButtonTitle = <b>Remove button</b>
tour.filterPanelTitle = <b>Filter Panel</b>
tour.fieldGroupTitle = <b>Field group</b>
tour.windowActionsTitle = Window <b>actions</b>
tour.back = Back
tour.next = Next
tour.cancel = Cancel
tour.finish = Finish
```

Next, we need to create a theme extension in Project Properties.

![](screenshots/3-theme-extention.png "Create extension theme")  

Теперь при переходе на экран `ProductBrowse` каждый раз пользователь будет видеть тур. Также у него будет возможность 
запускать тур нажатием на кнопку с заголовком "Tutorial". При переходе на `ProductEdit` будет запущен тур при первом 
переходе на экран для каждого пользователя.

This is how the ProductBrowse screen looks in the running app:

![](screenshots/4-run-application-product-browse.png "View of the ProductBrowse screen in the running app")

This is how the ProductEdit screen looks in the running app:

![](screenshots/5-run-application-product-edit.png "View of the ProductEdit screen in the running app")
