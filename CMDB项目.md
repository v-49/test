接触新的CMDB项目。

CMDB初感觉就是某某系统  只不过做的更加高大上，特点满足及时性 稳定性 自动化 属于大型的某某系统。

同时准确 可操控的监控 配置 服务器管理资源等。



首先进行MongoDB的表文档填写，我在![image-20230831155422851](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230831155422851.png)

此目录下进行表比对。// CloudAccount 云账户
type CloudAccount struct

![image-20230831155827606](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230831155827606.png)

type HostApplyRule struct

![image-20230831160631264](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230831160631264.png)

type ProcInstanceModel struct 

 [procserver.go](D:\1STudy\go\CMDB\bk-cmdb-master\src\common\metadata\procserver.go)  ProcInstanceDetail   





进入mongodb  usetables 查询： db.cc_ObjAttDes.find().pretty().limit(1)

输出：

```
{
        "_id" : ObjectId("60ffee26cba62a75b5f3cdc1"),
        "unit" : "",
        "editable" : true,
        "bk_issystem" : false,
        "bk_isapi" : false,
        "bk_property_id" : "bk_biz_name",
        "ispre" : true,
        "isrequired" : true,
        "isreadonly" : false,
        "creator" : "cc_system",
        "create_time" : ISODate("2021-07-27T11:29:42.082Z"),
        "id" : NumberLong(1),
        "bk_supplier_account" : "0",
        "bk_property_group" : "default",
        "bk_property_index" : NumberLong(0),
        "bk_property_type" : "singlechar",
        "bk_obj_id" : "biz",
        "bk_property_name" : "业务名",
        "placeholder" : "",
        "option" : "",
        "last_time" : ISODate("2021-07-27T11:29:42.082Z"),
        "bk_biz_id" : NumberLong(0),
        "description" : "",
        "isonly" : false
}

```

下面开始进行对项目的深入了解

# 项目源代码阅读

**要求：摸清项目主体结构，设计模式，各个模块分成了如何**

首先先去github看看有哪些带中文说明的。



上来就是一个重量级结构--apiserver    

他包装了很多接口，用这些接口带上一些方法，以此来特化一部分内容。比如![image-20230906145855878](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230906145855878.png)

这个接口就包括了一个清除数据库方法，一个set 一个migrate 一个sync    然后又是一个new

adminserver.go文件是相当于api.go的一个封装。



寻找main.go文件 ，为什么有这么多main.go文件  是说多个模块独立运行吗？那他们又是如何协作的。

![image-20230906184314733](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230906184314733.png)

昨天晚上睡觉前就想到以前写go中接口的方式是不对的，最好还是单独开个文件写入 ，文件名为XX模块或者XX功能，然后在使用的地方直接使用。而不是直接写入到使用的文件里。这样显得使用的文件比较整洁，也好看清这个模块或者功能的结构。

 那说回来这边的代码也是一样，梳理整体代码结构也许也是先大刀阔斧查看。



ac  access control  访问控制 包括身份认证之类 

extension里 base.go 身份认证，查询资源

biz_set.go  业务集合，业务授权，查询

business.go  业务获取，业务授权和认证逻辑

host.go  处理主机信息，生成主机相关资源

instance.go  

model.go









9.11

开发-在云账户-新建账号中 添加账户类型-我们的云服务



src-scene_server-cloud_server

*options.go*

1.ServerOption 结构体定义了服务器选项的参数，包括 ServConf 和 EnableCryptor。


2.ServConf 是一个指向 config.CCAPIConfig 结构体的指针，用于存储服务器的配置信息。
3.EnableCryptor 是一个布尔值，用于指示是否启用加密器。


4.NewServerOption 函数用于创建一个 ServerOption 对象，并初始化其中的 ServConf 字段。
5.AddFlags 方法用于向命令行参数中添加标志（flags）。


6.它使用 pflag.FlagSet 来管理命令行参数。
7.使用 fs.StringVar 和 fs.BoolVar 来定义不同类型的标志，并将其绑定到对应的字段。
8.通过调用 auth.EnableAuthFlag 的 Var 方法添加一个自定义的标志。

9.Config 结构体定义了一些配置参数，包括 SecretKeyUrl、SecretsAddrs、SecretsToken、SecretsProject、SecretsEnv 和 SyncPeriodMinutes。

*server.go*--运行逻辑

像腾讯蓝鲸这个项目，他的数据都是以包下文件的格式存放，比如有metadata包 下面有很多XXXX.GO文件 文件中又含有结构体  结构体中是这些字段

首先在提供商中创建tfcloud，这里初始化时涉及到metadata文件，

这里的云服务涉及API提供的信息，我需要什么，就去找子条件，然后根据返回的数据向上传递 最终获得：

创建云厂商客户端，     获取地域列表    ||     获取VPC列表



在aws，Tencent的cloud.go文件下具体了方法的实现，但是又在vendorclient.go进行接口实例，把二者的方法都放进去 ，然后使用这个vendorClient interface     所谓的实例化 这里作2级或者3级实例化，是整个调用API（调用云服务相关API）的集成。即所谓的云账号就是服务的集合+数据。



9.12

数据存储： 所有和云服务相关的数据字段都是由上cloud。go文件获取 然后返回存储在metadata里的struct中。

业务对应：![image-20230912104036683](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230912104036683.png)

下面看run内容。查看运行逻辑.

内容超出了云模块，结构很深，向上发展，ctx定义云服务内容，实际上又是由instance定义的，instance又由modalAPI带来，转到![image-20230912161603510](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230912161603510.png) 



![image-20230912172030339](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230912172030339.png) ->initRoute()   /service  ->run()  ->cloud_server.go  

  op+ctx   ->  op->NewServerOption()->config.CCAPIConfig()

```
type CCAPIConfig struct {
	AddrPort    string
	RegDiscover string
	RegisterIP  string
	ExConfig    string
	Qps         int64
	Burst       int64
}

// NewCCAPIConfig create ccapi config object
func NewCCAPIConfig() *CCAPIConfig {
	return &CCAPIConfig{
		AddrPort:    "127.0.0.1:8081",
		RegDiscover: "",
		RegisterIP:  "",
		Qps:         1000,
		Burst:       2000,
	}
}
```

在run中，op->NewServerInfo

```
func NewServerInfo(conf *config.CCAPIConfig) (*ServerInfo, error) {
	ip, err := conf.GetAddress()
	if err != nil {
		return nil, err
	}

	port, err := conf.GetPort()
	if err != nil {
		return nil, err
	}

	registerIP := conf.RegisterIP
	// if no registerIP is set, default to be the ip
	if registerIP == "" {
		registerIP = ip
	}

	hostname, err := os.Hostname()
	if err != nil {
		return nil, err
	}

	info := &ServerInfo{
		IP:         ip,
		Port:       port,
		RegisterIP: registerIP,
		HostName:   hostname,
		Scheme:     "http",
		Version:    version.GetVersion(),
		Pid:        os.Getpid(),
		UUID:       xid.New().String(),
	}
	return info, nil
}
```

从而获得svrInfo,  然后要对ctx进行处理来实例化一个service： service := svc.NewService(ctx)

再来看ctx





9.13

```
默认字段：
 const defaultMap = {
    singlechar: { operator: IN, value: [] },
    int: { operator: EQ, value: '' },
    float: { operator: EQ, value: '' },
    enum: { operator: IN, value: [] },
    date: { operator: RANGE, value: [] },
    time: { operator: RANGE, value: [] },
    longchar: { operator: IN, value: [] },
    objuser: { operator: IN, value: [] },
    timezone: { operator: IN, value: [] },
    bool: { operator: EQ, value: '' },
    list: { operator: IN, value: [] },
    organization: { operator: IN, value: [] },
    array: { operator: IN, value: [] },
    map: { operator: IN, value: [] },
    object: { operator: IN, value: [] }
  }相应的：
短字符：长度 256 个英文字符或 85 个中文字符
长字符：长度 2000 英文或 666 个中文字符
数字：正负整数
浮点数：可以包含小数点的数字
枚举：包含 K-V 结构的列表
日期：日期格式
时间：时间格式
时区：国际时区的枚举
用户：可以输入蓝鲸中已经录入的用户
布尔：布尔类型，常用于开关
列表：可以理解为数组类型，只包含值的列表
```

```
one per version:
1.vCPU核数：cpu_cores - int
2.内存大小：memory_size - float
3.磁盘容量：disk_capacity - float
4.IP地址：ip_address - singlechar
5.云区域名称：cloud_region - singlechar
6.私有IP：private_ip - singlechar
7.浮动IP：floating_ip - singlechar
8.宿主机ID1：host_id1 - singlechar
9.宿主机ID2：host_id2 - singlechar
10.名称：name - singlechar
11.操作系统类型：os_type - enum
12.操作系统：os - singlechar
13.系统类型：system_type - enum
14.区域名称：region_name - singlechar
15.状态：status - enum
16.可用分区：available_zone - singlechar
17.所属区域：belonging_region - singlechar
18.资源ID：resource_id - singlechar
19.创建时间：creation_time - date
20.标签：tag - list
21.虚拟机状态：vm_status - enum
22.虚拟化类型：virtualization_type - enum

```

