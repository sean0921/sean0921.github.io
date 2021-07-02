---
title: 成為 Git 獨行玩家的第一步 － 自架 cgit 網頁程式碼瀏覽服務
tags:
  - git
  - cgit
  - nginx
  - fastcgi
  - opensource
date: 2021-03-27 01:47:27
---

不想被單一特定程式碼平臺綁住你個人開發的程式碼又想將你的心血整合起來供其他人參考嗎? 又或是擔心在 GitHub 上一堆為了學術研究用途的專案哪一天被和諧掉嗎? 如果沒有多人合作 CI/CD 需求直接從網頁登入存取的需求 `cgit` 這個由 C 語言開發的網頁 cgi 服務, 簡潔又快速的功能或許可以輕易滿足你的需求喔

<!--more-->

## 步驟

### 1. 環境配置

#### 運行環境

任何 Unix-like 環境 ( Linux/FreeBSD/Cygwin/WSL ) 皆可, 本文章以 *Debian GNU/Linux Buster(10)* 為例進行解說, 相關過程指令也相容 *Ubuntu Linux*, *Debian GNU/Linux Bullseye(11)*

#### 安裝相關套件

```
apt install cgit fcgiwrap nginx git python3-markdown
```

### 2. 調校 nginx, cgit

- 刪除 `/etc/nginx/sites-{enabled,available}/default`, 用以下設定檔取代
- `/etc/nginx/conf.d/cgit.conf`:

```conf
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;


        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                #try_files $uri $uri/ =404;
                try_files $uri @cgit;    ## 這邊要使用 cgit 的 CGI 程式來解譯路徑
        }


        location /cgit-css/ {
                rewrite ^/cgit-css(/.*)$ $1 break;
                root /usr/share/cgit;    ## cgit 的 CSS 排版格式與相關預設圖片需要它
        }


        location @cgit {
                include             fastcgi_params;

                # Path to the CGI script that comes with cgit
                fastcgi_param       SCRIPT_FILENAME /usr/lib/cgit/cgit.cgi;

                fastcgi_param       PATH_INFO       $uri;
                fastcgi_param       QUERY_STRING    $args;
                fastcgi_param       HTTP_HOST       $server_name;

                # Path to the socket file that is created/used by fcgiwrap
                fastcgi_pass        unix:/run/fcgiwrap.socket;
        }
}
```

- `/etc/cgitrc`:
```conf
#
# cgit config
# see cgitrc(5) for details

## 這邊要搭配 nginx 設定的路徑來設定相關資源位置
css=/cgit-css/cgit.css
logo=/cgit-css/cgit.png
favicon=/cgit-css/favicon.ico

# if you don't want that webcrawler (like google) index your site
#robots=noindex, nofollow

root-title=Andas valiendo verga
root-desc=r2's Git Repositories
# if cgit messes up links, use a virtual-root. For example has cgit.example.org/ this value:
virtual-root=/
#virtual-root=/cgit.cgi/
enable-http-clone=1
cache-scanrc-ttl=1
#remove-suffix=1
#branch-sort=age
local-time=1
enable-blame=1
enable-commit-graph=1
enable-index-owner=1
enable-log-filecount=1
enable-log-linecount=1
max-stats=year
max-commit-count=250
max-repo-count=250
snapshots=tar.gz tar.zst

# Specify some default clone urls using macro expansion
#clone-url=http://myip/$CGIT_REPO_URL

# Highlight source code with python pygments-based highligher
source-filter=/usr/lib/cgit/filters/syntax-highlighting.py

# Format markdown, restructuredtext, manpages, text files, and html files
# through the right converters
about-filter=/usr/lib/cgit/filters/about-formatting.sh

enable-index-links=1
root-readme=/my/gitrepos/git/readme.md
#root-readme=/my/gitrepos/git/readme.html

## 這邊可以幫你設定 cgit 掃描的 repo 所在資料夾
scan-path=/srv/gitrepos/git/

##
## List of common mimetypes
##

mimetype.gif=image/gif
mimetype.html=text/html
mimetype.jpg=image/jpeg
mimetype.jpeg=image/jpeg
mimetype.pdf=application/pdf
mimetype.png=image/png
mimetype.svg=image/svg+xml

##
## Search for these files in the root of the default branch of repositories
## for coming up with the about page:
## 這邊如果沒安裝 python3-markdown 等套件的話README 仍然無法顯示 markdown 格式的喔
##

readme=:README.md
readme=:readme.md
readme=:README.mkd
readme=:readme.mkd
readme=:README.rst
readme=:readme.rst
readme=:README.html
readme=:readme.html
readme=:README.htm
readme=:readme.htm
readme=:README.txt
readme=:readme.txt
readme=:README
readme=:readme
readme=:INSTALL.md
readme=:install.md
readme=:INSTALL.mkd
readme=:install.mkd
readme=:INSTALL.rst
readme=:install.rst
readme=:INSTALL.html
readme=:install.html
readme=:INSTALL.htm
readme=:install.htm
readme=:INSTALL.txt
readme=:install.txt
readme=:INSTALL
readme=:install

###Catogory
#scan-path=/my/gitrepos/git/
## 這邊可以另外幫自己分類 git 專案
#
#section=a: BBS Projects
#scan-path=/my/gitrepos/git/category/bbs
#section=b: My Distro-Hacking Records
#scan-path=/my/gitrepos/git/category/distro
#section=c: My Personal Toys
#scan-path=/my/gitrepos/git/category/mytoy
```


### 3. 啟用服務與啟動服務

```bash=
nginx -t ## 檢查 nginx config 檔格式是否正確
systemctl enable --now nginx
systemctl enable --now fcgiwrap
```

### 4-1. 抓取你想要鏡像的 git repository

```bash=
git clone --mirror https://github.com/<github_user_name>/<github_repo_name>
```

### 4-2. 透過 git+ssh server 設定你想要託管的 git repository (optional)

```bash=
chsh -s /usr/bin/git-shell git
sudo -su git cat <place_of_your_ssh_public_key> >> <git_account_home>/.ssh/authorized_keys
```

### 5. 噹噹! 開始使用你的服務

輸入網址, 享用你的自架 git 服務: `http://<your_cgit_ip_or_domain>`

若服務對外公開的話, 建議還是設定一下 TLS 安全連線比較保險一些

詳細 TLS 憑證設定與 Let's Encrypt 憑證申請, 可以參考以下資源進行調整:
- https://wiki.gslin.org/wiki/Let%27s_Encrypt
- https://ssl-config.mozilla.org/

---

我的 git repository: <https://cgit.clam.ml>
