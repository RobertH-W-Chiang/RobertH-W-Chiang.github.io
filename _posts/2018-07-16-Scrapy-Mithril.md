---
categories: Programming
tags: Python Scrapy Crontab
---

前言
===

### 目標
> <i class="fa fa-pencil" aria-hidden="true"></i> 用 `scrapy` 定期去抓秘銀幣的價格，當價格高於一個門檻後，就寄送 mail 通知！

(以下操作皆在 ubuntu14.04 中執行)

---

Scrapy
===
1. 安裝 scrapy  
    `pip install scrapy`
2. 新增資料夾  
    `mkdir mithril_spider && cd mithril_spider`
3. 建立撰寫爬蟲程式的檔案  
	`vim mithril_spider.py`
4. `mithril_spider.py` 中大概的程式碼  
	``` python
	import scrapy

	from email.mime.multipart import MIMEMultipart
	from email.mime.text import MIMEText
	from smtplib import SMTP

	class MithrilSpider(scrapy.Spider):
	    name = 'mithril_spider'
	    start_urls = ['https://max.maicoin.com/markets/mithtwd']

	    def parse(self, response):
	    	smtp = SMTP(YOUR_SMTP_HOST, YOUR_SMTP_HOST_PORT)
			smtp.starttls()
			smtp.login(YOUR_EMAIL, YOUR_EMAIL_PASSWORD)
			sender = 'SENDER_NAME'
			recipients = ['recipient1', 'recipient2']

			msg = MIMEMultipart()
			msg['From'] = sender
			msg['To'] = ', '.join(recipients)

			mith_price = float(response.xpath('//*[@id="market-list-mithtwd"]/td[2]//text()').extract_first())

			mith_threshold_price = 1000
			if mith_price >= mith_threshold_price:
				msg['Subject'] = 'Subject'
				text = MIMEText('mail body')
				msg.attach(text)
				smtp.sendmail(sender, recipients, msg.as_string())
				smtp.quit()
	```

crontab
===

1. 編輯 crontab  
	`crontab -e`
2. 撰寫 `cron task`  
	``` shell
	PATH=/usr/local/bin
	*/5 * * * * cd /home/ubuntu/mithril_spider && scrapy runspider mithril_spider.py >> /home/ubuntu/mithril_spider/mithril_spider.log 2>&1
	```
	
	記得一定要加 `PATH`，否則會噴錯：`scrapy: command not found`  
	不知道 `PATH` 的話可以下 `whereis scrapy` 指令去查看 `scrapy` 在哪

