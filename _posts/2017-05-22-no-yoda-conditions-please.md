---
title: "No YODA conditions, please !"
date: "2017-05-22"
layout: post
---

One of the [Wordpress Coding Standards](https://github.com/WordPress-Coding-Standards/WordPress-Coding-Standards) rules is:

> You must use YODA conditions

What are YODA conditions and why don't I like them very much ?

## YODA conditions

YODA conditions stipulates that, instead of:
```php
if ($role == 'admin') {
  ...
}
```

You should write:
```php
if ('admin' == $role) {
  ...
}
```

What we did in the above code, is permuting `$role` and `'admin'` positions, by putting the constant before the variable. But why ?

## The good part

The point in using YODA conditions is to avoid unexpected behaviour from your code whenever you a make the mistake of writing `if ($role = 'admin')` instead of `if ($role == 'admin')`. Notice here the use of `=` instead of `==`.

What happens here is, instead of comparing `$role` and `'admin'`, `$role` value is set to `'admin'` first, and then only `$role` is the condition.  In other words
```php
if ($role = 'admin') {
  // some logic
}
```

is equivalent to:
```php
$role = 'admin';
if ($role) {
  // some logic
}
```

Now, since `$role` value is a non-empty string, the condition will always be true. For more infos about true/false values in PHP, check the [documentation](https://secure.php.net/manual/en/language.types.boolean.php#language.types.boolean.casting). So, if you ever happen to make this kind of typo, your code will sometimes work in an unexpected way.

By following the YODA conditions style, if you make the mistake of writing `if ('admin' = $role)`  instead of  `if ('admin' == $role)`  , a _syntax error(or exception)_ will be raised so that you can fix the error before runtime and avoid any unexpected behaviour.

## The parts I don't like

### Readability

One of the most important things about code, is its readability.  One thing that YODA conditions achieve well, is hitting your code readability.

Each time I see something like `'admin' == $role`, it takes me a moment to figure out what's going on.

### Unusual typo

I rarely find myself writing `=` instead of `==` when writing conditions. So using YODA conditions to avoid an error I almost never make, is like killing a fly with a sledgehammer.

### Testing handles it

Whether it's unit testing or manual testing, I will likely test the conditions in my code to make sure it behaves the way it should. In our case, I will test both cases when `$role == 'admin'` and `$role != 'admin'`. Thus, using YODA conditions may no be that useful.

## Don't get me wrong

I'm not saying that YODA conditions are not good or not useful at all. You should make your own decision based on the way you (or your team ) writes code.

 

Thanks for reading !
