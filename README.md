# MyGit
MyGit


# 說明
由於公司團隊使用GitLab 來託管代碼，同時，個人在Github 上還有一些代碼倉庫，可公司郵箱與個人郵箱是不同的，由此產生的SSH key 也是不同的，這就造成了衝突，文章提供此類問題的解決方案：如何在一台機器上面同時使用Github 與Gitlab 的服務 ? 參考步驟如下。


# 全局配置，Github倉庫中默認使用此配置
git config --global user.name '02445' && git config --global user.email 'gatsby.lin@topco-global.com'

# 團隊項目配置，每次新創建一個項目，需要執行下
git config --local user.name 'c7934597' && git config --local user.email 'c7934597@gmail.com'

# 生成公鑰、密鑰的同時指定文件名，Github使用
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "c7934597@gmail.com"

# 生成默認，Gitlab使用
ssh-keygen -t rsa -C "gatsby.lin@topco-global.com"

# 配置 config 文件
在 ~/.ssh目錄下，如果不存在，則新建 touch ~/.ssh/config文件 ，文件內容添加如下：

touch ~/.ssh/config

vim ~/.ssh/config

Host github.com
IdentityFile ~/.ssh/id_rsa.github
User c7934597

配置完成後，符合 github.com Git 倉庫，均採取~/.ssh/id_rsa.github 密鑰進行驗證，其它的採取默認的。

# 上傳public key 到 Github/Gitlab
vim ~/.ssh/id_rsa.github.pub

# 驗證是否OK
由於每個託管商的倉庫都有唯一的後綴，比如 Github的是 git@github.com:*，所以可以這樣測試：

➜ ~ ssh -T git@github.com

Hi c7934597! You've successfully authenticated, but GitHub does not provide shell access.

➜ ~ ssh -T git@gitlab.dev

Welcome to GitLab, 02445!

看到這些 Welcome 信息，說明就是 OK的了。

# 最後加上remote
git remote add origin https://github.com/c7934597/ASP.NET-CORE-2.git

# 參考資料來源
https://github.com/xirong/my-git/blob/master/use-gitlab-github-together.md
