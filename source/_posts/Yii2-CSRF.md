---
title: Yii2 CSRF
date: 2016-08-17 10:42:36
tags:
---
### 一、CSRF
即Cross-site request forgery跨站请求伪造，是指有人冒充你的身份进行一些恶意操作。
比如你登录了网站A，网站A在你的电脑设置了cookie用以标识身份和状态，然后你又访问了网站B,这时候网站B就可以冒充你的身份在A网站进行操作，因为网站B在请求网站A时，浏览器会自动发送之前设置的cookie信息，让网站A误认为仍然是你在进行操作。
对于csrf的防范，一般都会放在服务器端进行，那么我们来看下Yii2中是如何进行防范的。<!-- more -->
### 二、Yii2 CSRF
首先说明一下，我安装的是Yii2高级模版。
#### csrf token生成
vendor\yiisoft\yii2\web\Request.php
```
public function getCsrfToken($regenerate = false)
{
    if ($this->_csrfToken === null || $regenerate) {
        if ($regenerate || ($token = $this->loadCsrfToken()) === null) {
            $token = $this->generateCsrfToken();
        }
        // the mask doesn't need to be very random
        $chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_-.';
        $mask = substr(str_shuffle(str_repeat($chars, 5)), 0, static::CSRF_MASK_LENGTH);
        // The + sign may be decoded as blank space later, which will fail the validation
        $this->_csrfToken = str_replace('+', '.', base64_encode($mask . $this->xorTokens($token, $mask)));
    }

    return $this->_csrfToken;
}
```
getCsrfToken方法首先会用loadCsrfToken方法尝试加载已存在的token，如果没有则用generateCsrfToken方法再生成一个，并经过后续处理，得到最终的前台请求时携带的csrf token。
```
protected function loadCsrfToken()
{
    if ($this->enableCsrfCookie) {
        return $this->getCookies()->getValue($this->csrfParam);
    } else {
        return Yii::$app->getSession()->get($this->csrfParam);
    }
}
```
loadCsrfToken方法会尝试从cookie或session中加载已经存在的token，enableCsrfCookie默认为true，所以一般会从cookie中获取
```
public function getCookies()
{
    if ($this->_cookies === null) {
        $this->_cookies = new CookieCollection($this->loadCookies(), [
            'readOnly' => true,
        ]);
    }

    return $this->_cookies;
}
```
这里又调用了loadCookies方法
```
protected function loadCookies()
    {
        $cookies = [];
        if ($this->enableCookieValidation) {
            if ($this->cookieValidationKey == '') {
                throw new InvalidConfigException(get_class($this) . '::cookieValidationKey must be configured with a secret key.');
            }
            foreach ($_COOKIE as $name => $value) {
                if (!is_string($value)) {
                    continue;
                }
                $data = Yii::$app->getSecurity()->validateData($value, $this->cookieValidationKey);
                if ($data === false) {
                    continue;
                }
                $data = @unserialize($data);
                if (is_array($data) && isset($data[0], $data[1]) && $data[0] === $name) {
                    $cookies[$name] = new Cookie([
                        'name' => $name,
                        'value' => $data[1],
                        'expire' => null,
                    ]);
                }
            }
        } else {
            foreach ($_COOKIE as $name => $value) {
                $cookies[$name] = new Cookie([
                    'name' => $name,
                    'value' => $value,
                    'expire' => null,
                ]);
            }
        }

        return $cookies;
    }
```
这里就是解析验证$_COOKIE中的数据。
#### cookies设置
vendor\yiisoft\yii2\web\Response.php
```
protected function sendCookies()
{
    if ($this->_cookies === null) {
        return;
    }
    $request = Yii::$app->getRequest();
    if ($request->enableCookieValidation) {
        if ($request->cookieValidationKey == '') {
            throw new InvalidConfigException(get_class($request) . '::cookieValidationKey must be configured with a secret key.');
        }
        $validationKey = $request->cookieValidationKey;
    }
    foreach ($this->getCookies() as $cookie) {
        $value = $cookie->value;
        if ($cookie->expire != 1  && isset($validationKey)) {
            $value = Yii::$app->getSecurity()->hashData(serialize([$cookie->name, $value]), $validationKey);
        }
        setcookie($cookie->name, $value, $cookie->expire, $cookie->path, $cookie->domain, $cookie->secure, $cookie->httpOnly);
    }
}
```
sendCookies方法利用cookieValidationKey对cookie进行一系列处理，主要是为了获取的时候进行验证，防止cookie被篡改。
```
public function getCookies()
{
    if ($this->_cookies === null) {
        $this->_cookies = new CookieCollection;
    }
    return $this->_cookies;
}
```
这里的getCookies方法跟request中的不同，并不会从$_COOKIE中获取，_cookies属性在request中的generateCsrfToken方法中有进行设置
```
protected function generateCsrfToken()
{
    $token = Yii::$app->getSecurity()->generateRandomString();
    if ($this->enableCsrfCookie) {
        $cookie = $this->createCsrfCookie($token);
        Yii::$app->getResponse()->getCookies()->add($cookie);
    } else {
        Yii::$app->getSession()->set($this->csrfParam, $token);
    }
    return $token;
}
```
#### csrf验证
vendor\yiisoft\yii2\web\Request.php
```
public function validateCsrfToken($token = null)
{
    $method = $this->getMethod();
    // only validate CSRF token on non-"safe" methods http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.1.1
    if (!$this->enableCsrfValidation || in_array($method, ['GET', 'HEAD', 'OPTIONS'], true)) {
        return true;
    }

    $trueToken = $this->loadCsrfToken();

    if ($token !== null) {
        return $this->validateCsrfTokenInternal($token, $trueToken);
    } else {
        return $this->validateCsrfTokenInternal($this->getBodyParam($this->csrfParam), $trueToken)
            || $this->validateCsrfTokenInternal($this->getCsrfTokenFromHeader(), $trueToken);
    }
}
```
这里先验证一下请求方式，接着获取cookie中的token，然后用validateCsrfTokenInternal方法进行对比
```
private function validateCsrfTokenInternal($token, $trueToken)
{
    if (!is_string($token)) {
        return false;
    }

    $token = base64_decode(str_replace('.', '+', $token));
    $n = StringHelper::byteLength($token);
    if ($n <= static::CSRF_MASK_LENGTH) {
        return false;
    }
    $mask = StringHelper::byteSubstr($token, 0, static::CSRF_MASK_LENGTH);
    $token = StringHelper::byteSubstr($token, static::CSRF_MASK_LENGTH, $n - static::CSRF_MASK_LENGTH);
    $token = $this->xorTokens($mask, $token);

    return $token === $trueToken;
}
```
解析请求携带的csrf token 进行对比并返回结果。
### 三、总结
Yii2的做法就是先生成一个随机token，存入cookie中，同时在请求中携带随机生成的csrf token，也是基于之前的随机token而生成的，验证的时候对cookie和csrf token进行解析，得到随机token进行对比，从而判断请求是否合法。
最后，本文只是对大概的流程进行了分析，具体的细节还请查看源码。