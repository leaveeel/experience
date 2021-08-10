<kbd> 2021年6月10日 </kbd>  
  
# 前言  
  
> 重新工作也有一年，写vue也快一年，有些重复使用的特性/属性/代码/逻辑却不能立刻想好，顾记录工作时自认为有用的段落/结构/方法等。  
  
## 创建  
  
一般来说使用初始/配置package.json和vue.config.js文件，在目录内执行`npm install`。ui框架方面移动端使用 <kbd>vant</kbd> ，pc端使用 <kbd>elementui</kbd> 。  
  
### 结构  
  
__项目结构不尽相同，整体上应该差不多。__  
  
```
├── node_modules             //依赖文件夹，由npm下载。如果没有修改只要存在本地，不用上传服务器
├── public
│   ├── index.html          //入口文件，可以在这里引用js或初始化css
│   └── server.js           //用来配置服务器域名，方便修改生产或开发环境地址
├── src
│   ├── api
│   │   └── index.js       //全局配置接口地址
│   ├── assets              //资源文件夹
│   ├── components
│   │   ├── request.js     //封装axios
│   │   ├── index.js       //公共js
│   │   └── template       //存放各组件文件（夹）
│   │        └── ...
│   ├── router
│   ├── store
│   ├── views               //项目文件目录
│   ├── App.vue             //vue入口文件，在这里处理路由缓存、储存全局参数等
│   └── main.js             //初始化文件，全局依赖引用、配置
├── package.json             //包管理和配置
├── package-lock.json
└── vue.config.js            //vue配置文件
```
  
### 解析
  
- public
    - index.html
        - 基本的html文件，在这引用 <kbd>server.js</kbd> 给api使用，`<%= BASE_URL %>`为资源目录，由vue.config.js配置，默认为`/`。
        - 引用css或者在`<style />`里初始化样式
        - 移动端设置rem单位，可引用js或在`<script />`标签设置
    - server.js
        - 配置window全局变量，用来修改服务器地址或者公共配置
  
### 注意项
  
1. 通过地址访问的vue项目无法用`$route.query`获取参数，需要在 <kbd>App.vue</kbd> 或对应文件中截取url的方式获取。
2. 在 <kbd>App.vue</kbd> 中使用`<keepAlive />`缓存页面，可通过路由`meta`属性设置是否缓存。
3. `<style />`上添加`scoped`属性可以使样式不会影响到其他组件
4. <kbd>elementui</kbd> 中可使用插槽的方式自定义表格。通过`slot-scope`属性获取行数据、下标等信息
```
<el-table-column>
    <template slot-scope="scope">
        {{scope.row}}
        {{scope.$index}}
    </template>
</el-table-column>
```
5. 因为<kbd>echarts</kbd>插件加载顺序问题，会出现找不到dom的异常`Initialize failed: invalid dom`，可通过`Promise`处理，或给调用图形的方法加timeout
```
let newPromise = new Promise((resolve) => {
    resolve()
})
newPromise.then(() => {
    let chart = echarts.init(document.getElementById('chart'))
    chart.setOption({ ... })
})
```
6. 请求接口根据条件传递参数时将不需要传的字段设为`undefined`后端就不会接收到该字段
7. `new Date().setDate()`修改时间得到的是时间戳，需要在包一层`new Date()`才能调用Date的原型方法
8. 文件通常以`FormData`格式上传
```
let formData = new FormData()
formData.append("file", file)
```
9. 本地文件使用`URL.createObjectURL(file)`转换成url格式
10. iOS日期格式为`yyyy/MM/dd`，使用`new Date(date)`时需要把通用的`-`替换成`/`，`${date}.replace(/-/g, "/")`
11. 使用`let {key, ...newObj} = obj`获得不包括`obj.key`的新对象`newObj`和值为`obj[key]`的变量`key`
```
let o1 = {k1: 1, k2: 2}
let {k1, ...o2} = o1
//o1 = {k1: 1, k2: 2}
//o2 = {k2: 2}
//k1 = 1
```
12. `this.$forceUpdate()`刷新当前页面，通常重新请求接口刷新数据
13. `css`的`var()`属性可以动态设置样式，<kbd>vue</kbd>中给节点添加相应的样式值后在`css`中调用。
```
<template>
    <div :style="{'--width': width}"></div>
</template>
<script>
export default {
    name: "App",
    data() {
        return {
            width: "100px"
        }
    },
}
</script>
<style lang="scss">
div {
    width: var(--width);
}
</style>
```


