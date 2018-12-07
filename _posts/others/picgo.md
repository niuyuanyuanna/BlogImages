---
title: 七牛云图床迁移到github
date: 2018-12-06 23:14:28
tags:
- github
- 图床
- 乱七八糟
categories: others
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/computer_version.jpeg
---

# 七牛云图床迁移到github

前两天访问博客发现所有传到七牛云的图片都无法访问了。。。原来是临时域名被回收了，然鹅没有正经域名的我很方，备案好像很麻烦，于是，赶紧把七牛云Bucket的所有图片拉到本地。

操作环境：

- win10
- qshell

qshell是七牛云进行Bucket管理的工具，现在支持windows系统，在官网按照指引下载对应版本qshell

## 下载七牛云图床中的图片

### 安装qshell

电脑运行cmd，将目录切换到qshell解压路径下，如我的解压路径为：`E:\UnusualSoftwares\qshell-v2.3.4`切换到该目录后，运行qshell，这里需要把对应版本的`.exe`文件改名为`qshell.exe`然后再运行qshell，否则是没有安装成功的。

执行`qshell.exe`后得到下面信息

·```

```
E:\UnusualSoftwares\qshell-v2.3.4>qshell.exe
Qiniu commandline tool for managing your bucket and CDN

Usage:
  qshell [command]

Available Commands:
  account       Get/Set AccessKey and SecretKey
  alilistbucket List all the file in the bucket of aliyun oss by prefix
  b64decode     Base64 Decode, default nor url safe
  b64encode     Base64 Encode, default not url safe
  batchchgm     Batch change the mime type of files in bucket
  batchchtype   Batch change the file type of files in bucket
  batchcopy     Batch copy files from bucket to bucket
  batchdelete   Batch delete files in bucket
  batchexpire   Batch set the deleteAfterDays of the files in bucket
  batchfetch    Batch fetch remoteUrls and save them in qiniu Bucket
  batchmove     Batch move files from bucket to bucket
  batchrename   Batch rename files in the bucket
  batchsign     Batch create the private url from the public url list file
  batchstat     Batch stat files in bucket
  buckets       Get all buckets of the account
  cdnprefetch   Batch prefetch the urls in the url list file
  cdnrefresh    Batch refresh the cdn cache by the url list file
  chgm          Change the mime type of a file
  chtype        Change the file type of a file
  completion    generate autocompletion script for bash
  copy          Make a copy of a file and save in bucket
  d2ts          Create a timestamp in seconds using seconds to now
  delete        Delete a remote file in the bucket
  dircache      Cache the directory structure of a file path
  domains       Get all domains of the bucket
  expire        Set the deleteAfterDays of a file
  fetch         Fetch a remote resource by url and save in bucket
  fput          Form upload a local file
  get           Download a single file from bucket
  help          Help about any command
  ip            Query the ip information
  listbucket    List all the files in the bucket
  listbucket2   List all the files in the bucket using v2/list interface
  m3u8delete    Delete m3u8 playlist and the slices it references
  m3u8replace   Replace m3u8 domain in the playlist
  mirrorupdate  Fetch and update the file in bucket using mirror storage
  move          Move/Rename a file and save in bucket
  pfop          issue a request to process file in bucket
  prefop        Query the pfop status
  privateurl    Create private resource access url
  qdownload     Batch download files from the qiniu bucket
  qetag         Calculate the hash of local file using the algorithm of qiniu qetag
  qupload       Batch upload files to the qiniu bucket
  qupload2      Batch upload files to the qiniu bucket
  reqid         Decode qiniu reqid
  rpcdecode     rpcdecode of qiniu
  rpcencode     rpcencode of qiniu
  rput          Resumable upload a local file
  saveas        Create a resource access url with fop and saveas
  stat          Get the basic info of a remote file
  sync          Sync big file to qiniu bucket
  tms2d         Convert timestamp in milliseconds to a date (TZ: Local)
  tns2d         Convert timestamp in Nanoseconds to a date (TZ: Local)
  ts2d          Convert timestamp in seconds to a date (TZ: Local)
  unzip         Unzip the archive file created by the qiniu mkzip API
  urldecode     Url Decode
  urlencode     Url Encode
  user          Manage users
  version       show version

Flags:
  -C, --config string   config file (default is $HOME/.qshell.json)
  -d, --debug           debug mode
  -h, --help            help for qshell
  -L, --local           use current directory as config file path
  -v, --version         show version

Use "qshell [command] --help" for more information about a command.
```

### 下载所有图片

1. 在七牛云查找到自己的AK和SK，粘贴到命令：

```
qshell account ak sk
```

2. 在对象存储中新建一个Bucket，假设名字为NewBucket
3. 将原来Bucket中的所有图像复制到NewBucket

- 这里首先需要导出原Bucket中的图片list，使用命令：

```
qshell listbucket originBucket localFileName.txt
```

保存到本地的文件里包括空间中的文件名和文件的各种参数，需要使用编辑器打开（记事本打开会错行），然后把文件名以外的参数全部删掉，只保留文件名，一行一行的排列。

因为windows的cmd没有cat命令，那就写个python脚本把后面的删掉好了。
```python
def deal_with_file(file_in, file_out):
    with open(file_in, 'r') as fin:
        with open(file_out, 'w+') ad fout:
            for line in fin:
                line = line.split('\t')
                fout.write(line[0] + '\n')
