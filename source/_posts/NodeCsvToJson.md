---
title: "以 Node 之名, CSV 转 JSON"
date: 2017-05-07 23:09:52
categories: "技术" 
tags: 
  - node
  - csv 
type: "tags"
---

## 简述需求:
有时，我们会在项目中遇到导入文档数据的功能需求，具体的比如可能会需要将 excel 表格数据导入到项目系统的数据库中，通过读取文件内容，解析并转换数据，最终存储至数据库的表中；

<!--more-->

## 可行方案：
针对于上述的这一系列较为繁琐的处理过程，在 Node 开发中，NPM 提供了 [csv to json](https://github.com/Keyang/node-csvtojson) 的工具模块，它能够将 CSV 标准文件转换成 JSON 数据，从而方便程序进一步的数据存储；

具体的思路：
1. 需要将 Excel 表格文件转换为 CSV 标准文件；
2. 使用 csvtojson 包对 CSV 标准文件进行 JSON 化数据转换；
3. 将 JSON 数据存入数据库对应表当中；

> 注: 使用 Mac 版 MicroOffice 将 EXCEL 表格文件转为 CSV  格式文件后，在终端中打开 CSV 文件中文出现乱码?

也不知为何，微软在做 CSV 文件转换时，对字符集支持有限，导致中文乱码；该问题使用 [Libre Office](https://www.libreoffice.org/) 则可以解决，Libreoffice 在转换 CSV 时提供字符编码转换设置，转换时设置 UTF-8 即可, 据说使用 [Google Sheets](https://www.google.com/sheets/about/) 可以达到相同效果；

## 操作说明
安装：
`npm install csvtojson --save `

实例：
```js
/** csv file
 account,password,company,username,phone
 test1@demo.com,123456,安徽公司,小明,13112310168
 test2@demo.com,123456,浙江公司,小伟,13327223199
**/

const csvFilePath='<path to csv file>';
const csv=require('csvtojson');
let userData = [];
csv()
.fromFile(csvFilePath)
.on('json',(jsonObj)=>{
	// 结合 CSV 头部属性逐行将内容转换为 JSON 对象
	userData.push(jsonObj);
})
.on('done',(error)=>{
	console.log('result: ' + JSON.stringify(userData));
	/**
	[
    	{
        	account:'test1@demo.com',
        	password:'123456',
        	company:'安徽公司',
        	username:'小明',
        	phone: '13112310168'
    	},
    	{
        	account:'test2@demo.com',
        	password:'123456',
        	company:'浙江公司',
        	username:'小伟',
        	phone: '13327223199'
    	}
	]
	**/
})
```
