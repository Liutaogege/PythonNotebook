# Vue纯前端实现导出word文档

要实现前端纯js导出word文档，需要用到docxtemplater，jszip-utils，file-saver三个组件。

## 一、用到的组件

1. docxtemplater
   
   docxtemplater是一种邮件合并工具，它以编程方式使用，处理条件、循环，并且可以扩展为表格、HTML、图像等。
   
   参考链接：
   
   https://docxtemplater.readthedocs.io/en/latest/index.html
   
   ```js
   var zip = new PizZip(content);  //根据word模板内容来实例一个zip对象
   
   new window.docxtemplater().loadZip(zip);    //创建docxtemplater实例对象，返回一个新的docxtemplater对象,并加载zip实例
   
   // docxtemplater对象 注意：必须从jszip的2.x版本向该方法传递一个zip实例
   
   setData(Tags)  //  设置模板变量的值
   
   render()     // 此函数用模板变量的值替换所有模板变量
   
   getZip()     // 此函数返回代表docxtemplater对象的zip
   ```

2. jszip-utils是与jszip一起使用的跨浏览器的工具库
   
   参考链接：
   
   https://stuk.github.io/jszip-utils/

3. jszip
   
   jszip是一个用于创建、读取和编辑.zip文件的JavaScript库，且API的使用也很简单
   
   https://stuk.github.io/jszip/

4. FileSaver
   
   FileSaver.js 是在客户端保存文件的解决方案，非常适合需要生成文件，或者保存不应该发送到外部服务器的敏感信息的应用。
   
   https://www.cnblogs.com/yunser/p/7629399.html  
   https://www.npmjs.com/package/file-saver

## 二、组件安装与引入

```shell
– 安装 docxtemplater
npm install docxtemplater pizzip --save

– 安装 jszip-utils
npm install jszip-utils --save

– 安装 jszip
npm install jszip --save

– 安装 FileSaver
npm install file-saver --save


import docxtemplater from ‘docxtemplater’
import PizZip from ‘pizzip’
import JSZipUtils from ‘jszip-utils’
import {saveAs} from ‘file-saver’
```

## 三、构建模板文件

![](/Users/jared/Library/Application%20Support/marktext/images/2021-12-17-10-00-17-image.png)

word模板语法详见：

[Docxtemplater &mdash; docxtemplater documentation](https://docxtemplater.readthedocs.io/en/stable/)



## 三、完整代码

```js
// 获取数据
mounted() {
            this.$nextTick(() => {
                if (this.$root.$widget) {
                    this.data.s_year = this.$root.$widget.options.context.s_year;
                    this.data.s_month = this.$root.$widget.options.context.s_month;
                    this.data.s_day = this.$root.$widget.options.context.s_day;
                    this.data.station = this.$root.$widget.options.context.station;
                    this.data.reporter = this.$root.$widget.options.context.reporter;
                    this.data.d_rec = this.$root.$widget.options.context.d_rec;
                    this.data.p_principal = this.$root.$widget.options.context.p_principal;
                    this.data.group_worker = this.$root.$widget.options.context.group_worker;
                    this.data.security_measures = this.$root.$widget.options.context.security_measures;
                    this.data.cal_ticket_num = this.data.display_name;
                    this.data.cal_safety_code = this.safe_key[this.data.p_principal_safety_code];
                    let data_lst = [];
                    let index = 0;
                    let item_dic = {};

            });
        },


methods: {
            // 导出word
            loadFile(url, callback) {
                PizZipUtils.getBinaryContent(url, callback);
            },
            generate() {
                var that = this;
                this.loadFile("/construction_dispatch/static/变电所第二种工作票.docx", function (error, content) {
                if (error) {
                    throw error
                }
                var zip = new PizZip(content);
                var doc = new window.docxtemplater().loadZip(zip)
                doc.setData({
                    ...that.data
                });
                try {
                    // render the document (replace all occurences of {first_name} by John, {last_name} by Doe, ...)
                    doc.render()
                } catch (error) {
                    var e = {
                    message: error.message,
                    name: error.name,
                    stack: error.stack,
                    properties: error.properties,
                    }
                    console.log(JSON.stringify({
                        error: e
                    }));
                    // The error thrown here contains additional information when logged with JSON.stringify (it contains a property object).
                    throw error;
                }
                var out = doc.getZip().generate({
                    type: "blob",
                    mimeType: "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
                }) //Output the document using Data-URI
                saveAs(out, "变电所第二种工作票.docx")
                })
            },
            //自己加的方法，将换行符替换
            generate_lst(str) {
                if (!str) return [{"name": " "}];
                let lst = str.split("\n");
                let result_lst = [];
                for (let ele of lst) {
                    let result_dict = {"name": ele};
                    result_lst.push(result_dict);
                }
                return result_lst
            }
        },
```


