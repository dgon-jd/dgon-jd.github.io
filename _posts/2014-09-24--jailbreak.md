---
layout: post
published: true
title: Проверки на Jailbreak девайса.
mathjax: false
featured: false
comments: false
categories: 
  - Reverse engineering
tags: "ios, reverse engineering, jailbreak"
headline: Be curious
description: Defend your App.
---

![cover](http://media.idownloadblog.com/wp-content/uploads/2013/01/evasi0n-hero-1024x357.png)

Большинство reverse engineering действий проводятся на jailbroken девайсах. Что это такое, можно почитать [здесь](http://uk.wikipedia.org/wiki/Jailbreak). Если вы дорожите своим кодом/идеей/ресурсами, то один из вариантов пересечь разборку вашего приложения на мелкие детали - вставить проверку на Jailbreak.

Прежде всего стоит попробовать создать файл в какой-нибудь директории, доступа в которую у нас быть не должно:

```objective-c
NSError *error;
NSString *jailTest = @"Jailbreak time!";\
[jailTest writeToFile:@"/private/ test_jail.txt" 
			atomically:YES 
            encoding:NSUTF8StringEncoding error:&error];
if(error==nil) {
	...
}
```

Можно проверить, получится ли у shell'a создать дочерний процесс. Без джеилбрейка должны получить ошибку:
```objective-c
int result = fork();
if (!result) exit(0);
if (result >= 0) return isJail;
	return noJail;
if (system(0))  {
...
}
```

Проверим на наличие Cydia. Почти на всех jailbroken девайсах стоит это чудо:

```objective-c
NSURL *cydiaFakeURL = [NSURL URLWithString: @"cydia://package/com.fake.package"];
if ([[UIApplication sharedApplication] canOpenURL:cydiaFakeURL]) {
	return isJail;
} else {
	return noJail;
}
```

Еще одна дополнительная проверка - наличие самых известных и нужных для реверса приложений:

```objective-c
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
```

Еще один извращенный способ - это проверить наличие процесса MobileCydia в теущих процессах девайса. Как приложение может получить список всех запущенных процессов хорошо описано вот [здесь].(http://stackoverflow.com/questions/4312613/can-we-retrieve-the-applications-currently-running-in-iphone-and-ipad)

После чего, зная, что девайс хкера взломаный и есть рут доступ и больше нет sandbox режима для приложения - при должном энтузиазме, можете сами делать с ним все, что хотите, как и с информацией, которую можно достать.