```

把清理过后的list存成一个新的文件。

- 执行命令，将list中的file都复制到newBucket中：

```
qshell batchcopy originBucket newBucket cleanedLocalFileName.txt
```

- 在`qshell.exe`根目录新建`qshell.conf`文件

```
{
	    "dest_dir"  :   "./qiniu",	// 文件下载所保存的目录
	    "bucket"    :   "newBucket",	// 空间名
	    "domain"    :   "http://niuyuanyuanna.clouddn.com/",	// 空间域名
	    "access_key"    :"******",	 // ak
	    "secret_key"    :"******",	 // sk
	    "prefix"    :   "",
	    "suffix"    :   ""
	}
```

- 执行命令，将newBucket中的文件下载到本地：

```
qshell qdownload 10 qshell.conf	  	// `10` 为下载的并发协程数量
```

下载好的图像会以日期-文件名的形式存储，我下载好的如图所示：

<center>
   <img src="https://raw.githubusercontent.com/niuyuanyuanna/BlogImages/master/others/20181206235257.png" width=70%/>
</center>
接下来就是重头戏，使用github仓库作为图床啦。

## GitHub图床

在研友的介绍下，考虑了一波阿里云oos、腾讯云、微博图床还有免费的，综合考虑还是选择github，毕竟很稳呐，在此之前，下载了一个比较好用的图床管理应用，叫PicGo，js实现，贼强。可以直接支持github图床上传，但希望批量上传，并且之前的markdown里面的链接也可以直接替换，因此采用直接将图片上传到github仓库的方法。

首先需要新建一个仓库来存放图片，如我的仓库名为BlogImage。

下面就需要把刚刚下载好的图片对应到markdown中，需要进行正则匹配，查找到之前失效的url，然后替换掉，直接看python代码吧。

```python
def find_image_file(md_file):
    # 这个函数主要是将markdown中对应的url中的img移动到对应的目录下
    img_patten = r'!\[.*?\]\((.*?)\)|<img.*?src=[\'\"](.*?)[\'\"].*?>'
    if os.path.splitext(md_file)[1] != '.md':
        print('{}不是Markdown文件，不做处理。'.format(md_file))
        return
    cnt_replace = 0
    with open(md_file, 'r', encoding='utf-8') as f:
        post = f.read()
        matches = re.compile(img_patten).findall(post)
        if matches and len(matches) > 0:
            for match in list(chain(*matches)):
                if match and len(match) > 0:
                    match_l = match.split('/')
                    sub_dir_name = match_l[-2]
                    file_name = match_l[-1]
                    url_base = match_l[-3]
                    if url_base == 'p6um59a45.bkt.clouddn.com':
                        full_path = os.path.join('E:\\UnusualSoftwares\\qshell-v2.3.4\\blog_images', sub_dir_name, file_name)
                        dir_path = 'F:\\blog\\blog_imgs\\computerVersion\\'
                        output_name = os.path.join(dir_path, file_name)
                        if not os.path.exists(output_name):
                            if not os.path.exists(full_path):
                                print('path {} not exists'.format(full_path))
                            else:
                                shutil.move(full_path, dir_path)
                        cnt_replace = cnt_replace + 1
                        print('move {}'.format(full_path))
                    else:
                        print('{} is not in qiniuyun'.format(match))
        if post and cnt_replace > 0:
            print('{} done'.format(os.path.basename(md_file)))
        elif cnt_replace == 0:
            print('{}中没有需要替换的URL'.format(os.path.basename(md_file)))

def repalce_url(md_file):
    # 这个函数真正替换markdown中的url
    img_patten = r'!\[.*?\]\((.*?)\)|<img.*?src=[\'\"](.*?)[\'\"].*?>'
    if os.path.splitext(md_file)[1] != '.md':
        print('{}不是Markdown文件，不做处理。'.format(md_file))
        return
    cnt_replace = 0
    dir_ts = time.strftime('%Y-%m-%d-%H-%M-%S', time.localtime())
    with open(md_file, 'r', encoding='utf-8') as f:
        post = f.read()
        matches = re.compile(img_patten).findall(post)
        if matches and len(matches) > 0:
            for match in list(chain(*matches)):
                if match and len(match) > 0:

                    match_last_name = match.split('/')[-1]
                    new_url = 'https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/'
                    new_url = new_url + match_last_name
                    post = post.replace(match, new_url)
                    cnt_replace = cnt_replace + 1

        if post and cnt_replace > 0:
            open(md_file, 'w', encoding='utf-8').write(post)
            print('done')
        elif cnt_replace == 0:
            print('{}中没有需要替换的URL'.format(os.path.basename(md_file)))

```

因为上传到github仓库中的图片，其外链可以直接用`https://github.com/niuyuanyuanna/BlogImages/raw/master/computerVersion/filename`表示，即网页端打开仓库中的图片，将链接中的`blob`更改为`raw`就可以访问到。

all done！

再次吐槽！辣鸡七牛云!