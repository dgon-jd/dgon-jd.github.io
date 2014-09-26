---
layout: post
published: true
title: Проверки на Jailbreak девайса.
mathjax: false
featured: false
comments: false
category: iosdevelopment
---

## Why?
Большинство reverse engineering действий проводятся на jailbroken девайсах. Что это такое, можно почитать [здесь](http://uk.wikipedia.org/wiki/Jailbreak). Если вы дорожите своим кодом/идеей/ресурсами, то один из вариантов пересечь разборку вашего приложения на мелкие детали - вставить проверку на Jailbreak.

## How?
Прежде всего стоит попробовать создать файл в какой-нибудь директории, доступа в которую у нас быть не должно:
{% highlight objc %}
	NSError *error;
	NSString *jailTest = @"Jailbreak time!";
	[jailTest writeTofile:@"/private/test_jail" atomically:YES encoding:NSUTF8StringEncoding error:&error];
	if (error == nil) {
		// Something wrong!
	}
{% endhighlight %}

Можно проверить, получится ли у shell'a создать дочерний процесс. Без джеилбрейка должны получить ошибку

{% highlight objc %}
int result = fork();
if (!result) exit(0);
if (result >= 0) return isJail;
return noJail;
if (system(0))  {
	//Something wrong!
}
{% endhighlight %}

Проверим на наличие Cydia. Почти на всех jailbroken девайсах стоит это чудо:

{% highlight objc %}
NSURL *cydiaFakeURL = [NSURL URLWithString: @"cydia://package/com.fake.package"];
if ([[UIApplication sharedApplication] canOpenURL:cydiaFakeURL]) {
	return isJail;
} else {
	return noJail;
{% endhighlight %}

Еще одна дополнительная проверка - наличие самых известных и нужных для реверса приложений:
{% highlight objc %}
NSArray *jailbrokenPaths = @[@"/Applications/Cydia.app",
@"/Applications/RockApp.app",
@"/Applications/Icy.app",
@"/usr/sbin/sshd",
@"/usr/bin/sshd",
@"/private/var/lib/apt",
@"/private/var/lib/cydia",
@"/usr/libexec/sftp-server”,
@"/private/var/stash"];

for (NSString *string in jailbrokenPaths) {
	if ([[NSFileManager defaultManager] fileExistsAtPath:string]) {
		...
	}
}
{% endhighlight %}

Еще один извращенный способ - это проверить наличие процесса MobileCydia в теущих процессах девайса. Как приложение может получить список всех запущенных процессов хорошо описано вот [здесь](http://stackoverflow.com/questions/4312613/can-we-retrieve-the-applications-currently-running-in-iphone-and-ipad).

## Then?

После чего, зная, что девайс хакера взломан, есть рут доступ и больше нет sandbox режима для приложения - при должном энтузиазме, можете сами делать с ним все, что хотите, как и с информацией, которую можно достать.
