---
categories: Programming
tags: VisualCryptography Django GoogleCloudPlatform
---

前言
===

### 目標
> <i class="fa fa-pencil" aria-hidden="true"></i> 用 `Django` 撰寫關於 _Visual Cryptography_ (視覺密碼) 的網站

### 關於此專案
*  開發環境：`Ubuntu 14.04`
* `Django` 版本：1.10
* `Django template project`：[cookiecutter-django](https://github.com/pydanny/cookiecutter-django)

### Platform
原本想要在自己的電腦開發，然後 host 到 cloud platform 上，例如：[Heroku](https://www.heroku.com/)、[PythonAnywhere](https://www.pythonanywhere.com/)  
後來想想還是直接在雲端的 VM 中開發，然後設定一下 security group 就能用了  
最終我選擇的是 [Google Cloud Platform](https://cloud.google.com/?hl=zh-tw) (以下簡稱為 GCP)

---

在 GCP 中建立 VM
===
1. [加入免費試用 GCP](https://cloud.google.com/free/?hl=zh-tw)
2. 新建一個專案
3. 在專案中建立執行個體 (instance)，以下列出我有調整的項目：  
   * 區域：我是選 asia-east1-x 區域 (asia 比 us 的貴...)
   * 機器類型：我是選 1 vCPU 1.7 GB 記憶體的，不夠的話再換就好
   * 開機磁碟：Ubuntu 14.04
   * 網路：點擊網路 -> 編輯網路介面 (icon 是一支鉛筆) -> 在外部 IP 那邊選擇建立 IP 位址。這個步驟的作用是幫 VM 建立固定 IP，會比較方便使用
4. VM 建立完成後，有多種方式可以連進去，我是選擇用 [Cloud SDK](https://cloud.google.com/sdk/?hl=zh-tw)  
   看您的作業系統是什麼就下載該系統的 Cloud SDK，官方的文件有教學如何安裝
5. 用 Cloud SDK ssh 至 VM：`gcloud compute ssh YOUR_INSTANCE_NAME --zone YOUR_INSTANCE_ZONE`
6. 完成，可以開始使用 VM 了！

題外話，可以直接透過 GCP 的介面，用瀏覽器 SSH，蠻厲害的...  

![GCP_browser_SSH.png]({{site.baseurl}}/assets/images/GCP_browser_SSH.png)

---

使用 [cookiecutter-django](https://github.com/pydanny/cookiecutter-django) 建立 `Django` 專案
===
有一本權威的 `Django` 書籍：[Two Scoops of Django 1.11: Best Practices for the Django Web Framework](https://www.amazon.com/Two-Scoops-Django-1-11-Practices/dp/0692915729)  
作者是一對夫妻，他們實作了一個 `Django` template 專案：`cookiecutter-django`，以下是我在 Ubuntu 14.04 中安裝 `cookiecutter-django` 的過程及指令  ：
1. `apt-get update`
1. `apt-get install python-pip python3-dev git`  
   我是 Python 3.4，所以裝 `python3-dev`，若是用 Python 2.x，則安裝 `python-dev`
1. 安裝 Postgresql，請記得版本，創建 `cookiecutter-django` 專案時會用到，我是裝 9.6  
   * `add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main"`
   * `wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -`
   * `apt-get install postgresql-9.6`
1. 用 `pip` 安裝 `virtualenv` 或 `virtualenvwrapper`，我是裝後者：`pip install virtualenvwrapper`
1. 創建虛擬環境：  
   * `export WORKON_HOME=YOUR_WORKON_HOME_DIRECTORY`
   * `mkdir -p $WORKON_HOME`
   * `source /usr/local/bin/virtualenvwrapper.sh`
   * `mkvirtualenv YOUR_VENV_NAME --python=/usr/bin/python3`  
      特別注意，如果是用 Python 3.x 一定要加 `--python` 參數去指定虛擬環境中的 Python 版本
1. 安裝 `cookiecutter-django`：  
   * `pip install cookiecutter`
   * `cookiecutter https://github.com/pydanny/cookiecutter-django`
   接著就會出現一連串的設定需要你輸入，輸入完畢之後就會出現一個 `YOUR_PROJECT_SLUG` 的資料夾
   需要特別注意的是 `project_slug` 和 `postgresql_version` 這兩個設定：  
     * `project_slug`：等同於資料庫的名稱，務必記得，我也把 `project_slug` 當成 github repository 的名稱`
     * `postgresql_version`：和上述安裝的 `Postgresql` 版本需一致
1. `cd YOUR_PROJECT_SLUG`
1. `pip install -r requirements/local.txt`
1. 建立 `Postgresql` 資料庫 for `cookiecutter-django` 專案  
   * `sudo -u postgres psql`  
     輸入這行後會進入 postgresql console，輸入：`CREATE USER YOUR_CURRENT_USERNAME WITH SUPERUSER;`  
     例如：我登入到 GCP VM 的 username 是 gipai，則指令為 `CREATE USER gipai WITH SUPERUSER;`
   * `createdb YOUR_PROJECT_SLUG`
1. `./manage.py migrate`
1. `./manage.py runserver 0.0.0.0:8000`
1. 如果是用 GCP 的話，要幫 VM 開 security group，預設 8000 port 是被鎖的： 
   * 前往 GCP 主控台 -> VPC 網路 -> 防火牆規則
   * 點擊建立防火牆規則
   * 我有調整的項目如下：  
     * 目標：網路中的所有執行個體
     * 來源 IP 範圍：`0.0.0.0/0`
     * 指定的通訊協定和通訊埠：`tcp:8000`
1. 調整 `Django` settings 的 `ALLOWED_HOST` 設定，我是設定為 `['*']`
1. 接著就可以在瀏覽器輸入 YOUR_MACHINE_IP:8000 瀏覽網頁囉！

---

把程式碼 commit 到 GitHub
===
* `git init`
* `git add -A`
* `git commit -m 'YOUR COMMIT MESSAGE'`
* `git remote add origin YOUR_GITHUB_REPOSITORY_URL`
* `git push -u origin master`
