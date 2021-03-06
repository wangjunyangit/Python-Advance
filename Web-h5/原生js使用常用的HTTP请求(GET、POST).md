####　原生js实现常用的HTTP请求(GET、POST)

```javascript

// 常用工具函数
var tools = {


	/* ajax请求get
     * @param url     string   请求的路径
     * @param query   object   请求的参数query
     * @param succCb  function 请求成功之后的回调
     * @param failCb  function 请求失败的回调
     * @param isJson  boolean  true： 解析json  false：文本请求  默认值true
     */
    ajaxGet: function (url, query, succCb, failCb, isJson) {
        // 拼接url加query
        if (query) {
            var parms = tools.formatParams(query);
            url += '?' + parms;
            // console.log('-------------',url);
        }

        // 1、创建对象
        var ajax = new XMLHttpRequest();
        // 2、建立连接
        // true:请求为异步  false:同步
        ajax.open("GET", url, true);
        // ajax.setRequestHeader("Origin",STATIC_PATH); 

        // ajax.setRequestHeader("Access-Control-Allow-Origin","*");   
        // // 响应类型    
        // ajax.setRequestHeader('Access-Control-Allow-Methods', '*');    
        // // 响应头设置    
        // ajax.setRequestHeader('Access-Control-Allow-Headers', 'x-requested-with,content-type');  
        // ajax.withCredentials = true;
        // 3、发送请求
        ajax.send(null);

        // 4、监听状态的改变
        ajax.onreadystatechange = function () {
            if (ajax.readyState === 4) {
                if (ajax.status === 200) {
                    // 用户传了回调才执行
                    // isJson默认值为true，要解析json
                    if (isJson === undefined) {
                        isJson = true;
                    }
                    var res = isJson ? JSON.parse(ajax.responseText == "" ? '{}' : ajax.responseText) : ajax.responseText;
                    succCb && succCb(res);
                } else {
                    // 请求失败
                    failCb && failCb();
                }

            }
        }


    },
    
    
    	/* ajax请求post
     * @param url     string   请求的路径
     * @param data   object   请求的参数query  
     * @param succCb  function 请求成功之后的回调
     * @param failCb  function 请求失败的回调
     * @param isJson  boolean  true： 解析json  false：文本请求  默认值true
     */
    ajaxPost: function (url, data, succCb, failCb, isJson) {
    
        var formData = new FormData();
        for (var i in data) {
            formData.append(i, data[i]);
        }
        //得到xhr对象
        var xhr = null;
        if (XMLHttpRequest) {
            xhr = new XMLHttpRequest();
        } else {
            xhr = new ActiveXObject("Microsoft.XMLHTTP");

        }

        xhr.open("post", url, true);
		// 设置请求头  需在open后send前
		// 这里的CSRF需自己取后端获取，下面给出获取代码
        xhr.setRequestHeader("X-CSRFToken", CSRF);

        xhr.send(formData);

        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4) {
                if (xhr.status === 200) {
                    // 判断isJson是否传进来了
                    isJson = isJson === undefined ? true : isJson;
                    succCb && succCb(isJson ? JSON.parse(xhr.responseText) : xhr.responseText);
                }
            }
        }

    },

}



// 调用
// 接口地址
let url = ""
// 传输数据 为object
let data = {}

tools.ajaxGet(url, data, function(res){
	console.log('返回的数据:',res)
	// ....
})

```

#####  `CSRF`获取的代码
- 这里以`Django`为例
```python
from django.template.context_processors import csrf

def get_csrf(request):
    # 生成 csrf 数据，发送给前端
    x = csrf(request)
    csrf_token = x['csrf_token']
    # 这里的csrf_token 是一个对象 需要强转一下
    print('csrf_token:',csrf_token)
    return AccessReturn({'code': 0, 'msg': 'success', 'data': str(csrf_token)})
 

# 允许跨域
def AccessReturn(result_dict):
    response = HttpResponse(json.dumps(result_dict))
    response["Access-Control-Allow-Origin"] = " * "
    response["Access-Control-Allow-Methods"] = "POST, GET, OPTIONS"
    response["Access-Control-Max-Age"] = "1000"
    response["Access-Control-Allow-Headers"] = '*'
    return response
```


##### `CORS` (Corss-Orign Resource Sharing) 跨域

Name | Required | Comments
------ | ------ | ------
Access-Control-Allow-Origin | 必填 | 允许请求的域，比如：localhost或者；`*` 所有都允许 
Access-Control-Allow-Methods | 必填 | 允许请求的方法，比如：get、post、put、delete，多个用逗号分割；`*`允许所有 
Access-Control-Allow-Headers | 可选 | 预检请求后，告知发送请求需要有的头部 
Access-Control-Expose-Headers | 可选 | 自定义字段；CORS请求时，xmlhttprequest默认只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定 
Access-Control-Max-Age | 可选 | 本次预检的有效期，单位：秒；在有效期内不需要发出另一条预检 
Access-Control-Allow-Credentials | 可选 | 表示是否允许发送cookie，默认false；比如put或delete，如果content-type为 application/json等的有特殊要求，需要设置为true

