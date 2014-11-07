---
layout: post
published: true
title: UITraitCollection, UISplitView и подводные камни в iOS8.
mathjax: false
featured: false
comments: false
category: iOS-Development
tags: ios app ios8 objective-c
modified: "2014-11-07"

---

За несколько недель разработки проекта для iOS8, стало понятно, что на самом деле, с одной стороны Apple проделали громадную работу над почти всеми UI элементами, но с другой стороны - не все так радужно, как они рассказывали на WWDC. Советую сначала посмотреть [сессии по материалу](https://developer.apple.com/videos/wwdc/2014/) и почитать [документацию](https://developer.apple.com/Library/ios/documentation/UIKit/Reference/UITraitSet_ClassReference/index.html).

##UITraitCollection
###Что это?
Хотя Apple посвятили громадную часть перзентации UITraitCollection, но Америку они не открыли.

Для простоты понимания, что это и, главное,- зачем они это сделали, можно порыться немного в недалеком прошлом:

- Сначала был iPhone. Один. Единственное, что могло вызвать какие-либо изменения в дизайне - это была смена ориентации. Это проверка №1.
- Потом появилась Retina. В некоторых местах понадобились проверки на displayScale.
- Apple выпускает iPad. Ко всем прочим проверкам добавляется InterfaceIdiom.
- Выходит iPhone5. Начинаем проверять на высоту == 568.
- Мир увидели iPhone6 и iPhone 6+ c совершенно разными диагоналями экрана и Retina HD. Разработчики под iOS лишились одного из самых веских аргументов против Android, чем последние не применули воспользоваться. Нужно больше проверок?

И вот Ябл, видимо, понял, что врываться на рынок больших смартфонов нужно аккуратно, и дабы защитить богобоязненых девелоперов, а так же упростить жизнь им и себе, взял да и упаковал все перечисленные выше проверки в один простой контейнер под названием *UITraitCollection*. Что внутри:
{% highlight Obj-c %}
@property (nonatomic, readonly) UIUserInterfaceIdiom userInterfaceIdiom; // наша проверка на тип девайса
@property (nonatomic, readonly) CGFloat displayScale; // 2.f - на ретине, 3.f - на ретине HD
@property (nonatomic, readonly) UIUserInterfaceSizeClass horizontalSizeClass;
@property (nonatomic, readonly) UIUserInterfaceSizeClass verticalSizeClass; // Новое интересное
{% endhighlight %}

Каждый viewController и UIWindow имеет TraitCollection, который "передается" от parent к child по дереву.
Здесь Apple проделали громадную работу, переписав (или просто обнародовав эту инфу) поведение большинства UI элементов в зависимости от тех или иных параметров TraitCollection. Ах да, и добавив возможность на ходу переопределять их.
Здесь и кроются первые неудобства (имхо):

####Проблемы

1) Переопределять TraitCollection можно только у childViewController (с помощью метода  *setOverrideTraitCollection:forChildViewController*). У текущего контроллера? Нет, нельзя. Хотите переопределить - извольте делать "подложку" из еще одного viewController, класть на него нужный, делать его childViewController и вызывать нужный метод (см. "View Controller advancements in iOS 8" сессию).

2) Создать UITraitCollection можно только с одним предопределенным параметром. Все. После создания все остальные property - readonly. Хотите переопределить параметр у текущего TraitCollection и передать его дальше? Создавайте еще один с нужным параметром и суммируйте оригинал с ним (traitCollectionWithTraitsFromCollections:). Хотите переопределить 3 параметра? Создавайте 3 новых коллекции и суммируйте. Параметры перезаписываются при суммировании слева направо.

3) Если мы переопределили UITraitCollection у какого-то контроллера, то его детям поступит уже этот измененный traitCollection. Возможно иногда это полезно, но в моем случае требовалось переопределить колекцию только у SplitViewController, а дальше его detailView должен был отрабатывать оригинальный UITraitCollection. Пришлось делать дополнительное переопределение обратно.

###SizeClass
Когда я смотрел первый раз WWDC - мозг честно плавился и отказывался перестраиваться под новую систему. Одной из основных нововведений и пиаров Apple - были "Adaptive" applications. Простыми словами: один код и storyboard должен работать для всех девайсов и ориентаций правильно с минимумом проверок. Ябл решили разделить все девайсы и ориенации по табличке:
![](http://www.jessesquires.com/img/size_classes.png)

Сначала для запоминания я заменял Compact и Regular другими понятиями вроде: "Малый-большой", "Скукоженый-оригинальный", но это не совсем так. Основная логика, что Regular > Compact. И, кажется, разработчикам Apple просто надо было дать какие-то названия колонкам и строкам.
Также они упразнили почти все методы обработки поворотов девайсов, заменив их на *viewWillTransitionToSize:withTransitionCoordinator:* и *traitCollectionDidChange*.
Итак, что нас может поджидать здесь:

####Проблемы
1) iPad всегда имеет (Regular:Regular) размеры. Смекаете? *traitCollectionDidChange* НЕ будет вызываться. Видимо, решили, что разные ориентации iPadов должны не так уж и отличаться. **Выход:** Подписываться на *UIApplicationDidChangeStatusBarOrientationNotification* или *UIDeviceOrientationDidChangeNotification*. А дальше узнавать у notification положение девайса. Или пользоваться *viewWillTransitionToSize:withTransitionCoordinator:*.

