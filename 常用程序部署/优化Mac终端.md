> 本文作者：丁辉
>
> 本文介绍如何安装"Oh My Zsh" 终端和 "zsh-autosuggestions" 自动补全工具
>
> 没有解释，就是好用

## 安装Item2 终端

[官网地址](https://iterm2.com/index.html)

下载安装包后直接解压双击安装使用

## 部署 Oh My Zsh

[Github项目地址](https://github.com/ohmyzsh/ohmyzsh)

打开 Mac 终端输入(下列三种方式都可以，建议使用 curl 因为 mac 一般自带哈哈)

| Method    | Command                                                      |
| --------- | ------------------------------------------------------------ |
| **curl**  | `sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` |
| **wget**  | `sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` |
| **fetch** | `sh -c "$(fetch -o - https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` |

**启用插件**

```bash
vi ~/.zshrc
```

**替换如下内容**

原内容如下

```bash
plugins=(git)
```

修改后

```bash
plugins=(
  git
  bundler
  dotenv
  macos
  rake
  rbenv
  ruby
)
```

重启终端查看效果

## 部署 zsh-autosuggestions

[Github项目地址](https://github.com/zsh-users/zsh-autosuggestions)

**克隆仓库**

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions.git ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```

**修改配置文件**

```
vim ~/.zshrc
```

**添加 `zsh-autosuggestions` 到 plugins=() 模块内**

```bash
plugins=( 
    zsh-autosuggestions
)
```

重启终端查看效果

## 更换主题

[主题展示](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)

**编辑配置文件**

```
~/.zshrc
```

找到 `ZSH_THEME` 配置

```
ZSH_THEME="agnoster"
```

## 安装字体

[Github项目地址](https://github.com/powerline/fonts)

**克隆仓库**

```
git clone https://github.com/powerline/fonts.git --depth=1
```

**安装**

```
cd fonts
./install.sh
cd ..
rm -rf fonts
```

### 更换字体

#### Vscode更换字体

打开vscode > Settings > 搜索font > 找到`Terminal › Integrated: Font Family` 

填写字体(格式如下)

可以用 "," 来分隔多个字体

```
Go Mono for Powerline,Hack
```

#### Mac 默认终端更换字体

点击左上角终端 > 偏好设置 > 描述文件 > 更改字体 > 点击搜索添加

#### Item2 终端更换字体

点击左上角iTerm2 > Preferences > Profiles > Text > 勾选 Use built-in Powerline glyphs
