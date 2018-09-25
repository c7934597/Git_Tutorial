# MyGit
MyGit

說明
由於公司團隊使用GitLab 來託管代碼，同時，個人在Github 上還有一些代碼倉庫，可公司郵箱與個人郵箱是不同的，由此產生的SSH key 也是不同的，這就造成了衝突，文章提供此類問題的解決方案：如何在一台機器上面同時使用Github 與Gitlab 的服務？

問題產生場景
無密碼與遠程服務器交互的秘密 - SSH
如果採用ssh 協議或者git 協議通過終端命令對遠程倉庫進行push操作的時候，大概的過程如下：（前提在 Github 上已經配置的本機的 SSH Public Key）

客戶端發起一個 Public Key 的認證請求，並發送RSA Key的模數作為標識符。 （關於 RSA Key 詳細 維基百科）
服務端檢查是否存在請求帳號的公鑰（Linux中存儲在~/.ssh/authorized_keys文件中），以及其擁有的訪問權限。
服務端使用對應的公鑰對一個隨機的256位的字符串進行加密，並發送給客戶端。
客戶端使用私鑰對字符串進行解密，並將其結合session id生成一個MD5值發送給服務端。結合session id的目的是為了避免攻擊者採用重放攻擊（replay attack）。
服務端採用同樣的方式生成MD5值與客戶端返回的MD5值進行比較，完成對客戶端的認證。
將push的內容進行加密與服務端傳輸數據。
關於 SSH，請查看 SSH原理簡介 ，更通俗易懂的文章請查看阮一峰-SSH原理與運用（一）：遠程登錄 。

具體場景
無論使用哪種代碼託管服務商，對於Git 而言，郵箱是識別用戶的唯一手段，所以對於不同的服務商，由於郵箱不同，那麼通過郵件名創建的SSH Key 自然是不同的，這時候在不同的服務商之間進行push 命令的時候，Git 是不知道使用哪個SSH Key ，自然導致push 的失敗。場景如下：

在公司團隊使用搭建的 Gitlab 服務，提交郵箱gatsby.lin@topco-global.com， 個人 Github 服務，提交郵箱 c7934597@gmail.com （Bitbucket 同理）。
有兩個Github賬戶，不同的賬戶提交不同的倉庫內容。
解決方案
方案一：同一個郵箱
由於郵箱是識別的唯一手段，那麼自然的，這兩者採用同一個郵箱，生成的public key 也會是同一個，上傳到Github 或者Gitlab 上面，在Git 的配置中，設置好Global 的配置：git config --global user.name '02445' && git config --global user.email 'gatsby.lin@topco-global.com' 進行日常的開發是沒有問題的。

實際生活中採用同一個郵箱的可能性並不是太大，這就引出了方案二

方案二：基於config文件
所謂的方案二，原理上就是對 SSH 協議配置 config 文件，對不同的域名採用不同的認證密鑰。

git config 介紹
Git有一個工具被稱為git config，它允許你獲得和設置配置變量；這些變量可以控制Git的外觀和操作的各個方面。這些變量可以被存儲在三個不同的位置：

/etc/gitconfig 文件：包含了適用於系統所有用戶和所有庫的值。如果你傳遞參數選項’--system’ 給 git config，它將明確的讀和寫這個文件。
~/.gitconfig 文件 ：具體到你的用戶。你可以通過傳遞 ‘--global’ 選項使Git 讀或寫這個特定的文件。
位於 Git 目錄的 config 文件 (也就是 .git/config) ：無論你當前在用的庫是什麼，特定指向該單一的庫。每個級別重寫前一個級別的值。因此，在 .git/config 中的值覆蓋了在/etc/gitconfig中的同一個值，可以通過傳遞‘--local’選項使Git 讀或寫這個特定的文件。
由於採用了不同的郵箱，對不同的服務商進行提交，所以此時我們經常配置的 git config --global 就不能常用了，必須在每個倉庫的目錄下進行配置自己的用戶名、郵箱。 （嫌麻煩？xirong 是這麼解決的，由於個人的Github 上有較多的倉庫，而自己團隊的代碼基本上都是穩定的，有數的幾個，所以在git config --global user.email 'c7934597@gmail.com' 中全局配置的是個人郵箱，在團隊的項目中配置）

1. 配置 Git 用戶名、郵箱
如剛才所說，c7934597 的配置如下：

# 全局配置，Github倉庫中默認使用此配置
git config --global user.name '02445' && git config --global user.email 'gatsby.lin@topco-global.com'

# 團隊項目配置，每次新創建一個項目，需要執行下
git config --local user.name 'c7934597' && git config --local user.email 'c7934597@gmail.com'
2. 生成 ssh key 上傳到 Github/Gitlab
ssh key 默認生成後保存在 ~/.ssh/目錄下 ，默認為 id_rsa 和 id_rsa.pub 兩個文件，由於我們需要分開配置，所以這麼做：

# 生成公鑰、密鑰的同時指定文件名，Github使用
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "c7934597@gmail.com"

# 生成默認，Gitlab使用
ssh-keygen -t rsa -C "gatsby.lin@topco-global.com"
命令執行完成後，這時~/.ssh目錄下會多出id_rsa.github和id_rsa.github.pub兩個文件，id_rsa.github.pub 裡保存的就是我們要使用的key，這個key就是用來上傳到Github上的。

3. 配置 config 文件
在 ~/.ssh目錄下，如果不存在，則新建 touch ~/.ssh/config文件 ，文件內容添加如下：

Host github.com
     IdentityFile ~/.ssh/id_rsa.github
     User c7934597
配置完成後，符合 github.com後綴的 Git 倉庫，均採取~/.ssh/id_rsa.github 密鑰進行驗證，其它的採取默認的。

4. 上傳public key 到 Github/Gitlab
以Github為例，過程如下：

登錄github
點擊右上方的Accounting settings圖標
選擇 SSH key
點擊 Add SSH key
在出現的界面中填寫SSH key的名稱，填一個你自己喜歡的名稱即可，然後將上面拷貝的~/.ssh/id_rsa.github.pub文件內容粘帖到key一欄，在點擊“add key”按鈕就可以了。

添加過程github會提示你輸入一次你的github密碼 ，確認後即添加完畢。上傳Gitlab的過程一樣，請自己操作。

5. 驗證是否OK
由於每個託管商的倉庫都有唯一的後綴，比如 Github的是 git@github.com:*，所以可以這樣測試：

➜ ~ ssh -T git@github.com
Hi xirong! You've successfully authenticated, but GitHub does not provide shell access.
➜ ~ ssh -T git@gitlab.dev
Welcome to GitLab, xirong.liu!
看到這些 Welcome 信息，說明就是 OK的了。

➜ cd ~/.ssh
➜ ls -la
total 24
drwxr-xr-x 1 02445 1049089    0 九月 25 17:35 ./
drwxr-xr-x 1 02445 1049089    0 九月 25 17:35 ../
-rw-r--r-- 1 02445 1049089   64 九月 25 17:06 config
-rw-r--r-- 1 02445 1049089 1679 九月 25 16:56 id_rsa
-rw-r--r-- 1 02445 1049089 1679 九月 25 16:56 id_rsa.github
-rw-r--r-- 1 02445 1049089  400 九月 25 16:56 id_rsa.github.pub
-rw-r--r-- 1 02445 1049089  409 九月 25 16:56 id_rsa.pub
-rw-r--r-- 1 02445 1049089  407 九月 25 17:06 known_hosts
