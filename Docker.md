Docker
========================

**Install Docker**

echo "nameserver 178.22.122.100" > /etc/resolv.conf افزودن DNS  شکن در فایل مذکور برای دور زدن تحریم ها

curl -fsSL get.docker.com -o get-docker.sh دانلود اسکریپ ذکر شده و اجرای آن برای نصب کامل داکر 
sh get-docker.sh

sudo usermod -aG docker your-user افزودن یوزر با دسترسی که بتواند دستورات داکر را اجرا کند در غیر اینصورت فقط با روت قابل اجراست

systemctl start docker
systemctl status docker
systemctl enable docker


**Command**

docker info جزپیات سرویس داکر 
docker ps 
-a نمایش همه کانتینر ها کانتینر های روشن و خاموش
-f برای فیلتر نویسی مثل name=test
-l آخرین کانتینر
-s سایز کانتینر
--no-trunc اطلاعات کامل کانتینر ها

docker images نمایش ایمیجهای موجود
docker search image name جستجوی ایمیج از ریجیستری
docker pull "image name" دریافت ایمیج از ریجیستری
docker tag --help تگ زدن به یک ایمیج
docker login "registry addr" لاگین شدن به ریجیستری
docker logout
docker push image_name:tag انتقال ایمیج به ریجیستری
 
docker top کانتینر های درحال اجرا را نمایش میدهد
docker start "container" راه اندازی کانتینر مرده
docker stop "container" غیر فعال کردن کانتیر
docker pouse "container"
docker unpouse "container"
docker history image نمایش تغییرات در ایمیج
docker create
docker run --help "more option" برای اجرای ایمیج و تبدیل به کانتینر
docker run -it -d image -v /path/...
docker port "container"  بررسی پورت هایی که از سیستم به کانتینر داده شده
docker logs "container" بررسی لاگهای مربوط به یک کانتینر خاص
docker diff 
docker event --since 10m اتفاقاتی که داخل داکر افتاده است به طور مثال ده دقیقه آخر را نمایش میدهد

docker build ایجاد ایمیج توسط داکر فایل
docker commit ایجاد ایمیج از کانتینر درحال اجرا
docker cp  انتقال فایل از کانتینر به هاست و بالعکس

docker exec -it container command برای اجرای دستور خاص در داخل کانتینر توسط هاست

docker export container > path/test.tar خروجی از فایل سیستم کانتینر
docker import path/test.tar test:tag
docker rename old_name new_name تغییر نام کانتینر ها
docker rm container حذف کانتینر
docker rmi image حذف ایمیج

Docker Network: داکتر نتورک مبحث طولانی و مهمی است که نیاز به مطالعه جداگانه دارد
docker network -- help more option

docker save image  
docker save test:1 -o /path/test.tar

docker load -i /path/test.tar

docker kill container
docker inspect container 
docker inspect image


 