9.14

需要的资源类型：

![image-20230914153236548](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230914153236548.png)

导入华为包：[huaweicloud-sdk-go-v3/services/vpc/v3 at master · huaweicloud/huaweicloud-sdk-go-v3 (github.com)](https://github.com/huaweicloud/huaweicloud-sdk-go-v3/tree/master/services/vpc/v3)

那涉及到代码部分也要以包为主来进行修改，原demo废弃，要重新写

现在重新梳理结构：

metadata下cloud.go：主要实际应用：定义数据模型和进行数据验证

cloudserver下的Tencentcloud.go:实际与腾讯云交互的接口

那么我们的HWcloud.go 应该也是要写在cloudserver下，并且详细定义接口实习。

cloudserver下的vendorclient：提供统一，标准化的方式来管理和访问不同的云提供商



使用sdk explore来查看使用样例：

[API Explorer (huaweicloud.com)](https://console.huaweicloud.com/apiexplorer/#/openapi/ECS/sdk?api=NovaListAvailabilityZones)

[API Explorer - 云 API - 控制台 (tencent.com)](https://console.cloud.tencent.com/api/explorer?Product=cvm&Version=2017-03-12&Action=DescribeRegions)



![image-20230914183817566](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230914183817566.png)

![image-20230914183828251](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230914183828251.png)



主要业务逻辑回到![image-20230914184953547](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230914184953547.png)



最终指向数据声明定义还是在Service里  ![image-20230914185802890](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230914185802890.png)

这几个的结构都很重要，需要看看

现在似乎又比前几天开朗了不少，现在如何写方法都有了个大概 即通过API调用改写   写哪些方法 现在是腾讯或者aws云列表 不 是提供商列表里哪些  然后对应的数据字段 就是下面要搞懂的   。明天先把方法照着写一写  然后查找源字段以进行读写  最后能把所有结构梳理 列出图表  这就属于大成功了  那后来的人也能快速看懂整个云模块内容 也能快速上手。  所谓内容 就是数据字段 与 功能实现





9.15

主要内容还是 ![image-20230915110553363](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230915110553363.png)下run()内的内容。

上级为 scene_server  admin_server  migrate.go 里的main  跟下面的cloud.go一样的内容。



```
package main

import (
"fmt"
"github.com/huaweicloud/huaweicloud-sdk-go-v3/core/auth/basic"
vpc "github.com/huaweicloud/huaweicloud-sdk-go-v3/services/vpc/v2"
"github.com/huaweicloud/huaweicloud-sdk-go-v3/services/vpc/v2/model"
region "github.com/huaweicloud/huaweicloud-sdk-go-v3/services/vpc/v2/region"
)

func main() {
ak := "<YOUR AK>"
sk := "<YOUR SK>"

auth := basic.NewCredentialsBuilder().
    WithAk(ak).
    WithSk(sk).
    Build()

client := vpc.NewVpcClient(
    vpc.VpcClientBuilder().
        WithRegion(region.ValueOf("af-south-1")).
        WithCredential(auth).
        Build())

request := &model.ListVpcsRequest{}
response, err := client.ListVpcs(request)
if err == nil {
    fmt.Printf("%+v\n", response)
} else {
    fmt.Println(err)
}
}这是华为官方查询API的示例  我现在要用我自己的格式 如下：func (c *tcClient) GetVpcs(region string, opt *ccom.VpcOpt) (*metadata.VpcsInfo, error) {
credential := c.newCredential(c.secretID, c.secretKey)
client, err := tcVpc.NewClient(credential, region, profile.NewClientProfile())
if err != nil {
return nil, err
}

if opt == nil {
	opt = ccom.GetDefaultVpcOpt()
}
vpcsInfo := new(metadata.VpcsInfo)
loopCnt := 0
var totalCnt int64 = 0
request := c.newDescribeVpcsRequest(opt)
// 在limit小于全部数据量的情况下，获取limit数量的数据，否则获取全部数据
for {
	resp, err := client.DescribeVpcs(request)
	if err != nil {
		return nil, err
	}
	for _, vpc := range resp.Response.VpcSet {
		vpcsInfo.VpcSet = append(vpcsInfo.VpcSet, &metadata.Vpc{
			VpcId:   *vpc.VpcId,
			VpcName: *vpc.VpcName,
		})
	}
	totalCnt = int64(*resp.Response.TotalCount)
	// 在获取到limit数量或者全部数据的情况下，退出循环
	if opt.Limit == int64(len(vpcsInfo.VpcSet)) || len(vpcsInfo.VpcSet) == int(totalCnt) {
		break
	}
	// 设置分页请求参数
	offset := fmt.Sprintf("%d", len(vpcsInfo.VpcSet))
	request.Offset = &offset
	loopCnt++
	if loopCnt > ccom.MaxLoopCnt {
		blog.Errorf("DescribeVpcs loopCnt:%d, bigger than MaxLoopCnt, TotalCount:%d",
			loopCnt, resp.Response.TotalCount)
		return nil, ccom.ErrorLoopCnt
	}
}
vpcsInfo.Count = totalCnt

return vpcsInfo, nil
}
照着写一个获取华为vpc列表的func：func (c *hwClient) GetVpcs(region string, opt *ccom.VpcOpt) (*metadata.VpcsInfo, error) {  注意在这里面要把对应的tcvpc等参数变成hwvpc
```



"github.com/huaweicloud/huaweicloud-sdk-go-v3@v0.1.58/core/auth/basic"  华为的身份认证部分

已完成vendorclient部分的三个基本api部分，但是华为的API不像腾讯或者亚马逊那么准确 有的API概念也许不一样：// 获取地域列表
// https://console.huaweicloud.com/apiexplorer/#/openapi/APIExplorer/sdk?api=ListRegionsV4  

// 获取(数据库)实例列表
// https://console.huaweicloud.com/apiexplorer/#/openapi/RDS/doc?api=ListInstances

// 获取vpc列表
//https://console.huaweicloud.com/apiexplorer/#/openapi/VPC/sdk?version=v2&api=ListVpcs

下一步是看华为云官方SDK内容，进行API改写，要改写部分 华为云的认证不同 很多细节不同 看上面的basic代码。

首先搞清楚认证流程，所谓的AK  SK  然后通过nasic.newcreedentialsbuilder()创建认证，下面通过认证进行相关的请求。 最后对请求的结果 即api的返回进行处理以获得简洁的信息。

9.18

伪华为cloud 代码块

```
package cloudvendor

import (
	"configcenter/src/common/metadata"
	ccom "configcenter/src/scene_server/cloud_server/common"
	"fmt"
	"github.com/huaweicloud/huaweicloud-sdk-go-v3/core/auth/basic"
	hwCommon "github.com/huaweicloud/huaweicloud-sdk-go-v3/core/auth/basic"
	vpc "github.com/huaweicloud/huaweicloud-sdk-go-v3/services/vpc/v2"
	"github.com/huaweicloud/huaweicloud-sdk-go-v3/services/vpc/v2/model"
	hwregion "github.com/huaweicloud/huaweicloud-sdk-go-v3/services/vpc/v2/region"
	"io/ioutil"
	"net/http"
	"strings"
)

func init() {
	Register(metadata.HuaweiCloud, &hwClient{vendorName: metadata.HuaweiCloud})
}

type hwClient struct {
	vendorName string
	secretID   string
	secretKey  string
}

// 创建华为客户端
func (c *hwClient) NewVendorClient(secretID, secretKey string) VendorClient {
	return &hwClient{
		vendorName: metadata.HuaweiCloud,
		secretID:   secretID,
		secretKey:  secretKey,
	}
}

// GetInstancesTotalCnt 获取实例总个数
func (c *hwClient) GetInstancesTotalCnt(region string, opt *ccom.InstanceOpt) (int64, error) {
	//TODO implement me
	panic("implement me")
}

// 获取地域列表
// https://console.huaweicloud.com/apiexplorer/#/openapi/CAE/sdk?api=ListDomains
// https://console.huaweicloud.com/apiexplorer/#/openapi/APIExplorer/sdk?api=ListRegionsV4  zhege
func (c *hwClient) GetRegions() ([]*metadata.Region, error) {

	url := "https://apiexplorer.ap-southeast-3.myhuaweicloud.com/v4/regions?product_short=producs&api_name=WhatsApp&info_version=whatversion"
	method := "GET"

	payload := strings.NewReader(``)

	client := &http.Client{}
	req, err := http.NewRequest(method, url, payload)

	if err != nil {
		fmt.Println(err)
		return nil, err
	}
	req.Header.Add("Authorization", "<Your signed string>")
	req.Header.Add("X-Language", "zh-cn")

	res, err := client.Do(req)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer res.Body.Close()

	body, err := ioutil.ReadAll(res.Body)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(string(body))
}

// 获取vpc列表
// https://console.huaweicloud.com/apiexplorer/#/openapi/VPC/sdk?version=v2&api=ListVpcs
func (c *hwClient) GetVpcs(region string, opt *ccom.VpcOpt) (*metadata.VpcsInfo, error) {
	ak := "<YOUR AK>"
	sk := "<YOUR SK>"
	projectId := "whatproidwwwwwwwwwwww"

	auth := basic.NewCredentialsBuilder().
		WithAk(ak).
		WithSk(sk).
		WithProjectId(projectId).
		Build()

	client := vpc.NewVpcClient(
		vpc.VpcClientBuilder().
			WithRegion(hwregion.ValueOf("cn-north-1")).
			WithCredential(auth).
			Build())

	request := &model.ListVpcsRequest{}
	response, err := client.ListVpcs(request)
	if err == nil {
		fmt.Printf("%+v\n", response)
	} else {
		fmt.Println(err)
	}
}

// 获取(数据库)实例列表
// https://console.huaweicloud.com/apiexplorer/#/openapi/RDS/doc?api=ListInstances
func (c *hwClient) GetInstances(region string, opt *ccom.InstanceOpt) (*metadata.InstancesInfo, error) {

	return nil, nil

}

func (c *hwClient) newCredential(id string, key string) *hwCommon.Credentials {

	return hwCommon.NewCredentialsBuilder(id, key).Build()
}

func (c *hwClient) newDescribeVpcsRequest(opt *ccom.VpcOpt) interface{} {

}

```

刚刚发现很严重的问题：

上周的开发内容是参考了代码内腾讯云与亚马逊云的文件，他们的获取都是在腾讯云的官方服务下的API，于是创建了一个demo，包括了基础的方法，然后去华为云的官网上搜索的API请求示例，包括了获取地域列表，获取vpc列表，获取(数据库)实例列表三大基本服务内容，本来拟用其SDK里的数据结构与方法实现改写。 但是 我又看了这个managerone的文档 好像不同于华为云官方的文档 ，它好像是一个私有云服务，其中的API不同于官网的API，那么我在写前者华为云的文档时会用到官方的SDK，在这个managerONE里不知道是否还能使用同样的SDK来调用接口？ 两者用的甚至都不是同一个账号

目前的问题是北向managerONE文档的归属，面向API开发 managerONE下有IAM  运营  运维3大块 所需要的是哪一块或者哪些块的API   



DEMO:GET VPC  from managerone 运营API

```
package main

import (
	"fmt"
	"net/http"
	"io/ioutil"
)

func main() {
	X_Auth_Token := "ourToken"
	cloud_infra_id := "fc_001"

	url := fmt.Sprintf("/rest/orchestration/v3.0/fcvpc/vpcs?cloud_infra_id=%s", cloud_infra_id)

	req, _ := http.NewRequest("GET", url, nil)

	req.Header.Add("X-Auth-Token", X_Auth_Token)
	req.Header.Add("Content-Type", "application/json")
	req.Header.Add("Accept", "application/json")

	res, err := http.DefaultClient.Do(req)
	if err != nil {
		fmt.Println("请求失败：", err)
		return
	}

	defer res.Body.Close()
	body, _ := ioutil.ReadAll(res.Body)

	fmt.Println(string(body))
}


```

```
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

type ResInstanceMeterRequest struct {
	TenantID     string `json:"tenant_id"`
	ResourceType string `json:"resource_type"`
	TenantType   string `json:"tenant_type"`
	StartTime    string `json:"start_time"`
	EndTime      string `json:"end_time"`
	Local        string `json:"local"`
	TimeZone     string `json:"time_zone"`
	// 其他字段可以根据需要添加
}

type ResInsStatisticsResponse struct {
	Currency      string                `json:"currency"`
	Total         int32                 `json:"total"`
	ListStatistics []ResInsStatisticsList `json:"listStatistics"`
}

type ResInsStatisticsList struct {
	ID    string `json:"id"`
	State string `json:"state"`
	// 其他字段可以根据需要添加
}

func main() {
	url := "/rest/metering/v3.0/metrics/resource-instance"

	requestData := &ResInstanceMeterRequest{
		TenantID:     "6f7ae39d-3160-4ab8-ad26-1cd289592c7a",
		ResourceType: "hws.resource.type.vrmvpc",
		TenantType:   "upperVDC",
		StartTime:    "2018-08-01 00:00:00",
		EndTime:      "2018-08-21 00:00:00",
		Local:        "zh_CN",
		TimeZone:     "Asia/Shanghai",
	}

	jsonData, _ := json.Marshal(requestData)

	req, _ := http.NewRequest("POST", url, bytes.NewBuffer(jsonData))
	req.Header.Set("Content-Type", "application/json")
	req.Header.Set("Accept", "application/json")
	req.Header.Set("x-auth-token", "Token")

	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()

	body, _ := ioutil.ReadAll(resp.Body)

	var response ResInsStatisticsResponse
	err = json.Unmarshal(body, &response)
	if err != nil {
		panic(err)
	}

	fmt.Println("Currency:", response.Currency)
	fmt.Println("Total:", response.Total)
	for _, item := range response.ListStatistics {
		fmt.Println("ID:", item.ID, "State:", item.State)
	}
}

```





9.19

今天拆解华为云认证流程，把需要的core组件创作出来

AK/SK签名认证操作流程如下：

1. API调用信息收集。

   需要获取以下信息，包括：

   - 用于组成请求URL的Endpoint和URI。

   - 用于签名和认证的AK/SK。

   - 用于区分租户的项目ID、子项目ID。

   - 用于区分租户的帐号名、帐号ID。

     

在运营API文档中： 熟悉认证流程 



l 通过Token认证调用请求，使用Token认证时请注意Token的有效期，超期后使用将认证失败。

l 运营面Token认证的相关接口请参见以下配套文档：

华为云Stack场景：《ManageOne 8.2.0 OTenantSecurity服务API参考》 我参考的是这个 这个又转到下面的<API签名指南>其中网站链接为：https://support.huaweicloud.com/devg-apisign/api-sign-provide.html

HCS Online场景：《统一身份认证 6.1.19 API参考》

步骤 1 每次认证成功后会得到一个Token值。

步骤 2 后续每次请求都需要携带此Token值进行认证。



在manageone的tenantsecurity中，token认证步骤：

当您使用Token认证方式时，请采用如下步骤调用接口：

步骤 1 发送“POST https://IAM域名:443/v3/auth/tokens”，获取OTenantSecurity的Endpoint及消息体中的区域名称。

请向企业管理员获取区域和终端节点信息。

步骤 1 获取Token，请参考“3.9.2 获取用户Token”。



l Token值即接口返回消息Header中的“X-Subject-Token”的值。

l Token的有效期为24小时，需要同一个Token鉴权时，可以先缓存起来，避免频繁调用。

步骤 2 调用接口时，在业务接口请求消息头中增加“X-Auth-Token”，取值为[步骤2](#le72e60d43b544b0a9b376e225c70adee)中的Token值。

```
华为认证部分：AK/SK认证  在《API签名指南》中：
https://support.huaweicloud.com/devg-apisign/api-sign-sdk-go.html

请求示例详解：
https://support.huaweicloud.com/devg-apisign/api-sign-provide-start.html

```

转化为我的代码部分就是 cloud_server 下cloudvendor中的huawei.go中的认证部分，与hwcfg.go(数据配置文件)

下面core文件夹中的signer.go(认证主体文件)与escape.go（主体文件功能部分） 配合使用。

基本逻辑流程为：

```
	s := core.Signer{
		Key:    secretID,
		Secret: secretKey,
	}
	r, err := http.NewRequest("POST", "https://30030113-3657-4fb6-a7ef-90764239b038.apigw.cn-north-1.huaweicloud.com/app1?a=1&b=2",
		ioutil.NopCloser(bytes.NewBuffer([]byte("foo=bar"))))
	if err != nil {
		fmt.Println(err)
		return nil
	}

	r.Header.Add("content-type", "application/json; charset=utf-8")
	r.Header.Add("x-stage", "RELEASE")
	s.Sign(r)
	fmt.Println(r.Header)
	client := http.DefaultClient
	resp, err := client.Do(r)
	if err != nil {
		fmt.Println(err)
	}

	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(string(body))
```

```

func demoAppApigw() {
	s := core.Signer{
		Key:    "apigateway_sdk_demo_key",
		Secret: "apigateway_sdk_demo_secret",
	}
	r, err := http.NewRequest("POST", "https://30030113-3657-4fb6-a7ef-90764239b038.apigw.cn-north-1.huaweicloud.com/app1?a=1&b=2",
		ioutil.NopCloser(bytes.NewBuffer([]byte("foo=bar"))))
	if err != nil {
		fmt.Println(err)
		return
	}
	
	r.Header.Add("content-type", "application/json; charset=utf-8")
	r.Header.Add("x-stage", "RELEASE")
	s.Sign(r)
	fmt.Println(r.Header)
	client := http.DefaultClient
	resp, err := client.Do(r)
	if err != nil {
		fmt.Println(err)
	}
	
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(string(body))
}这是华为的认证demo，我现在要改写成我们正在编写的云服务模块中的内容，我们是微服务架构，像这样
type hwClient struct {
	vendorName string
	secretID   string
	secretKey  string
}

func (c *hwClient) GetVpcs(region string, opt *ccom.VpcOpt) (*metadata.VpcsInfo, error) {
	//TODO implement me
	panic("implement me")
}

func (c *hwClient) GetInstances(region string, opt *ccom.InstanceOpt) (*metadata.InstancesInfo, error) {
	//TODO implement me
	panic("implement me")
}

func (c *hwClient) GetInstancesTotalCnt(region string, opt *ccom.InstanceOpt) (int64, error) {
	//TODO implement me
	panic("implement me")
}
  尝试着把从这个demo中拆分我们所需要的内容，也可以创建数据类型用于存储
```

```
AK/SK签名认证操作流程
https://support.huaweicloud.com/devg-apisign/api-sign-provide-start.html
构造规范请求
https://support.huaweicloud.com/devg-apisign/api-sign-provide-start.html
地区和终端节点
https://developer.huaweicloud.com/endpoint
```



认证部分详解如下：

https://support.huaweicloud.com/devg-apisign/api-sign-sdk-go.html中

![image-20230919185925480](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230919185925480.png)

```
获取AK/SK
GET ENDPOINT
GET PROJECTID

构造规范请求。
将待发送的请求内容按照与API网关后台约定的规则组装，确保客户端签名、API网关后台认证时使用的请求内容一致。


创造待签字符串

把处理后的字段 加到规范请求中发送。

```

这个AK/SK签名通过了，发送过去 他返回什么呢？





9.20

首先把可能用到的API分类，疑问产生在这里，应当经过处理后再提问，昨天是发生了疑问 但是没有通过进一步处理，那获得更细致的分析就还得让他人也进行同样的处理，这里可以提升。

```
// GetRegions 获取地域列表 // GetVpcs 获取vpc列表 // GetInstances 获取实例列表
在OTenantSecurity中  3.6.1有查询区域列表  无实例查询  无VPC查询

在运营API文档中，第五章节有API列表汇总，其中ConsoleHome 第25 查询所有region 27查询region详情（13.27）
VRM云服务中 第63 查询VPC列表  无实例信息仅有资源实例信息查询 

在MOAPIAdapter中 第六章6.9节云服务管理下存在查询region信息。 上面管理侧也有查询region信息。 无查询VPC信息  无查询实例信息

```



9.21

智能回写功能，

期望：提供导出 标注错误的EXCEL文档  

许多部分都涉及到导出 也许各个部分的导出错误不一

找到错误信息对应的Excel部分 然后以处理错误手段标注



Excel第二列 或者某个地方汇总纪录部分



涉及到代码部分：
对导入部分错误的处理   错误暂存到Redis中    下载错误文件时 生成新错误文件



新方法放在comment中	



hosts .go 中的ImportHost内容：

```
请求ID和上下文: 从请求头中获取请求ID（rid）和上下文（ctx）。
语言设置: 根据HTTP请求获取语言设置并创建默认的语言对象和错误对象。
文件上传: 从表单数据中尝试获取名为“file”的文件。如果获取失败，它将记录错误并向客户端返回一个表示文件未找到的错误消息。
参数提取: 从表单中提取名为“params”的参数值。如果没有找到此参数，它将记录错误并返回需要设置参数的错误消息给客户端。
参数解析: 尝试将提取的“params”参数解析为metadata.ExcelImportAddHostInput结构。如果解析失败，会记录错误并向客户端返回一个参数值无效的错误消息。
设置代理头: 为请求设置代理头部，这通常在反向代理场景中使用。
文件保存: 生成一个随机的文件名并检查导入资源路径是否存在。如果不存在，它将尝试创建一个目录。然后，它尝试将从表单中获取的文件保存到这个路径下。如果保存失败，它会记录错误并向客户端返回内部服务器错误。
上半部分：函数处理从HTTP请求中上传的文件，并将其保存到服务器的指定目录，同时提取和解析与文件相关的参数。

保存文件: 生成一个唯一的文件名，然后尝试将上传的文件保存到这个名称的路径中。如果保存失败，将记录错误并返回给客户端一个文件保存失败的消息。
删除临时文件: 使用defer关键字来保证在函数退出之前删除之前保存的临时文件。如果删除失败，会记录错误。
读取Excel文件: 使用xlsx.OpenFile方法尝试打开保存的Excel文件。如果打开失败，将记录错误并返回给客户端一个打开文件失败的消息。
主机导入: 调用s.Logics.ImportHosts方法来实际执行主机导入的逻辑。这个方法接受一系列参数，包括上下文（ctx）、Excel文件对象（f）、请求头（c.Request.Header）、默认语言（defLang）、模块ID（inputJSON.ModuleID）、操作类型（inputJSON.OpType）、关联条件（inputJSON.AssociationCond）以及对象唯一ID（inputJSON.ObjectUniqueID）。
返回结果: 最后，函数使用c.JSON方法将ImportHosts返回的结果发送回客户端，状态码为http.StatusOK。
工作流程：
首先，获取并解析用户上传的文件和其他相关参数。
然后，保存用户上传的文件到临时位置，并确保函数结束时删除这个临时文件。
使用这个文件来调用实际的主机导入逻辑。
最后，返回执行结果给客户端。
这样的设计方式确保上传的文件能被正确解析和处理，同时还考虑到了错误处理和资源清理。
```





9.22

代码中存在时间戳 21年

logics\host.go

```
gethostdata方法接收了一系列参数，包括appID（应用程序ID）、hostIDArr（主机ID数组）、hostFields（主机字段数组）、exportCond（导出条件对象）、header（HTTP头部对象）、defLang（语言相关对象）。
首先，获取HTTP请求的ID，并创建一个空的主机信息列表 hostInfo 和一个用于构建搜索条件的 sHostCond 对象，并初始化其中的字段。
检查 hostIDArr 和 exportCond.Condition 的长度，如果两者都为空，则返回错误信息 "both_hostid_exportcond_empty"。
如果 hostIDArr 非空，则构建查询条件。首先检查 hostIDArr 的长度是否超过限制，如果超过限制，则返回错误信息 "host_id_len_err"。然后，逐步构建主机、业务、集群、模块的搜索条件，并将它们加入 condArr 数组中。
如果 hostIDArr 为空，则表示使用了导出条件，根据导出条件中的信息构建搜索条件。检查导出条件中的分页限制是否合法，如果不合法，则返回错误信息 "export_page_limit_err"。然后，根据需要设置主机字段，并更新搜索条件中的主机条件。
最后，调用 lgc.Engine.CoreAPI.ApiServer().GetHostData 方法来获取主机数据。方法接收上下文对象、HTTP头部和搜索条件，并返回结果和错误信息。
如果发生错误，则记录错误日志并返回空的主机信息列表和错误。
如果结果的 Result 字段为假，则记录错误日志并返回相应的错误信息。
如果一切顺利，返回获取到的主机信息列表。
```

目前我在编写CMDB系统下的模型导入导出时的智能回写功能，就是在导入时如果导入的excel文档有错误的内容，如不符合规则的字段，意料之外的字段等等 我们的设计是对这些错误的内容进行标注 以便于修改，然后再sheet页面汇总错误信息 以便对照。  详细设计文档 内容包括：4.7.1.功能描述4.7.2.设计思路4.7.3.性能指标4.7.4.输入项4.7.5.输出项
4.7.6.设计方法(算法)4.7.7.流程逻辑
4.7.8.接口设计4.7.9.存储分配4.7.10.限制条件4.7.11.测试计划4.7.12.验证标准



4.7 智能回写功能设计文档
4.7.1 功能描述
为了提高用户导入数据的效率和准确性，CMDB系统提供一个智能回写功能，该功能能够在用户导入的Excel文件中自动标注出不合规的数据，同时在一个单独的sheet页面上汇总所有错误信息，以方便用户进行修改和对照。
4.7.2 设计思路
采用两阶段的校验机制：首先对整个文档进行预校验，找出明显的格式和结构错误；其次，对每个数据项进行深入校验，包括数据完整性、格式和逻辑正确性等。
4.7.3 性能指标

1.处理速度：能够在5秒内处理1000行数据。
2.错误识别率：&gt;98%
3.错误汇总生成速度：&lt;2秒

4.7.4 输入项

4.用户上传的Excel文件。
5.CMDB系统中的数据模型和字段规则。

4.7.5 输出项

6.带有错误标注的Excel文件。
7.错误汇总sheet页面。

4.7.6 设计方法(算法)

8.使用表格解析库如xlsx读取Excel文件。
9.根据CMDB数据模型进行数据结构预校验。
10.对每个数据项进行深度校验。
11.对于发现的错误，用特定颜色或注释进行标注。
12.汇总所有错误并生成一个新的sheet页面。

4.7.7 流程逻辑

13.用户上传Excel文件。
14.系统进行预校验，检查文档结构。
15.针对每个数据项进行深度校验。
16.标注错误并生成汇总sheet页面。
17.返回处理后的Excel文件。

4.7.8 接口设计

18.输入接口：POST /cmdb/upload/excel
19.输出接口：GET /cmdb/download/excel/:file_id

4.7.9 存储分配

20.临时存储：用于存放用户上传的Excel文件。
21.结果存储：用于存放处理后的Excel文件，包括标注错误和汇总sheet。

4.7.10 限制条件

22.支持的Excel版本：Excel 2007及以上版本。
23.单次上传的最大行数：10,000行。

4.7.11 测试计划

24.针对各种数据格式和结构进行单元测试。
25.进行集成测试，确保流程的连贯性和正确性。
26.进行性能测试，确保满足性能指标。

4.7.12 验证标准

27.所有测试案例应正确运行，无明显的错误。
28.性能指标应满足设计要求。



9.25

上周完成了华为云AK/SK代码部分的编写，实现了身份认证文档到代码的转变。参与讨论如何设计智能回写实施，熟悉了CMDB中智能回写部分的代码



先进行本地配置，看看能不能把代码运行：

s.Config.Site.HtmlRoot = "D:/1STudy/go/CMDB/src/web_server/ui/"

配置运行为：

![image-20230925113255162](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230925113255162.png)

还是要回归到具体内容。

 git -c http.sslVerify=false push -u CCDD testing 

下午先把主要内容整理出来，然后开始挪。



9.26

昨天已经看了一遍结构与内容，今天涉及到具体的接入。

主要处理内容为D:\1STudy\go\CMDB\src\web_server\logics\host.go  中GetImportHosts，其中是以云区域名称为主 然后从excel中获取的信息进行验证  



今天开了很多会，了解了导入错误处理逻辑， 这里是每个导入对应的云区域为主然后从excel中获取的信息进行验证 以此来发现错误   



9.27  

导入功能存在问题，目前的字段导入是只有鉴重功能 无纠错功能  

1 如果导入默认字段 （灰色字块 ）则后续都无法更改 无法删除  只能删除整个模型重建。

2 如果导入新文件 只是旧文件的更新名称 则以旧文件为主 不导入 

3 default默认分组的字段都不可编辑 不可修改 



4 导入实现部分有错误或者遗漏， 

- 第5行：字段 server_room_7，禁止修改字段: bk_template_id
- 第9行：字段 server_room_10，禁止修改字段: bk_template_id
- 第10行：字段 server_room_11，禁止修改字段: bk_template_id

而我分别在第六行的必填文本为空 第七行的英文名为空  第8行中文名为空 文本为空 第九行是bool未填

![image-20230927163835820](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230927163835820.png)

新建导入字段，只有7 10 11成功导入 但是不返回导入失败信息：

![image-20230927164238241](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230927164238241.png)

![image-20230927164247041](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20230927164247041.png)






方案1：直接操作Excel文件

1.上传并读取Excel：使用例如excelize这样的Go库来读取上传的Excel文件。
2.验证内容并标红错误：在读取Excel的内容后，对内容进行验证。对于验证失败的内容，使用库函数对相应的单元格进行标红处理。
3.汇总错误信息：在一个新的Sheet页中汇总所有的错误信息。
4.保存并提供下载：保存处理后的Excel文件，并为用户提供一个下载链接。

方案2：结合临时存储

5.上传并读取Excel：与方案1相同，读取上传的Excel文件。
6.验证内容：进行验证，并将错误信息存储在一个数据结构中。
7.存储错误信息：将这个错误信息的数据结构临时存储在Redis或其他缓存中。
8.用户请求回写：当用户请求智能回写时，从Redis中读取错误信息，对原Excel进行标红处理，并在新Sheet中汇总错误信息。
9.保存并提供下载：保存处理后的Excel并为用户提供下载链接。

方案3：前端+后端结合

10.上传并读取Excel：与方案1相同。
11.验证内容：进行验证，并将错误的行和列信息返回给前端。
12.前端标红和汇总：前端使用例如JavaScript的Excel库（例如SheetJS）对上传的Excel进行标红处理，并汇总错误信息。
13.提供下载：前端生成新的Excel文件并为用户提供下载链接。

方案4：结合临时文件系统

14.上传并读取Excel：与方案1相同。
15.验证内容：进行验证，并将错误信息存储在一个数据结构中。
16.保存错误信息：将Excel和错误信息的数据结构保存在服务器的临时文件系统中。
17.用户请求回写：根据存储的错误信息对Excel进行标红处理，并汇总错误信息。
18.保存并提供下载：保存处理后的Excel并为用户提供下载链接。

这些方案都有其优点和缺点。例如，方案1可能更简单，但可能不适合大量的并发请求。方案2和4提供了一种将验证和回写过程分离的方法，这可能会提高用户体验。方案3则将大部分逻辑移到前端，可能更适合web应用。




​	目前的情况是 已经能捕捉错误内容errMsg 然后如何进行错误设计。 暂不清楚这个errMsg输出是个什么样的。 我们要调用其中的字段进行标红，把错误内容转换语言汇总放在另一个地方。

​	目前期望能看到errMsg内容，有两种方法 1是把整个项目或者这个模块跑起来 动底层代码以进行调试 

​	2 已废弃

​	目前已经能在本地配置运行现有模块，通过使用postman来进行调试，这方面我会回去了解。我会尝试调用代码  在构建完之后测试错误输出 然后根据错误输出 错误类型进行智能回写的设计

​	至于如何保存如何导出 ，一是要保存原Excel内容与错误内容 导出也是要导出经过处理后的Excel

​	保存部分  f, err := xlsx.OpenFile(filePath, xlsx.UseDiskVCellStore)  这里使用xlsx.UseDiskVCellStore  可能相关的内容会有临时保存的部分。或者有什么其他的包。但是这样不知道能不能实现根据错误内容来进行错误标注。

​	另一种就是保存Excel与错误内容到Redis，然后再建函数用错误内容对Excel进行处理。

​	第三种，在代码中进行导入读取内容的时候，如果有错误就进行Excel字段处理？  但是似乎目前的代码在这读取层没有与Excel具体内容进行交互 只是记录错误  与Excel进行交互也只是在一开始读取其内容。

​	然后如何根据错误内容来操作Excel，这其中要根据错误内容格式来进行处理，也要用到Excel相关的包。

这里以 f, err := xlsx.OpenFile(filePath, xlsx.UseDiskVCellStore)   f的内容为Excel内容，我们可能到时候有个func  

handleerrMsg（msg errMsg，f  file） any{}    返回根据错误内容修改的Excel文件？ 暂时不考虑那么深 先把目前的内容先做完。

9.28

生成请求调试。



关于智能回写功能，目前的进度是 已经能运行相关模块 并发送请求和返回响应。 可是对于importXXX 在接口文档中没有对应的内容。 暂时无法得知这个相应的请求参数内容包括哪些。  目前进行的测试： POST  127.0.0.1:8090 /object/object/:bk_obj_id/import  在其body form-data中上传文件 Excel  ；返回错误信息{

  "bk_error_code": 1111001,

  "bk_error_msg": "未找到上传文件",

  "data": null,

  "result": false

}





10.5

查找了相关内容，发现都是一些设计方案。而我们现在需要的是项目中的内容。暂时获取这些部分，要么看代码 把代码看的特别透，要么



10.7

![image-20231007095709322](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007095709322.png)

：

POST /object/object/testingmode1/import HTTP/1.1 Accept: application/json, text/plain, */* Accept-Encoding: gzip, deflate Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6 Cc_Request_Id: cc0000cmgbn95dr5ng000pj1ag Connection: keep-alive Content-Length: 10169 Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryrsvYOkO58i6Bxsre Cookie: HTTP_BLUEKING_SUPPLIER_ID=0; http_scheme=http; cc3=MTY5NjY0MzcxOHxOd3dBTkVFeVRFdFJWRkpZVkVjMFZ6WkRVMVZSVFZKQlNWcEdSRmN5UkZOR1ZreENNek5NVkZWUE1sWkdSbEJZTmtwWk56Uk9Ta0U9fBYzPLdXEeMoOryfJu-XDMqIQ0D595n8PDyr3Exnevgr HTTP_BLUEKING_SUPPLIER_ID: 0 Host: 10.0.254.245:8081 Origin: http://10.0.254.245:8081 Referer: http://10.0.254.245:8081/ User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36 Edg/117.0.2045.47 traceparent: 00-e27591d8cd6c47ff8ccb663afd3d3a7c-30a093e9e3ed496b-01



导入错误内容：

![image-20231007102230692](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007102230692.png)

一个字段的参数名称  一个行数 一个错误信息





代码部分，

最外层导入：

拆分请求 第一步处理数据，包括处理Excel 解析参数 然后传给下一个导入

第二层导入：

根据操作类型不同，执行关联关系导入之流或者主机导入

第二层导入中的第三层导入会逐行比较Excel内容，然后返回errmsg汇总给第二层 第二层返回错误汇总data中。

在第一层最下方进行错误内容的处理

预处理函数进行了什么处理：

获取模型字段信息；获取每一条数据，得到它从哪一行开始，哪一行结束，

获取表格字段以及表格字段里的属性在excel中的列的位置，字段的所占列的启始位置，以及将表格字段里的属性构造出property。

![image-20231007140934353](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007140934353.png)



目前已经弄清楚各个部分 各个字段在导入中是如何获取的。 可是似乎目前的错误处理部分还是不是很理想，我们期望其为有特点 或者说是分类了进行处理 像他的输出那样至少是显示（导入字段错误：

![image-20231007145853370](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007145853370.png)

 这里已经有鉴定其内容更新失败， 那这个错误分类埋藏在哪里呢？

今天务必要把这个搞出来



目前导入逻辑： 如果有重名 不更改  如果有空白 不导入直接跳过。



知道错误内容，但是不清楚处理逻辑。   这里的data记录了错误信息 既有包括内容相关的 也有包括结构上的 

下一步就是把错误内容区分，先获得内容错误的内容，查看哪里有包括其的部分。

![image-20231007174610927](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007174610927.png)

主要部分在此。那就是要查看导入Excel预处理详细涉及部分与这里get的部分。

![image-20231007191056660](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007191056660.png)

预处理返回值![image-20231008141357593](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231008141357593.png)



10.8

目前已经非常清楚结构，今天主要是把errmsg内容分离，获取Excel错误内容，根据其标志。如果没有标致 我们就手动写代码加上。

存入redis中设置时间ttl。到时就过期

第一部分，如果单元格内容为空，原本是跳过这一行，也许我们在此加上报空部分。

![image-20231008115911456](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231008115911456.png)

```
	if fields != nil {
			field, hasField = fields[fieldName]
			// 如果这个字段是表格类型，那么不在这个函数中去处理获取
			if hasField && field.PropertyType == common.FieldTypeInnerTable {
				continue
			}
		}
```

这里的 field, hasField = fields[fieldName]  是判断fields里是否有X ，并且取X下对应的值。

后续表格类型如何处理。

![image-20231008144632842](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231008144632842.png)



![image-20231008162249967](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231008162249967.png)

包括成功内容与失败内容，

![image-20231007102230692](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007102230692.png)

在失败内容中 获取row  找到对应错误地方 Excel处理。



错误内容： 如果已经存在 则报错 不更新；  如果有空 则跳过 无视



已知func (lgc *Logics) ImportInsts 返回resultData mapstr.MapStr, errCode int, err error  其函数内是返回return lgc.importInsts  而lgc.importInsts中有func (lgc *Logics) importInsts(ctx context.Context, f *xlsx.File, objID string, header http.Header,
    defLang lang.DefaultCCLanguageIf, modelBizID int64, asstObjectUniqueIDMap map[string]int64, objectUniqueID int64) (
    resultData mapstr.MapStr, errCode int, err error) {

​    rid := util.GetHTTPCCRequestID(header)
​    resultData = mapstr.New()   

var successMsgs []string
var errMsgs []string 

resultData["success"] = successMsgs
resultData["error"] = errMsgs 

然后return 

则在最外层data, errCode, err := s.Logics.ImportInsts(context.Background(), f, objID, c.Request.Header, defLang,
    inputJSON.BizID, inputJSON.OpType, inputJSON.AssociationCond, inputJSON.ObjectUniqueID)  

我下面要提取到这个data中的resultData["error"] = errMsgs  这个errmsgs部分，并将其转为一个整体string存储起来



记录第一次修改，修改内容为将错误存入redis：

![image-20231008175808642](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231008175808642.png)

```
data, errCode, err := s.Logics.ImportInsts(context.Background(), f, objID, c.Request.Header, defLang,
		inputJSON.BizID, inputJSON.OpType, inputJSON.AssociationCond, inputJSON.ObjectUniqueID)

	if err != nil {
		msg := getReturnStr(errCode, err.Error(), data)
		errMsgs, ok := data["error"].([]string) //代码问题
		if ok {

			errMsg := strings.Join(errMsgs, "\n")
			// 存在errMsgs就将其存入Redis
			key := "ImportInsts error_msg" // 为每个errMsg生成一个唯一的key
			ttl := 120 * time.Second       // 设置TTL为120秒
			、、ctx, cancel := context.WithTimeout(context.Background(), ttl)
defer cancel()
			redisErr := s.CacheCli.Set(ctx, key, errMsg, ttl).Err()
			if redisErr != nil {
				// 处理Redis存储错误，可选择记录日志
				c.JSON(http.StatusInternalServerError, gin.H{"error": fmt.Sprintf("Failed to store errmsg in Redis: %s", errMsg)})
				return
			}

		}
		c.String(http.StatusOK, string(msg))
		return
	}

	c.String(http.StatusOK, getReturnStr(0, "", data))
}
```

Set(ctx context.Context, key string, value interface{}, expiration time.Duration) StatusResult

```
第二次：
	f, err := xlsx.OpenFile(filePath, xlsx.UseDiskVCellStore)
	if err != nil {
		msg := getReturnStr(common.CCErrWebOpenFileFail, defErr.Errorf(common.CCErrWebOpenFileFail,
			err.Error()).Error(), nil)
		c.String(http.StatusOK, string(msg))
		return
	}
	errsheet, err := f.AddSheet("errsheet")
	// 在新的工作表中添加错误内容
	// 存储错误信息的键值对，键为错误类型，值为错误信息
	errorMap := map[string]string{
		"web_srv_not_found_file":   "未找到上传文件",
		"web_srv_file_save_failed": "保存文件失败",
		"web_en_name_required":     "英文名(必填)",
		//... 更多错误类型和信息
	}
	for errorType, errorMsg := range errorMap {
		row := errsheet.AddRow() // 创建新行
		cell1 := row.AddCell()   // 创建新单元格
		cell1.Value = errorType  // 第一个单元格存储错误类型
		cell2 := row.AddCell()   // 创建新单元格
		cell2.Value = errorMsg   // 第二个单元格存储错误信息
	}
	// 下面的错误汇总如何进行区分成上面的格式，这是后期要处理的问题，也许还有要添加多个新的错误类型
	err = f.Save("yourfile.xlsx")
	if err != nil {
		blog.Errorf("cannot save file: %v", err)
	}
	if err != nil {
		blog.Errorf("AddErrSheet failed, err: %+v, rid: %s", err, rid)
	}

	data, errCode, err := s.Logics.ImportInsts(context.Background(), f, objID, c.Request.Header, defLang,
		inputJSON.BizID, inputJSON.OpType, inputJSON.AssociationCond, inputJSON.ObjectUniqueID)

	if err != nil {
		msg := getReturnStr(errCode, err.Error(), data)
		errMsgs, ok := data["error"].([]string) //代码问题
		if ok {

			errMsg := strings.Join(errMsgs, "\n")
			// 存在errMsgs就将其存入Redis
			key := "ImportInsts error_msg" // 为每个errMsg生成一个唯一的key
			ttl := 120 * time.Second       // 设置TTL为120秒

			redisErr := s.CacheCli.Set(context.Background(), key, errMsg, ttl).Err()
			if redisErr != nil {
				// 处理Redis存储错误，可选择记录日志
				c.JSON(http.StatusInternalServerError, gin.H{"error": fmt.Sprintf("Failed to store errmsg in Redis: %s", errMsg)})
				return
			}

		}
		c.String(http.StatusOK, string(msg))
		return
	}

	c.String(http.StatusOK, getReturnStr(0, "", data))
}
```

已经把错误内容存储在Redis中，下面要新建业务，把原Excel看是复制还是直接操作  然后也存到Redis中，然后 新业务是把Excel errmsg都拿出来 然后根据errmsg 操作Excel 然后可以选择导出。



新api时候， 使用xlsx第三方库 代码中的  来进行创建Excel 和 构筑。

研究导出模式。



definitions中存在颜色设置，这里也许我会添加表格框 或者格子背景颜色

![image-20231009153811947](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231009153811947.png)



处理错误信息，  对原本的内容进行标注修改  然后再是导出。

start to end  X列  row  行 根据这个内容 进行错误标注。

错误语言内容：

![image-20231009162714965](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231009162714965.png)

错误代码内容：

![image-20231009162831200](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231009162831200.png)

那这里

![image-20231007145853370](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231007145853370.png)

内容为 %d row   fieldname   ，  errid    field



剩余工作内容：将错误分离   生成Excel副本 并将数据存储到redis  //根据分离错误修改Excel副本数据  将修改后的数据变为Excel  设置导出 



先分拣错误类型吧。

![image-20231009180230020](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231009180230020.png)

这里  如果表头有 但是单元格没有  如果单元格内容为必填  他这里是直接跳过去找下面的咯  我觉得这应该是个遗漏的错误。

这里的上传在某个过程中把他转换为更新了，然后到更新有更新的错误。

```
预处理部分：
1.检查Excel表: 首先检测Excel文件有无内容，是否包含至少一个Sheet，同时选取第一个Sheet用于后续处理。
2.获取模型字段信息： 调用lgc.GetObjFieldIDs获取字段信息。
3.检查Excel表头： 检查Excel表的头部是否符合字段信息的要求，也就是看看Excel表头的列名是否和此对象的字段名匹配。同时建立字段与其在Excel中的列编号的映射。
4.获取数据范围： 识别出Excel中每一条数据记录的起始行和结束行，形成一个记录范围列表。
5.获取表格字段属性在Excel中的位置： 此步骤会获取每个字段属性在Excel表中所处的列位置，以及构造出Property属性。
最后会返回一个ImportExcelPreData结构体实例，其中包括模型的字段信息，字段名与Excel列的映射，数据范围列表，字段属性与类位置关系等信息

```

//将错误分离  : 整体，转换前

   生成Excel副本  ：未完成

   并将数据存储到redis   ：已完成 错误整体  Excel未存入 在代码中使用了磁盘存储

//根据分离错误修改Excel副本数据   ：直接将错误整体导入第二页 未完成 

   将修改后的数据变为Excel  设置导出  ：将带有错误页的文件转化为Excel 导出下载

​	

```
func (s *Service) ExportInst(c *gin.Context) {

	rid := util.GetHTTPCCRequestID(c.Request.Header)
	ctx := util.NewContextFromGinContext(c)
	webCommon.SetProxyHeader(c)
	language := webCommon.GetLanguageByHTTPRequest(c)
	defLang := s.Language.CreateDefaultCCLanguageIf(language)
	defErr := s.CCErr.CreateDefaultCCErrorIf(language)
	pheader := c.Request.Header
 
	input := &excelExportInstInput{} 	usernameMap, propertyList, err := s.Logics.GetUsernameMapWithPropertyList(c, objID, instInfo, s.Config)
	if err != nil {
		blog.Errorf("ExportInst failed, get username map and property list failed, err: %+v, rid: %s", err, rid)
		_, _ = c.Writer.Write([]byte(getReturnStr(common.CCErrWebGetUsernameMapFail, defErr.Errorf(
			common.CCErrWebGetUsernameMapFail, objID).Error(), nil)))
	}

	org, orgPropertyList, err := s.Logics.GetDepartmentDetail(c, objID, s.Config, instInfo)
	if err != nil {
		blog.Errorf("get department map and property list failed, err: %+v, rid: %s", err, rid)
		_, _ = c.Writer.Write([]byte(getReturnStr(common.CCErrWebGetDepartmentMapFail, defErr.Errorf(
			common.CCErrWebGetDepartmentMapFail, err.Error()).Error(), nil)))
	}

	instInfo, rowCountArr, err := s.Logics.BuildDataWithTable(objID, instInfo, fields, pheader)
	if err != nil {
		blog.Errorf("build data with table failed, err: %v, rid: %s", err, rid)
		c.String(http.StatusInternalServerError, fmt.Sprintf("build data with table failed, err: %+v", err))
		return
	}

	file := xlsx.NewFile()
	err = s.Logics.BuildExcelFromData(ctx, objID, fields, nil, instInfo, file, pheader, modelBizID, usernameMap,
		propertyList, org, orgPropertyList, input.AssociationCond, input.ObjectUniqueID, rowCountArr)
	if nil != err {
		blog.Errorf("ExportHost object:%s error:%s, rid: %s", objID, err.Error(), rid)
		_, _ = c.Writer.Write([]byte(getReturnStr(common.CCErrCommExcelTemplateFailed, defErr.Errorf(
			common.CCErrCommExcelTemplateFailed, objID).Error(), nil)))
		return
	}

	dirFileName := fmt.Sprintf("%s/export", webCommon.ResourcePath)
	if _, err = os.Stat(dirFileName); err != nil {
		blog.Warnf("os.Stat failed, filename: %s, will retry with os.MkdirAll, err: %+v, rid: %s", dirFileName, err, rid)
		if err := os.MkdirAll(dirFileName, os.ModeDir|os.ModePerm); err != nil {
			blog.Errorf("os.MkdirAll failed, filename: %s, err: %+v, rid: %s", dirFileName, err, rid)
		}
	}

	dirFileName = fmt.Sprintf("%s/%s", dirFileName, fmt.Sprintf("%dinst.xlsx", time.Now().UnixNano()))
	if err := logics.ProductExcelCommentSheet(ctx, file, defLang); err != nil {
		blog.Errorf("export instance failed, err: %v, rid: %s", err, rid)
		_, _ = c.Writer.Write([]byte(getReturnStr(common.CCErrWebCreateEXCELFail, defErr.Errorf(
			common.CCErrCommExcelTemplateFailed, err.Error()).Error(), nil)))
		return
	}
	if err = file.Save(dirFileName); err != nil {
		blog.Errorf("ExportInst save file error:%s, rid: %s", err.Error(), rid)
		_, _ = c.Writer.Write([]byte(getReturnStr(common.CCErrWebCreateEXCELFail, defErr.Errorf(
			common.CCErrCommExcelTemplateFailed, err.Error()).Error(), nil)))
		return
	}
	logics.AddDownExcelHttpHeader(c, fmt.Sprintf("bk_cmdb_export_inst_%s.xlsx", objID))
	if err := s.writeFile(c, dirFileName, rid); err != nil {
		c.String(http.StatusInternalServerError, fmt.Sprintf("write exported file failed, err: %+v", err))
		return
	}

	defer func(dirFileName string, rid string) {
		if err := os.Remove(dirFileName); err != nil {
			blog.Errorf("remove file %s failed, err: %+v, rid: %s", dirFileName, err, rid)
		}
	}(dirFileName, rid)
}
 这个函数是将哪些数据转化为Excel内容 然后如何构建这个Excel 最后是如何导出Excel的？
```



10.10 

直接将错误整体导入第二页 完成

下面要将修改后的数据变为Excel 导出



导出部分：

```
1.准备上下文和语言信息：

2.rid：从请求头获取请求 ID。
3.创建上下文 ctx。
4.设置代理头 webCommon.SetProxyHeader(c)。
5.获取请求中的语言信息 language。
6.创建默认的 CC 语言 defLang 和错误 defErr 对象。

7.获取数据：

8.创建一个空的 excelExportInstInput 对象 input。
9.调用 s.Logics.GetUsernameMapWithPropertyList 和 s.Logics.GetDepartmentDetail 来获取用户名映射、属性列表、组织信息等数据。

10.构建数据：
11.调用 s.Logics.BuildDataWithTable 来构建数据，这可能包括从数据库或其他源收集数据，并返回 instInfo 和 rowCountArr。

12.创建 Excel 文件：

13.创建一个新的 Excel 文件对象 file 使用 xlsx.NewFile()。

14.将数据填充到 Excel 中：

15.调用 s.Logics.BuildExcelFromData 来将数据填充到 Excel 文件中，该函数接受数据、文件对象、配置信息等，将数据映射到 Excel 单元格中。

16.准备导出目录和文件名：

17.准备一个导出目录 dirFileName，通常是在 webCommon.ResourcePath 下的一个子目录。
18.使用当前的时间戳为文件名创建一个唯一的文件名，并将其与目录名组合。

19.添加 Excel 注释（可选）：

20.调用 logics.ProductExcelCommentSheet 来添加 Excel 注释或元数据信息。

21.保存 Excel 文件：

22.使用 file.Save(dirFileName) 保存构建好的 Excel 文件到指定的目录和文件名。

23.设置 HTTP 响应头并发送文件：

24.调用 logics.AddDownExcelHttpHeader 来设置 HTTP 响应头，以便浏览器能够下载 Excel 文件。
25.最后，通过 s.writeFile 方法将生成的 Excel 文件发送给客户端。

26.清理临时文件（可选）：
27.在函数的最后，通过 os.Remove 方法删除生成的临时 Excel 文件。

总之，这个函数首先准备上下文和语言信息，然后获取所需的数据，构建一个 Excel 文件，将数据填充到 Excel 文件中，保存文件，然后通过 HTTP 响应头将文件发送给客户端。最后，它可以选择性地清理生成的临时文件。这个过程允许你将数据转化为 Excel 内容并导出给用户。
```



关键内容：BuildExcelFromData

处理的参数  header  -用于获取rid - 获取ccLang

然后在传入进来的xlsxFile中加一个inst页面

添加系统字段，补充了一部分fields的属性。用于区分主机，模型

添加了一个过滤器，可能是用于删除一些不需要的字段

然后规定了一些文件头部的内容  包括风格 参数 处理字段等



使用书签





10.11

梳理下包含的文件 definitions。go  web.json 



```
	errsheet, err := f.AddSheet("errsheet")
	// 在新的工作表中添加错误内容
	// 存储错误信息的键值对，键为错误类型，值为错误信息
	errorMap := map[string]string{
		"web_srv_not_found_file":   "未找到上传文件",
		"web_srv_file_save_failed": "保存文件失败",
		"web_en_name_required":     "英文名(必填)",
		// 更多错误类型和信息
	}
	for errorType, errorMsg := range errorMap {
		row := errsheet.AddRow() // 创建新行
		cell1 := row.AddCell()   // 创建新单元格
		cell1.Value = errorType  // 第一个单元格存储错误类型
		cell2 := row.AddCell()   // 创建新单元格
		cell2.Value = errorMsg   // 第二个单元格存储错误信息
	}
	// 下面的错误汇总如何进行区分成上面的格式，这是后期要处理的问题，也许还有要添加多个新的错误类型
	err = f.Save("yourfile.xlsx")
	if err != nil {
		blog.Errorf("cannot save file: %v", err)
	}
	if err != nil {
		blog.Errorf("AddErrSheet failed, err: %+v, rid: %s", err, rid)
	}

	data, errCode, err := s.Logics.ImportInsts(context.Background(), f, objID, c.Request.Header, defLang,
		inputJSON.BizID, inputJSON.OpType, inputJSON.AssociationCond, inputJSON.ObjectUniqueID)

	if err != nil {
		msg := getReturnStr(errCode, err.Error(), data)
		errMsgs, ok := data["error"].([]string) //代码问题
		if ok {

			errMsg := strings.Join(errMsgs, "\n")
			// 存在errMsgs就将其存入Redis
			key := "ImportInsts error_msg" // 为每个errMsg生成一个唯一的key
			ttl := 120 * time.Second       // 设置TTL为120秒

			redisErr := s.CacheCli.Set(context.Background(), key, errMsg, ttl).Err()
			if redisErr != nil {
				// 处理Redis存储错误，可选择记录日志
				c.JSON(http.StatusInternalServerError, gin.H{"error": fmt.Sprintf("Failed to store errmsg in Redis: %s", errMsg)})
				return
			}

		}
		c.String(http.StatusOK, string(msg))
		return
	}
	
```

```

s.ImportInstErrhd(c, data)
	ErroraddintoExcel(f, data)
	if err != nil {
		msg := getReturnStr(errCode, err.Error(), data)
		c.String(http.StatusOK, string(msg))
		return
	}

	c.String(http.StatusOK, getReturnStr(0, "", data))
}
func ErroraddintoExcel(f *xlsx.File, data map[string]interface{}) {
	errorMap := data["error"].(map[string]string)
	//待处理
	errsheet, err := f.AddSheet("errsheet")
	if err != nil {
		blog.Errorf("ErroraddintoExcelfailed, err: %v", err)
	}
	for errorType, errorMsg := range errorMap {
		row := errsheet.AddRow()
		cell1 := row.AddCell()
		cell1.Value = errorType
		cell2 := row.AddCell()
		cell2.Value = errorMsg
	}
	err = f.Save("xlslpath")
	if err != nil {
		blog.Errorf("cannot save file: %v", err)
	}

}
func (s *Service) ImportInstErrhd(c *gin.Context, data map[string]interface{}) {
	if data["error"] != nil {

		errMsgs, ok := data["error"].([]string) //代码问题
		if ok {

			errMsg := strings.Join(errMsgs, "\n")
			// 存在errMsgs就将其存入Redis
			key := "ImportInsts error_msg" // 为每个errMsg生成key,每个import不同，这里要额外加标识吗
			ttl := 120 * time.Second       // 设置TTL为120秒

			redisErr := s.CacheCli.Set(context.Background(), key, errMsg, ttl).Err()
			if redisErr != nil {
				// 处理Redis存储错误，可选择记录日志
				c.JSON(http.StatusInternalServerError, gin.H{"error": fmt.Sprintf("Failed to store errmsg in Redis: %s", errMsg)})
				return
			}
		}

	}
}

```

jiaru导出

```
data, errCode, err := s.Logics.ImportInsts(context.Background(), f, objID, c.Request.Header, defLang,
		inputJSON.BizID, inputJSON.OpType, inputJSON.AssociationCond, inputJSON.ObjectUniqueID)
	s.ImportInstErrhd(c, data)
	s.ErroraddintoExcel(c, rid, objID, f, data)
	if err != nil {
		msg := getReturnStr(errCode, err.Error(), data)
		c.String(http.StatusOK, string(msg))
		return
	}

	c.String(http.StatusOK, getReturnStr(0, "", data))
}
func (s *Service) ErroraddintoExcel(c *gin.Context, rid string, objID string, f *xlsx.File, data map[string]interface{}) {
	errorMap := data["error"].(map[string]string)
	//待处理
	errsheet, err := f.AddSheet("errsheet")
	if err != nil {
		blog.Errorf("ErroraddintoExcelfailed, err: %v", err)
	}
	for errorType, errorMsg := range errorMap {
		row := errsheet.AddRow()
		cell1 := row.AddCell()
		cell1.Value = errorType
		cell2 := row.AddCell()
		cell2.Value = errorMsg
	}

	dirFileName := fmt.Sprintf("%s/export", webCommon.ResourcePath)
	_, err = os.Stat(dirFileName)
	if nil != err {
		blog.Warnf("os.Stat failed, will retry with os.MkdirAll, filename: %s, err: %+v, rid: %s", dirFileName, err, rid)
		if err := os.MkdirAll(dirFileName, os.ModeDir|os.ModePerm); err != nil {
			blog.Errorf("os.MkdirAll failed, filename: %s, err: %+v, rid: %s", dirFileName, err, rid)
		}
	}
	fileName := fmt.Sprintf("%d_%s.xlsx", time.Now().UnixNano(), objID)
	dirFileName = fmt.Sprintf("%s/%s", dirFileName, fileName)
	err = f.Save(dirFileName)
	if err != nil {
		blog.Errorf("ExportInst save file error:%s, rid: %s", err.Error(), rid)
		fmt.Printf(err.Error())
	}
	logics.AddDownExcelHttpHeader(c, fmt.Sprintf("bk_cmdb_model_%s.xlsx", objID))
	c.File(dirFileName)

	if err := os.Remove(dirFileName); err != nil {
		blog.Errorf("os.Remove failed, filename: %s, err: %+v, rid: %s", dirFileName, err, rid)
	}

}
func (s *Service) ImportInstErrhd(c *gin.Context, data map[string]interface{}) {
	if data["error"] != nil {

		errMsgs, ok := data["error"].([]string) //代码问题
		if ok {

			errMsg := strings.Join(errMsgs, "\n")
			// 存在errMsgs就将其存入Redis
			key := "ImportInsts error_msg" // 为每个errMsg生成key,每个import不同，这里要额外加标识吗
			ttl := 120 * time.Second       // 设置TTL为120秒

			redisErr := s.CacheCli.Set(context.Background(), key, errMsg, ttl).Err()
			if redisErr != nil {
				// 处理Redis存储错误，可选择记录日志
				c.JSON(http.StatusInternalServerError, gin.H{"error": fmt.Sprintf("Failed to store errmsg in Redis: %s", errMsg)})
				return
			}
		}

	}
}
```



错误内容整理：

![image-20231011163014432](C:\Users\Jackie\AppData\Roaming\Typora\typora-user-images\image-20231011163014432.png)

响应体包含了success和err的内容

其中err的内容是由  xlsx字段 +  错误信息 + 行数

代码中体现为：errMsg = append(errMsg, defLang.Languagef("web_excel_row_handle_error", fieldName, (rowIndex+1)))    

前部分  



10.12

剩下内容：

根据错误内容进行区分，获取对应的行和列  操作其field的单元格style 

导入真实环境进行测试。



今天开会确定了错误内容处理模式，下面进行代码实现  然后下周可能就是校对代码 上线



错误内容在响应请求中为："update_failed": [
                {
                    "bk_property_id": "server_room_6",
                    "info": "禁止修改字段: bk_template_id",
                    "row": 4
                },
                {
                    "bk_property_id": "server_room_12",
                    "info": "禁止修改字段: bk_template_id",
                    "row": 8
                }

