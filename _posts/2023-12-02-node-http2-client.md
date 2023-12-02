---
layout: post
title: "分享一个支持socks代理的HTTP2请求客户端Node.js模块，仿fetch"
categories: node.js
---

将下列代码存为http2_fetch.js
```
const tls = require('node:tls');
const http2 = require('node:http2');
const {SocksClient} = require('socks');

async function getSocketByProxy(_options) {
  // SOCKS 代理服务器的信息
  const proxyOptions = {
    proxy: {
      // 代理服务器的 IP 地址
      host: _options.proxy.hostname,
      // 代理服务器的端口号
      port: parseFloat(_options.proxy.port),
      // SOCKS 版本，通常为 5
      type: 5,
    },
    destination: {
      // 目标服务器的 IP 地址
      host: _options.hostname ? _options.hostname : _options.servername,
      // 目标服务器的端口号
      port: _options.port ? parseFloat(_options.port) : 443,
    },
    command: 'connect'
  };
  const infoSocksClient = await SocksClient.createConnection(proxyOptions);
  return infoSocksClient.socket;
}

module.exports = async function (_options) {
  const {
    HTTP2_HEADER_PATH,
    HTTP2_HEADER_STATUS,
  } = http2.constants;

  function getURLByHost(_host) {
    return new URL(`https://${_host}`);
  }

  if (_options.proxy) {
    _options.socket = await getSocketByProxy(_options);
  }
  const oPrivate = {};
  const oURL = getURLByHost(_options.servername);
  if (_options.port) {
    oURL.port = _options.port;
  }
  const sUrl = oURL.toString();

  const options_connect = {};
  if (_options.hostname) {
    options_connect.createConnection = (mUrl) => {
      const options = {};
      // 创建 SOCKS 代理客户端
      if (_options.socket) {
        options.socket = _options.socket;
      } else {
        options.host = _options.hostname ? _options.hostname : mUrl.hostname;
        options.port = mUrl.port ? parseFloat(mUrl.port) : 443;
      }
      options.servername = mUrl.hostname;
      options.ALPNProtocols = ['h2'];
      return tls.connect(options);
    }
  }
  oPrivate.client = http2.connect(sUrl, options_connect);

  const oPublic = {};
  oPublic.request = async function (path) {
    const options = {};
    options[HTTP2_HEADER_PATH] = path;
    return new Promise((fReqReslove, fReqReject) => {
      const req = oPrivate.client.request(options);
      req.on('response', (headers) => {
        const oResResult = {};
        oResResult.headers = JSON.parse(JSON.stringify(headers));
        delete oResResult.headers[HTTP2_HEADER_STATUS];
        oResResult.status = headers[HTTP2_HEADER_STATUS];
        oResResult.arrayBuffer = async () => {
          return new Promise((fResReslove, fResReject) => {
            const chunks = [];
            req.on('data', (chunk) => {
              chunks.push(chunk);
            });
            req.on('end', () => {
              fResReslove(Buffer.concat(chunks));
            });
          });
        };
        oResResult.text = async () => {
          return (await oResResult.arrayBuffer()).toString();
        };
        oResResult.json = async () => {
          return JSON.parse(await oResResult.text());
        };
        fReqReslove(oResResult);
      }); 
    });
  };
  return oPublic;
};
```

将下面代码保存为main.js
```
const http2_fetch = require('./http2_fetch');

async function start() {
  const options = {};
  options.servername = 'xiangxisheng.github.io';
  options.hostname = '185.199.108.153';
  options.port = 443;
  options.proxy = new URL('socks5://127.0.0.1:1080');
  const clientSession = await http2_fetch(options);
  if (true) {
    const res = await clientSession.request('/about');
    console.log(res.status, res.headers.date);
    await res.text();
  }
}

async function main() {
  if (true) {
    try {
      await start();
    } catch (ex) {
      console.log(ex);
    }
  }
  process.exit();
}
main();
```
