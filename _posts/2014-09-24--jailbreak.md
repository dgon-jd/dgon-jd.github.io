---
layout: post
published: false
title: Проверки на Jailbreak девайса.
mathjax: false
featured: false
comments: false
---

## 
Большинство reverse engineering действий проводятся на jailbroken девайсах. Что это такое, можно почитать [здесь](http://uk.wikipedia.org/wiki/Jailbreak). Если вы дорожите своим кодом/идеей/ресурсами, то один из вариантов пересечь разборку вашего приложения на мелкие детали - вставить проверку на Jailbreak.

Прежде всего стоит попробовать создать файл в какой-нибудь директории, доступа в которую у нас быть не должно:

```c++
NSError *error;
NSString *jailTest = @"Jailbreak time!";
[jailTest writeTofile:@"/private/test_jail" atomically:YES encoding:NSUTF8StringEncoding error:&error];
if (error == nil) {
	// Something wrong!
}
```

Можно проверить, получится ли у shell'a создать дочерний процесс. Без джеилбрейка должны получить ошибку:
```c++
int result = fork();
if (!result) exit(0);
if (result >= 0) return isJail;
return noJail;
if (system(0))  {
	//Something wrong!
}
```