### 常用方法
  
1. 截取url参数
  
```javascript
export function getUrlParam(param) {
    let pageURL = window.location.href.split('?')[1],
        value = ''
    if(pageURL) {
        let urlParamArray = pageURL.split('&')
        urlParamArray.map(item => {
            let paramName = item.split('=')
            value = paramName[0] === param ? paramName[1] : value
        })
        return value
    }
}
```
  
2. 日期时间格式化
  
> 在 <kbd>main.js</kbd> 中给`Date`原型对象添加方法`Format`，使用`new Date().Format('yyyy-MM-dd')`格式化日期为`yyyy-MM-dd`格式，其他同理
  
```javascript
Date.prototype.Format = function (fmt) {
    let o = {
        "M+": this.getMonth() + 1,
        "d+": this.getDate(),
        "h+": this.getHours(),
        "m+": this.getMinutes(),
        "s+": this.getSeconds(),
        "q+": Math.floor((this.getMonth() + 3) / 3),
        "S": this.getMilliseconds()
    }
    if(/(y+)/.test(fmt)) {
        fmt = fmt.replace(RegExp.$1, String(this.getFullYear()).substr(4 - RegExp.$1.length))
    }
    for (let k in o) {
        if (new RegExp(`(${k})`).test(fmt)) {
            fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (String(o[k]).padStart(RegExp.$1.length, 0)))
        }
    }
    return fmt
}
```
  
3. 根据对象数组中某个字段排序
  
> `arr.sort(sortRule('key'[, lang = String('zh'|''), true|false]))`，通过`lang = zh`对中文排序，倒序设置`des = true`
  
```javascript
function sortRule(key, lang, des) {
    return function(m, n) {
        if(lang == 'zh') {
            return des ? m[key].localeCompare(n[key]) * -1 : m[key].localeCompare(n[key])
        }
        let a = Number(m[key]) ? Number(m[key]) : (m[key] === 0 || m[key] === '0' ? 0 : des ? -9999999999 : 9999999999)
        let b = Number(n[key]) ? Number(n[key]) : (n[key] === 0 || n[key] === '0' ? 0 : des ? -9999999999 : 9999999999)
        if(!isNaN(a) && !isNaN(b)) {
            return des ? (b - a) : (a - b)
        }else {
            if (m[key] < n[key] ) {
                return -1
            }
            if (m[key] > n[key] ) {
                return 1
            }
            return 0
        }
    }
}
```
  
4. 递归复制
  
```javascript
export function deepCopy(obj) {
    var result = Array.isArray(obj) ? [] : {}
    for(let key in obj) {
        if(obj.hasOwnProperty(key)) {
            if(typeof obj[key] === 'object') {
                result[key] = deepCopy(obj[key])
            }else {
                result[key] = obj[key]
            }
        }
    }
    return result
}
```
  
5. 用户环境判断
  
```javascript
export function userAgent(e) {
    let userAgentInfo = navigator.userAgent,
        Agents = ["Android", "iPhone", "SymbianOS", "Windows Phone", "iPad", "iPod"],
        agent = Number
    Agents.map(item => {
        if(userAgentInfo.indexOf(item) !== -1) {
            //移动
            agent = 1
        }else {
            //pc
            agent = agent === 1 ? agent : 2
        }
    })
    let ua = userAgentInfo.toLowerCase()
    if(ua.indexOf('micromessenger') !== -1) {
        e.miniProgram.getEnv((res) => {
            if (res.miniprogram) {
                //小程序
                agent = 4
            }else {
                //微信
                agent = 3
            }
        })
    }
    return agent
}
```
  
6. 安卓ios判断

```javascript
export function userPhone() {
    let u = navigator.userAgent,
        isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1, //android
        isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) //ios
    let agent = {
        isAndroid: isAndroid,
        isiOS: isiOS
    }
    return agent
}
```

7. 下载文件流
```javascript
const link = document.createElement('a')
let blob = new Blob([data], { type: 'application/vnd.ms-excel' })
link.style.display = 'none'
link.href = URL.createObjectURL(blob)
link.setAttribute('download', '校友录.xlsx')
document.body.appendChild(link)
link.click()
document.body.removeChild(link)
window.URL.revokeObjectURL(link.href)
```

