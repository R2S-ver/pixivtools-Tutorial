# pixivtools

爬取pixiv网站的图片，支持多种爬取模式

## 支持爬取功能

* 按照artwork id下载图片
* 按照画师id(userid)下载图片
* 按照pixivision id下载图片(pixvision站)
* 按照关注的画师最新上传下载图片
* 按照首页推荐的作品下载图片
* 按照排行榜下载图片
* 按照接稿的推荐作品下载图片
* 按照用户的收藏下载图片
* 按照指定标签的热门作品下载图片
* 按照指定的artwork id的相似作品下载图片
* 按照指定的画师id的所有相似画师下载图片
* 按照平台推荐的画师下载图片
* 按照接稿的最新接稿画师下载图片

## 使用方法
### 1、部署运行环境(已经有可以跳过)
- 下载[Git Bash](https://git-scm.com/install/)(选择操作版本)
- 下载[Python](https://www.python.org/downloads/)(选择下载版本，适合自己的操作系统，然后运行安装程序.msix文件)
如果后面的操作不成功，报错无法找到python，就问deepseek检查一下是不是path没配置
### 2、安装pixivtools(会存在系统盘Python的文件夹里）
```shell
pip install pixivtools --upgrade
```

### 3、创建配置对象

这里有两种办法可以完成配置的设置，可根据喜好选择

#### 1) 通过构造器创建(更自由的配置)
```python
import pixivtools

cfg_maker = pixivtools.pixiv_config_maker()
# 可根据自己需要做调整
cfg_maker.set_phpsessid("输入pixiv的PHPSESSID")
cfg_maker.set_proxy("127.0.0.1:7890")
cfg_maker.set_img_dir("./imgs")             # 图片存放的位置
cfg_maker.set_log_file("./out.log")         # 日志打印位置
cfg_maker.set_sql_url("sqlite:///pixiv.db") # sqlalchemy格式的链接，如不理解可以无视
cfg = cfg_maker()
```

#### 2) 通过配置文件
创建一个`config.yaml`文件，配置示例参见仓库的`config.yaml.example`
在config.yaml里面有phpsessid(你的P站ID）；获取方式为：登录Pixiv，在首页按F12，选择`网络`，按Ctrl+F搜索`phpsessid`，寻找phpsessid后面跟着一大串代码直到;符号中止，这个就是phpsessid，找不到问ai。
然后 `cfg = pixivtools.load_pixiv_config("config.yaml")`


### 4. 开始操作
```python
# 将上一步得到的cfg对象，传到这里
service = pixivtools.new_pixiv_service(cfg)
# 获取爬虫实例
crawler = service.crawler()
# 不同爬取模式的示例
crawler.get_by_artwork_id(98538269)
crawler.get_by_user_id(23279364)
crawler.get_by_pixivision_aid(9374)
crawler.get_by_follow_latest(1)
crawler.get_by_recommend()
crawler.get_by_rank(pixivtools.RankType.MONTHLY, 20250313, 1)
crawler.get_by_request_recommend()
crawler.get_by_user_bookmark(92803629, 1)
crawler.get_by_tag_popular("ホロライブ")
crawler.get_by_similar_artwork(115812789)
crawler.get_by_similar_user(20015785)
crawler.get_by_recommend_user()
crawler.get_by_request_creator()

### 5. 爬取完成后，图片会保存到指定路径，数据库会记录所有相关的元信息，可根据自行需求食用

### 6. (可选)并行下载多个画师
下载的时候遇到下载一个画师时，下载速度为0.3MB/s,平均五秒钟一张
因此通过同时下载多个画师ID来达到并行下载，显著提升下载速度
个人用的方式:
#### 1) 在放config.yaml的文件夹下新建文件，名字改成Run.py
里面输入以下代码，记得修改为自己的SESSIED; 没有代理服务器时cfg_maker.set_proxy("")留空; 
```import pixivtools
import threading

# 你要下载的画师 ID 列表（无重复）
USER_IDS = [爬去画师的ID，ID为画师网址的尾部数字] #参考代码 USER_IDS = [67921731, 30370756, 105818043]

def download_user(uid):
    # 使用构造器创建独立的配置，完全不需要 YAML 文件
    cfg_maker = pixivtools.pixiv_config_maker()
    
    # 这里直接写你的登录信息和代理（和 config.yaml 里一样）
    cfg_maker.set_phpsessid("你的P站ID")   # 修改为真实值
    cfg_maker.set_proxy("")                      	       # 如果没有代理可注释掉这一行
    cfg_maker.set_img_dir(f"./imgs_{uid}")                     # 每个画师独立文件夹
    cfg_maker.set_sql_url(f"sqlite:///pixiv_{uid}.db")         # 独立数据库
    cfg_maker.set_log_file(f"./out_{uid}.log")                 # 独立日志（可选）
    
    cfg = cfg_maker()                                          # 生成配置对象
    
    service = pixivtools.new_pixiv_service(cfg)
    crawler = service.crawler()
    
    print(f"【开始】画师 {uid} → 图片保存至 imgs_{uid}/")
    try:
        crawler.get_by_user_id(uid)
        print(f"【完成】画师 {uid} 下载完毕")
    except Exception as e:
        print(f"【出错】画师 {uid} 下载失败: {e}")

# 多线程并行下载
threads = []
for uid in USER_IDS:
    t = threading.Thread(target=download_user, args=(uid,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()

print("全部任务结束！")
input("按回车键退出...")
```