2) iPhone6+. Головная боль. Теоретически, судя по interface builder'у ориентация (regular:compact) предназначена для iPhone6+ в режиме landscape. Практически - дулю вам с маком. В **симуляторе** все равно приходит traitCollection с (compact,compact).
**Выход:** Делать проверку на displayScale и при повороте и, если надо, делать нужный sizeClass.
**Update**: Проект делался в XCode 6. В XCode 6.1 этого бага уже нет.

##UISplitView
У нас в проекте стояла задача сделать выезжающее боковое меню. Недолго думая, я решил попробовать на практике новый UISplitViewController с переопределенны UITraitCollection. Фишка в том, что поведение UISplitViewController задано Apple так, что при horizontal class sizee == Compact, masterView недоступен и все происходит в navigationController. Мне же нужна была боковая панель всегда. Следуя WWDC сессии "Building adaptive apps with UIKit", я сделал дополнительную "подложку" - viewController, на котором лежал containerView и указывал на нужный SplitViewController, делая его childViewController.
![](/images/skitch1.png)
 После чего переопределил UITraitCollection:
{% highlight Obj-C %}
- (void)performTraitCollection {
    UITraitCollection *forcedTraitCollection = [UITraitCollection traitCollectionWithHorizontalSizeClass:UIUserInterfaceSizeClassRegular];
    [self setOverrideTraitCollection:forcedTraitCollection forChildViewController:self.childSplitViewController];
}
{% endhighlight %}

Казалось, все круто! Но SplitViewController в развернутом режиме по умолчанию ставит левой части (W:Compact, H:Regular), а правой части (W:Regular, H:Regular). Поэтому контроллеры, которые лежали в дереве правой части отрабатывали "по-айпадовски" Пришлось переопределить обратно на оригинальный TraitCollection:

{% highlight Obj-C %}
-(void)traitCollectionDidChange:(UITraitCollection *)previousTraitCollection {
    [super traitCollectionDidChange:previousTraitCollection];
    [self.childSplitViewController overrideDetailScreenTraitCollection:self.traitCollection];
}
{% endhighlight %}

А в SplitViewController:

{% highlight Obj-C %}
-(void)overrideDetailScreenTraitCollection:(UITraitCollection *)traitCollection {
    [self setOverrideTraitCollection:traitCollection forChildViewController:self.detailViewController];
}
{% endhighlight %}

Опять все должно быть хорошо, НО! При появлении меню UISplitViewController по умолчанию сжимает detailView.
![](/images/skitch2.png)
![](/images/skitch3.png)

Мне же надо было, чтобы парвая часть "уезжала", не деформируясь. Пришлось делать подобную подложку с containerView, как и описывалось раньше, но с другой целью: containerView я привязал с помощью constraints к краям экрана, но к правому краю - с приоритетом 750. После чего добавил constraint на ширину с приоритетом 1000.
![](/images/skitch4.png)

Этот constraint привязан outlet'ом и вычисляется
{% highlight Obj-C %}
- (void)calculateMinimumWidthConstraint {
    UIWindow *window = [[UIApplication sharedApplication].windows firstObject];
    self.minimumWidthConstraint.constant = window.frame.size.width;
}
{% endhighlight %}

Получилось как раз то, что нужно:
![](/images/skitch5.png)
![](/images/skitch6.png)

Для переключения контроллеров из меню пришлось набросать cutom Segue, вроде такого:

{% highlight Obj-C %}
@implementation CRCChangeDetailViewSegue
-(void)perform {
    UIViewController *svc = (UIViewController *)self.sourceViewController;
    CRCSplitViewController *splitVC = (CRCSplitViewController *)svc.splitViewController;

    UIViewController *dvc = (UIViewController *)self.destinationViewController;
    [splitVC changeDetailViewControllerTo:dvc withOverlayMenuMode:[CRCMenuSwitcher modeForViewController:dvc]];
}
@end
{% endhighlight %}

И просто соединить в Storyboard:
![](/images/skitch7.png)

Так что если ваш заказчик тоже хочет разные дизайны для iPhone6, iPhone6+  и iPad, то сделать это сейчас гораздо легче, но нужно это делать аккуратно, и не верить на слово симулятору.
