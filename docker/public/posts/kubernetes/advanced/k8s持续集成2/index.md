# 

安装gitlib

```bash
root@ceamg-server:~# dpkg -i /tmp/gitlab-ce_16.5.2-ce.0_amd64.deb
Selecting previously unselected package gitlab-ce.
(Reading database ... 75661 files and directories currently installed.)
Preparing to unpack .../gitlab-ce_16.5.2-ce.0_amd64.deb ...
Unpacking gitlab-ce (16.5.2-ce.0) ...
Setting up gitlab-ce (16.5.2-ce.0) ...
It looks like GitLab has not been configured yet; skipping the upgrade script.

       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.



     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/


Thank you for installing GitLab!
GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

Help us improve the installation experience, let us know how we did with a 1 minute survey:
https://gitlab.fra1.qualtrics.com/jfe/form/SV_6kVqZANThUQ1bZb?installation=omnibus&release=16-5
```



```bash
vim /etc/gitlab/gitlab.rb
external_url 'http://10.1.0.140'
```





```bash
gitlab-ctl reconfigure
```



```bash
root@gitlab:~# cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: I5qRTkn+LrdefGIo43ifT0cBHQUuEAHoJDuZuaS4CG8=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```



![image-20231201155418428](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231201155418428.png)

![image-20231201155619418](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231201155619418.png)







![image-20231201155704811](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231201155704811.png)

![image-20231201155912403](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231201155912403.png)

![image-20231201160625876](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231201160625876.png)

![image-20231201160735931](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231201160735931.png)

![image-20231201163209885](C:\Users\xx9z\AppData\Roaming\Typora\typora-user-images\image-20231201163209885.png)

---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/kubernetes/advanced/k8s%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%902/  

