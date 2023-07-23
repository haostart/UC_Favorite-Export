# UC_Favorite-Export
UC浏览器书签导出备份为HTML文件,并导入到其他浏览器的方法,python实现.UC_Favorite-Export





---

# 前言
>  由于历史惯性等原因，**手机上**一直使用的UC浏览器，用了这么多年。
>  最近觉得Via浏览器比较简洁方便，所以准备切换过去，那我最宝贵的书签肯定要迁移过去的，
>  但万恶的资本家不想你这么做，你要走了我的钱怎么办？
>  所以UC浏览器离谱的没有书签导出功能（以前有），气死我了，正好自己会Python，于是就有了这篇文章。




# 一、迁移方案

 1. 通过**UC云**的API获取到所有书签。
 2. 将书签保存为HTML格式的导出文件，方便其他浏览器导入。
 ```mermaid
flowchart TB
  登陆UC云获取到token和cookies  -->通过UC云的API获取到所有书签--> 将书签保存为HTML格式的导出文件方便其他浏览器导入
```
最后要变成的书签HTML文件格式如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/30a2aca8aac44c7fb680b2a8dec504a9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/213579d8ff2649b1b1dcd6428b5d227e.png)

# 二、相关代码及过程

首先进入书签官网,登陆进去:
https://cloud.uc.cn/home/phone
[UC书签官网](https://cloud.uc.cn/home/phone)
然后浏览器按F12进入开发者模式,按图点
![在这里插入图片描述](https://img-blog.csdnimg.cn/e8ebe76fbd3f4c5889316d209ab08a00.png)
然后刷新书签页面
![在这里插入图片描述](https://img-blog.csdnimg.cn/06e9a0f8929f43e291823a401bac0765.png)
随便选一个listdata,然后找到cookie和 token,将它们的值放到代码相应位置.
![在这里插入图片描述](https://img-blog.csdnimg.cn/5bb9daf2fdf24289aedccf8b24504a5e.png)




## ***运行代码前记得换 cookie和 x-csrf-token,自己浏览器F12开发者工具找***

![在这里插入图片描述](https://img-blog.csdnimg.cn/5bb9daf2fdf24289aedccf8b24504a5e.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/0b64e92d37124a44a939dd4db754492f.png)

***记得换 下面代码的cookie和 x-csrf-token***
***记得换 cookie和 x-csrf-token***
***记得换 cookie和 x-csrf-token***
```python
import json,requests,time

host = f'https://cloud.uc.cn/api/bookmark/listdata'
headers = {
    'Accept': 'application/json, text/plain, */*',
    'Accept-Encoding': 'gzip, deflate, br',
    'Accept-Language': 'zh-Hans-CN, zh-Hans; q=0.5',
    'Connection': 'Keep-Alive',
    'origin': 'https://cloud.uc.cn',
    'referer': 'https://cloud.uc.cn/home/phone',
    'sec-fetch-dest': 'empty',
    'sec-fetch-mode': 'cors',
    'sec-fetch-site': 'same-origin',
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36',
    'x-csrf-token': 'okpv0j1zpChodmXpmlilE9X5',	#这里换上自己的token，登录后在浏览器开发者工具里找。
    'cookie': r"123"    #这里换上自己的cookie，登录UC云后在浏览器开发者工具里找。
}
session = requests.session()

count=0

def get_text(guid):
    global text
    global count
    for i in range(1,200):	#假设最多加载200页,有200多页书签的话改这里
        time.sleep(0.1)
        post_data={'cur_page': i, 'type': "phone", 'dir_guid': str(guid)}
        response = session.post(host, headers=headers,data=post_data).json()
        print(f'dir_guid: {guid}  cur_page:{i} ')
        if response['msg']!='ok':
            print(f"i:{i},quit{response['msg']}")
            break
        try:
            data=response['data']['list']
            print(f'i:{i}')
        except:
            print('error')
            continue
        for book in data:
            if book['is_directory']==1:	#如果是书签目录，则递归调用，相当于DFS
                print('it is a directory')
                bk=f"\t<DT><H3>{book['name']}</H3>\n\t<DL><p>\n"
                text+=bk
                get_text(book['guid'])	#这里是递归遍历目录
                text+="\t</DL><p>\n"
            else:
                bk=f"\t<DT><A HREF=\"{book['origin_url']}\">{book['title']}</A>\n"
                count=count+1
                text+=bk
        
        if not response['data']['meta']['has_last_page']:
            print(f"本页结束")
            break

with open('bookmark.html',mode='w',encoding='utf-8') as f:
    text='''<!DOCTYPE NETSCAPE-Bookmark-file-1>
			<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">
			<TITLE>Bookmarks</TITLE>
			<H1>Bookmarks</H1>
			<DL><p>
			'''
    f.write(text)
    text=""
    get_text('0')
    f.write(text+'</DL><p>')
    print(f'生成书签总数 {count}')
```

**书签文件保存在同目录的 bookmark.html**
现在可以导入到其他浏览器了.
#  三、导入其他浏览器
![在这里插入图片描述](https://img-blog.csdnimg.cn/56a70b7e315f4e5390b13d4c2aea331a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3237bd42a73449ce9ff4b2921d076d04.png)


# 总结
幸苦了一下午，完成了一件有实际意义的事，舒服了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/922816921cf4402991862559a439c1fe.png)
