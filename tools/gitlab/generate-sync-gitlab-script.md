# 利用python迁移gitlab中项目

## 思路

- 利用python 调用gitlab api 获取项目信息，创建信息，并同步项目
- 由于python 调用同步项目代码较多，可以通过python 生成同步的脚本

## 手工同步gitlab 的方式

``` bash
## 这种方式适合单向同步，git迁移；由于该种方式会完全替换目标的工程，不适合双向同步的
## 从源gitlab克隆镜像到本地
git clone --mirror git@gitlab1.com:group-name/project-name.git
## 进入克隆的镜像目录
cd project-name.git
## 将克隆的镜像推送至目标
git push --mirror git@gitlab2.com:group-name/project-name.git

```

## 利用python 同步项目

由于精力有限，工作较多，实现成了利用python 同步创建空工程，并生成上述手工脚本。

### 前置技能要求

- 了解Git工作方式，能理解上述手工同步的原理
- 了解gitlab，并了解gitlab api的原理机制
- 同步项目利用了python-gitlab这个库（v2.5.0,支持gitlab v4 版本api, gitlab 10.0 以上版本支持（具体翻gitlab官方文档））
- python 版本3.7.X 需要使用pip安装python-gitlab

### 代码

- main.py

``` python
import gitlab
import time
import os

def main():
    # 主要业务逻辑
    #print('begin')
    # 配置
    # 目标仓库
    tempDir = '/d/temp'
    whGitlab = gitlab.Gitlab.from_config('wh', 'python-gitlab.cfg')
    # 源仓库
    kfbGitlab = gitlab.Gitlab.from_config('kfb', 'python-gitlab.cfg')

    # 查找所有源仓库的分组
    kfbAllGroup = kfbGitlab.groups.list(all=True)
    os.chdir('D:/temp')
    for kfbGroup in kfbAllGroup:
        kfbGroupName = kfbGroup.name
        kfbGroupDesc = kfbGroup.description
        kfbGroupPath = kfbGroup.path
        whGroup = None
        try:
            whGroup = whGitlab.groups.get(kfbGroupName)
            # print('获取到目标仓库分组' + whGroup.name)
        except Exception as err:
            whGroup = whGitlab.groups.create({'name': kfbGroupName, 'path': kfbGroupPath})
            whGroup.description = kfbGroupDesc
            whGroup.save()
            # print('创建目标仓库分组' + whGroup.name)

        # 列出源仓库分组中所有的项目
        kfbGroupProjects = kfbGroup.projects.list(all=True)
        for kfbGroupProject in kfbGroupProjects:
            kfbProjectName = kfbGroupProject.name
            kfbProjectDesc = kfbGroupProject.description
            kfbProject = kfbGitlab.projects.get(kfbGroupProject.id, lazy=True)
            whGroupProject = None

            whGroupProjectList = whGroup.projects.list(search=kfbProjectName)

            if len(whGroupProjectList) == 0:
                whGroupProject = whGitlab.projects.create({'name': kfbProjectName, 'namespace_id': whGroup.id})
                whGroupProject.description = kfbProjectDesc
                whGroupProject.save()
                # print('创建目标项目成功' + kfbProjectName)
            else:
                whGroupProject = whGroupProjectList[0]
                # print('创建目标项目成功' + kfbProjectName)
            groupName = kfbGroupName
            projectName = kfbProjectName
            whGitlbUrl = whGroupProject.ssh_url_to_repo
            kfbGitlbUrl = kfbGroupProject.ssh_url_to_repo
            # print('group:' + groupName + ',project:' + projectName + ',wh:' + whGitlbUrl + ',kfb:' + kfbGitlbUrl)

            # tempPath = Path('D:/temp/' + kfbGroupName)

            # if tempPath.exists() == False:
            #    os.mkdir('D:/temp/' + kfbGroupName)
            # os.chdir('D:/temp/' + kfbGroupName)

            groupDir = tempDir + '/' + kfbGroupName
            print('mkdir -p ' + groupDir)
            print('cd ' + groupDir)
            expCmd = 'git clone --mirror ' + kfbGitlbUrl
            print(expCmd)
            print('cd ' + kfbProjectName + '.git')
            impCmd = 'git push --mirror ' + whGitlbUrl
            print(impCmd)


if __name__ == '__main__':
    main()


```
  
- python-gitlab.cfg

``` properties
[global]
default = git
ssh_verify = False
timeout = 10

[kfb]
url = http://16.16.5.172
api_version = 4
private_token = 1gTwRkqPRzDYfnmH1c4b

[wh]
url = http://16.16.5.172:81
api_version = 4
private_token = 7xd29qXyo3xfHXg-T9XF
```