8. 动态规划
> 由接口获取多级菜单列表，通过瀑布流展示使高度差最小。
> 问题分解为：将一个包含`m`个元素的数组分成`n`个数组，使每个数组的子元素数量尽量接近
> this.menu是获取到的数据，结构为：[{..., servers: [{..., menus: [{}, ...]}]}, ...]


```javascript
const that = this
this.menu.map((item, ind) => {
    //创建新数组，如果直接用menuLine[0][0]会报undefined错
    that.menuLine[ind] = []

    let arr = item.servers
    //储存每个menus数组内元素的数量和对应index，方便计算高度
    let numList = []

    //储存每列数据
    let sortList = []

    //判断servers数量是否大于列数，如果小于则平铺
    if(arr.length > that.lineLen) {
        that.menuLine[ind] = new Array(that.lineLen)
        arr.map((menu, index) => {
            //因为要展示父级的分类名，所以每个len加1
            numList.push({len: menu.menus.length + 1, index})

        })
        //根据len值倒序排序，sortRule方法
        numList = numList.sort(that.sortRule('len', '', true))

        //计算总数和平均数，lineLen为列数
        let sum = numList.reduce((p, n) => p + n.len, 0)
        let avg = sum / that.lineLen

        //根据列数循环
        for(let a = 0; a < that.lineLen; a++) {

            //新建a列
            sortList[a] = []

            //平均数和当前列已存数量的差值
            let delta = 0

            //最后一列加入剩下的全部元素
            if(a == 5) {
                sortList[a] = numList
            }

            //储存暂时存放的元素index
            let setInd = []

            for(let num in numList) {
                if(numList[num].len >= avg) {
                    //数量大于平均数将该元素设为a列，删除该元素，重新计算总数和剩下列数的平均数。跳出numList循环，执行下一个lineLen循环
                    sortList[a].push(numList[num])
                    numList.splice(num, 1)
                    sum = numList.reduce((p, n) => p + n.len, 0)
                    avg = sum / (that.lineLen - a - 1)
                    break

                }else {
                    if(sortList[a].length == 0) {
                        //对应列没有数据将该元素放入，计算和平均数的差值。小于1删除该元素跳出循环，否则将当前index存入暂存器
                        sortList[a].push(numList[num])
                        delta = Math.abs(avg - numList[num].len)
                        if(delta < 1) {
                            numList.splice(num, 1)
                            break
                        }
                        setInd.push(num)

                    }else {
                        if(num == numList.length - 1 || delta == numList[num].len) {
                            //如果是最后一个元素或者数量和差值相等，将其添加进a列，将index存入暂存器并倒序暂存器，从numList中遍历删除暂存器内元素，跳出循环
                            sortList[a].push(numList[num])
                            setInd.push(num)
                            setInd.sort((a, b) => b - a)
                            setInd.map(item => numList.splice(item, 1))
                            break

                        }else {
                            //为方便比较，这里设置了如果数量大于差值，或者暂存器内最后一个index为上一个index，则执行下一个循环
                            if(numList[num].len > delta || setInd[setInd.length - 1] == +num - 1) continue

                            //对差值和数量进行比较，存入绝对值较小部分，取当前元素len值和旧差值的差为新差值，如果差值小于1则删除numList中对应元素，并跳出循环
                            if(Math.abs(delta - numList[num].len) > Math.abs(numList[+num - 1].len - delta)) {
                                sortList[a].push(numList[+num - 1])
                                setInd.push(+num - 1)
                                delta = Math.abs(numList[+num - 1].len - delta)
                            }else {
                                sortList[a].push(numList[num])
                                setInd.push(num)
                                delta = Math.abs(delta - numList[num].len)
                            }
                            if(delta < 1) {
                                setInd.sort((a, b) => b - a)
                                setInd.map(item => numList.splice(item, 1))
                                break
                            }

                        }
                    }
                }
            }
        }
        // 取出列数组中的index为新数组，结构为[[0],[1,2],[3,4,5], ...]
        let indexArr = sortList.map(m => m.map(n => n.index))

        indexArr.map((m, i) => {
            //创建menuLine[ind][i]，原因同上
            that.menuLine[ind][i] = new Array()

            m.map((n, x) => {
                that.menuLine[ind][i].push(arr[n])
            })
        })
    }else {
        for(let i = 0; i < arr.length; i++) {
            that.menuLine[ind][i] = [arr[i]]
        }
    } 
})
```
