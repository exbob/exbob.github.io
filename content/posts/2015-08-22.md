---
title: 获得 IP 所在地的网站 freegeoip.net
date: 2015-08-22T08:00:00+08:00
draft: false
toc:
comments: true
---


## About

<http://freegeoip.net> provides a public HTTP API for software developers to search the geolocation of IP addresses. It uses a database of IP addresses that are associated to cities along with other relevant information like time zone, latitude and longitude.

You're allowed up to 10,000 queries per hour by default. Once this limit is reached, all of your requests will result in HTTP 403, forbidden, until your quota is cleared.

The freegeoip web server is free and open source so if the public service limit is a problem for you, download it and run your own instance.

## API

The HTTP API takes GET requests in the following schema:

    freegeoip.net/{format}/{IP_or_hostname}

Supported formats are: csv, xml, json and jsonp. If no IP or hostname is provided, then your own IP is looked up.

## Examples

* CSV

        freegeoip.net/csv/8.8.8.8
    
* XML

        freegeoip.net/xml/4.2.2.2

* JSON

        freegeoip.net/json/github.com
        
要获取本机的公网 IP 可以用 <http://httpbin.org> 提供的服务。在 Linux 中只需 `curl http://httpbin.org/ip` 即可返回本机的公网 IP         ：

    ~$ curl http://httpbin.org/ip
    {
        "origin": "218.18.232.122"
    }

其他类似功能的网站还有：

* <http://ip-api.com/>
* <https://ipstack.com/>
