# Neo域名服务(Neo Name Service)技术白皮书
## 名词定义
### 域名服务
&emsp;&emsp;域名服务旨在将钱包地址、智能合约Hash等人类难以记忆的无规则的字符串用单词短语简写等代替。通过域名服务，人们再也不用记忆看不懂的地址和Hash，只要知道一个单词或一个短语就能进行转账、使用合约。   
&emsp;&emsp;域名命名格式参照大家熟知的DNS规范，用英文句号分割各个组成部分,各个组成部分别从右往左为域、主域名、子域名。
### 域
&emsp;&emsp;域是域名的根，决定了其下所有域名的统一规范，如主域名结构规范、子域名规范等。比如我们会定义.test域的主域名不能超过7个字符，不能设置子域名等。
### 主域名
&emsp;&emsp;主域名从属于域，受到域规则的约束。由任意字符组成，主域名的所有权需要通过公开竞标获得。拥有主域名的地址或账户称为登记员。
### 子域名
&emsp;&emsp;子域名从属于主域名，由任意字符组成，子域名的所有权继承自主域名，管理权可由登记员任意分配。
### 登记员
&emsp;&emsp;登记员，对主域名和子域名在有效期内拥有所有权。其可以依据所有权对域名进行解析器设置、解析设置、注销操作等。可以出让域名管理权、甚至是出让域名所有权。登记员对即将到期域名的所有权拥有优先获得权。
### 有效期
&emsp;&emsp;每一个主域名与登记员的映射关系都有有效期，每次获得所有权的有效期为365天。有效期到期的域名会在被第一次使用时强制注销，调用者会收到0地址返回。被强制注销的域名可以被即时启动竞标。有效期即将到期的一定时间内，登记员可以优先申请延期并获得新的365天有效期。

---

## 服务构成

---

### 竞标服务
竞标服务以确定谁有权注册某一个主域名为服务目标。服务进程分为四个阶段（以1月1日开标为例）:

![image](gantt.png)

```
gantt
dateFormat MM-DD
section 开标
T+0: 01-01, 0d
section 投标
T+7: 01-01, 7d
section 揭标
T+2: 01-08, 2d
section 中标
T+2: 01-10, 2d
(T+2): 01-12, 2d
(T+2): 01-14, 2d
```

#### 开标
&emsp;&emsp;任意未被注册或已过期且不违反域定义的域名都可被任意标准地址（账户）申请开标。一旦开标即意味着该域名的所有权投标开始。
#### 投标
&emsp;&emsp;投标由开标启动，为期7天，在这段时间内任意标准地址（账户）可以提交一个加密的报价，此过程并不需要为此支付任何保证金。投标人通过发送报价附加其自定义的一组8位任意字符的32位sha256散列值作为报价用以隐藏真实报价，以防止没有必要的恶性竞争。投标人不足1人，竞标自动结束，域名立即进入可开标状态。
#### 揭标
&emsp;&emsp;投标过程结束后进入48小时的揭标过程。在此期间，投标人需要提交报价的明文和加密字符串明文，用以验证揭标人没有更改投标报价。在此过程中，投标人需提交与报价等额的保证金。在此过程中，如果投标人没有揭标，视为放弃竞标。揭标人不足1人，竞标自动结束，域名立即进入可开标状态。
#### 中标
&emsp;&emsp;揭标结束后，进入中标过程。确定出价最高的前三揭标人，确认为第一、第二、第三中标人，每个中标人有48小时中标权。中标权首先赋予第一中标人，第一中标人48小时内未领取域名所有权，则中标权赋予第二中标人，以此类推。如果所有中标人均没有在48小时内领取域名所有权，竞标流标，相关域名立即进入可开标状态。中标人支付报价后，即获得该主域名356天的所有权。中标人行驶中标权，则保证金自动支付竞标报价；中标人放弃中标权，则保证金将自动退回。

---

### 注册服务
#### 域名Hash方法
&emsp;&emsp;所有域名都将通过此方法转换为固定32位长度的字节数组
#### 查询方法
&emsp;&emsp;查询方法支持输入域名，输出域名所有者publickey的hash
#### 主域名注册方法
&emsp;&emsp;在竞标中获胜中标的中标人将自动被注册为主域名的所有权人，即登记员。
#### 子域名注册方法
&emsp;&emsp;域名登记员可以授予任意标准地址（账户）其子域名的管理权。同样，登记员可以转移、注销子域名管理权。
#### 域名解析器设置方法
&emsp;&emsp;域名管理员可以设置域名对应的解析器地址（scripthash），每个主域名或子域名只能有唯一一个解析器。
#### 主域名转让方法
&emsp;&emsp;域名登记员可以自行将自己的域名所有权转让给一个标准地址（账户）。一旦执行，不可撤销。域名所有权转移后，其有效期计算不变（仍以中标时间计算）。
#### 注销方法
&emsp;&emsp;域名登记员可以自行注销主域名，即立即放弃对主域名的所有权，也就代表此域名立即进入可开标状态。一旦执行，不可撤销。主域名注销会连带强制注销与其关联的所有子域名和解析器设置。

---

### 解析服务（规范）
&emsp;&emsp;NEL会提供一些常用的解析器（如地址、合约hash等），任何人也可以自行定义解析器，但是必须符合一定规范以实现兼容。
#### 查询方法
&emsp;&emsp;查询方法支持输入域名，输出解析值（如地址、合约hash等）。
#### 修改方法
&emsp;&emsp;域名管理员可以设置域名的解析值。
#### 删除方法
&emsp;&emsp;域名管理员可以删除域名解析值，没有设置解析的域名将返回0地址。

---

### 交易服务  
&emsp;&emsp;交易服务支持域名登记员发布域名所有权转让邀约，对域名有获取需求的人可以进行竞标，其过程类似于竞标服务。  

---

## 域名价值量化
&emsp;&emsp;NEL会发行一种NEP5代币，暂定名NNSV（NNS价值），以此对域名价值进行量化。所有域名服务涉及的费用均由NNSV支付或消费。测试阶段，NEL会以接受申请的方式发放NNSV。正式阶段，发放方式依据测试情况和反馈确定。

---

## 技术路线图
- 2017.12.XX 正式发布NNS技术白皮书
- xxxx.xx.xx 完成原理测试
- xxxx.xx.xx NNS服务上线测试网，发行测试网NNSV
- xxxx.xx.xx NNS服务上线正式网，发行正式网NNSV
