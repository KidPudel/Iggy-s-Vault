authentication and user session is done via [[JWT]] and storing it with a binding with a private key 


file that contains the logic is auth_sg.php
# params

- phone
- code: sms code random between 1111 and 9999

# response
- code: sms
- hash: md5 encrypted combined with login, code ```
```php
$hash = strtoupper(md5(strtoupper($login . $smscode . strtoupper(md5(strtoupper($login . $smscode))))));

// or

$this->hash = strtoupper(md5(strtoupper($this->login.$smscode.strtoupper(md5(strtoupper($this->login.$smscode))))));
```
> where login is filtered *cleared* phone
```php
$phone_clean = preg_replace(["/^\+?7/is", "/\D+/is", "/^(\d{10})$/is", "/^8(\d{10})$/is"], ["7", "", "7$1", "7$1"], $phone);

$params['phone'] = str_replace(["-", "+", "(", ")", " "], '',$params['phone']);
			$params['phone'][0] = $params['phone'][0] == '8' && $params['phone'][1] != '8' ? '7' : $params['phone'][0];
			$this->login = $params['login'] ?: $params['phone'];
```

- token: md5 encrypted using user login
```php
$this->token = md5(($this->user->login).time());
```

- salesId
```php
Auth::$auth->user->salesid  = md5(Auth::$auth->hash.Auth::$auth->login.time());
```

# info to save

log info as a global state kind of

```php
$query = "insert into log_device_info ( userid, imei, os_version, app_version, telephone_info, fcmtoken,pushgroupid,versioncode) 
				values (:userid, :imei, :osversion, :appversion, :telephoneinfo, :fcmtoken,:pushgroupid,:versioncode) returning *";
}

$stm = DB::$db->prepare($query);
$res = $stm->execute(array_combine(
			['userid', 'imei', 'osversion', 'appversion', 'telephoneinfo', 'fcmtoken','pushgroupid','versioncode'],
			[Auth::$auth->user->id,Auth::$auth->imei,Auth::$auth->mobileOS, Auth::$auth->appversion,''.$this->source,$this->fcmtoken,$this->source,''.Auth::$auth->version]));
		$result = $stm->fetchObject();
```
*from salestable*




# flow

## overview

```php
$this->initFromCookie();
$this->initFromHeaders();
$this->params = filter_input_array(INPUT_POST,[
'login'=>FILTER_SANITIZE_STRING,
'psw'=>FILTER_SANITIZE_STRING,
'phone'=>FILTER_SANITIZE_STRING,'name'=>FILTER_SANITIZE_STRING],true);
Auth::$auth->responseType = 'json';
// check the user and act accordingly
Auth::$auth->identify();
Auth::$auth->authenticate();
Auth::$auth->authorize();
```


## verify called user (exists, session is relevant etc)
- if we don't have a token (lives 30 days) saved in a user (not logged in),
- then find user by login or hash
- if that fails, register the user


else
```postgresql
SELECT u.*,ut.token,ut.status,
	       (select uo.salesid from salestable uo where ut.userid = uo.userid and uo.oracle_id isnull and uo.status < 2  order by uo.id desc limit 1 ) as salesid,
	       ut.code as smscode,ut.edate > now() as tokenactive,
	       (select phone from phones join phones_links link on phones.id = link.phoneid and link.relationid = u.id and relationtype = 1 where phonetype = 'default' limit 1) as phone,
			u.subscribe
	FROM users u 
	    join user_tokens ut on ut.userid = u.id and ut.token = :token limit 1

```

```php
$stmt->execute(['token'=>$this->token]);
```

and check if the user exists
and set cookie
```php
if (!$this->user)
			{
				setcookie('token','',-1,'/');
				$msg = $this->responseType == 'json' ? json_encode(["status"=>'error','msg'=>'Пользователь не зарегистрирован.','token_in'=>$this->token]) : 'Пользователь не зарегистрирован.';
				die($msg);
			}
			else
			{
				$this->showloginform = false;
				$this->hash = $this->user->password;
				if (php_sapi_name() !== 'cli' && $this->setcookie)
				{
					setcookie('token',$this->token,time() +2592000,'/');
				}
			}
```


## register (create user)

## send sms

```php
public function sendsms()
	{
		$sms_client_init = null;
		//$this->errors[] ='rty sms sended';
		$src = 5;
		if (Auth::$auth->source && in_array(Auth::$auth->source,[22,23,24]))
		{
			$src = 6;
		}
		if ($this->user->phone && !DEBUG)
		{
			$sms_client_init = file_get_contents("https://new-api.supergood.ru/sms_supergood.php?to={$this->user->phone}&source={$src}&message=" . rawurlencode($this->smstext()));
		}

		if (is_null($sms_client_init) or $sms_client_init === false)
		{
			$this->errors[]='error sms sending';
			return false;
		}
		return true;
	}
```




## verify that code is correct
`authenticate($smscode)`