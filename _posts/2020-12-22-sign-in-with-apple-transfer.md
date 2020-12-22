# 苹果登录用户转移

## 0.背景说明

* 由于不同公司主体， 苹果登录生成的唯一标识不一样， 所以做应用迁移前， 需要做用户关系的映射。

账号A  =》 应用1   迁移到  账号B =》 应用2

* 相同主体生成的苹果登录唯一标识相同， 所以 ， 我们可以在2个账号下各创建一个应用，这些命名为应用3和应用4

账号A  =》 应用1 ， 应用3

账号B =》 应用2， 应用4 （这些的应用2  实际是等待迁移过来后才有）

* 应用转移后， 苹果登录用户的唯一标识会立即改变，一定要在转移前做好关系映射，最好衔接工作


## 1. 准备工作

### 1.1 生成账号A应用1的client_secret1

由于应用迁移后，应用1的client_secret会立即失效，这里建议使用测试的应用3生成client_secret,需要准备的参数

*  应用3的苹果登录的p8格式密钥，另存为key.txt，保存在client-secret.rb同级目录下，创建后保存好，不能重复下载  [这里创建](https://developer.apple.com/account/resources/authkeys/list)
* 账号A的Team ID  [查看Team ID](https://developer.apple.com/account/#/membership)
* 应用3的bundle_id, 也是下面的client_id或者叫Services ID
* p8格式密钥对应的key id, 一般下载的p8格式密钥， 名称的后面部分就是key id ,或者[点击](https://developer.apple.com/account/resources/authkeys/review/)这里对应的应用查看

可以使用以下ruby脚本client-secret.rb命令生成client_secret

```
require 'jwt'

# Save your private key from Apple in a file called `key.txt`
key_file = 'key.txt'

# Your 10-character Team ID
team_id = 'your team_id'

# Your Services ID, e.g. com.aaronparecki.services
client_id = 'your app bundle_id'

# Find the 10-char Key ID value from the portal
key_id = 'key_id'

ecdsa_key = OpenSSL::PKey::EC.new IO.read key_file

headers = {
  'kid' => key_id
}

claims = {
  'iss' => team_id,
  'iat' => Time.now.to_i,
  'exp' => Time.now.to_i + 86400*180,   # This will be valid for 180 days
  'aud' => 'https://appleid.apple.com',
  'sub' => client_id,
}

token = JWT.encode claims, ecdsa_key, 'ES256', headers

puts token

```

### 1.2 生成账号B应用4的client_secret2

参考1.1



## 2. 获取client_secret1对应的access_token1

```
func getToken(cid string, csecret string) (access_token string){
	url := "https://appleid.apple.com/auth/token"
	body := fmt.Sprintf("grant_type=client_credentials&scope=user.migration&client_id=%s&client_secret=%s",cid, csecret)

	headers := map[string]string{}
	headers["Content-Type"] = "application/x-www-form-urlencoded"

	res,_ := http.Post(url, body, headers)
	fmt.Println(res)

	access_token = jsoniter.Get([]byte(res), "access_token").ToString()
	return
}
```

# 3. 使用access_token1获取transfer_sub

* uid 为账号A需要转移的openid
* recipient_team_id  为账号B的team_id
* token是上面步骤2获取的access_token1
* client_id和client_secret是上面1.1对应生成的

```
func getTransferSub(token string, uid string) string{
	url := "https://appleid.apple.com/auth/usermigrationinfo"
	body := fmt.Sprintf("sub=%s&target=%s&client_id=%s&client_secret=%s", uid, recipient_team_id, client_id, client_secret)
	headers := map[string]string{}

	headers["Content-Type"] = "application/x-www-form-urlencoded"
	headers["Authorization"] = "Bearer " + token

	res,_ := http.Post(url, body, headers)
	fmt.Println(uid + ":" + res)

	return jsoniter.Get([]byte(res), "transfer_sub").ToString()
}
```

## 4. 获取client_secret2对应的access_token2

参考 步骤2

## 5. 使用access_token2获取sub

* transfer_sub  为步骤3生成的
* rec_client_id和rec_client_secret是上面1.2对应生成的
* token是上面步骤4获取的access_token2

```
func exchange(token string, transfer_sub string) (sub string, email string){
	url := "https://appleid.apple.com/auth/usermigrationinfo"
	body := fmt.Sprintf("transfer_sub=%s&client_id=%s&client_secret=%s", transfer_sub, rec_client_id, rec_client_secret)
	headers := map[string]string{}

	headers["Content-Type"] = "application/x-www-form-urlencoded"
	headers["Authorization"] = "Bearer " + token

	res,_ := http.Post(url, body, headers)
	fmt.Println(transfer_sub + ":" + res)

	sub = jsoniter.Get([]byte(res), "sub").ToString()
	email = jsoniter.Get([]byte(res), "email").ToString()

	return
}
```

