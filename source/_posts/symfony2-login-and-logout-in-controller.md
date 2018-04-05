---
title: Symfony2在Controller中实现登陆以及登出
date: 2015-10-12 00:00:00
tags:
	- symfony2
	- php
---
首先自己实现一个用户实体（实现 Symfony\Component\Security\Core\User\UserInterface 中的方法)，

```php
// ...
use Symfony\Component\Security\Core\User\UserInterface;

class UserEntity implements UserInterface
{
    // ...
}
```

在 security.yml 定义Encoder（密码加密用）和Providers（验证令牌提供类），

```yaml
security:
    ...
    encoders:
        Demo\DemoBundle\Entity\User: sha512
    # ...

    providers:
        demo_login:
            entity:
                class: Demo\DemoBundle\Entity\User
                property: account # 用户名property
    # ...
```

登陆代码，

```php
use Symfony\Component\Security\Core\Authentication\Token\UsernamePasswordToken;
use Symfony\Component\Security\Core\AuthenticationEvents;
use Symfony\Component\Security\Core\Event\AuthenticationEvent;
// ...

// $user是需要登陆的用户实体
// 产生令牌
$token = new UsernamePasswordToken($user, null, 'demo_login', $user->getRoles());
// 设置令牌
$this->get('security.token_storage')->setToken($token);
// 广播验证通过事件
$this->get('event_dispatcher')->dispatch(
    AuthenticationEvents::AUTHENTICATION_SUCCESS,
    new AuthenticationEvent($token)
);
```

登出代码，

```php
$this->get('security.token_storage')->setToken(null);
$request->getSession()->invalidate();
```