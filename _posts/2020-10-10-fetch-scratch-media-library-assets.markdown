---
layout: post
title:  "本地获取Scratch3素材"
date:   2020-10-10 10:20:06 +0800
categories: jekyll update
---
由于国内政策原因，导致scratch3官方部分域名无法访问。受此影响开发时候需要使用官方的一些在线素材将无法获取。例如：角色，背景和声音等。

如果熟悉scratch3桌面版本开发的读者，处理这个比较得心应手，拿来就用。官方桌面版本自带了下载素材脚本，方便了在无网络情况下也可以使用scratch3。本文将讲解如何将scratch-desktop项目下的下载素材脚本移动至当前开发的项目中。可能有部分人有疑问为什么不直接官方的scratch-desktop的项目中直接运行此脚本呢？回答是可以的，但是需要注意需要将您当前项目的相关素材配置json文件覆盖到scratch-desktop中，否则导致部分素材不匹配无法显示。当然这样做也方便了后续再次更新资源。

具体操作步骤：

1. 迅速打开您的梯子（VPN/SS/SSR）

2. 打开或者克隆scratch-desktop（https://github.com/LLK/scratch-desktop.git）至本地

3. 参考scratch-desktop项目将下列文件拷贝至当前项目（我的是scratch-gui）

   > scratch-desktop/scripts/lib/libraries.js    -->  scratch-gui/scripts/lib/libraries.js
   > scratch-desktop/scripts/fetchMediaLibraryAssets.js   -->  scratch-gui/scripts/fetchMediaLibraryAssets.js

4. 拷贝scratch-desktop项目中文件package.json的**fetch**命令至scratch-gui项目中package.json文件中

   > "fetch": "rimraf ./static/assets/ && mkdirp ./static/assets/ && node ./scripts/fetchMediaLibraryAssets.js"

5. 更新相关文件中引用路径

6. 如果使用SSR等代理软件时需要添加依赖“https-proxy-agent”并修改fetchMediaLibraryAssets.js文件中相关配置。具体参考如下代码：
   
   ```javascript
   const fetchAsset = function (md5, callback) {
       // const myAgent = connectionPool.pop() || new https.Agent({ keepAlive: true });
       var proxy = process.env.http_proxy || 'http://127.0.0.1:1080';
       var agent = new HttpsProxyAgent(proxy);
       const getOptions = {
           host: ASSET_HOST,
           path: `/internalapi/asset/${md5}/get/`,
           // agent: myAgent
           agent: agent
       };
       const urlHuman = `//${getOptions.host}${getOptions.path}`;
       https.get(getOptions, response => {
           if (response.statusCode !== 200) {
               callback(new Error(`Request failed: status code ${response.statusCode} for ${urlHuman}`));
               return;
           }
           const stream = fs.createWriteStream(path.resolve(OUT_PATH, md5), { encoding: 'binary' });
           stream.on('error', callback);
           response.on('data', chunk => {
               stream.write(chunk);
           });
           response.on('end', () => {
               // connectionPool.push(myAgent);
               stream.end();
               console.log(`Fetched ${urlHuman}`);
               callback();
           });
       });
   };
```
   
   
   
7. 待所有资源下载完成后请根据需要修改项目中原官方的一些素材链接至新的地址即可。