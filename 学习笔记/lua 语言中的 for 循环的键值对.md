#### lua 语言中的 for 循环的键值对



```lua
-- 以http 为例， 

local function cbFnc(result,prompt,head,body)  -- http 的回调函数，这里的body
    log.info("testHttp.cbFnc",result,prompt)
    if result and head then
        for k,v in pairs(head) do
            log.info("testHttp.cbFnc",k..": "..v)
        end
    end
    if result and body then
        log.info("testHttp.cbFnc","bodyLen="..body:len())
    end
end
```

- 上方代码被执行的时候得到的结果是

  ![image-20230406100023809](https://raw.githubusercontent.com/MR-liao-955/Note/main/img/202306161726394.png?token=AO67QCRZJTGA6UJLVZ4F3EDERQVPA)



> Lua 中的for 循环分为两种
>
> 1. 数值循环，就是普通的循环
>
> 2. 泛型循环。
>
>    ![image-20230406100258480](https://raw.githubusercontent.com/MR-liao-955/Note/main/img/202306161726395.png?token=AO67QCTXAQIPX3HTTBF2MJTERQVPA)



- 说到泛型，那就不困了，这里的循环

  他会自动适应 table表 中的每个元素并进行遍历

  - pairs 则是table表 中的所有元素都会进行遍历， 且table 表的key 可以是任意类型
  - inpairs 中table表 的key 只能是整形，



> 测试结果如图，pairs 和 ipairs 的区别

![image-20230406102710934](https://raw.githubusercontent.com/MR-liao-955/Note/main/img/202306161726396.png?token=AO67QCVZHOPHPM7G6ZKDZFLERQVPE)

当 ipairs( )  中的table 遇到 nil 类型的时候他就会停止for循环遍历

而  pairs( )  并不会出现这个问题





> 关于pairs 的索引问题， ipairs 索引得连续

![image-20230406103353437](https://raw.githubusercontent.com/MR-liao-955/Note/main/img/202306161726397.png?token=AO67QCQMVJ3IXDQQ7O47FQTERQVPG)

![image-20230406103615067](https://raw.githubusercontent.com/MR-liao-955/Note/main/img/202306161726398.png?token=AO67QCVEF4KFXQ5DIHH7V43ERQVPK)

如上图，不言而喻





























