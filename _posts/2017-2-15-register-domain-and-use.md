---
layout:     post   
title:      "记我脱离GoDaddy-教大家注册自己的域名并使用"     
date:       2017-2-18 14:55:00   
author:     "Wheat7"        
header-img: "img/post-bg-01.jpg"
--- 


## 流水账
小站的域名是我16年初在GoDaddy注册的，当时没有选择在国内的域名提供商（如阿里云、新网）注册主要是听说国内的话需要录入的信息比较多，并且不好转移，并且当时国内域名商没有价格优势，狗爹第一年优惠较多，用了优惠码小站的域名才40多rmb，所以就选择了在狗爹注册，现在一年了最近快要过期，在狗爹续费需要100块，觉得太贵，并且自己手上还有很多tk的域名（下边教大家怎么注册），不舍得续费，看了一下阿里云的定价，已经下调很多，转过去续一年只要45，后边续60一年，所以准备转域名到阿里云，奈何最后一步阿里云需要提交身份证扫描件验证，需要好几天，没有弄成，然后在狗爹转移的时候，狗爹跳出来给我优惠20%，于是80块在狗爹续了（后来亏了），然后这几天再研究了一下其他的域名提供商，发现了[namesilo](namesilo.com)，一直以来都是8.99美元没有变过价，并且使用优惠码还可以-1美元，所以洒脱的抛弃万恶的狗爹，投入了namesilo的怀抱

## 科普一下域名管理
不作详细科普，就简单说一下，域名总的是[ICANN](https://www.icann.org/)这个机构管的，定价也是ICANN定的，然后分发到各个服务商，国外有namesilo、name、godaddy等，国内有万网（被阿里云收购，也就是现在的阿里云），新网等，然后提供给用户，用户可以选择在各个ICANN认证的服务商注册、使用域名服务，在各个服务商之间转移（听说国内服务商转移比较麻烦），当然，各个服务商的服务价格都是不同的

## 注册免费的.tk域名
.tk的域名是南太平洋一个仅有1,268人的岛国托克劳（历史上亦称联合群岛或托克劳群岛）的国家顶级域名,任何人都可以免费注册（某些服务商还拿来卖钱 - -），可以到[freenom](https://freenom.com)免费注册
### 注册步骤
下边来给大家说一下注册的步骤                
- 登陆网站，首先是注册账号这个就不多说了     
- 点击Service 选择Register New Domian 如图 
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-17%20at%2017.00.40.png)    
- 然后搜索自己想要的域名，点击Get it now，然后点击Checkout，如图 
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-17%20at%2017.00.54.png)
- 选择时间为12个月，然后注册
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-17%20at%2017.01.10.png)         
- 这一步比较重要，输入自己的注册信息，这些注册信息是公开的，其他人可以根据域名查询到，一些域名服务商提供了隐私保护服务，但是大多需要付费，而namesilo则是免费的,点击Complete Order，注册成功
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-17%20at%2017.02.12.png)

## 注册.com .net .me等付费域名
### 国内还是国外
国外的服务商的注册过程和.tk大同小异，国内的话需要填写中文的相关信息，还要上传身份证的扫面件，那到底是在国内注册好呢还是在国外注册好？传说的话国内域名转出转让都不太好转，等待时间较长，并且如果你的网站有不符合国内规范的地方（你懂的），就会被服务商停用，国外的话就比较自由，但是在国内注册也有好处，可以很方便的备案你的网站，并且使用一些免费的增值服务，比如CDN加速，云解析等，究竟是在国内还是国外注册，我个人是在价格都差不多的情况下，倾向于在国外注册
### 选择服务商
一般第一年服务商都会给出一个很优惠的价格，第二年就会大幅加价，比如小站的域名，
我这里推荐第一年哪便宜哪注册，然后第二年转到续费相对便宜的服务商，我前边推荐的[namesilo](namesilo.com)价格就很稳定，一直都是8.99美元，并且支持支付宝付款，GoDaddy比较被熟知的原因也是因为他的营销比较充足，并且支持支付宝，但是现在狗爹续费的价格确实较贵，大家可以自己搜索后选择合适的服务商注册（关键词：域名服务商），推荐服务商[namesilo](namesilo.com)。
### 注册过程
和注册.tk域名类似，不作赘述了

## 如何使用
### 定位到自己的网站服务器
定位到自己的服务器首先要设置DNS，国内的可以使用默认的，国外的服务商默认的DNS服务器在国内使用一般比较慢，推荐大家使用[DNSPod](https://www.dnspod.cn/)    
现将默认的DNS服务器改为DNSPod的服务器，以[namesilo](namesilo.com)为例，如下所示      
- 登陆不作赘述，在主页点击Manage My Domains,然后点击相应的域名
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.05.22.png) 
- 在NameService处点击Change
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.05.50.png)
- 将解析服务器改为下图所示服务器    
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.05.57.png)
然后到[DNSPod](https://www.dnspod.cn/)，注册，进入控制台，这些就不赘述了，然后进行解析设置      
- 点击添加域名，填写自己的域名确定
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.19.06.png)
- 要设置www @ . 三条 如图，其他的默认，在记录址处填上自己服务器的域名
@ .的设置与www相同，不作赘述 
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.19.35.png)    
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.19.57.png)

### 跳转到自己的社交媒体（如Github、知乎、微博等）
自己的社交媒体使用自己的域名，是不是感觉更高大上了呢    
大多数服务商都提供了这个服务，大家可以根据自己的服务商搜索设置方法，下面以[freenom](https://freenom.com)作为示例      
- 登陆后在主页点击Service 然后点击 My Domains，选择要设置的域名点击Manage Domains
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.22.47.png)
- 在Magagement Tools选择 URL Forwarding，输入自己的要跳转的域名，Forward mode默认即可
![](http://ogzwf5uv0.bkt.clouddn.com/Screen%20Shot%202017-02-18%20at%2014.23.08.png)








