# Automatic backup your file to Google Drive

Althought I could make the backup file on  server but I don't trust it, sometime  server could be crashed or hard disk get problem.

So this tool help you auto backup your file to google drive by using Google Drive API and this's good service to make sure my file is safe, I think.

it's use if you want make sure your data is backuped daily such as sql data, database, upload folder ,etc.

[Web guide link](http://dethoima.com/tu-dong-backup-du-lieu-len-google-drive/)

### Note

if you want backup a folder, you must compress it first, also, you can compress file to faster backup time.

### Preparing:

1. Install goolge api package for python

		easy_install --upgrade google-api-python-client

   or

		pip install --upgrade google-api-python-client

2. Create google project at [Link](https://console.developers.google.com/cloud-resource-manager?authuser=0).

3. Go to *API & Services* dashboard from sidebar menu

4. Click ENABLE APIS AND SERVICES

5. Search "Google Drive", and select "Google Drive API", enabled the API.

6. Create Credentials, follow the steps.

7. Choose *JSON* format (do not choose p12), and it will automatically downloading json file

8. Create file p12, you can name it with private_key_id (you can get the id from the json file), insert the private key into the file and save the file _private_key_id.p12_

4. You come back to your google drive [drive.google.com](https://drive.google.com) and create share folder( you create an empty folder and right click on the folder and share to user xxxxx-5kfab22qfu82uub2887gi0c9e6eincmu@developer.gserviceaccount.com  ) and copy the folder id ( You can look at the url after visit folder and the id is there ) and in my case the backup folder url is https://drive.google.com/#folders/0B0XTTQmH9aXreFdxS0txVU5Xb1U so that the id is 0B0XTTQmH9aXreFdxS0txVU5Xb1U

5.  Create config file( such as config_file.json ) and input into this file with json format such as

		{
			"service_account":"xxxxx-5kfab22qfu82uub2887gi0c9e6eincmu@developer.gserviceaccount.com",
			"private_key12_path":"private_key_id.p12",
			"backup_folder_id":"0B0XTTQmH9aXreFdxS0txVU5Xb1U",
			"description" : "Description for backup file",
			"max_file_in_folder": 5
		}

You can see "max_file_in_folder" that is the number max file on backup folder, so if I backup new file to that folder and that's sixth file. so the oldest file will be removed, this solution to prevent google drive is out of capacity.



### using

    python backup.py path/configs/config_file.json /path/backup_file.tar.gz

### Automatic backup

*To automatic backup, you can put this script in crontab and decide the schedule for it.*
Below is example 

    00 01 * * * python /my/backup/repository/path/backup.py path/your/your_config.json /path/your/backup/file >>/var/log/drive.log

Above example to set crontab run daily on 1am:00 to backup file with path = */path/your/backup/file* by using config = *path/your/your_config.json*, you can see the result at file */var/log/drive.log*

### Example backup mysql database
Below is script to backup mysql database to google drive by using this tool, You can look at backup_mysql.example.sh

	BACKUP_DIR="/your/backup/folder"
	BACKUP_CODE="/backup/code/folder"
	#Sql file name with year-month-date-hour-minute
	NAME=`date +%Y%m%d-%H%M`
	SQLFILE=$NAME'.sql'
	ZNAME=$NAME'.tar.gz'

	FILE_PATH=$BACKUP_DIR'/'$ZNAME

	DBNAME='your_db_name'
	DBUSER='mysql_user'
	DBPASSWORD='mysql_password'
	echo "Dumping DB name=$DBNAME"
	#Create backup folder if folder is not existing
	mkdir -p $BACKUP_DIR
	cd $BACKUP_DIR
	#Dump sql file to $SQLFILE
	mysqldump -u $DBUSER -p $DBNAME --password=$DBPASSWORD --skip-lock-tables --force >$SQLFILE
	#Zip sql file
	tar -zcvf $ZNAME $SQLFILE
	#Remove sql file
	rm -f $SQLFILE
	#Cd to backup code folder
	cd $BACKUP_CODE
	echo "UPloading to google drive"
	#UPload backup file to google drive
	python backup.py configs/dethoima.com.json $FILE_PATH
	#Remove backup file since file is uploaded to google drive
	rm -f $FILE_PATH
	echo "Done"

### Setup cronjob for backup mysql script
Below is my example 

    00 05 * * * bash /my/backup/repository/backup_mysql.sh

## Restore
**syntax**

```
python restore.py configfile_path restore_path
```

- The configfile__path is same when you backup.
- restore_path is the folder is used to store file is downloaded from  google drive.


**Example**

```
python restore.py configs/abc.com.json /var/www
```

## Other Libraries
Commands

	$ pip install oauth2client==1.5.2
	$ pip install pycrypto
