---
title: "جنگو در یک نگاه"
date: 2018-10-17T15:26:15Z
lastmod: 2018-10-26T15:26:15Z
draft: false
weight: 20
---

جنگو با فلسفه توسعه سریع و راحت، ایجاد شده. در این بخش شما خیلی سریع از نظر تکنیکی می‌فهمید که جنگو چطور کار می‌کنه. این بخش برای آموزش طراحی نشده. اگر چیزی ازش نمی‌فهمید یا به نظرتون خیلی گیج‌کننده و درهم برهم میاد، نگران نباشید، ما یک آموزش جالب و جذاب هم داریم. هر وقت آماده بودید تا یک پروژه واقعی جنگو رو شروع کنید، می‌تونید از [شروع آموزش](/tutorial01) دیدن کنید!

## مدل‌تون رو طراحی کنید

گرچه شما می‌تونید از جنگو، بدون دیتابیس هم استفاده کنید، اما جنگو با یک ORM یا [object-relational mapper](https://en.wikipedia.org/wiki/Object-relational_mapping) که یک روش مدرن در مدیریت اطلاعات هست همراه شده. به این صورت که شما می‌تونید مدل پایه دیتابیس‌تون رو با استفاده از کدهای پایتون پیاده‌سازی کنید.

سینتکس منحصر به فرد مدل‌های جنگو یک عالم راه‌های خوب برای شبیه‌سازی و پیاده کردن مدل‌های مدیریت اطلاعات‌تون ارائه می‌ده. خب این باعث شده که یک‌سری از مشکلات چندین ساله دیتابیس حل بشه.
یک مثال سریع ببینید:


<p style="direction:ltr; text-align: left; color:#0C4B33; background:#C9F0DD">
<code style="background:#C9F0DD">
   mysite/news/models.py:
</code></p>
<pre style="background:none; margin:0; padding: 0;">
from django.db import models
class Reporter(models.Model):
	full_name = models.CharField(max_length=70)
	def __str__(self):
		return self.full_name
class Article(models.Model):
	pub_date = models.DateField()
	headline = models.CharField(max_length=200)
	content = models.TextField()
	reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)
	def __str__(self):
	    return self.headline
</pre>

***

## نصب‌ش کنید

بعد از این‌که مدل‌تون پیاده‌سازی شد، این دستورات رو در ترمینال وارد کنید.

```
$ python manage.py makemigrations
$ python manage.py migrate
```

دستور [makemigrations](https://docs.djangoproject.com/en/2.2/ref/django-admin/#django-admin-makemigrations) همه‌ی مدل‌هایی که پیاده‌سازی کردید رو نگاه می‌کنه و یک migrations از اون‌ها برای جداولی که از قبل وجود نداشتن می‌سازه. پس از اون دستور [migrate](https://docs.djangoproject.com/en/2.2/ref/django-admin/#django-admin-migrate) می‌آد و migrationsهای ساخته شده رو اجرا می‌کنه و جداول مورد نیاز در دیتابیس شما ساخته می‌شه.

***

## از APIهای آزاد استفاده و کِیف کنید!


شما با استفاده از [Python API](https://docs.djangoproject.com/en/2.2/topics/db/queries/) که آزاد و خیلی هم خفن هست به دیتاهاتون دسترسی دارید. این APIها در آسمان‌ها ساخته شدن؛ یعنی نیاز به تولید هیچ کدی نیست:

```
# Import the models we created from our "news" app
>>> from news.models import Article, Reporter
# No reporters are in the system yet.
>>> Reporter.objects.all()
<QuerySet []>
# Create a new Reporter.
>>> r = Reporter(full_name='John Smith')
# Save the object into the database. You have to call save() explicitly.
>>> r.save()
# Now it has an ID.
>>> r.id
1
# Now the new reporter is in the database.
>>> Reporter.objects.all()
<QuerySet [<Reporter: John Smith>]>
# Fields are represented as attributes on the Python object.
>>> r.full_name
'John Smith'
# Django provides a rich database lookup API.
>>> Reporter.objects.get(id=1)
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__startswith='John')
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__contains='mith')
<Reporter: John Smith>
>>> Reporter.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Reporter matching query does not exist.
# Create an article.
>>> from datetime import date
>>> a = Article(pub_date=date.today(), headline='Django is cool',
...     content='Yeah.', reporter=r)
>>> a.save()
# Now the article is in the database.
>>> Article.objects.all()
<QuerySet [<Article: Django is cool>]>
# Article objects get API access to related Reporter objects.
>>> r = a.reporter
>>> r.full_name
'John Smith'
# And vice versa: Reporter objects get API access to Article objects.
>>> r.article_set.all()
<QuerySet [<Article: Django is cool>]>
# The API follows relationships as far as you need, performing efficient
# JOINs for you behind the scenes.
# This finds all articles by a reporter whose name starts with "John".
>>> Article.objects.filter(reporter__full_name__startswith='John')
<QuerySet [<Article: Django is cool>]>
# Change an object by altering its attributes and calling save().
>>> r.full_name = 'Billy Goat'
>>> r.save()
# Delete an object with delete().
>>> r.delete()
```
***

## یک رابط مدیریت پویا: این یک اسکلت خالی نیست، کل ساختمونه!

وقتی که مدل شما پیاده‌سازی شد، جنگو می‌تونه به صورت اتوماتیک یک رابط مدیریتی حرفه‌ای، قابل استفاده و خوانا بسازه. در وبسایت، کاربران می‌تونن اشیا جدید بسازند، اون‌ها رو ویرایش کنند یا حذف‌شون کنند. این کار به سادگی با ثبت کردن مدل‌تون در بخش ادمین امکان پذیره:

<p style="direction:ltr; text-align: left; color:#0C4B33; background:#C9F0DD">
<code style="background:#C9F0DD">
   mysite/news/models.py:
</code></p>
<pre style="background:none; margin:0; padding: 0;">
from django.db import models
class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)
</pre>


<p style="direction:ltr; text-align: left; color:#0C4B33; background:#C9F0DD">
<code style="background:#C9F0DD">
  mysite/news/admin.py:
</code></p>
<pre style="background:none; margin:0; padding: 0;">
from django.contrib import admin
from . import models
admin.site.register(models.Article)
</pre>

فلسفه کار اینه که سایت شما توسط همکاران شما، کاربران شما یا فقط خودتون ویرایش می‌شه - همچنین شما نمی‌خواید که با مشکلات زیاد ایجاد یک رابط پشتی برای مدیریت محتوا سایت‌تون دست و پنجه نرم کنید.

یک روش کار معمول در ساخت پروژه‌های جنگو اینه‌که شما مدل‌هاتون رو می‌سازید و بخش ادمین سایت‌تون رو هرچه زودتر راه‌اندازی می‌کنید؛ حالا همکاران یا کاربران شما می‌تونن دیتاهایی رو در سایت وارد کنند. بعد از اون یک راهی رو طراحی می‌کنید که اون اطلاعات به صورت عمومی منتشر و استفاده بشه.

***

## آدرس‌هاتون رو طراحی کنید
