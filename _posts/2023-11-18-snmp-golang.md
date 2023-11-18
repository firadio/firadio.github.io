---
layout: post
title: "这是一个用于读取SNMP的golang代码"
categories: SNMP
author:
- Xiang Xisheng
---

# 这是一个用于读取SNMP的golang代码

## 首先创建一个空白的文件夹，然后执行下列命令
```
go mod init snmp-go
go get github.com/gosnmp/gosnmp
```

## 将下列代码保存为main.go文件
```
package main

import (
        "fmt"
        "log"
        "github.com/gosnmp/gosnmp" 
)

func main() {
        target := "172.20.0.3"
        community := "public"
        oid := "1.3.6.1.2.1.31.1.1.1.1"

        gosnmp.Default.Target = target
        gosnmp.Default.Community = community
        gosnmp.Default.Version = gosnmp.Version2c

        err := gosnmp.Default.Connect()
        if err != nil {
                log.Fatal("Connect error: ", err)
        }
        defer gosnmp.Default.Conn.Close()

        err = walk(oid)
        if err != nil {
                log.Fatal("Walk error: ", err)
        }
}

func walk(oid string) error {
        result, err := gosnmp.Default.BulkWalkAll(oid)
        if err != nil {
                return fmt.Errorf("Walk error: %v", err)
        }

        for _, variable := range result {
                printVariable(variable)
        }

        return nil
}

func printVariable(variable gosnmp.SnmpPDU) {
        fmt.Printf("%s = ", variable.Name)

        switch variable.Type {
        case gosnmp.OctetString:
                fmt.Printf("%s\n", string(variable.Value.([]byte)))
        default:
                fmt.Printf("%v\n", variable.Value)
        }
}
```
