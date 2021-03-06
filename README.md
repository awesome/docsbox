# docsbox [![Build Status](https://travis-ci.org/docsbox/docsbox.svg?branch=master)](https://travis-ci.org/docsbox/docsbox)

```bash
$ curl -F "file=@kittens.docx" http://dev.scrib.ro/api/v1/

{
    "id": "9b643d78-d0c8-4552-a0c5-111a89896176",
    "status": "queued"
}

$ curl http://dev.scrib.ro/api/v1/9b643d78-d0c8-4552-a0c5-111a89896176

{
    "id": "9b643d78-d0c8-4552-a0c5-111a89896176",
    "result_url": "/media/9b643d78-d0c8-4552-a0c5-111a89896176.zip",
    "status": "finished"
}

$ curl -O http://dev.scrib.ro/media/9b643d78-d0c8-4552-a0c5-111a89896176.zip

$ unzip -l 9b643d78-d0c8-4552-a0c5-111a89896176.zip 

Archive:  9b643d78-d0c8-4552-a0c5-111a89896176.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
    11135  2016-07-08 05:31   txt
   373984  2016-07-08 05:31   pdf
   147050  2016-07-08 05:31   html
---------                     -------
   532169                     3 files
```

```bash
$ cat options.json 
{
  "formats": ["pdf"],
  "thumbnails": {
    "size": "640x480",
  }
}

$ curl -i -F "file=@kittens.ppt" -F "options=<options.json" http://dev.scrib.ro/api/v1/

{
  "id": "afb58e2b-78fa-4dd7-b7f9-a64f75f50cb1",
  "status": "queued"
}

$ curl http://dev.scrib.ro/api/v1/afb58e2b-78fa-4dd7-b7f9-a64f75f50cb1

{
  "id": "afb58e2b-78fa-4dd7-b7f9-a64f75f50cb1",
  "status": "finished",
  "result_url": "/media/afb58e2b-78fa-4dd7-b7f9-a64f75f50cb1.zip"
}

$ curl -O http://dev.scrib.ro/media/afb58e2b-78fa-4dd7-b7f9-a64f75f50cb1.zip

$ unzip -l afb58e2b-78fa-4dd7-b7f9-a64f75f50cb1.zip
Archive:  afb58e2b-78fa-4dd7-b7f9-a64f75f50cb1.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
   779820  2016-07-10 02:02   pdf
   177357  2016-07-10 02:02   thumbnails/0.png
                              ...
   130923  2016-07-10 02:02   thumbnails/30.png
---------                     -------
 13723770                     32 files

```

# API

```
POST (multipart/form-data) /api/v1/
file=@kittens.docx
options={ # json, optional
    "formats": ["pdf"] # desired formats to be converted in, optional
    "thumbnails": { # optional
        "size": "320x240",
    } 
}

GET /api/v1/{task_id}
```

# Install
Currently, installing powered by docker-compose:

```bash
$ git clone https://github.com/docsbox/docsbox.git && cd docsbox
$ docker-compose build
$ docker-compose up
```

It'll start this services:

```bash
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
7ce674173732        docsbox_nginx         "/usr/sbin/nginx"        8 minutes ago       Up 8 minutes        0.0.0.0:80->80/tcp       docsbox_nginx_1
f6b55773c71d        docsbox_rqworker      "rq worker -c docsbox"   15 minutes ago      Up 8 minutes                                 docsbox_rqworker_1
662b08daefea        docsbox_rqscheduler   "rqscheduler -H redis"   15 minutes ago      Up 8 minutes                                 docsbox_rqscheduler_1
0364df126b36        docsbox_web           "gunicorn -b :8000 do"   15 minutes ago      Up 8 minutes        8000/tcp                 docsbox_web_1
5e8c8481e288        redis:latest          "docker-entrypoint.sh"   9 hours ago         Up 8 minutes        0.0.0.0:6379->6379/tcp   docsbox_redis_1
```

# Settings (env)

```
REDIS_URL - redis-server url (default: redis://redis:6379/0)
REDIS_JOB_TIMEOUT - job timeout (default: 10 minutes)
ORIGINAL_FILE_TTL - TTL for uploaded file in seconds (default: 10 minutes)
RESULT_FILE_TTL - TTL for result file in seconds (default: 24 hours)
LIBREOFFICE_PATH - path to libreoffice (default: /usr/lib/libreoffice/program/)
```

# Supported filetypes

| Input                              | Output              | Thumbnails |
| ---------------------------------- | ------------------- | ---------- |
| Document (`doc` `docx` `odt`)      | `pdf` `txt` `html`  | `yes`      |
| Presentation (`ppt` `pptx` `odp`)  | `pdf` `html`        | `yes`      |
| Spreadsheet (`xls` `xlsx` `ods`)   | `pdf` `csv` `html`  | `yes`      |
