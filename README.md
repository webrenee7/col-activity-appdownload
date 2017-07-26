> 以下下载功能实现需要服务端和客户端配合完成

## 需要从后台代入的值
```html
<input type="hidden" id="customerId" value="${customerId}" />
<input type="hidden" id="token" value="${token}" />
<input type="hidden" id="channel" value="${channel}" />
```

## js代码
```javascript
var UA = window.navigator.userAgent;
//终端
var os = {
    'android': UA.match(/(Android);?[\s\/]+([\d.]+)?/),
    'iphone': UA.match(/(iPhone\sOS)\s([\d_]+)/)
};
//客户端
var client = {
    'weixin': UA.match(/.*?(MicroMessenger\/([0-9.]+))\s*/)
};

var customerId = $('#customerId').val();
var token = $('#token').val();	
var channel = $('#channel').val();	

var bridge;
document.addEventListener('WebViewJavascriptBridgeReady', onBridgeReady, false);
function onBridgeReady(event) {
    bridge = event.bridge;
    bridge.init(function(message, responseCallback) {
        
    });
}

$(".downloadBtn").click(function(){
    if(os.iphone){//ios
        if(channel=="iOS"){//app
            if(this.id=="lookBtn"){//立即查看
                if(customerId==""||token==null){//未登录
                    //跳转到登录页
                    var data = {"data": "gologin"};
                    bridge.send(data, function (responseData) {

                    });
                }else{//已登录
                    //跳转到我的优惠券页面
                    var data = {"data":"goCoupon","refresh": "yes"};
                    bridge.send(data, function(responseData){
                        
                    });  
                }
            }		
            if(this.id=="bidBtn"){//立即投资
                //跳转到首页
                var data = {"goVC":"TimeBoundBidVC","refresh":"yes"};
                bridge.send(data, function(responseData){
                    
                });  
            }
        }else{//h5
            if(client.weixin){
                window.location.href  = "${rc.getContextPath()}/wxDownload.html";	
            }else{
                var ifr = document.createElement('iframe');
                ifr.src = 'kkdApp://';
                ifr.style.display = 'none';
                document.body.appendChild(ifr);
                window.setTimeout(function(){  
                    window.location.href="itms-apps://itunes.apple.com/app/id938522778";
                },2000); 
            } 
        }
    }else if(os.android){//android
        if(channel=="Android"){//app
            if(this.id=="lookBtn"){//立即查看
                if(customerId==""||token==null){//未登录
                    //跳转到登录页
                    window.KKD_Control.jumpto("com.kkd.kkdapp", "com.kkd.kkdapp.activity.LoginActivity");
                }else{//已登录
                    //跳转到我的优惠券页面
                    window.KKD_Control.jumpto('com.kkd.kkdapp','com.kkd.kkdapp.activity.MyDiscountCouponActivity');
                }
            }
            if(this.id=="bidBtn"){//立即投资
                //跳转到首页
                KKD_Control.jumptoHome(1); 
            }
        }else{//h5
            if(client.weixin){
                window.location.href  = "${rc.getContextPath()}/wxDownload.html?fromUrl=activity";	
            }else{
                var ifr = document.createElement('iframe');
                ifr.src = 'kkdapp://splash';
                ifr.style.display = 'none';
                document.body.appendChild(ifr);
                window.setTimeout(function(){  
                    document.body.removeChild(ifr);
                    //window.location.href="http://image.kuaikuaidai.com/apk/android/kuaikuaidai.apk";
                    window.location.href="http://image.kuaikuaidai.com/apk/android/kuaikuaidai_wltg1.apk"; //需要统计app下载量
                },2000); 
            }
        }
    }
})
```

## Tips
1、微信不能直接下载或打开app
可以通过以下两种方式进行下载或打开app
- 提示用户到浏览器中打开或者下载
- 通过腾讯应用宝传入要下载的app
快快贷理财：http://a.app.qq.com/o/simple.jsp?pkgname=com.kkd.kkdapp

2、js不能判断用户是否下载过app，但是可以用打开app的协议，尝试打开（用iframe），如果在2秒内打不开，就认为没有下载，然后去下载app

```javascript
var ifr = document.createElement('iframe');
ifr.src = 'kkdapp://splash';
ifr.style.display = 'none';
document.body.appendChild(ifr);
window.setTimeout(function(){  
    document.body.removeChild(ifr);
    window.location.href="http://image.kuaikuaidai.com/apk/android/kuaikuaidai_wltg1.apk";
```

3、ios下需要在链接后面添加参数isPost=true才能得到customId和token的值
